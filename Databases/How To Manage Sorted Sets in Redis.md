# How To Manage Sorted Sets in Redis

```Databases``` ```Ubuntu 22.04``` ```Redis``` ```NoSQL```

## Introduction


Redis is an open-source, in-memory key-value data store. In Redis, sorted sets are a data type similar to sets in that both are non-repeating groups of strings. The difference is that each member of a sorted set is associated with a score, allowing them to be sorted from the smallest score to the largest. As with sets, every member of a sorted set must be unique, though multiple members can share the same score.


This tutorial explains how to create sorted sets, retrieve and remove their members, and create new sorted sets from existing ones.


##$ How To Use This Guide


This guide is written as a cheat sheet with self-contained examples. We encourage you to jump to any section that is relevant to the task you’re trying to complete.


The commands shown in this guide were tested on an Ubuntu 22.04 server running Redis version 6.0.16. To set up a similar environment, you can follow Step 1 of our guide on How To Install and Secure Redis on Ubuntu 22.04. We will demonstrate how these commands behave by running them with redis-cli, the Redis command line interface. If you’re using a different Redis interface — Redli, for example — the exact output of certain commands may differ.


Alternatively, you could provision a managed Redis database instance to test these commands, but depending on the level of control allowed by your database provider, some commands in this guide may not work as described. To provision a DigitalOcean Managed Database, follow our Managed Databases product documentation. Then, you must either install Redli or set up a TLS tunnel in order to connect to the Managed Database over TLS.


# Creating Sorted Sets and Adding Members


To create a sorted set, use the zadd command. zadd accepts as arguments the name of the key that will hold the sorted set, followed by the score of the member you’re adding and the value of the member itself. The following command will create a sorted set key named faveGuitarists with one member, "Joe Pass", that has a score of 1:


```
zadd faveGuitarists 1 "Joe Pass"


```


zadd will return an integer that indicates how many members were added to the sorted set if it was created successfully:


```
Output(integer) 1

```


You can add more than one member to a sorted set with zadd. Note that their scores don’t need to be sequential, there can be gaps between scores, and multiple members held in the same sorted set can share the same score:


```
zadd faveGuitarists 4 "Stephen Malkmus" 2 "Rosetta Tharpe" 3 "Bola Sete" 3 "Doug Martsch" 8 "Elizabeth Cotten" 12 "Nancy Wilson" 4 "Memphis Minnie" 12 "Michael Houser"


```


```
Output(integer) 8

```


zadd can accept the following options, which you must enter after the key name and before the first member score:


- NX or XX: These options have opposite effects, so you can only include one of them in any zadd operation:

NX: Tells zadd not to update existing members. With this option, zadd will only add new elements.
XX: Tells zadd to only update existing elements. With this option, zadd will never add new members.


- NX: Tells zadd not to update existing members. With this option, zadd will only add new elements.
- XX: Tells zadd to only update existing elements. With this option, zadd will never add new members.
- CH: Normally, zadd only returns the number of new elements added to the sorted set. With this option included, though, zadd will return the number of changed elements. This includes newly added members and members whose scores were changed.
- INCR: This causes the command to increment the member’s score value. If the member doesn’t yet exist, the command will add it to the sorted set with the increment as its score, as if its original score was 0. With INCR included, the zadd will return the member’s new score if it’s successful. Note that you can only include one score/member pair at a time when using this option.

Instead of passing the INCR option to zadd, you can instead use the zincrby command which behaves the exact same way. Instead of giving the sorted set member the value indicated by the score value like zadd, it increments that member’s score up by that value. For example, the following command increments the score of the member "Stephen Malkmus", which was originally 4, up by 5 to 9:


```
zincrby faveGuitarists 5 "Stephen Malkmus"


```


```
Output"9"

```


As is the case with the zadd command’s INCR option, if the specified member doesn’t exist then zincrby will create it with the increment value as its score.


# Retrieving Members from Sorted Sets


The most fundamental way to retrieve the members held within a sorted set is to use the zrange command. This command accepts as arguments the name of the key whose members you want to retrieve and a range of members held within it. The range is defined by two numbers that represent zero-based indexes, meaning that 0 represents the first member in the sorted set (or, the member with the lowest score), 1 represents the next, and so on.


