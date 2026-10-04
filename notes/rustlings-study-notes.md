# Rustlings 学习笔记

这份笔记整理了我们一起做 Rustlings 时聊到的内容：怎么判断当前题、怎么读报错，以及 `struct`、`enum`、`match`、`String` 等基础语法。

## 1. `bash setup.sh` 之后为什么会报题目错误

`setup.sh` 不只是配置环境，它最后还会自动启动练习检查器：

```bash
cargo run -- watch
```

所以看到类似下面的输出，不代表环境坏了：

```text
Compiling of exercises/structs/structs1.rs failed!
Welcome to watch mode!
```

这表示环境已经跑起来了，Rustlings 正在检查练习，并且当前卡在某一道未完成的题。

## 2. watch 模式是什么

运行：

```bash
cargo run -- watch
```

会进入 watch 模式。它会一直开着，自动检查当前练习。

常用命令：

```text
hint    查看当前题提示
quit    退出 watch 模式
```

注意：在 watch 模式里不要输入完整命令：

```text
rustlings hint strings1
```

这个会报 `unknown command`。如果已经退出 watch 模式，在普通终端里应该用：

```bash
cargo run -- hint strings1
```

## 3. 怎么判断当前要做哪道题

看 watch 输出里的文件路径。

比如：

```text
Compiling of exercises/strings/strings1.rs failed!
```

当前要做的就是：

```text
exercises/strings/strings1.rs
```

如果 watch 不再卡在某个文件，而是进入下一题，说明上一题已经通过。

还可以用：

```bash
cargo run -- run next
cargo run -- list
```

## 4. 怎么判断一道题做好了

一般看两个信号：

1. 保存后 watch 不再报同一道题。
2. 如果文件里有下面这一行，做完后删掉：

```rust
// I AM NOT DONE
```

单独检查某题可以用：

```bash
cargo run -- run 题目名
```

例如：

```bash
cargo run -- run structs2
```

## 5. 测试代码怎么看

Rustlings 里常见测试代码：

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn your_order() {
        assert_eq!(your_order.count, 1);
    }
}
```

含义：

- `#[cfg(test)]`：只有运行测试时才编译这个模块。
- `mod tests`：定义一个测试模块。
- `use super::*`：把外层定义的结构体、函数等拿进来用。
- `#[test]`：标记下面这个函数是测试函数。
- `assert_eq!(a, b)`：检查 `a` 和 `b` 是否相等。

通常不要改测试代码，而是根据测试要求补上方的 TODO。

## 6. struct：结构体

`struct` 用来把多个字段放在一起。

```rust
struct Order {
    name: String,
    year: u32,
    count: u32,
}
```

创建结构体：

```rust
let order = Order {
    name: String::from("Hacker in Rust"),
    year: 2019,
    count: 1,
};
```

访问字段：

```rust
order.name
order.count
```

### 结构体更新语法

如果想基于已有结构体创建一个新结构体，只改几个字段：

```rust
let your_order = Order {
    name: String::from("Hacker in Rust"),
    count: 1,
    ..order_template
};
```

意思是：

```text
name 和 count 用新值；
其他字段从 order_template 来。
```

## 7. impl：给类型实现方法

`impl` 是 implementation 的缩写，意思是“给某个类型实现功能”。

```rust
impl Package {
    fn get_fees(&self, cents_per_gram: i32) -> i32 {
        self.weight_in_grams * cents_per_gram
    }
}
```

`struct` 负责定义数据长什么样，`impl` 负责定义这个数据能做什么。

调用方法：

```rust
package.get_fees(3);
```

## 8. self 是谁

规则：

```text
谁调用这个方法，self 就是谁。
```

例如：

```rust
state.process(Message::Quit);
```

点号左边是 `state`，所以 `process` 方法里的 `self` 就是 `state`。

Rust 会把它理解成类似：

```rust
State::process(&mut state, Message::Quit);
```

所以：

```text
self    <- state
message <- Message::Quit
```

## 9. `&self`、`&mut self`、`self`

可以先这样记：

```text
&self      借来看，只读
&mut self  借来改，可以修改字段
self       直接拿走所有权
```

只读取字段：

```rust
fn is_quit(&self) -> bool {
    self.quit
}
```

修改字段：

```rust
fn quit(&mut self) {
    self.quit = true;
}
```

如果方法里要改结构体字段，就需要 `&mut self`。

## 10. enum：枚举

`enum` 用来表示“一个值只能是几种情况之一”。

```rust
enum Message {
    Quit,
    Echo,
    Move,
    ChangeColor,
}
```

值可以是：

```rust
Message::Quit
Message::Echo
Message::Move
Message::ChangeColor
```

### 带数据的 enum

枚举变体可以携带数据：

