# HashSet: A Custom C++ `std::unordered_set<int>` Implementation

This project is a C++ implementation of a simplified, integer-only hash set, similar to `std::unordered_set<int>`. It was developed as part of a programming assignment to demonstrate a deep understanding of hash table mechanics, including collision handling, dynamic resizing, and efficient iterator design.

The implementation is split into two levels:
*   **Level 1:** A foundational resizable hash table using separate chaining.
*   **Level 2:** An advanced version with support for constant-time bidirectional iterators.

## Key Features

### Level 1: Core Functionality
*   **Separate Chaining:** Resolves hash collisions by storing elements that hash to the same bucket in a sequence container (e.g., `std::list`).
*   **Dynamic Resizing:** Automatically grows the hash table's capacity when the load factor exceeds a user-defined maximum.
*   **Prime Number Buckets:** Resizing chooses the next available prime number from a predefined sequence to improve key distribution.
*   **Core Set Operations:** Supports `insert`, `contains`, and `erase` with average-case constant time complexity.

### Level 2: Iterator Support
*   **Bidirectional Iterators:** Provides `begin()` and `end()` to traverse the set. Iterators support `++`, `--`, `*`, `==`, and `!=`.
*   **Constant-Time Iteration:** Incrementing and decrementing iterators is a guaranteed O(1) operation, even across empty buckets.
*   **Efficient Find and Erase:** Includes iterator-based `find(key)` and `erase(iterator)` methods with average-case constant time complexity.

## Implementation Details

### Level 1: Resizable Hash Table with Chaining

The Level 1 `HashSet` is built upon a C-style array (`std::list<int>* buckets`). Each element in the array represents a "bucket". When an integer is inserted, its hash is calculated to determine its bucket index. The integer is then stored in the `std::list` at that index.

**Load Factor and Rehashing**
The load factor is a measure of how full the hash table is, calculated as:

$$ \text{load\_factor} = \frac{\text{number of elements}}{\text{number of buckets}} $$

The `HashSet` maintains a `max_load_factor`. If an insertion causes the current load factor to exceed this maximum, a `rehash` operation is triggered. This involves:
1.  Allocating a new, larger bucket array. The new size is the next prime number in a predefined sequence that is large enough to accommodate the elements while respecting the max load factor.
2.  Iterating through all elements in the old table and re-inserting them into the new, larger table based on their new hash values.

### Level 2: Advanced Iterator-Aware Design

A naive iterator implementation for the Level 1 structure would be inefficient. Incrementing an iterator past the end of a bucket's list would require a linear scan to find the next non-empty bucket, violating the constant-time requirement.

To solve this, the Level 2 architecture is redesigned:
1.  **A Single Linked List for All Elements:** All elements of the `HashSet` are stored in a single `std::list<int>`. This allows for trivial, constant-time traversal from one element to the next.
2.  **A Bucket Array of Iterators:** The bucket array is changed from `std::list<int>*` to `std::vector<std::list<int>::iterator>`. Each bucket stores an iterator pointing to the *first* element in the main `std::list` that hashes to that bucket's index.


*Illustration: All elements reside in a single linked list. The bucket array stores iterators pointing to the start of each collision chain within that list.*

This design provides the best of both worlds:
*   **Fast Lookups (`find`, `contains`):** Hash the key to get a bucket index, retrieve the iterator from the bucket array, and then traverse the short, contiguous section of the main list corresponding to that bucket.
*   **Fast Iteration (`++`, `--`):** Simply use the underlying `std::list` iterator's increment/decrement operations, which are always O(1).

The most complex operation in this model is `rehash`, which requires carefully reorganizing the nodes within the main `std::list` to maintain the property that elements hashing to the same bucket are adjacent. This is achieved efficiently using `std::list::splice`.

## API Reference

### Constructors, Destructor & Assignment
*   `HashSet(const HashSet& other)`: Copy constructor.
*   `~HashSet()`: Destructor.
*   `HashSet& operator=(HashSet other)`: Copy-and-swap assignment operator.

### Core Operations
*   `void insert(int key)`: Inserts an element. Average O(1).
*   `bool contains(int key) const`: Checks if an element exists. Average O(1).
*   `void erase(int key)`: Erases an element by its value. Average O(1).

### Iterators (Level 2)
*   `Iterator begin()`: Returns an iterator to the first element. O(1).
*   `Iterator end()`: Returns a sentinel iterator to the position after the last element. O(1).
*   `Iterator find(int key)`: Returns an iterator to the element if found, otherwise `end()`. Average O(1).
*   `Iterator erase(Iterator it)`: Erases the element at the iterator's position and returns an iterator to the next element. O(1).

### Capacity & Utility
*   `bool empty() const`: Checks if the set is empty. O(1).
*   `std::size_t size() const`: Returns the number of elements. O(1).
*   `std::size_t bucketCount()`: Returns the number of buckets. O(1).
*   `float loadFactor() const`: Returns the current load factor. O(1).
*   `void maxLoadFactor(float maxLoad)`: Sets the maximum load factor. May trigger a rehash.
*   `float maxLoadFactor()`: Returns the current maximum load factor.
*   `void rehash(std::size_t newSize)`: Resizes the bucket array to have at least `newSize` buckets.
*   `std::size_t bucket(int key) const`: Returns the bucket index for a given key. O(1).
*   `std::size_t bucketSize(std::size_t n) const`: Returns the number of elements in bucket `n`.

## How to Build and Run

### Prerequisites
*   A C++ compiler supporting C++17 (e.g., `g++` or `clang++`).
*   Google Test framework (for running the provided tests in `main.cpp`).

### Building and Running Tests
You can compile and run the project from the command line. The tests provided in `main.cpp` will execute and verify the functionality of both Level 1 and Level 2 implementations.

```bash
# Compile the program
# Assumes your implementation is in HashSet.h and HashSet.cpp
# You may need to link the gtest library (-lgtest -lgtest_main -pthread)
g++ -std=c++17 -Wall -Wextra -o main main.cpp

# Run the tests
./main
```
