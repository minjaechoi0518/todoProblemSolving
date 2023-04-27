### [과제] 숙련주차 과제 답

14기 최민재

문제 내용

- 추가하기 버튼을 클릭해도 추가한 아이템이 화면에 표시되지 않음.
- 추가하기 버튼 클릭 후 기존에 존재하던 아이템들이 사라짐.
- 삭제 기능이 동작하지 않음.
- 상세 페이지에 진입 하였을 때 데이터가 업데이트 되지 않음.
- 완료된 카드의 상세 페이지에 진입 하였을 때 올바른 데이터를 불러오지 못함.
- 취소 버튼 클릭시 기능이 작동하지 않음.
- 과제를 마쳤다면 배포도 한번 해볼까요? < issue 생성해서 !! >

## 1번 문제

추가하기 버튼 클릭해도 추가하기가 작동되지 않음

```
(에러내용)
src/features/todos/components/Form.jsx
  Line 3:10:  'useDispatch' is defined but never used  no-unused-vars
  Line 5:10:  'addTodo' is defined but never used      no-unused-vars
  Line 8:9:   'id' is assigned a value but never used  no-unused-vars
```

### 문제 예상 dispatch함수가 잘 작동하고있지 않은 것 같고 추가하기 핸들러에도 문제가 있을 것 같다.

```
< 수정 전 기존 코드 >
// todo.js

// Todo를 추가하는 action creator
export const addTodo = (payload) => {
  return {
    type: ADD_TODO,
    payload,
  };
};

// Form.js

const Form = () => {
  const id = nextId();

  const [todo, setTodo] = useState({
    id: 0,
    title: "",
    body: "",
    isDone: false,
  });

  const onChangeHandler = (event) => {
    const { name, value } = event.target;
    setTodo({ ...todo, [name]: value });
  };

  const onSubmitHandler = (event) => {
    event.preventDefault();
    if (todo.title.trim() === "" || todo.body.trim() === "") return;

    setTodo({
      id: 0,
      title: "",
      body: "",
      isDone: false,
    });
  };

  return (
    <StAddForm onSubmit={onSubmitHandler}>
      <StInputGroup>
        <StFormLabel>제목</StFormLabel>
        <StAddInput
          type="text"
          name="title"
          value={todo.title}
          onChange={onChangeHandler}
        />
        <StFormLabel>내용</StFormLabel>

```

# 1번 문제해결

useDispatch 훅 사용 선언이 일단 빠져서 넣었고

```
const dispatch = useDispatch();
```

todo.js에 있는 addTodo 추가하기 액션크리에이터를 dispatch에 담아서
form.js의 추가하기 핸들러에 넣어준다. 그러면 새로운 todo data가 onSubmitHandler에 들어간다. 추가하기 버튼을 클릭하면 이제 새로운 데이터가 화면에 나타난다.

```
const onSubmitHandler = (event) => {
    event.preventDefault();
    if (todo.title.trim() === "" || todo.body.trim() === "") return;

    dispatch(addTodo(todo));
```

```
수정한 소스코드

...

const Form = () => {
  const id = nextId();

  const [todo, setTodo] = useState({
    id: 0,
    title: "",
    body: "",
    isDone: false,
  });
  // useDispatch 훅 사용해주고.
  const dispatch = useDispatch();

  const onChangeHandler = (event) => {
    const { name, value } = event.target;
    setTodo({ ...todo, [name]: value });
  };

  const onSubmitHandler = (event) => {
    event.preventDefault();
    if (todo.title.trim() === "" || todo.body.trim() === "") return;

    dispatch(addTodo(todo));

    setTodo({
      id: 0,
      title: "",
      body: "",
      isDone: false,
    });
  };

  return (
    //여기 onSubmitHandler 이 추가하기 핸들러가 제대로 작동하지 않는것같다.
    <StAddForm onSubmit={onSubmitHandler}>
      <StInputGroup>
        <StFormLabel>제목</StFormLabel>
        <StAddInput
...

```

---

## 2번문제

추가하기 버튼 클릭 후 기존에 존재하던 아이템들이 사라짐.

## 2번 문제해결

기존에 있던 todo data의 배열이나 객체에 추가되지 않는 것 같아서
카드들이 담겨져 있는 todos 리듀서 부분을 봤다.
기존의 코드는 새로운 값만을 가져와서 대체를 하는 코드였고
`…state.todos` 전개연산자를 사용해서 기존 배열안의 todo 객체들을 풀어주고 기존의 todo 배열에 새로운 값을 넣어서 리듀서에 새로운 배열을 추가했다.

```

<수정전>
const todos = (state = initialState, action) => {
  switch (action.type) {
    case ADD_TODO:
      return {
        ...state,
        todos: [action.payload],
      };

<수정후>
const todos = (state = initialState, action) => {
  switch (action.type) {
    case ADD_TODO:
      return {
        ...state,
        todos: [...state.todos, action.payload],
      };
    default:
      return state;
  }
}
```

# 3번문제

- 삭제 기능이 동작하지 않음.

