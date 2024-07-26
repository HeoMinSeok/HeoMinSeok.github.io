---
title: "[유데미x스나이퍼팩토리] 프로젝트 캠프 : Next.js 2기 - 직무교육 React"
date: 2024-07-25
categories: [Next.js 직무교육, React]
tags: 
  - React
  - TypeScript
---
<style>
  .responsive-link-card {
    display: flex;
    align-items: center;
    border: 1px solid #d0d0d0;
    border-radius: 8px;
    overflow: hidden;
    cursor: pointer;
    text-decoration: none;
    color: inherit;
    margin-top: 20px;
    padding: 10px;
  }
  
  .responsive-link-card img {
    width: 150px;
    height: 100px;
    object-fit: cover;
    border-radius: 8px;
    margin-right: 15px;
  }
  
  .responsive-link-content {
    display: flex;
    flex-direction: column;
  }
  
  .responsive-link-content h2 {
    margin: 0;
    font-size: 1.2em;
  }
  
  .responsive-link-content p {
    margin: 5px 0 10px;
    color: gray;
    font-size: 0.9em;
  }
  
  .responsive-link-content small {
    color: #777;
  }
  
  @media (max-width: 768px) {
    .responsive-link-card {
      flex-direction: column;
      align-items: flex-start;
    }
    
    .responsive-link-card img {
      margin-right: 0;
      margin-bottom: 10px;
    }

    .responsive-link-content {
      width: 100%;
    }
  }
</style>
![Context API - Props Drilling](/assets/img/posts/training/context-drilling.png)
_Context API - Props Drilling_

## Props-Drliing 문제점

- 데이터를 전달하기 위해 많은 중간 컴포넌트들이 불필요하게 props를 받아서 다시 전달해야하는데, 이는 코드 가독성을 저하시킴
- 컴포넌트 트리가 변경되거나 중간에 새로운 컴포넌트를 추가하거나 제거해야 하는 경우, 모든 관련된 컴포넌트의 props를 업데이트해야해서 유지보수가 어려움
- 중간 컴포넌트들이 단순히 데이터를 전달하는 역할만 할 경우, 동일한 코드가 반복
- 특정 props를 전달하는 로직이 중간 컴포넌트에 포함되어 있는 경우, 이 컴포넌트를 재사용하기 어려워짐
- props가 자주 변경되는 경우, 많은 컴포넌트가 불필요하게 리렌더링 

이를 해결하기 위한 여러 방법들 중 `Context API`, `useReducer` 등이 있다.

## Context API란?
<span>`일일이 props를 넘겨주지 않고 컴포넌트 트리 전체에 데이터를 제공할 수 있는데, 이를 통해서 Props-Drilling 해결 가능`</span>

### Context API의 주요 구성 요소

- **React.createContext**: Context 객체를 생성
- **Context.Provider**: Context를 구독하는 컴포넌트들에게 Context 값을 제공
- **Context.Consumer**: Context의 현재 값을 구독하여 사용, 일반적으로 Consumer 대신 `useContext` 훅을 사용

### DevTools에서 Context API 확인

| 컴포넌트 트리 | Provider | Team | Player |
|---------------|---------------|---------------|---------------|
| ![컴포넌트 트리](/assets/img/posts/training/context-provider.png) | ![Provider](/assets/img/posts/training/context.png) | ![Team](/assets/img/posts/training/context-team.png) | ![Player](/assets/img/posts/training/context-player.png) | 

  - **컴포넌트 트리**: 첫 번째 이미지는 DevTools에서 본 컴포넌트 트리이다. `App` 컴포넌트 아래에 `TeamProvider`가 있으며, 그 안에 `Context.Provider`가 있는데,`ContextTeam`과 `ContextPlayer` 컴포넌트는 `Context.Provider`의 자식으로 있어 `Context.Provider`의 props를 훅 호출로 사용할 수 있다.
  - **Provider**: `Context.Provider`는 `teams` 데이터를 포함하고 있으며, `dispatch` 함수와 `state`를 props로 전달하는데, 이를 통해 하위 컴포넌트들이 Context의 값을 사용할 수 있게 된다.
  - **Team, Player**: Context를 통해 전역으로 있는 데이터를 가져왔기 때문에 props 필드는 비어있는 것을 볼 수 있다.
  
