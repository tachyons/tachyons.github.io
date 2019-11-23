---
layout: single
title: "Behind the scenes of hash table performance in ruby 2.4"
modified:
categories:
excerpt:
comments: true
tags: ['ruby', 'hash', '2.4', 'data-structure']
canonical_url: 'https://redpanthers.co/behind-scenes-hash-table-performance-ruby-2-4/'
image:
  feature:
date: 2017-01-30T09:30:13+05:30
---

Ruby 2.4 got released this Christmas with lot of exciting features. One of the most underrated features in ruby 2.4 is hash table improvements. Before going into details about implementation, let’s first check the benchmark to know how this change gonna affect your ruby application.

- Getting keys and values of a hash

```ruby
h = {}

10000.times do |i|
  h[i] = nil
end

# Get all hash values
Benchmark.measure { 50000.times { h.values } }

# Get all hash keys
Benchmark.measure { 50000.times { h.keys } }
```

Ruby 2.3
```
=> #<Benchmark::Tms:0x00000002a0f990 @label="", @real=2.8023432340005456, @cstime=0.0, @cutime=0.0, @stime=0.13000000000000012, @utime=2.6799999999999997, @total=2.8099999999999996>
#<Benchmark::Tms:0x00000002963398 @label="", @real=2.767410662000657, @cstime=0.0, @cutime=0.0, @stime=0.029999999999999805, @utime=2.729999999999997, @total=2.7599999999999967>
```

Ruby 2.4
```
#<Benchmark::Tms:0x0000000226d700 @label="", @real=0.8854832770002758, @cstime=0.0, @cutime=0.0, @stime=0.08999999999999997, @utime=0.7999999999999998, @total=0.8899999999999998>
#<Benchmark::Tms:0x000000022b1018 @label="", @real=0.8476084579997405, @cstime=0.0, @cutime=0.0, @stime=0.06999999999999995, @utime=0.7799999999999994, @total=0.8499999999999993>
```

the above two operations executed ~ 3 times faster on my laptop. Though these numbers can vary with your machine and processor, the performance improvements will be significant on all modern processors. Not all operations became 3 times faster , average perfomence improvement is more than 50%.

### Hash Table

    In computing, hash table (hash map) is a data structure that is used to implement an associative array, a structure that can map keys to values. Hash table uses a hash function to compute an index into an array of buckets or slots, from which the desired value can be found. Wikipedia

In other words, it is a data structure that allows you to store key value pair and helps to fetch specific data using the key in an efficient way. Unlike arrays, you don’t have to iterate through all elements to find a given element in the hash. If you are new to this data structure, check this [s50](https://www.youtube.com/watch?v=h2d9b_nEzoA) for the better understanding.

```ruby
hash = {key1: value1, key2: value2}
```

### Pre Ruby 2.4

Let’s first check how ruby implemented Hash in pre 2.4 old hash representation , hash table

![Pre hash]({{ site.url }}/images/prehash.png)


Ruby internally uses a structure called st_table to represent hash. st_table contains type, the number of bins, the number of entries and pointers to bin array. Bin array is an array with 11 default size and can grow when required. Let’s take an example of hash and see how it will be represented using above diagram.

```ruby
sample_hash = {a: 10, b: 20, c: 30, d: 40, e: 50}
```

Let’s take keys :c and :d

#### Step 1:

First thing ruby does is it will take the hash of the key using the internal hash function.

```
2.3.1 :075 > :c.hash
=> 2782
2.3.1 :076 > :d.hash
=> 2432
```

#### Step 2: 
After getting the hash value, result ?it  takes modulo of 11 to get figure in which bin ruby can store the given pair

```
2.3.1 :073 > :c.hash % 11
=> 10
2.3.1 :074 > :d.hash % 11
=> 1
```

This means we can put :c => 30 in 10’th bin and :d in 1st bin

#### Step 3

What if there are multiple modulo operations that give the same result? This is called hash collision. To resolve this, ruby uses a separate chaining approach  i.e, it will make a doubly linked list and add it to the existing value in the bin

#### Step 4

What if the hash is too large ?? Linked list will start growing and will make the hash slower. So, ruby will allocate more bins and do an operation called Rehashing to utilise newly added bins.

##### Improvements in 2.0

In ruby 2.0, Ruby eliminated the need for extra data structures for smaller hashes and uses linear search for better performance.

##### Improvements in 2.2

In 2.2.0, Ruby has used bin array sizes that correspond to powers of two (16, 32, 64, 128, …). 

### Changes in 2.4
new hash structure in ruby 2.4 , hash table
![Pre hash]({{ site.url }}/images/prehash.png)
Source: https://github.com/ruby/ruby/blob/trunk/st.c

In ruby 2.4, Hash table is moved to open addressing model i.e, we no longer have the separate doubly linked list for collision resolution. Here, we will be storing records in the entries array itself i.e, there is no need of pointer chasing and data will be stored in the adjacent memory location (Data locality). The hash table has two arrays called bins and entries. Entry array contains hash entries in the order of insertion and the bin array provides access to entry the by their keys. The key hash is mapped to a bin containing the index of the corresponding entry in the entry array.

### Inserting entries in Hash

#### Step 1:

Ruby will insert an entry to entries array in sequential order.

#### Step 2:

Ruby will identify the bin from which the entry is to be mapped. Ruby uses first few bits of the hash as the bin index, Explaining the whole process is beyond the scope of this article. You can check the logic in MRI source code [here](https://github.com/ruby/ruby/blob/ruby_2_4/st.c#L962)

### Accessing element by key

Let’s examine it with a sample hash **

```ruby
sample_hash = {a: 10, b: 20, c: 30, d: 40, e: 50}
```

Here, ruby will create two internal arrays, entries and bins as shown below

```ruby
entries = [[2297,a,10], [451,b,20], [2782,c,30], [2432,d,40],[1896,e,50]]
```
 

each record in entries array will contain a hash, key, and value respectively

Default bin size in ruby is 16 so Bins array for the above hash will be somewhat like this
```ruby
bins = [
3,
nil,
nil,
nil,
1,
nil,
nil,
nil,
5,
0,
nil,
nil,
nil,
nil,
2,
nil
]
```

Now what if we want to fetch an element from hash, say ‘c’
```ruby
sample_hash[:c]
```

#### Step 1:

Find the hash using ruby’s internal hash function
```	
:c.hash
2782
``` 

#### Step 2

Find the location in bin array by using find bin method

```
find_bin(2782)
```

#### Step 3

Find the location entries array using bin array
```	
bins[14] => 2
```
 

#### Step 4. Find the entry using the key we got
```	
entries[2] => [2782,c,30]
 ```
### Deleting an item
Now we have value for the key ‘c’ without iterating through all the records
Deleting an item

If the item to be deleted is first one, then ruby will change the index of ‘current first entry ‘, otherwise ruby will mark the item as reserved using a reserved hash value.

In the ruby source code, DELETED is marked using 0 and EMPTY is marked using 1.

To summarise this approach made hash implementation in ruby faster because the new bins array stores much smaller reference to the entries instead of storing entries themselves. Hence, it can take advantage of the modern processor caching levels

** Small hashes will use the linear search to find entries from ruby 2.0 onwards to avoid extra overhead and improve performance. Given example is just for reference only.

References

- [https://bugs.ruby-lang.org/issues/12142]()
- [https://blog.heroku.comruby-2-4-features-hashes-integers-rounding#hash-changes]()
- [https://en.wikipedia.org/wiki/Hash_table]()
- [http://patshaughnessy.net/ruby-under-a-microscope]()
