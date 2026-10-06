1. memory leak : expired entries are never removed.

problem : get() function returns null for an expired entry, but leaves it. Nothing else removes entries.

impact on production : every key written stays in memory during the life time of the process. we can expect GC pauses that get longer, and finally "OufOfMemoryError" can be occured (leads service down). this builds up slowly but fails after days in production.

2. no size limit

problem : there is no max number of entries.

impact on production : if a lot of new keys arrive within one min (attacker who sends requests with random, none-repeating IDs), that can make the service down. 
critical : 

3. cache stampede

problem : 

when the key expires, the first request goes to the database.
If the db takes 200ms to answer, then during 200ms, the cache still has no value for the key.
But requests keep arriving during those 200ms and makes db many reqeusts.

Fix:
The cache should have "already loading, please wait" flag.

4. size function is misleading

problem : size function returns cache.size that includes expired entries that get function treats as missing.

impact on production :
  - gives the wrong number to people
  - people can make wrong decision on memory or server count.


5. nullable types make get function ambiguous

problem : null has many meanings.
for example
  - never stored
  - expired
  - real answer is nothing

impact on production : 
  - for example, every lookup whose real answer is "nothing" misses the cache every time (a user without a profile picture, ...)
  - lookups always go to the db, even though the answer never changes.
