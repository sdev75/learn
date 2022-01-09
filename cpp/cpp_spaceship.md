## Spaceship Comparison Operator (C++20)

It's not a real spaceship, but it does look like one. The operator `<=>` is a new operator introduced in C++20. It greatly reduces the number of overloads required  to provide support for all the six relational comparison operators and instructing the compiler to define all the comparison operators automatically.

In order for an object of type T to support ordering and to be comparable with another type, it would require you to define all the comparison operators manually.

```c++
friend constexpr bool operator<(const Foo&, const Foo&);
friend constexpr bool operator==(const Foo&, const Foo&);
friend constexpr bool operator<=(const Foo&, const Foo&);
friend constexpr bool operator!=(const Foo&, const Foo&);
friend constexpr bool operator>(const Foo&, const Foo&);
friend constexpr bool operator>=(const Foo&, const Foo&);
```

That's quite a lot of work to do for some basic comparison. C++ introduced the `operator<=>`, also known as the `three-way comparison operator` which instructs the compiler to generate relational operators for automatically and simplify the code as follows:

```c++
friend auto operator<=>(const Foo&, const Foo&) = default;
```

This will implement general behaviour for all the comparison operators such as `==`,`!=`,`<`,`>`,`<=`,`>=`.

The `spaceship operator` returns a value of comparison category type, which is similar to an enumeration-like value type, rather than a plain `Boolean` type. However, the type returned by the `spaceship operator` cannot be used in a switch statement or using enum declarations. In fact, unlike the other relational operators which only evaluate to `true` or `false`, the `three-way comparison` operator will tell the program wether a value is equal, less than or greather than another value. If the operands are integral types, the result will be using `strong ordering`.

The result value returned by the `three-way comparison` can be compared using a literal zero value as follows:

```c++
int a{1};
int b{2};

(a <=> b > 0);    // gt
(a <=> b >= 0);   // gteq
(a <=> b == 0);   // eq
(a <=> b != 0);   // neq
(a <=> b < 0);    // lt
(a <=> b <= 0);   // lteq
```

### Strong Ordering

Strong ordering is returned by the `spaceship operator` when the operands are of integral type. They support all six relational operators. Strong ordering will return either one of the following types, depending on the result achived:

- strong_ordering::less - Left operand is less than the right operand
- strong_ordering::greater - Left operand is greater than the right operand
- strong_ordering::equal - Both operands are equal

```c++
std::strong_ordering res { 11 <=> 10 };
```

In the case above, the result of the three-way comparisong expression will be `std::strong_ordering::greater`, identifying the left integral type operand to be greater than the right operand.

The result cannot be used in a switch statement, but it can be used on a normal if-else statement:

```c++
if (res == std::strong_ordering::less) {}
if (res == std::strong_ordering::greater) {}
```
    
### Partial Ordering

Partial ordering is used when the operands are floating-point types.
In that case the result will be one of the following:

- partial_ordering::less - Left-hand side operand less than the right-hand side operand
- partial_ordering::greater - Left operand greater than the right operand
- partial_ordering::equivalent - Both operands are equal
- partial_ordering::unordered - At least one operand is not a number

Consider comparing a `floating-point` type with a `nan`. A nan is a value used to identify undefined or non-representable floating-point data types. In which case it is clearly not a floating-type value but a rather a unique `not-a-number` type. The result using partial_ordering::unordered will fit this need.

Consider the following code comparing a double to a Nan:

```c++
double foo{1.2};
double bar{std::numeric_limits<double>::quiet_NaN()};
foo > bar;
```

In the case above, the expression `foo > bar` would return false, because `NaN` cannot be compared to a number. Thus any comparison would return `false` boolean type.

Partial ordering could also be used to compare two different objects, such an employee object which could be a the same as another employee, or the manager of another employ, or the employ of another manager. Meaning that it could be equivalent, less or greater than or unordered if the employee would not belong to the same office division. 

### Weak Ordering

Weak ordering is used when implementing comparisong for custom types. In that case the following return values will be used as result:

- weak_ordering::less - Left operand is less than the right one
- weak_ordering::greater - Left operand is greater than the right one
- weak_ordering::equivalent - Both operands are equal

Consider two strings having different case sensitivity, they might be looking alike, but internally they are using totally different data such as `hello` and `HELLO`. In that case both strings might be considered equivalent but not equal. Weak ordering could be useful for comparing two equivalent data types.
