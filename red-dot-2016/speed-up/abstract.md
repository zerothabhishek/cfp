Conventional wisdom is to use raw Sql when you need improved performance on queries. This story is about one of those times when ActiveRecord worked better.

ActiveRecord provides features for eager loading and batched querying, and their careful use can lead to surprising improvements in speed. There are some interesting gotchas involved with them, and those too are worth knowing. You'll also get to know some interesting bits on indexing, and will find some useful tips on measurement too.