The following example will return the first four members from the faveGuitarists sorted set created in the previous section:


```
zrange faveGuitarists 0 3


```


```
Output1) "Joe Pass"
2) "Rosetta Tharpe"
3) "Bola Sete"
4) "Doug Martsch"

```


Note that if the sorted set you pass to zrange has two or more elements that share the same score, it will sort those elements in lexicographical, or alphabetical, order.


The start and stop indexes can also be negative numbers, with -1 representing the last member, -2 representing the second to last, and so on:


```
zrange faveGuitarists -5 -2


```


```
Output1) "Memphis Minnie"
2) "Elizabeth Cotten"
3) "Stephen Malkmus"
4) "Michael Houser"

```


zrange can accept the WITHSCORES argument which, when included, will also return the members’ scores:


```
zrange faveGuitarists 5 6 WITHSCORES


```


```
Output1) "Elizabeth Cotten"
2) "8"
3) "Stephen Malkmus"
4) "9"

```


zrange can only return a range of members in ascending numerical order. To reverse this and return a range in descending order, you must use the zrevrange command. Think of this command as temporarily reversing the order of the given sorted set before returning the members that fall within the specified range. So with zrevrange, 0 will represent the last member held in the key, 1 will represent the second to last, and so on:


```
zrevrange faveGuitarists 0 5


```


```
Output1) "Nancy Wilson"
2) "Michael Houser"
3) "Stephen Malkmus"
4) "Elizabeth Cotten"
5) "Memphis Minnie"
6) "Doug Martsch"

```


zrevrange can also accept the WITHSCORES option.


You can return a range of members based on their scores with the zrangebyscore command. In the following example, the command will return any member held in the faveGuitarists key with a score of 2, 3, or 4:


```
zrangebyscore faveGuitarists 2 4


```


```
Output1) "Rosetta Tharpe"
2) "Bola Sete"
3) "Doug Martsch"
4) "Memphis Minnie"

```


