# Learning it
In a nutshell, 
here: https://react.dev/reference/react/hooks

## Stuff you need to know
`Getter`, `setter (update)` function
> Unlike Vue.js (mutation re-renders UI), React.js uses setters to re-render UI

```ts
useState, useActionState, useEffect, useRef, useMemo, useCallback, useContext
```

## useState
```ts
const [value, setValue] = useState("initial value")
```

## useActionState
Use for async functions
```ts
// count == value, dispatchAction == the async function 
// dispatchAction() == call the function with default arg 0
const [count, dispatchAction, isPending] = useActionState(async (prevCount, actionPayload) => {
    switch (actionPayload.type) {
        case 'ADD': return await addToCart(prevCount);
        case 'REMOVE': return await removeFromCart(prevCount);
    }
    return prevCount;
}, 0);

function handleAdd() {
    startTransition(() => {
        dispatchAction({ type: 'ADD' });
    });
}

function handleRemove() {
    startTransition(() => {
        dispatchAction({ type: 'REMOVE' });
    });
}
return (<button onClick={handleClick}>Add Ticket{isPending ? ' 🌀' : '  '}</button>)
```

## useEffect
Triggered after the component has rendered and the DOM has been updated.  
Clean up function (return function) is called: 
1. Before the effect runs again (if dependencies change)
2. When the component unmounts

```ts
// Define here: function createConnection(...){}

function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    // Or define here: function createConnection(...){}

    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]); // dependency changed => re-render => useEffect()
}

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');
  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });
  return (
    <>
      <label>
        Server URL:{' '}
        <input value={serverUrl} onChange={e => setServerUrl(e.target.value)} />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}
```

## useRef
Changing a ref does not trigger a re-render.

```ts
export default function Counter() {
  let ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert('You clicked ' + ref.current + ' times!');
  }

  return (
    <button onClick={handleClick}>Click me!</button>
  );
}
```

## useMemo
```ts
function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
  return (
    <ul>
        {visibleTodos.map(todo => (
        <li key={todo.id}>
            {todo.completed ?
            <s>{todo.text}</s> :
            todo.text
            }
        </li>
        ))}
    </ul>
  )
}
```

## useCallback
Memoize your function (and keep function singleton, to pass it around) so that re-render won't happen (because function is the same. Not newly defined) until dependency change:

```ts
function TodoList() {
    const [todos, setTodos] = useState([]);

    const handleAddTodo = useCallback((text) => {
        const newTodo = { id: nextId++, text };
        setTodos([...todos, newTodo]);
    }, [todos]); // with dependency
    // ...
    return (<Button onClick={()=>handleAddTodo("Example todo")}>Add</button>)
}

function TodoList() {
    const [todos, setTodos] = useState([]);

    const handleAddTodo = useCallback((text) => {
        const newTodo = { id: nextId++, text };
        setTodos(todos => [...todos, newTodo]);
    }, []); // no dependency
    // ...
    return (<Button onClick={()=>handleAddTodo("Example todo")}>Add</button>)
}
```

## useContext
Important stuff. Data sharing across components.

Note: Seriously use context management library (like Zustand). Don't torture yourself.

Passing in Object:
```ts
function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  const login = useCallback((response) => {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }, []);

  const contextValue = useMemo(() => ({
    currentUser,
    login
  }), [currentUser, login]);

  return (
    <AuthContext value={contextValue}>
      <Page />
    </AuthContext>
  );
}
```

Reducer:
```ts
export default function TaskApp() {
  return (
    <TasksProvider>
      <h1>Day off in Kyoto</h1>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}

const TasksContext = createContext(null);
const TasksDispatchContext = createContext(null);
export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    [{ id: 0, text: "Philosopher's Path", done: true },]
  );

  return (
    <TasksContext value={tasks}>
      <TasksDispatchContext value={dispatch}>
        {children}
      </TasksDispatchContext>
    </TasksContext>
  );
}

export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
    // How to use:
    // const dispatch = useTasksDispatch();
    // <button onClick={() => {setText(''); dispatch({type: 'added', id, text, done}); }}>Add</button>
  return useContext(TasksDispatchContext);
}

function tasksReducer(tasks, action) {
    // No mutation here
    switch (action.type) {
        case 'added': return [...tasks, {
            id: action.id,
            text: action.text,
            done: false
        }];
        case 'changed': return tasks.map(t => {
            if (t.id === action.task.id)
                return action.task;
            return t;
        });
        case 'deleted': return tasks.filter(t => t.id !== action.id);
        // ... throw Error()
    }
}

```