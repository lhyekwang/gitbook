리덕스 
===
### 리덕스 소개   
- 컴포넌트 코드로 부터 상태 관리 코드를 분리 할 수 있다.   
- 미들웨어를 활용한 다양한 기능추가   
  >데이타를 처리하는 중간과정에서 어떤 로직을 넣어서 필요한 기능을 추가 할수 있다.   
  - 강력한 미들웨어 라이브러리 (ex. redux-saga) 사용가능함   
  - 로컬 스토리지에 데이타 저장하기 및 불러오기  
- SSR(서버사이드렌더링)시 데이터 전달이 간편하다.   
  >리덕스의 상태값은 하나의 객체로 표현이 가능하다. 그 하나의 객체만 문자열로 변환해서 서버에서 클라이언트로 전달하면 된다.   
  >클라이언트는 받은 문자열을  하나의 객체로 만들어서 사용하면 된다.
  > 클라이언트에서 전체 애플리케이션의 상태를 저장해서 불러오는 기능도 있고, 과거의상태를 저장했다가 과거의 상태로 돌아가는것도 가능
- 리액트 콘텍스트보다 효율적인 렌더링 가능
> 하나의 context로 관리하게되면 내부 상태값이 여러개일경우 렌더링이 필요없는 컴포넌트의 경우에도 렌더링이 된다.
>> 각각만들던가.. 컨텍스트안에서 자동으로 provider 를 생성하던가.. 
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
액션 type 속성값은 유니크 해야 한다.   

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
>미들웨어는 함수이다.  함수를 연달아서 두번 리턴하는 함수임.
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
> 세번 감싼 이유는 action=>next(action) 에서 store, next를 사용하려고  하기 위함이다.   
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
store.dispactch( {type : 'someAction'} )

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
   // ? 는 optional chaning 이라는 기능임.   meta 가 undefined 라도 에러가 발생하지 않음   
    // action.meta && action.meta.delay 와 같은 의미임   
   if( !delay ){
     return next(action);
   }
   const titmeoutId = setTimeout( () => next(action), delay );
   return function cancel(){
     claearTimeout(timeoutId);
   }
 }
 const myReducer = (state = { name:'mink' }, action) =>{
   console.log('myReducer');
   if(action.type === 'someAction'){
     return { name : 'mk23'};
   }
   return state;
 }

const store = createStore( myReducer, applyMiddleware( delayAction ) );
store.dispatch ( { type : 'someAction' ,meta : {delay : 3000}} );

```
**delayAction**   
>액션에 meta에 delay 라는 값이 있을때    
> 없다면 return next(action); 호출하고 끝내고,    
> delay가 있을때는 setTimeout으로 딜레이를 해줘서 리듀서를 늦게 실행하는 코드   
```javascript
const cancel = store.dispatch ( { type : 'someAction' ,meta : {delay : 3000}} );
cancel();
```
> 그리고 cancle이라는 함수가 반환되어서  next가 실행되는것을  취소될수 있게 해주는것   
> stroe.dispatch 를 다른값으로 받아서 실행을 시킬때 
> 미들웨아가 반환한 값이 최종적으로 displatch 반환값이 된다(?) 받아서 뭔가 처리할수 있다???

로컬스토리지에 저장할수 있음. (//코드생략)
### 리듀서, 스토어
>리듀서는 현재상태와 액션 객체를 파라미터로 받아와서 새로운 상태값을 반환해주는 함수이다.   
>리덕스의 상태값을 수정하는 유일한 방법은 액션 객체와 함께 dispatch 메서드를 호출하는것이다.   
```javascript
  function reducer (state, action){
    // 새로운 상태를 만드는 로직
    // const nextState = ...
    return nextState;
  }
```
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

  const person = { name : 'mike' , age : 22 };
  const newPerson = produce ( person, draft => {
    draft.age = 32;
  });
```
produce( 변경하고 싶은것,  상태값 변경 로직)   

