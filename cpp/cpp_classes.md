## Classes and Objects

The main goal of the C++ language is to provide object oriented programming (OOP) functionality. 
This concept allows designing programs with features such as:

- abstraction
- encapsulation
- polymorphism
- inheritance

All these features are the core of object-oriented programming, giving the programmer the flexibility to describe the data in terms of its interface making the code to reusable and easier to maintain.

Classes can be considered a blueprint of a instance of an object, which programmers use to represent general concepts. You can have many objects with the same blueprint. In other words, objects are entities that represent the result of an abstraction or blueprint, allowing to have `state`, unique `identity` and react to changes, thus having `behavior`.

### Compiler and classes

Consider the following code of a basic class declaration with some method definitions:

```c++
class Foo {
    int id;
    void method();
};
```

In this case the compiler would translate the class blueprint to procedural code such as the following:

```c++
void foo_constructor(Foo&);
void foo_copy_constructor(Foo& this, const Foo&);
void foo_move_constructor(Foo& this, Foo&&);
Foo& operator=(Foo& this, const Foo&);
void foo_set_id(Foo& this, int id);
int foo_get_id(const Foo& this) const;
```

This is pretty much how you could implemnent OOP in C language, although slightly different.

### Class members and methods

A class specification is made up of two parts:

- Class declaration
- Class method definition

A class declaration is what describes the components of the objects such as its data members and member functions also called methods. The class definition describes how these data members and data functions (methods) are implemented.

```c++
// Simple class definition
class Foo {
    void setValue(int value);
    int getValue() const;
    int value;
};
```

### Member access control

Every member in a class has 3 differnet access specifiers:

- public
- protected
- private

All members have the private access specified by default, which means that they can only be accessed by the class itself, thus having a `scope class`, hiding the data from other classes or any outside code.

Public access specifier allows code and other classes to access private and protected members. Protected access specifier can only be accessed by the class itself, allowing other subclasses to access protected members. Private members allow only the class itself to access private members, including subclasses. 

```c++
/**
 * The order of declaration does not matter. C++ does not impose restrictions.
 */

class Foo {
    int x; // private by default 'class-scope' only.
public:
    int y; // any code can access it
protected:
    int z; // class-scope + subclass access
};
```

### Classes and memory layout

Like C structs, C++ is no different. Consider the following class:

```c++
class Foo {
    std::string x;	// 24 bytes
    double y;		//  8 bytes
    int z;		//  4 bytes
    bool b;		//  1 bytes
};
```

The above class would be placed in a contiguous virtual memory block, having a full size of 37 bytes. However, considering the CPU reads memory in fixed-size words the actual size of the structure might be using padding to make it possible for the CPU to work with it. In fact, the actual size of the object Foo would be 40 bytes instead of 37 bytes.

### Constructors and destructors

Constructors and destructors are methods used to build an object using the class blueprint and tear it down when the object is no longer needed. This gives the possibility to inizialize member variables, execute methods and much more, thus giving an object an identity, behavior and state.

C++ will automatically generate definitions for certain functions (if your object requires them) such as:

- default constructor
- copy constructor
- assignment operator
- default destructor
- address operator

#### Default constructors

```c++
class Foo {
   Foo() = default; 	// user-declared default constructor (compiler)
   Foo(int bar){};  	// user-declared constructor
   ~Foo(){}; 		// user-declared destructor
};

/**
 * The compiler will generate an implicit default constructor if 
 * there are no user-declared default constructors.
 */
Foo:Foo(){};

Foo foo;		// invokes default constructor;
```

It's important to notice that if you define a constructor, the compiler will not create one for you.

#### Copy constructors

A copy constructor is invoked to create a copy of an object. It is used when an object is excplicitly initialized with an existing object. The copy constructor will perform a `memberwise copy` or `shallow copy` of the data members including pointers. Thus it's always a good idea to overload the copy constructors when dealing with pointers to prevent leaking memory data or freeing memory twice.

```c++
Foo foo(foo2);			// invokes Foo(const Foo&)
Foo foo = foo2;			// invokes Foo(const Foo&)
Foo foo = Foo(foo2);		// invokes Foo(const Foo&)
Foo *foo = new Foo(foo2);	// invokes Foo(const Foo&)
```

A copy constructors takes a const reference to the source object and it does not return any value. 

A copy constructor it is often invoked when passing arguments to functions using pass-by-value semantics. In that case, the function or method receives a copy of the variable, calling the copy constructor to copy the original object.

```c++
/**
 * Foo is passed as argument with pass-by-value semantics
 * A new local object would be constructed 
 * using the copy constructor of Foo.
 *
 * The local object would then go out of scope
 * and destroyed accordingly.
 */
void test(Foo foo){}	// copy required 
void test(Foo& foo){}	// copy NOT required
```

#### Assignment operators

An assignment operator is used to assign an object to another. Sometimes this operation is invoked in two steps, first using the copy constructor followed by a copy assignment. Like a copy constructor, an implicit assignment operator does a `memberwise copy` or `shallow copy`. 

It's also worth noticing that static members will not be copied as they are not part of the actual object, but rather the class itself. Also in this case, it would be crucial implementing a user-declared overloaded assignment operator that would handle pointers, in order to prevent memory leak or double frees.

```c++
Foo& foo::operator=(const Foo& rhs);
```

#### Implicitly-declared constructor and destructors

If there are no user-declared constructors of any kind, the compiler will declare a default constructor as an inline public member of its class. However, if user-declared constructors are present, the compiler will not generate one unless specified by using the `default` keyword as per the example above.

#### User-declared constructor

When providing a non-default constructor, the compiler will not generate any constructors for you. You will have to provide your own version of a default constructor making the following declaration invalid:

```c++
Foo::Foo(int bar){} 	// user-declared constructor
			// all data members are uninitialized

Foo foo;		// ! Compiler ERROR !
```

Object constructors can be initialized implicitly or explicitly. 

```c++
Foo foo;		// call constructor implicitly
Foo foo = Foo();	// call constructor explicitly
Foo *foo = new Foo;	// call constructor implicitly
Foo foo();		// disambiguated as function declaration
```

#### Reusing constructors

It is possible to call constructors on already initialized variables. If the object already exists the statement will assign a new value to the object by creating a temporary object and then copying the contents to the receiving variable.

```c++
Foo foo = Foo(5);	// call constructor
foo = Foo(10);		// call temporary constructor
			// copy assignment operation (foo.x = temporary.x)
			// call temporary destructor
```

#### Initializer Lists

Constructors offer a different method for initializing data members, called the initializer list. The constructor below takes advantage of the initializer list syntax:

```c++
Foo::Foo() : m_x(10) {};
```

There are quite a few advantages for initializing data members using the initializer list. When an object is constructed, C++ has already finished setting up all the data members, and by assigning values to an object in the constructor's body, you are not constructing the object, as the object is already constructed when the constructor's body begins executing. 

Therefore, initializer lists allow to initialize data members at the time of the object creation. For example, `const data members` can only be defined during object constructor, making the initializer list efficient for this operation. This would not work without initializer list, the compiler would not allow to modify a `const data member` once it's created.

#### Designated initializers (from C++20)

C++20 added a new feature to perform aggregate initialization. The order in which the designators are used should match the same order as the data members of the class.

```c++
struct Foo { int x; int y; int z; };

Foo a{.y = 2, .x = 1}; 	// error; designator order does not match declaration order
Foo b{.x = 1, .z = 2}; 	// ok, b.y initialized to 0
```

#### Destructors

Destructors unlike constructors must not have any arguments. The compiler will call the destructor when the class object is not longer in scope. If the destructor is not user-declared, the compiler will implicitly declare a default destructor if the object will require it.

```c++
Foo::~Foo(){};		// destructor
```

It is rare the need to call a constructor manually. However, when using placement new allocation, it is the best way to free allocated memory.

```c++
class Foo {
public:
    Foo(){
        std::cout << "ctor @ " << this << std::endl;
    }
    ~Foo(){
        std::cout << "dtor @ " << this << std::endl;
    }
};

int main(){
    char *buf = new char[512];

    Foo *x, *y;

    x = new (buf) Foo;
    y = new (buf+sizeof(Foo)) Foo;

    y->~Foo();
    x->~Foo();
    delete[] buf;
}
```

In the example above, the call to `delete[] buf` would not invoke the destructors. It merely delete the block of memory allocated at the address of buf, it does not call the destructors for any object that `placement new` constructs. An explicit call to the destructor will solve this problem. It's also worth nothing that it might be a good idea to destruct object in the opposite order they were created to ensure any dependencies are removed before destroying the object.


#### Constructor delegation (from C++11)

C++ allows the constructor to reuse an existing constructor to avoid code duplication. In that case the target constructor is selected by overload resolution and invoked first.

```c++
class Foo {
public: 
  Foo(int x, int y) {}
  Foo(int y) : Foo(10, y) {} 	// Foo(int) delegates to Foo(int,int)
};
```

During the construction of the Foo object, the delegated constructor `Foo(int,int)` would be invoked first executing its body, then the delegating constructor `Foo(int)` would get the control back and its body executed.

### Operator overloading 

C++ allows you to overload operator function to design how objects relate to each other. Operator overloading can either be done by invoking an operator function like `operator=()` or by using the operator syntax such as `obj1 = obj2`.

An operator function takes an argument list as its parameters. In the case of the operator function being a class member function, then the first operand is the invoking object and is not part of the arguments.

```c++
/**
 * The first operand is the invoking object (Foo)
 */
Foo::operator+(int bar);	// class member operator function	
```

### Class Friends

C++ classes offer encapsulation, and data hiding. Control access might prevent external function from accessing private or protected members. Friend functions, classes and member functions provide another form of access to the class.

Consider the following example when operator overloading might not suffice:

```c++
class Foo {
public:
    friend Foo operator+(int x, const Foo& rhs);
    explicit Foo(int x) : m_x(x){}
    Foo operator+(const Foo& rhs){
        Foo foo{0};
        foo.m_x = this->m_x + rhs.m_x;
        return foo;
    }
    int x() const {
        return m_x;
    }
private:
    int m_x{};
};

Foo operator+(int x, const Foo& rhs){
    Foo foo{0};
    foo.m_x = x + rhs.m_x;
    return foo;
}

int main(){
   /**
     * We declared our Foo constructor as explicit to prevent the compiler
     * from doing any implicit conversion and thus searching for a valid
     * operator function that would take int as the first operand
     *
     * We declared a global operand+ function and flagged it as friend of 
     * class of Foo in order to allow it to access private members without 
     * making any changes to the blueprint of our Foo class.
     */

   Foo bar{10};
   Foo baz{40};
   //bar = bar + baz;	// Foo.operand+(Foo); 
   bar = 10 + baz;	// operand+(int,Foo);
}
```

In the second statement the compiler would not find a corresponding member function to perform the addition between integer and Foo type. Declaring a global function of the operand+ overload would make the statement compile. We've flagged the function as friend of the class to let the compiler know that the function is a friend of the class and will be able to use private members. If we were not to flag the function as friend, the statement would not compile as it would not be able to access the `m_x` private member variable.

### Static data members

When a data member is marked as `static`, the data member will be associated with the class itself rather than the instantiated objects. 

```c++
struct Foo {
    static int c;
    Foo(){ c++; }
};

int Foo::c = 0;
Foo a;			// Foo::c  = 1
Foo b;			// Foo::c  = 2
Foo c;			// Foo::c  = 3
```

Static data member must have their data allocated in the source file. Another possibility would be to declare the static data member as inline to initialize the member without having to allocate space in the source file.

```c++
struct Foo {
    inline static int c{0};	// ok; legal
    static const int a{0};	// ok; allowed
};
```

The C++ standard allows declaring a `static const` data member without having to initialize it in the source file. Older version of C++ only allow this for integral type such as `int` and `char`. Newever version of C++ do not have this limitation any longer.

#### Static Methods

Like static data members, method can be associated with the class itself instead of the object. It's important to notice that non-static data members cannot be accessed within a static method, as they do not belong to the same scope.

```c++
struct Foo {
    static void bar(){};
};
```

#### Reference Data Members

It's quite useful to have two classes communicate together. This would allow the flexibility of having a class within another class. References must be initialized in both the constructor and copy constructor. They cannot be initialized using the assignment operator as a reference cannot change after being initialized.

```c++
struct Bar {
    Bar()=default;
};

struct Foo {
    Bar& m_bar;
    // Foo(Bar &bar) { m_bar = bar; }	// error; references cannot be reassigned
    Foo(Bar &bar) : m_bar(bar) {};	// ok; reference is initialized during creation
};

Bar bar;
Foo foo(bar);
```

#### Const Data Members and Methods

Const data members are used to declare data that won't change after it is created and initialized. It is usually common to declare them as `static` as well.

#### Mutable Data Members

Const methods might require to perform a change within the body of the function, in that case the compiler allows a const method to modify an internal data member by declaring it as `mutable`.

```c++
struct Foo {
    Foo()=default;
    int bar() const {
        m_num++;		// ok; mutable data members can change
        return m_x;
    }
    int m_x{0};
    mutable int m_num{0};
};
```

#### Structured Binding (since C++17)

Structured bindings are a way to create an alias to an existing object. They are similar to references, except that the binding does not have to be of a reference type.

