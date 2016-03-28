This is the story of the time I am supposed to fix the performance of a report that was talking almost a 45 minutes to complete. 

#### The situation

The report is supposed to cover user data spread over more than a dozen tables, apply some filters and produce a Csv. Many of these tables are normalized over several others. The main query for the job is a massive piece of interpolated SQL, that contains a multiple _LEFT JOINs_, and a few _INNER JOINS_. It selects only the necessary columns, the right indexes are put in place, and we even use a `GROUP_CONCAT` to do concatenation in SQL.

The Sql is terrible to read and understand, but that's a well thought trade-off. The query is meant to execute fast, and every other virtue is compromised. And yet it takes too long to execute - even under moderate loads  (a few thousand user records).

#### Sticking with the basics

The only best practices I knew on improving query performance are batching and eager loading. We try to load all the data needed for a user in a single go, _and_ we do that for a small batch of users at a time. To be honest, at this point I was not confident this would be sufficient.

One way to proceed would be to add _LIMIT_ and batch clauses at the end of the Sql. And maybe place the entire query within a loop and then join the results for the output (its a csv here). That may have worked.

Another approach is to rewrite the whole querying using the ActiveRecord Api. This may or may not speed thing up, but at least the code will will look prettier. And I'll know the limit of what can be achieved with that approach. Thus, like every flippant Ruby programmer out there, I start with the second approach.

#### Surprises with eager loading

The biggest surprise is how easy is it lose the gains of eager loading. This code for example, within a typical Comment model:

    belongs_to :post
    def post_summary
      post.title[0..20]
    end

    posts = Post.includes(:comments)
    posts.each do |post|
      post.comment.post_summary  ## <= N+1 here
    end

Such queries add-up quite badly sometimes. Even when it is not N+1, the slow down is quite significant. This means our eager loading should be done rather meticulously. And sometimes it can get ridiculous -

    Post.includes(:comments => [:post])

And then there is the decision on optimal batch size. A low batch size means we fire too many queries. Higher means our _LEFT JOIN_ queries get heavy. There is a sweet spot in between somewhere, that we must figure out.

#### Unusable indexes

There are surprises with Indexes too. We all know missing indexes can degrade performance. But throwing indexes on a slow query does not always help. The `join` clause doesn't use indexes the same way the `where` clauses does. The rules are slightly confusing, and vary database to database. In Mysql, for example, there is no way to force an index for the inner loop of a `join` operation.

#### Bits about measurement

Optimizing database queries fixes a majority of our problems. But there may still be room for improvement. There may be an accidentally quadratic code somewhere hiding from direct view. Or just some search loop that could be improved with a _memoized_ hash. To see these and more, we need to be able to measure the suspected chunks of our code.

The easiest way of doing this is to add `Time.now` calls, which is what I did. It gives good numbers for comparison, but we need to litter our code with several of the those calls. With the availability of the trace-point APi now, we can do this much better. Using the APi, we can get notified when a method gets invoked, and do the measurements there. There are also some gems that simplify such measurements, we'll walk through them too.

#### Conclusion and Audience takeaways

Careful use of ActiveRecord features for eager loading and batched queries can help solve many of our performance problems. The total report generation time came down from a 45 mins to under a minute. And the code looks much cleaner now.

The primary audience takeaways here are gotchas and surprises as described earlier. The audience will also trust ActiveRecord more, and will try using it in similar situations instead of jumping into raw SQL.
