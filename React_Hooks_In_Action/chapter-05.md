# 5장 useRef 훅으로 컴포넌트 상태 관리하기

## 핵심 요약

- `useRef`는 렌더링 사이에 값을 유지하는 참조 객체를 만든다.
- 참조 객체의 `current` 값을 바꿔도 컴포넌트는 재렌더링되지 않는다.
- 화면에 표시되는 값은 `useState`나 `useReducer`로 관리하고, 렌더링에 직접 필요하지 않은 값은 `useRef`로 관리할 수 있다.
- 책 예제에서는 예약 가능 자원을 자동으로 넘기는 타이머 ID, Next 버튼 DOM, 날짜 입력 텍스트 박스를 저장하기 위해 `useRef`를 사용한다.

## 5.1 재렌더링을 촉발하지 않고 상태를 갱신하는 방법

- 리액트 상태를 변경하면 컴포넌트가 다시 렌더링된다.
- 하지만 모든 값 변경이 UI 업데이트를 필요로 하지는 않는다.
- `useRef`를 사용하면 *UI 재렌더링 없이 값을 갱신할 수 있다.*

### 5.1.1 상태 값을 갱신할 때 useState와 useRef 비교

- `useState`로 관리하는 값은 갱신 함수를 호출할 때마다 리액트가 새 렌더링을 예약한다.
  - 렌더링 사이에 상태를 유지하고, 매번 상태 값을 컴포넌트에 대입한다.
- `useRef`로 만든 참조 객체는 `current` 값을 바꿔도 렌더링을 예약하지 않는다.
  - 값을 저장하며 렌더링 사이에 값을 유지하고, 매번 같은 참조 객체를 컴포넌트에 전달해 ref 변수에 대입한다.

```jsx
const [count, setCount] = useState(1);
const ref = useRef(1);

const incCount = () => setCount(c => c + 1);
const incRef = () => ref.current++;
```

### 5.1.2 useRef 호출하기

```jsx
const ref = useRef(initialValue);
```

- `useRef` 함수는 `current` 프로퍼티를 가진 객체를 반환하며, 같은 컴포넌트 인스턴스 안에서는 항상 동일한 참조 객체를 반환한다.
- 참조 객체의 `current` 프로퍼티에 새 값을 대입해도 재렌더링이 발생하지 않는다. 하지만 리액트는 동일한 참조 객체를 유지하기 때문에 컴포넌트가 다시 실행될 때 이전에 대입한 값을 사용할 수 있다.

## 5.2 참조객체를 사용해 타이머 ID 저장하기

- `0501-timer-ref` 예제에서는 `BookablesList`가 예약 가능 자원을 3초마다 자동으로 다음 항목으로 넘긴다.
- `setInterval`이 반환한 타이머 ID는 화면에 표시할 값이 아니지만, `clearInterval`을 호출하려면 기억해둬야 한다.
- 이때 `timerRef`를 사용한다.

```jsx
const timerRef = useRef(null);

useEffect(() => {
    timerRef.current = setInterval(() => {
        dispatch({ type: "NEXT_BOOKABLE" });
    }, 3000);

    return stopPresentation;
}, []);

function stopPresentation() {
    clearInterval(timerRef.current);
}

<button
    className="btn"
    onClick={stopPresentation}
>
    Stop
</button>
```

- `setInterval`은 3초마다 `NEXT_BOOKABLE` 액션을 디스패치한다.
- 리듀서는 `NEXT_BOOKABLE` 액션을 받아 현재 그룹 안에서 다음 예약 가능 자원으로 `bookableIndex`를 변경한다.
- `timerRef.current`에는 타이머 ID가 저장된다.
- cleanup 함수로 `stopPresentation`을 반환하기 때문에 컴포넌트가 언마운트될 때 타이머가 정리된다.
- 사용자가 Stop 버튼을 누를 때도 같은 `stopPresentation` 함수를 호출해 자동 전환을 멈출 수 있다.
- 타이머 ID를 `useState`에 저장하면 ID가 바뀔 때 불필요한 렌더링이 발생한다.
- 타이머 ID는 UI를 구성하는 값이 아니므로 `useRef`에 저장하는 편이 더 적절하다.

## 5.3 DOM 엘리먼트에 대한 참조 유지하기

