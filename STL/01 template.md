## 1. 模板起源

### C语言的中的宏

- 预处理阶段进行纯文本替换
- 不会进行数据类型检查

### 利用宏构建通红函数框架（宏和函数组合在一起）

- 通过实例化宏， 让预处理器将这个宏带换成针对不同数据类型的真正函数

- 将宏的通用性和函数的类型安全性结合

  ```
  #define MAX(T) T max_##T(T x,T y){\
  					return x>y?x:y;
  				\}
  				
  MAX(int)//由预编译器生成的函数定义
  // int max_int(int x,int y){return .....}
  
  #define Max(T) max_##T
  
  int main()
  {
  	Max(int)(10,20);
  
  }
  ```

  

