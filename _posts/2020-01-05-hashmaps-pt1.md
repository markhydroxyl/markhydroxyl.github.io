---
title: Hashmaps Pt 1
layout: post
categories: hashmap
---
# Hashmaps
## Introduction to Hashmaps
A **hashmap** or **hash table** is an implementation of the *associative array abstract data type*, a data structure that stores `(key, value)` pairs. Associative arrays are also called *maps*, *symbol tables*, and *dictionaries*. Let's take a look at a simple example:

key | value
--- | ---:
`"apple"` | `$10.99`
`"orange"` | `$8.99`
`"banana"` | `$7.35`

In the *table*, the keys are stored on the left side and *mapped* or *associated* to their respective values or *definitions* on the right side. In this case, the name of a fruit is being mapped to a price; this is a logical, organized presentation of information and makes it easy to find the price of these three fruits. Now, let's consider what happens when the list grows and there are more fruit to consider.

### Unsorted
Let's start with a very simple approach: for every new fruit, add it to the bottom of the list. This approach is very easy upfront; inserting at the end of the list doesn't require any fancy work like shifting elements around or searching for anything (let's ignore the fact that we might run out of space in the list and will need to expand it every now and then). In technical terms, insertion has amoritized constant time complexity, which is very, *very* fast.

However, there is a significant drawback: in order to find an arbitrary fruit in the list, the only way will be to go through the list one element at a time. In the worst case, when the fruit isn't even in the list, we'll have gone through every single row in the table! Removing fruit is very similar, since you'll need to find the row where the fruit is (or if the fruit isn't present) before actually deleting (which itself is also painful). In technical terms, lookup and removal have linear time complexity, which isn't great. Since lookup is an important feature for tables, we want something faster. Let's see if we can do better.

### Sorting
If we sorted the table so that the fruits were in alphabetical order, lookup becomes a lot faster! There's a technique called *binary searching* that makes lookup significantly faster. The technical details of binary searching are outside the scope of this post, but the general idea is to take advantage of the fact that the table is sorted and to repeatedly halve the range of elements to consider until the `key` is found. Lookup then gets logarithmic time complexity, which is pretty good.

Here's the problem with a sorted table, however: what happens when we need to add or remove a fruit from the list? On top of finding the position for the fruit, we also need to shift the other elements of the table around. It isn't possible to just magic a row into or out of existence at a specific location, we need to shift every element into the row above or below it (depending on insertion or deletion). This means that adding and removing get linear time complexity, which means we just flipped the issue we had with the unsorted array.

There are different ways to solve the problem of shifting elements around in a sorted table, but keeping the elements sorted puts an upper limit on the potential speed. Instead, let's try something completely different.

### Hashing
Enter the **hash function**. At a high level, a hash function takes a `key` as input, and produces a **hash** as output. The hash is a number which represents a **bucket** or **index** in an array or table. For this example, let's assume we have a perfect hashing function (what this exactly means, how to construct one, and other technical details will come later). What this means is that whenever we want to look for a fruit, we calculate the hash of the fruit's name, and then simply look at the hash's bucket in the table. No need to look at irrelevant entries in the table, we can jump straight to the correct entry. The same deal applies with insertion and removal: just calculate the hash for the `key`, and you can jump straight to the row to do whatever you need to do. Great, all three functions have constant time complexity! A hash table is kind of like a magic dictionary that tells you exactly which page to look at before you even open the book.

So if hashmaps are so good, what's the drawback? The first issue is that the elements in the table usually aren't even close to being in a sorted order. This means that if you want to get any kind of sorted or aggregate data, you'll need to go through the entire table and sort the data yourself. Using the magic dictionary analogy again, if you were to try reading the dictionary, the words would appear in a seemingly random order; good luck with finding the last word (by alphabetical order) in the dictionary, who knows which page it might be on? The second issue is with the hash function: making good hash functions is very hard, not to mention perfect ones. This means that in practice, in order to keep up with the promised speed benefits, some extra memory is sacrificed. Using the magic dictionary analogy, a decent fraction of the entries are blank for no obvious reason, and the dictionary has more pages than it should theoretically need.

## More about hashing
As we just discussed, making good hash functions is hard. Let's dive into some of the more technical details of hash functions. Take a look at this table:

key | hash
--- | ---:
`"apple"` | `0x058B835A`
`"orange"` | `0xC3DE262E`
`"banana"` | `0xACC54F65`
`"cherry"` | `0xAED8F599`
`"pear"` | `0x003470E6`

The hashes here are from Java's `String.hashCode()` printed as hex values. Hashes are usually *compressed* into the size of the hash table. Then, the *compressed hash* can represent the index in an array to store the key and value. In this case, since there are 5 elements, we can take the modulus of the hash with 5, the size of our table. Then, we get

key | hash | index (mod 5)
--- | ---: | ---:
`"apple"` | `0x058B835A` | `0`
`"orange"` | `0xC3DE262E` | `0`
`"banana"` | `0xACC54F65` | `2`
`"cherry"` | `0xAED8F599` | `3`
`"pear"` | `0x003470E6` | `4`

Hmm, `"apple"` and `"orange"` both get 0 for their indices. While `"banana"`, `"cherry"`, and `"pear"` would get their own unique array slots, `"apple"` and `"orange"` both want to take the 0th bucket. This is called a **collision**, and there are a few **collision resolution** strategies to deal with this problem. Also note that none of the keys took index 1, so it is (for now) effectively a wasted slot.

### Expanding the table
Let's start with the intuitive strategy of expanding the table. After all, if we added more slots, the chances of any two random keys colliding should go down. Let's take a look:

mod | indices
--- | ---
5 | 0 0 2 3 4
6 | 2 4 1 5 4
7 | 1 0 1 1 5
8 | 2 2 3 7 6
9 | 8 1 7 2 7
10 | 0 0 7 3 4
**11** | 10 6 3 9 0
12 | 2 10 7 11 10
13 | 1 8 7 1 3
14 | 8 0 1 1 12
**15** | 5 10 7 8 4
**16** | 10 2 11 7 6
**17** | 8 10 13 12 3
**18** | 8 10 7 11 16
**19** | 4 12 7 5 16
20 | 10 10 7 3 14
21 | 8 7 1 8 19
**22** | 10 6 3 9 0
**23** | 6 18 19 3 22
**24** | 2 10 19 23 22

I bolded all of the rows with no collisions. The very first such row is 11 elements! With these five particular keys, we need to more than double the size of the table to avoid any collisions. And that's with just five keys; when the number of keys starts growing, the probability of a collision occuring starts skyrocketing. If we assume a perfectly random distribution of hashes, we can use the birthday problem (appendix?) to find the probability of at least two keys colliding. Here are some examples: