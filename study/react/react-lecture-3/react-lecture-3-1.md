# react-lecture-3-1

## Class 상속

> import KeywordList from "./components/KeywordList.js"
>
> import HistorywordList from "./components/HistorywordList.js"
>
> 상속을 이용해서 각각의 키워드 목록 최근검색어를 만든다.

https://github.com/lhyekwang/coding/blob/e87cffac09c1098a939ec3c02981b4116cdec3f6/react_study/3-component/src/components/List.js
```javascript
import React from "react" ;

export default class List extends React.Component{
  constructor(){
    super();

    this.state = {
      data : [],
    }
  }
  renderItem( item, index ){
    throw "구현하세요"
  }

  render(){
    return(
      <ul className="list">
        {this.state.data.map( (item, index)=>(          
          <li 
            key={item.id}
            onClick = { () => (this.props.onClick (item.keyword) ) }
          >
            { this.renderItem(item, index) }            
          </li>
        ))}
      </ul>
    )
  }
}

```
https://github.com/lhyekwang/coding/blob/react-lecture/react_study/3-component/src/components/KeywordList.js
```javascript
...
export default class KeywordList extends List{

  componentDidMount(){
    const data = store.getKeywordList();
    this.setState({
      data, 
    })
  }

  renderItem(item, index){
    return(
      <>
        <span className="number"> {index + 1}</span>
        <span>{item.keyword}</span>
      </>
    )
  }
}
```
https://github.com/lhyekwang/coding/blob/react-lecture/react_study/3-component/src/components/HistorywordList.js
```javascript
export default class HistorywordList extends List{

```



