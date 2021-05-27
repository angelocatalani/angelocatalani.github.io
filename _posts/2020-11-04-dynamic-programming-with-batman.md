---
layout: post
title: Dynamic Programming with Batman
subtitle: An interesting problem about increasing and decreasing subsequences
cover-img: /assets/img/thor.jpg
thumbnail-img: /assets/img/batman.jpg
share-img: /assets/img/batman.jpg
tags: [algorithm,dynamic-programming,batman]
comments: true
---

I could not celebrate my first post without talking about the most fascinating  algorithmic paradigm: *Dynamic Programming* - the sledgehammers of problem solving.

To me, it is an incredible point of contact between the abstraction of Maths and the pragmatism of Programming: a love story about Maths, Recursion and Optimization. It is almost impossible to imagine something more powerful.

Here, I would like to discuss  [the *Batman* problem](https://www.spoj.com/problems/BAT2/) proposed by a [great course I have recently joined](https://www.commonlounge.com/discussion/cbb1cf5102e24774a2bd43daa5211230).

{: .box-note}
Given a list of unique integers, find the increasing (`IS`) and decreasing (`DS`) subsequencses such that the overall number of distinct elements visited is maximum.  For example given `[5, 3, 4, 6, 1, 2]` the output is `5`  with `IS = [3, 6]` and `DS = [5, 4, 2]`.

My first approach was to apply the same idea beneath the classical `Longest Increasing Subsequence` problem: unfortunately, this is not an effective strategy.

 In fact  we need somehow to take into consideration the possibility not to add an element because it was already added to the decreasing subsequence or viceversa.

In a few words my strategy to model the state as  `opt(i,j) := the optimal solution when you add the i-th items to IS and the j-th item to DS` does not go anywhere **but it would be great if you could  prove me wrong!**.

After spending more than 2 hours, I decided to opt for a simple reasoning and try to implement it. Trivially for each item in the list you need to decide exclusively  to:

- Add it (if possible) to the `IS`
- Add it (if possible) to the `DS`
- Discard it

I implemented it through  a recursive functions with exponential complexity.

```C++
int solve_batman(vector<int> values, int current_position, vector<int> increasing_seq,vector<int> decreasing_seq);
```

Then, I noticed  I did not need the actual increasing or decreasing sequences but just the index of the latest element taken into consideration. This is what I came up with:

```c++
int solve_batman(vector<int> values, int current_position, int last_increasing_idx, int last_decreasing_idx)
{
    if (current_position >= values.size())
        return 0;

    int opt_1 = 0, opt_2 = 0, opt_3 = 0;
    if (last_increasing_idx == -1 || values[last_increasing_idx] < values[current_position])
    {	
      	// add the current item to the increasing sequence
        opt_1 = 1 + solve_batman(tags, current_position + 1, current_position, last_decreasing_idx);
    }
    if (last_decreasing_idx == -1 || values[last_decreasing_idx] > values[current_position])
    {
      	// add the current item to the decreasing sequence
        opt_2 = 1 + solve_batman(tags, current_position + 1, last_increasing_idx, current_position);
    }
  
  	// discard the current item
    opt_3 = solve_batman(tags, current_position + 1, last_increasing_idx, last_decreasing_idx);
  	
    // the optimal solution is the best one among the three options  
    return max({opt_1, opt_2, opt_3});
}
```



At this point it is straightforward to use memoization to cache the result of each recursive call and hope the complexity is reduced. 

More rigorously, it is clear how the three functions parameters (all except the input list `values`)  can only assume a value in the range: `[0,values.size()]` . This means among all the recursive calls that are: `~ 3^values.size()` only `values.size()^3` produce a new result.

The next step was to create a tridimensional cache initialiazed with `-1` values to store intermidiate values.

```c++
int cache[101][101][101];

// std::fill_n(&cache[0][0][0], 101 * 101 * 101, -1);

int solve_batman(vector<int> tags, int current_position, int last_increasing_idx, int last_decreasing_idx)
{
    if (current_position >= tags.size())
        return 0;
  
    if (cache[current_position + 1][last_increasing_idx + 1][last_decreasing_idx + 1] != -1)
        return cache[current_position + 1][last_increasing_idx + 1][last_decreasing_idx + 1];

    int opt_1 = 0, opt_2 = 0, opt_3 = 0;
  
    if (last_increasing_idx == -1 || tags[last_increasing_idx] < tags[current_position])
    {
        opt_1 = 1 + solve_batman(tags, current_position + 1, current_position, last_decreasing_idx);
    }
  
    if (last_decreasing_idx == -1 || tags[last_decreasing_idx] > tags[current_position])
    {
        opt_2 = 1 + solve_batman(tags, current_position + 1, last_increasing_idx, current_position);
    }
    opt_3 = solve_batman(tags, current_position + 1, last_increasing_idx, last_decreasing_idx);
    
    cache[current_position + 1][last_increasing_idx + 1][last_decreasing_idx + 1] = max({opt_1, opt_2, opt_3});
    return max({opt_1, opt_2, opt_3});
}
```

In this way the complexity is reduced to `O(N^3)`.

Solved this problem I asked myself to replicate the same approach to solve the classical  `Longest Increasing Subsequence` (`LIS`).

Generally, the state for the `LIS` is defined as: `opt(i) := the optimal solution with the i-th element in the sequence = 1 + max( opt(j) over j < i)` and `opt(0) = 1`.

Following, the same approach of  `Batman`, `LIS` could be also solved in this way:

```c++
int cache[101][101];

// fill_n(&cache[0][0], 101 * 101, -1);

int lis_as_batman(vector<int> values, int current_position, int last_increasing_idx)
{
    if (current_position >= values.size())
        return 0;
    if (cache[current_position + 1][last_increasing_idx + 1] != -1)
        return cache[current_position + 1][last_increasing_idx + 1];

    int opt_1 = 0;
    if (last_increasing_idx == -1 || values[current_position] > values[last_increasing_idx])
    {
        opt_1 = 1 + lis_as_batman(values, current_position + 1, current_position);
    }

    int opt_2 = lis_as_batman(values, current_position + 1, last_increasing_idx);
  
    cache[current_position + 1][last_increasing_idx + 1] = max(opt_1, opt_2);
    return cache[current_position + 1][last_increasing_idx + 1];
}
```



This is my takeaway:

- approach the problem in the easiest possible way
- when dealing with recursion, memoization is the best friend you can have
- a recursive approach to list all the possible solutions is not necessarily exponential if the parameters have bounded values
- do not  force yourself to excessively think by similarities: try new routes

Have a nice day 🚀 

![Tree](/assets/img/tree.jpg)

