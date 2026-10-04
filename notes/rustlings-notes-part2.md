# Rustlings 学习笔记（二）

这份笔记整理的是第一次生成笔记之后继续学习的内容，按学习顺序排列。

## 1. `mod` 是什么

`mod` 是 module 的缩写，意思是“模块”。

它用来把代码分组：

```rust
mod tests {
    // 测试代码
}
```

可以理解成创建一个小房间，把相关代码放进去。

在 Rustlings 里常见：

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_something() {
        // 测试内容
    }
}
```

这里的 `mod tests` 就是专门放测试代码的模块。

## 2. modules1：模块里的东西默认私有

例子：

```rust
mod sausage_factory {
    fn get_secret_recipe() -> String {
        String::from("Ginger")
    }

    fn make_sausage() {
        get_secret_recipe();
        println!("sausage!");
    }
}

fn main() {
    sausage_factory::make_sausage();
}
```

问题是：`make_sausage` 在模块里面，默认是私有的，模块外面的 `main` 不能调用。

解决：

```rust
pub fn make_sausage() {
    get_secret_recipe();
    println!("sausage!");
}
```

但是 `get_secret_recipe` 不应该加 `pub`，因为题目说不要让模块外面看到秘密配方。

## 3. `pub` 是什么

`pub` 表示 public，公开。

模块里的函数、常量、结构体等，默认只能在模块内部访问。

加上 `pub` 后，模块外面才可以访问。

```rust
pub fn make_sausage() {
    println!("sausage!");
}
```

## 4. `use` 是什么

`use` 用来把一个路径里的东西引入当前作用域。

例如：

```rust
use std::collections::HashMap;
```

之后就可以直接写：

```rust
HashMap::new()
```

不用每次都写完整路径：

```rust
std::collections::HashMap::new()
```

## 5. modules2：`pub use ... as ...`

题目代码类似：

```rust
mod delicious_snacks {
    pub use self::fruits::PEAR as fruit;
    pub use self::veggies::CUCUMBER as veggie;

    mod fruits {
        pub const PEAR: &'static str = "Pear";
        pub const APPLE: &'static str = "Apple";
    }

    mod veggies {
        pub const CUCUMBER: &'static str = "Cucumber";
        pub const CARROT: &'static str = "Carrot";
    }
}
```

这句：

```rust
pub use self::fruits::PEAR as fruit;
```

意思是：

```text
从当前模块 self 里的 fruits 模块中找到 PEAR，
把它引入当前模块，
并改名为 fruit，
再公开给模块外面使用。
```

所以外面可以写：

```rust
delicious_snacks::fruit
```

它实际指向的是：

```rust
delicious_snacks::fruits::PEAR
```

## 6. `::` 是什么

`::` 是路径访问符。

它表示“进入某个模块或类型里面找东西”。

常见例子：

```rust
std::collections::HashMap
Message::Quit
String::from("hello")
delicious_snacks::fruit
```

可以这样理解：

```text
模块::里面的东西
类型::关联函数
枚举::变体
```

## 7. `pub const PEAR: &'static str = "Pear";`

这是一句常量定义。

拆开看：

```rust
pub
```

表示公开。

```rust
const
```

表示常量，值不能改。

```rust
PEAR
```

常量名字。Rust 习惯常量用全大写。

```rust
&'static str
```

表示字符串切片，并且这个字符串能活到程序结束。

```rust
"Pear"
```

字符串字面量。

所以整句意思是：

```text
定义一个公开常量 PEAR，值是字符串 "Pear"。
```

## 8. `&'static str` 和 `'`

你之前常见的是：

```rust
&str
```

`&'static str` 是带生命周期的写法。

拆开：

```text
&        引用
'static  生命周期
str      字符串切片
```

`'static` 表示这个数据能活到程序结束。

字符串字面量比如：

```rust
"Pear"
```

通常就是：

```rust
&'static str
```

这里的 `'` 是生命周期标记，不是字符串，也不是字符。

常见生命周期名字：

```rust
'a
'b
'static
```

现在先记住：

```text
看到 'xxx，通常是在说引用能活多久。
```

## 9. modules3：从标准库引入多个名字

题目需要从 `std::time` 引入：

```rust
SystemTime
UNIX_EPOCH
```

写法：

```rust
use std::time::{SystemTime, UNIX_EPOCH};
```

这样下面就可以直接写：

```rust
SystemTime::now()
UNIX_EPOCH
```

不用写：

```rust
std::time::SystemTime::now()
std::time::UNIX_EPOCH
```

## 10. HashMap 是什么

Rust 里的哈希表叫 `HashMap`。

使用前需要引入：

```rust
use std::collections::HashMap;
```

基本写法：

```rust
let mut map = HashMap::new();

map.insert(String::from("apple"), 3);
map.insert(String::from("banana"), 5);
```

它存的是 key-value：

```text
apple  -> 3
banana -> 5
```

## 11. HashMap 的基本操作

创建：

```rust
let mut map = HashMap::new();
```

插入：

```rust
map.insert(String::from("apple"), 3);
```

读取：

```rust
map.get("apple")
```

遍历：

```rust
for (key, value) in &map {
    println!("{}: {}", key, value);
}
```

注意：

```rust
get()
```

返回的是 `Option`，因为 key 可能不存在。

## 12. hashmaps1：函数最后要返回 HashMap

如果函数定义是：

```rust
fn fruit_basket() -> HashMap<String, u32> {
```

它必须返回：

```rust
HashMap<String, u32>
```

错误情况：

```rust
basket.insert(String::from("watermelon"), 4)
```

如果把 `insert` 放在最后一行且不加分号，Rust 会把 `insert` 的返回值当成函数返回值。

但 `insert` 返回的不是 `HashMap`，而是：

