# 2장 useState 훅으로 컴포넌트 상태 관리하기

## 핵심 요약

- `useState`는 컴포넌트 내부에서만 필요한 상태를 간단하게 관리할 수 있게 해준다.

## 2.1 예약 관리 앱 설정하기

- `jrlarsen/react-hooks-in-action` 저장소의 `0201-pages` 브랜치를 내려받아 프로젝트를 설정한다.

## 2.2 useState를 사용해 값을 저장하고 사용하며 설정하기

- `useState` 훅은 컴포넌트 호출과 호출 사이에 상태를 유지하고, 상태 변경을 리액트에 알리는 가장 간단한 방법이다.
- 이 훅을 호출하면 최신 상태 값과 그 값을 변경할 때 사용할 수 있는 갱신 함수(updater function)를 함께 반환한다.

### 2.2.2 useState를 호출하면 값과 갱신 함수를 돌려받는다

```jsx
const [value, setValue] = useState(initialValue);
const [selectedRoom, setSelectedRoom] = useState(initialValue);
```

### 2.2.3 갱신 함수를 호출하면 이전 값이 치환된다

- `useState`의 갱신 함수는 이전 상태를 병합하지 않고 새 상태로 완전히 치환한다.

```jsx
function BookablesList() {
    const [state, setState] = useState({ bookableIndex: 1, group: 'Rooms' });
}

function handleClick(index) {
    setState({ bookableIndex: index });  // <- group 값이 제거된다.
}
```

- 따라서 *객체를 사용하는 경우, 이전 객체의 모든 프로퍼티를 복사*해야 한다.

```jsx
function handleClick(index) {
    setState(state => ({ ...state, bookableIndex: index }));
}
```

### 2.2.4 초깃값으로 함수를 useState에 전달하기

- 초기 상태 값을 생성하기 위해 비용이 많이 드는 계산이 필요한 경우, 계산 결과를 바로 전달하면 렌더링될 때마다 매번 계산이 실행된다.

```jsx
function untangle(aFrayedKnot) {
    // 값비싼 데이터 해석 절차를 수행함
    return nugget;
}

function ShinyComponent({ tangledWeb }) {
    const [shiny, setShiny] = useState(untangle(tangledWeb));  // 렌더링될 때마다 계산됨.
}
```

- `지연 계산 초기 상태(lazy initial state)`를 사용하면 리액트는 함수를 최초 렌더링 시 한 번만 호출하고, 그 반환값을 초기 상태로 사용한다.

```jsx
function ShinyComponent({ tangledWeb }) {
    const [shiny, setShiny] = useState(() => untangle(tangledWeb));  // 컴포넌트가 맨 처음 렌더링될 때 한 번만 호출됨.
}
```

### 2.2.5 새 상태를 설정할 때 이전 상태 참조하기

- 갱신 함수를 호출할 때 이전 값을 바탕으로 새로운 상태 값을 계산할 수 있다.

```jsx
// 갱신 함수(이전 값 => 설정 값);
setBookableIndex(oldValue => oldValue + 1);
```

## 2.3 여러 값을 다루기 위해 useState를 여러 번 호출하기

- 가장 쉬운 방법은 `useState`를 여러 번 호출하는 것이다.
- 리액트는 훅의 호출 순서에 따라 각 상태 값을 연결한다.
- 상태 전환이 복잡해지면 `useReducer`를 통해 더 쉽게 관리할 수 있다.

## 2.4 함수 컴포넌트 개념 다시 살펴보기

- 컴포넌트는 props를 받아 UI에 대한 서술을 반환하는 함수이다.
- 리액트는 컴포넌트를 호출한다. 컴포넌트는 함수이므로 자신의 코드를 실행한 다음 종료된다. 이것이 초기 렌더링이다.
- 이벤트 핸들러가 만든 클로저 안에는 변수를 유지할 수 있다. 다른 지역 변수들은 함수 실행이 끝나면 사라진다.
- 훅을 사용해서 리액트에게 값 관리를 위임할 수 있다. 리액트는 최신 값과 그 값을 갱신하는 함수를 컴포넌트에게 돌려준다.
- 갱신 함수를 사용하면 리액트에게 값이 변경됐음을 알릴 수 있고, 리액트는 새로운 UI 서술을 얻기 위해 컴포넌트를 다시 실행할 수 있다. 이것이 재렌더링이다.
