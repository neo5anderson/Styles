#### 特性

##### 智能指针

+ 优先使用 `scoped_ptr`，安全第一
+ 特殊情况使用 `std::tr1::shared_ptr`
+ 任何情况都不要用 `auto_ptr`
+ 设计对象隶属明确的代码，最明确的对象隶属是不适用指针

##### 引用参数

+ 必须使用 `const` 修饰，强调不是拷贝出来的
+ 在对象生命周期内一直存在或特定接口时可用常数指针代替

##### 函数重载

+ 输入参数类型不同，功能相同时再重载
+ 不要模仿使用默认函数参数
+ 尽可能让函数名包含参数信息

##### 友元

+ 定义在同一文件目录内，以 `Builder` 结尾
+ 常用于做类的单元测试
+ 不打破类封装的边界，也减少了不必要的 `public` 成员

##### 类型转换

+ 不再使用 C 的强制类型转换
+ 除测试外，不使用 `dynamic_cast`

##### 流

+ 日志接口中可选择使用
+ 不要在代码中使用流，保持一致性

##### 前置自增减

因为后置会多做一次拷贝，所以迭代器和模板对象应使用前置

##### const

任何可以使用的情况下都使用 `const`，推荐添加在描述名词前面

##### 整数

+ 使用 `<stdint.h>` 指定精确宽度的整数
+ 无符号类型只在表示位组时使用
+ 如果数值非负，使用 `assert` 判断
+ C 整数中只使用 `int`，适当使用 `size_t` 和 `ptrdiff_t`

##### 64 位可移植性

+ `sizeof(void*)` 不再等于 `sizeof(int)`，整数指针的大小要用 `intptr_t`
+ 64 位上拥有 `[u]int64` 的类或结构体都是 8 字节对齐
+ 创建 64 位常量时使用 `LL` 或 `ULL` 作为后缀
+ 尽量不要让 32 位和 64 位系统的代码不同
+ C99 定义了 `printf` 的可移植版本，但 MSVC 7.1 并不完善，还需要自定义： 

```cpp
// printf macros for size_t
#ifdef _LP64
#define __PRIS_PREFIX "z"
#else
#define __PRIS_PREFIX
#endif

// macros after %
#define PRIds __PRIS_PREFIX "d"
#define PRIxs __PRIS_PREFIX "x"
#define PRIus __PRIS_PREFIX "u"
#define PRIXs __PRIS_PREFIX "X"
#define PRIos __PRIS_PREFIX "o"
```

前后对照

| 类型      | 不要使用          | 使用      | 备注      |
|:---------:|:-----------------:|:---------:|:---------:|
| void*     | %lx               | %p        |           |
| int64_t   | %qd, %lld         | %"PRId64" |           |
| uint64_t  | %qu, %llu, %llx   | %"PRIu64" |           |
| size_t    | %u                | %"PRIuS"  | C99: %zu  |
| ptrdiff_t | %d                | %"PRIdS"  | C99: %zd  |

##### 预处理宏

+ 尽量用内联、枚举和常量代替
+ 最好不用条件编译
+ 不要在头文件中定义宏
+ 正确的使用 `#define` 和 `#undef`
+ 尽量选择一个不会冲突的宏名

##### sizeof

尽可能用 `sizeof(varname)` 代替 `sizeof(type)`，解决变更类型的同步问题

##### 禁止使用列表

+ 默认函数参数
+ 变长数组
+ `alloca()`，用安全的分配器代替，比如 `scoped_ptr`/`scoped_array`
+ 异常
+ 运行时类型识别

