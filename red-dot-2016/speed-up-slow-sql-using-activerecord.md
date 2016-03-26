## Title
_(Title of the talk)_
_(Please limit your title to 60 characters or less.)_

Speed up slow Sql, using ActiveRecord

## Abstract
_(What is your talk about)_
_(Provide a concise description for the program limited to 600 characters or less.)_

Conventional wisdom is to use raw sql when you need improved performance on queries. This is about one of those times when ActiveRecord worked better.

ActiveRecord provides features for eager loading and batched querying, and their careful use can lead to surprising improvements in speed. There are some intresting gotchas involved with them, and those too are worth knowing.You'll also get to know about some surprising bits on indexing.


Details
(Explain the theme and flow of your talk. What are the intended audience takeaways?)
(Include any pertinent details such as outlines, outcomes or intended audience.)

This is the story of the time I am supposed to fix the performance of a report that is talking almost a 45 minutes to complete. 

#### The situation

The report is supposed to cover user data spread over more than a dozen tables, apply some filters and produce a Csv. Many of these tables are normalized over several others. The main query for the job is a massive piece of interpolated SQL, that contains a multiple `LEFT JOINs`, and a few `INNER JOINS`. It selects only the necessary columns, the right indexes are put in place, and we even use a `GROUP_CONCAT` to do concatenation in SQL.

The Sql is terrible to read and understand, but that's a well thought trade-off. The query is written to execute fast, and every other virtue is compromised. And yet it too long to execute - even under moderate loads of a few thousand user records.

#### Starting with the basics

The only best practices I knew on improving query performance are batching and eager loading. We try to load all the data needed for a user in a single go, _and_ we do that for a small batch of users at a time. To be honest, at this point I was not confident this would be sufficient.

One way to proceed would be to add `LIMIT` and batch clauses at the end of the Sql. And then maybe place the entire query within a loop and join the results for the output (its a csv here). That may have worked.

Another approach is to rewrite the whole querying using the ActiveRecord Api. This may or may not speed thing up, but at least the code will start looking prettier. And I'll know the limit of what can be achieved with that approach. 

And like every flippant Ruby programmer out there, I start with the second approach.

#### Surprises with eager loading

The biggest surprise is how easy is it lose the gains of eager loading. This code for example, within Comment model:

    belongs_to :post
    def post_summary
      post.title[0..20]
    end

    posts = Post.includes(:comments)
    posts.each do |post|
      post.comment.post_summary  ## <= N+1 here
    end

Such queries add-up quite badly sometimes. Even when it is not N+1, the slow down is quite significant. This means our eager loading should be done rather meticulously. And sometimes it can get ridiculous too -

    Post.includes(:comments => [:post])

And then there is the decision on optimal batch size. A low batch size means we fire too many queries. Higher means our `LEFT JOIN` queries get heavy. There is a sweet spot in between somewhere, that we must figure out.

#### Unusable indexes

Indexes are also not surprise free. We all know missing indexes can degrade performance. But throwing indexes on a slow query does not always help. The `join` clause doesn't use indexes the same way the `where` clauses does. The rules are slightly confusing, and vary database to database. For Mysql (which was used in my project), there is no way to force an index for the inner loop of a `join` operation.

#### Bits on measurement

Optimizing database queries fixes a majority of our problem. But there may still be room for improvement. There may be an accidentally quadratic code somewhere hiding from direct view. Or just some search loop that could be improved with a _memoized_ hash. To see these and more, we need to be able to measure the suspected chunks of our code.

The easiest way of doing this is to use `Time.now` calls, which is what I did. It gives good numbers for comparison, but we need to litter our code with several of the those calls. But with the availability of the trace-point APi, we can do this much better. Instead of making changes to the code directly, we can get notified when a method gets invoked, and do the measurements there. Somewhat like this:

    h = { total: 0 }
    trace = TracePoint.new(:call, :return) do |trace|
      return if method != 'my_slow_method'
      h[:start] = Time.now if trace.event == :call
      h[:total] += (Time.now - h[:start]) if trace.event == :return
    end
    trace.enable
    my_slow_method
    h[:total]


#### Conclusion

Careful use of ActiveRecord features for eager loading and batched queries can help solve many of our performance problems. The total report generation time came down from a 45 mins to under a minute. And the code looks much cleaner now.

#### Audience takeaways

The primary audience takeaways here are gotchas and surprises as described earlier. The audience will also start to trust ActiveRecord more, and would try using it in similar situations instead of jumping into raw SQL.




Pitch

(Why is this talk pertinent? What is your involvement in the topic?)
(Explain why this talk should be considered and what makes you qualified to speak on the topic.)

This talk takes the audience through a performance optimization tour. The example here is a slow running reporting module that we try to optimize using ActiveRecord features. We talk about quite a few gotcha while dealing with eager loading, and a few ways of measuring performance.

Most of the talk is based on lessons from personal experience.



Bio
(Bio for the event program)
(Bio is limited to 500 characters)