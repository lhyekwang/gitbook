리액트 실전 활용법
===
### 추천하는 컴포넌트 파일 작성법
1. 속성값의 타입정리.   
함수의 proTypes 라는 속성으로 객체입력 한다.   
```javascript
MyComponent.proTypes = {
  //...속성값의 타입정보 입력
}
```
2. 함수의 이름 부여   
2-1. 컴포넌트의 매개변수는 명명된 매개변수 문법으로 작성할것
```javascript
export default function MyComponent({ prop1, prop2 }){
  //...
}
```
3. 컴포넌트 바깥에 있는 변수와 함수는 파일의 가장 밑에 정의.   
3-1. 변수명은 대문자로 
3-2. 다소 큰 객체를 생성할때는 컴포넌트 바깥에서 작성해주는게 좋음
```javascript
const COLUMNES = [
  {id : 1, name : 'afafea'}
  {id : 2, name : 'afafea'}
]
```
4. 컴포넌트 내부에서 서로 연관된 코드는 한곳으로 몰기.  (가독성도 좋고, 이후 custom hook 으로 빼기도 좋음.)

```javascript
 const [user, setUser] = useState(null);
 useEffect( ()=>{
   // 사용자 데이터 관리
 } , [userId])

const [width, setWidth] = useState(null);
 useEffect( ()=>{
   // 창너비 관리
 } )
```
훅으로 변경.   
```javascript
 const user = userUser(userId);
 const width = useWindowWidth();
```
### 속성값 타입 정의하기 : prop-types
prop-types 패키지는 속성값의 타입정보를 정의 할때 사용하는 공식 패키지. (타입 오류 사전검사가능).   **타입의 중요성**   
주석으로 옆에 자세하게 적어주세요.   
함수라고 해서 반환값의 정보를 따로 정의하지 않는다. 어떤값이 반환되는지 주석으로 써주면 된다.   
```javascript
User.propTypes = {
  mail : PropTypes.bool.isRequired, // 불린타입, 필수값 ,isRequired)
  age : PropTypes.number, 
  type : PropTypes.oneOf([ 'gold' , 'silver' , 'bronze' ]), // 3가지 값만 받음
  onChangeName : PropTypes.func, // {name : string} => void (반환값이 없다)
  onChangeTitle : PropTypes.func.isRequired, // 필수값
  menu : PropTypes.element , // 리액트 요소,  숫자는 거짓 <div>, <SomeComponent>
  description : PropTypes.node , // 리액트 요소,  리액트 컴포넌트 number, string, array, element, <SomeComponent>
  message : PropTypes.instanceOf(Message) , // 특정 클래스인지 객체인지 검사, new Message() 
  name : PropTypes.oneOf(['one', 'mike']) , // 저중에 하나만 만족
  width : PropTypes.oneOfType([PropTypes.number, PropTypes.string]) , // 타입은 number 나 string 
  age : PropTypes.arrayOf(PropTypes.number) , //넘버타입으로 된 배열
  info : PropTypes.shape({
    color : PropTypes.string,
    weight : PropTypes.number,
  }) , // 객체의 타입 
  age : PropTypes.objectOf(PropTypes.number) , //각 속성의 타입이  value 값이 number 
}
// prop-types 
```
```javascript
MyComponent.propTypes = {
  age : funciton( props, proName, componentName){
    const value = props [propName];
    if( value < 10 || value > 20){
      return new Error (
        `Invalid pro ${proName} supplied to ${componentName}.
        It must be 1- <= value <20.`,
      );
    }
  },
};
```
### 가독성을 높이는 조건부 렌더링 방법
  컴포넌트 내부에서 선택ㄷ적으로 렌더링하는것.   
  조건부 렌더링을 너무 많이 사용하면, JSX는 너무 복잡해져요.   
  삼항연산자보다는 &&를 쓰는게 좋아요.   
  ```javascript
    { isLogin ? (
      ...
    ) : null}
    // 삼항연산자보다 &&를 사용하면 더 간단하다.
    { isLogin && (
      ...
    ) }

  ```
  ... 부분이 너무 길때는 끊어주는게 좋아요.   
  ```javascript
    { isLogin ? (
      ...
    ) : isLogin2 ? (
      ...
    )}
    // 삼항연산자보다 &&를 사용하면 더 간단하다.
    { isLogin && (
      ...
    ) }
    { !isLogin && isLogin2 (
      ...
    ) }
  ```
  헷갈리는 &&   
  && 처음으로 거짓을 만나거나 끝까지 이동할때까지 평가한다.   
  || 처음으로 참을 만날때까지만 평가한다.   
  ```javascript
  const v1 = 'ab' && 0 && 2 ; // v1 ==== 0 ,   참 &&거짓
  const v2 = 'ab' && 2 && 3 ; // v2 ==== 3  참 && 참 && 참
  const v3 = 'ab' || 0 ; // v3 ==== 'ab'  참 &&
  const v4 = '' || 0 || 2 ; // v4 ==== 2 거짓 && 거짓 && 
  ```
