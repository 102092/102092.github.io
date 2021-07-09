---
author: Han
title: Double exclamation
date: 2021-07-09
description: what is the double exclamation of javascript
tags: ['javascript']
ShowToc: true
---


# !! ?
- `!` 는 많이 봤는데.. `!!` 는 생소하다. 
  - 전자는 기본적으로 `boolean` 값을 반전하는 역할을 담당함.
- 영어 이름은.. **double exclamation**

# 사용 목적 
- 명시적으로 형 변환을 하기 위해
- 즉 다른 타입의 데이터를 `boolean` 타입으로 변환해서 판단하기 위해서임.

# 왜 사용하는가?
- `undefined` , `null` 값에 대한 condition check를 위해서 사용하는 것이 아닐까 생각함.

# 예시
```javascript
var case1;  // undefined

console.log("case1  : " + (case1));
// undefined

console.log("!case1 : " + (!case1));
// true

console.log("!!case1: " + (!!case1));
// false

var case2 = true; // 불리언 데이터 

console.log("case2  : " + (case2));
// true

console.log("!case2 : " + (!case2));
// false

console.log("!!case2: " + (!!case2));
// true


var case3 = null; // null 

console.log("case3  : " + (case3));
// null, Boolean(null) -> false

console.log("!case3 : " + (!case3));
// true

console.log("!!case3: " + (!!case3));
// false

```
- javascript 논리연산자 (NOT) 은 입력값을 `boolean` 으로 변환하여 `true` 이면 반전되어 `false` 를 반환, `false` 면 `true` 값을 반환함.

## Boolean 참
- False
```javascript
var bNoParam = new Boolean();
var bZero = new Boolean(0);
var bNull = new Boolean(null);
var bEmptyString = new Boolean('');
var bfalse = new Boolean(false);
var bUndefined = new Boolean(undefined);
```

- True
```javascript
var btrue = new Boolean(true);
var btrueString = new Boolean('true');
var bfalseString = new Boolean('false');
var bSuLin = new Boolean('Su Lin');
var bArrayProto = new Boolean([]);
var bObjProto = new Boolean({});
```

# 참고
- https://ifuwanna.tistory.com/278
- https://penguingoon.tistory.com/178
- https://brianflove.com/2014-09-02/whats-the-double-exclamation-mark-for-in-javascript/#:~:text=If%20you%20have%20ever%20noticed,(true%20or%20false)%20value.
- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean