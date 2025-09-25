# Concepts for URL Shortening

### 1. Contexts


*The NonceGeneration concept ensures that the short strings it generates will be unique and not result in conflicts. What are the contexts for, and what will a context end up being in the URL shortening app?*
<br>
<br>
Contexts in NonceGeneration are namespaces that ensure nonces are unique within that context. In the URL shortener, the context will usually be the short URL base domain. That way, nonces generated for one domain won’t conflict with those for another.
<br>
<br>
<br>

### 2. Storing used strings


*Why must the NonceGeneration store sets of used strings? One simple way to implement the NonceGeneration is to maintain a counter for each context and increment it every time the generate action is called. In this case, how is the set of used strings in the specification related to the counter in the implementation? (In abstract data type lingo, this is asking you to describe an abstraction function.)*
<br>
<br>
NonceGeneration stores sets of used strings so it can guarantee uniqueness. If implemented with a counter, then the abstraction function maps the counter’s history to the “set of used strings” in the spec. In other words, the counter value n corresponds to {s₀, s₁, …, sₙ₋₁} (the nonces generated so far).
<br>
<br>
<br>

### 3. Words as nonces


*One option for nonce generation is to use common dictionary words (in the style of yellkey.com, for example) resulting in more easily remembered shortenings. What is one advantage and one disadvantage of this scheme, both from the perspective of the user? How would you modify the NonceGeneration concept to realize this idea?*
<br>
<br>
One advantage is that it is easier for users to remember and share verbally since it is easier for users to use ‘computer/table/smile’ as the nonces instead of ‘12Xd82Jd9h’.
<br>
One disadvantage is that collisions happen more quickly since there is a finite dictionary, and nonces may be guessable which is a serious security risk. Yellkey itself specifically calls this out “yellkeys are NOT private. anyone can access your URL if they want to. please be careful what links you choose to share through yellkey.” Having been personally curious about this, I tried out 25 basic words and one led to an actual google form that someone created so assuming that computers are faster than I am, it would be easy to find a large collection of nonces just by guessing or systematically going through words.
<br>
I would modify the NonceGeneration concept to realize this idea by changing the concept so that instead of arbitrary strings, the generated nonces come from a dictionary word pool, removing each word once used so one link doesn’t overwrite another.
<br>
<br>
<br>
<br>

# Synchronizations for URL Shortening

### 1. Partial matching


*In the first sync (called generate), the Request.shortenUrl action in the when clause includes the shortUrlBase argument but not the targetUrl argument. In the second sync (called register) both appear. Why is this?*
<br>
<br>
The first sync only needs shortUrlBase because it’s about nonce generation and the targetUrl isn’t necessary yet since it only matters later when registering. The second sync needs both since registration ties the generated nonce to a specific target.
<br>
<br>
<br>

### 2. Omitting names


*The convention that allows names to be omitted when argument or result names are the same as their variable names is convenient and allows for a more succinct specification. Why isn’t this convention used in every case?*
<br>
<br>
We omit names only when variable and parameter names are identical and if they differ then we keep both to avoid ambiguity. This prevents confusion when multiple variables are in scope.
<br>
<br>
<br>

### 3. Inclusion of request


*Why is the request action included in the first two syncs but not the third one?*
<br>
<br>
The request is included in the first two syncs because they’re triggered by user actions when they request to shorten. The third sync is setExpiry which is a system-driven event triggered by registration completion so there is no request involved.
<br>
<br>
<br>

### 4. Fixed domain


*Suppose the application did not support alternative domain names, and always used a fixed one such as “bit.ly.” How would you change the synchronizations to implement this?*
<br>
<br>
If only a single fixed domain is supported such as “bit.ly”, then I would drop the shortUrlBase argument and let the NonceGeneration context be "bit.ly" which would make syncs look like NonceGeneration.generate(context: "bit.ly").
<br>
<br>
<br>

### 5. Adding a sync


*These synchronizations are not complete; in particular, they don’t do anything when a resource expires. Write a sync for this case, using appropriate actions from the ExpiringResource and URLShortening concepts.*
<br>
<br>