기존코드

```
// todos.js

// Action value
const ADD_TODO = "ADD_TODO";
const GET_TODO_BY_ID = "GET_TODO_BY_ID";
const DELETE_TODO = "DELETE_TODO";
const TOGGLE_STATUS_TODO = "TOGGLE_STATUS_TODO";

// Action Creator
// Todo를 추가하는 action creator
export const addTodo = (payload) => {
  return {
    type: ADD_TODO,
    payload,
  };
};

// Todo를 지우는 action creator
export const deleteTodo = (payload) => {
  return {
    type: DELETE_TODO,
    payload,
  };
};

// Todo를 isDone를 변경하는 action creator
export const toggleStatusTodo = (payload) => {
  return {
    type: TOGGLE_STATUS_TODO,
    payload,
  };
};

// 상세 페이지에서 특정 Todo만 조회하는 action creator
export const getTodoByID = (payload) => {
  return {
    type: GET_TODO_BY_ID,
    payload,
  };
};

// initial state
const initialState = {
  todos: [
    {
      id: "1",
      title: "리액트",
  ...
};

const todos = (state = initialState, action) => {
  switch (action.type) {
    case ADD_TODO:
      return {
        ...state,
        todos: [...state.todos, action.payload],
      };

    case TOGGLE_STATUS_TODO:
      return {
        ...state,
       ...

    case GET_TODO_BY_ID:
      return {
        ...state,
     ...
```

# 3번 문제해결

DELETE_TODO 의 action value와 action creator는 있는데 리듀서에서는 아예 빠져있다. 리듀서에 삭제하기 case 를 만들고
내가 클릭한 payload의 id값과 일치하지 않는 모든 todo를 반환하는 filter 함수를 통해 삭제 상태를 업데이트 하였다.

<todos.js 수정후>

```
case DELETE_TODO:
  return {
    ...state,
    todos: state.todos.filter((todo) => todo.id !== action.payload),
  };
```

---

# 4번 문제

- 상세 페이지에 진입 하였을 때 데이터가 업데이트 되지 않음.

상세페이지 관련 - Detail.js

```
import React, { useEffect } from "react";
import styled from "styled-components";
import { useDispatch, useSelector } from "react-redux";
import { useNavigate, useParams } from "react-router-dom";
import { getTodoByID } from "../redux/modules/todos.js";

const Detail = () => {
  const dispatch = useDispatch();
  const todo = useSelector((state) => state.todos.todo);

  const { id } = useParams();
  const navigate = useNavigate();

  return (
    <StContainer>
      <StDialog>
        <div>
          <StDialogHeader>
            <div>ID :{todo.id}</div>
            <StButton
              borderColor="#ddd"
              onClick={() => {
                navigate("/");
              }}
            >
              이전으로
            </StButton>
          </StDialogHeader>
          <StTitle>{todo.title}</StTitle>
          <StBody>{todo.body}</StBody>
        </div>
      </StDialog>
    </StContainer>
  );
};

export default Detail;

```

이 페이지 오류

```
src/pages/Detail.jsx
  Line 1:17:   'useEffect' is defined but never used          no-unused-vars
  Line 5:10:   'getTodoByID' is defined but never used        no-unused-vars
  Line 8:9:    'dispatch' is assigned a value but never used  no-unused-vars
  Line 11:11:  'id' is assigned a value but never used

```

기존코드.

```
todos.js  기존코드

// 상세 페이지에서 특정 Todo만 조회하는 action creator
export const getTodoByID = (payload) => {
  return {
    type: GET_TODO_BY_ID,
    payload,
  };
};

...

case GET_TODO_BY_ID:
      return {
        ...state,
        todo: state.todos.find((todo) => {
          return todo.id === action.payload;
        }),
      };
    default:
      return state;

```

# 4번 문제해결

useEffect, getTodoById 와 dispatch 그리고 id 가 사용되지 않았다.
dispatch가 사용되지 않았으니 action객체의 값이 스토어로 가지 못해 렌더링이 되지 않았을 것이다. 상세 페이지에서 특정 Todo만 조회하는 action creator인 `getTodoByID` 로 todo를 가져오지 못했기 때문에 todo를 dispacth와 useEffect 훅을 사용해서 데이터값을 가져오도록 고쳐야한다.

useEffect 훅은 두 개의 인자를 받는데 하나는 부수 효과(side effect) 함수이고, 다른하나는 의존성 배열이 필요하다. 의존성 배열은 부수 효과 함수가 실행 되는 조건을 결정한다.

여기서 useEffect는, useParams에서 추출한 id라는 변수의 값이 변경될 때마다 getTodoByID(id)라는 함수를 실행하게 된다. 그 id값을 액션객체에 담아 dispatch를 통해 스토어로 전달하고 useSelector로 값을 가져와서 화면에 나타내준다.

