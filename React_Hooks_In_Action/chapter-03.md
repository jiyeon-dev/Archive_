# 3장 useReducer 훅을 사용해 컴포넌트 상태 관리하기

## 핵심 요약

- 여러 상태 값을 함께 갱신해야 하는 경우 `useReducer`를 사용해 상태 관리 로직을 한곳에 모아 관리할 수 있다.
- `useReducer`는 상태 변경을 직접 수행하는 대신 액션을 디스패치하고, 리듀서가 액션에 따라 새 상태를 계산하게 한다.
- 상태 전환 규칙이 복잡해질수록 `useState` 여러 개보다 `useReducer`가 더 명확하다.

## 3.1 단일 이벤트에 대한 응답으로 여러 상태 값 갱신하기

- 하나의 이벤트가 여러 상태 값에 영향을 줄 때, 각각의 상태를 따로 갱신하면 UI가 예측하기 어려워질 수 있다.
- 서로 관련된 상태 값은 하나의 상태 객체로 묶고, 변경 규칙을 리듀서에 모아두면 흐름을 이해하기 쉬워진다.

### 3.1.1 예측할 수 없는 상태 변경으로 사용자 방해하기

- 상태 값이 서로 의존하는데 따로 관리되면, 한 상태는 바뀌었지만 다른 상태는 이전 값을 가리키는 순간이 생길 수 있다.
- 예를 들어 그룹을 변경했는데 선택된 항목 인덱스가 이전 그룹 기준으로 남아 있으면 잘못된 항목이 선택될 수 있다.

### 3.1.2 예측 가능한 상태 변경으로 사용자의 집중력 유지하기

- 관련 상태를 함께 갱신하면 사용자가 보고 있는 UI의 흐름이 자연스럽게 유지된다.
- `useReducer`를 사용하면 “무슨 일이 일어났는지”를 액션으로 표현하고, 그 결과 상태가 어떻게 바뀌는지는 리듀서에서 관리할 수 있다.

## 3.2 useReducer로 더 복잡한 상태 관리하기

### 3.2.1 미리 정의된 액션과 리듀서를 사용해 상태 갱신하기

- 액션은 상태 변경을 일으키는 사건을 표현한다.
- 리듀서는 이전 상태와 액션을 받아 새 상태를 반환하는 함수이다.
- 일반적으로 액션은 `type`과 `payload`를 가진 객체로 작성한다.

```jsx
dispatch({ type: 'SET_GROUP', payload: 'Rooms' });
```

### 3.2.2 BookablesList 컴포넌트를 위한 리듀서 만들기

```jsx
export default function reducer(state, action) {
    switch (action.type) {
        case 'SET_GROUP':
            return { ...state, group: action.payload, bookableIndex: 0 };
        case 'SET_BOOKABLE':
            return { ...state, bookableIndex: action.payload };
        case 'TOGGLE_HAS_DETAILS':
            return { ...state, hasDetails: !state.hasDetails };
        case 'NEXT_BOOKABLE':
            const count = state.bookables.filter(b => b.group === state.group).length;
            return { ...state, bookableIndex: (state.bookableIndex + 1) % count };
        default:
            return state;
    }
}
```

- `SET_GROUP`: 그룹을 변경하고 선택된 항목 인덱스를 `0`으로 초기화한다.
- `SET_BOOKABLE`: 선택된 항목 인덱스를 변경한다.
- `TOGGLE_HAS_DETAILS`: 상세 정보 표시 여부를 반대로 바꾼다.
- `NEXT_BOOKABLE`: 현재 그룹 안에서 다음 항목으로 이동한다.

### 3.2.3 useReducer를 사용해 컴포넌트 상태에 접근하고 액션 디스패치하기

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

- `state`: 현재 상태 값
- `dispatch`: 액션을 리듀서에게 전달하는 함수
- `reducer`: 이전 상태와 액션으로 새 상태를 만드는 함수
- `initialState`: 컴포넌트가 처음 렌더링될 때 사용할 초기 상태

```jsx
const initialState = {
    group: 'Rooms',
    bookableIndex: 0,
    hasDetails: true,
    bookables
};

export default function BookablesList() {
    const [state, dispatch] = useReducer(reducer, initialState);
    const { group, bookableIndex, bookables, hasDetails } = state;

    function changeGroup(e) {
        dispatch({ type: 'SET_GROUP', payload: e.target.value });
    }
}
```

