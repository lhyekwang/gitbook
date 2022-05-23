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
      {username => <p>{`${ username }님 안녕하세요`}</p>} // children 을 함수 형태로 작성. 필요한 값을 찾기위해 부모로 올라가면서 가장 가까운 provider를 찾는다. 근데 profile이 왜 부모지??
    </UserContext.Consumer>
  )
}
```
provider 에 value 값이 변경이 된다면, consumer 는 리로딩된다. 
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
useContext (hook 을 이용해서 ?)를 이용해서 greeting 에서 이용핤 있도록 username 에 담아둔다.

<hr/>
Context 를 여러개 사용가능.  
여러개 사용하는것이 성능상 더 좋을수 있다. 값이 변경여부를 따로 관리하여 렌더링에 대한 이슈를 줄일수 았다.

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
* * *


#### 자식요소에서 변경하고 싶을때
 

SetUserContext 하위콤포넌트에서 수정하고 싶을때 데이타 수정하고 싶은 함수를 별도의 함수를 설정 

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
child.js.  
```javascript
const setUser = userContext (SetUserContext);
const {username, helloCount } = userContext( UserContext );
return (
  <React.Fragment>
    <p>{`${username}님 안녕하세요`}</p>
    <p>{`인사 횟수 : ${helloCount}`}</p>
    <button onClick = { () => setUser({ username, helloCount : helloCount +1 }) }> // 필요할때 호출가능. 여러데이타를 객체형식으로 관리할때는 useState 보다는 useReduce가 낫다.
    인사하기
    </button>
  </React.Fragment>
)
```
* * *
#### context 주의사항.
불필요한 렌더링을 줄이는게 좋다. 
부모로 있는 경우도 같이 렌더링이 되기 때문에 분리해주는게 좋다.  
count 값이 변경되면서 렌더링이 되는데  Greeting 도 같이 렌더링이 된다. (username 이 만들어지니까, 감싸고 있는 username 때문에.)

```javascript
const [username, setUsername] = userState('mika');
const [age, setAge] = userState(0);
const [count, setCount ] = useState(0);

...
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
***질문사항***
https://ko.reactjs.org/docs/context.html 부모에서 사용하는 값을 부모의 state로 끌어올리세요. 라고 되어있는데요.ㅠㅠ뭐가 다른지 모르겠어요

### ref 속성값으로 자식요소에 접근하기
Dom 요소에 직접 접근할때 ref 속성값을 이용해서 접근한다. Componet 나 Dom 요소 접근 가능   

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
다른처리를 하지 않고,함수에 ref를 입력할수 없음. 그래서 inputRef 라는 속성이름으로 받아서 InputAndSave 내부에 있는 자식 input에게 입력을 하고 있음. 
Button 이라는 굉장히 직관적이고 기초적인 컴포넌트를 만들어서 사용하는경우에는 ref로 입력하는게 좋음. 하지만 ref 라고 하면 리액트가 내부적으로 처리하게 때문에 이 button 컴포넌트 내부에서 그 값을 사용할수 없음.   
```javascript
<Button ref={buttonRef} />
```
이런경우를 위해서 forwardRef 라는 함수를 사용할 수 있음. ref를 사용할수 있음.  
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
ref 속성값에 함수를 직접 입력하는것. 이 함수는 해당하는 요소가 생성(ref 값이)되거나 사라질때(null값이 넘어옴) 한번씩 호출됩니다.
생성될때 INITIAL_TEXT를 초기 상태값으로 입력하는 코드이다.
showText 라는 상태값이 있는데, 버튼을 누를때 그값을 토글시킴. 그값에 따라서 input 요소를 보였다가 사라졌다가 하게하는 코드이다.   
const INITIAL_TEXT = '안녕하세요'때문에 입력이 안되고, 계속 저 값을 유지하고 있음.
컴포넌트가 렌더링될때마다 새로운 함수를, **ref => ref && setText(INITIAL_TEXT)** 이 새로운 ref 함수를 입력하고 있기 때문이다.   
리액트는 ref속성값으로 새로운 함수가 들어오면 이전 함수에 null인수를 넣어서 호출한다.   
그리고 새로운 함수에는 요소의 참조값을 넣어서 다시 호출한다.   
따라서 문자를 입력할때 계속 컴포넌트가 렌더링이 될텐데 그때마다 새로운 함수가 입력되면서 INITIAL_TEXT가 계속해서 입력되기 때문이다.
이를 해결하려면 함수를 고정할 필요가 있다.   
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
>useCallback 훅의 메모제이션 기능덕분에 한번 생성된 이 함수를 계속해서 재사용한다는것만 알아두면 된다. 그래서 이 함수는 변하지 않는 값이 된다.
>e=> setText(e.target.value)  값이 변경이 될때 setText(INITIAL_TEXT) 가 실행이 되지 않는다.

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
>const ref1 = useRef(); // 각 값마다 다 넣어야 하는가? 그래서 ref도 로직으로 처리한다.

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
### ref 속성값 사용시 주의할점
- 컴포넌트가 생성된 이후라도 currnt 속성이 없을수 있음. 조건부 렌더링인 경우 ref를 검사하는게 필요하다.