- `useRef`를 사용하면 일반적인 리액트의 선언적 흐름을 우회해 DOM 엘리먼트와 직접 상호작용할 수 있다.
- JSX 엘리먼트의 `ref` 프로퍼티에 참조 객체를 넘기면, 리액트가 실제 DOM 엘리먼트를 `current`에 저장한다.

### 5.3.1 이벤트에 응답해 엘리먼트에 포커스 설정하기

- 예제에서는 사용자가 목록에서 예약 가능 자원을 선택하면 다시 Next 버튼에 포커스를 준다.

```jsx
const nextButtonRef = useRef();

function changeBookable(selectedIndex) {
    dispatch({
        type: "SET_BOOKABLE",
        payload: selectedIndex
    });

    nextButtonRef.current.focus();  // 참조 객체를 사용해 포커스를 맞춤
}

<button
    className="btn"
    onClick={nextBookable}
    ref={nextButtonRef}   // JSX ref 속성에 nextButtonRef 지정
    autoFocus
>
    <FaArrowRight />
    <span>Next</span>
</button>
```

- 참조 객체 `nextButtonRef`를 생성해 Next 버튼 엘리먼트에 대한 참조를 보관한다.
- `dispatch`로 선택된 예약 가능 자원을 바꾼다.
- 이어서 `nextButtonRef.current.focus()`를 호출해 Next 버튼에 포커스를 준다.
- DOM 엘리먼트에 직접 명령해야 하는 작업에는 ref가 잘 맞는다.

### 5.3.2 참조객체를 사용해 텍스트 박스 관리하기

- 예제에서는 `WeekPicker`에 날짜를 직접 입력할 수 있는 텍스트 박스와 Go 버튼이 추가된다.
- 텍스트 박스의 값을 매 입력마다 상태로 관리하지 않고, Go 버튼을 누르는 순간 ref에서 현재 값을 읽는다.

#### 비제어 컴포넌트

```jsx
const textboxRef = useRef();

function goToDate() {
    dispatch({
        type: "SET_DATE",
        payload: textboxRef.current.value
    });
}

<input
    type="text"
    ref={textboxRef}
    placeholder="e.g. 2020-09-02"
    defaultValue="2020-06-24"
/>

<button
    className="go btn"
    onClick={goToDate}
>
    <FaCalendarCheck />
    <span>Go</span>
</button>
```

- `textboxRef.current`는 텍스트 박스에 대한 참조를 저장하고, `textboxRef.current.value`는 입력된 텍스트를 나타낸다.
- 사용자가 Go 버튼을 누를 경우에만 `textboxRef.current.value`를 읽고 리듀서에게 전달한다.
- 이렇게 DOM이 상태를 관리하도록 내버려둔 컴포넌트를 "비제어 컴포넌트(uncontrolled component)"라고 한다.

#### 제어 컴포넌트

```jsx
const [dateText, setDateText] = useState("2020-06-24");

function goToDate() {
    dispatch({
        type: "SET_DATE",
        payload: dateText
    });
}

return (
    <div>
        <input
            type="text"
            value={dateText}
            onChange={e => setDateText(e.target.value)}
        />
    </div>
);
```

- 사용자가 텍스트 박스에 입력할 때마다 `dateText` 상태가 갱신된다.
- 리액트는 상태를 관리할 때 더 많은 도움을 제공할 수 있는 "제어 컴포넌트(controlled component)"를 사용하는 쪽을 권장한다.
- 더 이상 ref 참조가 필요 없어지며, 리액트의 표준 접근 방식과 일치하게 된다.

## 5.4 요약

- `useRef`는 값을 관리하되 값 변경 시 재렌더링을 피하고 싶은 경우 사용한다.
- 같은 `useRef`에 대해서는 항상 동일한 참조 객체가 반환된다.
- DOM 엘리먼트와 상호작용하기 위해 참조 객체를 사용할 수 있다.
  - JSX에서 `ref`에 참조 객체 변수를 대입하면 된다.
  - 이를 사용해 비제어 컴포넌트의 상태를 읽거나 변경할 수 있다.
- 리액트에서는 가능하면 "제어 컴포넌트"를 사용하는 편이 낫다.

## 내 생각

-
