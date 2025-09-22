# Danh sách các Type trong TypeScript

## 1. Kiểu dữ liệu cơ bản (Primitive Types)

### number

Đại diện cho số (nguyên, thập phân, NaN, Infinity…).

```ts
let age: number = 25;
let pi: number = 3.14;
```

### string

Chuỗi ký tự, có thể dùng `"`, `'`, hoặc `` ` ``.

```ts
let name: string = "TypeScript";
let greeting: string = `Hello, ${name}`;
```

### boolean

Chỉ có `true` hoặc `false`.

```ts
let isAdmin: boolean = true;
```

### null và undefined

```ts
let empty: null = null;
let notAssigned: undefined = undefined;
```

### symbol

Dùng để tạo giá trị duy nhất, thường cho key của object.

```ts
let id1 = Symbol("id");
let id2 = Symbol("id");
console.log(id1 === id2); // false
```

### bigint

Dùng cho số nguyên rất lớn.

```ts
let big: bigint = 123456789012345678901234567890n;
```

---

## 2. Kiểu nâng cao (Advanced Types)

### any

Bỏ qua kiểm tra kiểu (dùng khi không biết chính xác).

```ts
let randomValue: any = 10;
randomValue = "Hello"; // không báo lỗi
```

### unknown

Giống `any` nhưng an toàn hơn, bắt buộc check trước khi dùng.

```ts
let input: unknown = "hello";
if (typeof input === "string") {
  console.log(input.toUpperCase());
}
```

### void

Thường dùng cho hàm không trả về giá trị.

```ts
function logMessage(): void {
  console.log("This is a log");
}
```

### never

Đại diện cho giá trị không bao giờ xảy ra (hàm ném lỗi hoặc vòng lặp vô hạn).

```ts
function throwError(): never {
  throw new Error("Something went wrong");
}
```

---

## 3. Kiểu cấu trúc dữ liệu

### array

```ts
let numbers: number[] = [1, 2, 3];
let strings: Array<string> = ["a", "b"];
```

### tuple

Mảng với số phần tử cố định và type cố định.

```ts
let user: [string, number] = ["Alice", 25];
```

### object

```ts
let person: { name: string; age: number } = {
  name: "Bob",
  age: 30,
};
```

### enum

Tập hợp giá trị cố định.

```ts
enum Direction {
  Up,
  Down,
  Left,
  Right,
}
let move: Direction = Direction.Up;
```

---

## 4. Kiểu kết hợp (Union & Intersection)

### union (|)

Một biến có thể mang nhiều loại type.

```ts
let id: number | string;
id = 101;
id = "abc";
```

### intersection (&)

Kết hợp nhiều type thành một type.

```ts
type A = { name: string };
type B = { age: number };
type Person = A & B;

let user: Person = { name: "Tom", age: 20 };
```

---

## 5. Type Aliases & Interfaces

### type

Đặt tên cho kiểu dữ liệu.

```ts
type ID = number | string;
let userId: ID = "123";
```

### interface

Dùng để mô tả cấu trúc object, hỗ trợ mở rộng.

```ts
interface User {
  id: number;
  name: string;
}
let user: User = { id: 1, name: "Alice" };
```

---

## 6. Generics (Kiểu tổng quát)

Cho phép viết hàm/class/type mà không cố định kiểu trước.

```ts
function identity<T>(value: T): T {
  return value;
}

let num = identity<number>(10);
let str = identity("hello");
```

---

## 7. Utility Types (Có sẵn trong TS)

### Partial<T>

Biến tất cả các thuộc tính thành tùy chọn.

```ts
interface User {
  id: number;
  name: string;
}
let updateUser: Partial<User> = { name: "Bob" };
```

### Required<T>

Ngược lại với `Partial`, bắt buộc tất cả thuộc tính.

```ts
type FullUser = Required<User>;
```

### Pick<T, K>

Lấy ra một số thuộc tính.

```ts
type UserName = Pick<User, "name">;
```

### Omit<T, K>

Bỏ đi một số thuộc tính.

```ts
type UserWithoutId = Omit<User, "id">;
```

### Record<K, T>

Tạo object với key kiểu `K` và value kiểu `T`.

```ts
type UserRole = Record<string, number>;
let roles: UserRole = { admin: 1, user: 2 };
```

### Readonly<T>

Biến tất cả thuộc tính thành chỉ đọc.

```ts
let readonlyUser: Readonly<User> = { id: 1, name: "Alice" };
// readonlyUser.id = 2; // ❌ lỗi
```

---

## 8. Kiểu nâng cao khác

### keyof

Lấy ra tất cả tên thuộc tính của type.

```ts
type UserKeys = keyof User; // "id" | "name"
```

### typeof

Lấy type của biến.

```ts
let person = { name: "Tom", age: 30 };
type PersonType = typeof person;
```

### Indexed Access Types

Truy cập kiểu theo key.

```ts
type NameType = User["name"]; // string
```

### Conditional Types

Định nghĩa type dựa theo điều kiện.

```ts
type IsString<T> = T extends string ? true : false;
type A = IsString<string>; // true
type B = IsString<number>; // false
```

# Các dạng file định nghĩa Type

#### 1. Global Declaration

📌 Dùng khi: mở rộng type toàn cục (window, document, globalThis…).

- Khai báo biến, type, interface dùng toàn cục trong project.
- Không cần import, có thể dùng ở bất cứ đâu.

```tsx
// global.d.ts
declare global {
  interface Window {
    myAppVersion: string;
  }
  const APP_NAME: string;
}
```

### 2. Ambient Declaration

📌 Dùng khi: muốn TS biết biến/hàm tồn tại nhưng code thật được định nghĩa ở nơi khác (JS runtime). Cần bổ sung type cho thư viện bên ngoài.

- Dùng declare để khai báo mà không cần implement.
- Mở rộng type có sẵn của module hoặc interface.
- Tự viết .d.ts để quản lý type trong dự án. (viết ở src/types/...d.ts)

```tsx
// ambient.d.ts
declare function logMessage(msg: string): void;
declare const VERSION: string;

// express.d.ts
declare module "express" {
  interface Request {
    user?: { id: number; name: string };
  }
}

// api.d.ts
declare namespace API {
  interface User {
    id: number;
    name: string;
  }
}
```

### 3. Module Declaration

📌 Dùng khi: import 1 thư viện không có type (.js), ta viết .d.ts để TS hiểu.

- Khai báo type cho 1 module (thường dùng cho thư viện JS không có type).

```tsx
// some-lib.d.ts
declare module "some-lib" {
  export function hello(name: string): string;
}
```
