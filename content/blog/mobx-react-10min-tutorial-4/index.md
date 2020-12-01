---
title: "[번역] Mobx와 React를 위한 10분 안내서 (4)"
date: "2020-11-30T17:19:25.236+09:00"
description: "Mobx와 React를 위한 10분 안내서"
---

# React를 반응형으로 만들기
지금까지 아주 단순한 report를 반응형으로 만들어 보았습니다. 같은 저장소(store)를 기반으로 반응형 UI를 만들어볼 차례입니다. React(이하 리액트) 컴포넌트는 이름과 달리 반응형이 아닙니다. `mobx-react-lite` 패키지에서 제공하는 고차 컴포넌트(HoC) `observer` 이 문제를 해결해줍니다. `observer`는 리액트 컴포넌트를 감싸는 `autorun`입니다. 이로써 컴포넌트가 자동적으로 상태와 동기화되도록 만들어줍니다. 이는 위에서 `report`에 적용한 것과 같은 개념입니다.
(역주: 현재 `mobx-react-lite`는 deprecated되었습니다. `mobx` 패키지를 통해 Mobx6를 사용하시는 걸 권장합니다.)

아래 예제 코드에서 몇가지 리액트 컴포넌트를 정의하고 있습니다. 이 중에서 `observer`로 감싼 부분만이 Mobx 코드입니다. 오직 이것만으로 이제 데이터에 따라 자동으로 컴포넌트가 재렌더링됩니다. `useState`를 호출할 필요도 없고, 복잡한 선택자나 고차컴포넌트를 사용하여 어플리케이션 상태를 가져올 방법을 고민할 필요도 없습니다. 모든 컴포넌트들이 스마트해졌지만, 아직 답답하고 선언적인 규칙을 따라야합니다.
(역주: 예전에는 decorator를 많이 사용했으나, Mobx6부터는 decorator 사용을 지양합니다.)

*실행 버튼*을(없는 거 아시죠?) 눌러 코드가 어떻게 동작하는지 확인해봅시다. 미리보기는 수정해볼 수 있으니 마음껏 테스트해보십시오.  예제 코드에서 `observer`를 모두 지우거나, `TodoView`에만 남겨서 어떻게 되는 지도 확인해보세요. 미리보기에서 우측의 숫자들은 컴포넌트가 몇 번이나 렌더링 되었는 지 나타냅니다.

```javascript
const TodoList = observer(({store}) => {
  const onNewTodo = () => {
    store.addTodo(prompt('Enter a new todo:','coffee plz'));
  }

  return (
    <div>
      { store.report }
      <ul>
        { store.todos.map(
          (todo, idx) => <TodoView todo={ todo } key={ idx } />
        ) }
      </ul>
      { store.pendingRequests > 0 ? <marquee>Loading...</marquee> : null }
      <button onClick={ onNewTodo }>New Todo</button>
      <small> (double-click a todo to edit)</small>
      <RenderCounter />
    </div>
  );
})

const TodoView = observer(({todo}) => {
  const onToggleCompleted = () => {
    todo.completed = !todo.completed;
  }

  const onRename = () => {
    todo.task = prompt('Task name', todo.task) || todo.task;
  }

  return (
    <li onDoubleClick={ onRename }>
      <input
        type='checkbox'
        checked={ todo.completed }
        onChange={ onToggleCompleted }
      />
      { todo.task }
      { todo.assignee
        ? <small>{ todo.assignee.name }</small>
        : null
      }
      <RenderCounter />
    </li>
  );
})

ReactDOM.render(
  <TodoList store={ observableTodoStore } />,
  document.getElementById('reactjs-app')
);
```

다음 예제는 데이터만 변경하면 나머지는 자동으로 처리된다는 것을 보여줍니다. Mobx가 저장소의 상태에 따라 연관된 부분이나 UI를 자동적으로 업데이트해줍니다.

```javascript
const store = observableTodoStore;
store.todos[0].completed = !store.todos[0].completed;
store.todos[1].task = "Random todo " + Math.random();
store.todos.push({ task: "Find a fine cheese", completed: true });
// etc etc.. add your own statements here...
```