```c++
struct Foo {
    mutable int x;
    double y;
};

Foo bar(){ return Foo{10,99.5}; }

const auto [x, y] = bar();	// structured binding
x = 2;                  	// ok;
y = 2.22;               	// error; read-only value due to const class

Foo foo{1,22.4};   
auto& [x, y] = foo;		// structured binding using references
x = 2;                  	// ok; modifies foo.x via reference
assert(&x == &foo.x);		// valid; addresses would be the same        
```

#### Inline Methods

Inlining allows the compiler to insert the method or function body directly into the code where the method or function call is made. The definition of inlined methods or functions should be available in every source file in which they are invoked because the compiler will substitute the function body with its inlined defintion. Putting a function definition directly in the class defintion will implicitly declare the function as inline. It's important to notice that the compiler might silently ignore the inline directive. In some cases, inlined function might lead to code bloat as the body of the function will be replicated.

### Nested Classes

A class can be defined within another class definition. 

```c++
struct Foo {
    Foo()=default;
    struct Bar {
        char x{2};
    };

    void useBaz(){ Baz baz; }
    char x{1};

private:
    struct Baz {
        char x{3};
    };
};

Foo foo;
Foo::Bar bar;	// ok; public scope
Foo::Baz baz;	// error; private scope
foo.useBaz();	// ok; outer has access to inner class
```

Nested classes are a convenient way to nest structures or class declaration in another class and giving it a class scope. A perfect example would be to encapsulate a structure within another class so that have the type restricted to the class itself.

```c++
class Foo {
    // class scope members
    struct Bar { int x; };
    Bar bar;
public:
    Foo(){ bar.x = 100; }
    int baz()  { return bar.x; }
};

Foo foo;	// initialize Bar structure internally
foo.baz();	// 100
```

### Class Inheritance

Code reusability is the main goal of the C++ language. With class inheritance it is possible to add functionality to an existing class, add to the data used for representation and change how the class behaves.

```c++
class Foo {
public:
    Foo()=default;
    int x() const { return m_x; }
private:
    int m_x{};
};

/**
 * The Foo bar becomes the base class of the derived Bar class
 */
class Bar : public Foo {
    /*
     * A derived class needs its own constructor
     */
    Bar()=default;
};

```

When a class inherits from another, the original class is called `base class`. In the case of the example above, the `Foo` class would be the base class of `Bar`.

#### Extending classes and member access

When inheriting or extending a base class, the compiler will automatically add the data members of the `base class` also sometimes referred as the `parent class` or `superclass` to the `derived class` or `subclass`.
A derived class does not have direct access to the private members of the base class. In order to access private members of the base class, the derived class has to use base-class methods.

When a derived object is constructed, the base-class constructor is invoked first. 

```c++
struct Foo {
public:
    Foo()=default;
    char m_x{1};
protected:
    char m_y{2};
private:
    char m_z{3};
};

struct Bar : public Foo {  
    Bar()=default;
};

Bar bar;	// Foo constructor invoked
		// Bar constructor invoked

bar.m_x;	// ok; public access
bar.m_y;	// error; protected access within the class or subclass only
bar.m_z;	// error; private members are always unaccessible
```

Declaring the relationship with the parent class using different access modifier such as `protected` and `private` will constrain the derived class members to the specified access modifier. 

In the case of a `protected` inheritance, all the public and protected data members and method of the derived class will become `protected` in the context of the derived class.

Similarly, declaring the relationship with the parent class using the `private` access modifier will turn all `public`, `protected` and `private` data members and method of the parent class to become `private` to the context of the derived class.

```c++
struct Bar : public Foo {};	// all members will retain their original access specifier
struct Bar : protected Foo {}; 	// all public members will have protected access
struct Bar : private Foo {};	// all members will have private access
```

#### Member access 

Parent class members can be constrained to a more refined level with the use of the `using` keyword. This allows to change the member access level for a specific member of the base class.

```c++
struct Foo {
public:
    Foo()=default;
    char m_x{1};
    void bar(){};
protected:
    char m_y{2};
private:
    char m_z{3};
};

/**
 * All members of Foo will be private 
 * within the context of Bar
 */
struct Bar : private Foo {  
    Bar()=default;
protected:
    using Foo::m_x; // ok; scope constrained to protected scope
public:
    using Foo::bar; // ok; overload of access from private to public
    using Foo:m_z;  // error; private member of base class cannot be overloaded
};

/**
 * All members of Bar will be public 
 * within the context of Bar.
 *
 * Bar.m_x will be accessible within the context of Baz
 * because it was declared as protected within
 * the context of Bar
 */
struct Baz : public Bar {
    Baz()=default;
    void x(){m_x;}  // ok; m_x redeclared with different access modifier within the Bar context
    void y(){m_y;}  // error; m_y inherited private access from Bar context

};
```

#### Constructors, Destructors and Inheritance

Derived objects must be constructed along with their base object or any objects contained within them. The C++ compiler will perform the following steps when initializing a derived object recursively:

- Construct the base class
- Construct non-static data members in the order in which they were declared
- Execute the body of constructor

Derived objects can reuse constructors from the parent class when initializing data members. This helps to reuse code without having to duplicate logic.

```c++
struct Foo {
    Foo(int x, int y) : m_x(x), m_y(y) {}
protected:
    int m_x;
private:
    int m_y;
};

struct Bar : public Foo {
    /*
     * Foo constructor is reused rather than redeclaring the initializer list
     */
    Bar(int x, int y, int z) : Foo(x, y), m_z(z) {}
private:
    int m_z;
};

{
    Bar bar(1,2,3);	// calls Foo(int, int)
			// calls Bar(int, int, int)
    bar.m_x; 		// 1
    bar.m_y;		// 2
    bar.m_z;		// 3
}                       // calls Bar destructor
                        // calls Foo destructor
```

#### Casting and slicing

A derived object and be assigned to its parent class. This usually ends up in slicing, due to not having the functionality defined in the derived class. Consider the example below:

```c++
Foo foo = bar;	// slicing; missing functionality from bar
```

However, assigning the derived class to a base pointer or reference will not slice the derived object, also called `upcasting`.

```c++
Foo& foo = bar;	// ok; foo refers to the bar class using the base class Foo
```

Casting from the base class to the derived class would perform `downcasting`.

```c++
Foo foo;
Bar* bar = static_cast<Bar*>(&foo);
```

However, this approach might not be the best one, as it might lead to undesired behaviour. Consider the following example:

