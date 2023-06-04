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

<br>

## Lists:

Redis List uses Linked Lists concept to add new element in a list. <br>
**cmd:** LPUSH, RPUSH, LRANGE, LLEN

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
> llen mylist
(integer) 3
```

**_Note that_** LRANGE takes two indexes, the first and the last element of the range to return. -1 means the last element of the list.

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

<br>

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

<br>

## Blocking operations on lists

**cmd:** BRPOP, BLPOP

- **BRPOP** and **BLPOP** which are versions of RPOP and LPOP able to block if the list is empty: they'll return to the caller only when a new element is added to the list, or when a user-specified timeout is reached.

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

<br>

## Hashes

**cmd:** HSET, HGET, HGETALL, HMGET

- **HGET** returns string value and we can get only one key's value at a time
- **HGETALL** returns all the key and values in array
- **HMGET** returns an array of values

```bash
hset user:1000 username leo birthyear 1986 verified 1
(integer) 3
> hget user:1000 username
"antirez"
> hget user:1000 birthyear
"1977"
> hgetall user:1000
1) "username"
2) "antirez"
3) "birthyear"
4) "1977"
5) "verified"
6) "1"
> hmget user:1000 username birthyear no-such-field
1) "antirez"
2) "1977"
3) (nil)
```

**cmd:** HINCRBY

```bash
> hincrby user:1000 birthyear 10
(integer) 1987
> hincrby user:1000 birthyear 10
(integer) 1997
```

<br>

## Sets

Redis Sets are unordered collections of strings
**cmd:** SADD, SMEMBERS, SISMEMBER
- **SMEMBERS** returns all the elements
- **SISMEMBER** checks if an element exists
```bash
> sadd myset 1 2 3
(integer) 3
> smembers myset
1. 1
2. 2
3. 3

> sismember myset 3
(integer) 1
> sismember myset 30
(integer) 0
```
```bash
> sadd news:1000:tags 1 2 5 77
(integer) 4

> smembers news:1000:tags
1. 5
2. 1
3. 77
4. 2
```
**cmd:** SINTER
- **SINTER** performs the intersection between different sets
```bash
> sinter tag:1:news tag:2:news tag:10:news tag:27:news
... results here ...
```
**cmd:** SPOP, SUNIONSTORE, SCARD, SRANDMEMBER
- **SPOP** extracts a random element from a set
- **SUNIONSTORE** performs the union between multiple sets, and stores the result into another set
- **SCARD** provides the number of elements inside a set
- **SRANDMEMBER** When you need to just get random elements without removing them from the set, there is the SRANDMEMBER command suitable for the task. It also features the ability to return both repeating and non-repeating elements.
```bash
> sadd deck C1 C2 C3 C4 C5 C6 C7 C8 C9 C10 CJ CQ CK
  D1 D2 D3 D4 D5 D6 D7 D8 D9 D10 DJ DQ DK H1 H2 H3
  H4 H5 H6 H7 H8 H9 H10 HJ HQ HK S1 S2 S3 S4 S5 S6
  S7 S8 S9 S10 SJ SQ SK
(integer) 52

> sunionstore game:1:deck deck
(integer) 52

> spop game:1:deck
"C6"
> spop game:1:deck
"CQ"
> spop game:1:deck
"D1"
> spop game:1:deck
"CJ"
> spop game:1:deck
"SJ"

> scard game:1:deck
(integer) 47
```







