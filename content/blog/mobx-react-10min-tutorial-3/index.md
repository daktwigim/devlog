---
title: "[번역] Mobx와 React를 위한 10분 안내서 (3)"
date: "2020-11-27T14:27:56.152+09:00"
description: "Mobx와 React를 위한 10분 안내서"
---

# 반응형으로 만들기

지금까지의 코드에는 특별한 점이 없었습니다. 하지만 `report`를 매번 명시적으로 호출할 필요가 없어지고, 대신에 상태가 변경될 때마다 호출되도록 선언할 수 있다면 어떨까요? 코드에서 `report`가 필요할 때마다 호출해야하는 번거로움이 없어질 것입니다. 최신 report가 출력되는지 확인하고 싶지만, 그건 굉장히 번거로울 수 있습니다.

다행히도 Mobx가 하는 일이 바로 그것입니다. 상태에 의존적인 코드를 자동으로 실행합니다. 그렇게 `report` 함수가 자동적으로 업데이트 되도록합니다, 마치 스프레드 시트의 차트처럼 말이죠. 그러기위해서는 `TodoStore`를 *관찰가능((observable)*하게 만들어 Mobx가 모든 변경사항을 추적할 수 있도록 만들어야합니다. 그렇게 되도록 클래스를 변경해봅시다.

또한 `completedTodosCount` 속성(property)도 todo 목록에 따라 자동적으로 값을 얻게 됩니다. `observable`와 `computed` 어노테이션(annotation)을 사용해서 객체에 관찰가능한 속성을 만들 수 있습니다. 아래 예제에서는 `makeObservable`을 사용하여 어노테이션을 명시했지만, `makeAutoObservable(this)`를 사용해서 더 간단하게 작성할 수도 있습니다.

```javascript
class ObservableTodoStore {
  todos = [];
  pendingRequests = 0;

  constructor() {
    makeObservable(this, {
      todos: observable,
      pendingRequests: observable,
      completedTodosCount: computed,
      report: computed,
      addTodo: action,
    });
    autorun(() => console.log(this.report));
  }

  get completedTodosCount() {
    return this.todos.filter(
      todo => todo.completed === true
    ).length;
  }

  get report() {
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

const observableTodoStore = new ObservableTodoStore();
```

됐습니다! `observable`을 사용하여 몇몇 속성들이 언제든지 변경될 수 있는 값이라는 걸 Mobx에게 알려주었습니다. 계산되는 속성(computation)들은 `computed`로 꾸며주어(decorate) 상태와 캐시로부터 파생(derive)될 수 있다고 알려주었습니다.(역주: 어노테이션(annotation)을 이용하여 명시하는 것을 꾸민다(decorate)고 표현합니다.)

`pendingRequests`와 `assignee`는 아직 사용하지 않지만, 나중에 사용합니다.

생성자(constructor)에서 report를 출력하는 함수를 만들고 `autorun`으로 감싸주었습니다. 자동실행(Autorun)은 최초에 한 번 실행되고, 함수 내에 있는 관찰가능한 값들이 변경될 때마다 자동적으로 재실행되는 *리액션(reaction)*을 만듭니다. 여기에서는 `report`가 관찰가능한 `todos` 속성을 사용하므로, 앞으로 report가 자동적으로 실행될 것입니다. 이것은 다음 동작의 결과를 보면 알 수 있습니다. 실행 버튼을 누르세요(여긴 버튼 없어요).

```javascript
observableTodoStore.addTodo("read MobX tutorial");
observableTodoStore.addTodo("try MobX");
observableTodoStore.todos[0].completed = true;
observableTodoStore.todos[1].task = "try MobX in own project";
observableTodoStore.todos[0].task = "grok MobX tutorial";
```

결과:
```
Next todo: "read MobX tutorial". Progress: 0/1
Next todo: "read MobX tutorial". Progress: 0/2
Next todo: "try MobX". Progress: 1/2
Next todo: "try MobX in own project". Progress: 1/2
```

괜찮지 않나요? `report`가 누락되는 값 없이 자동적, 동기적으로 실행되었습니다. 로그를 자세히 보면, 동작의 5번째 줄에 대한 로그가 없습니다. 왜냐하면 두번째 todo의 이름이 변경되기는 했지만 그 변경사항이 report에 직접적인 영향을 미치지 않았기 때문입니다.(역주: 5번째 줄을 실행한 후에 report를 출력해보면, 결과의 4번째 줄과 동일합니다.) 반면에 첫 todo의 이름이 변경되면 report는 영향을 받습니다. 단순히 `todos`가 변경되거나 그 안의 속성이 변경된다고 해서 `autorun`이 실행되지 않는다는 말입니다.