```rust
enum Message {
    ChangeColor(u8, u8, u8),
    Echo(String),
    Move(Point),
    Quit,
}
```

含义：

- `ChangeColor(u8, u8, u8)`：带三个颜色值。
- `Echo(String)`：带一个字符串。
- `Move(Point)`：带一个位置。
- `Quit`：不带数据。

## 11. match：模式匹配

`match` 可以理解成更强的 `switch`。

```rust
fn process(&mut self, message: Message) {
    match message {
        Message::ChangeColor(r, g, b) => self.change_color((r, g, b)),
        Message::Echo(s) => self.echo(s),
        Message::Move(p) => self.move_position(p),
        Message::Quit => self.quit(),
    }
}
```

含义：

```text
看 message 是哪一种 Message；
如果是 ChangeColor，就拆出 r、g、b；
如果是 Echo，就拆出字符串 s；
如果是 Move，就拆出点 p；
如果是 Quit，就执行 quit。
```

重点：

```rust
Message::ChangeColor(r, g, b)
```

这里不是创建新消息，而是在匹配并拆开已有消息。

如果已有：

```rust
Message::ChangeColor(255, 0, 255)
```

那么匹配后：

```text
r = 255
g = 0
b = 255
```

## 12. tuple 参数的小坑

如果函数定义是：

```rust
fn change_color(&mut self, color: (u8, u8, u8)) {
    self.color = color;
}
```

它要的是一个三元组，不是三个单独参数。

错误：

```rust
self.change_color(r, g, b)
```

正确：

```rust
self.change_color((r, g, b))
```

外层括号是函数调用，内层括号是 tuple。

## 13. String 和 &str

常见区别：

```text
String  拥有字符串数据，可增长、可修改
&str    字符串切片，通常是借来的字符串视图
```

字符串字面量：

```rust
"blue"
```

类型是：

```rust
&str
```

如果函数要求返回 `String`：

```rust
fn current_favorite_color() -> String {
    "blue".to_string()
}
```

或者：

```rust
fn current_favorite_color() -> String {
    String::from("blue")
}
```

## 14. `expected String, found &str`

报错：

```text
expected `String`, found `&str`
```

意思是：这里需要 `String`，但你给了字符串切片 `&str`。

常见修法：

```rust
"blue".to_string()
String::from("blue")
```

## 15. `expected String, found ()`

如果你写：

```rust
fn trim_me(input: &str) -> String {
    input.trim().to_string();
}
```

会报类似：

```text
expected `String`, found `()`
```

原因是最后一行有分号。Rust 里函数最后一行没有分号时，才会作为返回值。

正确：

```rust
fn trim_me(input: &str) -> String {
    input.trim().to_string()
}
```

也可以显式 `return`：

```rust
fn trim_me(input: &str) -> String {
    return input.trim().to_string();
}
```

## 16. strings3 常用字符串方法

去掉首尾空白：

```rust
input.trim().to_string()
```

拼接：

```rust
format!("{} world!", input)
```

替换：

```rust
input.replace("cars", "balloons")
```

## 17. 判断表达式是 `String` 还是 `&str`

常见规则：

```text
字符串字面量              -> &str
trim()                    -> &str
字符串切片                 -> &str
to_string()               -> String
to_owned()                -> String
String::from(...)         -> String
format!(...)              -> String
replace(...)              -> String
to_lowercase()            -> String
```

例子：

```rust
string_slice("blue");                                // &str
string("red".to_string());                           // String
string(String::from("hi"));                          // String
string("rust is fun!".to_owned());                   // String
string("nice weather".into());                       // String
string(format!("Interpolation {}", "Station"));      // String
string_slice(&String::from("abc")[0..1]);            // &str
string_slice("  hello there ".trim());               // &str
string("Happy Monday!".to_string().replace("Mon", "Tues")); // String
string("mY sHiFt KeY iS sTiCkY".to_lowercase());    // String
```

## 18. 读 Rust 报错的顺序

推荐顺序：

1. 看当前失败文件：

```text
Compiling of exercises/strings/strings1.rs failed!
```

2. 看错误类型：

```text
error[E0308]: mismatched types
```

3. 看位置：

```text
--> exercises/strings/strings1.rs:15:5
```

4. 看 Rust 期望什么、你给了什么：

```text
expected `String`, found `&str`
```

5. 看 help，但不要盲目照抄。Rustlings 里很多时候应该改类型定义，而不是强行转换。

例如 enums3 里：

```text
expected `u8`, found `i32`
```

更合理的修法是把枚举定义从：

```rust
ChangeColor(i32, i32, i32)
```

改成：

```rust
ChangeColor(u8, u8, u8)
```

而不是马上写 `try_into().unwrap()`。
