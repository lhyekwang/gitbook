# react-lecture-3

> react componenet로 구현 법
> 
> * 파일을 각각분리하고 각각의 파일로 작업한후에 연결해준다. APP.js 에서 제어하므로 전체가 변경되어야 되는 이벤트는 APP.js 에서 관리한다.
> *   App.js 
>     * SearchForm.js < 검색폼
>     * Tabs.js < 탭
>     * List.js < 검색결과

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
https://github.com/lhyekwang/coding/blob/react-lecture/react_study/3-component/src/components/Header.js

```javascript
const Header = (props) => {
  return(
    <header className='container'>{props.title}</header>
  );
};
export default Header;
```

https://github.com/lhyekwang/coding/blob/react-lecture/react_study/3-component/src/components/SearchForm.js
```javascript
const SearchForm = ({ value, onChange, onSubmit, onReset }) => {

  const handleSubmit = (event) => {
    event.preventDefault();
    onSubmit();
  }
...
  return (
    <form 
        onSubmit={ handleSubmit }
...
```


https://github.com/lhyekwang/coding/blob/react-lecture/react\_study/3-component/src/App.js

```javascript
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