```c++
struct Foo {
    int a{100};
    int y{33};
    int z{44};
}; // sizeof Foo = 12 bytes

struct Bar : public Foo {
    int z{500};
}; // sizeof Bar = 16 bytes


int main(){
    Foo foo;
    Bar* bar = static_cast<Bar*>(&foo);
    Bar baz;

    for(size_t i=0; i<sizeof(Foo)/sizeof(int); i++){
        std::cout 
        << "foo@" << ((int*) &foo + i) << " -> "
        << (int) *((int*) &foo + i)
        << std::endl;
    }

    std::cout << "----" << std::endl;
 
    for(size_t i=0; i<sizeof(Bar)/sizeof(int); i++){
        std::cout 
        << "bar@" << ((int*) bar + i) << " -> "
        << (int) *((int*) bar + i)
        << std::endl;
    }
}   
```

The output would look something like this:

```
foo@0x7ffd7348ecec -> 100
foo@0x7ffd7348ecf0 -> 33
foo@0x7ffd7348ecf4 -> 44
----
bar@0x7ffd7348ecec -> 100
bar@0x7ffd7348ecf0 -> 33
bar@0x7ffd7348ecf4 -> 44
bar@0x7ffd7348ecf8 -> 1934159084
```

As you can see `static_cast` would not prevent invalid data offsets from being accessed. Downcasting would better be handled with the use of `dynamic_cast`. This is due to the way the compiler places data in the stack.

#### Inheriting Base Contructors

It is possible to instruct the compiler to use the inherited constructor within the derived class.

```c++
struct Foo {
    Foo(int x, int y) : m_x(x), m_y(y) {}
    Foo(int x) : m_x(x), m_y(100) {}
    Foo(const Foo& rhs) : m_x(rhs.m_x), m_y(rhs.m_y){}
protected:
    int m_x;
private:
    int m_y;
};

struct Bar : public Foo {
    using Foo::Foo;     // Inherits all constructors from Foo
                        // Foo(int,int) and Foo(int)
                        // Copy constructors will not be inherited
    Bar(int x, int y, int z) : Foo(x, y), m_z(z){}
private:
    int m_z;
};

int main(){
    Bar a{1,2};         // calls Foo(int, int)
    Bar b{1};           // calls Foo(int)
    Bar c{1,2,3};       // calls Foo(int, int)
                        // calls Bar(int, int, int)
}
```

#### Copy constructors and Inheritance

The compiler will generate a copy constructor for you and construct the object as it normally would.

```c++
struct Foo {
    Foo(int x, int y) 
        : m_x(x), m_y(y)
    {
        std::cout << "Foo()"<< std::endl;
    }
    Foo(const Foo& rhs) : m_x(rhs.m_x), m_y(rhs.m_y){
        std::cout << "Foo copy()" << std::endl;
    }
    int x () const { return m_x; }
protected:
    int m_x;
private:
    int m_y;
};

struct Bar : public Foo {
    Bar(int x, int y, int z) 
        : Foo(x, y), m_z(z)
    {
        std::cout << "Bar()" << std::endl;
    }
    Bar(const Bar& rhs) : 
    // Foo(rhs.m_x, rhs.m_y), m_z(rhs.m_z) // error; m_y is private in Foo, thus private in Bar
    Foo(rhs), m_z(rhs.m_z)                 // ok; the compiler will Foo copy constructor
    {
        std::cout << "Bar copy()" << std::endl;
    }
private:
    int m_z;
};

void foobar(Bar bar){}

int main(){
    Bar bar1{10,20,30};     // Foo constructor called
                            // Bar constructor called

    Bar bar2 = Bar(bar1);   // Foo copy constructor called
                            // Bar copy constructor called

    foobar(bar2);           // Foo copy constructor called
                            // Bar copy constructor called
}
```

### Polymorphic Inheritance

Polymophism allows a derived class to behave differently to the base class depending on the object that invokes it, giving a derived class behaviours depending on the context of the object. It allows a derived class to be used as a contract beween object and the code executed in a specific context, allowing code to be used interchangeably.

It is possible to implement polymorphic inheritance by redefining `base-class methods` in a derived class or by using `virtual methods`.

#### Static binding

The C++ compiler has the responsibility to look at the function arguments, the naem and figure out which function to use. The process of intepreting a function call at compile time is called `static binding` or `early binding`.

```c++
struct Foo {
    void a(){ std::cout << "foo a called" << std::endl; }
protected:
    int m_x;
};

struct Bar : public Foo {
    void a(){ std::cout << "bar a called" << std::endl; }
};

struct Baz : public Bar {
    void a(){ std::cout << "baz a called" << std::endl; }
};

int main(){
    Foo foo;
    Bar bar;
    Baz baz;

    Foo* foo_ptr = &foo;
    foo_ptr->a();		// prints foo a called
    
    foo_ptr = &bar;
    foo_ptr->a();		// prints foo a called

    foo_ptr = &baz;
    foo_ptr->a();		// prints foo a called
}     
```

#### Dynamic Binding

In order for the compiler to know which function to call, virtual functions give the compiler instructions on how to interpret the code and perform the binding at runtime also referred to as `dynamic binding` or `late binding`. 

Dynamic binding is achieved by instructing the compiler with the keyword `virtual` prefixed to a method.

```c++
struct Foo {
    virtual void a(){ std::cout << "foo a called" << std::endl; }
protected:
    int m_x;
};

struct Bar : public Foo {
    virtual void a(){ std::cout << "bar a called" << std::endl; }
};

struct Baz : public Bar {
    virtual void a(){ std::cout << "baz a called" << std::endl; }
};

int main(){
    Foo foo;
    Bar bar;
    Baz baz;

    Foo* foo_ptr = &foo;
    foo_ptr->a();		// prints foo a called
    
    foo_ptr = &bar;
    foo_ptr->a();		// prints bar a called

    foo_ptr = &baz;
    foo_ptr->a();		// prints baz a called
}     
```

Dynamic binding allows a pointer to a base class to downcast to a derived class within a hierarchical structure. The compiler will return a nullptr in case the casting will not be possible, allowing the programmer to prevent access to bad memory at runtime.

