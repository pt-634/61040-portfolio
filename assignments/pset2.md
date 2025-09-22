## exercise 1 - concept questions

#### Contexts
- The contexts (or, rather, the ability to have multiple contexts under which we consider nonces to exist) are for cases where uniqueness is needed in relation to multiple things, but uniqueness isn't required across each thing. For example, Discord allows users to appear as if they're sharing usernames, when it really just adds a nonce of sorts to the end of each name. There's no reason for `steve_urkel` and `michael_kyle` to share a nonce pool, since the usernames are inherently unique regardless of what nonce is appended to the end. So each of these could be treated as separate contexts, so that the next person that makes an account with the username context `steve_urkel`, his true username ends up with `stevel_urkel#1` or something, and the user identities can be treated separately.
- For the URL shortening app, the context is the base URL (tinyurl.com, bit.ly, etc.)
#### Storing used strings
- Can't tell if "Why must the _NonceGeneration_ store sets of used strings?" is a rhetorical question or not.
	- After doing the abstraction function below, I'm assuming it is, and that the point was that the strings don't need to be *physically* stored if the nonce's are just the set of numbers between some minimum and maximum value.
	- However, if it isn't rhetorical, them I'm assuming the emphasis is on the word "used" here, and it's simply that a way to avoid duplicates is absolutely necessary.
- Abstraction function, mapping a counter to a set of used strings:
	- `AF(counter) = the set of strings with counter items where each string is a number from 1 to counter`

