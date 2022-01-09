## Memory handling with Smart Pointers

Unlike some languages which make use of memory garbage collection, C++ memory management requires the programmer to perform manual memory allocation and deallocation when storing objects in memory heap. Memory allocated on the stack however, is automatically freed when the object goes out of scope.

### Smart Pointers to the rescue

The C++ standard library provides a collection of useful objects for managing pointers allocation and deallocation at runtime by leveraging the `RAII` idiom (Resource Aquisition Is Initialization), which is a C++ programming technique to guarantee the resource aquired by an object is available to any function that may access the object itself. It also gurantees that all the resources aquired by that object are released when the lifetime of the object ends. It is a way to encapsulate a resource within a class, where the constructor will perform the aquisition and the destructor the releasing.

The standard library offers several RAII wrappers to manage user-provided resources.

- `std::unique_ptr`
- `std::shared_ptr`
- `std::weak_ptr`

#### std::unique_ptr

The `unique_ptr` smart pointer provides a semantics of strict ownership, which destroys the aquired object when the object goes out of scope. The `unique_ptr` cannot be copies, it doesn't have a copy constructor or an assignment operator, making the object strict in away that there can only be 1 instance of the pointer throughout the execution of the program.

```c++
std::unique_ptr<Foo> foo{new Foo};			// allocate using new operator
std::unique_ptr<Foo> foo = std::make_unique<Foo>();	// allocate using make_unique
```

The `unique_ptr` class will automatically deallocate the data pointed to by invoking the delete operator when the unique_ptr goes out of scope. This would remove the burden of deallocating the pointer by the programmer and improve code reliability and safety.

It is also possible to access the raw pointer wrapped by the `unique_ptr` class. The `release()` method will give back ownership of the raw pointer. Once released the pointer will have to be deallocated manually by the programmer using the `delete` operator.

```c++
Foo *foo = res.get();
delete foo;
```

It is possible to access the raw pointer using the `get()` method. 

```c++
std::unique_ptr<Foo> foo{new Foo};	// shared_ptr 
*foo;					// deferencing raw pointer
foo.get();				// address of raw pointer
```

It is not possible to copy ownership of the `unique_ptr`, however it's possible to move ownership between two unique pointers. 

```c++
std::unique_ptr<int> a = std::make_unique<int>(1);	// unique_ptr pointing to heap
std::unique_ptr<int> b {nullptr};			// unique_ptr pointing to 0
b = std::move(a);  					// move ownership of a to b
```

By leveraging the use of the `move semantics`, the unique pointer stored in `a` would be transferred to `b`, thus invalidating the unique pointer `a` by resetting it to nullptr. In this case `b` would gain total ownership of the raw pointer.

Unique pointers can also be used to store dynamically allocated arrays.

```c++
std::unique_ptr<int[]> array = std::unique_ptr<int[]>(new int[2]{10,20});
array[0];	// 10
array[1];	// 20
```

#### std::shared_ptr

Unlike unique pointers, shared pointers can be copied around and passed in as value in a function call without deallocating the resources. Shared pointers keep track of all the references shared using an internal counter. The underlying object stored by a shared pointer will be destructed and deallocated only when all the reference count reaches zero, meaning that all the shared pointers will have ran out of scope.

```c++
void foo(std::shared_ptr<int> ptr){
    // use_count() = 2 when copying 
    // use_count() = 1 when out of scope
}

int main(){
    std::shared_ptr<int> a {new int{1}};
    {   
	// copying a shared_ptr increases `use_count`
        std::shared_ptr<int> b = a;
        
        *b = 100;
    }
    // use_count is decreased when exiting a scope
    foo(a);     
    // use_count will reach zero, thus deallocating memory
}
```

It is possible to transfer ownership of a unique pointer to a shared pointer by leveraging move semantics. This would give the flexibility to share the pointer without having to change the design of the application.

```c++
std::unique_ptr<int> a = std::make_unique<int>(10);	
std::shared_ptr<int> b = std::move(a);			
```

The unique pointer `a` will be invalidated as soon as the ownership is transfered using `std::move()`. 

#### std::weak_ptr

Weak pointers do not own the data they point to. The memory is not deallocated when a weak_ptr goes out of stock. They cannot be used for read or writing data pointed to. The main use of `weak_ptr` is to be able to access data by turning them into different smart pointers for manipulation. This is done with the `lock()` method offered by the weak_ptr.


```c++
std::shared_ptr<int> a = std::make_shared<int>(10);
std::weak_ptr<int> b(a);
std::shared_ptr<int> c = a.lock();
```

The most useful feature of weak pointers is to solve cyclic dependency. Consider the following example where both objects would hold shared pointers of each other, creating a cycling dependency:

```c++
/**
 * Cyclic dependency and shared pointers
 */
struct Foo {
    void setFoo(std::shared_ptr<Foo> foo){
        foo_ = foo;
    }
    std::shared_ptr<Foo> foo_ {nullptr};
};

int main(){
    std::shared_ptr<Foo> a = std::make_shared<Foo>();   // use_count = 1
    std::shared_ptr<Foo> b = std::make_shared<Foo>();   // use_count = 1

    /**
    * Assigning the objects to each other would cause 
    * cyclic dependency. The pointers would 
    * always have a `use_count` reference greater than zero
    * thus causing a memory leak due to the inability to
    * deallocate memory correctly.
    *
    *   a  use_count 1            b use_count 1
    *   |                         |
    *   b  use_count 2            a use_count 2
    *
    */

    /*
     * `b` will be assigned to `a`
     *   `a` use_count = 1
     *   `b` use_count = 2
     */
    a->setFoo(b);

    /*
     * `a` will be assigned to `b`
     *   `b` use_count = 2
     *   `a` use_count = 2
     */
    b->setFoo(a);


    /**
     * `b` will go out of scope
     * `a` will not be destroyed because it will be pointing to `b`
     *   `a` use_count = 2
     *   a               b
     *   |               |
     *   |               x
     *   |               |
     *   b               a still points to b 
     *                   a will keep use_count = 2
    */
        
    std::cout << a.use_count() << std::endl; // use_count = 2
    std::cout << b.use_count() << std::endl; // use_count = 2

    /**
     * Finally a and b will go out scope
     * use_count will remain to 1 and the memory will leak
     */
} 
```
 
Using weak pointers would prevent the use_count from increasing and allowing to have both items hold a copy of the pointer pointing to each other without leaking any memory.

```c++
std::weak_ptr<Foo> foo_;
```

This will prevent the use_count to increase and thus the memory will be deallocated when both a and b will go out of scope.