```c++
struct Foo {
    virtual ~Foo() = default;
    virtual void foo_virtual(){std::cout << "foo virtual " << std::endl;}
    void foo(){std::cout << "foo " << std::endl;}
    virtual void t(){std::cout << "foo t" << std::endl; }
};

struct Bar : public Foo {
    virtual void bar_virtual(){std::cout << "bar virtual " << std::endl;}
    void bar(){std::cout << "bar " << std::endl;}
    virtual void t(){std::cout << "bar t " << std::endl; }
};

struct Baz : public Bar {
    virtual void baz_virtual(){std::cout << "baz virtual " << std::endl;}
    void baz(){std::cout << "baz " << std::endl;}
    virtual void t(){std::cout << "baz t " << std::endl; }
};

int main(){
    /**
     * foo is a pointer to the base class Foo,
     * pointing to a derived object in memory
     */
    Foo* foo_ptr = new Bar();
    Foo& foo_ref = *foo_ptr;

    /**
     * Calling a non-polymorphic (virtual) method using 
     * the base pointer will not work.
     *
     * The compiler will invoke the methods using the 
     * base class pointer context Foo.
     */
    foo_ptr->foo();         // valid; foo
    foo_ptr->foo_virtual(); // valid; foo virtual
    foo_ptr->t();           // valid; foo t
                            // static binding will use foo::t()
                            // rather than the overridden method

    // foo_ptr->bar();         // error; Foo has no member `bar`
    // foo_ptr->bar_virtual(); // error; Foo has no member `bar_virtual`
    
    /**
     * Dynamic casting will instruct the compiler to
     * do the casting at runtime, thus allowing to use
     * a derived class using a pointer to the base class
     * when possible.
     *
     * In the case below, the compiler will call the 
     * methods using the context of `Bar`.
    */
    std::cout << "----" << std::endl;
    Bar* bar_ptr = dynamic_cast<Bar*>(foo_ptr);

    bar_ptr->foo();         // valid; foo
    bar_ptr->foo_virtual(); // valid; foo virtual
    bar_ptr->t();           // valid; bar t
    bar_ptr->bar();         // valid; bar
    bar_ptr->bar_virtual(); // valid; bar virtual
                     
    /**
     * Dynamic casting between invalid objects will
     * return a nullptr. In the case of dynamic casting
     * the object (Bar (via Foo pointer)) to a Baz object
     * will not work, thus retun a nullptr.
     *
     * Bar will not contain any information about
     * Baz. Bar being a parent class of Baz, will not
     * share any data required to cast to Baz correctly.
     *
     * It's important to notice that even though `foo_ptr`
     * is declared as a pointer to Foo, it is actually 
     * pointing to the derived object of type Bar.
     *
     * In this case baz_ptr will be a nullptr. 
     *
     */ 
    std::cout << "----" << std::endl;
    Baz* baz_ptr = dynamic_cast<Baz*>(foo_ptr); 
    // assert(baz_ptr != nullptr);
    
    /*
     * Unlike dynamic casting, a static cast will still
     * be considered a pointer pointing to random memory 
     * and cause undefined behaviour.
     *
     * Calling baz_virtual() on baz_ptr will attempt to
     * invoke a method on Bar (foo_ptr is pointing to Bar object)
     *
     * It is highly recommended to use dynamic casting to
     * prevent potential bugs.
     *
     */
    baz_ptr = static_cast<Baz*>(foo_ptr);
    // baz_ptr->baz_virtual();     // undefined; segmentation fault

    bar_ptr = dynamic_cast<Bar*>(foo_ptr);
    bar_ptr->bar();         // valid; bar
    bar_ptr->bar_virtual(); // valid; bar virtual
    bar_ptr->t();           // valid; bar t
    

    /**
     * Dynamic casting using references will work the same way
     * allowing an objec to be casted to another at runtime.
     *
     * A bad_cast exception will be thrown if the cast fails.
     */

    std::cout << "----" << std::endl;
    Bar bar;
    foo_ref = bar;
    //foo_ref.bar();          // error; Foo has no member named 'bar'
    //foo_ref.bar_virtual();  // error; Foo has no member 'bar_virtual'
    foo_ref.t();                // valid; bar t

    std::cout << "----" << std::endl;
    Bar& bar_ref {dynamic_cast<Bar&>(foo_ref)};
    bar_ref.bar();              // valid; bar
    bar_ref.bar_virtual();      // valid; bar virtual
    bar_ref.t();                // valid; bar t;

    delete foo_ptr;
}

```

Here is another trivial example that might help to get the idea of how dynamic cast might help out:

```c++
struct Human {
    virtual ~Human()=default;
    virtual const char* id(){
        return typeid(*this).name();
    }
};

struct Person : public Human {
    Person(std::string name) : m_name(name) {}
    virtual ~Person()=default;
    virtual void do_person_stuff(){
        std::cout << "Person::name() " << m_name << std::endl;
    }
protected:
    std::string m_name;
};
 
struct John : public Person {
    John() : Person("John"){}
    virtual ~John(){}
    virtual void do_john_stuff(){
        std::cout << "John::talk() " << m_name << std::endl;
    }
};

int main(){
    John john;
    Person jane{"Jane"};

    Human* humans[]{&john, &jane};

    for(Human* human : humans){
        std::cout << "human id: " << human->id() << std::endl;
        // 'john' is John type dynamic cast will work
        // 'jane' is Person type and dynamic cast will not work
        John* john_ptr = dynamic_cast<John*>(human);
        if(john_ptr){
            // john can do what jane cannot do (john stuff)
            john_ptr->do_john_stuff();
        }

        Person* person_ptr = dynamic_cast<Person*>(human);
        if(person_ptr){
            // both john and jane can do person stuff
            person_ptr->do_person_stuff();
        }
    }
}

```

#### Virtual Member functions and Dynamic Binding

Let's consider another example where a derived object is pointing to a base object. 

```c++
struct Base {
    void a() {std::cout << "Base a()" << std::endl; }
};

struct Derived : public Base {
    void a() {std::cout << "Derived a()" << std::endl; }
};

int main(){
    Derived derived;
    Base* base = &derived;

    /**
    * The Base::a() method is not declared virtual. 
    * The pointer type is known at compile time, therefore
    * the compiler will bind the a() method to Base::a() method
    * at compile time. This is static binding.
    */
    base->a();       // valid; prints `Base a()`
}
```

If the `a` method in the base class is declared as virtual, the compiler will instead perform dynamic binding at runtime and invokes `Derived::a()` instead. This allows a program to choose the correct method.

```c++
struct Base {
    virtual void a() {std::cout << "Base a()" << std::endl; }
};

struct Derived : public Base {
    void a() {std::cout << "Derived a()" << std::endl; }
};

int main(){
    Derived derived;
    Base* base = &derived;

    /**
    * The Base::a() method is not declared virtual. 
    * The pointer type is known at compile time, therefore
    * the compiler will bind the a() method to Base::a() method
    * at compile time. This is static binding.
    */
    base->a();       // valid; prints `Derived a()`
}

```

#### Virtual Table

Classes are generated at compile time using hard-coded values, this is true for non-virtual methods, making the CPU jump at exact specific locations known at compile time. However, when a method is declared as `virtual` the compiler will find the implementation in a special memory area called `virtual table` or `vtable`. Each class that has one or more virtual methods will have a `vtable` with pointers to the actual implementation of the virtual method, allowing the compiler to run the appropriate version of the method executed based on the type of the object rather than the variable itself.

Unfortunately, virtual methods require the program to perform an extra operation by dereferencing the pointer to the function call to execute.

```
Foo 	foo() -> Foo::foo()
	bar() -> Foo::bar()

Bar	foo() -> Foo::foo()
	bar() -> Bar::bar()
```

