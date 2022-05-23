3강 2048 게임 만들기
===
코드 확인 https://github.com/landvibe/inflearn-react-project

https://github.com/landvibe/inflearn-react-project/tree/master/game2048/

<hr/>

lodash 라는 함수형 프로그램을 사용하면 반복적인 작업을 간편하게 해주는 기능이 있긴하다.

```
{ new Array(4).fill(0).map() } 
```
배열을 사용하는 방식이 있긴하지만, lodash 를 사용하면 좀더 쉽다.

```
npm i lodash 
```
lodash 는 함수형 프로그래밍에서 많이 쓰는데, 함수형 프로그래밍을 굳이 안하더라도, lodash 에서 제공하는 여러가지 기능이 있기때문에 그 기능만 활용하는 용도로도 괜찮다. times 라는 기능.
http://kbs0327.github.io/blog/technology/lodash/

Game.js 
```
import times from 'lodash/times';

import { MAX_POS } from '../constant';

...
      <div className="grid-container">
        {times(MAX_POS, y => (
          <div key={y} className="grid-row">
            {times(MAX_POS, x => (
              <div key={y * MAX_POS + x} className="grid-cell"></div>
            ))}
          </div>
        ))}
      </div>
 ...

```
constant.js
```
export const MAX_POS = 4;
```

상태값을 나타내는 부분은 상태값으로 만들어주는게 좋다.

Game.js.  
```
...
  const [tileList, setTileList] = useState(getInitialTileList);
  useMoveTile({ tileList, setTileList, setScore });
...
  <div className="tile-container">
        {tileList.map(item => (
          <Tile key={item.id} {...item} />
        ))}
  </div>

```
Game.js

```
import React, { useState } from 'react';
import times from 'lodash/times';
import useMoveTile from '../hook/useMoveTile';
import { getInitialTileList } from '../util/tile';
import { MAX_POS } from '../constant';
import Tile from './Tile';


export default function Game({ setScore }) {
  const [tileList, setTileList] = useState(getInitialTileList);
  useMoveTile({ tileList, setTileList, setScore });
  return (
    <div className="game-container">
      <div className="grid-container">
        {times(MAX_POS, y => (
          <div key={y} className="grid-row"> 
            {times(MAX_POS, x => (
              <div key={y * MAX_POS + x} className="grid-cell"></div>
            ))}
          </div>
        ))}
      </div>


      <div className="tile-container">
        {tileList.map(item => (
          <Tile key={item.id} {...item} />
        ))}
      </div>
    </div>
  );
}
```

함수관련은 util 안에 title.js 파일안에 넣기.  
title.js.  
```
export function getInitialTileList() {
  const tileList = [];
  return tileList;
}
  
```
빈배열을 넣으면 안되고 x 랜덤하게 생성. XY값을 랜덤하게 생성. 1~4 사이에 랜덤하게 반환. 숫자과 관련된 유틸만들기   
number.js 만들기   
```
export function getInitialTileList(from, to) {
  return Math.floor(Math.random()* to + from);
}
```

title.js.  
```
export function getInitialTileList() { // 여기서 maketitle 을 두번하면 된다.
  const tileList = [];
  const tile1 = makeTile(tileList); // title1에 넣고
  tileList.push(tile1);
  const tile2 = makeTile(tileList); // title2에 넣고
  tileList.push(tile2);
  return tileList;
}

function checkCollision(tileList, newTile) { 
  return tileList.some(tile => tile.x === newTile.x && tile.y === newTile.y); // 이아이템중에 하나라도 만족하는게 있으면 true 반환
}
let currentId = 0;
export function makeTile(tileList) {
  let newTile; // 기존에 만들었던 타일들과 충돌하면 안되니까 검사하는것
  while (!newTile || (tileList && checkCollision(tileList, newTile))) { // 타일이 없거나, titleList 가 있는지 검사 checkCollision
    newTile = {
      id: currentId++,
      x: getRandomInteger(1, MAX_POS),
      y: getRandomInteger(1, MAX_POS),
      value: 2, // 고정값
      isNew: undefined,
      isMerged: undefined,
    };
  }
  return newTile;
}
  
```

