## STL Components

The C++ Standard Library has been improving in the last years, now a new feature called ranges has been introduces to help keeping the coding constrained and less error prone for the programmer.

As you know, the STL library is built in separate components, which can cooperate to work together. The STL is an example of `generic programming`. It allows to write code that is indipendent of data types that work interchangeably with different algorithms.

### Containers

Containers are used to manage collections of objects or elements. The implementation might different based on the type of container used. The most commonly used containers are divided in 2 types:

- Sequence Containers
- Associative Containers

#### Sequence Containers

Sequence containers are a collection of ordered elements which are put in a sequential order in memory. Common sequence containers: `vector`, 'deque', `list`.

##### Vectors

Vectors manage elements within the collection in a dynamic array. They can be accessed randomly by index. Appending and removing elements is considered to be very fast using the `O(1)` notation.

##### Deque

Deque is a double-ended queue, which is stored in a dynamic array allowing its elements to be added and removed in both directions. The insertion and deletion of element at each boundary is considered fast.

##### Lists

A list is a doubly linked list which allows its elements to be connected like a chain of memory chunks. In order to access an element at a specified index, the list needs to be iterated though every elements by following the chain of links. Insertion and removal is quite fast, and access to its element takes linear time, which means the distance is proportional to the number of elements, thus accessing an element which is 100 blocks away, will take 100 iterations, and accessing an element which is 1 block away would take 1 iteration. This is also referred to as `O(n)` notation. Moving an element in the middle of a list is also very fast, as it only requires the modification of the links within that block.

#### Associative Containers

Associative containers are collections with the ability to be sorted using random access iterators. Unlike sequence containers, the memory layout is not contiguous. Common associative containers: `set`, `multiset`, `map`, `multimap`.

##### Sets and Multisets

A set is a collection of elements, in which these elements are sorted according to their own distinct value (duplicate are not allowed). A multiset on the other hand allows elements to have multiple values.

##### Maps and Multimaps

A map is a collection of elements stored in key/value pairs. Keys are distinct values which are used to perform lookup algorithms. Maps are often used as `associative arrays`, which allow to access values by arbitrary index types (keys). A multimap allows multiple to have keys with the same key, thus allowing duplicated keys.

### Container Adapters

The Standard Library provides an additional predefined container adapters. Container adapters act as a wrapper around sequential containers.

#### Stacks

A stack provides FILO (first-in-last-out) semantics. It is almost identical to `queues` except for the order in whcih its elements are pushed and popped.

#### Queues

The `queue` container adapter managed its elements using FIFO semantics (first-in-first-out). The queue interface provides several methods and a constructor including some comparisong operators.

#### Priority Queues

A priority queue keeps its elements sorted in order by different priorities. Instead of enforcing a strict FIFO ordering, the element at the head of the queue has the highest priority. The priority is based on a sorting criterion. This can be provided otherwise the `operator<` will be used instead. It's basically a buffered list, which always contains the next element with the highest priority.

### Iterators

Iterators are an essential component to STL's generic programming. Iterators make algorithms indipendent of the type of container used. They provide a common interface to step through the elements stored in a collection of objects. 

### Algorithms

Algorithms provide a way to process the elements of collections. They can search, modify or sort data stored in collections. Algorithms work mutually with iterators to implement the perform the actual logic behind the algorithms. This allows each containers to be implementing different algorithms, using different implementation of an iterator.

### Iterator Traits

Different algorithms require different information about an iterator. Iterators are subdivided into five categories to identify the functionality they implement. Iterators form a hierarchy, as such as a forward iterator has all the capabilities of an inpt iterator and of an output iterator and its own capabilities. A bidrectional iterator has all the capabilities of a forward iterator, plus its own capabilities and so on.

#### Input Iterators

An `input iterator` also known as `read iterator` is a one-way iterator which can read-only values incrementally, it cannot move backwards (operator--), only forward. This functionality is supported by implementing both the prefix `operator++` and the postfix `operator++(int)`. Input iterators can be assigned, copied and compared for equality, thus supporting `operator=`, `operator==` and `operator!=` respectively.

The C++ compiler excepts the following properties to be defined when dealing with an iterator:

##### iterator_category

An iterator can be categorized to fit specific needs. An input iterator uses the `std::input_iterator_tag` category.

```c++
using iterator_category = std::input_iterator_tag;
```