#### Method Override (from C++11)

The `override` identifier is used to specify that a virtual function is overriding another virtual function. It is a mechanism used by the C++ compiler generate a compile-time error if the base class method is not overriden correctly.

```c++
virtual void foo() const override {}
```

#### Overloading and overriding

Once a function is overridden, all other overloaded method must be explicitly defined, otherwise the compile will produce an error.

```c++
struct Foo {
    virtual void a()  {}
    virtual void a(int a) {}
};

struct Bar : public Foo {
   virtual void a(int) {}
};

struct Baz : public Bar {
    virtual void a() {}
    virtual void a(int) {}
};

int main(){
    Foo foo;
    Bar bar;
    Baz baz;

    bar.a();    // error; no matching function for call 
    bar.a(10);  // valid; a(int) is overriden, thus a() should too
    baz.a();    // valid; overridden
    baz.a(10);  // valid; overridden
}        
```

Once a method is overriden, the compiler will hide all other overloaded methods from the derived class. The remaining overloaded functions will have to be overriden explicitly, otherwise the compiler will simply hide them. Once an overloaded function is overriden, the rest should be implemented as well unless you dont need them, the compiler will hide them.

```c++
struct Foo {
    virtual void a(){}
    virtual void a(int){};
};

struct Bar : public Foo {	
    virtual void a(int) override {};
};
int main(){
    Bar bar;
    bar.a();	// error; no matching funtion for call to Bar::a()
    bar.a(10);	// valid; method is overriden correctly
}
bar.a(10);
}
```

#### Inheritance Access Specifiers

The compiler will ignore access modifier when accessing a derived object using a base class pointer. 

```c++
class Foo {
public:
    virtual void a() {
        std::cout << "Foo a (public)" << std::endl;
    }
private:
    virtual void b() {
        std::cout << "Foo b (private)" << std::endl;
    }
};

class Bar : public Foo {
public:
    virtual void b() override {
        std::cout << "Bar b (public)" << std::endl;
    }
private:
    virtual void a() override {
        std::cout << "Bar a (private)" << std::endl;
    }
};

int main(){
    Foo foo;
    Bar bar;

    /**
     * Accessing members using a base class pointer
     */
    Foo* base = &foo;
    base->a();      // valid; Foo a (public)
    // base->b();   // error; Foo::b() is private within this context
    
    base = &bar;
    base->a();      // valid; Bar a (private)
    // base->b();   // error; Foo::b() is private within this context
    
    /**
     * Accessing data using static binding
     */
    // bar.a();     // error; Bar::a() is private within this context
    bar.b();        // validl; Bar b (public)

    /**
     * Converting from Bar to Foo would cause slicing
     */
    Foo baz = Bar();
    baz.a();        // valid Foo a (public)
    // baz.b();     // error; Foo::b() is private within this context
}      
```

Even though `base = &bar` might be pointing to a derived object, the access modifier of the base class Foo will still apply. Therefore, executing the private Bar::b() method will work because the same method in Foo has public access.

Similar behaviour can be recreated when using non-virtual methods. The compiler will perform `static binding` and ignore access specified from the derived classes, therefore it will not use polymorphic behaviour.

```c++
class Foo {
public:
    void a() {
        std::cout << "Foo a (public)" << std::endl;
    }
private:
    void b() {
        std::cout << "Foo b (private)" << std::endl;
    }
};

class Bar : public Foo {
public:
    void b() {
        std::cout << "Bar b (public)" << std::endl;
    }
private:
    void a() {
        std::cout << "Bar a (private)" << std::endl;
    }
};

int main(){
    Foo foo;
    Bar bar;

    Foo* base = &foo;
    base->a();      // valid; Foo a (public)
    // base->b();   // error; Foo::b() is private within this context
    
    base = &bar;
    base->a();      // valid; Foo a (public)
    // base->b();   // error; Foo::b() is private within this context
    
    // bar.a();     // error; Bar::a() is private within this context
    bar.b();        // valid; Bar b (public)

    Foo baz = Bar();
    baz.a();        // valid Foo a (public)
    // baz.b();     // error; Foo::b() is private within this context
} 
```

Another example would be having a derived class with a protected member method. The compiler would prevent the 
protected method if it were accessed directly. However, using a reference or a pointer to the base class would override this restriction as the compiler would use the base class access level to determine whether the call to that method is restricted or not. Below is another simple example:

```c++
struct Foo {
    virtual void talk(){
        std::cout << "Foo::talk()" << std::endl;
    }
};

struct Bar : public Foo {
protected:
    virtual void talk(){
        std::cout << "Bar::talk()" << std::endl;
    }
};

int main(){
    Bar bar;
    // bar.talk();      // error; Bar::talk() is protected within this context

    Foo& base = bar;
    base.talk();        // valid; Bar::talk()
}     
```

It would also be possible to lessen the restrictions of derived classes by providing a public method that calls a protected method from a base class.

```c++
struct Foo {
protected:
    virtual void secret(){
        std::cout << "Foo::secret()" << std::endl;
    }
};

struct Bar : public Foo {
public:
    void reveal_secret(){
        secret();
    }
    virtual void secret(){ Foo::secret(); }
};

int main(){
    Bar bar;
    bar.reveal_secret();    // valid; Foo::secret()
    bar.secret();	        // valid; Foo::secret()

    Foo& base = bar;
    base.secret();          // error; protected method using Foo context
}    
```

To keep things simple, an object member access levels depend on the context in which the the object is called from. By using a reference or a pointer to a class, the context of that class will be bind to that class.

#### Inheritance Hierarchy

It is possible to have a polymorphic hierarchy to manage objects of different types belonging to the same hierarchical structure. This is what polymorphism offers in object-oriented programming because it takes advantage of what inheritance offers, allowing multiple objects to adhre to a standard set of properties and beheviour while being used interchangeably.

```c++
struct Animal {
    Animal() = default;
    virtual void id() const { std::cout << "animal id()" << std::endl; }
};

struct Feline : public Animal {
    Feline() = default;
    virtual void run() const {
        std::cout << "Feline run()" << std::endl;
    }
};

struct Bird : public Animal {
    Bird() = default;
    virtual void fly() const {
        std::cout << "Bird fly()" << std::endl;
    }
};

struct Dog : public Feline {
    Dog() = default;
    virtual void speak() const {
        std::cout << "Dog bark()" << std::endl;
    }

    virtual void id() const {
        std::cout << "Dog id()" << std::endl;
    }
};

struct Cat : public Feline {
    Cat() = default;
    virtual void speak() const {
        std::cout << "Cat meow()" << std::endl;
    }

    virtual void id() const {
        std::cout << "Cat id()" << std::endl;
    }

    virtual void run() const {
        std::cout << "Cat run()" << std::endl;
    }
};

struct Pigeon : public Bird {
    Pigeon() = default;
    virtual void speak() const {
        std::cout << "Pigeon sound()" << std::endl;
    }

    virtual void id() const {
        std::cout << "Pigeon id()" << std::endl;
    }
};


int main(){
    Dog dog;
    Cat cat;
    Pigeon pigeon;

    Animal* animals[]{&dog,&cat,&pigeon};
    for(const auto& animal : animals){
        animal->id();
    }

    Feline* felines[]{&dog,&cat};
    for(const auto& feline: felines){
        feline->run();
    }
}    
```


