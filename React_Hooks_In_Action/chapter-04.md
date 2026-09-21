# 4장 부수 효과 활용하기

## 핵심 요약

- 어떤 방식으로든 외부 세계에 영향을 미치는 동작을 "부수 효과"라 한다.
  - 페이지 제목을 명령형 방식으로 설정하기
  - setInterval이나 setTimeout 같은 타이머 작업
  - DOM에서 엘리먼트의 너비, 높이, 위치 측정하기
  - 콘솔이나 다른 서비스에 로그 남기기
  - 지역 저장소에 값을 기록하거나 읽어오기
  - 서비스에서 데이터를 읽어오거나 서비스를 구독하거나 구독 취소하기
- `useEffect`는 렌더링이 끝난 뒤 부수 효과를 실행하게 해주는 훅이다.
- 의존성 배열을 사용하면 효과가 언제 다시 실행될지 제어할 수 있다.
- 효과 안에서 구독, 타이머, 이벤트 리스너처럼 정리가 필요한 작업을 했다면 cleanup 함수를 반환해야 한다.
- 데이터 읽어오기는 로딩, 성공, 실패 상태를 함께 관리해야 UI 흐름이 안정적이다.

## 4.1 간단한 예제를 통해 useEffect API 탐색하기

- 함수 컴포넌트의 본문은 렌더링 중에 실행된다.
- 반면 `useEffect`에 전달한 함수는 렌더링 결과가 화면에 반영된 뒤 실행된다.
- 따라서 DOM 업데이트 이후에 처리해야 하는 작업이나 외부 시스템과의 동기화는 `useEffect` 안에서 다루는 것이 좋다.

### 4.1.1 매번 렌더링이 일어난 다음에 부수 효과 실행하기

```jsx
useEffect(() => {  /* 부수 효과를 실행 */
    document.title = `Count: ${count}`;
});
```

- *의존성 배열을 생략하면 컴포넌트가 렌더링될 때마다 효과가 실행된다.*
- 렌더링 결과와 외부 상태를 항상 동기화해야 할 때 사용할 수 있다.
- 하지만 필요 이상으로 자주 실행될 수 있으므로 실제로는 의존성 배열을 명시하는 경우가 많다.

### 4.1.2 컴포넌트가 마운트될 때만 효과 실행하기

```jsx
useEffect(() => {
    console.log('mounted');
}, []);  // <- 의존 관계 인자로 빈 배열을 전달
```

- *빈 의존성 배열(`[]`)을 전달하면 효과는 컴포넌트가 처음 마운트된 뒤 한 번만 실행된다.*
- 초기 데이터 요청, 최초 설정, 외부 라이브러리 초기화처럼 한 번만 필요한 작업에 사용할 수 있다.

### 4.1.3 함수를 반환해서 부수 효과 정리하기

```jsx
const [size, setSize] = useState(getSize());

function getSize() {
    return { width: window.innerWidth, height: window.innerHeight };
}

useEffect(() => {
    function handleResize() {
        setSize(getSize());
    }

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);  // <- 정리 함수 반환
}, []);
```

- 반환 함수(cleanup 함수)를 실행하여 부수 효과를 정리할 수 있다.
- cleanup 함수는 컴포넌트가 언마운트되거나, 효과가 다시 실행되기 전에 호출된다.
- 타이머 제거, 이벤트 리스너 해제, 구독 취소처럼 정리가 필요한 작업은 cleanup에서 처리한다.

### 4.1.4 의존 관계를 지정해서 효과 실행 시기 제어하기

```jsx
useEffect(() => {
    document.title = selectedRoom;
}, [selectedRoom]);
```

- 의존성 배열에 값을 넣으면 그 값이 바뀔 때만 효과가 다시 실행된다.

### 4.1.5 useEffect 훅을 호출하는 방법 요약

```jsx
useEffect(effect);
useEffect(effect, []);
useEffect(effect, [dependency]);
```

- 두 번째 인자를 생략하면 매 렌더링 후 실행된다.
  - 모든 렌더링이 끝난 후 효과 함수를 실행한다.
- 빈 배열을 전달하면 마운트 후 한 번 실행된다.
  - 효과 안에서 상태를 갱신하면 그 상태 변경 때문에 다시 렌더링될 수 있다.
- 의존성 배열을 전달하면 배열 안의 값이 변경될 때 실행된다.
- cleanup 함수가 필요하면 effect 함수에서 함수를 반환한다.

### 4.1.6 useLayoutEffect를 호출해 브라우저가 화면을 다시 그리기 전에 효과를 실행하기

- `useEffect`는 브라우저가 화면을 그린 뒤 비동기적으로 실행된다.
- *`useLayoutEffect`는 DOM 변경 후, 브라우저가 화면을 그리기 전에 동기적으로 실행된다.*
- 일반적으로는 필요 없지만, 레이아웃 측정이나 화면 깜빡임을 막아야 하는 DOM 작업에는 `useLayoutEffect`가 필요할 수 있다.

## 4.2 데이터 읽어오기

### 4.2.1 새 db.json 파일 만들기

- 예제에서는 `json-server`가 읽을 수 있는 `db.json` 파일을 만들어 로컬 API처럼 사용한다.

### 4.2.2 JSON 서버 설정하기

- `json-server`를 실행하면 `db.json`에 있는 데이터를 HTTP 요청으로 읽어올 수 있다.

### 4.2.3 useEffect 훅 안에서 데이터를 읽어오기

