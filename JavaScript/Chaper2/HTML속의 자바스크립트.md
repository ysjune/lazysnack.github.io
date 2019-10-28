## 다루는 내용
1. <script> 요소 사용
2. 인라인 스크립트와 외부 스크립트의 비교
3. 문서 모드가 자바스크립트에 미치는 영향
4. 자바스크립트가 비활성화된 상황에 대한 대비
  
    
### 1. <Script> 요소
여섯가지의 속성이 있다.
  
1. async
    - 비동기 옵션으로 스크립트를 받지만, 페이지 로딩은 방해하지 않음. `외부 스크립트를 불러올 때만 유효`
2. charset
    - 코드의 문자셋 지정 (자주 사용X)
3. defer
    - 문서의 콘텐츠를 완전히 표시할 때까지 스크립트 실행을 지연해도 OK, `외부 스크립트를 불러올 때만 유효`
4. language
    - Deprecated
5. src
    - 외부 파일 위치 지정
    - ```javascript
      <script src="example.js"></script>
      ```
6. type
    - language 대체, 콘텐츠 타입([마임 타입](https://developer.mozilla.org/ko/docs/Web/HTTP/Basics_of_HTTP/MIME_types)) 지정
    - application/javascript, text/javascript(생략가능, default값)
    - ```javascript
        <script type="text/javascript">
        function blah() {}
        ```  
          
* 코드를 가져온 방법과 관계 없이 코드는 위에서 아래로 순차적으로 해석되며, 예외는 defer 과 async 속성이 있을 때 뿐이다.  
    
    > 자바스크립트 파일은 보통 *.js 라는 확장자를 붙이지만, 브라우저는 js 파일을 불러올 때 확장자를 확인하지 않기 때문에 필수는 아니다.  
    > 하지만 서버에서 종종 파일 확장자를 기준으로 응답에 쓸 마임 타입을 결정하는 경우도 있으므로 염두해 둬야 함  
  
* __`<script> 와 </script> 사이에 스크립트 코드가 있고 src 속성을 사용하고 있다면 인라인 코드는 무시됨`__  

* 일반적으로 자바스크립트 코드를 모두 <body> 요소 안에 페이지 콘텐츠 마지막에 쓰는데, 자바스크립트 파일을 전부 <head> 요소에서 불러온다면  
전부 받고, 파싱하고, 해석이 끝날 때까지 페이지 랜더링이 멈추기 때문임 (페이지 렌더링이 지연되면 흰 화면만 보임)
```html
    <html>
      <head>
        <title> title <title>
      </head>
      <body>
        <!-- contents -->
        <script type="text/javascript" src="example1.js"></script>
      </body>
    </html>
```  
  
* defer 속성을 설정하면 해당 요소를 만나는 즉시 코드를 내려받지만 실행은 지연한다.  
외부 스크립트를 불러올 때만 유효하며, 인라인 스크립트의 defer 속성은 무시한다.  
  
* async 속성은 defer 와 같이 외부스크립트에만 적용된다는 점은 비슷하나, 스크립트가 순차적으로 실행된다는 보장이 없다.  
비동기는 페이지의 load 이벤트 전에는 실행되지만, DOM ContentLoaded 이벤트보다 반드시 앞선다는 보장이 없으므로 (비동기니까) DOM 을 조작하는 스크립트는 async 으로 불러오지 않는 편이 좋음  
---
### 2.인라인 코드와 외부 파일
* 가능한 자바스크립트는 외부 파일로 분리하는게 좋음
    1. 괸리 용이
    2. 캐싱되어 빠른 로딩속도
---  
### 3.문서 모드
* 퀵스모드
* 표준모드
---
### 4.<nonscript> 요소
* 브라우저가 스크립트 미지원
```html
  <html>
    <head>
      <title> title </title>
      <script type="text/javascript" src="example.js"></script>
    </head>
    <body>
      <noscript>
        <a>not support</a>
      </noscript>
    </body>
  </html>
```
---  

##### 생각해볼 점
- script 옵션에 대해서 알게되었다. 이전에는 src와 type 밖에 몰랐는데, async 와 defer 에 대해서 알게되었다.  
async 는 자바에서도 익숙하므로 비동기라 생각하면 되겠고, defer 는 promise 에 나오는 deferred 와 관련이 있으려나? (추후 공부하면서 알아보는 걸로)  
또한 type 의 text/javscript는 기본값이라 적지 않아도 된다는 사실은 깨알팁
- async 와 defer 는 상당히 헷갈리는 개념(?) 인 것 같다. 둘 다 외부스크립트를 불러올 때만 유효하며, 당장 실행하지 않는다.  
실제로 책의 내용을 보면 async 와 defer 는 async 가 `실행순서를 보장하지 않는다` 라는 차이점을 들어 설명하는데  
defer 쪽의 설명을 보면 `하지만 현실에서는 defer 속성으로 지연시킨 스크립트가 항상 순서대로 실행되지는 않으며...` 라고 하는데...?
- xhtml 에 대한 내용은 그냥 읽어보고 지나갔는데, 많은 브라우저가 아직은 비호환이라는 말과 실제로도 많이 본 적이 없기에...?
