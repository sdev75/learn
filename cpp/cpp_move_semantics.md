## Move Semantics

The `move semantics` were introduced from C++11 as a way to instruct the compiler to take advantage of temporary objects without throwing away the data, making the program more memory efficient. It allows to move ownership of memory from one object to another, including temporaries that the compiler generates to fill out the data before being discarded.

### Lvalues and Rvalues

In C++ everything that has an address or can be addressed in memory is considered to be an `lvalue`. Whereas, everything that cannot be addressed is considered an `rvalue`.

```c++
int foo { 1 + 2 };	// l-value
```

In the case above, `foo` is considered an `lvalue`, because it has a name and it is addressable using `&foo`. However, the expression `1 + 2` is an `rvalue`. It's a transiet, temporary value which will go out of scope once the statement `1+2` is executed.  

Let's consider a case scenario when an object is passed as parameter in a function call. In this case, the compiler will invoke the function and call the copy constructor of the object.

```c++
/**
 * foo_param would be constructed using the copy constructor operator
 */
void a(Foo foo){}	
int main(){
    /**
     * foo is constructed on the main stack frame
     */
    Foo foo{};

    /**
     * A temporary object will be copy constructed from foo
     * and passed in as parameter to a(). 
     * The temporary will construct a copy with all the data. 
     * If the data is huge, it would affect performance
     */
    a(foo);

    /**
     * The temporary is destroyed when a() is out of scope.
     * All data copy constructed is destroyed with it.
     */

    /**
     * foo will go out scope and destroyed 
     */
}
```

As you can see, this might be the behavior that you want, but it's worth noticing how passing an object by value requires the use of a temporary to be copy constructed and then destroyed once the function goes out of scope.

In order to take advantage of the temporary object, `std::move` comes into play. Allowing to transfer ownership of a temporary object.

```c++
void a(Foo&&);
a(std::move(foo));
```

Doing so would intruct the compiler to treat local foo as a `rvalue`, effectively treating it as if we were to pass in a reference value itself. In this case, no copy or move cosntructor would be called.

Now, let's consider a function that would return a temporary object using a copy assignment.

```c++
/**
 * By default, foo_local would be constructed on the stack
 * and then list[0].operator= would execute using foo_local as 
 * temporary right before its destructor is called.

 * Here is a better visualization of the whole process
 * -----------
 * a()                              activation record created
 * foo_local                        constructed with DATA
 * ~a()                             activation record destroyed
 * list[0].operator=(foo_local)     invoked with `foo_local`  (perform another copy from temporary)
 * foo_local                        destroyed
 */
Foo build_foo(){
    std::cout << "a called" << std::endl;
    Foo foo_local;
    std::cout << "a exiting" << std::endl;
    return foo_local;
}

Foo list[1] {};
list[0].operator=(build_foo());
```

The local object `foo_local` is constructed with data in it, thus using memory and cpu cycles. Then, the data is again assigned using the copy assignment operator, making a copy using the temporary object which we called. This would again unnecessary extra computations.

Now, let's consider the same example using a move assignment operator and see how this could help to use memory efficiently without wasting it.

```c++
/**
 * The move semantics would allow us to turn an lvalue to an rvalue.
 * In this case, giving us a chance to access the temporary object,
 * its data and perform operations on it, such as transferring 
 * ownership to another object before the temporary goes out of scope.
 *
 * In this case, `rhs` would be a temporary, allowing us to get its data:
 *
 * this->data = rhs.data
 * rhs.data = nullptr
 *
 * This would effectively transfer ownership from the temporary object,
 * without having to perform copy operation that could affect the performance
 * of the application.
 */
void operator=(const Foo&& rhs){}

/**
 * Once a move assignment operator is defined, the compiler will
 * invoke the move assignment operator on the temporary object.
 * This is where we would be able to take advantage of the temporary 
 * object and steal its data, without having to do a full copy of the object

 * Here is a better visualization of the whole process
 * -----------
 * a()                              activation record created
 * foo_local                        constructed with DATA
 * ~a()                             activation record destroyed
 * list[0].operator=(foo_local)     invoked with `foo_local` 
 *
 *  In the case of a move assignment list[0] would use `foo_local` to 
 *  grab its data, such as `list[0].data = rhs.data` and set the data to
 *  a nullptr to finalize the transfer or owernship to list[0], `rhs.data = nullptr`.
 *  The temporary `rhs` would be destroyed normally, but the data would be 
 *  contained and handled by list[0].
 *
 * foo_local                        destroyed without the data
 */
Foo build_foo(){
    Foo foo_local;
    return foo_local;
}

int main(){
    Foo list[1] {};
    list[0].operator=(build_foo());
}
```