##### difference_type

Difference type is used to perform pointer arithmetic when iterating though the container. Containers use the difference_type member with `std::ptrdiff_t` sa default. `std::ptrdiff_t` is a signed integer type used for pointer airithmetic and array indexing.

```c++
using difference_tyoe = std::ptrdiff_t;
```

##### value_type

Identifies the iterator value

```c++
using value_type = T;
```

##### pointer and reference types

Define a pointer and a reference to the type iterated over:

```c++
using pointer_type = value_type*;
using reference_type = value_type&;
```

It is possible to create custom iterators for by defining an `Iterator` or `ConstIterator` nested class depending on the use case requirements.

```c++
template <typename T>
requires std::is_default_constructible_v<T>
class Foo {
public:
    Foo (std::initializer_list<T> l) {
        std::cout << "Constructor called" << std::endl;
        m_items = new T[l.size()];
        m_size = l.size();
        for(size_t i{}; auto item : l){
            m_items[i++] = item;
        }
    }
    ~Foo() {
        delete[] m_items;
        m_size = 0;
    }
private:
    T* m_items;
    size_t m_size;

public:
    class Iterator {
    public:
        using iterator_category = std::input_iterator_tag;
        using difference_type = std::ptrdiff_t;
        using value_type = T;
        using pointer_type = value_type*;
        using reference_type = value_type&;
        Iterator() = default;
        Iterator(pointer_type ptr) : m_ptr(ptr) {}
        reference_type operator* () const { return *m_ptr; }
        pointer_type operator-> () const { return m_ptr; }
        Iterator& operator++ () { m_ptr++; return *this; }
        Iterator operator++ (int) { 
            Iterator tmp = *this; 
            ++(*this);  
            return tmp; 
        }

        friend bool operator== (const Iterator& lhs, const Iterator& rhs){ 
            return lhs.m_ptr == rhs.m_ptr; 
        }
        friend bool operator!= (const Iterator& lhs, const Iterator& rhs){
            return lhs.m_ptr != rhs.m_ptr;
        }
    private:
        pointer_type m_ptr;
    };

    Iterator begin() { return Iterator(&m_items[0]); }
    Iterator end() { return Iterator(&m_items[m_size]); }
};

int main(){
    Foo<int> foo{10,20,30,40,50};
    for(auto item : foo){
        std::cout << item << std::endl;
    }

    for(auto it=foo.begin(), end=foo.end(); it != end; it++){
        std::cout << *it << std::endl;
    }

    const int n = 40;
    std::find(foo.begin(),foo.end(),n); 
} 

```

#### Output Iterators

Output iterators are used for transferring data from a program to a container. Output iterators provide write-only access, forward only. THey can be assigned, but they cannot be compared for equality. They provide both prefix and postfix `operator++`, `operator++(int)`. They are copy constructable and support writing to the value referenced to by the pointer using `operator*`.

The code used in the input iterator implements the `operator*` which is enough in order for algorithms to write data. The code below would compile and work fine:

```c++
using iterator_category = std::output_iterator_tag;	// category used for output iterator

Foo<int> foo{10,20,30,40,50};
Foo<int> bar{0,0,0,0,0};
std::copy(foo.begin(), foo.end(), bar.begin());		// valid;
std::replace(foo.begin(),foo.end(),30,100);		// valid;
```

#### Forward Iterators

Like input and output iterators, forward iterators nagivate though the container forward-only. A forward iterator can be assigned, copied and compared for equality. Unlike input and output iterators, a forward iterator will always go through a sequence of values in the same order it is used, allowing multi-pass algorithms to reand and write the elements of the container multiple times.

```c++
using iterator_category = std::forward_iterator_tag;
```

#### Bidirectional Iterators

Bidirectional iterators allow algorithms to trasverse a container in both directions. Consider a reverse function that will go though the container backwards or swap elements around. A bidirectional iterator has all the features of a foreward iterator plus the prefix `operator--` and postfix `operator--(int)`.

```c++
using iterator_category = std::bidirectional_iterator_tag;
Iterator& operator-- () { m_ptr--; return *this; }
Iterator operator-- (int) { Iterator tmp = *this; --(*this);  return tmp; }
```

#### Random Access Iterators