>JSX에서는 false , null 은 무시가 된다.   

  ```javascript
    {cash && <p> {cash}원 보유  </p>} 
    // cash 가 0일경우 0으로 출력되므로, 
    {!!cash && <p> {cash}원 보유  </p>} 
    // cash를 블리언 타입으로 변환을 해줘야 한다.

    {memo && <p> {200 - memo.length} 입력가능</p>}
    // memo는 빈문자열인 경우 {''} 출력된다.
    {cash != null &&  <p> {cash}원 보유  </p>} 
    {memo != null && <p> {200 - memo.length} 입력가능</p>}
    // cash 가  0인경우에도, 
    // memo가 빈문자일경우에도

    <div>{students && students.map(/**/)}</div>;
    //students 은 배열이고 undefinded도 될수 있어서

    let students = [],
    <div>{students && students.map(/**/)}</div>;
  // 기본값을 넣어주는게 좋다.  
  ```
  

### 재사용성을 고려한 컴포넌트 구분법
>관심사의 분리. (비슷한 성능의 코드를 분리)   
>자식콤포넌트에서 부모데이타를  별도의 상태값을 관리하면 좋지 않음.   
>상태값은 일부 컴포넌트로 한정해서 관리하는게 좋다.   
>그리고 컴포넌트가 비즈니스 로직과 상태값을 갖고 있으면 재사용하기가 힘들다.   
비즈니스 로직과 상태값의 유무로 컴포넌트를 분리하는것을 추천.
재사용성이 좋은 컴포넌트와 그렇지 않은 컴포넌트로 구분.

**재사용성이 좋은 컴포넌트**의 정의는 비즈니스 로직이 없고, 상태값이 없는것이다.
단, mouse over와 같은 UI효과를 위한 상태값은 제외.   
UI만 담당하는 컴포넌트.   

**component (재사용성이 있음)**
- 친구목록만 렌더링하는 콤포넌트 (FriendList.s) : 속성값으로 friends를 받음
  ```javascript


  ```
- 드랍다운 형식으로 보여주면서 옵션이 선택되었을때 선택된 옵션을 알려주는 컴포넌트(NumberSelect.js) : 속성값으로( 현재값 , 옵션목록, label, onChange)
  ```javascript


    ```
**container (재사용성이 없음)**
- 비즈니스 모델도 있음, 상태값도 여기서 관리. FriendPage.js 
```javascript

  const [friends, setFriends] =userState();
  const [ageLimit, setAgeLimit] =userState(MAX_AGE_LIMIT);

  const friendsWithAgeLimit = friends.filter( item => item.age <= ageLimit );
  
  function onAdd(){
    const friend = getNextFriend();
    setFriends([...friends, friend]);
  }

    return(
      <div>
        <button onClick = {onAdd}>버튼 추가</button>
        <NumberSelect 
          value = {ageLimit} 
          options = {AGE_LIMIT_OPTIONS} 
          label='세 이하만 보기' 
          onChange = {setAgeLimit}
        />
        <FriendList friends = {friendsWithAgeLimit}/>
      </div>
    )
  

```

**App.js**
- 라우팅역활

