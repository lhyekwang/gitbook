3강 2048 게임 만들기
===
코드 확인 https://github.com/landvibe/inflearn-react-project

https://github.com/landvibe/inflearn-react-project/tree/master/game2048/

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
https://github.com/landvibe/inflearn-react-project/blob/f68d7e7aa277b102f5ccf402134a3e57619d069f/game2048/final/src/component/Game.js#L8-L30
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
>외부라이브러리 사용ㅙㅛ

keyboard

```
import hotkeys from 'hotkeys-js';

const observerMap = {};
export function addKeyCallback(key, callback) { // key 값과 불려질함수
  if (!observerMap[key]) {
    observerMap[key] = [];
    hotkeys(key, () => executeCallbacks(key));
  }
  observerMap[key].push(callback);
}
export function removeKeyCallback(key, callback) { // 키 지우기
  observerMap[key] = observerMap[key].filter(item => item !== callback);
}


function executeCallbacks(key) {
  for (const ob of observerMap[key]) {
    ob();
  }
}
```



