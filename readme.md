# Redis
## Keys:
- "object-type:id" is a good idea, as in "user:1000". Dots or dashes are often used for multi-word fields, as in "comment:4321:reply.to" or "comment:4321:reply-to".

<br>

## Strings:

**cmd:** SET, GET
```bash
> set mykey somevalue
OK
> get mykey
"somevalue"
```

**cmd:** INCR, INCRBY
```bash
> set counter 100
OK
> incr counter
(integer) 101
> incr counter
(integer) 102
> incrby counter 50
(integer) 152
```

**cmd:** DECR, DECRBY
```bash
> set counter 100
OK
> decr counter
(integer) 99
> decr counter
(integer) 98
> decrby counter 50
(integer) 48
```

**cmd:** MSET, MGET
- We can use MSET to store multiple key value pair in a single command
- When MGET is used, Redis returns an array of values.
```bash
> mset a 10 b 20 c 30
OK
> mget a b c
1) "10"
2) "20"
3) "30"
```

<br>

## Altering and querying the key space:

**cmd:** EXISTS, DEL, TYPE
```bash
> set mykey x
OK
> exists mykey
(integer) 1
> type mykey
string
> del mykey
(integer) 1
> exists mykey
(integer) 0
> type mykey
none
```

<br>

## Key expiration:
**cmd:** EXPIRE, TTL (time to live)
- They can be set both using seconds or milliseconds precision.
- However the expire time resolution is always 1 millisecond.
```bash
> set key some-value
OK
> expire key 5
(integer) 1
> get key (immediately)
"some-value"
> get key (after some time)
(nil)
```
We can set the expiration time in another way and check how much time is available before expiration.
```bash
> set count 100 ex 10
OK
> ttl key
(integer) 9
```

## Lists:
**cmd:** LPUSH, RPUSH, LRANGE
- Redis List uses Linked Lists concept to add new element in a list. 
- The **LPUSH** command adds a new element into a list, on the left (at the head), while the **RPUSH** command adds a new element into a list, on the right (at the tail). Finally the **LRANGE** command extracts ranges of elements from lists:
```bash
> rpush mylist A
(integer) 1
> rpush mylist B
(integer) 2
> lpush mylist first
(integer) 3
> lrange mylist 0 -1
1) "first"
2) "A"
3) "B"
```
***Note that*** LRANGE takes two indexes, the first and the last element of the range to return. -1 means the last element of the list.
```bash
LRANGE key start stop
```
- Both commands are variadic commands, meaning that you are free to push multiple elements into a list in a single call:
```bash
> rpush mylist 1 2 3 4 5 "foo bar"
(integer) 9
> lrange mylist 0 -1
1) "first"
2) "A"
3) "B"
4) "1"
5) "2"
6) "3"
7) "4"
8) "5"
9) "foo bar"
```

**cmd:** RPOP, LPOP
```bash
> rpush mylist a b c
(integer) 3
> rpop mylist
"c"
> rpop mylist
"b"
> rpop mylist
"a"
> rpop mylist
(nil)
```

## Capped lists
**cmd:** LTRIM
- The LTRIM command is similar to LRANGE, but instead of displaying the specified range of elements it sets this range as the new list value. All the elements outside the given range are removed.
```bash
LTRIM key start stop

> rpush mylist 1 2 3 4 5
(integer) 5
> ltrim mylist 0 2
OK
> lrange mylist 0 -1
1) "1"
2) "2"
3) "3"
```

## Blocking operations on lists
**cmd:** BRPOP, BLPOP
-  **BRPOP** and **BLPOP** which are versions of RPOP and LPOP able to block if the list is empty: they'll return to the caller only when a new element is added to the list, or when a user-specified timeout is reached.
```bash
BRPOP key [key ...] timeout

> rpush mylist 1 2 3
(integer) 3
> brpop mylist 5
1) "mylist"
2) "3"
> brpop mylist 5
1) "mylist"
2) "2"
> brpop mylist 5
1) "mylist"
2) "1"
> brpop mylist 5
(nil)
(5.01s)
```
It means: "wait for elements in the list tasks, but return if after 5 seconds no element is available".
