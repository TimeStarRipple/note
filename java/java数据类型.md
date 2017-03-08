# Java数据类型

## 标识符
命名规范：字母、数字、$、_(下划线)，但不可用数字开头，但不可以是关键字

关键字：Java的关键字对java的编译器有特殊的意义，他们用来表示一种数据类型，或者表示程序的结构等。（注：true，false，null，sizeof，这只是值和方法名）

保留字：为java预留的关键字。现在还没用到，但是在升级版本中可能作为关键字。

关键字列表：(依字母排序共51组)

```
abstract, assert,boolean, break, byte, case, catch, char, class, const, continue, default, do, double, else, enum,extends, final, finally, float, for, if, implements, import, instanceof, int, interface, long, native, new,package, private, protected, public, return, short,static, strictfp, super,switch, synchronized,this, throw, throws, transient,
try, void, volatile, while
```
保留字列表 (依字母排序共14组) :
```
byValue, cast, false, future, generic, inner, operator, outer, rest, true, var , goto ，const,null
```


## 八种基本类型数据
- byte：1字节
- short：2字节
- int：4字节
- long：8字节
- float：4字节
- double：8字节
- char：2字节
- boolean：作为int处理时4字节，作为数组的时候1个字节（没有明确的指定）

### byte和char的区别：
1. byte是一个字节的，char是两个字节的，
2. byte可以为负数，但是char不能是负数
3. byte用于存数字，char用于存字符

### 初始化
局部变量在使用前必须初始化，局部变量存在栈空间中，并不会初始化
静态变量可以不用初始化，类加载器会默认初始化，初始化结果如下：
byte，short，int，long ： 0
float，double：0.0
boolean：false

### 数据类型判定
- 有声明的数据类型，数据类型就是声明类型
- 加标签的数据类型，“L”标签代表long，“f”为float
- 常量，整型默认是int，浮点型默认是double

### 运算
> 注：1. boolean数据类型不能参与运算与其他类型数据的赋值. 2.常量间运算，编译器会直接运算结果，不要我们考虑类型转换，结果是整型就是int，是浮点型就是double

#### 运算方式
计算机采用补码的方式对数据进行运算
原因：
1. 可以将符号位和数值域统一处理；同时，加法和减法也可以统一处理。
2. 补码与原码相互转换，其运算过程是相同的，不需要额外的硬件电路。

#### 运算结果数据类型
先要确定所有数据的类型，然后按照下面的运算规则运算

如果当前有double类型数据，表达式结果为double，否则查找存在float类型的结果，表达式结果为float，否则查找存在long类型数据，表达式结果为long，否则表达式结果为int

#### 运算结果
如果数据运算发生溢出，那么溢出的结果丢弃

## 赋值
这里分两种情况讨论：
1. 赋值的不是含变量的语句：一般情况整型是无法转型为byte，short和char，但是只要结果在相应数据类型的存储范围内，就不会发现编译错误。但是整型不能超过自己的限制，如果是long类型，就要加上标签`L`，如果是浮点型给float赋值，要加上`f`，否则会编译报错。
2. 赋值的是含变量语句：观察运算结果的数据类型，判定赋值后是否会发生精度丢失，丢失则编译报错。

精度范围判定如下：
double > float > long > int > short > byte
double > float > long > int > char
boolean不参与其他数据类型的赋值转换

注：这里之所以那是否含变量来，是因为运算只有常量的话，编译器会直接用结果替代语句

### 比较
浮点型和某个数进行比较的时候，不能直接用==，因为java赋予的值不是精确的，要用`Math.abs()`方法取绝对值，转化之后判定小于0.0001f

## 八种包装类型
Character：char的包装类
Byte：byte的包装类
Short：short的包装类
Integer：int的包装类
Long：long的包装类
Boolean：boolean的包装类
Float：float的包装类
Double：double的包装类

常量池：Java的8种基本类型(Byte, Short, Integer, Long, Character, Boolean, Float, Double), 除Float和Double以外, 其它六种都实现了常量池, 但是它们只在大于等于-128并且小于等于127时才使用常量池。

当我们给Integer赋值时，实际上调用了Integer.valueOf(int)方法（编译期进行装箱）

下面的语句不会缓存常量：
Integer integer = new Integer(127)；
因为使用了new关键字，一定是创建了一个新的对象，无法进行缓存优化。

## String
基本类型转字符串：String.valueOf()
字符串转基本类型：Integer.parseInt()，Long.parseLong()。。。
字符串转包装类：Integer.getInteger()，Long.getLong()。。。


