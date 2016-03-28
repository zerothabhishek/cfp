(Explain the theme and flow of your talk. What are the intended audience takeaways?)
(Include any pertinent details such as outlines, outcomes or intended audience.)

The talk is structured as five parts - one for introduction and the rest about each of the gotchas.

__Introduction:__

This part introduces databases indexes to a complete beginner, and sets the context for the subsequent discussion. It talks about how indexes improve search queries, covers a few basic examples of usage, some details on how they work internally, and their overheads.

__1: Indexes are not used for low selectivity columns.__

Columns that have very few possible values, like the gender column, are less selective. Indexes on such columns may get completely ignored. The selectivity boundary varies across database engines, but the idea is the same.

__2: Indexes for the `ON` predicates columns in `JOIN` queries may not get used.__

This has a big impact on how we use joins in our code, especially when we do eager loading. The behavior varies across different implementations, but the basic ideas are similar.
We'll go in to the details of the algorithms used and the trade-offs involved. And also discuss on how to overcome the limitation and make the most of what is possible.

__3: Order by clause needs index too - perhaps a compound index.__

The order-by clause can also make use of an index. And the lack of one will make it lose the performance gained by any other indexes in the query.
Also, when the clause contains two columns, it is better to use a compound index instead of two separate indexes. The order of columns within the compound index is also important.

__4: `LIKE` with leading wildcards can't use the index.__

The position of the wildcard with the LIKE operator becomes a decided for the index usage. A LIKE clause with a leading wildcard, say `'%foo'`, will not be able to use the index at all. The characters occurring before the wildcard can be used as filter predicates on an index, and will be more efficient.

We generally try LIKE based search before implementing a full-text search. Knowing this limitation will keep it efficient within reasonable expectations.

#### Conclusion and intended outcomes

Since some of the points are surprising, the audience can't be expected to agree immediately. I'll provide some evidence for the claims using `explain` based examples.
This talk will help sow the seed of caution in the audience's minds - such that when needed in code, the ideas can be used after research and verification.


----

Will also discuss the rationale behind the behavior, and possible approaches to get around the limitation.



