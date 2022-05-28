중요하지만 헷갈리는 리액트 개념 이해하기 2
===
### 콘텍스트 API로 데이터 전달하기   

>상위컴포넌트에서 하위컴포넌트로 속성값을 전달한다. 많은수의 하위 컴포넌트로 내려줄때는 반복적인 작업이 불가피하다.   
>자신은 사용하지 않으면서 내려주기위에 그 컴포넌트가 직접 사용하지 않는데도 기계적으로 적어줘야 할때가 있다. 
>이럴때는 ***context*** 를 사용하면 좀더 간편하게 코드를 만들수 있다.

createContext

1-no-context.js.  
```javascript
export default function App(){
  return(
    <div>
      <Profile usename= "mike"/>
    </div>
  )
}
function Profile ({ username }) {// profile 은 직접 username 을 사용하지 않으면서 기계적으로 내려주고 있음.
  return(
    <div>
      <Greegint username = {username} />
    </div>
  )
}
function Greegint( {username}) {
  return <p>{`${ username }님 안녕하세요`}</p>;
}
```
2-context.js.  
```javascript
const UserContext = createContext ('unknown'); // createContext 함수사용, provider 의 value 값이 없다면 기본적으로 'unknown' 값이 사용된다.

export default function App(){
  return(
    <div>
      <UserContext.Provider value = "mkik"> // provider 에서 value 값을 넣어주면 consumer 에서 그 값을 받아서 처리할 수가 있다
        <Profile />
      </UserContext.Provider>
    </div>
  )
}
function Profile () {
  return(
    <div>
      <Greeting />
    </div>
  )
}
function Greeting( {username}) {
  return (
    <UserContext.Consumer>  
      {username => <p>{`${ username }님 안녕하세요`}</p>} // renderprops로 작성한것.  
      children 을 함수 형태로 작성. 필요한 값을 찾기위해 부모로 올라가면서 가장 가까운 provider를 찾는다. 근데 profile이 왜 부모지??
    </UserContext.Consumer>
  )
}
```
provider 에 value 값이 변경이 된다면, consumer 는 리로딩된다. 
consumer는 가장가까운 provider 를 찾는다 없어도 createContext의 기본값 사용.

```javascript
const UserContext = createContext ('unknown'); 

export default function App(){
  const [name, setName] = useState('mike');
  return(
    <div>
      <UserContext.Provider value = {value}> 
        <Profile />
        <input 
          type="text"
          value = {name}
          onChange = {e => setName(e.target.value) }
         />
      </UserContext.Provider>
    </div>
  )
}
//function Profile () { 
const Profile = React.memo(function(){ // profile 이 렌더링이 되지 않아도 될지 확인해보지만 안됨.
  consoel.log('profile')
  return(
    <div>
      <Greeting />
    </div>
  )
});

function Greeting( {username}) {
  return (
    <UserContext.Consumer>  
      {username => <p>{`${ username }님 안녕하세요`}</p>} // children 을 함수 형태로 작성. 필요한 값을 찾기위해 부모로 올라가면서 가장 가까운 provider를 찾는다. 근데 profile이 왜 부모지??
    </UserContext.Consumer>
  )
}
```
profile 컴포넌트가 렌더링 되지 않아도 consumer 콤포넌트는 업데이트 잘된다.

```javascript
function Greeting( {username}) {
  const username = useContext(userContext);
  return <p>{`${ username }님 안녕하세요`}</p>;
}
```
위와 같이 useContext hook 을 이용하면 Consumer 컴포넌트를 사용하지 않고 간편하게 사용할수 있음.

<hr/>

```javascript
const UserContext = createContext('unknow');
const ThemeContext = createContext('dark');
...
<ThemeContext.Provider value='light'>
  <UserContext.Provider value='mkik'>
  ..
  </UserContext.Provider>
</ThemeContext.Provider>

...

function Greeting(){
  const username = useContext(UserContext);
  const theme = useContext(ThemeContext);
  return(
    <p
      style={{color:theme === 'dark' ? 'gray' : 'green'}}
      >{`${username}님 안녕하세요`}</p>
  )
  
}
```