```rust
Option<u32>
```

正确结构：

```rust
fn fruit_basket() -> HashMap<String, u32> {
    let mut basket = HashMap::new();

    basket.insert(String::from("banana"), 2);
    basket.insert(String::from("apple"), 3);
    basket.insert(String::from("watermelon"), 4);

    basket
}
```

重点：

```text
insert 语句后面加分号；
最后单独写 basket，并且不加分号。
```

## 13. `insert` 的返回值

`HashMap::insert` 的作用是插入 key-value。

```rust
map.insert(key, value);
```

但它的返回值是：

```rust
Option<旧值>
```

如果这个 key 原来有值，就返回旧值。

如果这个 key 原来没有值，就返回 `None`。

所以它不是用来返回整个 HashMap 的。

## 14. hashmaps2：保证每种水果至少有一个

题目里已有：

```text
Apple = 4
Mango = 2
Lychee = 5
```

要求：

```text
不能修改已有水果数量；
要保证 Banana 和 Pineapple 也存在；
水果总数大于 11；
每种水果数量不能是 0。
```

已有总数：

```text
4 + 2 + 5 = 11
```

所以给缺失水果各加 1 个就够了。

核心写法：

```rust
for fruit in fruit_kinds {
    basket.entry(fruit).or_insert(1);
}
```

## 15. `entry(...).or_insert(...)`

这是 HashMap 里非常常见的写法。

```rust
basket.entry(fruit).or_insert(1);
```

意思是：

```text
如果 fruit 已经在 basket 里，就不动它；
如果 fruit 不在 basket 里，就插入数量 1。
```

所以它适合这种需求：

```text
有就保留；
没有就放一个初始值。
```

## 16. hashmaps3：统计足球比赛结果

这个程序的作用是：

```text
把多场足球比赛结果统计成一张表。
```

输入每行类似：

```text
England,France,4,2
```

意思是：

```text
England 进 4 球；
France 进 2 球。
```

同时也表示：

```text
England 丢 2 球；
France 丢 4 球。
```

最后要得到：

```rust
HashMap<String, Team>
```

其中：

```rust
String
```

是队名。

```rust
Team
```

存这个队的总进球和总失球：

```rust
struct Team {
    goals_scored: u8,
    goals_conceded: u8,
}
```

## 17. hashmaps3：解析每一行比赛结果

代码：

```rust
for r in results.lines() {
    let v: Vec<&str> = r.split(',').collect();
    let team_1_name = v[0].to_string();
    let team_1_score: u8 = v[2].parse().unwrap();
    let team_2_name = v[1].to_string();
    let team_2_score: u8 = v[3].parse().unwrap();
}
```

假设一行是：

```text
England,France,4,2
```

这句：

```rust
r.split(',').collect()
```

会得到：

```rust
["England", "France", "4", "2"]
```

然后：

```rust
v[0] -> "England"
v[1] -> "France"
v[2] -> "4"
v[3] -> "2"
```

`parse().unwrap()` 把字符串数字转成真正的数字：

```rust
"4" -> 4
```

## 18. hashmaps3：为什么要先放初始值

因为同一个队可能出现多次。

例如：

```text
England,France,4,2
Germany,England,2,1
```

England 出现了两次。

第一次看到 England 时，HashMap 里还没有它，所以要插入初始值：

```rust
Team {
    goals_scored: 0,
    goals_conceded: 0,
}
```

第二次再看到 England 时，不能重新插入 `0,0`，否则第一场的数据会丢。

所以逻辑是：

```text
没有这个队：创建初始记录；
有这个队：拿出旧记录继续累加。
```

这就是 `entry(...).or_insert(...)` 的用途。

## 19. hashmaps3：更新球队数据

队 1 的逻辑：

```text
进球 += 队 1 进球
失球 += 队 2 进球
```

队 2 的逻辑：

```text
进球 += 队 2 进球
失球 += 队 1 进球
```

代码：

```rust
let team_1 = scores.entry(team_1_name).or_insert(Team {
    goals_scored: 0,
    goals_conceded: 0,
});
team_1.goals_scored += team_1_score;
team_1.goals_conceded += team_2_score;

let team_2 = scores.entry(team_2_name).or_insert(Team {
    goals_scored: 0,
    goals_conceded: 0,
});
team_2.goals_scored += team_2_score;
team_2.goals_conceded += team_1_score;
```

## 20. `unwrap()` 是什么

你问到的 `unwarp` 实际应该是 `unwrap`。

例子：

```rust
let score: u8 = "4".parse().unwrap();
```

`parse()` 尝试把字符串转成数字。

它可能成功，也可能失败，所以返回 `Result`。

大概像这样：

```text
成功：Ok(4)
失败：Err(...)
```

`unwrap()` 的意思是：

```text
如果是 Ok，就把里面的值取出来；
如果是 Err，就直接 panic。
```

所以：

```rust
"4".parse::<u8>().unwrap()
```

结果就是数字：

```rust
4
```

Rustlings 里经常可以用 `unwrap()`，因为题目给的数据通常是确定正确的。

实际项目中一般要更谨慎地处理错误。

## 21. 今日重点

1. `mod` 用来组织代码。
2. 模块里的东西默认私有，外面要用就加 `pub`。
3. `use` 用来引入路径，少写长路径。
4. `::` 用来访问模块、类型、枚举里的东西。
5. `HashMap` 是 key-value 表。
6. `insert` 是插入，但返回的是旧值的 `Option`，不是整个表。
7. `entry().or_insert()` 适合“有就拿出来，没有就插入初始值”。
8. 做累计统计时，常用 `HashMap + entry().or_insert()`。
9. `unwrap()` 是从 `Ok` 或 `Some` 里直接取值，失败会 panic。