#### Inheritance with static members

Static data members are shared with the derived objects. In order to have unique state, the static data member would need to be declared in the derived class as well. The following example should make this clear:

```c++
struct Foo {
    Foo(){
        m_count++;
    }
    inline static int m_count{0};
};

struct Bar : public Foo {
    Bar() : Foo() {
        m_count++;
    }
    inline static int m_count{0};
};

int main(){
    Foo foos[5];
    std::cout << Foo::m_count << std::endl; // 5
    Bar bar;
    std::cout << Bar::m_count << std::endl; // 1
}  
``` 

#### Virtual methods with default arguments

The compiler will use the default arguments depending on the context of the object. If the derived object is being accessed using a base pointer, the compiler will static bind the default arguments from the base class. However, if the derived class is accessed directly, the compiler will static bind the default arguments of the derived class. 


```c++
class Foo {
public:
    virtual void a(int x = 1, int y = 2) {
        std::cout << "Foo::foo(int,int) " 
            << x << " " << y << std::endl;
    }
};

class Bar : public Foo {
public:
    virtual void a(int x = 10, int y = 20) override {
        std::cout << "Bar::bar(int,int) " 
            << x << " " << y << std::endl;
    }
};

int main(){
    Foo foo;
    foo.a();            // Foo:foo(int,int) 1 2

    /**
     * Using base pointer to invoke virtual function
     * default arguments will be built at compile time
     * using static binding.
     *
     * Rather than having 10, 20, it will statically 
     * compile and bind the values to 1 and 2 respectively.
     *
     */
    Foo* base_ptr = &foo;
    base_ptr->a();      // Foo:foo(int,int) 1 2

    /**
     * The same behaviour can be reproduced with references
     * Static binding will use the default arguments from
     * the base class.
     */
    Foo& base_ref = foo;
    base_ref.a();        // Foo:foo(int,int) 1 2

    /**
     * The compiler will static bind default arguments
     * using the derived class Bar
     */
    Bar bar;
    bar.a();            // Bar::bar(int,int) 10 20


    // slicing 
    Foo baz = bar;
    baz.a();            // Foo::Foo(int,int) 1, 2
    /**
     * Arguments are 
     */
}    
```

#### Virtual Constructors

Constructors cannot be declared virtual. Creating a derived class constructor will call a derived class constructor. The devired constructor then uses the base class constructor, howerver this operation does not affect inheritance and it makes it pointless to be declared as virtual. 

#### Virtual Destructors

The order of destruction is in the reverse order of construction. First the body of the destructor is called, then any data member are destroyed in the reverse order of their construction, finally the parent class is destructed.

```c++
struct Qux {
    Qux(int x): m_x(x){}
    ~Qux(){
        std::cout << "~Qux " << m_x << std::endl;
    }
protected:
    int m_x;
};

struct Foo {
    ~Foo(){
        std::cout << "~Foo" << std::endl;
    }
};

struct Bar : public Foo {
    ~Bar(){
        std::cout << "~Bar" << std::endl;
    }
protected:
    Qux m_qux1{1};
    Qux m_qux2{2};
};

struct Baz : public Bar {
    ~Baz(){
        std::cout << "~Baz" << std::endl;
    }
};


int main(){
    Baz baz;
}
```

The above code would output the following:

```
~Baz	// The body of the constructor is called
~Bar	// The body of the constructor is called
~Qux 2	// Data members are destructed in the reverse order
~Qux 1  
~Foo	// The parent class is destructed
```

Even though the code above compiles and runs fine, the destructor should be declared virtual. Deleting a pointer of a base class pointing to a derived object would only call the destructor of the base object without calling the destructor of the derived class.

The code below would only call the destructor of the Foo object without calling any derived destructors.

```c++
Foo* foo = new Bar();
delete foo;
```

Output:
```
~Foo
```

By making the Foo destructor virtual, it would make the compiler call the constructor of the derived class.

```c+++
virtual ~Foo(){  }
```

Output:
```
~Bar
~Qux 2
~Qux 1
~Foo
```

The same would happen if a pointer to the class Bar would be used to construct an object of type Baz. In this case the compiler would consider the pointer to the Bar class as base class pointer and call the destructor of the Bar class itself, including the its data members, without propagating the call to all destructors.

```c++
Bar* bar = new Baz();
delete bar;
```

Output:
```
~Bar
~Qux 2
~Qux 1
~Foo
```

Setting the parent class destructor virtual would technically solve the problem as it would automatically be used any derived classes. Therefore, setting the destructor of Foo virtual would fix the all the issues above, even though it is better to always make all destructors virtual to avoid making such mistakes in the first place.

It's best recommended to avoid calling virtual functions within constructors and destructors as the order of execution is inverted when objects are pushed and popped from memory, which could then lead to underised results.

```c++
struct Base {
    Base(){
        std::cout << "Base constructor" << std::endl;
        a();
    }
    virtual ~Base(){
        std::cout << "Base destructor" << std::endl;
        b();
    }
    virtual void a(){
        std::cout << "Base::a()" << std::endl;
    }
    virtual void b(){
        std::cout << "Base::b()" << std::endl;
    }
};

struct Derived : public Base {
    Derived(){
        std::cout << "Derived constructor" << std::endl;
        a();
    }
    virtual ~Derived(){
        std::cout << "Derived destructor" << std::endl;
        b();
    }
    virtual void a(){
        std::cout << "Derived::a()" << std::endl;
    }
    virtual void b(){
        std::cout << "Derived::b()" << std::endl;
    }
};

int main(){
    /**
     *******************************
     * The following will output
     *******************************
     * Base constructor     // Base is constructed before Derived
     * Base::a()            // The compiler will statically bind
     *                      //  a() to Base::a() because Derived is still
     *                      // not existing at runtime.
     *
     * Derived constructor  // 
     * Derived::a()         // Derived will call its own version
     * Derived destructor   // Derived is destructed
     * Derived::b()         
     * Base destructor      // The base destructor will call Base::b()
     * Base::b()            // because the compiler won't be able to 
     *                      // call the polymorphic version on Derived
     *                      // since the object is no longer in memory.
     */
    Derived derived;
}
```

#### Final specifier

