# pset 1
## exercise 1
#### invariants
- invariant 1: all `count` fields nonnegative
- invariant 2: every purchase should be associated with some request for an item
- personally i find it really important that the `count` field stays nonnegative. "important" is a weird word to me here, but if i pick this as the invariant, i literally don't think question 2 can be answered. but anyways, lets assume invariant 2 is the important one.
	- the `purchase` action requires the `item` to exist in the `registry` so it preserves it
#### fixing an action
- `removeItem` potentially breaks the invariant above, as it can remove an `item` that already has a `purchase` associated with the `request`
	- this could likely be fixed by adding a requirement that no purchases already be made before this removal is processed. however, i have some concerns.
		- i spent an entire day mulling over what these invariants could be and im still not 100% satisfied with this answer because i don't actually see why invariant 2 is fully necessary or as important as keeping the `count` fields nonnegative. it's just the only other thing i could come up with that isn't fundamentally untrue or redundant with the first invariant. my reason is that i perceived removing the item as not "i no longer want this item" but as "i no longer want this item *listed*". and the significance is that the only way to stop people from continuing to make purchases toward an item is by either closing the registry (which stops all purchases entirely), or by removing an individual item.
		- but also, i guess i get why it might be an issue if the whole record of any of these transactions is erased entirely.
#### inferring behavior
- it might be worth it to take down a registry to make changes, especially if you want to pause changes to the state by anyone other than the owner for a little bit. one case might be if you want to freeze it then take time to think about removing an item or something
#### registry deletion
- in practice, it probably doesnt matter?
	- most likely not not, because if someone wants a registry to stop being used, they can just close it
	- though a lot less important, one could argue yes, if there isn't some sort of time limit on which that data just persists for owners. its unlikely that would be a problem users would be particularly concerned about, but some might possibly dislike the lack of deletion capabilities
#### queries
- a registry owner might want to see the purchases that have been made
- a giver of a gift might want to see the unsatisfied requests
#### hiding queries
- add a owner visibility flag to registries, set it to false by default
#### generic types
- SKU codes encapsulate all of that information, and only having to serve one uniform piece of information for each item would simplify putting a GiftRegistration together
- additionally, the goal of a gift registration is not to present detailed information on items, that work should be left to external resources, which is what using an SKU code works towards
## exercise 2
1. complete the definition of the state
**concept** `PasswordAuthentication`
**purpose** limit access to known users
**principle** after a user registers with a username and a password,
    they can authenticate with that same username and password
    and be treated each time as the same user
**state**
	a set of Users with
		a `username` String
		a `password` String
2. write a requires/effects specification 
**actions**
	`register(username: String, password: String) : (user: User)`
		**requires** `username` not be taken by another User
		**effects** creates and returns a User with the corresponding `username` and `password`
	`authenticate(username: String, password: String) : (user: User)`
		**requires** some User with `username` and `password` exist in the set of Users
		**effects** returns the corresponding User
3. one essential invariant that must hold on the state
- the set of Users contains no `username` duplicates. this is preserved by the **requires** clause of `register`
4. extend concept to include email confirmation for registration
**state**
	a set of Users with
		`username` String
		`password` String
	a set of PendingUsers with
		a `user`
		a `token` String
**actions**
	`register(username: String, password: String, email: String) : (user: User, token: String)`
		**requires** `username` not be taken by another User
		**effects** creates & returns a User with the corresponding `username` and `password` and adds it to the set of Users. sends a secret `token` to the given `email` for verification, and creates a corresponding PendingUser with the `user` and `token`
	`confirm(username: String, password: String, token: String) : (user: User)`
		**requires** a PendingUser with a `token` and a `user` corresponding to the given `username` and `password` exists in the set of PendingUsers
		**effects** removes the PendingUser and returns the corresponding User
	`authenticate(username: String, password: String) : (user: User)`
		**requires** some User with `username` and `password` exist in the set of Users and is not a PendingUser
		**effects** returns the corresponding User

## exercise 3
#### concept specification for *PersonalAccessToken*
**concept** `PersonalAccessToken[User, Resource]`
**purpose** authenticate actions on a resource associated with the owner
	without requiring a password 
**principle** users (owners) can create tokens with which another user can use to get
	authorization for an action that they might not have the required permissions
	for, provided that the owner has the necessary permissions
**state**
	a set of Tokens with
		a `tokenString` String
		a set of Scopes with
			a `permission` String
		an `expiration` Date
		an `owner` User
		an active `status` Flag
	a set of Resources with
		a set of Actions
		a set of `permission` Strings
