---
layout: article
title: Pandas and Numpy Notes
key: pandas-and-numpy-notes
tags: python
---

`pandas` and `numpy` quick-ref list index

<!-- more -->

### HowTo

#### Reading the **Notorious** Excel files

```python
df = pd.read_excel(filename,
                   skiprows=num_title_rows,
                   skip_footer=num_footer_rows)
# normal pd.DataFrame operations
```

#### select where col = 'some-col'

```python
# 1. create a Boolean mask
mask = df['col'] == 'some-col'
# 2. apply the mask to the DF
cursor = df[mask]

# or simpler all-in-once
cursor = df[df['col'] == 'some-col']
```

#### select where col contains sub-string 'das' or 'der'

```python
# 1. get col series
series = df['col'].str
# 2. make mask
mask = series.contains('das|der')   # regexp supported!
# 3. apply mask to DF
cursor = df[mask]
```

#### Append A Column of Ones

```python
In [4]: l1 = np.random.randint(0, 10, (3, 4))

In [5]: l1
Out[5]:
array([[5, 7, 2, 8],
       [0, 0, 0, 7],
       [0, 8, 6, 7]])

In [6]: np.concatenate([l1, np.ones((3, 1))], axis=1)
Out[6]:
array([[ 5.,  7.,  2.,  8.,  1.],
       [ 0.,  0.,  0.,  7.,  1.],
       [ 0.,  8.,  6.,  7.,  1.]])

In [7]: _
```
