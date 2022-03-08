---
layout: post
title:  "Kata Sort the Odd Solution for PHP"
date:   2022-03-08 18:10:15 +0300
categories: php
excerpt_separator: <!--more-->
---

Kata problem: You will be given an array of numbers. You have to sort the odd numbers in ascending order while leaving the even numbers at their original positions.
<!--more-->

### Task

You will be given an array of numbers. You have to sort the odd numbers in ascending order while leaving the even numbers at their original positions.

#### Examples

```
[7, 1]  =>  [1, 7]
[5, 8, 6, 3, 4]  =>  [3, 8, 6, 5, 4]
[9, 8, 7, 6, 5, 4, 3, 2, 1, 0]  =>  [1, 8, 3, 6, 5, 4, 7, 2, 9, 0]
```
### My Solution (too long and impractical)

```php
function sortArray(array $arr) : array {
    $oArr = [];
    $eArr = [];
    $newArray = [];
    foreach($arr as $array){
        if(($array % 2) == 1){
           array_push($oArr, $array);
        }else if(($array % 2) == 0){
            array_push($eArr, $array);
        }
    }
    sort($oArr);
    for ($i = 0; $i < count($arr); $i++) {
        if ($arr[$i]%2 === 0) {
          array_push($newArray, array_shift($eArr));
        } else {
          array_push($newArray, array_shift($oArr));
        }
    }
    
    return $newArray;
}
```
I'm not suggesting you my own solution because I chose to solve it the long way.

### Best Practices Solution

```php
function sortArray(array $arr) : array {
  $odds = array_filter($arr, function ($n) { return $n % 2 != 0; });
  sort($odds);
  return array_map(function ($n) use (&$odds) {
    if ($n % 2 == 0) return $n;
    return array_shift($odds);
  }, $arr);
}
```
[Solver's Profile](https://www.codewars.com/users/dfhwz)

Best practices and clever solution for sort the odd.