```jsx
useEffect(() => {
    fetch("http://localhost:2001/users")  // 브라우저의 fetch API를 사용해 데이터베이스 요청을 만듦
        .then(response => response.json())  // 반환된 JSON 문자열을 자바스크립트 객체로 변환
        .then(data => setUsers(data));  // 적재한 사용자로 상태 갱신
}, []);
```

- 마운트될 때 한 번 실행되고 `useEffect` 안에서 데이터를 요청한다.
- 컴포넌트가 먼저 렌더링된 뒤 데이터 요청을 시작하는 방식을 '렌더링 시 읽기(fetch on render)'라고 한다.

### 4.2.4 async와 await 사용하기

잘못된 예:

```jsx
useEffect(async () => {
    const resp = await fetch("http://localhost:2001/users");
    const data = await (resp.json());
    setUsers(data);
}, []);
```

- `async` 함수는 기본적으로 Promise를 반환하기 때문에 효과 함수 자체를 `async`로 지정하면 안 된다.
  - 경합 조건(race condition)을 방지하기 위해 *효과 콜백은 동기적으로 처리*되어야 한다.
  - 비동기 함수는 효과 내부에 따로 정의해서 호출한다.

올바른 예:

```jsx
useEffect(() => {
    async function getUsers() {  // async 함수 정의
        const resp = await fetch("http://localhost:2001/users");
        const data = await (resp.json());
        setUsers(data);
    }
    getUsers();  // 비동기 함수 호출
}, []);
```

## 4.3 BookablesList 컴포넌트가 사용할 데이터 읽어오기

### 4.3.1 데이터 적재 과정 살펴보기

- 데이터 적재 과정은 보통 `요청 시작(로딩) -> 성공(결과) / 실패(오류 메시지)` 흐름으로 나뉜다.

### 4.3.2 적재 및 오류 상태를 관리하도록 리듀서 변경하기

```jsx
function reducer(state, action) {
    switch (action.type) {
        // .. 이전과 동일 ..

        case 'FETCH_BOOKABLES_REQUEST':  // 컴포넌트가 요청을 시작했다.
            return { ...state, isLoading: true, error: null, bookables: [] };
        case 'FETCH_BOOKABLES_SUCCESS':  // 서버에서 예약 가능 자원 정보가 도착했다.
            return { ...state, isLoading: false, bookables: action.payload };
        case 'FETCH_BOOKABLES_ERROR':  // 오류가 발생했다.
            return { ...state, isLoading: false, error: action.payload };
        default:
            return state;
    }
}
```

- 액션별로 관련된 상태 변경이 많을 때는 이런 변경을 그룹화하고 중앙화하기 위해 리듀서를 사용하면 도움이 된다.

### 4.3.3 데이터를 적재하기 위한 도우미 함수 만들기

```jsx
function getData(url) {
    return fetch(url)  // 브라우저의 fetch 함수에게 URL 전달
        .then(resp => {
            if (!resp.ok) {
                throw new Error('Failed to fetch data');  // 문제가 있으면 예외를 던짐
            }

            return resp.json();  // 응답받은 JSON 문자열을 자바스크립트 객체로 변환
        });
}
```

- 이 함수는 fetch에서 얻은 응답에 대한 전처리를 수행한다. 약간 미들웨어처럼 작동한다.
- fetch 호출과 응답 검증을 도우미 함수로 분리하면 컴포넌트 코드가 단순해진다.
- 요청 실패를 예외로 던지면 컴포넌트에서는 `try/catch`로 성공과 실패를 나누어 처리할 수 있다.

### 4.3.4 예약 가능 자원 적재하기

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
const { group, bookableIndex, bookables } = state;
const { hasDetails, isLoading, error } = state;

useEffect(() => {
    dispatch({ type: 'FETCH_BOOKABLES_REQUEST' });  // 데이터 읽기를 시작하기 위한 액션을 디스패치

    getData("http://localhost:3001/bookables")  // 데이터 읽기
        .then(bookables => dispatch({  // 적재한 예약 가능 자원 목록을 상태에 저장
            type: "FETCH_BOOKABLES_SUCCESS",
            payload: bookables
        }))
        .catch(error => dispatch({  // 오류를 사용해 상태 갱신
            type: "FETCH_BOOKABLES_ERROR",
            payload: error
        }));
}, []);

if (error) {
    return <p>{error.message}</p>;
}

if (isLoading) {
    return <p><Spinner /></p>;
}

return ( /* 예약 가능 자원과 세부정보 UI */ );
```

- 컴포넌트가 마운트되면 예약 가능 자원을 요청한다.
- 요청이 시작되면 로딩 상태로 바꾸고, 성공하면 데이터를 저장한다.
- 실패하면 오류 상태를 저장해서 사용자에게 실패 UI를 보여줄 수 있다.

## 4.4 요약

- 서로 다른 부수 효과는 별도의 `useEffect` 호출 안에 넣어라.
  - 각 부수 효과로 인한 영향을 쉽게 이해할 수 있다.
  - 서로 다른 의존 관계 배열을 사용해 부수 효과가 실행되는 시점을 쉽게 제어할 수 있다.
  - 효과를 커스텀 훅으로 더 쉽게 추출할 수 있다.
- 재렌더링 시 렌더링이 끝나고 실행해야 하는 효과가 여럿인 경우, 리액트는 먼저 이전 효과들의 정리 함수를 호출한 다음 새 효과를 실행한다.


## 내 생각

-
