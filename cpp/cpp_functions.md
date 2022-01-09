## Functions 

Functions are nothing but addresses in memory which can be called at runtime. In C++ it is possible to create function pointers, which are functions prototypes pointing to memory addresses, making the pointer to be invokable in the code.

### Function Pointers

Functions can be accessed by pointers. There are many ways a function pointer can declared: 

```c++
int (*foo_ptr) (int, int) = &foo;   	// valid; 
int (*foo_ptr) (int, int) = foo;    	// valid; 
int (*foo_ptr) (int, int) = {foo};  	// valid; brace initialization
int (*foo_ptr) (int, int) = {&foo};  	// valid; brace initialization
```

Function pointers can also be declared using auto type deduction as follows:

```c++
auto foo_ptr = foo; 	// valid; auto type deduction
auto foo_ptr = &foo; 	// valid;
auto* foo_ptr = &foo; 	// valid;
auto* foo_ptr = foo; 	// valid;
```

### Callback Functions and Type Aliases

Functions can be passed in as parameters to other functions.

```c++
void foo(void(*bar)(int)){}
void bar(int x){}
foo(bar);
```

The same could be rewritten using type aliases with `using` keyword. This would help to make the code more readable and easier to maintain in some cases.

```c++
using bar_ptr = void(*)(int);
void foo(bar_ptr bar){}
void bar(int x){}
```

### Functors

Functors are objects that override the operator `()`. This allows the object to be invoked as a function call. 

```c++
struct Foo {
    int operator () (){ return 42; }
};
Foo foo;
foo();	// 42
```
#### Functors and Lamba Functions

C++ lambdas are internally anonymous classes designed to execute as functors. 

```c++
int a = [] (int x, int y) { return x + y };
```

The lamba would be defined as an anonymous class at compile time:

```c++
struct Lambda_981298392 {
	auto operator() (int x, int y) const { return x + y };
}
int a = Lambda_981298392(x, y);
```

### Lamba Functions Callbacks

Lambda can be used as callbacks by taking advantage of the `operator()`. This allows to lambas as function callbacks interchangeably.

```c++
void foo(void(*cb)(int)){ cb(42); }
void bar(int a){ std::cout << a; }
auto baz = [](int a) { std::cout << a; };
int main(){
   foo(bar);
   foo(baz);
}
```

The above will work just fine.

### Lambda and Classes

#### Capture by Value

When capturing data by value, the compiler will generate a class with a functor. This class will have the variables captured passed in by value during the constructor and stored using private access.

```c++
int a{1};
int b{2};

auto foo = [a, b] (int x, int y){};
foo(a,b);
```

The above code will be compiled to something similar to the class below:

```c++
class lambda_1234567890{
public:
    lambda_1234567890(int a, int b) : a(a), b(b){}
    auto operator() (int a, int b) const {}
private:
    int a;
    int b;
};
```

When capturing all, the compiler will setup only the members used in the lambas, thus generate an anonymous class that with only the data required. When capturing by value, the lamba functor will be declared using const, thus any modification within the lambda will generate a compile error.

In order to modify the data inside the lamba, it is possible to flag the lamba as `mutable` as follows:

```c++
auto foo = [a, b] (int x, int y) mutable {}
```

However, this will not modify the variable outside of its scope, thus when leaving the functor, the variable `a` will keep its original value.

#### Capture by Reference

Capturing by reference would generate a class with the ability to modify the variables within the lambda.

```c++
int a{1};
int b{2};

auto foo = [&a, &b] (int x, int y){ a++; };
foo(a,b);
```

The compiler would generate a functor that will capture the data as reference, allowing the captured values to be modified and reflected accordingly.

```c++
class lambda_1234567890{
public:
    lambda_1234567890(int& a, int& b) : a(a), b(b){}
    auto operator() (int a, int b) {}
private:
    int& a;
    int& b;
};
```

#### Capturing the This Pointer

When dealing with lambda within classes, in order to use the `this` pointer of the outer class, it should be captured as follows:

```c++
struct Foo {
    auto foo = [this](){}
};
```

### std::function

This is a C++ utility to work with multiple type of callbacks: function pointers, functors, lamba functions. It would act as a sort of a wrapper to be able to accept all type of function types available while keeping the code easy to read and to maintain.

```c++
void foo(std::function<void(int)> callback){
    callback(42);
}

void bar(int a){ std::cout << a;}
auto baz = [](int a){ std::cout << a; };
struct Qux { void operator () (int a){ std::cout << a; } };

int main(){
    foo(bar);
    foo(baz);
    Qux qux;
    foo(qux);
} 
```