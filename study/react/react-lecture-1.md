# 1-st

## vanilla JS로 검색 사이트 개발

### 특징
티끌같은 JS 기반지식으로 볼때 신기했고 생소했던것 위주로 정리

1. 파일간의 함수나 클래스를 import 와 export를 이용해서 주고 받음 

```js
export function qs(selector, scope = document) {
...

```
2. Class 생성 초기화

```js
export default class Controller { // 클래스생성
  constructor( 
    store, ...
  ){
    this.store = store // 초기화
  }
  render() {... // 페이지그리기  
...
  }
}
```

3. 위에처럼 불러와서 초기화 할수 있다.

```js
  import {delegate, qs, formatRelativeDate } from "../helpers.js"; // 함수 불러 오고
  ...
  export default class HistoryListView extends KeywordListView{ // HistoryeListView 라는 애를 내보낼려고 할때 
  
  constructor(){
    super(qs('#history-list-view') , new Template() ) // 일단 불러와서 (super) 초기화함 (constructor)
    ...
    }
  }
  
  class Template {
    ...
  }   
  
  ...
```



### 코드보기
https://github.com/lhyekwang/coding/tree/1519550e41b6933cf2ed5512556999811492660c/react_study/1-vanilla 

