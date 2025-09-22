# Thư viện Hook của react

## use-undo

- Quản lý biến đổi state làm lịch sử
  `npm install use-undo`

```jsx
import React, { useState, useRef, useEffect } from "react";
import useUndo from "use-undo";

export default function UndoExample() {
  // State hiện tại của textarea
  const [text, setText] = useState("");

  // useUndo quản lý lịch sử undo/redo
  const [{ present }, { set, undo, redo, canUndo, canRedo }] = useUndo(text);

  // Cờ chặn render liên tục
  const isChange = useRef < boolean > false;

  // Tự động đồng bộ text với undo, thực tế nên dùng debound
  useEffect(() => {
    if (text !== present) {
      if (isChange.current) {
        // Cập nhật history undo/redo
        set(text);
      } else {
        setText(present);
      }
      isChange.current = false; // reset cờ
    }
  }, [text, present, set]);

  // Khi textarea thay đổi
  const handleChange = (e: React.ChangeEvent<HTMLTextAreaElement>) => {
    isChange.current = true; // bật cờ
    setText(e.target.value);
  };

  return (
    <div>
      <div style={{ marginBottom: 8 }}>
        <button onClick={undo} disabled={!canUndo}>
          Undo
        </button>
        <button onClick={redo} disabled={!canRedo}>
          Redo
        </button>
      </div>

      <textarea
        value={text}
        onChange={handleChange}
        rows={5}
        cols={50}
        placeholder="Nhập nội dung..."
      />

      <p>Text hiện tại: {text}</p>
    </div>
  );
}
```

## fuse.js

Full-text fuzzy search `npm install fuse.js`

```jsx

// hightlight.tsx
import React from "react";

type HighlightMatchProps = {
  text: string;
  matches: [number, number][];
};

export default function HighlightMatch({ text, matches }: HighlightMatchProps) {
  if (!matches || matches.length === 0) {
    return <>{text}</>;
  }

  let lastIndex = 0;

  return (
    <>
      {matches.map(([start, end], idx) => {
        const before = text.slice(lastIndex, start);
        const match = text.slice(start, end + 1);
        lastIndex = end + 1;

        return (
          <React.Fragment key={idx}>
            {before}
            <span style={{ backgroundColor: "yellow" }}>{match}</span>
          </React.Fragment>
        );
      })}
      {text.slice(lastIndex)}
    </>
  );
}

// fuse-app.tsx
import React, { useState, useDeferredValue } from "react";
import Fuse from "fuse.js";
import HighlightMatch from "./hightlight"; // Component riêng để highlight phần trùng

// Danh sách items để search
const items = ["React", "Angular", "Vue", "Svelte", "Next.js", "Nuxt.js"];

// Cấu hình Fuse.js
// - includeMatches: true → trả về chỉ số match để highlight
// - threshold: 0.3 → mức độ fuzzy (0 = chính xác, 1 = lỏng lẻo)
const fuse = new Fuse(items, {
  includeMatches: true,
  threshold: 0.3,
});

export default function FuzzySearch() {
  // State lưu giá trị input gốc
  const [query, setQuery] = useState("");

  // useDeferredValue tạo bản “trễ” của query
  // Giúp UI input mượt khi danh sách items lớn
  const deferredQuery = useDeferredValue(query);

  // Kết quả search dựa trên deferredQuery
  // Nếu deferredQuery rỗng → hiển thị toàn bộ items
  // Nếu deferredQuery rỗng → trả về toàn bộ items (với matches = [])
  const results =
    deferredQuery === ""
      ? items.map(item => ({ item, matches: [] }))
      : fuse.search(deferredQuery);

  return (
    <div>
      {/* Input search */}
      <input
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="Search..."
        style={{ marginBottom: 8, width: 200 }}
      />

      <ul>
        {results.map((res, i) => (
        <li key={i}>
            {/* HighlightMatch nhận text và các chỉ số match để tô vàng */}
            <HighlightMatch
            text={res.item as string}
            matches={res.matches?.[0]?.indices || []}
            />
        </li>
        ))}
      </ul>
    </div>
  );
}

```


## zustand

- cú pháp đơn giản thay thế context `npm install zustand`

```jsx
import create from "zustand";

// Tạo store
interface CounterStore {
  count: number;
  increment: () => void;
  decrement: () => void;
}

const useCounterStore = create<CounterStore>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
}));

// Component sử dụng store
export default function Counter() {
  const { count, increment, decrement } = useCounterStore();

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Tăng</button>
      <button onClick={decrement}>Giảm</button>
    </div>
  );
}
```



