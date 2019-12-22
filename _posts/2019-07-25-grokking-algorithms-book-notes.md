---
layout: post
title: Notes from The Grokking Algorithms
tags: [Knapsack, DynamicProgramming ]
---

Maths is fun. Lot of concepts that we learn during academia gets rusty over time. I thought of giving it a revival. Recently I read the book [“Grokking Algorithms”](https://www.manning.com/books/grokking-algorithms). I tried to capture the notes in the following post.

## Logarithm

> How many of one number do we multiply to get another number?

* How many two's do we multiply to get 8? log<sub>2</sub>(8) = 3
* Common Logarithm (Base 10) log<sub>10</sub>(n) - It is how many times we need to use 10 in a multiplication, to get our desired number.
* Logs are flips of exponentials


## Big O notation

* Big O notation tells you how fast an algorithm is.
* It doesn't tell the algorithm speed in seconds or milliseconds.
* It rather tells you how the algorithm grows in number of operations.
* It focuses on worst case scenario.
* O(log n) is faster than O(n), but it gets a lot faster once the list of items you’re searching through grows.

## Time Complexities

| Big O notation| Name| Example
| --- | --- | ---
| O(1) | Constant Time | 
| O(log n)| Log time| Binary search
| O(n) | Linear time | Simple search
| O(n * log n) | Linearithmic | A fast sorting algorithm, like quicksort, merge sort 
| O(n<sup>2</sup>) | Quadratic | A slow sorting algorithm, like selection sort, bubble sort 
| O(n<sup>3</sup>) | Cubic | A slow sorting algorithm, like selection sort 
| O(n!) | Factorial | A really slow algorithm, like the traveling salesperson (coming up next!)
| O(2<sup>n</sup>) | Exponential | 

## Feynman algorithm :wink:
* Write down the problem.
* Think real hard.
* Write down the solution.

## Dynamic programming

* Dynamic programming is useful when you’re trying to optimize something given a constraint.
* Every dynamic-programming solution involves a grid.
* The values in the cells are usually what you’re trying to optimize. 
* Each cell is a subproblem, so think about how you can divide your problem into subproblems.
