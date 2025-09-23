# react-hook-form

- quản lý form hiệu năng cao
  `npm install react-hook-form`

## 1. rules

```jsx
// 1 required: Bắt buộc nhập.
<input
  {...register("name", { required: "Name is required" })}
  placeholder="Name"
/>

// 2 minLength / maxLength: Kiểm tra số ký tự tối thiểu / tối đa

<input
  {...register("username", {
    minLength: { value: 3, message: "Min 3 chars" },
    maxLength: { value: 10, message: "Max 10 chars" }
  })}
/>

// 3 min / max: Kiểm tra giá trị số tối thiểu / tối đa

<input
  type="number"
  {...register("age", {
    min: { value: 18, message: "Tuổi phải >= 18" },
    max: { value: 60, message: "Tuổi phải <= 60" }
  })}
/>

// 4 pattern: Kiểm tra theo regex.

<input
  {...register("email", {
    pattern: { value: /^[\w-.]+@([\w-]+\.)+[\w-]{2,4}$/, message: "Email không hợp lệ" }
  })}
/>

// 5 validate
//    - Custom validation function.
//	- Trả về true nếu hợp lệ, hoặc string lỗi nếu không.

<input
  type="number"
  {...register("score", {
    validate: value => value >= 50 || "Score phải >= 50"
  })}
/>
// hoặc

<input
  {...register("number", {
    validate: {
      isEven: v => v % 2 === 0 || "Phải là số chẵn",
      lessThan100: v => v < 100 || "Phải < 100"
    }
  })}
/>

/* 6 valueAsNumber / valueAsDate / setValueAs
	- valueAsNumber: tự convert input thành số
	- valueAsDate: convert thành Date object
	- setValueAs: transform giá trị trước khi validate
*/

<input
  type="number"
  {...register("age", { valueAsNumber: true, min: 0 })}
/>

<input
  type="text"
  {...register("date", { valueAsDate: true })}
/>

<input
  {...register("price", { setValueAs: v => parseFloat(v) })}
/>

// 7 disabled: Rule mặc định nếu input disabled sẽ không validate.

<input {...register("field", { disabled: true })} />

// 8 deps: Dùng khi rule phụ thuộc vào các field khác.

<input
  type="password"
  {...register("confirmPassword", {
    validate: value => value === watch("password") || "Passwords must match",
    deps: ["password"]
  })}
/>
```
## 2. handleSubmit

có thể nhận hàm `async function` để xử lý các state

## 3. formState

Trong React Hook Form (RHF), `formState` là một object chứa **trạng thái
hiện tại của form**.\
Nó giúp xác định form đang ở trạng thái nào: có dirty không, đã submit
chưa, có lỗi không...

### Các thuộc tính của formState

***Không cần state isLoading, tận dụng các state này***

---

`isDirty` boolean `true` nếu có ít
nhất một field
thay đổi giá trị
so với
`defaultValues` luôn `false` sau khi reset

`dirtyFields` object Liệt kê các field
nào đã bị thay
đổi.

`touchedFields` object Field nào đã được
"focus + blur"
(người dùng đã
chạm vào).

`isSubmitted` boolean `true` nếu form đã
submit ít nhất 1
lần.

`isSubmitting` boolean `true` trong lúc
form đang xử lý
submit async.

`isSubmitSuccessful` boolean `true` nếu submit
thành công (không
lỗi).

`submitCount` number Số lần submit
form.

`isValid` boolean `true` nếu toàn bộ
form hợp lệ (dựa
trên
rules/schema).

`errors` object Chứa lỗi của các
field (nếu có).

`defaultValues` object Giá trị mặc định
ban đầu. khi reset sẽ thay đổi, nếu dùng `reset()` thì sẽ giữ nguyên nhưng isDirty sẽ là `false`

### Khi nào dùng formState?

- Disable nút Submit cho tới khi form hợp lệ (`isValid`) hoặc đã thay
  đổi (`isDirty`).
