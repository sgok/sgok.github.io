---
layout: post
title:  "Kata Valid Braces Solution for PHP"
date:   2022-03-11 19:49:15 +0300
categories: php
excerpt_separator: <!--more-->
---

Kata problem: Write a function that takes a string of braces, and determines if the order of the braces is valid. It should return true if the string is valid, and false if it's invalid.

This Kata is similar to the Valid Parentheses Kata, but introduces new characters: brackets [], and curly braces {}. Thanks to @arnedag for the idea!

All input strings will be nonempty, and will only consist of parentheses, brackets and curly braces: ()[]{}.
<!--more-->

### Task

Write a function that takes a string of braces, and determines if the order of the braces is valid. It should return true if the string is valid, and false if it's invalid.

This Kata is similar to the Valid Parentheses Kata, but introduces new characters: brackets [], and curly braces {}. Thanks to @arnedag for the idea!

All input strings will be nonempty, and will only consist of parentheses, brackets and curly braces: ()[]{}.

##### What is considered Valid?
A string of braces is considered valid if all braces are matched with the correct brace.

#### Examples

```
"(){}[]"   =>  True
"([{}])"   =>  True
"(}"       =>  False
"[(])"     =>  False
"[({})](]" =>  False
```
### My Solution

```php
function validBraces($braces){
    $t = [];
    for($i=0;$i < strlen($braces); $i++){
        if ( $braces[$i] == "(" || $braces[$i] == "{" || $braces[$i] == "["){
          array_push($t, $braces[$i]);
        } else{
          if(count($t) == 0){ 
              return false;
          }else{
              $lastValue = (count($t) - 1);
              if( ($braces[$i] == ']' && $t[$lastValue] == '[') || ($braces[$i] == '}' && $t[$lastValue] == '{') || ($braces[$i] == ')' && $t[$lastValue] == '('))
              {
                array_pop($t);
              } else {
                break;
              }
          }
          
        }
    }
  
  if(count($t) == 0){
      return true;
  }else{
      return false;
  }
    
}
```

### Clever Solution

```php
function validBraces($braces){
    do{
        $braces = str_replace(['()', '[]', '{}'], '', $braces, $count);
    }while($count);
    
    return empty($braces);
}
```
[Solver's Profile](https://www.codewars.com/users/SergeyLevchuk)

Best practices and clever solution for valid braces problem.