When a resource expires we are automatically deleting mapping by: <br>
sync expire <br>
when ExpiringResource.expireResource(): (resource: shortUrl) <br>
then UrlShortening.delete(shortUrl) <br>
<br>
<br>
<br>
<br>

# Extending the Design

### 1. Additional Concepts 


**concept** AnalyticsLog <br>
**purpose** keep track of every time a short url is accessed <br>
**principle** each access is recorded in the log so we can later count them <br>
**state** <br>
&nbsp;&nbsp;&nbsp;&nbsp;    a set of Logs with <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        shortUrl String <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        entries List[Event]   (could just be timestamps or empty markers) <br>
**actions** <br>
&nbsp;&nbsp;&nbsp;&nbsp;    record(shortUrl: String) <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        **effects** adds one entry to the log for that shortUrl <br>
&nbsp;&nbsp;&nbsp;&nbsp;    count(shortUrl: String): (n: Number) <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        **requires** log exists <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        **effects** returns how many entries are in the log <br>
<br>
<br>

**concept** UserVisibility <br>
**purpose** control who is allowed to see analytics for a short url <br>
**principle** analytics are only viewable by the user who made the short url <br>
**state** <br>
&nbsp;&nbsp;&nbsp;&nbsp;    a set of Permissions with <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        shortUrl String <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        viewer User <br>
**actions** <br>
&nbsp;&nbsp;&nbsp;&nbsp;    grant(shortUrl: String, user: User) <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        **effects** gives user the right to view analytics for shortUrl <br>
&nbsp;&nbsp;&nbsp;&nbsp;    canView(shortUrl: String, user: User): (ok: Boolean) <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;        **effects** returns true if user has permission, false otherwise <br>
<br>
<br>
<br>

### 2. Three Essential Synchronizations


**sync** createLog <br>
&nbsp; **when** <br>
&nbsp;&nbsp;&nbsp;   Request.shortenUrl(user: User, targetUrl, base) <br>
&nbsp;&nbsp;&nbsp;   UrlShortening.register(): (shortUrl) <br>
&nbsp; **then** <br>
&nbsp;&nbsp;&nbsp;   AnalyticsLog.record(shortUrl) (start empty log) <br>
&nbsp;&nbsp;&nbsp;   UserVisibility.grant(shortUrl, user) <br>
<br>
<br>
<br>
**sync** addAccess <br>
&nbsp; **when** UrlShortening.lookup(shortUrl) <br>
&nbsp; **then** AnalyticsLog.record(shortUrl) <br>
**sync** viewLog <br>
&nbsp; **when** <br>
 &nbsp;&nbsp;  Request.viewAnalytics(user: User, shortUrl) <br>
 &nbsp;&nbsp;  UserVisibility.canView(shortUrl, user): (ok = true) <br>
&nbsp; **then** <br>
&nbsp;&nbsp;&nbsp;   AnalyticsLog.count(shortUrl) <br>

<br>
<br>
<br>

### 3. Modularity of our Solution

**1. Allowing users to choose their own short URLs**
<br>
We can reuse UrlShortening.register but with a request that carries the desired suffix. The sync would pass it directly. No new concept needed.
<br>
<br>
**2. Using the “word as nonce” strategy to generate more memorable short URLs** <br>
This can hook in with the earlier WordNonceGeneration idea. Just add syncs so that shortenUrlMemorable uses the word generator instead of the default.
<br>
<br>
**3. Including the target URL in analytics, so that lookups of different short URLs can be grouped together when they refer to the same target URL** <br>
I do not think this feature should be included since it breaks the privacy rule since multiple users might shorten the same target. Grouping by target means one user could see traffic for a URL they didn’t create.
<br>
<br>
**4. Generate short URLs that are not easily guessed** <br>
This could be done by making a stronger nonce generator (like SecureNonceGeneration) and wiring it in with a new request type but at the end of the day it is still modular since it only changes how nonces are made.
<br>
<br>
**5. Supporting reporting of analytics to creators of short URLs who have not registered as user** <br>
This doesn’t fit with the ownership idea. If no user registered the URL, then there’s no one who should be allowed to see private analytics and doing this would require some kind of temporary token system, which adds complexity and weakens privacy.
