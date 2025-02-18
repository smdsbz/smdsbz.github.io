---
layout: article
title: Swimming in logs with PySpark
key: swimming-in-logs-with-pyspark
tags: Log PySpark BigData Utility Gist
---

Handy helpers for swimming in the sea of logs with PySpark.

<!-- more -->

Setting up
----------

### All imports

```python
from typing import Iterable, Optional, Callable, Any
from pyspark.sql import DataFrame, Row
from pyspark import RDD
import pyspark.sql.functions as F
from pyspark.sql.types import *

import pandas as pd
from matplotlib import pyplot as plt
# %matplotlib widget

import re
from datetime import datetime
import json
import itertools
import collections
```

### Running locally

```python
!python -m pip install -U pip
!python -m pip install pyspark

from pyspark import SparkContext
from pyspark.sql import SparkSession
spark: SparkSession = SparkSession.builder.getOrCreate()
sc: SparkContext = spark.sparkContext
```

Dealing with logs structurally
------------------------------

### Loading from raw text files

```python
def load_logs(
    path: str,
    log_regex: re.Pattern,
    log_schema: dict[str, tuple[DataType, Callable[[str], Any]]],
) -> DataFrame:
    '''
    Loads all log text files into Spark.

    The returned `DataFrame` will have all the columns listed in `log_schema`,
    plus the followings:
    * `raw`: the original log line
    * `seq`: a monotonically increasing field to preserve the ordering in the
        original log file
    * `log_source`: the path to the file where the line of log came from
    
    Lines that do not match `log_regex` will be discarded.

    Parameters
    ----------
    path
        The path where PySpark should read the log text files.

    log_regex
        A Python regex where its named capture groups correspond to those specified
        in `log_schema`.

    log_schema
        The basic schema of logs, described as a mapping from log field names to
        their corresponding PySpark `DataType`s, accompanied with a conversion
        function that converts the string captured from the log line to value of
        its corresponding Python type.
    '''
    @F.udf(returnType=StructType([StructField(n, t) for (n, (t, _)) in log_schema.items()]))
    def extract_log(line: str):
        m = log_regex.match(line)
        if m is None:
            return None
        return tuple(conv(m.group(name)) for name, (_, conv) in log_schema.items())
    return (
        spark.read.text(path)
        .withColumns({
            'raw': F.col('value'),
            'seq': F.monotonically_increasing_id(),
            'log_source': F.input_file_name(),
            '__packed__': extract_log('value'),
        })
        .filter('__packed__ is not null')
        .withColumns({c: F.col(f'__packed__.{c}') for c in log_schema})
        .drop('__packed__')
        .select(*log_schema, 'raw', 'seq', 'log_source')
    )
```

### Classifying and extracting events

```python
def extract_events(
    logs: DataFrame, content_col_name: str,
    event_schema: dict[str, tuple[re.Pattern, dict[str, Callable[[str], Any]]]],
    event_type_col_name: str = 'type',
    param_col_name: str = 'param',
) -> DataFrame:
    '''
    Extract events and their structured parameters.

    This procedure will add the follwing columns to the input `DataFrame`:
    * Event type: the type of the events specified as keys of `event_schema`
    * Event parameters: the extracted event parameters serialized as JSON string

    Log entries that do not match any of the given event schema will be discarded.

    Parameters
    ----------
    logs
        The `DataFrame` containing all logs.

    content_col_name
        The column name of log contents in `logs`.

    event_schema : event_type --> (event_regex, param_name --> convert_fn)
        The schema definitions of events, described as a mapping from event type
        names to their parameter schemas. Parameter schema is described as a tuple
        where the first element is the regex, the second is a mapping from
        parameter names to their type conversion functions.

    event_type_col_name
        The column name of the added field where event type is stored.

    param_col_name
        The column name of the added field where event parameters are stored.
        Defaults to "param".
    '''
    ExpandType = StructType([
        StructField(event_type_col_name, StringType()),
        StructField(param_col_name, StringType()),
    ])
    @F.udf(returnType=ExpandType)
    def match_event(cont: str):
        for event_type, (event_regex, param_schema) in event_schema.items():
            m = event_regex.match(cont)
            if m is None:
                continue
            return (
                event_type,
                json.dumps({
                    name: conv(m.group(name))
                    for name, conv in param_schema.items()
                })
            )
        return None
    return (
        logs
        .withColumn('__expand__', match_event(content_col_name))
        .filter('__expand__ is not null')
        .withColumns({f: F.col(f'__expand__.{f}') for f in ExpandType.fieldNames()})
        .drop('__expand__')
        .select(*ExpandType.fieldNames(), *logs.schema.fieldNames())
    )
```