- Hiển thị loading khi submit async (`isSubmitting`).
- Thông báo kết quả submit (`isSubmitSuccessful`).
- Đếm số lần submit (`submitCount`).
- Theo dõi field nào người dùng đã chạm vào (`touchedFields`).

## 4. useForm

- `useForm({mode: 'onSubmit'})`: ✅ (Mặc định) Validation chỉ chạy khi submit form (handleSubmit).
- `useForm({mode: 'onBlur'})`:
Validation chạy khi người dùng rời khỏi input.
- `useForm({mode: 'onBlur'})`: Validation chạy ngay khi input thay đổi.
- `useForm({mode: 'onTouched'})`: Validation chạy khi input bị chạm vào và blur.
- `useForm({mode: 'all'})`: Kết hợp: chạy validation cả blur + change + submit.
- `useForm({mode: 'onChange'})`: Validation sẽ chạy mỗi khi giá trị input thay đổi.

Sau khi nhấn submit thì nó sẽ chuyển thành 'onChange' chủ động thay đổi bằng state

```jsx
const [mode, setMode] = useState("onSubmit")
const [disable, setDisable] = useState(false) // Disable form
const form = useForm({mode, disable})
```

## 5. useFieldArray

Thường áp dụng cho:
-	Danh sách item có thể thêm / sửa / xóa (vd: số điện thoại, email, sản phẩm, địa chỉ…).
-	Biểu mẫu nhiều hàng lặp lại cùng cấu trúc.

```tsx
import React from "react";
import { useForm, useFieldArray, Controller } from "react-hook-form";

type Todo = { task: string; done: boolean };
type FormValues = { todos: Todo[] };

export default function TodoList() {
  const { control, handleSubmit } = useForm<FormValues>({
    defaultValues: {
      todos: [
        { task: "Học React Hook Form", done: false },
        { task: "Làm todo list", done: true },
      ],
    },
  });

  const { fields, append, remove } = useFieldArray({
    control,
    name: "todos",
  });

  const onSubmit = (data: FormValues) => {
    console.log("Todo List:", data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} style={{ maxWidth: 400, margin: "20px auto" }}>
      <h2>📋 Todo List (Controller)</h2>

      {fields.map((field, index) => (
        <div
          key={field.id}
          style={{
            display: "flex",
            alignItems: "center",
            marginBottom: "10px",
            gap: "10px",
          }}
        >
          {/* Checkbox done */}
          <Controller
            control={control}
            name={`todos.${index}.done`}
            render={({ field }) => (
              <input
                type="checkbox"
                checked={field.value}
                onChange={(e) => field.onChange(e.target.checked)}
              />
            )}
          />

          {/* Editable text */}
          <Controller
            control={control}
            name={`todos.${index}.task`}
            rules={{ required: "Không được để trống" }}
            render={({ field }) => (
              <input
                {...field}
                placeholder="Nhập việc cần làm"
                style={{
                  flex: 1,
                  padding: "4px 8px",
                  border: "1px solid #ccc",
                  borderRadius: "4px",
                }}
              />
            )}
          />

          {/* Remove button */}
          <button type="button" onClick={() => remove(index)}>
            ❌
          </button>
        </div>
      ))}

      {/* Add new task */}
      <button
        type="button"
        onClick={() => append({ task: "", done: false })}
        style={{ marginTop: "10px" }}
      >
        ➕ Thêm việc
      </button>

      <div style={{ marginTop: "20px" }}>
        <button type="submit">💾 Lưu Todo</button>
      </div>
    </form>
  );
}
```

## 6. trigger 
dùng để check validate cho field nhất định ở từng trường hợp cụ thể (khác handleSubmit là ko kích hoạt `mode: 'onChange'`)


## 7. createFormControl 

Tạo context sử dụng trên nhiều component khác nhau mà ko cần truyền qua nhiều nơi, ứng dụng có form phức tạp