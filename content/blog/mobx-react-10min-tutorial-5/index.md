---
title: "[번역] Mobx와 React를 위한 10분 안내서 (5)"
date: "2020-12-01T16:38:48.895+09:00"
description: "Mobx와 React를 위한 10분 안내서"
---

# Working with references

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