실제 코드에서도 컴포넌트에 props를 전달하지 않고, 해당 컴포넌트에서 Context를 호출해 데이터를 가져왔다.
```typescript
/* useTeams는 ContextTeams를 커스텀 훅으로 만들었음 */
const { teams, dispatch, state } = useTeams(); 
이런 식으로 호출해서 필요한 데이터를 뽑아올 수 있음

/* useTeams 커스텀 훅 */
import { useContext } from 'react';
import TeamContext from '../context/ContextTeams';

const useTeams = () => {
  const context = useContext(TeamContext);
  if (!context) {
    throw new Error('useTeams must be used within a TeamProvider');
  }
  return context;
};

export default useTeams;
```

### DevTools에서 Props Drilling 확인

| 컴포넌트 트리| 부모 컴포넌트에서 Props 전달  | Team | Player |
|---------------|---------------|---------------|---------------|
| ![컴포넌트 트리](/assets/img/posts/training/props-tree.png) | ![컴포넌트 트리](/assets/img/posts/training/props.png) | ![Team](/assets/img/posts/training/props-team.png) | ![Player](/assets/img/posts/training/props-player.png) | 

  - **컴포넌트 트리**: 첫 번째 이미지는 DevTools에서 본 컴포넌트 트리이다. `App` 컴포넌트 아래에 `DrillingTeam`, `DrillingPlayer`가 있다. 
  - **props 전달**: props를 자식 컴포넌트로 전달하는 코드 사진이다.
  - **Team, Player**: 이미지를 보면 명확하게 props로 데이터를 받아오는 것을 볼 수 있다.
  
> 실제 프로젝트에서는 데이터도 많고 한데, props로 일일이 다 전달하면 위에서 말한 문제점이 생겨나게 되는 것이다. Context API는 전역으로 상태를 담아주어서 따로 props를 컴포넌트마다 전달하지 않아도 useContext나 Context를 커스텀 훅에 담아서 값을 쉽게 뽑아올 수 있다. 하지만 Context도 구독하는 모든 컴포넌트가 값 변경될 때마다 리렌더링 되어 성능에 문제가 생길 수 있다.
  
## Reducer란?
<span>`하나의 컴포넌트 내에서 state를 다루는 로직을 해당 컴포넌트로 분리하여 외부에서 처리 할 수 있게 해줌`</span>

![useReducer](/assets/img/posts/training/reducer.webp)
_useReducer Flow_

### useReducer의 주요 구성 요소

- **State**: 현재 컴포넌트의 상태
- **Dispatch**: 상태를 업데이트하는 함수, 특정 액션을 dispatch 함수에 전달하면, 이 액션에 따라 상태가 업데이트
- **Reducer**: 상태 업데이트 로직을 정의하는 함수, 이 함수는 현재 상태와 액션 객체를 인수로 받아 새로운 상태를 반환

### useReducer의 흐름
1. EventHandler: 컴포넌트에서 이벤트 핸들러가 호출
2. Dispatch: 이벤트 핸들러는 dispatch 함수를 호출하여 액션 객체를 전달
3. Reducer: dispatch 함수는 액션 객체를 reducer 함수로 전달
4. State Update: reducer 함수는 현재 상태와 액션 객체를 기반으로 새로운 상태를 반환
5. Component Re-render: 상태가 업데이트되면 컴포넌트가 다시 렌더링

### DevTools에서 useReducer 확인

<div style="text-align: center;">
  <img src="/assets/img/posts/training/reducer.gif" alt="reducer" style="height: 100%;">
</div>
<br>
> 1.각 팀의 버튼을 누른다. <br>
> 2.dispatch 함수가 호출되어 액션 객체를 reducer 함수로 전달한다. <br>
> 3.reducer 함수는 현재 상태와 액션 객체를 기반으로 새로운 상태를 반환한다. 즉, 해당 팀의 선수를 반환한다. <br>
> 4.상태가 업데이트되면 컴포넌트가 다시 렌더링된다.
   
**이와 같이 useReducer를 사용하면 복잡한 상태 로직을 컴포넌트 내에서 효율적으로 관리할 수 있다.<br> useReducer는 특히 상태 업데이트 로직이 복잡하거나 여러 액션 타입을 처리해야 할 때 유용하다.**

<br>

### 예제 코드

<div class="responsive-link-card" onclick="window.open('https://stackblitz.com/edit/vitejs-vite-genj61?file=src%2FApp.tsx', '_blank');">
  <div>
    <img src="./assets/img/posts/training/stack-blitz.webp" alt="stackblitz">
  </div>
  <div class="responsive-link-content">
    <h2>Voiteks - Vote (forked)</h2>
    <p>StackBlitz</p>
    <small>stackbiltz.com</small>
  </div>
</div>
<br>