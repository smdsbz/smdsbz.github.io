---
layout: article
title: Node.js Notes
tags: Web Javascript
---

Random notes on Nodejs.

### HowTo

#### Start a Waterfall

```javascript
module.exports = async (param) {
  return new Promise((resolve, reject) => {
      
    let cond = doSomething(param);
    let wanted = 'wanted';
    let unwanted = 'unwanted';

    if (cond === wanted) {
      resolve({  // goes to `.then`
        code: 200,  // docs are needed afterwards
        data: 'values here will be parameters of the callback',
        msg:  'of `Promise.then()` method'
      });
    } else if (cond === unwanted) {
      reject({  // goes to `.catch`
        code: 400,
        data: 'values here will be parameters of the callback',
        msg:  'of `Promise.catch()` method'
      });
    } else {
      reject({
        code: 500,
        data: 'unexpected'
      });
    }

  });  // return new Promise
}
```

#### Get Return Value from `Promise` Blocks

```javascript
async () => {  // requires the function block to be async
  let retval = someAsyncFunction()
  .then(retval => {
    return doSomethingElseAsync(retval);  // returns a `Promise`
  })
  .then(retval => {
    return retval;
  })
  .catch(err => {
    console.error(err);
  });
  
  console.log(await retval);  // this line will be executed only
                              // after `retval` is assigned a value
}
```
