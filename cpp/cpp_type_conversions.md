## Automatic conversions and type casts
C++ automatically attempts to convert values from one type to another, provided that both the variables are compatible. A common type conversion happens with double (33.3) to int (3), losing precision.

C++ provides three mechanisms to perform type conversions:

### Type conversion operator

When the compiler attempts to conver an object of type Bar to Foo, 
it will attempt to do this using the type conversion operator first.

```c++
Foo::operator Bar() const { return Bar(); }
```

### Constructor conversion
Another type of conversion used by the compiler is by using a class constructor 
that has a single argument used for converting the value passed in as argument to another class type.

The use of the explicit keyword eliminates implicit conversions by the compiler.

```c++
explicit Foo::Foo(const Bar& bar){}
```

### Copy assignment operator
The final step the compiler will attempt is the conversion using the copy assignment operator.

```c++
void Foo::operator= (const Bar& rhs){}
```

## Order of conversions

Consider the following example having all three mechanisms implemented to our Foo class. 

```c++
class Bar;

class Foo {
public:
    Foo(){ std::cout << "Foo constructor" << std::endl; }
    ~Foo() = default;

    // Copy constructor
    Foo (const Bar& bar) {
        std::cout << "copy ctor" << std::endl;
    }

    // Copy assignment operator
    void operator= (const Bar& rhs) {
        std::cout << "copy assignment operator" << std::endl;
    }
};

class Bar {
public:
    Bar(){ std::cout << "Bar construtor" << std::endl; }
    ~Bar() = default;

    // Type conversion operator
    operator Foo() const {
        std::cout << "type conversion operator Bar to Foo" << std::endl;
        return Foo();
    }

};

```

#### 1. Copy assignment operator
Running the following code would make the compiler to choose the copy assignment operator `void operator= (const Bar&)` first. 

```c++
Foo foo{};
Bar bar{};

/* 
 * Copy assignment operator has higher priority
 * void operator= (const Bar&) 
 */ 
foo = bar;  
```

#### 2. Copy constructor
If we were to comment out the copy assignment operator from the class, the compiler would then use the copy constructor `Foo (const Bar&)`.

```c++
Foo foo{};
Bar bar{};

/*
 * Copy assignment constructor has been commented out
 * Copy constructor has higher priority
 * Foo (const Bar&)
 */ 
foo = bar;  	
```

#### 3. Type conversion operator
Commenting out the copy assignment operator and the copy constructor would force the compiler to use the type conversion operator `Bar::operator Foo() const`. This would allow the class of type Bar to be converted to class of type Foo.

```c++
Foo foo{};
Bar bar{};

/**
 * Copy constructor has been commented out
 * Copy assignment operator has been commented out
 * Type conversion operator has higher priority
 * Bar::operator Foo() const
 *
 */
foo = bar;  	// Bar::operator Foo() const
		// Foo constructor called
            
```

Another case scenario would be if we had implemented a copy assignment operator for the class Foo within the class Foo. This would make the compiler invoke the `copy assignment operator` rather than only calling the `Foo constructor` as see in the previous example. 

Consider the implementation of the Foo copy assignment operator as follows:

```c++
Foo& Foo::operator= (const Foo& rhs){
    std::cout << "Foo copy assignment operator" << std::endl;
     return *this;
}
```

In this case, the compiler would use the type conversion operator to convert the Bar object to a Foo object. This would invoke the Foo constructor to create a new Foo object and then it would call the copy assignment operator `Foo::Foo& operator= (const Foo& rhs)` to perform the final assignment as per the code below:

```c++
Foo foo{};
Bar bar{};


/**
 * Copy constructor has been commented out
 * Copy assignment operator has been commented out
 * Type conversion operator has higher priority
 * 1. Bar::operator Foo() const
 * 2. Foo() constructor
 * 3. Foo& operator= (const Foo& rhs)
 *
 */
foo = bar;  	// Bar::operator Foo() const
		// Foo() constructor
		// Foo& operator= (const Foo& rhs)
            
```

## Function parameters and type conversions

Another case of type conversion is during the creation of temporary objects passed in as parameters. 

Consider the previous class defintion without any of the code commented out and the statements below. In this case the compiler would construct the Bar object using its constructor as normal and then it would call the `Foo (const Bar& bar)` to perform the conversion to a Foo object as per the baz function signature's requirements.

```c++
void baz(const Foo& foo){}

Bar bar{}; 	// Bar constructor
baz(bar); 	// Copy constructor Foo::Foo (const Bar& bar) 
```
If we were to comment out the Foo copy constructor `Foo::Foo (const Bar& bar)`, the compiler would then use the type conversion operator to perform the conversion `Bar::operator Foo() const`.

```c++
void baz(const Foo& foo){}

/**
 * Foo's copy constructor has been commended out
 * The Bar object would be converted to a Foo object
 *      using : `Bar::operator Foo() const`
 *  returning : new constructed Foo object
 */

Bar bar{}; 	// Bar constructor
baz(bar); 	// Type conversion operator (Bar to Foo)
		// A new Foo object would be constructed and passed in
```

