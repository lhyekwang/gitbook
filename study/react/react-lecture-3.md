# react-lecture-3

> react componenet 로 만들기
> **클래스이용법**
> *   App.js 
>
>     * SearchForm.js < 검색

각 컴포넌트 파일별 특징에 **함수**로 만들지 **클래스**로 만들지 결정해야 한다.

함수 : 내부상태가 정적인 상태인경우, state 가 필요없는 경우는 함수를 사용한다. 기존에 사용했었지만 state를 관리하는 상위 컴포넌트를 제외하고 함수로 고쳐주게되면 코드도 짦아질뿐아니라 가독성이 좋아진다. **props**를 활용해서 데이타를 한곳에서 관리한다.

**함수형**

https://github.com/lhyekwang/coding/blob/react-lecture/react\_study/3-component/src/components/SearchForm.js

```
const SearchForm = ({ value, onChange, onSubmit, onReset }) => {
...
  return (
  ...
  )
}
export default SearchForm;
```

https://github.com/lhyekwang/coding/blob/react-lecture/react\_study/3-component/src/App.js

```
...
import SearchForm from "./components/SearchForm.js"
...
export default class App extends React.Component {
    constructor(){
      super();

      this.state = { 
        ...
      };
    }
    render(){
    const { searchKeyword , searchResult, submitted, selectedTab} = this.state;
    return(
    <>
          <SearchForm 
            value = {searchKeyword}
            onChange={(value) => this.handleChangeInput(value)}
            onSubmit ={ () => this.search(searchKeyword)} 
            onReset = { () => this.handleReset() }
          />
    </>
    )
}

```
