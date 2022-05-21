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

2강
==
리액트를 사용한 코드의 특징
--

각 클릭버튼을 사용해서 할일목록 만들기   
* 기본자바스크립트로 만든것은 DOM API를 사용해서 주로 만들었음.

```
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