***처음에 생성하기 끝***

키보드이벤트 핸들러
--
keyboard.js      
>외부라이브러리 사용. hotkeys-js. npm i hotkeys-js
>사용법 //hotkey(key,() =>);

```
import hotkeys from 'hotkeys-js';

const observerMap = {}; // obser 관리용, 이곳에 넣음
export function addKeyCallback(key, callback) { // key 값과 불려질함수
  if (!observerMap[key]) { // 없으면 초기화
    observerMap[key] = [];
    hotkeys(key, () => executeCallbacks(key)); //등록해주기
  }
  observerMap[key].push(callback);
}
export function removeKeyCallback(key, callback) { // 키 지우기
  observerMap[key] = observerMap[key].filter(item => item !== callback); // callback 이 아닌것만 제거
}


function executeCallbacks(key) { // 키에 해당되는 callback 모두 실행
  for (const ob of observerMap[key]) {
    ob();
  }
}
```

Game.js 에서 필요한건 up, down, left, right 혹을 이용해서 관리
```
useMoveTile({ tileList, setTileList, setScore });
```
useMoveTitle.js.  
```
import { useEffect } from 'react';
import { makeTile, moveTile } from '../util/tile';
import { addKeyCallback, removeKeyCallback } from '../util/keyboard';


export default function useMoveTile({ tileList, setTileList, setScore }) { 
  useEffect(() => { // 키보드 util 
    function moveAndAdd({ x, y }) { // 움직이고 추가하는것까지 x,y 를 intiger 로 받아서 오른쪽, 왼쪽, 위, 아래
      const newTileList = moveTile({ tileList, x, y }); // 움직여서 새로 만들어저 주는것
      const score = newTileList.reduce( //움직인다음에 추가
        (acc, item) => (item.isMerged ? acc + item.value : acc),
        0,
      );
      setScore(v => v + score);
      const newTile = makeTile(newTileList);
      newTile.isNew = true;
      newTileList.push(newTile);
      setTileList(newTileList);
    }


    function moveUp() {
      moveAndAdd({ x: 0, y: -1 });
    }
    function moveDown() {
      moveAndAdd({ x: 0, y: 1 });
    }
    function moveLeft() {
      moveAndAdd({ x: -1, y: 0 });
    }
    function moveRight() {
      moveAndAdd({ x: 1, y: 0 });
    }
    addKeyCallback('up', moveUp);
    addKeyCallback('down', moveDown);
    addKeyCallback('left', moveLeft);
    addKeyCallback('right', moveRight);
    return () => {
      removeKeyCallback('up', moveUp);
      removeKeyCallback('down', moveDown);
      removeKeyCallback('left', moveLeft);
      removeKeyCallback('right', moveRight);
    };
  }, [tileList, setTileList, setScore]);
}
```
util/ tile.js에서 키만들어서 Game.js 에 넣어주기.  
```
let currentId = 0; // key으로 
export function makeTile(tileList) {
  let newTile;
  while (!newTile || (tileList && checkCollision(tileList, newTile))) {
    newTile = {
      id: currentId++, // 아이디값으로 넘기기
      value: 2,
      isNew: undefined,
      isMerged: undefined,
    };
  }
  return newTile;
}
```

Game.js
<pre><Tile key={item.id} {...item} /> // 여기 key 값으로 활용</pre>

key관련된 에러 수정완료.   
```
export const assert = function (condition, message) {
  if (!condition) {
    throw new Error(`Assertion failed: ${message}`);
  }
};
```
<hr/>

util/tile.js. 
```
import { assert } from './assert';
assert(x === 0 || y === 0, ''); //x , y중에 하나는 무조건 0이어야 한다. 
```
util/assert.js 
```
export const assert = function (condition, message) {
  if (!condition) {
    throw new Error(`Assertion failed: ${message}`);
  }
};
```
프로덕션 빌드를 할때 제거해주면 좋긴하지만, 제거안해도 되긴해요.  

