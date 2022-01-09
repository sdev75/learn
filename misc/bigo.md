## Big-O notation

Big-O notation is used to describe the performance of an algorithm. Some algorithm take linear time and some take quadratic time in the worst case. This helps to determine which algorithm is the fastest and most efficient based on the requirements and give an approximation on the worst performance possible.

#### O(N) - Linear time

Considering the example below. The algorithm is using a complexity of O(N). That means that in order to find an element in the list, it will take up to N times, which in this case, it corresponds to the size of the list.

```python
foo = ["bar"]
for item in foo:
    if item == "bar":
        print("found bar")
```

Becase the array has only 1 item, this code is using a constant time O(1) complexity. Accessing an array by index is also referred to as O(1) time or constant time regardless of the size of the array because the algorithm would take only one step to execute.

#### O(n^2) - Quadratic time

The following code is using two nested loops to iterate through each and every item. Thus accessing 3 items would result in total iteration of 3^2 (9 iterations in total).

```python
foo, bar = [1,2,3], []
for item in foo:
    for subitem in foo:
        bar.append((item,subitem))

print(len(bar)) # 9
```

#### O(2^n)

A common example for the O(2^n) would be the fibonacci function, which uses an algorithm that grows exponentially with every addition to the input data set.

```python
def fibonacci(num: int):
    if not num:
        return num
    # fn associativity L to R
    return fibonacci(num - 2) + fibonacci(num - 1)
```
