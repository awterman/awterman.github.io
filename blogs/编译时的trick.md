# 编译时的trick
yuxuan 2019-3-24

## Golang调度
在当前版本(go v1.12)goroutine一旦开始执行，只能在函数调用的时候被抢占，Golang编译器会给每个函数加一个标志位，在调用时判断是否需要让出CPU。


## C语言switch语句的实现
> 参考：[switch语句的几种反汇编结构及效率分析](https://blog.csdn.net/Apollon_krj/article/details/76793914)

在翻译为汇编语句时，至少有两种策略，下面进行简单说明。

### 普通结构
即翻译为if...else if...
### 查表
switch的参数为int，可以简单的映射到指令地址上。例如：
```c
switch(a) {
case 31: f31(); break;
case 32: f32(); break;
case 34: f34(); break;
default: break;
}
```
定义`table = {f31, f32, default_f, f34}`，这段switch语句就可翻译为`a-30>2? default_f() : table[a-30]()`  
这里有一个trick，即将中间空出的`33`用default_f填充。

#### 查表优化
- 在间隔较大的情况下，如：
```c
switch(a) {
case 11: f11(); break;
case 21: f21(); break;
case 31: f31(); break;
case 32: f32(); break;
// ...
}
```
这里继续使用上面的方法，会出现`table = {f11, f22, default_f, default_f... , f31, f32}`，由于table中的函数都是内联的汇编指令，这样就形成较大的内存浪费。  
所以采用如下方式进行压缩:
```
table = {f11, f22, f31, f32}
table2 = {0, 1, ..., 30, 31} // 为节省内存，使用[]int8类型，因此方式只适用于间隔小于256的情况。
```
这样switch语句可翻译为`table[ table2[a-10] ]()`

- 间隔更大的情况下，使用树形结构，不再详述。

#### 一般适用场景
- if-else结构：分支少于等于4条时。
- 查表：视间隔而定，是否采用**大表结构**，**大表+小表结构**，**树形结构**。

## C++中的析构函数
析构函数的“自动”调用，实际上是通过编译器在代码块的结尾加入析构函数调用实现的。
