本文将从一个例子开始，带大家串起函数，函数指针，function，仿函数，lambda（并不是说后面的方式出现的时间晚。

我们有这样的一个需求，输出一个int数组中大于10的数的个数。

```cpp
int count_arr(int* st, int* ed) {
    int cnt = 0;
    while (st != ed) {
        if (*st > 10)cnt++;
        st++;
    }
    return cnt;
}
int main(){
    int arr[] = { 1,12,7,13,23 };
    cout << count_arr(arr, arr + 5) << endl;
    return 0;
}
```

如果我是需要找到小于5的个数/大于20的个数......这样的，我们就需要写出一个个的函数

```cpp
int count_arr2(int* st, int ed) {
    //
}

int count_arr3(int* st, int ed) {
    //
}

.....
```

所以我们希望改造我们的函数，多传入一个函数，计算函数返回true的数量。

```cpp
int count_arr(int* st, int* ed,bool(*is)(int)) {
    int cnt = 0;
    while (st != ed) {
        if (is(*st))cnt++;
        st++;
    }
    return cnt;
}
bool big(int x) {
    return x > 10;
}
bool small(int x) {
    return x <= 20;
}
int main(){
    int arr[] = { 1,12,7,13,23 };
    cout << count_arr(arr, arr + 5, big) << endl;
    cout << count_arr(arr, arr + 5, small) << endl;
    return 0;
}
```

感觉很好了。

然后我们用着用着发现，这个东西只能计算int数组的啊，所以我们引入模板

```cpp
template<class T>
int count_arr(T* st, T* ed,bool(*is)(T)) {
    int cnt = 0;
    while (st != ed) {
        if (is(*st))cnt++;
        st++;
    }
    return cnt;
}
bool mod2(int x) {
    return (x % 2) == 0;
}
bool big(double x) {
    return x > 3.14;
}
int main(){
    int iarr[] = { 1,12,8,13,23 };
    cout << count_arr(iarr, iarr + 5, mod2) << endl;
    double darr[] = { 3.141,114,514,1.9,1.9810 };
    cout << count_arr(darr, darr + 5, big) << endl;
    return 0;
}
```

之后我们在函数指针的基础上，多了function

function和函数指针很像 [C++ std::function 和函数指针相比有啥区别吗？](https://www.zhihu.com/question/314660217)

```cpp
template<class T,class F>
int count_arr(T* st, T* ed, F is) {
    int cnt = 0;
    while (st != ed) {
        if (is(*st))cnt++;
        st++;
    }
    return cnt;
}
bool mod2(int x) {
    return (x % 2) == 0;
}
bool big(double x) {
    return x > 3.14;
}
int main(){
    int iarr[] = { 1,12,8,13,23 };
    function<bool(int)> fmod2 = mod2;
    function<bool(double)>fbig = big;
    cout << count_arr(iarr, iarr + 5, fmod2) << endl;
    double darr[] = { 3.141,114,514,1.9,1.9810 };
    cout << count_arr(darr, darr + 5, fbig) << endl;
    return 0;
}
```

后面，我们发现，如果希望计算数组中大于3的个数，大于5的个数，大于7的个数，难道我们要创建 big3,big5,big7...... 这样的函数吗？

一个朴素的想法是，我搞个全局变量，需要用到的时候就修改。

```cpp
int level;
bool big(int x) {
    return x > level;
}
int main(){
    int iarr[] = { 1,12,8,13,23 };
    level = 10;
    cout << count_arr(iarr, iarr + 5, big) << endl;   
    return 0;
}
```

但是这样呢，很容易出现我们忘记去改这个全局变量。所以我们引入仿函数这个概念。

**如果一个东西，用起来像函数，那它就是函数**。

所以我们可以创建一个结构体，里面有个level，然后初始化level，重载()运算符，那么这样，这个结构体就可以当成一个函数来使用了。

```cpp
template<class T>
struct big {
    T level;
    big(T _level) { level = _level; }
    bool operator ()(T num) const {
        return num > level;
    }
};
int main(){
    int iarr[] = { 1,12,8,13,23 };
    cout << count_arr(iarr, iarr + 5, big<int>(3)) << endl;   
    cout << count_arr(iarr, iarr + 5, big<int>(10)) << endl; 
    double darr[] = { 3.141,114,514,1.9,1.9810 };
    cout << count_arr(darr, darr + 5, big<double>(10.00)) << endl;
    return 0;
}
```

用着用着呢，我们就感觉，如果就是为了一件小事，就是全局里面声明一个函数，很不爽（写着写着就要跳到上面去

所以引入了 lambda

我们捕获一个数来当成level，这样我们就不需要创建一个结构体了。

```cpp
int main(){
    int iarr[] = { 1,12,8,13,23 };
    int level = 10;
    cout << count_arr(iarr, iarr + 5, [level](int x) {return x > level; }) << endl;
    return 0;
}
```

我们来看一个lambda的经典应用，sort从大到小。

```cpp
int main(){
    int iarr[] = { 1,12,8,13,23 };
    sort(iarr, iarr + 5, [](int a, int b) {return a > b; });
    for (int x : iarr)cout << x << endl;
    return 0;
}
```

输出

> 23
> 13
> 12
> 8
> 1

我们来看下sort的声明

```cpp
template <class _RanIt, class _Pr>
_CONSTEXPR20 void sort(const _RanIt _First, const _RanIt _Last, _Pr _Pred) { // order [_First, _Last)
    _Adl_verify_range(_First, _Last);
    const auto _UFirst = _Get_unwrapped(_First);
    const auto _ULast  = _Get_unwrapped(_Last);
    _Sort_unchecked(_UFirst, _ULast, _ULast - _UFirst, _Pass_fn(_Pred));
}
```

是不是和我们的count_num很像（指上面那一坨

```cpp
template<class T,class F>
int count_arr(T* st, T* ed, F is) 
```

然后我们看下不传入函数sort是什么样子的

```cpp
template <class _RanIt>
_CONSTEXPR20 void sort(const _RanIt _First, const _RanIt _Last) { // order [_First, _Last)
    _STD sort(_First, _Last, less<>{});
}
```

有个less，这个是啥玩意

```cpp
template <>
struct less<void> {
    template <class _Ty1, class _Ty2>
    _NODISCARD constexpr auto operator()(_Ty1&& _Left, _Ty2&& _Right) const
        noexcept(noexcept(static_cast<_Ty1&&>(_Left) < static_cast<_Ty2&&>(_Right))) // strengthened
        -> decltype(static_cast<_Ty1&&>(_Left) < static_cast<_Ty2&&>(_Right)) {
        return static_cast<_Ty1&&>(_Left) < static_cast<_Ty2&&>(_Right);
    }
    using is_transparent = int;
};
```

虽然可能看不懂，但是这个

```cpp
 operator()(_Ty1&& _Left, _Ty2&& _Right) const
```

告诉了我们这玩意是个仿函数

所以我们sort可以这样

```cpp
struct cmp{   
    bool operator()(int a, int b)const {
        return a > b;
    }
};
int main(){
    int iarr[] = { 1,12,8,13,23 };
    sort(iarr, iarr + 5, cmp());
    for (int x : iarr)cout << x << endl;
    return 0;
}
```