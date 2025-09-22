# Props là gì

- Props (properties) là cách truyền dữ liệu từ component cha → component con.
- Component con nhận props như tham số của hàm.

```tsx
// Khởi tạo props
type ButtonProps = {
  label: string;
  color?: string; // optional
};

function Button({ label, color = "blue" }: ButtonProps) {
  return <button style={{ background: color }}>{label}</button>;
}

// Props mặc định
type Props = {
  text?: string;
};

function Greeting({ text = "Xin chào" }: Props) {
  return <h1>{text}</h1>;
}
```

# Props đặc biệt (built-in props)

## 1. Children

- Là prop đặc biệt chứa nội dung nằm giữa thẻ mở và thẻ đóng của component.
- Loại: ReactNode (string, number, element, fragment, null, undefined…).

```tsx
function Card({ children }: { children: React.ReactNode }) {
  return <div className="card">{children}</div>;
}

export default function App() {
  return (
    <Card>
      <h2>Tiêu đề</h2>
      <p>Nội dung bên trong Card</p>
    </Card>
  );
}
```

## 2. key

- Dùng khi render danh sách để React xác định phần tử nào thay đổi/giữ nguyên.
- Không thể đọc props.key bên trong component → chỉ React dùng.

**_Hạn chế sử dụng index list làm key_**

```jsx
<ul>
  {["A", "B", "C"].map((item, i) => (
    <li key={item}>{item}</li>
    // No: <li key={i}>{item}</li>
  ))}
</ul>
```

**Lưu ý: Khi xử lý list không nên xử lý onchange cho các input. Sẽ dẫn đến lag hệ thống do render liên tục**

## 3. ref

- Dùng để tham chiếu đến DOM hoặc component.
- Không phải prop thông thường, mà là special attribute.
- Có thể dùng useRef, forwardRef để truy cập.
- **Với component custom, cần forwardRef mới nhận được ref**

```tsx
import { useRef, useEffect } from "react";

function InputBox() {
  const inputRef = useRef < HTMLInputElement > null;

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return (
    <>
      <input ref={inputRef} />
      <div
        ref={(t) => {
          // - ref có thể dùng callback sau khi render xong.
          // - có thể ứng dụng để scroll đến item này
          console.log(t.innerText); // log: "Xin chào mọi người"
        }}
      >
        Xin chào mọi người
      </div>
    </>
  );
}
```

Ví dụ về forwardRef **Trong MUI `<Fade in/>` cần nó**

```tsx
import React, { forwardRef, useRef, useImperativeHandle } from "react";

// Custom Input nhận ref từ cha
const CustomInput = forwardRef<HTMLInputElement, { label: string }>(
  ({ label }, ref) => {
    return (
      <div>
        <label>{label}</label>
        <input ref={ref} />
      </div>
    );
  }
);

export default function App() {
  const inputRef = useRef<HTMLInputElement>(null);

  return (
    <div>
      <CustomInput ref={inputRef} label="Nhập tên:" />
      <button onClick={() => inputRef.current?.focus()}>Focus input</button>
    </div>
  );
}
```

## 4. dangerouslySetInnerHTML

- Cho phép set HTML trực tiếp (giống innerHTML trong DOM).
- Nếu dữ liệu HTML chèn vào đến từ user input hoặc server không kiểm soát tốt, kẻ tấn công có thể nhúng JavaScript độc hại (XSS – Cross Site Scripting).

```jsx
function HtmlBox() {
  return <div dangerouslySetInnerHTML={{ __html: "<b>In đậm</b>" }} />;
}
```

## 5. style & className

- Không phải đặc biệt như key/ref, nhưng luôn có sẵn.
- style: object CSS inline.
- className: tên class CSS.
- defaultValue, defaultChecked (cho form input uncontrolled).
- suppressContentEditableWarning, suppressHydrationWarning (ít dùng, cho SSR).
- onClick, onChange, … (event handler — bản chất cũng là props).
