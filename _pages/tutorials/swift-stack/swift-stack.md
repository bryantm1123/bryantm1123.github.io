---
title: Create a Stack in Swift
permalink: /swift-stack/
---

## Prerequisites
This tutorial assumes some general knowledge of the Swift programming language. For an excellent introduction to programming in Swift, please see [100 Days of Swift](https://www.hackingwithswift.com/100) from Paul Hudson at [hackingwithswift.com](https://www.hackingwithswift.com/).

## What is a Stack?
A stack is a data structure that represents the concept of a physical stack. Items are placed on top of the stack, and the items placed last are the first to be taken off.

![firewood-stack](firewood-stack.jpeg)

Or in pancake terms, you eat one pancake at a time, and starting with the top one. 😋

![pancake stack](pancake-stack.jpeg)

## A Basic Stack in Swift
Now that we have an idea of what a stack is, let's take a look at a few of its basic features.
- **Push**: adds an item to the top of the stack
- **Pop**: removes an item from the top of the stack
- **Peek**: takes a look to see if there is an item on top of the stack

Let's implement a stack with these features:

```
// 1
public class Stack<Element> {
    // 2
    private var storage = [Element]()
    
    // 3
    public func push(_ element: Element) {
        storage.append(element)
    }
    
    // 4
    @discardableResult
    public func pop() -> Element? {
        storage.popLast()
    }
    
    // 5
    public func peek() -> Element? {
        storage.last
    }
    
    // 6
    public var isEmpty: Bool {
        peek() == nil
    }
}
```

**Step 1**: Create a `Stack` class that accepts a generic type called `Element`. This will allow us to store any type we want in the stack.

**Step 2**: Declare the backing storage, in this case an array. As you may have guessed by the title of the next section, a Swift Array provides features similar to the stack. We'll discuss in the upcoming section why we create the stack instead of using an array directly.

**Step 3**: Push a new element onto the stack by appending it to the end of the storage array.

**Step 4**: Remove the last element and return it by calling the `.popLast()` function of the storage array. We return an optional result here as the array may be empty, and use the `@discardableResult` annotation so the caller can ignore the returned value if it isn't needed.

**Step 5**: Return the last item in the storage array. The returned element is optional again because the array might be empty.

**Step 6**: Add an additional computed boolean variable so the caller can easily check if the stack is empty.

## Why Not Just Use an Array?
While an array is a good choice for the backing storage of our stack, it does not have the *interface* of a stack, which is well-defined in the field of computer science, with the operations of **push**, **pop**, and **peek**. 

Some languages, such as JavaScript and Python, do offer these operations on their collection types, but Swift does not. Therefore, the short answer is, to be a good citizen of the programming community, and to follow a widespread convention that other programmers will recognize, we declare the stack according to the prescribed interface. 

## Let's Start Stack-ing!
### Match the Parentheses
Stacks can be used for certain string validations. Let's say, for example, we have a string that may contain parentheses `()`. We want to ensure that if the string contains an opening parenthesis `(`, it should be followed at some point by a closing parenthesis `)`. We can use the Stack class we created earlier to acheive this.

```
func isValid(str: String) -> Bool {
    // 1
    var stack = Stack<Character>()
    
    // 2
    for char in str {
        // 3
        if char == "(" {
            stack.push(char)
        } else if char == ")" {
            // 4
            if stack.isEmpty {
                return false
            } else {
                stack.pop()
            }
        }
    }
    // 5
    return stack.isEmpty
}
```

**Step 1**: Create an instance of the Stack class with Swift's `Character` type. Hint: In Swift, a `String` type is a collection of `Character` elements.

**Step 2**: Iterate through the string, examining each character.

**Step 3**: If the character is an opening parenthesis `(`, push it onto the stack.

**Step 4**: If the character is a closing parenthesis `)`, check if the stack is empty. If so, return *false*.

If the stack is empty, this means that we have a closing parenthesis with no matching opening parenthesis. 

If the stack is *not* empty, this means we *do* have an opening parenthesis stored there, and we can pop it, as we have found its matching closing parenthesis.

**Step 5**: After we have finished iterating through each character in the string, we return the result of the `isEmpty` check on our stack. 

If the stack is empty, then all parentheses have been matched, and the string is valid.

If the stack is *not* empty, that means we have an unmatched opening parenthesis, and the string is not valid. 

## How does a Stack, Stack Up?
As we have seen, Stacks provide a specific set of functionality, which is well-defined in the programming world. They can also be used with different types of backing storage, for example an Array or a Linked List, but they must always provide the **push**, **pop**, and **peek** operations. 

They can be especially useful in scenarios where *backtracking* is important. For example, if you have a function which navigates a Binary Tree, a stack a can be used to remember the previous decision, and backtrack, or pop that decision, if a course correction is needed. 

Another popular application of a stack is found in Git's [`stash`](https://git-scm.com/docs/git-stash) feature, which allows you to save the state of modified files, without commiting them, and return to an unmodified version of your working directory. When using the command `git stash push`, you are pushing uncommited files onto the stash tool's stack. And to remove them from the stack and back into your working directory, use `git stash pop`.

## Further Reading
For more general information on Stacks, including their history and connection to Alan Turing, see the Wikipedia article [Stack (abstract data type)](https://en.wikipedia.org/wiki/Stack_(abstract_data_type)).

For more on information on Arrays and other Swift collection types, see the [official Swift language documentation](https://docs.swift.org/swift-book/documentation/the-swift-programming-language).