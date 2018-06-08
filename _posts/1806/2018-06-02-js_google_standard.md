---
title: "Call by reference in Java"
layout: post
date: 2018-05-06 18:22
image: 
headerImage: true
tag:
- Call by reference
- Call-by-{Value | Reference} 
category: blog
author: betterFLY
description: 자바는 call by reference 매커니즘이다?
sitemap :
  changefreq : daily
  priority : 1.0
---


아직 익숙하지 않은 사람들을 위해, 구글은 깨끗하고 이해할 수 있는 코드를 쓰는 데 있어 가장 좋은 lays out을 제시하는 자바 스크립트(JavaScript) 작성을 위한 [스타일 가이드](https://google.github.io/styleguide/jsguide.html)를 내놓았다.
이러한 규칙은 유효한 JavaScript를 작성하기 위한 어렵고 빠른 규칙이 아니며 원본 파일 전체에 걸쳐 일관되고 매력적인 스타일 선택 사항을 유지하기 위한 규정만 있습니다. 이것은 자바 스크립트에서 특히 흥미 롭습니다. 자바 스크립트는 다양한 스타일 선택을 허용하는 유연하고 관대 한 언어입니다.
Google과 Airbnb는 가장 인기 있는 스타일 가이드 2개를 보유하고 있습니다. 나는 JS 작성에 많은 시간을 할애한다면 그들 모두를 확인하는 것을 추천합니다.
다음은 Google의 JS 스타일 가이드에서 가장 흥미롭고 관련있는 규칙이라고 생각되는 13가지입니다.
그들은 논쟁 거리가 많은 이슈 (탭과 스페이스, 그리고 세미콜론의 사용 방법에 대한 논란이 많은 논점)에서 저를 놀라게 한 몇 가지 더 명확하지 않은 사양에 이르기까지 모든 것을 처리합니다. 그들은 앞으로 JS를 쓰는 방식을 확실히 바꿀 것입니다.
저는 이글에서 각 규칙마다 사양에 대한 내용을 정리하고 규칙을 자세히 설명하는 스타일 가이드를 예로하는 예시를 제공하겠습니다. 해당하는 경우 실제로 사용하는 스타일의 예를 제공하고 규칙을 따르지 않는 코드와 대조하겠습니다.


### 탭이 아닌 공백을 사용하십시오. (Use spaces, not tabs)

>줄 경계선 시퀀스와는 별도로 ASCII 수평 공백 문자-horizontal space character (0x20)는 소스 파일의 모든 곳에 나타나는 유일한 공백 문자입니다. 탭 문자는 들여 쓰기에 사용되지 않습니다.


```javascript 
    // bad
    function foo() {
    ∙∙∙∙let name;
    }
    
    // bad
    function bar() {
    ∙let name;
    }
    
    // good
    function baz() {
    ∙∙let name;
    }

```


### 세미콜론이 필요합니다. (Semicolons ARE required)

>모든 명령문은 세미콜론으로 끝나야합니다. 자동 세미콜론 삽입에 의존하는 것은 금지되어 있습니다.

왜 누군가가이 아이디어에 반대하는지 상상할 수는 없지만 JS에서 세미콜론을 일관되게 사용하는 것이 새로운 '공간 vs 탭' 논쟁이 되고 있습니다. 구글은 세미콜론을 방어하기 위해 이곳에 단호하게 나왔습니다.



```javascript 
    // bad
    let luke = {}
    let leia = {}
    [luke, leia].forEach(jedi => jedi.father = 'vader')
    // good
    let luke = {};
    let leia = {};
    [luke, leia].forEach((jedi) => {
      jedi.father = 'vader';
    });
```


### ES6 모듈을 사용하지 마십시오 (아직은..)-Don’t use ES6 modules (yet)

>의미론이 아직 확정되지 않았으므로 ES6 모듈 (즉, 내보내기(export) 및 가져 오기(import) 키워드)을 사용하지 마십시오. 이 정책은 의미가 완전히 표준화되면 다시 검토됩니다.


```javascript 
    // Don't do this kind of thing yet:
    //------ lib.js ------
    export function square(x) {
        return x * x;
    }
    export function diag(x, y) {
        return sqrt(square(x) + square(y));
    }
    
    //------ main.js ------
    import { square, diag } from 'lib';

```


### 가로 정렬(Horizontal alignment)은 권장되지 않습니다 (그러나 금지되지는 않음). 

> 이러한 관행은 허용되지만 일반적으로 Google 스타일에서는 권장하지 않습니다. 이미 사용 된 위치에서 가로 정렬(Horizontal alignment)을 유지할 필요가 없습니다.
 Horizontal alignment는 코드에 추가 공백을 가변적으로 추가하여 이전 줄의 특정 토큰 바로 아래에 특정 토큰을 표시하는 연습입니다.


```javascript 
    // bad
    {
      tiny:   42,  
      longer: 435, 
    };
    // good
    {
      tiny: 42, 
      longer: 435,
    };

```


### 더 이상 var를 사용하지 마십시오. - Don’t use var anymore

>모든 로컬 변수를 const 또는 let으로 선언하십시오. 변수를 재 할당해야하는 경우가 아니면 const를 사용하십시오. var 키워드는 사용하지 않아야합니다.

StackOverflow 및 다른 곳에서 코드 샘플에서 var를 사용하는 사람들은 여전히 볼 수 있습니다. 누가 그것으로 인해 사건을 만들지, 아니면 단지 열심히 죽어 가고있는 옛 습관의 사건 일지는 알 수 없습니다.

```javascript 
    // bad
    var example = 42;
    // good
    let example = 42;
```


### 화살표 기능 선호 - Arrow functions are preferred

> arrow functions는 간결한 구문을 제공하고 this에 대한 많은 어려움을 수정합니다. function 키워드보다 arrow functions를 선호합니다. 특히 nested functions의 경우 유용합니다.

나는 솔직히 말해서, 나는 arrow functions이 더 간결하고 보기에 더 좋았 기 때문에 좋다고 생각했다. 그들이 중요한 목적을 달성하는 것으로 밝혀졌습니다.

```javascript 
    // bad
    [1, 2, 3].map(function (x) {
      const y = x + 1;
      return x * y;
    });
    
    // good
    [1, 2, 3].map((x) => {
      const y = x + 1;
      return x * y;
    });

```

### 연결 대신 템플릿 문자열 사용 - Use template strings instead of concatenation

>multiple string literals이 관련되어있는 경우 특히 complex string 연결에 대해 Template string (`로 구분)을 사용하십시오. Template strings은 여러 줄에 걸쳐있을 수 있습니다.

```javascript 
    // bad
    function sayHi(name) {
      return 'How are you, ' + name + '?';
    }
    
    // bad
    function sayHi(name) {
      return ['How are you, ', name, '?'].join();
    }
    
    // bad
    function sayHi(name) {
      return `How are you, ${ name }?`;
    }
    
    // good
    function sayHi(name) {
      return `How are you, ${name}?`;
    }

```


### 긴 문자열에는 줄 연속을 사용하지 마십시오. - Don’t use line continuations for long strings

>일반 또는 template string literal에서 줄 연속을 사용하지 마십시오. 즉, string literal에서 backslash로 줄을 끝냅니다. ES5에서는 이것을 허용하지만 slash 뒤에 공백이 오는 경우에는 까다로운 오류가 발생할 수 있으며 독자에게는 덜 분명합니다.

흥미롭게도 이것은 Google과 Airbnb가 동의하지 않는 규칙입니다 [(Airbnb’s spec 참조).](https://github.com/airbnb/javascript#strings--line-length)
Google은 긴 strings을 연결하는 것을 권장하지만 (아래 그림 참조) Airbnb의 스타일 가이드는 본질적으로 아무 것도하지 말고 긴 문자열을 필요할 때까지 계속 사용할 것을 권장합니다.

```javascript 
    // bad (sorry, this doesn't show up well on mobile)
    const longString = 'This is a very long string that \
        far exceeds the 80 column limit. It unfortunately \
        contains long stretches of spaces due to how the \
        continued lines are indented.';
        
    // good
    const longString = 'This is a very long string that ' + 
        'far exceeds the 80 column limit. It does not contain ' + 
        'long stretches of spaces since the concatenated ' +
        'strings are cleaner.';
```


### "for ... of"는 'for loop'의 올바른 유형입니다.

>ES6에서 이 언어는 이제 3 가지 다른 종류의 forloops를 갖습니다. 가능한 경우for-of loops를 선호해야하지만 모두를 사용할 수 있습니다.

당신이 나에게 묻는다면 이것은 이상한 것이지만, Google이 for loop의 기본 유형을 선언하는 것이 꽤 흥미 롭기 때문에 그것을 포함시킬 것이라고 생각했습니다.
저는for... in loops는 객체에 더 좋았고 for... of 는 배열에 더 적합하다고 생각했습니다. 올바른 직업을위한 올바른 도구 유형의 상황.
Google의 사양은 반드시 그 아이디어와 상반되는 것은 아니지만, 특히이 loop에 대한 선호도가 있다는 것을 아는 것은 여전히 흥미 롭습니다.


### eval ()을 사용하지 마십시오.

>eval 또는 Function(...string) 생성자를 사용하지 마십시오 (code loaders제외). 이러한 기능은 잠재적으로 위험하며 단순히 CSP 환경에서 작동하지 않습니다.

eval()의 [MDN page](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/eval)에는 "eval을 사용하지 마십시오!"라는 섹션이 있습니다.

```javascript 
    // bad
    let obj = { a: 20, b: 30 };
    let propName = getPropName();  // returns "a" or "b"
    eval( 'var result = obj.' + propName );
    
    // good
    let obj = { a: 20, b: 30 };
    let propName = getPropName();  // returns "a" or "b"
    let result = obj[ propName ];  //  obj[ "a" ] is the same as obj.a
```

### Constants는 underscores로 구분 된 ALL_UPPERCASE에 명명되어야합니다.

>Constant 이름은 CONSTANT_CASE를 사용합니다. 단어는 모두 underscores로 구분하여 대문자로 모두 사용하십시오.

변수(variable)가 변경되지 않아야한다고 절대적으로 확신하는 경우, 상수(constant)의 이름을 대문자로 표시하여 이를 나타낼 수 있습니다. 이렇게하면 상수(constant)의 불변성이 코드 전체에서 사용됨에 따라 명확 해집니다.
이 규칙의 주목할만한 예외는 상수(constant)가 함수 범위(function-scoped) 인 경우입니다. 이 경우 camelCase로 작성해야합니다.

```javascript 
    // bad
    const number = 5;
    
    // good
    const NUMBER = 5;

```

### 선언(declaration) 당 하나의 변수(variable)

>모든 지역 변수( local variable) 선언은 하나의 변수만(only one variable) 선언합니다. let a = 1, b = 2; 가 사용되지 않습니다.

변수(variable)가 변경되지 않아야한다고 절대적으로 확신하는 경우, 상수(constant)의 이름을 대문자로 표시하여 이를 나타낼 수 있습니다. 이렇게하면 상수(constant)의 불변성이 코드 전체에서 사용됨에 따라 명확 해집니다.
이 규칙의 주목할만한 예외는 상수(constant)가 함수 범위(function-scoped) 인 경우입니다. 이 경우 camelCase로 작성해야합니다.

```javascript 
    // bad
    let a = 1, b = 2, c = 3;
    
    // good
    let a = 1;
    let b = 2;
    let c = 3;
```

### Use single quotes, not double quotes

>Ordinary string literals은 큰 따옴표-double quotes ( ") 대신 작은 따옴표-single quotes ( ')로 구분됩니다.
 팁 : string에 작은 따옴표(single quotes) character가 포함되어 있으면 따옴표를 escape하지 않아도 되도록 template string을 사용하는 것이 좋습니다.

변수(variable)가 변경되지 않아야한다고 절대적으로 확신하는 경우, 상수(constant)의 이름을 대문자로 표시하여 이를 나타낼 수 있습니다. 이렇게하면 상수(constant)의 불변성이 코드 전체에서 사용됨에 따라 명확 해집니다.
이 규칙의 주목할만한 예외는 상수(constant)가 함수 범위(function-scoped) 인 경우입니다. 이 경우 camelCase로 작성해야합니다.

```javascript 
    // bad
    let directive = "No identification of self or mission."
    
    // bad
    let saying = 'Say it ain\u0027t so.';
    
    // good
    let directive = 'No identification of self or mission.';
    
    // good
    let saying = `Say it ain't so`;

```

--- 
### 최종 메모

제가 처음에 말했듯이, 이것들은 명령이 아닙니다. Google은 많은 기술 대기업 중 하나 일뿐입니다.
즉, 우수한 코드를 작성하는 데 많은 시간을 할애하는 훌륭한 사람들을 고용하고있는 Google과 같은 회사가 제안한 스타일 권장 사항을 살펴 보는 것이 흥미 롭습니다.
'Google 준수 소스 코드(‘Google compliant source code’ )'에 대한 가이드 라인을 따르고 싶다면 이 규칙을 따라야합니다. 물론 많은 사람들이 동의하지 않을 수 있으며 이 중 일부 또는 전부를 마음껏 쓸 수 있습니다.
저는 개인적으로 Airbnb의 사양이 Google보다 더 매력적이라고 생각합니다. 이러한 특정 규칙을 취하는 태도에 상관없이 모든 종류의 코드를 작성할 때 문체의 일관성을 염두에 두는 것이 중요합니다.


---
원본 링크 [https://www.vobour.com/@early-adopter](https://www.vobour.com/%EA%B5%AC%EA%B8%80%EC%9D%80-%EC%9E%90%EB%B0%94-%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%8A%A4%ED%83%80%EC%9D%BC-%EA%B0%80%EC%9D%B4%EB%93%9C%EB%A5%BC-%EB%B0%9C%ED%96%89-%ED%95%A9%EB%8B%88%EB%8B%A4-%EB%8B%A4%EC%9D%8C%EC%9D%80-%EB%AA%87-%EA%B0%80%EC%A7%80-%ED%95%B5%EC%8B%AC)