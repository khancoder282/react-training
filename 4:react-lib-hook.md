# Thư viện Hook của react

## fuse.js

Full-text fuzzy search `npm install fuse.js`

```tsx
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
      ? items.map((item) => ({ item, matches: [] }))
      : fuse.search(deferredQuery);

  return (
    <div>
      {/* Input search */}
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
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

```tsx
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

## ahooks

### 1. useRequest

- Công dụng: Quản lý async request (API call, fetch, v.v).
- Trường hợp: Gọi API có loading, error, retry, pagination, polling.

```tsx
import { useRequest } from "ahooks";

function UserList() {
  const { data, loading, error } = useRequest(() =>
    fetch("/api/users").then((res) => res.json())
  );

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error!</p>;

  return (
    <ul>
      {data.map((u: any) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
}
```

### 2. useBoolean

- Công dụng: Quản lý state boolean nhanh gọn.
- Trường hợp: Toggle modal, switch, expand/collapse.

```tsx
import { useBoolean } from "ahooks";

function Demo() {
  const [visible, { toggle }] = useBoolean(false);
  return (
    <>
      <button onClick={toggle}>Toggle</button>
      {visible && <p>Hello World</p>}
    </>
  );
}
```

### 3. useDebounce

- Công dụng: Trì hoãn cập nhật state/hàm khi nhập liệu liên tục.
- Trường hợp: Search input, tránh gọi API liên tục.

```tsx
import { useDebounce } from "ahooks";

function Search() {
  const [value, setValue] = React.useState("");
  const debounced = useDebounce(value, { wait: 500 });

  React.useEffect(() => {
    if (debounced) {
      console.log("Search API:", debounced);
    }
  }, [debounced]);

  return <input value={value} onChange={(e) => setValue(e.target.value)} />;
}
```

### 4. useThrottle / useThrottleFn

- Công dụng: Giới hạn tần suất gọi hàm.
- Trường hợp: Scroll, resize, drag.

```tsx
import { useThrottleFn } from "ahooks";

function Scroll() {
  useThrottleFn(() => console.log("scrolling..."), { wait: 1000 });

  return <div style={{ height: "200vh" }}>Scroll down</div>;
}
```

### 5. useInterval

- Công dụng: Chạy interval an toàn trong React.
- Trường hợp: Đồng hồ, polling API, animation lặp.

```tsx
import { useInterval } from "ahooks";

function Timer() {
  const [count, setCount] = React.useState(0);
  useInterval(() => setCount((c) => c + 1), 1000);
  return <p>{count}</p>;
}
```

### 6. useCountDown

- Công dụng: Tạo bộ đếm ngược (ms → days/hours/minutes/seconds).
- Trường hợp: Flash sale, OTP countdown.

```tsx
import { useCountDown } from "ahooks";

function OTP() {
  const [leftTime, { start }] = useCountDown({
    targetDate: Date.now() + 10000,
  });
  return (
    <div>
      <p>Time left: {Math.round(leftTime / 1000)}s</p>
      <button onClick={start}>Restart</button>
    </div>
  );
}
```

### 7. useLocalStorageState

- Công dụng: State lưu trong localStorage.
- Trường hợp: Dark mode, remember user settings.

```tsx
import { useLocalStorageState } from "ahooks";

function ThemeToggle() {
  const [theme, setTheme] = useLocalStorageState("theme", {
    defaultValue: "light",
  });
  return (
    <button onClick={() => setTheme(theme === "light" ? "dark" : "light")}>
      {theme}
    </button>
  );
}
```

### 8. useInViewport

- Công dụng: Kiểm tra element có trong viewport không.
- Trường hợp: Lazy load hình ảnh, trigger animation khi vào khung nhìn.

```tsx
import { useInViewport } from "ahooks";

function Demo() {
  const ref = React.useRef(null);
  const [inView] = useInViewport(ref);
  return <div ref={ref}>{inView ? "In view" : "Out of view"}</div>;
}
```

### 9. useScroll

- Công dụng: Lấy vị trí scroll của element hoặc window.
- Trường hợp: Back-to-top button, lazy load, infinite scroll.

```tsx
import { useScroll } from "ahooks";

function Demo() {
  const scroll = useScroll(document);
  return <p>Scroll Y: {scroll?.top}</p>;
}
```

### 10. useHistoryTravel

- Công dụng: Undo/Redo cho state.
- Trường hợp: Text editor, form multi-step.

```tsx
import { useHistoryTravel } from "ahooks";

function Demo() {
  const { value, setValue, back, forward } = useHistoryTravel(0);
  return (
    <div>
      <p>Value: {value}</p>
      <button onClick={() => setValue((v) => v + 1)}>+1</button>
      <button onClick={back}>Undo</button>
      <button onClick={forward}>Redo</button>
    </div>
  );
}
```

### 11. useDrop

- Công dụng: Kéo thả file (drag & drop).
- Khi dùng: Upload file, drop zone.

```tsx
import React, { useState } from "react";
import { useDrop } from "ahooks";

export default () => {
  const [files, setFiles] = useState<File[]>([]);

  const ref = useDrop({
    onFiles: setFiles,
  });

  return (
    <div ref={ref} style={{ border: "2px dashed gray", padding: 20 }}>
      Drop files here
      <ul>
        {files.map((file) => (
          <li key={file.name}>{file.name}</li>
        ))}
      </ul>
    </div>
  );
};
```

### 12.useUpdateEffect
-	Công dụng: Giống useEffect nhưng bỏ qua lần render đầu tiên.
-	Khi dùng: Chỉ chạy effect khi state thay đổi sau lần mount.

```tsx
import React, { useState } from 'react';
import { useUpdateEffect } from 'ahooks';

export default () => {
  const [count, setCount] = useState(0);

  useUpdateEffect(() => {
    console.log('Count changed:', count);
  }, [count]);

  return <button onClick={() => setCount(c => c + 1)}>Add {count}</button>;
};
```

### 13. useExternal
-	Công dụng: Load script/css ngoài vào React.
-	Khi dùng: Thêm Google Maps, analytics, thư viện UI ngoài.

```tsx
import React from 'react';
import { useExternal } from 'ahooks';

export default () => {
  const status = useExternal('https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.21/lodash.min.js');

  return <div>Script load status: {status}</div>;
};
```

### 14. useSelections
-	Công dụng: Quản lý danh sách chọn (select all, invert, clear).
-	Khi dùng: Table checkbox, list select.

```tsx
import React from 'react';
import { useSelections } from 'ahooks';

export default () => {
  const list = ['Apple', 'Banana', 'Orange'];
  const { selected, allSelected, toggle, selectAll, unSelectAll } = useSelections(list);

  return (
    <div>
      <button onClick={selectAll}>Select All</button>
      <button onClick={unSelectAll}>Unselect All</button>
      {list.map(item => (
        <label key={item}>
          <input
            type="checkbox"
            checked={selected.includes(item)}
            onChange={() => toggle(item)}
          />
          {item}
        </label>
      ))}
      <p>Selected: {selected.join(', ')}</p>
      <p>All selected? {allSelected ? 'Yes' : 'No'}</p>
    </div>
  );
};
```