```
// 수정 후 코드

import { useDispatch, useSelector } from "react-redux";
import { useNavigate, useParams } from "react-router-dom";
import { getTodoByID } from "../redux/modules/todos.js";

const Detail = () => {
  const dispatch = useDispatch();
  const todo = useSelector((state) => state.todos.todo);

  const { id } = useParams();
  const navigate = useNavigate();

  useEffect(() => {
    dispatch(getTodoByID(id));
  }, [dispatch, id]);

```

---

# 5번.문제

- 완료된 카드의 상세 페이지에 진입 하였을 때 올바른 데이터를 불러오지 못함.

기존코드 List.jsx

```
<h2>Working.. 🔥</h2>
      <StListWrapper>
        {todos.map((todo) => {
          if (!todo.isDone) {
            return (
              <StTodoContainer key={todo.id}>
                <StLink to={`/${todo.id}`} key={todo.id}>
                  <div>상세보기</div>
                </StLink>
                <div>
                ...

<h2 className="list-title">Done..! 🎉</h2>
      <StListWrapper>
      //
        {todos.map((todo, index) => {
          if (todo.isDone) {
            return (
              <StTodoContainer key={todo.id}>
                <StLink to={`/${index}`} key={todo.id}>
                  <div>상세보기</div>
                </StLink>
                <div>


```

# 5. 문제해결

```
// Done 존 .
return (
              <StTodoContainer key={todo.id}>
                <StLink to={`/${todo.id}`} key={todo.id}>
                  <div>상세보기</div>
```

Work 존의 Link와 Done 존의 Link 가 달랐다.
Done존은 Link 경로에 index 를 넣었는데 index는 카운트를 0부터 하고 todolist의 id는 1부터 시작하기 때문에 서로 값이 일치하지 않는다.
그렇기 때문에 index+1을 해주거나 위의 Work 존 처럼 {todo.id}를 사용해 정확하게 값을 지목해준다.

---

# 6번 문제

- 취소 버튼 클릭시 기능이 작동하지 않음.

```
// 기존코드
//List.jsx

const List = () => {
  const dispatch = useDispatch();
  const todos = useSelector((state) => state.todos.todos);

  const onDeleteTodo = (id) => {
    dispatch(deleteTodo(id));
  };

  const onToggleStatusTodo = (id) => {
    dispatch(toggleStatusTodo(id));
  };

  return (
    <StListContainer>
      <h2>Working.. 🔥</h2>
      <StListWrapper>
        {todos.map((todo) => {
          if (!todo.isDone) {
            return (
              <StTodoContainer key={todo.id}>
                <StLink to={`/${todo.id}`} key={todo.id}>
                  <div>상세보기</div>
                </StLink>
                <div>
                  <h2 className="todo-title">{todo.title}</h2>
                  <div>{todo.body}</div>
                </div>
                <StDialogFooter>
                  <StButton
                    borderColor="red"
                    onClick={() => onDeleteTodo(todo.id)}
                  >
                    삭제하기
                  </StButton>
                  <StButton
                    borderColor="green"
                    onClick={() => onToggleStatusTodo(todo.id)}
                  >
                    {todo.isDone ? "취소!" : "완료!"}
                  </StButton>
                </StDialogFooter>
              </StTodoContainer>
            );
          } else {
            return null;
            }
        })}
      </StListWrapper>
      <h2 className="list-title">Done..! 🎉</h2>
      <StListWrapper>
        {todos.map((todo, index) => {
          if (todo.isDone) {
            return (
              <StTodoContainer key={todo.id}>
                <StLink to={`/${todo.id}`} key={todo.id}>
                  <div>상세보기</div>
                </StLink>
                <div>
                  <h2 className="todo-title">{todo.title}</h2>
                  <div>{todo.body}</div>
                </div>
                <StDialogFooter>
                  <StButton
                    borderColor="red"
                    onClick={() => onDeleteTodo(todo.id)}
                  >
                    삭제하기
                  </StButton>
                  <StButton borderColor="green" onClick={onToggleStatusTodo}>
                    {todo.isDone ? "취소!" : "완료!"}
                  </StButton>
                </StDialogFooter>
              </StTodoContainer>
            );
          } else {
            return null;
          }
        })}
      </StListWrapper>
    </StListContainer>
  );
};

export default List;
```

# 6. 문제해결

Work 존의 완료 부분과 Done 의 취소 버튼 부분 비교했을때
Done 취소 부분에 onClick 이벤트핸들러에 있는 onToggleStatusTodo에 id 인자가 전달되지 않은걸 볼 수 있다. (todo.id)로 인자를 전달해주었다.

```
// 수정전 기존코드  List.jsx

<StButton borderColor="green"
                    onClick={() => onToggleStatusTodo(todo.id)}
                  >
                    {todo.isDone ? "취소!" : "완료!"}
                  </StButton>

                  //Done
                  <StButton borderColor="green" onClick={onToggleStatusTodo}>
                    {todo.isDone ? "취소!" : "완료!"}
                  </StButton>

```

```
//수정후

 <StButton
                    borderColor="green"
                    onClick={() => onToggleStatusTodo(todo.id)}
                  >
                    {todo.isDone ? "취소!" : "완료!"}
```