<hr/>

check js를 해서 에러를 줘야 하는건데, game.js 에서 움직일때 title-mergee 라는걸 표현할때 그정보를 가지고있는것이다
title.js
```
isNew : undefinded,
isMerge : undifinded,
...
tile.isMerged = true;
```
에러처리   
<hr>
useMoveTitle.js    
<pre>newTile.isNew = true</pre>

Game.js.  렌더링하는부분에서 간단하게 처리하기위에 아래부분을 분리.  
```
...
<div
      key = {item.id}
      className = {`tile tile-${item.value} tile-position-${item.x}-${item.y}`}
      >
      <div className="title-inner">{item.value}</div>
</div>
...
```
Title.js.  
npm i classnames 클래스네임 플로그인사용   

```
import React from 'react';
import cn from 'classnames'; // 클래스네임패키지


export default function Tile({ x, y, value, isMerged, isNew }) { 
  return (
    <div
      className={cn(`tile tile-${value} tile-position-${x}-${y}`, {
        'tile-merged': isMerged, // 클래스네임패키기 사용하기
        'tile-new': isNew, // 클래스네임패키기 사용하기
      })}
    >
      <div className="tile-inner">{value}</div>
    </div>
  );
}
```
Game.js.  
```
...
<div className = 'title=container">
      <Tile key={item.id} {...item} />
</div>
...
```
<hr/>

score 표현하기.  
Header.js. 
```
import React from 'react';


export default function Header({ score, bestScore }) { // 값보냄
  return (
    <header className="heading">
      <h1 className="title">2048</h1>
      <div className="scores-container">
        <div className="score-container" style={{ marginRight: 5 }}>
          {score}
        </div>
        <div className="best-container">{bestScore}</div>
      </div>
    </header>
  );
}
```
App.js.  
```
import React, { useState, useEffect } from 'react';
import Header from './component/Header';
import AboveGame from './component/AboveGame';
import Game from './component/Game';
import useLocalStorageNumber from './hook/useLocalStorageNumber';


export default function App() {
  const [score, setScore] = useState(0);
  const [bestScore, setBestScore] = useLocalStorageNumber('bestScore', 0);


  useEffect(() => {
    if (score > bestScore) {
      setBestScore(score);
    }
  });


  return (
    <div className="container">
      <Header score={score} bestScore={bestScore} />
      <AboveGame />
      <Game setScore={setScore} /> // 스코어올려주기
    </div>
  );
}
```
score 를 올려주는 부분은 game 쪽에서 하는게 좋고, game.js move 할때 올려주는게 좋고 title.js isMerged 할때 점수가 올라갈때 준다.         
hook/useMoveTile.js.   
```
const score = newTileList.reduce(
        (acc, item) => (item.isMerged ? acc + item.value : acc),
        0,
);
setScore(v => v + score);
```
<hr/>

best 넣기.      
로컬스토리지 활용.      
hook 으로 관리   
hook/useLocalStorageNumber.js.  
```
import { useState, useEffect } from 'react';


export default function useLocalStorageNumber(key, initialValue) {
  const [value, setValue] = useState(initialValue);


  useEffect(() => {
    const valueStr = window.localStorage.getItem(key);
    if (valueStr) {
      setValue(Number(valueStr));
    }
  }, [key]);


  useEffect(() => { // set 하는것
    const prev = window.localStorage.getItem(key);
    const next = String(value);
    if (prev !== next) {
      window.localStorage.setItem(key, next);
    }
  }, [key, value]);


  return [value, setValue];
}
```
src/App.js 여기서 관리   
```

import useLocalStorageNumber from './hook/useLocalStorageNumber';
...

 const [bestScore, setBestScore] = useLocalStorageNumber('bestScore', 0); 
 
...

useEffect(() => { // 기존bestScore 보다 커진순간에 하면 됨.
    if (score > bestScore) {
      setBestScore(score);
    }
});
 
```