Some algorithms require the ability to access artibitrary element of a container, thus `random access`. A random access operator has all the features of a bidirectional iterator, plus it it adds operations to support random access and relational operators as follows: `operator+`, `operator-`, `operator+=`, `operator-=`, `operator<`, `operator>`, `operator<=`, `operator>=`, `operator[]`. This gives the container the ability to access its elements with pointer arithmetic, array index syntax and all comparison operators.

```c++
 using iterator_category = std::random_access_iterator_tag;

Iterator operator+ (const difference_type offset){ Iterator tmp = *this; return tmp += offset; }
Iterator operator- (const difference_type offset){ Iterator tmp = *this; return tmp -= offset; }

Iterator& operator+= (const difference_type offset){ m_ptr += offset; return *this; }
Iterator& operator-= (const difference_type offset){ return *this += -offset; }

difference_type operator- (const Iterator& rhs) const { return m_ptr - rhs.m_ptr; }
reference_type operator[] (const difference_type offset) const { return *(*this + offset); }

bool operator< (const Iterator& rhs) const { return m_ptr < rhs.m_ptr; }
bool operator> (const Iterator& rhs) const { return m_ptr > rhs.m_ptr; }
bool operator<= (const Iterator& rhs) const { return !(rhs < *this); }
bool operator>= (const Iterator& rhs) const { return !(*this < rhs); }
     
friend Iterator operator+ (const difference_type offset, const Iterator& it){
    Iterator tmp = it;
    return tmp += offset;
}
```

The above implementation would make sorting and other algorithms that require `random access` work.

#### Contiguous Iterators

Contiguos iterators provide random-access capability for items adjacent in memory, such as `std::array`, `std::vector`, `std::string` and `std::string_view`.

### Stream Iterators

The STL provides some predefined stream iterators. Stream Iterators allow you to handle input and output as input and output iterators and serve as source and destination for algorithms.

#### Output Stream Iterator

The `ostream_iterator` is used to write elements using the `operator<<`. Consider copying data to an output stream  to write to the display. This could easily be achived by using the `std::copy` along with an `ostream_iterator`. 

```c++
std::vector<int> foo{10,20,30,40,50};
std::copy(foo.begin(),foo.end(),std::ostream_iterator<int>{std::cout,","}); // 10,20,30,40,50,
```

#### Input Stream Iterator 

The `istream_iterator` would act as a model of the input iterator and use to read data from a source.

```c++
std::vector<int> foo{10,20,30,40,50};
std::copy(std::istream_iterator<int, char>(std::cin), 	// read from cin
        std::istream_iterator<int, char>(),         	// read until interruption
        foo.begin()); 					// destination = foo
```

### Ranges (C++20) 

With the introduction of `ranges` library, it is possible to omit passing in the begin and end of a container and simply using the container itself to iterate through a container. It automatically provides a begin iterator and an end sentinel. Ranges also enforce constraints using concepts to better organize code and give out better error messages. Ranges can also be accompanied using range adapters to gain powerful functionality. Any objects data support `begin()` and `end()` methods are considered valid ranges.

```c++
namespace ranges = std::ranges;
std::vector<int> foo{10,20,30,40,50};
ranges::for_each(foo,[](int& n){ n *= 2; });				// for_each range-based algorithm
ranges::copy(foo,std::ostream_iterator<int>{std::cout, " "}); 		// copy range-based algorithm
```

One could also implement the `three-way comparison operator` (<=>) to fine tune the outcomes of the comparison algorithms.

```c++
bool operator== (const Foo& rhs) const {
    return (priority == rhs.priority);
}
std::partial_ordering operator<=> (const Foo& rhs) const {
    if(priority > rhs.priority)
        return std::partial_ordering::greater;

    else if(priority == rhs.priority)
        return std::partial_ordering::equivalent;

    else if(priority < rhs.priority)
        return std::partial_ordering::less;

    else
        return std::partial_ordering::unordered;
}
ranges::sort(list, std::equal_to{}); 
```

### Projections 

Projections are introduced in C++20. These callbacks allow range-based algorithms to receive projected values. A projection is a callback which transforms each element before it is passed to the algorithm. Consider the following example:

