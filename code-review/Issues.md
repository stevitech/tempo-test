# SimpleCache: Code Review Issues

## 1. Memory leak: expired entries are never removed

**Problem:** `get()` returns `null` for an expired entry, but leaves it in the map. Nothing else removes entries.

**Impact on production:** Every key written stays in memory for the lifetime of the process. We can expect GC pauses to get longer, and finally an `OutOfMemoryError` can occur, which takes the service down. This builds up slowly, so it only fails after days in production.

## 2. No size limit

**Problem:** There is no maximum number of entries.

**Impact on production:** If a lot of new keys arrive within one minute (for example, an attacker sending requests with random, non-repeating IDs), memory fills up and the service can go down.

## 3. Cache stampede

**Problem:**

- When a key expires, the first request goes to the database.
- If the database takes 200 ms to answer, the cache still has no value for the key during those 200 ms.
- Requests keep arriving during those 200 ms, and each one sends its own request to the database.

**Fix:** The cache should have an "already loading, please wait" flag, so only one request loads the key and the others wait for its result.

## 4. `size()` is misleading

**Problem:** `size()` returns `cache.size`, which includes expired entries that `get()` treats as missing.

**Impact on production:**

- It gives people the wrong number.
- People can make wrong decisions about memory or server count.

## 5. Nullable types make `get()` ambiguous

**Problem:** `null` has several meanings:

- never stored
- expired
- the real answer is "nothing"

**Impact on production:**

- Every lookup whose real answer is "nothing" misses the cache every time (for example, a user without a profile picture).
- These lookups always go to the database, even though the answer never changes.