With the use of a move assignment operator we can have our object transfer ownership of the data without performing any copy assignment operation.

It is possible to instruct the compiler to explicitly use an rvalue with `std::move`. By default all parameters passed in a function call are implicitly treated as `lvalues`. `std::move()` will force the compiler to treat variables as rvalues.

```c++
void swap1(Foo& a, Foo& b){
    Foo tmp { a };  // copy constructor Foo(Foo&)
    a = b;          // copy assignment  operator=(Foo&)
    b = tmp;        // copy assignment  operator=(Foo&)
}

void swap2(Foo& a, Foo& b){
    Foo tmp { std::move(a) };   // move constructor Foo(Foo&&)
    a = std::move(b);           // move assignment  operator=(Foo&&)
    b = std::move(tmp);         // move assignment  operator=(Foo&&)
}
```

It's important to explicitly use the `std::move()` utility in order to treat an lvalue as rvalue. All rvalue declared in the code will be implicitly turned into an lvalue by the compiler.

```c++
int &&r {get_val()}; 	// r is an rvalue
int a = r;		// r is implicitly treated as lvalue by the compiler
int a = std::move(r);	// r would still continue to be treated as rvalue
```

The same would apply to function parameters.

```c++
void foo(int&& r);
int &&r {get_val()}; 	// r is an rvalue
foo(r); 		// compiler will treat `r` as lvalue and trigger a compiler error
foo(std::move(r)); 	// compiler will treat `r` as rvalue
```

Rvalues are turned into lvalue as soon as they have a name and adress. The compiler will treat rvalues are lvalues unless explicitly declared using the `std::move` semantics as shown above.


### Reference collapsing and forwarding

There are a few concepts to understand when it's about referencing a variable, such as reference collapsing.

Consider the following code:
```c++
template <typename T> void foo(T& x);
int n = 100;
int& x = n;
foo(x);
```

Calling `foo` using template parameter `T&` would 


```c++
struct Foo {
    Foo(){ std::cout << "Foo()\n"; };
    Foo(const Foo&) {std::cout << "Foo(copy)\n"; }
    void operator=(const Foo&) {std::cout << "Foo.operator=()\n"; }
    int x_{};
    void get() & { std::cout << "get()&" << std::endl; }
    void get() && { std::cout << "get()&&" << std::endl; }
};

template <typename T> 
void foo_wrapper_with_forward(T&& t){
    foo(std::forward<T>(t));
}

template <typename T> 
void foo_wrapper_without_forward(T&& t){
    std::cout << std::is_rvalue_reference_v<decltype(t)> << std::endl;
    foo(t);
}

template <typename T> void foo(T&&) {
    std::cout << "foo(foo&&) invoked" << std::endl;
    std::cout << std::is_const_v<T> << std::endl;
}

template <typename T> void foo(T& t) {
    std::cout << "foo(foo&) invoked" << std::endl;
    std::cout << std::is_const_v<T> << std::endl;
}

int main (){
    /** 
     * Rvalues are treated as rvalue once passed as parameter
     * std::forward allows to keep the value category intact
     */
    foo_wrapper_without_forward(Foo{});     // valid; foo(Foo&) invoked
    foo_wrapper_with_forward(Foo{});        // valid; foo(Foo&&) invoked


    Foo a;  // lvalue
    foo_wrapper_without_forward(a);             // valid; foo(Foo&) invoked
    foo_wrapper_with_forward(a);                // valid; foo(Foo&) invoked

    /**
     * `a` is an lvalue category, the compiler will treat it as one
     * std::move semantics will instruct the compiler to treat `a` as `rvalue`
     * category
     */
    foo_wrapper_with_forward(std::move(a));     // valid; foo(Foo&&) invoked

    /**
     * In the code above, the compiler will pass an `rvalue` 
     * However, the rvalue will change category to `lvalue` once 
     * it is passed as parameter and lose its category value type
     * std::forward is the solution to this 
     *
     * std::is_rvalue_reference_v<decltype(t)> will return 1
     */
    foo_wrapper_without_forward(std::move(a));     // valid; foo(Foo&) invoked
}
```