```c++
struct Foo {
    size_t priority {0};
    int a;

    size_t getPriority() const { return priority; }
};

namespace ranges = std::ranges;
std::vector<Foo> list {{10, 1}, {10, 2}, {100, 1}, {99, 3}};

/**
    * The lambda function is used to return projected data
    * The projected data is passed into the comparison algorithm
    * The result of the algorithm is passed into the sorting algorithm.
    * 
    * Projections allow data filtering granularity and flexibility
    */
ranges::sort(list, ranges::greater{}, [](auto const& foo){return foo.priority; }); 

/**
    * Projections also work with direct public member variables
    * They also support member functions as well.
    */
ranges::sort(list, ranges::greater{}, &Foo::priority); 
ranges::sort(list, ranges::less{}, &Foo::getPriority); 

ranges::for_each(list, [](Foo& foo) { 
    std::cout << foo.a << ", priority: " << foo.priority << '\n'; 
});

/**
 * Using projections with the range-based for_each algorithm
 */
std::vector<std::pair<int, double>> pairs {
    {15, 1.25}, {10, 99.99} 
};
ranges::for_each(pairs,[](const auto &n){
    std::cout << " " << n;
}, &std::pair<int, double>::second);
```

### Views

Range-based algorithms support filtering of the data using the so called `views`. A `view` can be used to transform the elements of an underlying range and they can be composed with other views to perform powerful pipilined operations.

Views allow the compiler to perform operations on the underlying range's element without actually being evaluated at the time of construction. Views are lazy evaluated during iteration of the elements. Views do not own any elements, they merely provide a window to view the data in different ways without mutate the data in the underlying range.

#### Filter Views

Filter view uses a predicate to construct views to filter data when iterating over the elements of the container.

```c++
/**
* A filter_view will not perform any computation, 
* it will merely setup the view to be used later on
* when iterating through the elements of the collection.
* 
* In the example below we use a predicate to determine 
* if the element of our list is an odd number
*/
ranges::filter_view view = ranges::filter_view(list, [](int n){ return (n%2) != 0; });

for(auto n : view){ std::cout << n << std::endl; }
```

#### Transform Views

Transform views instruct the compiler on how to transform the data during iteration.

```c++
ranges::transform_view view = ranges::transform_view(list, [](int n){ return n*=2; });
```

#### Other Views

The range library provides with more views such as `take_view`, `drop_view`, `take_while_view` and so on. All of which help to make the creation of views an easy process.

#### Views Composition

Views can be compounded using the pipe operator. This will allow to pass data from one view to another view and combine outputs.

Consider using raw function composition by passing filters within other filters as follows:

```c++
/**
 * Raw function composition without using the pipe operator
 */
auto filter = [](int n){ return n%2 == 0; };
auto transform = [](int n) { return n*=2; };
auto view = std::views::transform(
    std::views::filter(list,filter), transform);

/**
 * During the iteration process the view would start to 
 * evaluate using our `filter` function first, followed
 * by our `transform` function. This will repeat for 
 * every element in our collection.
 */ 
for(auto n : view){ std::cout << n << std::endl; }
```

A better approach is to use `function composition` using the pipe operator.

```c++
/**
 * Function composition using pipe operator
 * The view will filter out the data, reverse it 
 * and then transform it, in the same order.
 */
auto filter = [](int n){ return n%2 == 0; };
auto transform = [](int n) { return n*=2; };
auto view = std::views::filter(list,filter) | 
    std::views::reverse | 
    std::views::transform(transform);

ranges::copy(view, std::ostream_iterator<int>(std::cout, " \n"));

```

### Range Factories

The `range` library provides range factories to construct views that produce elements evaluated on demand. A common range factory being used is the `iota_view`.

```c++
/**
 * Take 10 elements out of an infinite iota view.
 */
auto view = std::views::iota(1) | std::views::take(10);
for(auto i : view){ std::cout << i << std::endl; }
```

Range factories could be used to generate numbers and perform transformation on it.

```c++
/**
 * Create infinite values starting from number 5 inclusive
 */
auto data { std::views::iota(5) };

/**
 * Filter out the values and trasnform them
 * We loop through them and take only 5
 */
auto view { 
    data | std::views::filter([](const auto& val){ return val % 2 == 0; })
    | std::views::transform([](const auto& val) { return val * 2; })
};
 
for(auto i : view | std::views::take(5)){ std::cout << i << " "; } 	// valid; 12 16 20 24 28 
```

The `data` variable represent an infinite range, which is subsequently filtered and then transformed. These operations are lazily evaluated when the range iterates over the elements of the view, in this case it is evaluated in our range-based for loop.





