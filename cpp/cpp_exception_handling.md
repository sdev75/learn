## Exception Handling

Even though programs are designed to run flawlessly, it is quite often possible to encounter runtime errors that prevent the program from running normally. The C++ language offers a mechanism to handle runtime errors by stopping the execution of the code and transferring control from one part of the program to another, giving the programmer a powerful tool for dealing with exceptional situations.

The `throw` keyword is used to indicate that an error has occurred. Such errors are controlled by implementing an exception handler using the `try` and `catch` blocks. The `try-block` statement identifies a block of code for which exceptions will be activated, followed by one ore more `catch` blocks. Each `catch` clause is an exception handler.

```c++
try {
    throw 1;
} catch(int ex){
    std::cout << "exception: " << ex << std::endl;
}
```

Each try-block has its own scope, thus local variables will run out of stock once the control is transfered to the catch block.

```c++
    try {
        Foo foo;		// Foo is constructed
        throw 1;		// Foo is destructed
    } catch(int ex){
        std::cout << "exception: " << ex << std::endl;
    }
```

Attention need to be paid when using local variables or pointers as the local variable will no longer be in scope when the try block is executed, which can easily lead to a memory leak. A solution could be to use smart pointers and destruct them as they run out of scope.

```c++
try {
    int local{12};
    int* ptr = &local;
    throw ptr;
    
} catch (int* e){
    // the content of ptr ran out of scope
    // this is a memory leak
    std::cout << e << std::endl;
}
```

### Exceptions Using Objects

An exception can be thrown using an object that represent an exception. In that case, the objects thrown should implement a `copy constructor`. Objects can be useful as they can carry information with them and they can be used to trace the condition that caused the exception.

It is highly recommended using a reference when throwing an exception to prevent using the copy constructor to copy the object twice.

```c++

```

### Stack Unwinding

C++ normally stores a return address on the stack along with its variables and other information. When throwing an exception the program will continue to free the stack until the try block address is reached and then pass the control to the catch block. Put it simply, instead of going back to one frame, it goes back to as many frames as needed to reach the end of the try block and then it passes control to the catch blocks. During this process, as with any functions, destructors are invoked and local variables are popped out. The behaviour of the stack unwining will depend on the implementation and optimization used by the compiler, thus it might behave differently. Some compilers might not store information that will not be used, some might store some data into registers. 

Consider the following example showing the stack unwinding:

```c++
#include <iostream>
#include <iomanip>

struct Foo {
    Foo(int lvl):m_lvl(lvl){
        std::cout << "\n" << std::string(m_lvl, '\t') 
             << "    " << m_lvl <<  " Foo() " << (void*) this; 
    }
    ~Foo(){
        std::cout << "\n " << std::string(m_lvl, '\t') 
            << "    " << m_lvl << " ~Foo() " << (void*) this ; 
    }
    int m_lvl;
};

void b();
void c();

void a(){
    std::cout << "\t\\- a_beg " << (void*) a;
    Foo foo{1};
    b();
    std::cout << "\n\t\t" << "   b_end " <<  (void*) b;
}

void b(){
    std::cout << "\n\t\t\\-" << " b_beg " << (void*) b;
    Foo foo{2};
    c();
    std::cout << "\n\t\t\t" << "   c_end " << (void*) c;
}

void c(){
    std::cout << "\n\t\t\t\\-" << " c_beg " << (void*) c;
}

int main(){
    std::cout << " main() " << (void*) main << std::endl;
    /**
     * The compiler might reuse the same stack used by main 
     * to generate code for the try-block statement. 
     * However, the C++ language will treat them as completly
     * separate blocks and will perfom copy construction when
     * passing objects by value in the catch clause.
    */
    try {
        int x{3};
        std::cout << "\ttry stack top: " << &x << std::endl;
        a();
        std::cout << "\n\t" << " a_end " << (void*) a;
    } catch (int e) {
        std::cout << "\n" << std::string(e, '\t') << " catch stack top: " << &e << std::endl;
        int x{5};
        std::cout << "\nException " << e << std::endl;
    }

    int x{};
    std::cout << "\tmain stack bottom: " << &x << std::endl;
}
```

In the case the stack would use each stack frame to return the previous address recursively:

