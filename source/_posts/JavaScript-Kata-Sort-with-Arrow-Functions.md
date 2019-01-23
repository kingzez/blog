title: 'JavaScript Kata: Sort with Arrow Functions'
date: 2016-04-25 23:38:01
categories: Kata
tags: [Kata, js]
---
Sort and Order people by their age using Arrow Functions

<!-- more -->

* Task

Your task is to order a list containg people objects by age using the new Javascript Arrow Functions

* Input

Input will be a valid array with People objects containing an Age and Name

* Output

Output will be a valid sorted array with People objects sorted by Age in ascending order

* Answer Is

```
var OrderPeople = function(people){
  return people.sort( (obj1,obj2) => obj1.age - obj2.age ); 

}
```