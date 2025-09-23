# Danh sách Hook React (React 18 + React 19)

Hook là các logic xử lý DOM ảo trong react

#### ⭐⭐⭐⭐⭐ (Rất hay dùng)

- **useState** – quản lý state trong function component.
- **useEffect** – xử lý side effect (fetch API, DOM, timer...).

---

#### ⭐⭐⭐⭐ (Hay dùng)

- **useContext** – lấy dữ liệu từ Context.
- **useReducer** – quản lý state phức tạp.
- **useRef** – tham chiếu DOM hoặc giữ giá trị không re-render.
- **useId** – sinh id duy nhất.
- **use** (React 19) – sử dụng promise/resource trực tiếp trong render (Suspense).

---

#### ⭐⭐⭐ (Thường dùng trong tối ưu hoặc UI phức tạp)

- **useMemo** – ghi nhớ kết quả tính toán nặng.
- **useCallback** – ghi nhớ hàm, tránh re-render component con.
- **useDeferredValue** – trì hoãn cập nhật value, UI mượt hơn. _(dùng khi ta có 1 state → nhưng muốn “bản copy” của nó update chậm hơn.)_
- **useTransition** – đánh dấu update ít ưu tiên (non-urgent). _(dùng khi ta có 2 state khác nhau → cái nào gấp, cái nào không gấp)_.

---

#### ⭐⭐ (Ít gặp)

- **useLayoutEffect** – giống useEffect nhưng chạy đồng bộ trước khi paint

---

# Hook cơ bản, hay dùng nhất

## useState

`useState` dùng để khai báo state trong function component.

```jsx
import React, { useState } from "react";

function Counter() {
  // khởi tạo state
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Bạn đã bấm {count} lần</p>
      <button onClick={() => setCount(count + 1)}>Bấm tôi</button>
    </div>
  );
}
```

## useEffect

`useEffect` dùng để thực hiện side effects (fetch API, DOM manipulation, log, v.v.).

```jsx
import React, { useState, useEffect } from "react";

function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds((s) => s + 1);
    }, 1000);

    // cleanup function
    return () => clearInterval(interval);
  }, []);

  return <p>Đã trôi qua {seconds} giây</p>;
}
```

# hook cụ thể cho từng trường hợp

## useContext?

- Là hook dùng để lấy dữ liệu từ React Context.
- Thay vì truyền props qua nhiều tầng (prop-drilling), ta dùng Context để chia sẻ state/data trực tiếp cho nhiều component con.

### Các bước sử dụng

1. Tạo Context bằng createContext.
2. Dùng Context.Provider để cung cấp dữ liệu cho cây component con.
3. Dùng useContext(MyContext) trong component để lấy dữ liệu.

```tsx
import React, { createContext, useContext } from "react";

// 1. Tạo Context
const ThemeContext = createContext<"light" | "dark">("light");

function ThemedButton() {
  // 2. Lấy dữ liệu từ Context
  const theme = useContext(ThemeContext);
  return (
    <button
      style={{
        background: theme === "dark" ? "#333" : "#fff",
        color: theme === "dark" ? "#fff" : "#000",
      }}
    >
      Nút theo theme: {theme}
    </button>
  );
}

export default function App() {
  return (
    // 3. Cung cấp giá trị cho Context
    <ThemeContext.Provider value="dark">
      <ThemedButton />
    </ThemeContext.Provider>
  );
}
```

## useRef

- Tham chiếu đến phần tử DOM (giống document.getElementById nhưng theo cách React).
- Lưu trữ giá trị “mutable” qua nhiều lần render mà không gây re-render khi thay đổi.

```tsx
import React, { useRef, useEffect } from "react";

export default function FocusInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    inputRef.current?.focus(); // truy cập DOM thật
  }, []);

  return <input ref={inputRef} placeholder="Nhập gì đó..." />;
}
```

## useReducer

- useReducer là một hook trong React (từ React 16.8).
- Dùng thay cho useState khi logic state phức tạp (nhiều hành động, nhiều nhánh if/switch).
- hiểu đơn giản nó là useState + logic trực tiếp trong setState

## useDeferredValue
```tsx
import React, { useState, useDeferredValue } from "react";

export default function SearchList({ items }: { items: string[] }) {
  const [query, setQuery] = useState("");

  // deferredQuery sẽ update chậm hơn query
  const deferredQuery = useDeferredValue(query);

  // Filter list dựa theo deferredQuery
  const filtered = items.filter(item =>
    item.toLowerCase().includes(deferredQuery.toLowerCase())
  );

  return (
    <div>
      <input
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="Tìm kiếm..."
      />
      <ul>
        {filtered.map((item, i) => (
          <li key={i}>{item}</li>
        ))}
      </ul>
    </div>
  );
}
```

## startTransition

```jsx
import React, { useState, useTransition } from "react";

export default function SearchList({ items }: { items: string[] }) {
  const [query, setQuery] = useState("");
  const [filtered, setFiltered] = useState(items);

  const [isPending, startTransition] = useTransition();

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value;

    // update gấp: input hiển thị ngay
    setQuery(value);

    // update không gấp: filter list
    startTransition(() => {
      const newFiltered = items.filter(item =>
        item.toLowerCase().includes(value.toLowerCase())
      );
      setFiltered(newFiltered);
    });
  };

  return (
    <div>
      <input value={query} onChange={handleChange} placeholder="Tìm kiếm..." />
      {isPending && <p>Đang lọc dữ liệu...</p>}
      <ul>
        {filtered.map((item, i) => (
          <li key={i}>{item}</li>
        ))}
      </ul>
    </div>
  );
}
```

### useLayoutEffect
→ Chỉ dùng để sử lý DOM trực tiếp
```jsx
import { useLayoutEffect, useRef, useState } from "react";

export default function LayoutEffectDemo() {
  const boxRef = useRef < HTMLDivElement > null;
  const [width, setWidth] = useState(0);

  useLayoutEffect(() => {
    if (boxRef.current) {
      setWidth(boxRef.current.getBoundingClientRect().width);
    }
  }, []);

  return (
    <div>
      <div ref={boxRef} style={{ width: "50%" }}>
        Hộp
      </div>
      <p>Chiều rộng hộp: {width}px</p>
    </div>
  );
}
```



## Quy trình React xử lý khi thay đổi state

    1. Gọi hàm setState (ví dụ: setCount(newCount)):
        - React nhận yêu cầu thay đổi state.
        - Nếu newCount khác với giá trị cũ, React sẽ đánh dấu component cần re-render.
        - Nếu giá trị mới giống giá trị cũ (shallow equal), React sẽ bỏ qua render lại.
    2. Batching (gom các cập nhật lại):
        - Trong React 18+, mọi cập nhật state trong event handler, promise, setTimeout, async/await đều được gom lại (automatic batching).
        - Điều này giúp giảm số lần render và chỉ re-render một lần sau khi tất cả cập nhật đã gom.
    3. Reconciliation (so sánh Virtual DOM):
        - React gọi lại hàm component để tạo Virtual DOM mới.
        - Virtual DOM mới được so sánh (diffing) với Virtual DOM cũ.
        - React tìm ra sự khác biệt.
    4. Commit Phase (cập nhật UI thật):
        - React cập nhật DOM thật dựa trên sự khác biệt tìm thấy.
        - Gọi các lifecycle/hook tương ứng (ví dụ: useEffect sau khi commit xong UI).
    5. Gọi Effects:
        - Sau khi UI đã render xong, React chạy các useEffect (asynchronous, sau commit).
        - Nếu có cleanup (hàm return trong useEffect), cleanup được gọi trước khi chạy effect mới hoặc khi component unmount.