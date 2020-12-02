---
title: "[번역] Mobx와 React를 위한 10분 안내서 (5)"
date: "2020-12-01T16:38:48.895+09:00"
description: "Mobx와 React를 위한 10분 안내서"
---

# 참조 사용하기

지금까지 관찰가능한(observable) 객체(프로토 타입 기반과 일반 객체), 배열, 요소들을 만들었습니다. 아마도 여러분은 Mobx가 어떻게 참조를 다루는 지, 내 상태로 그래프를 그릴 수 있는 지 궁금할 것입니다. 이전 예제에 todo에 `assignee` 프로퍼티가 있는 걸 보았을 것입니다. 이제 이 프로퍼티에 값을 추가해봅시다. people(person)을 포함하는 새로운 저장소를 만들고, 거기에 todo을 부여할 것입니다.

```javascript
const peopleStore = observable([
  { name: "Michel" },
  { name: "Me" }
]);
observableTodoStore.todos[0].assignee = peopleStore[0];
observableTodoStore.todos[1].assignee = peopleStore[1];
peopleStore[0].name = "Michel Weststrate";
```

이제 두 개의 독립적인 저장소가 있습니다. 하나는 people에 대한 것이고, 하나는 todo에 대한 것입니다. `assignee`를 people 저장소의 person에게 할당하기 위해서, 참조를 할당했습니다. 이 수정사항들은 TodoView에 자동으로 반영될 것입니다. Mobx에서는 데이터 표준화를 먼저할 필요가 없으며, 컴포넌트를 업데이트하기 위해 선택자(selector)를 쓸 필요도 없습니다. 심지어 데이터가 어디에 있는 지도 중요하지 않습니다. 객체가 관찰가능하게 만들어졌다면, 그 객체가 변경되거나 파생되면 Mobx가 자동으로 추적합니다. 아래 입력창에 아무 이름이나 입력해서 테스트해보세요(그 전에 실행버튼을 꼭 누르세요!). (역주: 이번에도 입력창과 실행버튼은 본문에만 있습니다.)

위 입력창의 HTML은 아주 단순합니다:
```HTML
<input onkeyup="peopleStore[1].name = event.target.value" />
```

# 비동기 처리

이 Todo 어플리케이션의 모든 것들은 상태에서 파생되기 때문에, 상태가 *언제* 변경되는 지는 중요하지 않습니다. 이것은 비동기 처리를 매우 직관적으로 만듭니다. 아래 버튼을 누르면 새로운 todo 아이템을 비동기적으로 불러옵니다. 여러 번 눌러서 테스트해보세요.

아래의 코드는 매우 직관적입니다. 먼저 현재 로딩 상태를 표현하는 UI를 만들기위해 저장소의 `pendingRequests` 프로퍼티를 업데이트했습니다. 로딩이 완료되면, 저장소의 todos를 업데이트하고 `pendingRequests`의 카운터를 감소시킵니다. 이 코드와 이전 코드의 `TodoList`를 비교해보면 pendingRequests가 어떻게 쓰이는 지 볼 수 있습니다.

코드에서 `action`을 `setTimeout`으로 감싸고 있습니다. 이는 꼭 필요한 것은 아니지만, 두 변경사항이 완료되었을 때를 observer가 감지하여, 한 번의 트랜젝션(transaction)으로 두 변경사항이 처리되도록 합니다.

```javascript
observableTodoStore.pendingRequests++;
setTimeout(action(() => {
  observableTodoStore.addTodo('Random Todo ' + Math.random());
  observableTodoStore.pendingRequests--;
}), 2000);
```

# 결론

됐습니다. 보일러 플레이트는 필요 없습니다. UI를 만들기 위해 단순하고 선언적인 컴포넌트만 있으면 됩니다. 그리고 그 컴포넌트는 어플리케이션 상태에 대해 완벽하게 반응형입니다. 이제 `mobx`와 `mobx-react-lite` 패키지를 사용할 준비가 되었습니다. 지금까지 배운 것들에 대해 간단히 요약해보겠습니다.

1. Mobx가 상태를 추적할 수 있도록 `makeObservable` 또는 `makeAutoObservable`을 사용합니다.
2. `observable`은 변경될 수 있는 관찰가능한 상태입니다.
3. `computed`는 변경되는 상태나 캐시에 따라 계산된 값을 반환합니다.
4. `autorun`은 관찰가능한 상태에 따라 자동적으로 실행되는 함수입니다. 로그를 남기거나, 네트워크 요청을 하는 등의 동작에 유용합니다.

위의 수정가능한 예제 코드들로 더 연습하여, Mobx가 변경사항에 대해 어떻게 반응하는 지에 대해 익숙해져보세요. 예를들어 `report` 함수가 언제 호출되는 지 확인할 수 있는 로그를 추가해볼 수 있습니다. 또는 `report`가 더이상 보이지 않도록 만들고 `TodoList`가 렌더링되는데 어떤 영향을 미치는 지 볼 수도 있습니다. 이 외에도 여러가지를 시도해보세요.