**actions**
	`createToken(user: User, scopes: Set<String>, expiration: Date) : (token: Token)`
		**requires** nothing
		**effects** creates & returns a Token with `owner=user`, the given set of `scopes`, and the given `expiration` Date
	`expireToken(token: Token)`
		**requires** `token` to exist and be active
		**effects** sets the `status` of the `token` to "expired"
	`authenticate(token: Token, resource: Resource, action: Action) : (result: any)`
		**requires** token to exist and be active, the `owner` of the token have proper permissions to take the given `action` on the `resource`, and the `resource` contain permissions that overlap with the token `scopes`
		**effects** takes the given `action` on the `resource` and returns the associated result 
#### differences
- the primary difference is that a standard `PasswordAuthentication` system aims to identify if an actor is who they claim to be, whereas the goal of a `PersonalAccessToken` seems to be to allow identical access across many different Users.
- I feel as though the GitHub documentation gets that message across decently okay, but I would have maybe described example workflows that can come out of using one of these.
## exercise 4
#### URL Shortener
**concept** AliasURL
**purpose** create an alias for a URL
**principle** users can create an alias of any URL, which anyone else can then use
	and be redirected to the original URL. users can either pick unique aliases
	or have one auto-generated
**state**
	a `base` String
	a set of Aliases with
		a `suffix` String
		a `url` String
**actions**
	`generateAlias(url: String) : (aliasURL: String)`
		**requires** nothing
		**effects** creates an Alias with a unique suffix and the given `url` and returns the corresponding `aliasURL` `(base + suffix)`
	`makeAlias(url: String, suffix: String) : (aliasURL: String)`
		**requires** `suffix` to not exist in the set of Aliases
		**effects** creates an Alias with the given `suffix` and `url` and returns the corresponding `aliasURL` `(base + suffix)`
- the general purpose of tinyurl.com and bit.ly does generally extend past shortening URLs. I believe a more accurate description of the functional behavior is that they create aliases for URLs with unique suffixes. naturally, that alias can be shorter, but that shouldn't be baked into the concept directly, as it's not even a requirement of these tools
#### Conference Room Booking
**concept** `RoomReservation[User, Room]`\*
**purpose** reserve rooms
**principle** users can reserve a room for a specific date & time, then manage their reservations
**state**
	a set of Rooms
	a set of Reservations with
		a Room
		a `start` Date
		an `end` Date
		a User
**actions**
	`reserve(room: Room, start: Date, end: Date, user: User) : (res: Reservation)`
		**requires** `room` to exist and to not have a Reservation overlapping `[start, end)`
		**effects** creates & returns a Reservation of the `room` for the `user` from `start` to `end`
	`editReservation(reservation: Reservation, start: Date, end: Date) : (res: Reservation)`
		**requires** the `reservation` to exist, and the associated `room` to not have a *different*\*\* Reservation overlapping `[start, end)`
		**effects** creates & returns a Reservation of the `room` associated with the `reservation` for the `user` from `start` to `end`, and deletes the old `reservation`
	`deleteReservation(reservation: Reservation)`
		**requires** the `reservation` to exist
		**effects** deletes the `reservation`
	`viewReservations(user: User) : (reservations: Set<Reservation>)`
		**requires**\*\*\* nothing
		**effects**\*\*\* returns the set of Reservations listed for the given `user`

- \* Room as a generic type makes the most sense, as that allows this concept to be generalized out further
- \*\* if the current Reservation overlaps the new time frame, this should not prevent an action which results in a state where the current Reservation no longer exists.
- \*\*\* these are things i've been unsure about since recitation
	- i could require `user` to have a Reservation, or i could require nothing. one could argue that having no Reservations should inherently be framed differently, but one could also argue that an empty set of Reservations perfectly captures the situation
	- i'm unsure if the **effects** sections should include a description of what's returned. i'm also unsure if it's different depending on if it's a state-modifying action or a query. in particular, the word "effects" seems to particularly imply that it should indicate actual changes to the state, and not return types. i guess i feel particularly conflicted when thinking about JS TypeDoc tags (`@requires`, `@effects`, and `@returns` all exist)
#### Address Verification
**concept** `AddressVerification[User, Address, Database]`
**purpose** authenticate user identity with an address
**principle** users provide some level of address information, which is then
**state**
	a Database with
		a set of (User, Address) pairs
**actions**
	`authorize(user: User, address: Address)`
		**requires** `user` and `address` to be in the Database
		**effects** none

- i have very mixed feelings about this. i chose to leave the Database type arbitrary because, in theory, the approach for address lookups would likely involve referencing different types of external databases.
- i don't know how to properly address the whole "distributed" aspect of this. not sure if this makes sense, but i was envisioning an authenticator function that sends a request to some remote database, but this feels like implementation details, and i'm not quite sure how to capture that as a result.