#### 리듀서 사용시 주의할점
객체를 가르킬때는 객체 레퍼런스가 아니라 고유 아이디값을 이용하는게 좋다.   
(// 값이 변경되면 객체가 새로 생성이 된다. 다른곳에서 바라보고 있는 건 변경되기전의 값을 참조하게 된다. 그래서 객체를 가르킬때는 객체의 레버런스가 아니라 고유아이디값을 바라보아야 한다. )  
```javascript
switch (action, type){
  case SET_SELECTED_POEPLE;
    draft.selectedPeople = draft.peopleList.find(
      item => item.id === action.id,
    );
    // 아래와 같이 바꿈.
    draft.selectedPeople = action.id
  break;
  case EDIT_PEOPLE_NAME;
    const pople = draft.peopleList.find(item => item.id === action.id);
    people.name = action.name;
  break;
}
``` 
리듀서는 순수함수로 작성해야 한다.    
(// 부수효과가 없어야 한다. 부수효과라는것은 외부 상태를 변경하는것. )
서버 API를 호출하는것도 안된다.)   
어떤값이 나올지 모르게 random함수도 안되고,   
타임함수도 마찬가지로 사용하지 않는게 좋다.   
random 이 필요할때는 action 객체를 만들때, 그때 random을 호울해서 액션 객체를 만들어서 그걸로 데이터를 만들어서 넣어주면 된다.   
```javascript
  function createReducer(state = initialState, action){
    return produce(state = innitialState, action){
      const handler = handlerMap[action.typee];
      if (handler) {
        handler(draft, action);
      }
    }
  }
```
>> createReducer 를 만들어서 사용하면 좋다
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
    return () => unsubscribe(); // 액션이 변경될때마다 무조건 컴포넌트를 업데이트 한다. 
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
  let prevFriends = store.getState().friend.friends; // 이전데이타 저장해서 비교한다.
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
>이전값과 비교해서 렌더링해주는것   

### react-redux 사용하기   
provier 를 root 페이지에 사용함.
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
  const friends = useSelector ( state => state.friend.friends); // useSelector(선택자 함수 ) 리덕스의 상태값 함수) 사용하곧자 하는 데이타 가져옴.
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

  import { useSelector, shallowEqual } from 'react-redux';
 
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

import { useSelector, shallowEqual } from 'react-redux';

function useMySelecor(selector){
  return useSelector (selector, shallowEqual);
} // 커스텀훅을 사용해서  뒤에 shallowEqual 를 써준다.

function MyComponent(){
  const [value1, value2] = useMySelecor( state => [state.value1, state.value2] );
  const value3 = useMySelecor( state => state.value3); // value3이 가자고 있는 모든 값을 비교한다. 비효율적
  const [value4] = useMySelecor( state => [state.value4]); // 이렇게 배열로 해줘야  좋다.
}
```
useDispatch  사용법   

```javascript
  import { useSelector, shallowEqual } from 'react-redux';

  import default function FriendMain(){
  
    function onAdd(){
      ...
      store.dispatch(addFriend(friend));
    }
  }

  // useDispatch 를 사용해서 store 에서 가져오지 않아도 된다. 
  import { useSelector, shallowEqual , useDispatch } from 'react-redux';

  import default function FriendMain(){
    const dispatch = useDispatch();

    function onAdd(){
      ...
      dispatch(addFriend(friend));
    }
  }
```
>dispatch를 훅으로 가져온다는건 dispatch 가변할수도 있다는 말이다. 변할일은 없지만 중간에 미들웨어를 추가하거나 하면 아마 dispatch 가 변경될것이다. 
### reselect로 선택자 함수 만들기   
리덕스에 저장된 데이타를 화면에 보여줄때 는 다양한 형식으로 가공할 필요가 있다.   
모든 상황을 데이타에 넣어서 가져는 방식이 있는데, 이는 리덕스에 데이타가 중복해서 들어가거나, filter 의 조건이 복잡할때 불필요하게 데이타를  가져와 성능에 단점이 있음.    
리덕스에는 원본데이타만 저장해두고, 필터연산을 컴포넌트에서 하는 방법이 있다.    
FriendMain.js   
```javascript
export default function FriendMain(){
  const [
    ageLimit, 
    showLimit, 
    friendWithAgeLimit, 
    friendsWithAgeShowLimit,
  ] = useSelector( state => {
    const { ageLimit, showLimit, friends } = state.friend;
    const friendWithAgeLimit = friends.filter( item => item.age <= ageLimit ); // friend와 ageLimit 이 발생하지 않아도 filter 연상은 발생된다. 
    return [
      ageLimit, 
      showLimit, 
       friendWithAgeLimit, 
        friendWithAgeLimit.slice( 0, showLimit );
    ];
  }, shallowEqual);
}
```
reselect 사용   
state/selector.js 를 생성해서 데이타드을 가져온다.   
```javascript
import { createSelector } from 'reselect' ;

const getFriends = state => state.friend.friends;
export const getAgeLimit = state => state.friend.ageLimit;
export const getShowLimit = state => state.friend.getShowLimit;

export const getFriendsWithAgeLimit = createSelector(
  [ getFriends, getAgeLimit ], 
  ( friend, ageLimit ) => friends.filter( item=>item.age <= ageLimit ),
);

export const getFriendsWithAgeShowLimit = createSelector(
  [ getFriendsWithAgeLimit , getShowLimit ], 
  ( friendsWithAgeLimit, showLimit ) => friendsWithAgeLimit,slice( 0, showLimit);
)
```
>createSelector 함수를 이용해서 선택자 함수를 만들면, 메모이제이션 기능이 동작한다.       
friends.js. 에서  사용한다. 
```javascript
import { getAgeLimit } from '../state'

export default function FriendMain(){

  const [
    ageLimit, 
    showLimit, 
    friendWithAgeLimit, 
    friendsWithAgeShowLimit,
  ] = useSelector( 
    state = [
      getAgeLimit(state),
      getShowLimit(state),
      getFriendsWithAgeLimit(state),
      getFriendsWithShowLimit(state),
    ]
    , shallowEqual,
  );

  // 아래도 같은 방식
const ageLimit = useSelector(getAgeLimit);
const showLimit = useSelector(getShowLimit);
const friendWithAgeLimit = useSelector(getFriendsWithAgeLimit);
const friendsWithAgeShowLimit = useSelector(getFriendsWithShowLimit);

}
```
데이타를 가공하는 연산이 컴포넌트코드에 넣지 않는다.   ( // selector 에 넣는다. )
속성값이 변경되는건 메모이제이션 기능이 동작하지 않는다. 그래서 여러가지 방법으로 작업을 추가로 한다. 나중에 성능이슈가 생겼을때 작업해줘도 된다.   

### 몇 가지 리덕스 사용 팁   
#### redux-helper
: 상태값을 추가한다면, 액션도 추가해야 하고, 액션 크리에이터도 추가해야 한다. ageLimit 이나 showLimit 처럼 리듀서에서 간단하게 할당만 할꺼라면 간편하게 작성할수 있다.   
: 상태값을 변경하는 액션 SET_XXX , 와 액션 크리에이티브 set_xxx()함수를 한개씩 만들어서 사용했음. 

redux-helper.js
```javascript
  import produce from 'immer';

  export function createReducer(initialState, handlerMap) {
    return function (state = initialState, action) {
      return produce (state, draft => {
        const handler = handlerMap[action.type];      
        if (handler) {
          handler(draft, action);
        } 
      })
    };
  }
  export function createSetValueAction(type) {
    return (key, value) => ({ type, key, value });
  }
  export function setValueReducer(state, action) {
    state[action.key] = action.value;
  }
```
index.js   
```javascript

  // const SET_AGE_LIMIT = 'friend/SET_AGE_LIMIT' 
  // const SET_SHOW_LIMIT = 'friend/SET_SHOW_LIMIT' 
  const SET_VALUE = 'friend/SET_SHOW_LIMIT' 

  // export const setAgeLimit = ageLimit => ( { type :SET_AGE_LIMIT, ageLimit } ) ;
  // export const setShowLimit = showLimit => ( { type :SET_SHOW_LIMIT, ageLimit } ) ;
  export const setValue = createSetValueAction(SET_VALUE);

  
  const reducer = createReducer(INNITAL_STATE, {
    ...
    // [SET_AGE_LIMIT] : (state, action) => (state.ageLimit = action.ageLimit),
    // [SET_SHOW_LIMIT] : (state, action) => (state.showLimit = action.showLimit),
    [SET_VALUE] : setValueReducer,
  })

```
FriendMain.js
```javascript
...
  return (
    <NumberSelect
      // onChange = { v => dispatch(setAgeLimit(v))}
      onChange = { v => dispatch(setValue( 'ageLimit', v ))} //상태값을 저장할 이름과 값
      ...
    >
  )
```
상태값을 변경을 위해 값을 추가할경우 INNITAL_STATE에 초기값을 추가한다음, 바로 setValue 를 사용하면 된다.   
#### 리듀서와 액션과 관련된 코드 분리   

state.js
```javascript
  
  export const addFriend = friend => ( { type :ADD, friend } ) ;
  export const setValue = createValueAction(SET_VALUE) ;

// 변경후 
  export const actions = {
      addFriend : friend => ( { type :ADD, friend } ) ,
      setValue : createValueAction(SET_VALUE) ;      
  }
```
FriendMain.js
```javascript
  import { actions } from '../state';

  export default function FriendMain(){

    function onAdd(){      
      dispatch(actions.setValue('name', 'mike'));
      ..
      dispatch(actions.addFriend(friend));
    }
  }

```
### redux-saga 를 이용한 비동기 액션 처리1
리덕스에서 액션이 발생한 이후에 비동기 처리를 통해서 상태값을 변경하고 싶을때  API를 통해서 서버에서 데이타를 가져오는 경우 3가지 라이브러리 사용함.
- redux-thunk
  - 비동기 로직이 간단할때 사용 
  - 가장 간단하게 시작할 수 있다.
- redux-observable
  - 비동기 코드가 많을때 사용
  - RxJS패키지를 기반으로 만들어졌음
    - 리액티브 프로그래밍을 공부해야 되므로 진입장벽이 가장 높다.
- redux-saga
  - 비동기 코드가 많을 때 사용
  - 제너레이터(자바스크립트에 내장된 기능이기때문에 공부해두면 활용도가 높음) 를 적극적으로 활용한다.
  - 테스트 코드 작성이 쉽다.
  - 현재 현업에서 사용하고 있음.

#### redux-saga 사용하기   
  timeline/state/index.js
  ```javascript
    const ADD = 'timeline/ADD';
    const EDIT = 'timeline/REMOVE';

    export const addTimeline = timeline => ( {type:ADD, timeline})
    export const removeTimeline = timeline => ( {type:REMOVE, timeline})
  
  // saga 애서 가져다 쓰려고  액션 타입과 액션 크리에이트를 객체로 묶어줌
    export const types = {
        ADD : 'timeline/ADD',
        EDIT : 'timeline/REMOVE',
        REQUEST_LIKE : 'timeline/INCREASE_NEXT_PAGE',
        ADD_LIKE : 'timeline/ADD_LIKE',
        SET_LOADING : 'timeline/SET_LOADING',=
    };

    export const actions = {      
      addTimeline : timeline => ( {type:types.ADD, timeline}), 
      removeTimeline : timeline => ( {type:types.REMOVE, timeline}),
      requestLike : timeline => ( {type:types.REQUEST_LIKE, timeline}),
      addLike : ( tiemlineId, value ) => ( {type:types.ADD_LIKE, timeline}),
      setLoading : isLoading => ( {
        type : types.SET_LOADING, 
        isLoading,
      }),
    }   

    const INITIAL_STATE = { tiemline : [], nextPage : 0, isLoading : false }

    const reducer = createRecuder ( INITIAL_STATE , {
      ...
      [types.ADD_LIKE] : ( state, action) => {
        const timeline = state.timelines.find(
          item => item.id === action.timelineId,
        );
        if (timeline){
          timeline.likes += action.value;
        }
      }, 
      [types.SET_LOADING] : (state, action) => (state.isLoading = action.isLoading),
    } )
  ```
  timeline/component/timelineList.js
  ```javascript

  export default function TimelineList ({ timelines, onLike }){
    return (
      <ul>
        {timelines.map(tiemline => (
          <li key = {timeline.id}>
            {timeline.desc}
            <button
              data-id={timeline.id}
              onClick = {onLike}
            >{ `좋아요(${timeline.likes})`}</button>
          </li>
        ))}
      </ul>
    )
  }
  ```
  timeline/state/TimelineMain.js
  ```javascript
  export default function TimelineMain(){
    const dispatch = useDispatch();
    const timelines = useSelector (state => state.timeline.timelines);
    const isLoading = useSelector (state => state.timeline.isLoading);
    ...
    function onLike(e){
      const id = Number(e.target.dataset.id);
      const timeline = timelines.find( item =? item.id === id);
      dispatch (actions.requestLike(timeline));
    }
    return (
      <div>
        <button onClick = {onAdd}>타임라인 추가</button>
        <TimelineList tiemlines = {timelines} onLike = {onLike} />
        { isLoading && <p>전송중...</p>}
      </div>
    )
  }

  ```
  timeline/state/saga.js
  ```javascript
  import { put, call, all } from 'redux-saga/effects';
  import { actions, types } from './index';
  import { callApiLike } from '../../common/api';

  export function fetchData(){
      yield put (actions.setLoading(true));
      yield put (actions.addLike(action.timeline.id, 1));
      yield call (callApiLike);
      yield put (actions.setLoading(false));
  }
   // put, call all 는 모두 saga 에서 부수효과 라고 부름
   // saga 에서가져온다.
   // api 를 흉내내는 callApiLike를 하나 만들었음
   // put 리덕스 액션 발생, likecount 를 1을 증가
   // call 은 뒤에있는 함수 실행 
  

  export default function (){  // 제너레이터로 작성되었음.
    yield all ( [ takeLeading(types.REQUEST_LIKE, fetchData) ] ); 
    // yield도 제너레이터의 키워드, 
    // all과 takeLeading 은 saga 에서 제공하는 함수임
    // 배열안에서 원하는것들 여러개 나열할것임
    // takeLeading 첫번째 매개변수로 액션을 추가하면 이 액션이 추가했을때 두번째 있는 함수를 실행할것이다.
    // takeLeading 은 아직 처리되고있는 액션이  있을때 그 사이에 들어오는 액셔는 무시된다. 처음에 들어오는 액션에 우선순위를 높게 줘서 처리한다.( 그래서 전송중이라는 글자가 있을때 계속 눌러도 숫자가 올라가지 않는다.) takeLatest 는 뒤에 들어온것에 우선순위를 더 높게 준다. 
   
  }
  ```
redux-saga의 부수효과 함수는 해야할 일을 설명하는 자바스크립트 객체를 반환한다. 
이렇게 반환된 객체는 yield를 호출했을때 사가미들웨어에게 전달이 된다. 
리덕스의 미들웨어 쪽에서 사가 미들웨가 돌아가고 있는데 
사가미들웨너는 부수효과 객체가 설명하는 일을 한 다음에 그 결과와 함께 실행흐름을 다시 
이쪽 제너레이터쪽으로 넘겨준다. 
그래서 사마미들웨어 쪽으로 put 에 대한 정보를 넘기고 사가 미들웨악 처리를 한 다음에 다시 이쪽에 다음줄이 실행이 된다.
사가 미들웨오로 다시 객체가 넘어가고.. 그 과정을 반복한다.

 common/api.js
  ```javascript
  export function callApiLike(){
    return new Promis( (resolve, reject) => {
      setTimeout (resolve, 1000);
    } )
  }
  ```
   common/store.js
  ```javascript
  
  const sagaMiddleware = createSagaMiddleware();
  const store = createStore(
      reducer, 
      composeEnhancers(applyMiddleware(sagaMiddleward)),
  );
  ..
  ```
  >> 사가 미들웨어를 넣어주면 된다.
### 제너레이터 이해하기
redux-saga의 기본  이다.    
 f1은 제너레이터 함수이다.    
  *를 입력하면 제너레이터 함수로 만들수 있음      
  yield 라는 키워드가 있음.   
  제너레이터 객체가 반환이 된다.  
 ```javascript
  function* f1(){
    console.log('f-1');
    yield 10;
    console.log('f-2');
    yield 20;
    console.log('f-3');
    yield 'finished';
  }
  const gen = f1();

  console.log( gen.next());
  console.log( gen.next());
  console.log( gen.next());
  ```
콘솔결과   
f1-1   
{value : 10, done:false}   
f1-2   
{value : 20, done:false}   
f1-3   
{value : 'finished', done:true}   
// 그 객체로 next라는 메써드가 있는데, 한번 호출하면 멈췄던 실행을 계속하면서 다음으로 다음으로 넘어간다.   
 
#### iterator (반복자)
- 다음 조건을 만족하는 객체는 iterator (반복자) 이다.   
- next 메서드를 가지고 있다.
- next 메서드는 value와 done 속성값을 가진 객체를 반환한다.
- done 속성값은 작업이 끝났을때 참이 된다.   
- gen 객체가  iterator 인것이다.
#### iterable (반복기능)
- 다음조건을 만족하면 반복기능(iterable)한 객체이다.
- Symbol.iterator 속성값으로 함수를 갖고 있다.
- 해당 함수를 호출하면 반복자(iterator) 를 반환한다.
 ```javascript
 console.log(gen[Symbol.iterator]() ===gen) ; // true
 ```
 위의 함수를 실행하면 자기 자산이나온다. 즉, 이 함수 (gen[Symbol.iterator]()) 를 호출하면 자기자신이 니온다.( iterator가 나오는것이다.)
 ge(제너레이터)는 이터레이터이면서 이터레이블이다.  

- 배열도 iterable 이다.
 ```javascript
 const arr = [1,2,3];
 const iter = arr[Symble.iterator()];
 console.log(iter.next); // {value : 1, done:false}
 ```
 
 - iterable 을 만족하면 javascript의 몇가지 기능을 사용할수 있다.
 ```javascript
  function* f1() {
    yield 10;
    yield 20;
    yield 30;
  }
  for (const v of f1()) {
    console.log(v);
  } // 10 20 30
  const arr = [...f1()];
  console.log(arr) // [10, 20, 30]
 ```
 for of 라는것을 사용 할수 있다. 

 - 제너레이터는 표현력이 좋기 때문에 자연수의 집합을 표현할수 있다.
 ```javascript
 function* naturaNumbers(){
   let v = 1;
   while (true){
     yield v++;
   }
 }
 const gen = naturaNumbers()
 gen.next // {value : 1, done : false}
 gen.next // {value : 2, done : false}
 gen.next // {value : 3, done : false}
 ```
 일반함수 였다면 저 안에서 무한반복이 일어나서 먹통이 되었을것이다. 하지만 제너레이터는 값을 하나씩 던져준다.   
 제너레이터는 실행을 멈출수 있다 (yield++). 멈추고 실행권한을 외부로다시 주는것이다. 
 외부에서신호가 왔을때 다시 실행을 제가할 수 있다. 실행을 멈추고 재개할수 있다는 특성때문에 제너레이터는 협업이 가능하다.   

```javascript

   function* minsu(){
    const myMsgList  = [
      '안녕, 나는 민수야',
      '만나서, 반가워',
      '내일 영화 볼래?',
      '시간 안되나?',
      '내일 모레 시간은 어때?',
    ];
    for (const msg of myMsgList) {
      console.log( ' 수지 : ' , yield msg);
    }
  }

  function suji(){
    const myMsgList = [  '', '안녕 나는 수지','그래 반가워','...',];
    const gen = minsu();
    for (const msg of myMsgList) {
    console.log(' 민수 : ' ,  gen.next(msg).value)
    }    
  }

  suji();
```

### redux-saga 를 이요한 비동기 액션 처리2
하나의 사가함수가 두개의 액션을 처리할수도 있다. 

```javascript
  function loginFlow(){
    while(){
      const { id, password } = yield take(type.LOGIN);
      const userInfo = yield call( callApiLogin , id, password );
      yield put(types.SET_USER_INFO, unserInfo);
      yield take(types.LOGOUT);
      yield call(callApiLogout, id);
      yield call(types.SET_USER_INFO, null);
    }
  }
```
하나의 사가 함수가 로그인액션과 로그아웃액션을 처리하고 있다. 
take 는 매개변수로 입력한 액션을 기다림. 액션객체가 반환이 되어서 필요한 처리를 한다.
두개의 액션을 조합해서 하나의 사가함수를 만들 수 있다.

#### 사가함수에서 예외처리 하는법
```javascript
  export function callApiLike(){
    return new Promis( (resolve, reject) => {
      setTime(resolve, 1000)
    } )
  }
```
#### debounce
짧은시간내에 같은 이벤트가 반복해서 발생할때 모든 이벤틀르 처리하기 부담스러울수 있는데, 이때 디바운스를 사용한다. 
첫번째 또는 마지막 호출만 실행하는 기능이다. 
TRY_SET_TEXT 액션이 발생했을때 바로 함수를 호출하는게  아니고 05초 기다렸다가 더이상 액션이 발생하지 않으면 실행하는것이다.

```javascript
export function treySetText(action){
  yield put(action.setValue('text', action.text))
}

export default function(){
  yield all([
    takeLeading ( types.REQUEST_LIKE, fetchData),
    debounce ( 500, types.TRY_SET_TEXT, trySetText),
  ]);
}
```

테스트 코드를 쉽게 작성할수 있다는게 redux-saga의 장점이다. 
gen.next().value 로 데이타가 잘 나오고 있는지 확인해볼수 있다.    
saga.js
```javascript
export default function(){
  yield put(actions.setLoading(true));
  try{
    yield call(callApiLike);
  }catch(error){
    yield put (actions.setValue('error', error))
  }
}
```
saga.test.js
```javascript
import { cloneableGenerator } from 'redux-safa/test-utils';

describe ( 'fetchData', () => {
  const gen =  cloneableGenerator(fetchData)(action);
  expect(gen.next().value),toEqual(put(actions.setLoading(true))); // 직전까지 값을가져와서 검사한다.
  if( 'on fail callApiLike', () =>{
    const gen2 = gen.clone();
    ...
  })
})
```
사가에서는 직접 api를 호출하는게 아니라 api를 호출해야 한다고 설명하는 객체가 나오는지만 검사하면 되기 때문에 테스트 코드를 작성하기가 상당히 편하고 직관적이다.    
callApiLike 까지 검사가능. 그 이후 실행되는 코드는 예외발생 여부에 따라 달라진다. 
그래서 각각 클론해서 사용하는것이다. 
cloneableGenerator 함수 덕분에 clone 이라는 메서드를 사용할 수있다.   

