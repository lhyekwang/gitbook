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
<hr/>
#### 자식요소에서 변경하고 싶을때
하위콤포넌트에서 수정하고 싶을때 데이타 수정하고 싶은 함수를 별도의 함수를 설정.(SetUserContext) 
```
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
```
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
<hr/>
##### context 주의사항.  
불필요한 렌더링을 줄이는게 좋다. 
부모로 있는 경우도 같이 렌더링이 되기 때문에 분리해주는게 좋다.  
count 값이 변경되면서 렌더링이 되는데  Greeting 도 같이 렌더링이 된다. (username 이 만들어지니까, 감싸고 있는 username 때문에.)

```
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

### 콘텍스트 API로 데이터 전달하기  





