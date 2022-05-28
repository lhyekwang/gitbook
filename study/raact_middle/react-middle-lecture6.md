리덕스 
===
### 리덕스 소개   
- 컴포넌트 코드로 부터 상태 관리 코드를 분리 할 수 있다.   
미들웨어를 활용한 다양한 기능추가   
  >데이타를 처리하는 중간과정에서 어떤 로직을 넣어서 필요한 기능을 추가 할수 있다.   
  - 강력한 미들웨어 라이브러리 (ex. redux-saga) 사용가능함   
  - 로컬 스토리지에 데이타 저장하기 및 불러오기  
- SSR(서버사이드렌더링)시 데이터 전달이 간편하다.   
  >리덕스의 상태값은 하나의 객체로 표현이 가능하다. 그 하나의 객체만 문자열로 변환해서 서버에서 클라이언트로 전달하면 된다.   
  >클라이언트는 받은 문자열을  하나의 객체로 만들어서 사용하면 된다.
  > 클라이언트에서 전체 애플리케이션의 상태를 저장해서 불러오는 기능도 있고, 과거의상태를 저장했다가 과거의 상태로 돌아가는것도 가능
- 리액트 콘텍스트보다 효율적인 렌더링 가능   
### 액션, 미들웨어   
#### 리덕스 구조    
액션 => 미들웨어 => 리듀서 => 스토어 => 뷰 => 액션...   
뷰에서 화면을 바꾸고 싶으면 액션을 발생하고 그 액션을 미들웨어가 기능이 있다면 처리하고 리듀서로 넘어가서 해당액션에 의해서 상태값이 어떻게 변경되는지 로직을 담고 있으며 그 상태값이 어떻게 변하는지 스토어에게 알려준다. 스토어는 그 상태값을 저장하고 스토어에 등록되어있는 여러가지 옵져버들중 그 데이터에 관심있는 옵져버들에게  데이터의 변경사실을 알려준다.   
( 데이타가 변경되지않아도 액션이 발생하면 액션처리가 끝났을때 무조건 옵져버에게 알려준다. )   
뷰그 그 이벤트를 받아서 화면을 갱신한다.   
리덕스에서는 단방향으로 이동하기 때문에 직관적이고 예측가능하다. 
#### 액션    
> 타입 속성값을 가지고 있는 객체
```javascript
  store.dispatch( { type : 'todo/ADD',title : '영화보기', priority : 'high' }), 
  store.dispatch( { type : 'todo/REMOVE', id : 123, }), )
  store.dispatch( { type : 'todo/REMOVE_ALL', }), )
```
dispatch 는 액션이 발생되었다는것을 리덕스에게 알려주는 함수.   
{..} : 액션객체  (// 원하는대로 값을 전달할 수 있다.)   
type 속성값은 유니크 해야 한다.   

```javascript

function addTodo({ title, priority }) {
  return { type : 'todo/ADD' , title, priority };
}
function  removeTodo({ id}) {
  return { type : 'todo/REMOVE' , id };
}
function  removeAllTodo({ id}) {
  return { type : 'todo/REMOVE_ALL' };
}

  store.dispatch( addTodo({ title : '영화보기', priority : 'high' }));
  store.dispatch( removeTodo({ id: 123 }) );
  store.dispatch( removeAllTodo());
```
>일반적으로 action creator 함수를 만들어서 호출 해서 쓴다.   

```javascript
export const ADD = 'todo/ADD';
export const REMOVE = 'todo/REMOVE';
export const REMOVE_ALL = 'todo/REMOVE_ALL';

function addTodo({ title, priority }) {
  return { type : ADD , title, priority };
}
function  removeTodo({ id}) {
  return { type : REMOVE , id };
}
function  removeAllTodo({ id}) {
  return { type : REMOVE_ALL };
}

  store.dispatch( addTodo({ title : '영화보기', priority : 'high' }));
  store.dispatch( removeTodo({ id: 123 }) );
  store.dispatch( removeAllTodo());
```
>type 은 action creator 에서도 사용하기도 하지만 리듀서에서도 나중에 사용할것이기 때문에 상수 변수로 만드는게 좋다.
#### 미들웨어
>미들웨는 함수이다.  함수를 연달아서 두번 리턴하는 함수임.
```javascript
  const myMiddleware = store => next => action => next(action);
  //function 키워드로 보기

  function myMiddleware (store) {
    return function (next) {
      return function (action){
        next(action);
      }
    }
  }
```  
>store 가 실행을 하면 next가 반환이 되는거고 최종적으로 action =>next(action)이 나와서 실행을 하면 next라는걸 호출한다.   
> 여러번 감싼 이유는 action=>next(action) 에서 store, next를 사용하려고  하기 위함이다.   
> 미들웨어 실행시 store 를 많이 사용하기때문에 받아서 사용하고,   
> next 도 리덕스에서 만들어서 넘겨주는데, 다듬에 호출될 함수를 넘겨준다.   

>**store** 는 리덕스 스토어 인스턴스이다. 그 안에 dispatch, getState, subscribe 내장함수들이 들어있음.   
> **next** 는 액션을 다음 미들웨어에게 전달하는 함수이다. 사용방식은 next(action) 이다. 다음 미들웨어가 없다면 리듀서에게 액션을 전달한다.   만약에 next를 호출하지 않게 된다면 액션이 무시처리되어 리듀서에게 전달되지 않는다.
> **action** 은 현재 처리하고 있는 액션 객체

```javascript
import { createStore, applyMiddleware } from 'redux'

const middleware1 = store => next => action=> {
  console.log('middleware1 start');
  const result =  next (action);
  console.log('middleware1 end');
  return result;
}

const middleware2 = store => next => action=> {
  console.log('middleware2 start');
  const result =  next (action);
  console.log('middleware2 end');
  return result;
}

const myReducer = (state, action)  => {
  console.log('myReducer');
  return state;
}

const store = createStore( myReducer, applyMiddleware( middleware1, middleware2 ) );
store.dispatch ( { type : 'someAction' } );

export default function App(){
  return <div> 실전 리액트</div> ;
}

```
콘솔순서   
myReducer > middleware1 start > middleware2 start > myReducer > middleware2 end > middleware1 end   

myReducer 는 초기에 상태값을 초기화 하기 위해서  미들웨어 없이 리듀서만 호출하는 단계가 있다.
그 이후에 액션이 발생했을때는 (//dispatch를 호출했을때는)  로그가 출력이 된다.   
첫번째 middleware1가 실행이 되고, 
'console.log('middleware1 start')' 이게 출력이 되고,    
next 를 호출했을때 (next 는 middleware2 에 입력된 함수를 의미한다.)
'console.log('middleware2 start');' 이게 출력이 된다.
next를 호출했을때 그다음 미들웨어는 없기 때문에 reducer로 간다.
그래서 console.log('myReducer');가 찍히고, 
그다음 나와서    
console.log('middleware2 end'); 가 찍히고, 
return result; 저 부분이 리턴을 하면 next 로 나와서 
console.log('middleware1 end'); 이 찍히는거다.   

#### 미들웨어 활용 예
```javascript
const printing = store => next => action => {
  console.log( `prev state = ${JSON.stringify(store.getState())}` );
  const result = next(action);
  console.log( `next state = ${JSON.stringify(store.getState())}` );
  return result;
};

...
const myReducer = (state = { name : 'mike' } , action)  => {
  console.log('myReducer');
  if (action.type === 'someAction'){
    return {name :'mike2'} ;
  }
  return state;
}
const store = createStore( myReducer, applyMiddleware( printing ) );

```
>store를 상태값을 가져오기 위해서 사용하고 있음. 
> 리듀서가 전달된후에 새로운 상태 값을 확인할수 있다.   
콘솔순서   
myReducer > preve state = { "name" : "mike" } > myReducer >  preve state = { "name" : "mike1" }

```javascript
 const reportCrash = store => next => action => {
   try {
     return next(action);
   } catch (err){
     // 서버로 예외 정보 전송
   }
 }
```
>리듀서에서 뭔가 잘못처리 해서 예외가 발생되었을때 서버로 보내주는 방법.   

```javascript
 const delayAction = store => next => action => {
   const delay = action.meta?.delay; 
   // 이런방식처음봄    
   // ? 는 optional chaning 이라는 기능임.   meta 가 undefined 라면 에러가 발생하지 않음   
    // action.meta && action.meta.delay 와 같은 의미임   
   if( !delay ){
     return next(action);
   }
   const titmeoutId = setTimeout( () => next(action), delay );
   return function cancel(){
     claearTimeout(timeoutId);
   }
 }

const store = createStore( myReducer, applyMiddleware( delayAction ) );
store.dispatch ( { type : 'someAction' ,meta : {delay : 3000}} );

```
**delayAction**   
>액션에 meta에 delay 라는 값이 있을때    
> 없다면 return next(action); 호출하고 끝내고,    
> delay가 있을때는 setTimeout으로 딜레이를 해줘서 리듀서를 늦게 실행하는 코드   
> 그리고 cancle이라는 함수가 반환되어서  next가 실행되는것을  취소될수 있게 해주는것   
> stroe.dispatch 를 다른값으로 받아서 실행을 시킬때 
> 미들웨아가 반환한 값이 최종적으로 displatch 반환값이 된다(?) 받아서 뭔가 처리할수 있다???

로컬스토리지에 저장할수 있음.

### 리듀서, 스토어
>리듀서는 현재상태와 액션 객체를 파라미터로 받아와서 새로운 상태값을 반환해주는 함수이다.   
```javascript
  function reducer (state, action){
    // 새로운 상태를 만드는 로직
    // const nextState = ...
    return nextState;
  }
```

>리덕스의 상태값을 수정하는 유일한 방법은 액션 객체와 함께 dispatch 메서드를 호출하는것이다.   
>상태값은 dispatch 메서드가 호출된 순서대로 리덕스 내부에서 변경되기 때문에 상태값이 변화되는 과정을 쉽게 이해할 수 있다.   

> 상태값은 불변 객체로 관리해야 된다.   
```javascript
  function reducer( state = INITIAL_STATE, action ){ // state 에 undefinded 를 넣어서 reducer를  호출한다. 어디서..???
  // 첫번째 매개 변수는 초기상태값을 빈배열로 설정하고, 
  // 두번째는 액션개체
    switch (action.type){
      //...
      case REMOVE_ALL:
        return {
          ...state, 
          todos : [],
        };
      case REMOVE:
        return {
          ...state, 
          todos : state.todos.filter( todo => todo.id !== action.id ),
        };
      default :
        return state;        

    }
  }
  const INITIAL_SATTE = { todos : [] };
```
수정되어야 하는 값이 깊이 있다면 매번 전개연사자를 사용하는게 번거롭다.   
```javascript

prevState === nextState
//또는..
prevState.user === nextState.user
...
```
>불변객체로 관리하게 되면 단순비교롸 위와같이 사용할 수 있다.

불변객체로 코드를 작성할 수 있도록 immer 라는 라이브러리르 사용하면 좋다.   
```javascript
  import produce from 'immer';

  const person = { name : 'mike' , age : 32 };
  const newPerson = produce ( person, draft => {
    dragt.ag = 32;
  });
```
produce( 변경하고 싶은것,  상태값 변경 로직)   

#### 리듀서 사용시 주의할점
객체를 가르킬때는 객체 레퍼런스가 아니라 고유 아이디값을 이용하는게 좋다.   
리듀서는 순수함수로 작성해야 한다.    
서버 API를 호출하는것도 안되고,   
어떤값이 나올지 모르게 random함수도 안되고,   
타임함수도 마찬가지로 사용하지 않는게 좋다.   
random 이 필요할때는 action 함수를 이용해서 만들어서 데이터를 만들어서 넣어두면 된다.   

#### 스토어   
리덕스에서 스토어를 만들때는 createStore 라는 함수를 이용한다.   
```javascript
  const store = createStore(reducer);
```
store 는 상태값을 저장 하는 역활도 있고,    
액션이 끝났다고 외부에 알려주는 역활도 한다.  
( //subscribe 활용. 액션이 끝나면 subscribe 안에 있는 함수가 호출된다. )  
```javascript
  const INITIAL_STATE = { value : 0 };
  const reducer = createReducer(INITIAL_STAFF , {
    INCREMENT : state => (state.value +=1), 
  })
  const store = createStore(reducer);

  let prevState;
  store.subscribe( ()=>{
    const state = store.getState();
    if (state === preState){
      console.log('상태값 같음');
    }else{
      console.log('상태값 변경됨');
    }
    prevState = state;
  } );

  store.dispatch( {type : 'INCREMENT'} );
  store.dispatch( {type : 'OTHER_ACTION'} );
  store.dispatch( {type : 'INCREMENT'} );

```
콘솔순서   
상태값 변경됨 > 상태값 같음 > 상태값 변경됨   

### react-redux 없이 직접 구현하기   
불필요하게 렌더링 되는 부분 정리하기   

```javascript

import React, {useEffect, useReducer} from 'react';
import store from '../../common /store';
import { getNextFriend } from '../../common /mockData';
import { addFriend } from '../state';
import FriendList from '../../component/FriendList';

export default function FriendMain(){

  const [, forceUpdatae] = useReducer( v => v+1, 0 );

  userEffect( () => {
    const unsubscribe = store.subscribe( () => forceUpdate() );
    return () => unsubscribe();
  } , [] );

  function onAdd() {
    const friend = getNextFriend();
    store.dispatch( addFriend(friend) );
  }

  const friends = store.getState().friend.friends;
  ...
  return (
    <div>
      <button onClick = {onAdd}>친구추가</button>
      <FriendList friends = {friends} />
    </div>
  )
}

//변경하기

userEffect( () => {
  let prevFriends = store.getState().friend.friends;
  const unsubscribe = store.subscribe( () => {
    const friends = store.getState().friend.friends;
    if( prevFriends !== friends ){
      forceUpdate() 
    }
    prevFriends = friends;
    }
  );
  return () => unsubscribe();
} , [] );
```
이전값과 비교해서 렌더링해주는것   

### react-redux 사용하기   
App.js 변경전
```javascript
  import React from 'react';
  import TimelineMain from './timeline/container/TimelineMain';
  import FriendMain from './timeline/container/FriendMain';

  export default function App(){
    return (
      <div>
          <FriendMain />
          <TimelineMain />
        </div>
    )
  }

```
App.js 변경후   
```javascript
  import React from 'react';
  import TimelineMain from './timeline/container/TimelineMain';
  import FriendMain from './timeline/container/FriendMain';
  import { Provier } from 'react-redux';
  import store from './comon/store';

  export default function App(){
    return (
      <Provider store = {store}>
        <div>
          <FriendMain />
          <TimelineMain />
        </div>
      </Provider>
    )
  }

```
FriendMain.js 변경전   
```javascript

import React, {useEffect, useReducer} from 'react';
import store from '../../common /store';
import { getNextFriend } from '../../common /mockData';
import { addFriend } from '../state';
import FriendList from '../../component/FriendList';

export default function FriendMain(){

  const [, forceUpdatae] = useReducer( v => v+1, 0 );

  userEffect( () => {
    let prevFriends = store.getState().friend.friends;
    const unsubscribe = store.subscribe( () => {
      const friends = store.getState().friend.friends;
      if( prevFriends !== friends ){
        forceUpdate() 
      }
      prevFriends = friends;
      }
    );
    return () => unsubscribe();
  } , [] );

  function onAdd() {
    const friend = getNextFriend();
    store.dispatch( addFriend(friend) );
  }

  const friends = store.getState().friend.friends;
  
  return (
    <div>
      <button onClick = {onAdd}>친구추가</button>
      <FriendList friends = {friends} />
    </div>
  )
}
```  
FriendMain.js 변경후   
```javascript

import React, {useEffect, useReducer} from 'react';
import store from '../../common /store';
import { getNextFriend } from '../../common /mockData';
import { addFriend } from '../state';
import FriendList from '../../component/FriendList';
import { useSelector } from 'react-redux';

export default function FriendMain(){ 
  const friends = useSelector ( state => state.friend.friends); // useSelector(선택자 함수 : 리덕스의 상태값 함수)
  function onAdd() {
    const friend = getNextFriend();
    store.dispatch( addFriend(friend) );
  }
  
  return (
    <div>
      <button onClick = {onAdd}>친구추가</button>
      <FriendList friends = {friends} />
    </div>
  )
}
```   
useSelector 훅은 리덕스에서 액션이 처리가 되면, 여기서 반환하는 값의 이전값을 기억했다가 이값이 변경되었을때 이 컴포넌트(FriendMain)를 다시 렌더링을 해준다.   

만약에 여러 개의 상태값을 가져오고 싶을때는 
```javascript
//  여러개의 useSelector를 사용하는 방법
  const friends = useSelector ( state => state.friend.friends); 
  const friends2 = useSelector ( state => state.friend.friends2); 
  const friends3 = useSelector ( state => state.friend.friends3); 

//  배열을 반환 하는 방법
  const [ friends , freinds2, friend3 ] = useSelector ( 
    state => [
      state.friend.friends, 
      state.friend.friends2, 
      state.friend.friends3, 
  ]); 

//  객체형식으로 반환 하는 방법
  const { friends , freinds2, friend3 }= useSelector ( 
    state => ({
      friend : state.friend.friends, 
      friends2 : state.friend.friends2, 
      friends3 : state.friend.friends3, 
    })
  ); 

  // 하지만 배열방식과 객체방식은  값이 변경되지 않아도, 리덕스에서 액션이 처리될 때마다 불필요하게 렌더링이 된다는 단점이 있다.   그래서 두번째 변수  shallowEqual 를 사용해서 렌더링을 할지 말지 결정하는 것이다. 

  import { userSelector, shallowEqual } from 'react-redux';
 
 //배열일경우 
 const [ friends , freinds2, friend3 ] = useSelector ( 
    state => [
      state.friend.friends, 
      state.friend.friends2, 
      state.friend.friends3, 
] , shallowEqual ); 

//객체일경우 
const { friends , freinds2, friend3 }= useSelector ( 
    state => ({
      friend : state.friend.friends, 
      friends2 : state.friend.friends2, 
      friends3 : state.friend.friends3, 
  }) , shallowEqual ); 
```
매번 shallowEqual 를 써주는게 번거롭기 때문에 커스텀 훅을 만들어서 사용하면 좋다.   

```javascript

import { userSelector, shallowEqual } from 'react-redux';

function useMySelecor(selector){
  return userSelector (selector, shallowEqual);
} // 커스텀훅을 사용해서  뒤에 shallowEqual 를 써준다.

function MyComponent(){
  const [value1, value2] = userSelector( state => [state.value1, state.value2] );
  const value3 = userSelector( state => state.value3);
  const [value4] = userSelector( state => [state.value4]);
}
```
useDispatch 사용법   

```javascript
  import { userSelector, shallowEqual } from 'react-redux';

  import default function FriendMain(){
  
    function onAdd(){
      ...
      store.dispatch(addFriend(friend));
    }
  }

  // useDispatch 를 사용해서 store 에서 가져오지 않아도 된다. 
  import { userSelector, shallowEqual , useDispatch } from 'react-redux';

  import default function FriendMain(){
    const dispatch = useDispatch();

    function onAdd(){
      ...
      dispatch(addFriend(friend));
    }
  }
```

### reelect로 선택자 함수 만들기
### 몇 가지 리덕스 사용 팁
### redux-saga 를 이용한 비동기 액션 처리1
### 제너레이터 이해하기
### redux-saga 를 이요한 비동기 액션 처리2