```javascript

  <FriendPage />

```
> 나이제한 말고 이름길이로도 쓸수 있다. (FriendsPage.js 만 수정하면 됨)

### useEffect 실전 활용법1
의존성(dependency) 배열 다루는 법.   

```javascript
function profile(){

  const [user, setUser] = useState();
  userEffect( ()=> {
    fetchUser(userId).then(data => setUser(data));
  }, [의존성배열] );
  // ...

}

```
profile 컴포넌트가 렌더링 될때마나 노출되기 때문에 서버의 API가 호출하는 함수가 항상실행이 된다.   의존성배열에 [] (//빈배열)을 입력해서  마운트 된 후에 한번만 호출하게 할수도 있다.   
그러나 userId가 변경이 되어도 가져오지 못하기 때문에 의존성 배열에 userId 를 넣어주면 된다.   
fetchUser(userId) 의 매개변수가 추가가 되면 fetchUser(userId, needDeta) 의존성 배열에서 [userID, needData]를 넣어주어야 한다.
만약에 userId가 변경되지 않는다면 따로 hook를 만들어 주는게 좋다.
- 마운트되었을때만 작동하는 훅 
useOneMounted.js
```javascript
 export default funnction useOneMounted(effect){
   useEffect( effect, [])
 }
```
App.js
```javascript
import useOnMounted from '../useOnMounted'
 export default funnction App(){
   ...
    useOneMounted( () => fetchUser(userId, needDetail).then (data => setUser(data)) );
    ...
 }
```
부수효과의 함수의 반환값은 항상 함수이어야 한다.
```javascript
useEffect( async () => {
  const data = await fectchUser(userId);
  setUser(data);
} , [userId] );
```
부수효과 함수는 반환값이 항상 함수타입 이어야 하는데, async, await 은 Promise 객체를 반환하기때문에 부수효과 함수가 될수 없다.   
함수만 반환 할수 있고, 반환된 함수는 부수효과 함수가 호출되기 직전과 컴포넌트가 사라지기 직전에 호출된다.   

async, await를 쓰고 싶다면 함수릏 하나 만들어서 호출해줘야 한다.
```javascript
  useEffect( () => {
    async function fetchAndSetUser(){
      const data = await fetchUser( userId );
      setUser(data);
    }
    fetchAndSetUser();
  },[userId])
```

fetchAndSetUser 함수를 바깥에서도 사용하고 싶다면, 함수를 꺼내고, 의존성배열은 함수를 입력한다.   
함수안에서는 속성값과 함수값을 사용하고 있기 때문이다. 
```javascript
function Profile({ userId }){
    async function fetchAndSetUser(needDatail){
      const data = await fetchUser( userId, needDatail );
      setUser(data);
    }
     useEffect( () => {
        fetchAndSetUser(false);
      },[fetchAndSetUser])
}
```
그런데 이럴경우 Profile 컴포넌트가 렌더링될때마다 생성이 된다.   
의존성배열의 내용이 항상 변하고 부수효과 함수도 렌더링될때마나 실행됨다.   
그래서 이럴때 userCallback 훅을 사용한다.

```javascript
function Profile({ userId }){
    const fetchAndSetUser = userCallback(
      async function fetchAndSetUser(needDatail){
      const data = await fetchUser( userId, needDatail );
      setUser(data);
      },
      [userId],
    );
    useEffect( () => {
      fetchAndSetUser(false);
    },[fetchAndSetUser]);
}
```
useCallback 훅을 이용해서 userId가 변경될 때만 이 함수가 새로 생성되도록 하는것이다.   

### useEffect 실전 활용법2
가능하다면 의존성 배열을 사용하지 않는게 제일 좋음. 
의존성 배열을 관리하는데 생각보다 많은 시간과 노력이 들어가기 때문이다.   
특히, 속성값으로 전달된 함수를 의존성 배열에 넣는 순간 
그 함수는 부모 컴포넌트에서 useCallback 등을 사용해서 자주 변경되지 않도록 신경써서 관리해야 한다.   

1. 실행시점조절
```javascript
function Profile({ userId }){
   const [user, setUser] = userState();
   async function fetchAndSetUser(needDatail){
      const data = await fetchUser( userId, needDatail );
      setUser(data);
    }
  useEffect( () => {
    if (!user || user.id !== userId){
      fetchAndSetUser(false);
    }});
}
``` 
의존성 배열을 입력하지 않는 대신 부수효과 함수 내에서 **실행 시점을 조절** 할수 있다.
함수의 실행 시점을 의존성 배열로 관리하지 않고 **부수효과 함수 내에서 처리** (//!user || user.id !== userId)하면 이 부수효과 함수 안에서 사용하는 모든 변수는 최신화된 값을 참조 하므로 안심할 수 있다.   

2. 의존성배열에 상태값 변경 함수를 입력

```javascript
function MyComponent(){
   const [count, setCount] = userState();
  useEffect( () => {
    function onClick(){
      setCount( prev => prev +1 )
    }
    window.addEventListener( 'click' , onClick );
    return () => window.removeEventListener( 'click', onClick );
  });
}
``` 
이전상태값을 기반으로 다음 상태값 계산하기위해 의존성 배열에 추가하는데,    
상태값 변경함수(setCount)에 함수를 입력한다.

3. 여러 상태값을 참조하면서 값을 변경하는법. 
useReducer 함수를 사용한다. 

4. 

### 렌더링 속도를 올리기 위한 성능 최적화 방법1
componenet(data) > 가상돔과 비교 > 실제 돔에 반영   
: 데이타 (속성값이나 상태값)이 변경되면 자동으로 컴포넌트 함수를 이용해서 화면을 그림.
: 하나의 컴포넌트 단계에서 렌더링이 필요한지 판단하는 과정에 관하여-

1. memo 사용 (//React.memo ( 속성값 , 비교함수 )  )    
1-1. 객체를 불변객체로 관리
렌더링이 필요한지 아닌지 판단하는 과정이 (React. memo)   
  ```javascript
  function MyComponent(props){
    //...
  } 
  function isEqual (prevProps, nextProps){
    // true 또는 false 반환
  }
  React.memo(MyComponent, isEqual);
  ```

참또는 거짓을 반환해서 이전 렌더링을 사용하던가 가상돔을 업데이트 하던가 결정한다.   
isEqual 이 false 반환 : 이전과 달라.   
isEqual 이 true 반환 : 이전과 같아.   
비교방식 : 객체를 불변객체로 관리.   

  ```javascript
   preveProps.todos === nextProps.todos // 비교
```
```javascript
  const prevTodos = [1,2,3];
  const nextTodos = [ ..todos, 4]  // 새로운객체를 만듬

  prevTpdos === nextTodos

  ```
### 렌더링 속도를 올리기 위한 성능 최적화 방법2
컴포넌트 내부에서 생성되는 함수와 객체   
: 불필요하게 컴포넌트 함수가 실행되지 않게

내부 값이 너무 자주 변경이 되어서 문제가 될때,    
useCallback, useMemo, React.memo 성능이슈가 있을때만 쓰세요.    

값이 변경이 되어야 하는데, 변경이 되지 않아서 문제가 될때, 
```javascript
  fruits.push (newFruit); 
  // 상태값은 불편객체로 관리 해줘야 fruits 가 업데이트 된다.
  setFruits( [...fruits, newFruit ])
```

### 렌더링 속도를 올리기 위한 성능 최적화 방법3
componenet(data) > 가상돔과 비교 > 실제 돔에 반영   
: 가상돔에서  실제돔 부분에서 성능 최적화 방법   
자식요소가 많은 요소가 변경되면 화면이 끊기는 느낌이 볼수 있다.   
부모의 타입이 변경되면 자식이 새로 만들어진다. 자식요소에 영향이 있다.   
속성이 변경되면 해당속성만 변경된다. 자식요소에 영향이 없다. 
중간에 요소가 수정되는것도 뒤에요소도 다 영향이 간다.
**key 속성값을 이용**하면 중간에 값이 변경이 되어야 뒤의 요소가 변경이 되지 않았다는것을 안다.  

```javascript
<p key="apple">사과</p> 
```
