```
 main() 0x4023e7
	try stack top: 0x7ffde3df5dc8
	\- a_beg 0x402286
	    1 Foo() 0x7ffde3df5d9c
		\- b_beg 0x402318
		    2 Foo() 0x7ffde3df5d6c
			\- c_beg 0x4023b7
			   c_end 0x4023b7
 		    2 ~Foo() 0x7ffde3df5d6c
		   b_end 0x402318
 	    1 ~Foo() 0x7ffde3df5d9c
	 a_end 0x402286	main stack bottom: 0x7ffde3df5dcc
```

However, if the program would throw an exception inside one of the stack frames, for example in function c(), the stack would unwind and reach the first catch block available. During this process the program will execute the destructors of each stack frame.

```c++
void c(){
    std::cout << "\n\t\t\t\\-" << " c_beg " << (void*) c;
   throw 3;
}
```

Output:

```
 main() 0x402426
	try stack top: 0x7ffd4ba5fd68
	\- a_beg 0x4022a6
	    1 Foo() 0x7ffd4ba5fd3c
		\- b_beg 0x402338
		    2 Foo() 0x7ffd4ba5fd0c
			\- c_beg 0x4023d7
 		    2 ~Foo() 0x7ffd4ba5fd0c
 	    1 ~Foo() 0x7ffd4ba5fd3c
			 catch stack top: 0x7ffd4ba5fd64

Exception 3
	main stack bottom: 0x7ffd4ba5fd6c
```

As you can see above, even though function b() and a() are fully returned when the exception is `thrown`, the actual stack frame are unwinded and the destructors of any object invoked. This allows automatic destruction of objects even though an exception occurred. The remaining portion of the functions `a` and `b` are never executed and the control is transfered to the catch block.

### Ellipsis Catch-all block

Catch blocks can be used to catch all possible exceptions by using the three dot syntax as shown below:

```c++
try {} catch(...) {}
```

### Uncaught Exceptions

Exceptions that are not handled correctly will cause the program to terminate. If an exception does not have a matching `catch` handle, the built-in `terminate()` function will be executed and call the `abort()` function which will terminate the execution of the program. It is possible to override the `terminate` function to handle uncaught exceptions.

```c++
void foo(){
    std::cout << "Terminating..." << std::endl;
    std::abort();        // abort() will end the execution properly
}
int main(){

    std::set_terminate(foo);
    throw 100;
}

```


### Exceptions and Polymorphism

Classes being the most useful types of exceptions can be used to take advantage of polymorpshism when handling exceptions. 

```c++
struct Foo {};
struct Bar : Foo {};
struct Baz : Bar {};

int main(){
    try{
        throw Baz();
    }catch(Foo& e){	// Foo will catch all derived classes
       //  typeid(e)	// Foo
    }
}
```

### Noexcept Specifier 
The `noexcept` specifier instructs the compiler to perform a compile-time whether a function can throw exceptions. 

```c++
void foo() noexcept {
    throw 1;			// error; it will not compile
}
```
Class destructors are implicitly declared as noexcept. This can be overriden by using `noexcept(false)` in the destructor declaration. 

```c++
Foo::~Foo() noexcept(false){}
```

### Standard Exceptions

The C++ standard provides a consistent interface to handle errors. All exceptions generated by the standard library inherits from `std::exception` class. It's possible to extend these classes to create customized exception handling.

Custom exception classes will offer their advantages to provide specialized exception types to your logical workflow. Custom exception classes allow you to add extra information into the object and access this information when the custom exception is thrown.

```c++
class FooException: std::exception {
public:
    FooException(const char* description, int code) : code_(code) {
        description_ = new char [std::strlen(description) + 1];
        strcpy(description_, description);
    }
    void operator= (const FooException&) = delete;
    FooException (const FooException&) = delete;
    virtual ~FooException(){ delete description_; }
    virtual const char* what() const noexcept {
        return description_;
    }
    int getCode() const { return code_; }
private:
    char* description_;
    int code_;
};

int main (){
    try {
        throw FooException {"testing foo exception", 1};
    } catch (const FooException& e){
        std::cout << e.getCode() << " desc: " << e.what() << std::endl;
    }
}
```