# Danh sách các loại type trong TypeScript

## 1. **Kiểu dữ liệu cơ bản (Primitive Types)**

### `number`

``` ts
let age: number = 25;
let pi: number = 3.14;
```

### `string`

``` ts
let name: string = "TypeScript";
let greeting: string = `Hello, ${name}`;
```

### `boolean`

``` ts
let isAdmin: boolean = true;
```

### `null` và `undefined`

``` ts
let empty: null = null;
let notAssigned: undefined = undefined;
```

### `symbol`

``` ts
let id1 = Symbol("id");
let id2 = Symbol("id");
console.log(id1 === id2); // false
```

### `bigint`

``` ts
let big: bigint = 123456789012345678901234567890n;
```

------------------------------------------------------------------------

## 2. **Kiểu nâng cao (Advanced Types)**

### `any`

``` ts
let randomValue: any = 10;
randomValue = "Hello";
```

### `unknown`

``` ts
let input: unknown = "hello";
if (typeof input === "string") {
  console.log(input.toUpperCase());
}
```

### `void`

``` ts
function logMessage(): void {
  console.log("This is a log");
}
```

### `never`

``` ts
function throwError(): never {
  throw new Error("Something went wrong");
}
```

------------------------------------------------------------------------

## 3. **Kiểu cấu trúc dữ liệu**

### `array`

``` ts
let numbers: number[] = [1, 2, 3];
let strings: Array<string> = ["a", "b"];
```

### `tuple`

``` ts
let user: [string, number] = ["Alice", 25];
```

### `object`

``` ts
let person: { name: string; age: number } = {
  name: "Bob",
  age: 30
};
```

### `enum`

``` ts
enum Direction {
  Up,
  Down,
  Left,
  Right
}
let move: Direction = Direction.Up;
```

------------------------------------------------------------------------

## 4. **Kiểu kết hợp (Union & Intersection)**

### `union (|)`

``` ts
let id: number | string;
id = 101;
id = "abc";
```

### `intersection (&)`

``` ts
type A = { name: string };
type B = { age: number };
type Person = A & B;

let user: Person = { name: "Tom", age: 20 };
```

------------------------------------------------------------------------

## 5. **Type Aliases & Interfaces**

### `type`

``` ts
type ID = number | string;
let userId: ID = "123";
```

### `interface`

``` ts
interface User {
  id: number;
  name: string;
}
let user: User = { id: 1, name: "Alice" };
```

------------------------------------------------------------------------

## 6. **Generics (Kiểu tổng quát)**

``` ts
function identity<T>(value: T): T {
  return value;
}

let num = identity<number>(10);
let str = identity("hello");
```

------------------------------------------------------------------------

## 7. **Utility Types (Có sẵn trong TS)**

### `Partial<T>`

``` ts
interface User {
  id: number;
  name: string;
}
let updateUser: Partial<User> = { name: "Bob" };
```

### `Required<T>`

``` ts
type FullUser = Required<User>;
```

### `Pick<T, K>`

``` ts
type UserName = Pick<User, "name">;
```

### `Omit<T, K>`

``` ts
type UserWithoutId = Omit<User, "id">;
```

### `Record<K, T>`

``` ts
type UserRole = Record<string, number>;
let roles: UserRole = { admin: 1, user: 2 };
```

### `Readonly<T>`

``` ts
let readonlyUser: Readonly<User> = { id: 1, name: "Alice" };
// readonlyUser.id = 2; // ❌ lỗi
```

------------------------------------------------------------------------

## 8. **Kiểu nâng cao khác**

### `keyof`

``` ts
type UserKeys = keyof User; // "id" | "name"
```

### `typeof`

``` ts
let person = { name: "Tom", age: 30 };
type PersonType = typeof person;
```

### `Indexed Access Types`

``` ts
type NameType = User["name"]; // string
```

### `Conditional Types`

``` ts
type IsString<T> = T extends string ? true : false;
type A = IsString<string>; // true
type B = IsString<number>; // false
```
