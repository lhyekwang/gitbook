[실전 리액트 프로그래밍](https://www.inflearn.com/course/%EC%8B%A4%EC%A0%84-%EB%A6%AC%EC%95%A1%ED%8A%B8-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D#curriculum)
<pre><code>$ npx create-react-app Project</code></pre>

버전 낮추는법
```
$ nvm add react@16.8.0
$ nvm add react-dom@16.8.0
```
강의시 basic 환경   
router 특정버전 설치하기 : ( 6이후 변경된사항이 있어서 강의를 들을때 코드부분이 달라진 것들이 있다. 낮춰서 설치해야한다.)
```
react : ^16.8.4
node: v14.x
npm: 6.x
$ npm install router-react-dom@5
```

2강 중요하시잠 헷갈리는 리엑트개념 이해하기
==
리액트를 사용한 코드의 특징
--

각 클릭버튼을 사용해서 할일목록 만들기   
* 기본자바스크립트로 만든것은 DOM API를 사용해서 주로 만들었음.

```javascript
function onAdd(){
  const todo = {id:currentId, desc};
  setCurrendId(currentId + 1);
  setTodoList([...todoList, todo]);
}
function onDelete(e){
  const id= Number(e.target.dataset.id);
  const newTodoList= todoList.filter(todo => todo.id !== id);
  setTodoList(newTodoList)
}
```
* React : 이벤트 핸들러에서 데이타를 변경하는 작업만 하고 있음
* html : 데이타를 변경하고 있지만 UI코드도 섞여있음
* React 에서는 JSX코드로 한번만 작성해주면, 그이후에는 신경쓰지 않아도 된다.
* 비즈니스로직과 UI코드가 분리되어있음.
* html : 명령형
* React : 선언형
* html 은 프로그램이 실행되고 UI가 어떻게 실행되는지 처음은 보이지만 그 이후에 어떻게 보이는지 보이지 않음. dom API를 사용하기 때문에 dom환경이 아닌경우에는 사용하기 힘듬.
* React 는 UI가 어떻게 변경되는지 보임. 무엇을 그리고 있는지 나타나기때문에 다양한 방식으로 나타낼수 있음. 돔을 바꿔줘도 사용가능.
* 선언형 프로그램은 명령형보다 추상화단계가 높고 비즈니스로직에 좀더 집중할 수 있음.

**React에서는 UI코드는 선언형으로 작성해두고, 이벤트 헨들러에서는 데이타만 수정하게 되면 리액트가 자동으로 UI가 자동으로 렌더링을 해준다.**

컴포넌트의 속성값과 상태값
--
### 컴포넌트 상태값
리액트 컴포넌트에서 UI데이타를 다루는 방법
```javascript
let color = 'red'
function onClick(){
color = 'blue'
}
return (
  <button style={{ backgroundColor: color }} onClick = {onClick}>
  좋아요
  <button>
)
```
>리액트 컴포넌트에서는 UI데이타를 값이나 속성값이나 상태값으로 관리를 해야 한다. 위의 코드는 그렇게 관리하지 않음.
>외부에 color 라는 값을 backgroundColor에 입력을 했고, 처음에 레드였다가 다음에 블루로 변경되는 코드인데, 위에같은 경우는 리액트가 color가 변경이 되었지만 변경되어있는지 모르는 코드
```javascript
const [ color, setColor ] = useState('red');

function onClick(){
  setColor['blue']
}
return (
  <button style={{ backgroundColor: color }} onClick = {onClick}>
  좋아요
  <button>
)
```
>useState 를 사용 할 때에는 상태의 기본값을 파라미터로 넣어서 호출해줍니다. 이 함수를 호출해주면 배열이 반환되는데요, 여기서 첫번째 원소는 현재 상태, 두번째 원소는 Setter 함수입니다.
>비구조화함수를 통해서 각원소를 추출해주고, Setter 함수는 파라미터로 전달 받은 값을 최신 상태로 설정해줍니다.

### 컴포넌트 속성값

Title.js   title 안에 값은 부모로 부터 받은 속성값(props) 이다
```javascript
export default function Title(props){
  return <p>{props.title}</p>
}
// props 는 부모컴포넌트가 전달해주는 속성값 
// 변경 
export default function Title({ title }){
  return <p>{title}</p>
}
```


Counter.js
```javascript
export default function Counter(){
  const [count, setCount ] = useState(0);
  const [count2, setCount2 ] = useState(0);
  function onClick(){
    setCount (count +1 );
  }
  function onClick2(){
    setCount2 (count2 +1 );
  }
  return (
    <div>
      <Title title={`현재 카운트 : ${count}`} />
      <button onClick = {onClick}>증가</button>
      <button onClick = {onClick2}>증가2</button>
    </div>
  )
}
```
>부모컴포넌트(counter)가 렌더링이 될때마다 자식(Title)콤포넌트도 렌더링됨.  
>그러나 onClick2를 불렀을때도 상태값이 변하지 않았는데도 title 컴포넌트는 계속 호출이된다.   
>값이 변경될때만 호출할때는 React.memo를 사용할수있음.   

Title.js
```javascript 
function Title({ title }){
  return <p>{title}</p>
}

export default Reat.memo(Title)
```
>각 콤포넌트별로 상태값을 가지는 메모리공간이 있어서 같은 컴포넌트라고 하더라도 자신만의 상태값이 존재한다. 각자 사용된곳에서 각자의 상태값을 유지하고 있음.   

>속성값은 불변변수이지만, 상태값은 불변변수가 아니다. 하지만 상태값도 불변변수로 관리하는게 좋음. (불변변수 는 변수의 값을 바꿀 수 없는 변수를 말한다다.)
>속성값의 변경을 시도하려고 하면 아래와 같이 에러가 발생한다.

Title.js
```javascript 
function Title(props){
  props.title = 'aasef'
  return <p>{prpps.title}</p>
}

export default Reat.memo(Title)
```
>> 에러

>그래서 title을 수정하고 싶으면 title 을 갖은 부모컴포넌트에서 관리하는 상태값 변경함수를 이용해야 한다. 상태값도 불변변수로 관리하는게 좋은데, 상태값은 객체형식으로 변경해보자.
Counter.js
```javascript
export default function Counter(){
  const [count, setCount ] = useState({ value : 0 });
  function onClick(){
  count.value += 1 ;
    setCount (count);
  }
  return (
    <div>
      <Title title={`현재 카운트 : ${count}`} />
      <button onClick = {onClick}>증가</button>
    </div>
  )
}
```
>**안된다.** React 는 이전값과의 단순비교로 판단을 하는데, count  는 객체인데, 객체 참조값은 변하지 않았기때문에 값이 변경되지 않았다고 판단한다 그래서 setCount 는 무시가 된다.

Counter.js
```javascript
export default function Counter(){
  const [count, setCount ] = useState({ value1 : 0 ,value2 : 0, value3 : 0 });
  function onClick(){
    setCount({ ...count, value:count.value +1 });
  }
  return (
    <div>
      <Title title={`현재 카운트 : ${count}`} />
      <button onClick = {onClick}>증가</button>
    </div>
  )
}
```
>**setCount**로 불러넣고 변경하고자 하는것을 덮어쓰는 방식이다.

컴포넌트의 함수의 반환값
--
1. 리액트요소반환, 
2. 문자열, 
3. 숫자, 
4. div 리액트요소, 
5. 배열, 배열 요소로반환 (배열로 반환시 key 요소가 없으면 에러), 
6. fragment(div를 안쓰고 싶을때 , <></> 이렇게 쓰기도 함.), 
7. null, boolean 값, 
8. 리액트 portal (root 말고 다른곳에 렌더링이가능 / ReactDom 에 있음 (모달만들때 사용) 
  ```
  ReactDOM.createPortal
  ```
리액트 요소와 가상돔 1
--
빠른변경을 위해서 돔을 변경하지 않는게 제일 좋다. 리액트는 메모리에 가상돔을 올려놓고, 이전과 이후의 가상돔을 변경하는방식이다.
리액트 요소로 부터 가상돔을 만들고, 실제 돔을 변경하는 방식을 보여줌.
Object(기본돔)(02:37초) 과 콤포넌트를 이용한 리액트 요소 (02:37초)를 콘솔창에 찍히는걸로 확인해보면 각 요소들을 알수 있음.
리액트 요소의 type 은 컴포넌트 함수가 입력되어있음. 저 함수를 이용해서 렌더링을 위한 정보를 얻는다.   
리액트 요소는 변경하려고 하면 에러가 발생된다. 콘솔창보면 변경된것만 렌더링 되고있음.    
div 요소에 key가 변경되면 다른요소라고 생각해서 새로만들어 붙임.(component 의 키를 변경하게 되도, ummount와 mount를 하면서 새로 그려진다.)       
style 은 다른요소라고 생각하지 않는다.      


리액트 요소와 가상돔 2
--
렌더(render)단계 (가상돔) : 뭐가 변경될지 파악하는것 . 리액트로 부터 가상돔을 만들고 새로 만들어 질것을 이용함.   
커밋단계 (실제돔) : 파악되 변경사항을 실제돔에 반영하는 단계   

>트리구조로 만들고 요소들이 변경될때 각각 비교해서 변경된부분만 커밋된다. 
>render 단계에서 그리고, memo 단계에서 다른부분만 값을 바꾼다.   

리액트 훅 기초 익히기1
--
***훅에 대한 사용이유 맟 useEffect 함수설명***.  
컴포넌트에 기능을 추가할 때 사용하는 함수
- ex ) 컴포넌트에 상태값 추가, 자식 요소에 접근
- 리액트 16.8에 새로 추가됨
- - 그전에는 클래스형 컴포넌트 사용
- - 클래스형 컴포넌트보다 장점이 많으며 리액트 팀에서도 훅에 집중하고 있음
useState 상태값 추가
useEffect 부수효과 처리
- 서버 API 호출, 이벤트 핸들러 등록 등
```javascript
const [count, setCount ]= useState(0); 
function onClick(){
  setCount(count + 1);
  setCount(count + 1);
}
console.log('render called');
return (
  <div>
    <h2>{count}</h2>
    <button onClick={onClick}>증가</button>
  </div>
)
```
count 를 두번증가시키는걸 의도했지만, 하나씩만 추가된다.   
상태값 변경 함수는 **비동기 이면서 배치(batch)** 로 처리되기 때문에 이 코드에서 의도한대로 동작하지 않는다.
리액트는 효율적으로 작동하기 위해서 여러개의 상태값 변경 요청을 배치로 처리합니다.   
따라서 onClick 를 해도 여기 있는 로그는 한번만 출련된다.
상태값 변경하면서 배치로 처리하는 이유(동기는 순서대로, 비동기는 여러가지를 같이) 는 리액트 상태값 변경을 동기로 처리하면 하나의 상태값이 변경될때마다 화면을 다시 그리기때문에 성능이슈가 생긴다.
상태값 변경을 동기로 처리하고 화면을 그리지 않는다면 UI데이타가 다르게 보여지는 혼란이 생길것이다.   
```javascript
const [count, setCount ]= useState({ value : 0 }); 
function onClick(){
  setCount({value : count + 1});
  setCount(count + 1);
}
console.log('render called');
return (
  <div>
    <h2>{count}</h2>
    <button onClick={onClick}>증가</button>
  </div>
)
```
>객체여도 동일함 안된다.

```javascript
const [count, setCount ]= useState(0); 
function onClick(){
  setCount(v => v+1);
  setCount(v => v+1);
}
console.log('render called');
return (
  <div>
    <h2>{count}</h2>
    <button onClick={onClick}>증가</button>
  </div>
)
```
>상태값 변경 함수에 함수를 입력하는 방법이 있음. 이건된다.
>함수를 입력하면 처리되기 직전의 상태값을 매개변수로 받기 때문에 원하는대로 동작. 
>onClick 이벤트 핸들러는 리액트내부에서 관리되는 리액트 요소에 입력이 되어 있기때문에 배치로 처리된다. 

```javascript
const [count, setCount ]= useState(0); 
function onClick(){
  ReactDOM.unstable_batcheUpdate(() =>
    setCount(v => v+1);
    setCount(v => v+1);
  });
}
useEffect(() => { // 일괄처리불가능하게 2회 렌더링. unstable_batcheUpdate 사용해서 가능하게 하였음.
  window.addEventListener( 'click', onClick );
  return () => window.removeEventListener('click', onClick);
});
console.log('render called');
return (
  <div>
    <h2>{count}</h2>
    <button onClick={onClick}>증가</button>
  </div>
)
```
>unstable_batcheUpdate 이걸 사용해서 외부함수를 두번 실행할수 있도록 해준다. 

***useEffect***
이 안에 있는 함수는 렌더링이 완료된후에 비동기로 호출됨. 첫번째 인자가 부수효과라고 한다.
특별한 이유가 없다면 부수효과는 useEffect로 처리해주는게 좋다.   
useEffect 함수가 리턴하는 함수는 컴포넌트가 언마운트 될때 1회는 꼭 실행되는 함수입니다. 따라서 이벤트 함수의 트리거를 해제하는 함수를 호출합니다. 
두번째 인자로 의존성 배열 값을 빈 배열을 넣어주면, 컴포넌트가 생성될때만 호출되며, 컴포넌트가 사라질 때만 리턴 함수가 호출됩니다.

>컴포넌트의 렌더링중에 부수효과를 발생시키는 것은 프로그램의 복잡도를 크게 증가시키는데, 유닛테스트를 작성하기 힘들어지고, 순수함수가 가지는 여러장점을 포기하는것임.   
>컴포넌트 렌더링중에 부수효과를 발생시키는건 거의 없고, concurrent mode로 동작할 미래의 리액트는 컴포넌트 함수가 호출되도 중간에 렌더링을 취소될 수 있음.  

한번 렌더링하기 위에 컴포넌트 함수가 여러번 호출 될 수도 있음. 컴포넌트 내부에서 직접 실행하는 부수 효과가 있다면 한번 렌더링할때 여러번 부수효과가 발생할 수 있다는것이다.   
두번째인자를 활용해서 여러번 호출되는것을 막을수 있다.

```javascript
useEffect ( ()=> {
  document.title = `업데이트 횟수 : ${count}; // 
})
return <button onClick ={ ()=> setCount(count +1) }>increase</button>
```
>useEffect 훅의 사용법은 
>첫번째 매개변수로 함수를 입력하면, 컴포넌트가 렌더링이 된후에 호출이 된다.
>정확하게 말하면 렌더링결과가 실제 돔에 반영이 되고, 비동기로 호출이 된다.
>버튼을 클릭할때마다 컴포넌트는 렌더링이 될꺼고, 렌더링된후에 이 부수효과도 실행이 된다.

리액트 훅 기초 익히기2
--
***의존성배열(두번째 인자)의 성격과 넣어야 되는것들*** 
```javascript
const vaule = userId + 10;
function func1(){
  console.log(userId);
}
useEffect( ()=> {
  console.log(value);
  func1();
  getUserApi(userId).then(data => setUser(data));  
}, [userId, value, func1] )
```
>두번째 매개변수에는 배열(의존성배열)이 있을경우 이 배열이 변경될때만 부수효과 함수가 실행이 된다. userId가 변경될때만 실행이 된다.
>속성값,상태값 변경함수(setUser)는 안넣어도 된다.  
>지역성 변수(value) 를 사용한다면 그것도 넣어줘야 한다. 
>함수도 넣어줘야 한다. 외부함수(getUserApi)말고, 지역함수(func1)도 넣어줘야 한다.(문제가 될수 있다고 하단에 나옴) 필요한경우에만 함수를 넣어주는게 좋다
>매번 넣어야 되는것때문에 문제가 되는경우가 있음

***부수효과함수가 반환하는 값***
```javascript
userEffect(()=>{
  const onResize = () => setWidth(width, innerWidth);
  window.addEventListener('resize', onResize);
  return () =>{
    window.removeEventListener('resize', onResize);
  }
}, []) //[]가 있으니 한번만 그려지고 맘. 삭제할경우는 계속 실행이 된다.
return <div>{`width is ${width}`}</div>
```
>함수를 반환시 다음부수효과 함수가 호출되기 직전에 호출된다.또는 컴포넌트가 사라지기 직전에 호출된다.그러니까 unmount 되기 직전에 마지막으로 호출된다
>따라서 컴포넌트가 unmout가 되기 직전에(사라지기직전에) 마지막으로 호출된다.
>의존성 배열을 빈배열로 넣을때는 컴포넌트가 생성될 때만 부수효과 함수가 호출되고, 컴포넌트가 사라질때만 반환한 함수가 호출된다.
>그래서 이벤트 핸들러를 등록하고 해제하는 이러한 패턴의 코드를 작성할수 있음.

훅 직접만들기
--
재사용가능하도록 변경하기

```javascript
const [user, setUser] = userState(null);
userEffect(()=>{
  getUserApi(userId).then(data => setUser(data));
},[userId])

