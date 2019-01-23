title: 'JavaScript Kata: Find the odd int'
date: 2016-04-26 23:30:33
categories: Kata
tags: [Kata, js]
---
#### Description:
Given an array, find the int that appears an odd number of times.
<!-- more -->
There will always be only one integer that appears an odd number of times.
#### My Solution:

```
function findOdd(A) {
  //happy coding!
  var n = 0;
  for(var i=0; i<A.length; i++){
      n = n^A[i];
   }
  return n;
}
```

#### Test Cases:

```
function doTest(a, n) {
  console.log("A = ", a);
  console.log("n = ", n);
  Test.assertEquals(findOdd(a), n);
}
Test.describe('Example tests', function() {
  doTest([20,1,-1,2,-2,3,3,5,5,1,2,4,20,4,-1,-2,5], 5);
  doTest([1,1,2,-2,5,2,4,4,-1,-2,5], -1);
  doTest([20,1,1,2,2,3,3,5,5,4,20,4,5], 5);
  doTest([10], 10);
  doTest([1,1,1,1,1,1,10,1,1,1,1], 10);
  doTest([5,4,3,2,1,5,4,3,2,10,10], 1);
});
Test.describe('Random tests', function() {
  var i, sz, a, j, n;
  for(i = 0; i < 40; ++i) {
    sz = Math.round(Math.random()*1000+50);
    if (!sz%2) {
      ++sz;
    }
    a = [];
    for(j = 0; j < sz - 1; j+=2) {
      n = Math.round(Math.random()*1000);
      a.push(n);
      a.push(n);
    }
    n = Math.round(Math.random()*1000);
    a.push(n);
    Test.assertEquals(findOdd(a), n);
  }
});
```


