# react-lecture-3-2
> 조합 하여 각각의 컴포넌트별로 사용함
>
> List 컴포넌트를 **함수화**하고, 가져다 썼던 HistorywordList 와 KeywordList 의 상황에 따라 리액트컴포넌트화를 해서 각각 사용한다. 
>
분리 > 
https://github.com/lhyekwang/coding/commit/cd820e7dc7e96523b6cdbe867518f3c62a7d462e
각 컴포넌트별 분리 > 
https://github.com/lhyekwang/coding/commit/a6d778bbbbb4866c68db45b86b8c1f9f60d7186a


https://github.com/lhyekwang/coding/blob/react-lecture/react_study/3-component/src/components/List.js
```javascript
import React from "react";
import { formatRelativeDate } from "../helpers.js";

const List = ({  
  data = [],
  hasIndex = false, // 숫자
  hasDate = false, // 날짜
  onClick, // 클릭 이벤트
  onRemove, // 삭제 이벤트
}) => {
  const handleClickRemove = (event, keyword) => { // 검색어 삭제이벤트 함수 
    event.stopPropagation(); 
    onRemove(keyword); 
  };

  return (
    <ul className="list">
      {data.map(({ id, keyword, date }, index) => (
        <li key={id} onClick={() => onClick(keyword)}>
          {hasIndex && <span className="number">{index + 1}</span>} // 숫자 
          <span>{keyword}</span>
          {hasDate && <span className="date">{formatRelativeDate(date)}</span>} // 날짜
          {!!onRemove && ( // 삭제이벤트 이벤트니 !! 로 해주면 true false 값을 던져주면 
            <button
              className="btn-remove"
              onClick={(event) => handleClickRemove(event, keyword)}
            />
          )}
        </li>
      ))}
    </ul>
  );
};

export default List;
```

https://github.com/lhyekwang/coding/blob/react-lecture/react_study/3-component/src/components/KeywordList.js
```javascript
import React from "react" ;
import List from "./List.js"
import store from "../Store.js"


export default class KeywordList extends React.Component {
  constructor(){ // 리액트 컴포넌트 초기화 
    super();
    this.state = {
      keywordList : [],
    }
  }


  componentDidMount(){ 
    const keywordList = store.getKeywordList();
    this.setState({ keywordList });
  }


  render(){
    return(
      <List
      data = {this.state.keywordList}
      onClick={this.props.onClick}
      hasIndex // 인덱스값 true
      />
    )
  }
}
```
https://github.com/lhyekwang/coding/blob/react-lecture/react_study/3-component/src/components/HistorywordList.js
```javascript
import React from "react" ;
import List from "./List.js"
import store from "../Store.js"


export default class HistorywordList extends React.Component {


  constructor(){
    super();


    this.state={
      historywordList : []
    }
  }


  componentDidMount(){
    this.fetch();
  }
  
  fetch(){
    const historywordList = store.getHistoryList();
    this.setState({
      historywordList, 
    })
  }


  handleClickRemoveHistory(keyword){ // 확장막는 이벤트는 list 로 옮김
    store.removeHistory(keyword);
    this.fetch();
  }


  render(){
    return(
      <List
        data = {this.state.historywordList}
        onClick={this.props.onClick}
        hasDate // 데이타값 true
        onRemove={ (keyword) =>  this.handleClickRemoveHistory(keyword)} // 삭제이벤트
      />
    )
  }
}
```