- 컴포넌트는 상태를 직접 바꾸지 않고 `dispatch`를 통해 액션을 전달한다.
- 리액트는 같은 컴포넌트 호출 사이에서 동일한 `dispatch` 함수를 유지한다.
- `dispatch` 함수가 안정적이기 때문에 불필요한 재렌더링을 줄이는 데 도움이 된다.

## 3.3 함수를 사용해 초기 상태 생성하기

- 초기 상태를 만드는 과정이 복잡하거나 비용이 크다면 `useReducer`의 세 번째 인자로 초기화 함수를 전달할 수 있다.

```jsx
const [state, dispatch] = useReducer(reducer, initArg, initFn);
```

- `initArg(초기화 인자)`: 초기화 함수에 전달할 값
- `initFn(초기화 함수)`: 초기화 인자를 사용해 초기 상태를 생성하는 함수
- 초기화 함수는 최초 렌더링 시 한 번만 실행된다.

### 3.3.1 WeekPicker 컴포넌트 소개

- `WeekPicker`는 현재 선택된 주를 보여주고, 이전 주/다음 주/오늘로 이동할 수 있게 하는 컴포넌트이다.
- 주 정보는 기준 날짜, 시작일, 종료일처럼 서로 관련된 값으로 구성되므로 리듀서로 관리하기 좋다.

### 3.3.2 날짜와 주를 처리하는 유틸리티 함수 만들기

- `getWeek` 함수는 날짜를 받아 해당 날짜가 포함된 주의 정보를 계산한다.
- 날짜 계산 로직을 컴포넌트 밖으로 분리하면 컴포넌트는 UI와 액션 디스패치에 집중할 수 있다.

### 3.3.3 컴포넌트의 날짜를 관리하는 리듀서 만들기

```jsx
export default function reducer(state, action) {
    switch (action.type) {
        case 'NEXT_WEEK':
            return getWeek(state.date, 7);
        case 'PREV_WEEK':
            return getWeek(state.date, -7);
        case 'TODAY':
            return getWeek(new Date());
        case 'SET_DATE':
            return getWeek(new Date(action.payload));
        default:
            throw new Error(`Unknown action type: ${action.type}`);
    }
}
```

- 날짜 리듀서는 액션에 따라 기준 날짜를 바꾸고, `getWeek`를 호출해 새로운 주 상태를 만든다.
- 알 수 없는 액션 타입은 기존 상태를 그대로 반환하거나 예외를 던지도록 처리할 수 있다.

### 3.3.4 useReducer 훅에게 초기화 함수 전달하기

```jsx
const [week, dispatch] = useReducer(reducer, date, getWeek);
```

- `getWeek`는 최초 렌더링 시 한 번만 실행되어 초기 주 상태를 만든다.
- 이후에는 `dispatch`로 전달한 액션에 따라 리듀서가 새 주 상태를 계산한다.

### 3.3.5 WeekPicker를 사용하도록 BookingsPage 변경하기

- `BookingsPage`에서 직접 날짜 상태를 관리하는 대신 `WeekPicker`에게 주 선택 로직을 맡길 수 있다.
- 이렇게 하면 페이지 컴포넌트는 전체 화면 구성에 집중하고, 날짜 변경 로직은 별도 컴포넌트와 리듀서에 분리된다.

## 3.4 useReducer 개념 다시 살펴보기

- `useReducer`는 상태 값과 상태 변경 규칙을 함께 이해해야 할 때 유용하다.
- 액션은 “무슨 일이 일어났는가”를 설명하고, 리듀서는 “그 일 때문에 상태가 어떻게 바뀌는가”를 결정한다.
- 리듀서는 순수 함수로 작성하는 것이 좋다. 같은 상태와 같은 액션을 받으면 항상 같은 새 상태를 반환해야 한다.

## 3.5 요약

- 관련된 상태 값이 함께 변경된다면 `useReducer`를 고려한다.
- 컴포넌트는 `dispatch`로 액션을 전달하고, 리듀서는 액션에 따라 새 상태를 반환한다.
- 액션 타입을 명확하게 정의하면 상태 변경 흐름을 추적하기 쉬워진다.
- 초기 상태 계산이 복잡하면 초기화 함수를 사용해 최초 렌더링 때만 계산하도록 만들 수 있다.