```
변경후
useUser.js
```javascript
export default function useUser(userId){
  const [user, setUser] = userState(null);
  userEffect(()=>{
    getUserApi(userId).then(data => setUser(data));
  },[userId]);
  return user;
}

```
profile.js
```javascript
const user = useUser(userId);
```
위와같이 훅은 단순히 함수를 호출하는것만으로도 사용이 가능하다.

useWindowWidth.js
```javascript
export default function useWindowWidth(){
  userEffect(()=>{
    const onResize = () => setWidth(width, innerWidth);
    window.addEventListener('resize', onResize);
    return () =>{
      window.removeEventListener('resize', onResize);
    }
  }, [])
return width;
```
WidthPrint.js
```javascript
const width = useWindowWidth();
return <div>{`width is ${width}`}</div>;
```
mount 가 되었는지 안되었는지 확인하는 함수
```javascript
export default function useMounted(){
  const [mounted, setMounted] = useState(false);
  useEffect(() =>{
    setMounted(true);
  }, []);
  return mounted;
}
```
- 그외 로그인여부를 확인하는것( useBlockIfNotLogin() ), 
- 사용자가 작성한 내용이 있을때 메세지 보내는것 저장되지 않았다는 메세지( useBlockUnsavedChange(desc) ),
- 로그인 유저인경우에만 호출을 해줄때 ( useEffectIfLoginUser(callback,deps) )
- 로컬스토리지 사용할때 ( useLocalStorage(key,initialValue) => [value, setValue] )

훅 사용시 지켜야 할 규칙
--
>규칙1 : 하나의 컴포넌트에서 훅을 호출하는 순서는 항상 같아야 한다.
>* if문, for 문 으로 사용하면 안됨, 함수안에서 호출해도 안됨. 어쩔땐 되고 안되고 이러면 안된다.
>규칙2 : 훅은 함수형 컴포넌트 또는 커스텀 훅 안에서만 호출되어야 한다.

훅데이타관리는 모르겠어요.


