# Query trong react-router

Các props có thể có giá trị mặc định hoặc lưu trữ dữ liệu trên câu query

## 1. lưu trữ dữ liệu,
các dữ liệu ổn định (id của đối tượng cần Edit, các tab_id, hãy có mode cục bộ) nên đc lưu trữ trên câu query để?
- Tối ưu logic của người xem năng cao (power user)
- Tối ưu khi share link cho 1 người khác
- Dễ dàng tái hiện để debug
- Sync với back/forward của trình duyệt.

```tsx

import React from "react";
import { BrowserRouter, Routes, Route, useSearchParams } from "react-router-dom";

function DocumentPage() {
  const [searchParams, setSearchParams] = useSearchParams();

  // Lấy state từ query string
  const page = searchParams.get("page") || "1";
  const filter = searchParams.get("filter") || "all";

  // Cập nhật state vào query string
  const updateFilter = (newFilter: string) => {
    setSearchParams({ page, filter: newFilter });
    // setSearchParams({ page, filter }, { replace: true }); - không lưu vào history
  };

  const updatePage = (newPage: number) => {
    setSearchParams({ page: String(newPage), filter });
    // setSearchParams({ page, filter }, { replace: true }); - không lưu vào history
  };

  return (
    <main>
      <h1>Tài liệu (Page {page})</h1>
      <p>Bộ lọc: {filter}</p>

      <button onClick={() => updateFilter("all")}>Tất cả</button>
      <button onClick={() => updateFilter("active")}>Hoạt động</button>
      <button onClick={() => updateFilter("archived")}>Đã lưu trữ</button>

      <hr />

      <button onClick={() => updatePage(Number(page) - 1)} disabled={Number(page) <= 1}>
        Trang trước
      </button>
      <button onClick={() => updatePage(Number(page) + 1)}>Trang sau</button>
    </main>
  );
}

export default function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<DocumentPage />} />
      </Routes>
    </BrowserRouter>
  );
}

```

## 2. State trong navigation
dùng để gửi dữ liệu ẩn (không xuất hiện trong câu query - bảo mật)
	
```tsx
// Gửi dữ liệu tạm khi navigate: 
navigate("/profile", { state: { from: "/checkout" } });

// Trong component:
const location = useLocation();
console.log(location.state?.from);

```