---
title: "[번역] Mobx와 React를 위한 10분 안내서 (2)"
date: "2020-11-26T15:34:51.806+09:00"
description: "Mobx와 React를 위한 10분 안내서"
---

# 간단한 Todo 저장소

이론은 이제 충분합니다. 이론을 글로 읽는 것보다, 동작하는 것을 직접 보는 것이 더 좋을 것입니다. 그러기 위해 매우 간단한 todo 저장소(store)를 만들어보겠습니다. 아래의 예제 코드들은 직접 수정해볼 수 있고, *코드 실행* 버튼들로 바로 실행해볼 수 있습니다(여기서는 안 되고, 본문에서는 됩니다). 아래 코드는 todo 목록을 제공하는 매우 단순한 `TodoStore`입니다. 아직 Mobx가 적용되지 않았습니다.

```javascript
class TodoStore {
  todos = [];

  get completedTodosCount() {
    return this.todos.filter(
      todo => todo.completed === true
    ).length;
  }

  report() {
    if (this.todos.length === 0)
      return "<none>";
    const nextTodo = this.todos.find(todo => todo.completed === false);
    return `Next todo: "${nextTodo ? nextTodo.task : "<none>"}". ` +
      `Progress: ${this.completedTodosCount}/${this.todos.length}`;
  }

  addTodo(task) {
    this.todos.push({
      task: task,
      completed: false,
      assignee: null
    });
  }
}

const todoStore = new TodoStore();
```

`todos` 목록이 있는 `TodoStore` 인스턴스를 만들었습니다. 이제 todoStore에 몇 객체들을 채워봅시다. 변경 동작에 따른 결과를 확인하기 위해서 추가하거나, 각 동작 후에 `todoStore.report`를 호출하고 로그를 남기겠습니다. 현재 report는 의도적으로 첫 번째 todo만 출력하도록 되어있습니다. 이것 때문에 예제를 조금 어색해보일 수 있지만, 이러한 형태가 나중에 Mobx의 의존성 추적을 한 눈에 보기에 훨씬 적합합니다.


```javascript
todoStore.addTodo("read MobX tutorial");
console.log(todoStore.report());

todoStore.addTodo("try MobX");
console.log(todoStore.report());

todoStore.todos[0].completed = true;
console.log(todoStore.report());

todoStore.todos[1].task = "try MobX in own project";
console.log(todoStore.report());

todoStore.todos[0].task = "grok MobX tutorial";
console.log(todoStore.report());
```

결과:
```
Next todo: "read MobX tutorial". Progress: 0/1
Next todo: "read MobX tutorial". Progress: 0/2
Next todo: "try MobX". Progress: 1/2
Next todo: "try MobX in own project". Progress: 1/2
Next todo: "try MobX in own project". Progress: 1/2
```