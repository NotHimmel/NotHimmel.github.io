# Reference
https://course.rs/first-try/hello-world.html

## 基础
let a = "hello_world" 绑定变量与所有权转移

默认变量不可变，mut可变
let mut x = 5;
x = 6;

带下划线开头的变量，就算不使用也不会warning
let _x = 5;

常量用const声明，可以声明在全局范围
const MAX_POINTS: u32 = 100_000;

数值类型
浮点数只是近似，不能用于判等

### [所有权](https://course.rs/basic/ownership/ownership.html)
基本数据类型只是栈上值的拷贝、赋值
复杂数据类型如string，是所有权的转移，深拷贝需用clone()
不可变引用的copy并非所有权转移，原引用仍可使用。

### [复合类型](https://course.rs/basic/compound-type/intro.html)
