# React 로 검색 폼 개발하기
> 리액트를 한파일에서 만드는 방식

Main.js // https://github.com/lhyekwang/coding/blob/react-lecture/react_study/2-react/js/main.js
```JS

class App extends React.Component{ // 콤포넌트 만들기
  constructor(){  // 초기화
    super();     // 부모들 몽땅 불러오기
    this.state = {
    }
  }
  ...
  render(){ // 화면그리는 부분    
    return ( 
      <>
        <header>
          <h2 className="container">검색</h2>
        </header>
        <div className="container">
          { searchForm }
          <div className="content">{this.state.submitted ?  searchResult : tabs}</div>
        </div>
      </>
    )
  }
}

ReactDOM.render(<App/>, document.querySelector("#app"));    // id 값이 app 인곳에 렌더링해줌  
```

index.html // https://github.com/lhyekwang/coding/blob/react-lecture/react_study/2-react/index.html
```
    <div id="app"></div>
...
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    ...
```