The final identifier is used to specify that a virtual method cannot be overriden in a derived class or that a lass cannot be derived from. It helps the compiler to ensure that the function is virtual and specified that it cannot be overriden by any derived classes, or it will specify that a class cannot be derived from when used in a class defintion. 

```c++
struct Foo {
    virtual void a() {}
};

struct Bar : public Foo {
    virtual void a() final {}
};

struct Baz final : public Bar {
    virtual void a() override {}    	// error; cannot override final function
};

struct Qux : public Baz {};  		// error; cannot derive from 'final' base 'Baz'

```

#### The typeid operator

The `typeid` operator allows you to get type information of an object at runtime.

```c++
struct Foo {
    virtual void a(){}
};

struct Bar : public Foo {
    virtual void a(){}
};

int main(){
    Bar bar;
    Foo& foo_ref = bar;

    std::cout << typeid(bar).name() << std::endl;       // 3Bar
    std::cout << typeid(foo_ref).name() << std::endl;   // 3Bar

    Foo* foo_ptr = &bar;
    std::cout << typeid(foo_ptr).name() << std::endl;   // P3Foo
    std::cout << typeid(*foo_ptr).name() << std::endl;  // 3Bar 
}
```

The results are implementation-defined. Some compilers like `GCC` will `mangle` or `decorate` the returning names. These can be demangled using the utility `c++filt`. Other compilers might behave differently.

The typeid operator would return different results when using non-polymorphic objects.

```c++
struct Foo {
    void a(){}
};

struct Bar : public Foo {
    void a(){}
};

int main(){
    Bar bar;
    Foo& foo_ref = bar;

    std::cout << typeid(bar).name() << std::endl;       // 3Bar
    std::cout << typeid(foo_ref).name() << std::endl;   // 3Foo

    Foo* foo_ptr = &bar;
    std::cout << typeid(foo_ptr).name() << std::endl;   // P3Foo
    std::cout << typeid(*foo_ptr).name() << std::endl;  // 3Foo 
}
```

As you can see the compiler will simply perform slicing on the data and return an object of the same type. This might lead to undesired results and possible bugs.

### Virtual Base Classes

Sometimes, it is required for an object to avoid ambiguity in the class hierarchy. Consider a base class having a method, which is shared among multiple derived classes. When attempting to invoke this method, the compiler will not be able to select the correct method with error: `request for member 'foo' is ambiguous`. In order to solve this issue, it is possible to inherit a class using the `virtual` keyword.

```c++
struct Animal {
    virtual void sleep() {std::cout << "sleep()" << std::endl;}
};

struct Dog : public virtual Animal {
    virtual void bark() {std::cout << "bark()" << std::endl;}
    virtual void eat() {std::cout << "dog eat()" << std::endl;}
};

struct Bird : public virtual Animal {
    virtual void chirp() {std::cout << "chirp()" << std::endl;}
    virtual void eat() {std::cout << "bird eat()" << std::endl;}
};

struct SpecialAnimal : public Dog, public Bird {
    virtual void eat() {std::cout << "special animal eat()" << std::endl;}
};

int main(){
    SpecialAnimal animal;
    animal.sleep();				// valid; sleep()
}
```

#### Abstract Base Classes

An `abstract` class is a way to represent a general concept, which can then be implemented by derived classes (`concrete` classes). It acts as a blueprint without defining the logic. An abstract class type cannot be instantiated, and it can only be used as a base class. A class becomes `abstract` when one ore more functions are `pure virtual`. Pure virtual functions use the pure-specifier `= 0`, which appears immediately after the declarator or after the optional override or final specifier. 

```c++
struct Foo{
    virtual void a() = 0;	// pure virtual function
};

struct Bar : Foo {
    void a() override {}	// non-pure virtual function
    virtual void b();           // non-pure virtual function
};

struct Baz : Bar {
    void b() override = 0;	// pure virtual function
};

int main(){
    Foo foo;			// error;
}
```

It is possible to define a pure virtual function and the member functions of the derived class can call the abstract's pure virtual function.

```c++
struct Foo {
    virtual void a() = 0;	// pure virtual function
};

struct Bar : Foo {
    void a() override {}	// non-pure virtual function
    virtual void b(){}      // non-pure virtual function
};

struct Baz : Bar {
    void b() override = 0;	// pure virtual function
};

int main(){
    // Foo foo;		// error; cannot declare variable 'foo' 
                        //          to be of abstract type 'Foo'
    Bar bar;            // valid; concrete class
    Foo& foo = bar;     // valid; reference to abstract class
    foo.a();            // valid; virtual dispatch to Bar::a()
}
```

Another example:

```c++
struct Foo {
    virtual void a() = 0;	// pure virtual function
    virtual void b() {      // non-pure virtual function
        std::cout << "Foo::b() non-pure" << std::endl;
    }     
    ~Foo(){
        std::cout << "~Foo" << std::endl;
        b();                // valid; will call Foo::b();
        //a();              // error; undefined behaviour
        Foo::a();           // valid; non-virtual call
    }
};

void Foo::a(){ std::cout << "Foo::a() pure" << std::endl; }

struct Bar : Foo {
    void a() override {
        std::cout << "Bar::a() " << std::endl;
        Foo::a();           // valid; calls pure virtual function
    }	
    void b() override {
        std::cout << "Bar::b()" << std::endl;
    }
    ~Bar(){
        std::cout << "~Bar" << std::endl;
        b();                // valid; calls Bar::b();
        a();                // valid; calls Bar::a();
    }
};

```
When `~Bar` will be called, `Bar::b()` will be executed first. `Bar::a()` would then be executed, which in turn will make a call to `Foo::a()` pure virtual function. When `~Foo` will destruct, `Foo::b()` will be invoked followed by 
a call to the pure function `Foo::a()`. In this case both destructors will be able to access `Foo::a()` pure virtual function.

#### Abstract Classes as Interfaces

Any base class that should not define a behaviour could be used as an interface. A class having pure virtual methods without defining the body of the method are considered to be interfaces.

```c++
struct Printable {
    friend std::ostream& operator<< (std::ostream& lhs, const Printable& rhs);
    virtual void print(std::ostream& lhs) const = 0;
};

std::ostream& operator<< (std::ostream& lhs, const Printable& rhs) {
    rhs.print(lhs);
    return lhs;
}

struct Foo : Printable {
    virtual void print(std::ostream& os) const override {
        os << "foo@" << this;
    }
};

struct Bar : Printable {
    virtual void print(std::ostream& os) const override {
         os << "bar@" << this;
    }
};

int main(){
    Foo foo;
    Bar bar;

    std::cout << foo << std::endl;		// valid; prints foo@0x00...
    operator<<(std::cout,bar) << std::endl;	// valid; prints bar@0x00...
}
```