Context 를 여러개 사용가능. (데이터 종류별로 나눈다)
여러개 사용하는것이 성능상 더 좋을수 있다. 값이 변경여부를 따로 관리하여 렌더링에 대한 이슈를 줄일수 았다.

* * *


#### 자식요소에서 변경하고 싶을때
 

SetUserContext 하위콤포넌트에서 수정하고 싶을때 데이타 수정하고 싶은 함수를 별도의 함수를 설정한다..

```javascript
const UserContext = createContext({ username : 'unknown , helloCount : 0});
const SetUserContext = createContext( ()=>{} );
...

<SetUserContext.Provider value={setUser}> // 상태값 변경함수
  <UserContext.Provider value={user}>
    <Profile/>
  </UserContext.Provider>
</SetUserContest.Provider>

```
데이터를 수정하는 함수를 별도의 함수로 분리한다.  (SetUserContext)
데이터를 내려주는 Provider 컴포넌트가 있고 (UserContext.Provider)
상태값 변경함수를 별도의 컴포넌트로 내려준다. (SetUserContext.Provider )


child.js.  
```javascript
const setUser = userContext (SetUserContext);
const {username, helloCount } = userContext( UserContext );
return (
  <React.Fragment>
    <p>{`${username}님 안녕하세요`}</p>
    <p>{`인사 횟수 : ${helloCount}`}</p>
    <button onClick = { () => setUser({ username, helloCount : helloCount +1 }) }> 
    인사하기
    </button>
  </React.Fragment>
)
```
child 에서는 상태값 변경함수를 가져다가 (const setUser = userContext (SetUserContext);
 안에서 필요할때 호출할수 있다 (setUser) 08:32   
 현재 데이타가 객체이기 때문에 여기서는 hellCount 값만 변경을 하는데 다른속성값들을 나열해서 입력해주고 있다.   
 이렇게 여려 데이타를 사용할때는  useState 보다는 useReduce가 낫다.


* * *
#### context 주의사항.

불필요한 렌더링을 줄이는게 좋다. 
부모로 있는 경우도 같이 렌더링이 되기 때문에 분리해주는게 좋다.  

```javascript
...
export default function App(){
  const [username, setUsername] = userState('');
  const [age, setAge] = userState(0);
  return(
    <UserContext.Provider value={{ username, age }}> 
      <Profile/>
    </UserContext.Provider>
  )
  
}
```
username 만 입력을 하게되면 APP컴포넌트 전체가 렌더링이 된다. 그럼 매번 새로운 객체가 만들어 지게 된다.
따라서 값이 변경되지 않아도 consumer는 불필요하게 렌더링이 될수 있다.

```javascript
const [username, setUsername] = userState('mika');
const [age, setAge] = userState(0);
const [count, setCount ] = useState(0);

  <UserContext.Provider value={{ username, age }}> 
    <Profile/>
    <button onClick = { ()=> setCount(count + 1) }>증가</button> 
  </UserContext.Provider>
  
...
  function Greeting(){
    const { username } = useContext(UserContext);
    return <p>{`${username}님 안녕하세요`}</p>
  }
  
  <!-- 개선사항 -->
  
  const [user, setUsername] = userState( { username : 'mika', age : 23 } ); //값을 객체로 관리
  const [age, setAge] = userState(0);
  const [count, setCount ] = useState(0);
  
  <UserContext.Provider value={user}}> 
  
  
```
count 값이 변경되면서 렌더링이 되는데  Greeting 도 같이 렌더링이 된다. 
usUserContext 에 있는 값을 빼서  
//<UserContext.Provider value={{ username, age }}> 
>> 
// <UserContext.Provider value={user}}> 
하나의 객채로 관리한다.    
// const [username, setUsername] = userState('mika');
>>
// const [user, setUsername] = userState( { username : 'mika', age : 23 } );    

**주의해야 할것 두번째**   
provider 컴포넌트를 사용하는 쪽에서는 콤포넌트 안에 넣어야 되는데 보통 루트에서 jsx 전체를 감싸는 방식으로 작성한다. 그랬을때는 Consumer 콤포넌트가 Provider 콤포넌트를 못 찾을 일은 거의 없다.   
하지만 루트쪽이 아니라  중간중간에 Provider 콤포넌틀르 렌더링할때 위치에 주의를 해야 한다.

### ref 속성값으로 자식요소에 접근하기
Dom 요소에 직접 접근할때 ref 속성값을 이용해서 접근한다.    
Componet 나 Dom 요소 접근 가능   

Focus 가 자동으로 가있음
```javascript
...
  const inputRef = useRef(); //useRef 훅을 사용하면 된다.
  useEffect( ()=> {
    inputRef.current.focus(); // current속성은 실제 돔요소를 가르킴. 돔요소의 focus 함수를 쓸수 있다.
  }, [] )
...
  <input type="text" ref={inputRef} /> //반환된값을 속성으로 넣어준다.
  <Box ref={inputRef} /> // 컴포넌트가 클래스형 컴포넌트라면 해당컴포넌트의 인스턴스를 가르킨다. 따라서 current 속성은 해당클래스의 메서드를 호출할수 있다.

```
current 속성은 실제 돔 요소를 가르치고 있으며 해당 돔에 focus 를 사용할수 있다.
ref 사용은 useEffect 안에서 쓰고 있다.    
실제 돔 요소는 렌더링결과 실제 돔에 반영된 후에 접근할수 있기 때문에 useeffect 안에서 쓸수 있는것이다.   

함수형 컴포넌트는 인스턴스로 만들어지지 않지만, 
useImperativeHandle 이라는 훅을 사용하면 함수형 콤포넌트에서도 마치 클래스형 컴포넌트의 멤버 변수가 메서드에 접근하는것처럼 
함수형 컴포넌트 내부의 변수와 함수를 외부로 노출시킬수 있다.

component.js.  
```javascript
  const inputRef = useRef();
  useEffect( () =>{
    inputRef.current.focus();
  }, []);
  
  return (
    <div>
      <InputAndSave inputRef={inputRef} />
      <button onClick = { ()=> inputRef.current.focus() }>텍스트로 이동</button>
    </div>
  )
  
  function InputAndSave({ inputRef }, buttonRef){
    return(
      <div>
        <input type="text" ref={inputRef} />
        <button ref={buttonRef}>저장</button>
      </div>
    )
  }  
  
```
다른처리를 하지 않고,함수에 ref를 입력할수 없음.  // <InputAndSave ref = {inputRef}>
그래서 inputRef 라는 속성이름으로 받아서 InputAndSave 내부에 있는 자식 input에게 입력을 하고 있음. 
Button 이라는 굉장히 직관적이고 기초적인 컴포넌트를 만들어서 사용하는경우에는 ref로 입력하는게 좋음.  //button ref={buttonRef}>   
하지만 ref 라고 하면 리액트가 내부적으로 처리하게 때문에 이 button 컴포넌트 내부에서 그 값을 사용할수 없음.    
```javascript
<Button ref={buttonRef} />
```
이런경우를 위해서 forwardRef 라는 함수를 사용할 수 있음. 두번째 매개변수로 ref속성값을 받아서 사용할수 있음.
```javascript
  const Button = React.forwardRef (function ({ onClick }, ref) {
    return (
      <div>
        <button onClick={onClick} ref={ref}>저장</button>
      </div>
    )
  }
```

fuction.js.  
```javascript
  exprot default function App(){
    const [text, setText] = useState (INITIAL_TEXT);
    cosnt [showText, setShowText] = useState (true);
    return (
      <div>
        {showText && (
          <input
            type="text"
            ref = {ref => ref && setText(INITIAL_TEXT)} // ref 속성값에 함수 입력
            value = {text}
            onChange= { e=> setText(e.target.value) }
            />
        )}
        <button onClick={ ()=> setShowText(!showText) }>보이기/가리기</button>
      </div>
    )
  }
  
  const INITIAL_TEXT = '안녕하세요'
```
ref 속성값에 함수를 직접 입력하는것. 이 함수는 해당하는 요소가 생성(해당하는 요소의 ref 값 )되고, 사라질때(null값이 넘어옴) 한번씩 호출됩니다.
생성될때 INITIAL_TEXT를 초기 상태값으로 입력하는 코드이다.
showText 라는 상태값이 있는데, 버튼을 누를때 그값을 토글시킴. 그값에 따라서 input 요소를 보였다가 사라졌다가 하게하는 코드이다.   
const INITIAL_TEXT = '안녕하세요'때문에 입력이 안되고, 계속 저 값을 유지하고 있음.
컴포넌트가 렌더링될때마다 새로운 함수를, **ref => ref && setText(INITIAL_TEXT)** 이 새로운 ref 함수를 입력하고 있기 때문이다.   
리액트는 ref속성값으로 새로운 함수가 들어오면 이전 함수에 null인수를 넣어서 호출한다.   
그리고 새로운 함수에는 요소의 참조값을 넣어서 다시 호출한다.   
따라서 문자를 입력할때 계속 컴포넌트가 렌더링이 될텐데 그때마다 새로운 함수가 입력되면서 INITIAL_TEXT가 계속해서 입력되기 때문이다.
이를 해결하려면 ref => ref && setText(INITIAL_TEXT) 를 고정할 필요가 있다.   
이럴때는 useCallback 이라는 훅을 사용할 수가 있다.   
```javascript
   const setInitialText = useCallback(ref => ref && setText(INITIAL_TEXT), []);
   
   ...
          <input
            type="text"
            ref = {setInitialText}
            value = {text}
            onChange= { e=> setText(e.target.value) }
            />
  
```
seCallback 훅의 메모제이션 기능덕분에 한번 생성된 이 함수를 계속해서 재사용한다는것만 알아두면 된다. 그래서 이 함수는 변하지 않는 값이 된다.
setText(e.target.value)  값이 변경이 될때 setText(INITIAL_TEXT) 가 실행이 되지 않는다.

**접근하고자 하는 요소가 많을때**

function-multi.js.  
```javascript
export default function App(){
  return (
    <div>
      <div
           style = {{
              display:'flex',
              flexWrap : 'wrap',
              width : '100vw',
              height : '100%'
           }}
       >
          {BOX_LIST.map(item => {
            <div 
                key = {item.id}
                style = {{
                flex : '0 0 auto',
                width : item.width,
                height : 100,
                backgroundColor : 'yellow',
                border : 'sold 1px red',                 
               }}
                 />{ `box_${item.id}`}</div>          
          })}  
      </div>
      <button onClick = { ()=> {} }>오른쪽 끝 요소는? </button>
    </div>    
  )
}
const BOX_LIST = {
 { id:1 , width: 70 },
 { id:2 , width: 100 },
 { id:3 , width: 80 },
 { id:4 , width: 100 },
 { id:5 , width: 90 },
 { id:6 , width: 60 },
 { id:7 , width: 120 },
}
  
```
>const ref1 = useRef(); // 각 값마다 다 넣어야 하는가? 갯수가 작을수도 많을수도 있다. 훅 규칙에서 말했듯이 이 훅을 사용한 순서 개수는 항상 같아야 함. 그래서  요소마다 useRef를 사용하는게 애매하다. 
>함수를  입력하는 방법을 사용 할수 있다. 
```javascript
...
const boxListRef = useRef({}); //박스 저장하는곳  , Object 저장

ref={ref => (boxListRef.current[item.id] = ref)}
```

```javascript
export default function App(){
  
  const boxListRef = useRef({}); // 실제 돔을 넣었지만, 원하는값이면 어떤값이든 저장가능.
   
  function onClick(){
    let masRight = 0;
    let maxId = '';
    for (const box of Box_LIST){
      const ref = boxListRef.current[box.id];
      if(ref){
        const rect = ref.getBoundingCliendFect();
        if (maxRight < rect.right){
          maxRight = rect.right;
          maxId = box.id;
        }
      }
    }
    alert (`오른쪽 끝 요소는 ${max.id} 입니다.`)
  }  
  return (
    <div>
      <div
           style = {{
              display:'flex',
              flexWrap : 'wrap',
              width : '100vw',
              height : '100%'
           }}
       >
          {BOX_LIST.map(item => {
            <div 
              key = {item.id}
              ref = { ref => (boxListRef.currend[item.id] = ref ) } // 저장했음
               style = {{
                flex : '0 0 auto',
                width : item.width,
                height : 100,
                backgroundColor : 'yellow',
                border : 'sold 1px red',                 
               }}
                 />{ `box_${item.id}`}</div>          
          })}  
      </div>
      <button onClick = { onClick }>오른쪽 끝 요소는? </button>
    </div>    
  )
}

const BOX_LIST = {
 { id:1 , width: 70 },
 { id:2 , width: 100 },
 { id:3 , width: 80 },
 { id:4 , width: 100 },
 { id:5 , width: 90 },
 { id:6 , width: 60 },
 { id:7 , width: 120 },
}
  
```
#### ref 속성값 사용시 주의할점
- 컴포넌트가 생성된 이후라도 ref 객체에  currnt 속성이 없을수 있음. 조건부 렌더링인 경우 ref를 검사하는게 필요하다.   
>howText && ,  inputRef.current &&
```javascript
{ showText && <input type="text" ref={inputRef} />}
...
<button onClick ={ () => inputRef.current && inputRef.current.focus() }>
텍스트로 이동
</button>
```
### 리액트 내장 훅 살펴보기
>**useMemo** 와 **useCallback**은 메모이제이션 기능을 제공함. 이전값 재활용가능
>**useReducer**은  여러개의 상태값을 하나의 훅으로 관리할때 편리하게 사용할 수 있다.

#### useRef
```javascript
import React, {useState, useRef, useEffect} from 'react';

  export default function App(){
    const timerIdRef =  useRef(-1);
    useEffect( () => {
        timerIdRef.current = setTimeout( () => {}, 1000 );
    } )
    // ...
    useEffect( () => {
        if(timeIdRef.current >= 0){
          clearTimeout(timeIdRef.currnet);
        }
    } )
  }
```
여기에 timerIdRef 라는게 있는데요 초기갑 -1을 주었음. ref 객체는 꼭 돔요소를 참조할때만 하는게 아니고, setTimeout이 반환하는 id(timerIdRef.current) 값은 반환해서 이것을 다른곳에서 클리어(clearTimeout) 할때 사용한다. 로직을 처리하는 데이터 인데, 렌더링과 상관없는 값을 저장할때 useEffect를 사용하면 된다. 하지만 적합하지는 않다. useState 는 timerId 가 변경되었을때 다시 렌더링이 된다. 하지만 UI데이타가 아니기 때문에 렌더일 결과는 같다. 그래서 불필요한 렌더링을 한다. 그래서 ref 객체가 더 적합한 상황이다.   

*이전상태값을 기억하고 싶을때   ref 객채를 활용하였다.
```javascript
export default function App(){
  const [age, setAge] = useState(20);
  const preveAgeRef = useRef(20);
  useEffect( () =>{
    prevAgeRef.current = age;
  }, [age] );
  const prevAge = prevAgeRef.current;
  const text = age === prevAge ? 'same' : age > prevAge ? 'older' : 'younger';
  return (
    <div>
      <p>{`age ${age} is ${text} than age ${prevAge}`}</p>
      <button
        onClick = { () => {
          const age = Math. floor(Math.random() * 50 +1);
          setAge(age);
        }}
      >
        나이변경
      </button>
    </div>
  )
}
```
#### useMemo
계산량이 많은 함수의 반환값을 재활용하는 용도로 사용함
```javascript
  const [v1, setV1] = useState(0);
  const [v2, setV2] = useState(0);
  const [v3, setV3] = useState(0);
  const value = useMemo( () => runExpaensiveJob(v1,v2),[v1,v2] ));
  return(
    <>
      <p>{`value is ${value}`}</p>
      <button
        onClick={() => {
          setV1(Math.random());
          setV2(Math.random());
        }}
      >v1/v2수정
      </button>
      <p>{`v3 is ${v3}`}</p>
      <button onClick = {() => setV3(Math.rendom()) }>v3 수정</button>
    </>
  )
  function runExpaensiveJob(v1, v2) {
    console.log( 'runExpaensiveJob is called' );
    return v1+v2;
  }

```
두번째 버튼을 클릭해도 'runExpaensiveJob is called '가 찍히지 않는다.   

#### useCallback
useMemo와 비슷한데, 메모제이션 기능을 가지고 있는데 함수 메모이제이션에 특화된 함수로 생각하면 된다.
memo를 사용하는데도 출력이 된다.
```javascript
  <UserEdit
    onSave = { () => saveToServer(name, age) }
    setName = { setName }
    setAge = { setAge }
  />

const UserEdit = React.memo (funciton ({onSave, setName, setAge }) {
  console.log('userEdit render);
  return null
})

function saveToServer(name, age){}
```
```javascript
const onSave = useCallback( () => saveToServer (name, age), [name, age] );
<UserEdit 
  onSave = { onSave }
  setName = { setName }
  setAge = { setAge }
/>
```
#### userReducer
여러개의 상태값을 관리할때 사용.   
useState 랑 비슷함.  반환되는 값은 상태값(state)이 반환되고, useState 처럼 상태값을 변경할 수 있는 dispatch 함수가 반환된다.   
매개변수로 reducer 라는 함수를 입력하고, 초기값(INITIAL_STATE)을 입력한다.    
하단의 reducer 함수는 현재 상태값과, action 이 입력이 되고, action을 보고 상태값을 어떻게 변경할지 판단한다.   
reducer 는 현재 상태와 액션 객체를 파라미터로 받아와서 새로운 상태를 반환해주는 함수이다. 

```javascript
export default function App(){
 const [state, dispatch] = useReducer(
    reducer,
    INITIAL_STATE
  );
 return(
  <div>
    <p>{ `name is ${state.name}` }</p>
    <p>{ `age is ${state.age}` }</p>
    <input
      type ="text"
      value = { state.name }
      onChange = { e=>
        dispatch ( {type : 'setName', name : e.currentTarget.value})
      }
    >
  </div>
  )
}

  function reducer(state, action) {
  switch (action.type) {
    case 'setName':
      return {...state, name:action.name};
    case 'setAge':
      if( action.age > MAX_AGE ){
        return {...state, age:MAX_AGE}};
      }else{
        return {...state, age:action.age}};
      }
    default:
      return state;
  }
}
```
>**Reducer** 함수는 상태값을 변경하는 로직분리가 가능함.   
>리액트는 상위 콤포넌트에서 다수의 상태값을 관리한다. 이때 자식 콤포넌트로 부터 발생한 이벤트에서  상위컴포넌트의 상태값을 변경해야 하는 경우가 많이 있다.   
> 이를 위해서 상위 컴포넌트에서 트리 깊은곳까지 이벤트 처리 함수를 전달하기도 한다. 그 작업은 손이 많이 가고 가독성도 떨어지기 때문에  **useReducer** 훅이랑 **Context api** 를 같이 이용하면  상위 컴포넌트에서 트리 깊은곳으로 이벤트 처리 함수를 쉽게 전달 할 수 있다.

```javascript
export const ProfileDispatch = React.createContext(null);

export default function App(){
 const [state, dispatch] = useReducer(
    reducer,
    INITIAL_STATE
  );
 return(
  <div>
    <p>{ `name is ${state.name}` }</p>
    <p>{ `age is ${state.age}` }</p>
    <ProfileDispatch.Provider value = {dispatch}>
      <SomeComponent/>
    </ProfileDispatch.Provider>
  </div>
  )
}
```
>ContextApi(createContext) 를 이용해서 ProfileDispatch 라는 context를 만들었고, 상위 컴포넌트에서 
>Provider 컴포넌트를 이용해서 useReducer의 dispatch 함수를 내려준다. 필요한곳에서 상태값 변경이 가능하다. Reduxe를 사용하지 않아도 이렇게 관리힐수 있다.

#### useImperativeHandle
클래스형 콤포넌트의 부모 컴포넌트는 ref객체를 통해서 자식컴포넌트의 메소드를 호출할 수 있다. 이 방식으로 내부 구현에 대한 의존성이 생겨서 지양해야 하지만 필요한 경우가 있음. 그때 **useImperativeHandle** 를 이용하면 마치 함수형 컴포넌트에도 멤버변수나 멤버함수가 있는것처럼 만들수 있음.

```javascript

  function profile(_, ref){
    const [name, setName] = useState('miki');
    const [age, setName] = useState(0);

    useImperativeHandle(ref, () => ({
      addAge : value => setAge(age + value), 
      addNameLength : () => name.length,
    }));
    return (
      <div>
        <p>{`name is ${name}`}</p>
        <p>{`age is ${name}`}</p>
      </div>
    )
  } 

  export default forwardFef(profile);

```
>ref 속성값만 사용하고 있음. ref 속성값을 받아 useImperativeHandle 의 첫번째 매개변수로 입력하고 있고, 두번째 매개변수로 함수를 입력하고 있음. 두번째 매개면수 함수의 반환값이  이 부모의 ref 객채가 참조하는값이다.

>그리고 ref 참조값을 받기 위해서 forwardRef 함수를 사용을 했음

```javascript

export default function App(){
  const profiledRef = useRef();
  const onClick = () => {
    if (profileRef.current) {
      console.log('current name length'), pfofileRef.current.getNameLength());
      profileRef.current.addAge(5);
    }
  };
  return(
    <div>
      <Profile ref = {profileRef} />
      <button onClick = {onClick}>addage5</button>
    </div>
  )
}

```
>부모컴포넌트에서는 useRef 훅을 이용해서  이 ref 객체를 Pfofile 컴포넌트의 ref 속성값으로  입력하고 있다.
>이렇게 되면 profileRef.current는 이전에 반환된 값(forwardFef)을 참조하게 된다. 그래서 자식에서 제공해준 getNameLength, addAge 함수를 사용할 수 있다.

#### useLayoutEffect
useEffect 훅은 부수효과 함수는 렌더링 결과가 돔에 반영된후에 비동기로 호출된다. **useLayoutEffect** 훅은 useEffect과 비슷하지만 부수효과 함수를 동기로 호출한다. useLayoutEffect 훅의 부수효과 함수는 렌더링 결과가 돔에 반형된 직후에 바로 호출된다. 부수효과함수가 연산이 많으면 브라우저가 먹통이 될수 있다. 그래서 useEffect를 사용하는게 성능상 이점이 있다.   
렌더링 직후에 돔 요소의 값을 읽어들이는 경우 또는 조건에 따라서 컴포넌트를 다시 렌더링 하고 싶은경우 사용한다.   
#### useDebugValue
커스톰 훅 안에 useDebugValue를 사용하면 리액트 개발자도우게서 많은 정보를 받을수 있다. 디버깅할때 편리한 도구이다. 사용하지 않고 작성했을때 앱의 상태를 상태값으로 관리를 하고 있고 세가지 상태를 숫자로 관리하고 있다. 그리고 next 함수를 호출하면 상태값을 하나씩 증가시면서 stop를 start로 바뀌게 하는 코드임.





























