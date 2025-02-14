---
title: Solve the TwoSum Problem in Swift using Dictionary
permalink: /swift-two-sum/
---

## Prerequisites
This tutorial assums some general knowledge of programming concepts and the Swift programming language. For an excellent introduction to programming in Swift, please see [100 Days of Swift](https://www.hackingwithswift.com/100) from Paul Hudson at [hackingwithswift.com](https://www.hackingwithswift.com/). And for a primer on Time and Space Complexity, see [Complexity](https://www.kodeco.com/books/data-structures-algorithms-in-swift/v3.0/chapters/2-complexity) by Kevin Lau.

## Overview

The TwoSum problem can be stated as follows: *Given an array of integers and a target sum, return the indices of the first two numbers which add up to the target.*

For example, let's say we have the following array of `Integers`:

```
let numbers = [2, 7, 11, 15]
```

And we want to know the indices of the first two numbers which add up to a target sum of `9`.

Our first step would be to identify the first two numbers in the array that add up to our target. In this case, that would be `2` and `7`. 

The next step would be to identify the index position of `2` and `7` in the array. As Swift Arrays are [zero-indexed](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/collectiontypes/#Accessing-and-Modifying-an-Array), meaning the indexes begin counting at 0 instead of 1, the index of `2` would be `0`. And the index of `7` is `1`.

For clarity, let's take a look at the index position for each number in our array:

```
Number ---> 2, 7, 11, 15
Index  ---> 0, 1,  2,  3
```

So the solution to the TwoSum problem in this case is `[0, 1]` as those are the indices of `2` and `7` in the array.

## The Iterative Approach
As we said earlier, the first step in solving the TwoSum problem is to identify the first two numbers (or operands) which add up to our sum. One way to do this could look something like:

```
let numbers = [2, 7, 11, 15]
let targetSum = 26

// start with the first number in the array (our first operand)
// and add it to the next number in the array (our second operand)
2 + 7 = 9

// if the sum is equal to the target, return the indices
// else add the first number to the next, next number in the array
    2 + 11 = 13
    2 + 15 = 17
    
// once we've tried adding the first number to all the other numbers
// move on to the second number
7 + 2 = 9
7 + 11 = 18
7 + 15 = 22

// repeat until we find the two numbers that equal the sum
...
```

In code this would look something like: 

```
func findIndicesSumming(to target: Int, from numbers: [Int]) -> [Int] {
    // Step 1
    for i in 0..<numbers.count {
        // Step 2
        for j in 1..<numbers.count {
            // Step 3
            if numbers[i] + numbers[j] == target { return [i, j] }
        }
    }
    return []
}
```  

**Step 1**: We have an outer loop that iterates over each number in the array, indexed by `i`. This acts as the pointer for the first operand. 

**Step 2**: We have an inner loop which iterates over the remaining numbers after the current number, indexed by `j`. `j` starts at `i + 1` to avoid rechecking the same pair. This represents the pointer to the second operand.  

**Step 3**: With each pair of numbers, check if the sum is equal to the target number. If so, return the pair of indices as an array. 

Now that we have a workable solution, let's analyze it.

### Time Complexity
For this solution, we must iterate over our entire array twice, once for the first pointer `i`, and again for the second pointer `j`. This results in a time complexity of `O(n^2)`, or *Quadratic time*, which is the square of the input size *n*.

Let's look at our example array again, which has an input size `n = 4`:
```
let numbers = [2, 7, 11, 15]
```

If each iteration through our function required `1` second (which it doesn't, this is just for illustrative purposes), we would require `4^2` or `16` seconds for our function to complete. We'll work on improving this in the next section.

## Dictionary to the Rescue
Rather than iterate through the array twice, keeping pointers as a reference for our operands, we could use a [Dictionary](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/collectiontypes#Dictionaries) to act as a log book, keeping track of what operands we've already tried, as we pass through the array. We could also use a little subtraction to find our second operand or compliment. Consider the following pseudocode:

```
// create a dictionary log book to keep track of numbers we've tried

// iterate through the array
// subtract the number for the current iteration from the target number to give us our second operand
// if the log book contains our second operand, return the pair of indices for it and the current number

// else add the current number to the log book
```

We'll implement this using a dictionary to keep track of the numbers and their indices to make it easier to return the index later. We'll also use a special function of Swift Arrays called `.enumerated()` to give us access to the index of each number as we iterate through the array. 

Our code will look like this:
```
func findIndicesSumming(to target: Int, from numbers: [Int]) -> [Int] {
    // Step 1
    var numbersWithIndices = [Int:Int]()
    
    // Step 2
    for (index, number) in numbers.enumerated() {
        // Step 3
        if let complimentIndex = numbersWithIndices[target - number] {
            return [complimentIndex, index]
        }
        // Step 4
        numbersWithIndices[number] = index
    }
    
    return []
}
```

**Step 1**: Create a dictionary to hold the numbers that we've seen before, with the number as the *key*, and its accompanying index as the *value*.

**Step 2**: Iterate through the array using `.enumerated()` to give us a pointer to both the number in the array and its index.

**Step 3**: Check if the dictionary holds an index for a value that is equal to the target number minus the current number, i.e. the compliment. If so, return the index pair of the compliment and the current number.

**Step 4**: Add the current number and index key/value pair to the dictionary.

Let's analyze this solution for **Time Complexity**:

As we must iterate through our array only once, we have a time complexity of O(n), or *Linear Time*. This means that the complexity is equal to the input size *n*. To make the comparison with the previous example, for our array, with input size `n = 4`, if each iteration required `1` second to execute, our function would require just `4` seconds to complete! 

## Conclusion
As we've seen, dictionaries in Swift can be useful data structures for helping us to keep track of information. With the ability to store a `key` and corresponding `value`, we can also express relationships between two pieces of data. Dictionaries are also incredibly quick for actions like lookups and insertions, having a time complexity of O(1), or *Constant Time*, meaning the action requires the same amount of time regardless of the size of the dictionary. 

But what about *Space Complexity*? In the two solutions illustrated here, the first solution has a lower Space Complexity of O(1), or *Constant Space*. This is because it doesn't create any additional data structures in order to process the result. The dictionary solution on the other hand comes with a downside. The Space Complexity is O(n), where *n* equals the size of the input array. This is because in the worst case scenario, where no two numbers add up to the target, the function still creates the dictionary, and stores all the number\index pairs contained in the array. 

Often in programming one encounters tradeoffs like this. It's up to you to decide what is most important for your use case and application. 

## Further Reading
For more on Dictionaries and other collection types, see the [official Swift language documentation](https://docs.swift.org/swift-book/documentation/the-swift-programming-language).

For more about data structures and algorithms, I recommend [Data Structures and Algorithms in Swift](https://www.kodeco.com/books/data-structures-algorithms-in-swift/v3.0/) by the Ray Wenderlich team.

