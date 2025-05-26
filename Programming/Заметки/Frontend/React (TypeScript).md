## Введение

React — это библиотека для построения пользовательских интерфейсов, разработанная Facebook. Она использует декларативный подход, компонентную архитектуру и виртуальный DOM. [[TypeScript]] — это надстройка над JavaScript, добавляющая статическую типизацию. Комбинация React и TypeScript позволяет писать более безопасный, предсказуемый и масштабируемый код.

---

## 1. Основные концепции React

### Компоненты

Компоненты — строительные блоки UI. В React компоненты бывают:

- **Функциональные** — на основе функций
    
- **Классовые** — на основе классов (используются редко с появлением хуков)
    

Пример функционального компонента:

```tsx
import React from 'react';

type Props = {
  name: string;
};

const Greeting: React.FC<Props> = ({ name }) => {
  return <h1>Hello, {name}!</h1>;
};

export default Greeting;
```

### JSX

JSX — это синтаксис, похожий на HTML, который компилируется в вызовы `React.createElement()`.

---

## 2. Хуки (Hooks)

Хуки позволяют функциональным компонентам использовать возможности, которые раньше были доступны только в классовых (например, состояние, жизненный цикл).

### useState

```tsx
const [count, setCount] = useState<number>(0);
```

### useEffect

```tsx
useEffect(() => {
  console.log('Компонент отрендерился или count изменился');
}, [count]);
```

### useRef

```tsx
const inputRef = useRef<HTMLInputElement>(null);
```

### useContext

```tsx
const MyContext = React.createContext<string>('default');
const value = useContext(MyContext);
```

### useMemo / useCallback

- `useMemo` кэширует вычисленное значение
    
- `useCallback` кэширует функцию
    

```tsx
const memoizedValue = useMemo(() => computeValue(a, b), [a, b]);
const memoizedCallback = useCallback(() => doSomething(a), [a]);
```

---

## 3. Часто используемые типы и интерфейсы

### Типизация пропсов и стейта

```tsx
type MyProps = {
  title: string;
  isActive?: boolean;
};

const MyComponent: React.FC<MyProps> = ({ title, isActive = false }) => {
  // ...
};
```

### Типизация событий

```tsx
const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
  // ...
};

const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  setValue(e.target.value);
};
```

### Типизация useRef

```tsx
const inputRef = useRef<HTMLInputElement>(null);
```

### Типизация useState с массивами и объектами

```tsx
const [users, setUsers] = useState<Array<User>>([]);

interface User {
  id: number;
  name: string;
}
```

---

## 4. Работа с формами

```tsx
const [formData, setFormData] = useState<{ name: string; age: number }>({
  name: '',
  age: 0,
});

const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  const { name, value } = e.target;
  setFormData((prev) => ({
    ...prev,
    [name]: name === 'age' ? +value : value,
  }));
};
```

---

## 5. Обработка асинхронных данных

```tsx
useEffect(() => {
  const fetchData = async () => {
    const res = await fetch('/api/data');
    const json = await res.json();
    setData(json);
  };

  fetchData();
}, []);
```

---

## 6. Роутинг (React Router)

```tsx
import { BrowserRouter as Router, Route, Routes } from 'react-router-dom';

<Router>
  <Routes>
    <Route path="/" element={<Home />} />
    <Route path="/about" element={<About />} />
  </Routes>
</Router>
```

---

## 7. Управление глобальным состоянием

- **Context API** — для небольших глобальных состояний
    
- **[[Redux]] / Zustand / Recoil / Jotai** — для более сложных приложений
    

Пример с контекстом:

```tsx
interface AuthContextType {
  user: string;
  setUser: (user: string) => void;
}

const AuthContext = React.createContext<AuthContextType | undefined>(undefined);

const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [user, setUser] = useState('');

  return (
    <AuthContext.Provider value={{ user, setUser }}>
      {children}
    </AuthContext.Provider>
  );
};
```

---

## 8. Полезные утилиты

### Тип `ReactNode`

Используется для передачи компонентов или JSX:

```tsx
type Props = {
  children: React.ReactNode;
};
```

### Тип `Dispatch<SetStateAction<T>>`

```tsx
const setCount: React.Dispatch<React.SetStateAction<number>> = useState(0)[1];
```

---

## Заключение

React с TypeScript — это мощное сочетание для создания надежных и масштабируемых приложений. TypeScript помогает избежать ошибок, а React предоставляет гибкость и модульность. Знание хуков, типизации и архитектурных подходов делает разработчика более продуктивным и уверенным.

---

## Полезные ссылки

- [https://reactjs.org/](https://reactjs.org/)
    
- [https://www.typescriptlang.org/](https://www.typescriptlang.org/)
    
- [https://react-typescript-cheatsheet.netlify.app/](https://react-typescript-cheatsheet.netlify.app/)