The range is inclusive in this example, meaning that it will return members with scores of 2 or 4. You can exclude either end of the range by preceding it with an open parenthesis ((). The following example will return every member with a score greater than or equal to 2, but less than 4:


```
zrangebyscore faveGuitarists 2 (4


```


```
Output1) "Rosetta Tharpe"
2) "Bola Sete"
3) "Doug Martsch"

```


As with zrange, zrangebyscore can accept the WITHSCORES argument. It also accepts the LIMIT option, which you can use to retrieve only a selection of elements from the zrangebyscore output. This option accepts an offset, which marks the first member in the range that the command will return, and a count, which defines how many members the command will return in total. For example, the following command will review the first six members of the faveGuitarists sorted set but will only return three members from it, starting from the second member in the range, represented by 1:


```
zrangebyscore faveGuitarists 0 5 LIMIT 1 3


```


```
Output1) "Rosetta Tharpe"
2) "Bola Sete"
3) "Doug Martsch"

```


The zrevrangebyscore command returns a reversed range of members based on their scores. The following command returns every member of the set with a score between 10 and 6:


```
zrevrangebyscore faveGuitarists 10 6


```


```
Output1) "Stephen Malkmus"
2) "Elizabeth Cotten"

```


As with zrangebyscore, zrevrangebyscore can accept both the WITHSCORES and LIMIT options. Additionally, you can exclude either end of the range by preceding it with an open parenthesis.


There may be times when all the members in a sorted set have the same score. In such a case, you can force Redis to return a range of elements sorted lexicographically, or in alphabetical order, with the zrangebylex command. To try out this command, run the following zadd command to create a sorted set where each member has the same score:


```
zadd SomervilleSquares 0 Davis 0 Inman 0 Union 0 porter 0 magoun 0 ball 0 assembly


```


zrangebylex must be followed by the name of a key, a start interval, and a stop interval. The start and stop intervals must begin with an open parenthesis (() or an open bracket ([), like the following:


```
zrangebylex SomervilleSquares [a [z


```


```
Output1) "assembly"
2) "ball"
3) "magoun"
4) "porter"

```


Notice that this example returned only four of the eight members in the set, even though the command sought a range from a to z. This is because Redis values are case-sensitive, so the members that begin with uppercase letters were excluded from its output. To return those, you could run the following:


```
zrangebylex SomervilleSquares [A [z


```


```
Output1) "Davis"
2) "Inman"
3) "Union"
4) "assembly"
5) "ball"
6) "magoun"
7) "porter"

```


zrangebylex also accepts the special characters -, which represents negative infinity, and +, which represents positive infinity. Thus, the following command syntax will also return every member of the sorted set:


```
zrangebylex SomervilleSquares - +


```


Note that zrangebylex cannot return sorted set members in reverse lexicographical (ascending alphabetical) order. To do that, use zrevrangebylex:


```
zrevrangebylex SomervilleSquares + -


```


```
Output1) "porter"
2) "magoun"
3) "ball"
4) "assembly"
5) "Union"
6) "Inman"
7) "Davis"

```


Because it’s intended for use with sorted sets where every member has the same score, zrangebylex does not accept the WITHSCORES option. It does, however, accept the LIMIT option.


# Retrieving Information about Sorted Sets


To find out how many members are in a given sorted set (in other words, to determine its cardinality), use the zcard command. The following example shows how many members are held in the faveGuitarists key from the first section of this guide:


```
zcard faveGuitarists


```


```
Output(integer) 9

```


zcount can tell you how many elements are held within a given sorted set that fall within a range of scores. The first number following the key is the start of the range and the second one is the end of the range:


```
zcount faveGuitarists 3 8


```


```
Output(integer) 4

```


zscore outputs the score of a specified member of a sorted set:


```
zscore faveGuitarists "Bola Sete"


```


```
Output"3"

```


If either the specified member or key doesn’t exist, zscore will return (nil).


zrank is similar to zscore, but instead of returning the given member’s score, it instead returns its rank. In Redis, a rank is a zero-based index of the members of a sorted set, ordered by their score. For example, "Joe Pass" has a score of 1, but because that is the lowest score of any member in the key, it has a rank of 0:


```
zrank faveGuitarists "Joe Pass"


```


```
Output(integer) 0

```


There’s another Redis command called zrevrank which performs the same function as zrank, but instead reverses the ranks of the members in the set. In the following example, the member "Joe Pass" has the lowest score, and consequently has the highest reversed rank:


```
zrevrank faveGuitarists "Joe Pass"


```


```
Output(integer) 8

```


The only relation between a member’s score and their rank is where their score stands in relation to those of other members. If there is a score gap between two sequential members, that won’t be reflected in their rank. Note that if two members have the same score, the one that comes first alphabetically will have the lower rank.


Like zscore, zrank and zrevrank will return (nil) if the key or member doesn’t exist.


zlexcount can tell you how many members are held in a sorted set between a lexicographical range. The following example uses the SomervilleSquares sorted set from the previous section:


```
zlexcount SomervilleSquares [M [t


```


```
Output(integer) 5

```


This command follows the same syntax as the zrangebylex command, so refer to the previous section for details on how to define a string range.


# Removing Members from Sorted Sets


The zrem command can remove one or more members from a sorted set:


```
zrem faveGuitarists "Doug Martsch" "Bola Sete"


```


zrem will return an integer indicating how many members it removed from the sorted set:


```
Output(integer) 2

```


There are three Redis commands that allow you to remove members of a sorted set based on a range. For example, if each member in a sorted set has the same score, you can remove members based on a lexicographical range with zremrangebylex. This command uses the same syntax as zrangebylex. The following example will remove every member that begins with a capital letter from the SomervilleSquares key created in the previous section:


```
zremrangebylex SomervilleSquares [A [Z


```


zremrangebylex will output an integer indicating how many members it removed:


```
Output(integer) 3

```


You can also remove members based on a range of scores with the zremrangebyscore command, which uses the same syntax as the zrangebyscore command. The following example will remove every member held in faveGuitarists with a score of 4, 5, or 6:


```
zremrangebyscore faveGuitarists 4 6


```


```
Output(integer) 1

```


You can remove members from a set based on a range of ranks with the zremrangebyrank command, which uses the same syntax as zrangebyrank. The following command will remove the three members of the sorted set with the lowest rankings, which are defined by a range of zero-based indexes:


```
zremrangebyrank faveGuitarists 0 2


```


```
Output(integer) 3

```


Note that numbers passed to remrangebyrank can also be negative, with -1 representing the highest rank, -2 the next highest, and so on.


# Creating New Sorted Sets from Existing Ones


Redis includes two commands that allow you to compare members of multiple sorted sets and create new ones based on those comparisons: zinterstore and zunionstore. To experiment with these commands, run the following zadd commands to create some example sorted sets:


```
zadd NewKids 1 "Jonathan" 2 "Jordan" 3 "Joey" 4 "Donnie" 5 "Danny"
zadd Nsync 1 "Justin" 2 "Chris" 3 "Joey" 4 "Lance" 5 "JC"


```


zinterstore finds the members shared by two or more sorted sets — their intersection — and produces a new sorted set containing only those members. This command must include, in order, the name of a destination key where the intersecting members will be stored as a sorted set, the number of keys being passed to zinterstore, and the names of the keys you want to analyze:


```
zinterstore BoyBands 2 NewKids Nsync


```


zinterstore will return an integer showing the number of elements stored in the destination sorted set. Because NewKids and Nsync only share one member, "Joey", the command will return 1:


```
Output(integer) 1

```


Be aware that if the destination key already exists, zinterstore will overwrite its contents.


zunionstore will create a new sorted set holding every member of the keys passed to it. This command uses the same syntax as zinterstore, and requires the name of a destination key, the number of keys being passed to the command, and the names of the keys:


```
zunionstore SuperGroup 2 NewKids Nsync


```


Like zinterstore, zunionstore will return an integer showing the number of elements stored in the destination key. Even though both of the original sorted sets held five members, because sorted sets can’t have repeating members and each key has one member named "Joey", the resulting integer will be 9:


```
Output(integer) 9

```


Like zinterstore, zunionstore will overwrite the contents of the destination key if it already exists.


To give you more control over member scores when creating new sorted sets with zinterstore and zunionstore, both of these commands accept the WEIGHTS and AGGREGATE options.


The WEIGHTS option is followed by one number for every sorted set included in the command which weight, or multiply, the scores of each member. The first number after the WEIGHTS option weights the scores of the first key passed to the command, the second number weights the second key, and so on.


The following example creates a new sorted set holding the intersecting keys from the NewKids and Nsync sorted sets. It weights the scores in the NewKids key by a factor of three, and weights those in the Nsync key by a factor of seven:


```
zinterstore BoyBandsWeighted 2 NewKids Nsync WEIGHTS 3 7


```


If the WEIGHTS option isn’t included, the weighting defaults to 1 for both zinterstore and zunionstore.


AGGREGATE accepts three sub-options. The first of these, SUM, implements zinterstore and zunionstore’s default behavior by adding the scores of matching members in the combined sets.


If you run a zinterstore or zunionstore operation on two sorted sets that share one member, but this member has a different score in each set, you can force the operation to assign the lower of the two scores in the new set with the MIN suboption:


```
zinterstore BoyBandsWeightedMin 2 NewKids Nsync WEIGHTS 3 7 AGGREGATE MIN


```


Because the two sorted sets only have one matching member with the same score (3), this command will create a new set with a member that has the lower of the two weighted scores:


```
zscore BoyBandsWeightedMin "Joey"


```


```
Output"9"

```


Likewise, AGGREGATE can force zinterstore or zunionstore to assign the higher of the two scores with the MAX option:


```
zinterstore BoyBandsWeightedMax 2 NewKids Nsync WEIGHTS 3 7 AGGREGATE MAX


```


This command creates a new set with one member, "Joey", that has the higher of the two weighted scores:


```
zscore BoyBandsWeightedMax "Joey"


```


```
Output"21"

```


It can be helpful to think of WEIGHTS as a way to temporarily manipulate members’ scores before they’re analyzed. Likewise, it’s helpful to think of the AGGREGATE option as a way to decide how to control members’ scores before they’re added to their new sets.


# Conclusion


This guide details a number of commands used to create and manage sorted sets in Redis. If there are other related commands, arguments, or procedures you’d like to learn about in this guide, please ask or make suggestions in the comments.


For more information on Redis commands, check out our tutorial series on How to Manage a Redis Database.