#### Words as nonces
- Advantages/disadvantages
	- Reflecting back on 6.102 nanoquizzes, it made it easier to type and communicate the link if it was a common word ("yellkey.com/hello" is infinitely easier than tinyurl.com/a1b4jg or something random, and can actually be verbally communicated).
	- I guess a disadvantage here is that there's probably a theoretical cap on common dictionary words, so this wouldn't function very well outside of a situation where shortenings are frequently retired. I imagine users would have to relinquish URLs faster than necessary in other nonce-based platforms, or something of that nature (like quicker expiry required for something to work in practice, though I'm not sure if that's true anyways).
		- In case that answer is leaning too much into implementation, I'm not entirely sure why this would be a problem in practice, but a URL would become more guessable in this scenario. I guess an annoying circumstance would be if something like kahoot generated dictionary words instead of random number sequences for game room codes, resulting in random students in some classroom easily hijacking a kahoot going on in another classroom after hearing the game music through the walls (I'm sorta-kinda referencing a specific experience here, lol). Or like if the 6.102 nanoquizzes were changed to occur at the exact same time every day in class, it might be considered easier for students to guess what a nanoquiz site was without coming to class (but there's a class code at the end so it literally is irrelevant in practice, lol).
- Modifying _NonceGeneration_
	- This is more of an implementation detail, no? Or is the request to modify the concept itself to be something like a _DictionaryWordNonceGeneration_ concept?
	- Assuming the latter, I would modify the **state** by adding a WordBank (a set of strings). I would change the **effect** of generate to say "returns a nonce that is not already used by this context that exists in the WordBank" and then add a **requires** statement saying "the WordBank must contain words not yet in the set of used Strings associated with `context`".

## exercise 2 - synchronizations
#### Partial matching
- In the first sync, only `shortUrlBase` is referenced by the **then** clause, so the other is omitted for brevity and clarity. Both are referenced in the second sync's **then** clause and thus must both be referenced in the **when** clause.
#### Omitting names
- Well, in the case of all the shown **when** clauses, this remains true. This is untrue in all the **then** clauses, notably because the arguments local to the referenced actions have names that either need to be mapped to a variable from the **when** clause, or (in the case of `seconds: 3600`) have a real value that is not made clear by their names alone.
- I can maybe imagine one scenario where it isn't the case that all the actions in the **when** clause do not exercise this sort of brevity, and it's if their outputs have overlapping names that should, in theory, be distinct, or if their inputs don't have overlapping names, but are intended to have taken in the same input.
	- Turns out I ran into a scenario in the "Adding a sync" section below; I made the executive decision to rename the `expireResource` action's `resource` output to `shortUrl` because it fit the overarching context better, and then I did omission in the **then** clause instead with `delete`.
#### Inclusion of request
- Resource expiration is not a direct result of requesting one. It just so happens that one downstream effect of a request is the creation of the resource with `register`, and *that* is what's actually worth associating with resource expiration. The outputs of any Request action do not have any reason to be mentioned in the **then** clause of the third sync.
#### Fixed domain
- The two ideas that come to mind immediately are to either:
	- omitting the hardcoded argumetns
	- including the names of hardcoded arguments, with their hardcoded values
- I'm leaning towards the latter, both to pattern match the hardcoding situation in sync 3 (`seconds: 3600`) and because it's more clear than omitting things entirely.
#### Adding a sync
**sync** expire
**when** `ExpiringResource.expireResource () : (resource: shortUrl)`
**then** `UrlShortening.delete (shortUrl)`

## exercise 3 - extending the design
#### Additional concepts
- Note: Something makes me believe it makes more sense to maintain the Request philosophy [here](https://piazza.com/class/melw05erqym3qt/post/67) and add an action for requesting analytics to that concept, but alas.

- PasswordAuth (pulled from last week's Pset, bar for bar)
**concept** PasswordAuth
**purpose** limit access to known users
**principle** after a user registers with a username and a password,  
    they can authenticate with that same username and password  
    and be treated each time as the same user
**state**
	a set of Users with
		a username String
		a password String
**actions**
	register (username: String, password: String): (user: User)
		...
	authenticate (username: String, password: String): (user: User)
		...

- Note: unsure if we were supposed to generalize this to not be "Url" specific, as in general it could've just been something that's associated with users that increments by 1 every time an action occurs.
**concept** UrlAnalytics\[User\]
**purpose** allow users to view analytics about a URL they registered
**principle** tracks URL registrations by users and visit counts, allowing for 
	retrieval of how frequently the URLs are being used
**state**
	a set of URLs with
		a User
		a count Number
**actions**
	associate(user: User, url: String)
		**requires** `url` not yet exist in the set
		**effects** pairs this `url` with the `user` and starts its count at 0
	dissociate(user: User, url: String)
		**requires** `url` exist in the set
		**effects** remove this `url` from the set
	increment(url: String)
		**requires** `url` exist in the set
		**effects** increments the count associated with the `url` by 1
	requestView(user: User, url: String)
		**requires** `url` exist in the set and be associated with `user`
	view(url: String) : (count: Number)
		**requires** `url` exist in the set
		**effects** returns the `count` associated with the `url`
#### Three essential syncs
- shortenings created
**sync** trackUrl
**when** 
	`UrlShortening.register () : (shortUrl)`
	`PasswordAuth.register () : (user)`
**then** `UrlAnalytics.associate (user, url: shortUrl)`

- shortenings translated
**sync** countAccess
**when** `lookup (shortUrl)`
**then** `UrlAnalytics.increment (url: shortUrl)`

- user examines analytics
**sync** generateAnalytics
**when** 
	`authenticate () : (user)`
	`requestView (user, url)`
**then** `UrlAnalytics.view (url)`
#### Consider feature requests
- Allowing users to choose their own short URLs;
	- In my mind, that's already permitted by the inclusion of something like `shortUrlSuffix` in the UrlShortening concept. But the existing "generate" sync needs to be changed to not use `nonce` for that by default in order to change this. My additions only reference `shortUrl` directly, so this unrelated feature isn't really affected by that.
- Using the “word as nonce” strategy to generate more memorable short URLs;
	- I still don't understand why this is baked into the concept and not just left up to being an implementation detail if I'm being honest.
	- Anyways, I think the simplest approach would be changing the _NonceGeneration_ stuff as detailed earlier in the Pset. I don't see a reason for baking this into the concept still, though. But I strongly doubt it would be difficult to make the change.
- Including the target URL in analytics, so that lookups of different short URLs can be grouped together when they refer to the same target URL;
	- Would not be hard to integrate into the design with a few tweaks to the existing syncs (not omitting targetUrl in the trackUrl sync, etc.), and could potentially have interesting data. One scenario this could be useful is if some advertising business included different links to the same target for several channels of advertisement, then used the analytics to see which channel brought in the most traffic.
- Generate short URLs that are not easily guessed;
	- I just genuinely don't think this is a problem that's interesting to solve in practice, as most URLs where this might become a concern are either behind authentication walls or can handle guessability issues indirectly. This isn't worth packing into the existing system.
- Supporting reporting of analytics to creators of short URLs who have not registered as user.
	- I think this can be addressed with an accessibility flag on registered shortenings, and maybe an additional action to change this flag. Doesn't feel super useful, but doesn't feel super problematic to make work either though.
