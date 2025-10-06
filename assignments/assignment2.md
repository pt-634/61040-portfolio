## problem domain
### domain - playing card games
I play (and teach) a lot of card games across different friend groups and contexts. Over time, groups change, and I forget rules, variants, or which games fit which settings. I care about preserving the games I’ve enjoyed, recalling them quickly, and being able to quickly decide what to play next based on the group I'm with, while keeping everyone interested.
### problem - picking the right card game for a group, fast
When a group sits down to play, we often waste 10–20 minutes debating “what should we play?” and then another chunk of time re-learning rules. The friction comes from (1) poor recall of past favorites, (2) lack of filters that matter in the moment (players, time available, difficulty), and (3) instructions that are too long or inconsistent to explain purely from an infrequent memory recall. The result is decision fatigue, stalled momentum, and games that don’t fit the group or time box.
### stakeholders
- **selector** - the person selecting the game (often the initiator of the session); needs fast, confident picks that align with the number of players, time available, and difficulty.
- **player** - each participant; wants a fun and appropriately complex game with minimal setup overhead to keep things interesting
- **explainer** - the rules leader/teacher; needs short, consistent, and easy to understand instructions and quick references for edge cases
- **contributors** - community members proposing new games or variants; may appreciate this as a mode of outreach for their card game/variant (if an original invention) or as a way to help popularize a favorite game
	- *NOTE: contributors would need a submission path and some degree of moderation. may be out of scope for these reasons, but still in consideration.*
### evidence and comparables
- personal observations
	- **game choice friction** - in every category of gaming (card games, video games, board games), i've felt this phenomenon. it's particularly problematic for a person like me, who both struggles to come to decisions and is friends with many that share that same difficulty.
	- **setup overhead** - this is an important factor in getting people interested in doing anything for entertainment, so this also isn't limited to card games. if it seems too complicated to learn something (for whatever the level of experience the group has) or if an explanation of the proposed activity is not articulated clearly enough, the likelihood that the group moves forward with the activity is low. this can result in enjoyed activities becoming stale from being done repeatedly, and less exploration into activities that have potential but seem to be plagued with this setup overhead problem
	- **memory** - there are a number of card games that i remember being a lot of fun but enjoy less after playing with certain groups (and not just ones from childhood, when a larger set of games was considered "fun" to me). this can be attributed to the game not matching the group, but something clear to me is that part of the issue is i usually fail to remember the rules before we start playing. as a result, this results in me only recommending the games i've played recently to avoid incorrect explanations, which lowers the ceiling of possibilities within every session
- external references
	- These sources are evidence of conversation about struggles with remembering the rules for playing games. There seems to be some agreed level of difficulty with remembering how to play multiple games, and there are comments of it being a barrier to even wanting to set the game up, and that the frequency of which a game is played and taught is, in people's reported experiences, related to how well they remember them. I do recognize that these sources all generally refer to board games, but I believe these principles to be transferrable.
		- [How do you remember all the rules? (reddit)](https://www.reddit.com/r/boardgames/comments/rztztw/how_do_you_remember_all_the_rules/)
		- [BoardGameGeek post on remembering game rules](https://boardgamegeek.com/thread/2069761/okay-this-sounds-crazy-buthow-do-you-remember-game)
		- [Do you guys forget game rules after approximately a month of not playing it? (reddit)](https://www.reddit.com/r/boardgames/comments/7uou0h/do_you_guys_forget_game_rules_after_approximately/)
	- This post highlights the fact that it's quite few reasons needed to have a stake in this. The user who posted the question requested a certain level of complexity and gave some indication of aversion toward a high barrier to entry, particularly in reference to long manuals. 
		- [Looking for games I can play with a standard deck of cards and/or some dice](https://www.reddit.com/r/soloboardgaming/comments/1br5rfn/looking_for_games_i_can_play_with_a_standard_deck/)
	- I've yet to see anything similar already existence after searching the web and the Apple app store. The closest thing that comes up are apps that are specifically designed for a singular card game and seem to mostly be focused on serving as a digital platform to play the game on.
## application pitch
### name
- Decktionary
- still deciding if the "t" should be capitalized (Decktionary vs DeckTionary)
- icon will be a book with a joker on the cover
- potential tagline: "DeckTionary — always know what you're dealing with."
	- could use a lot of workshopping
### motivation
When friends sit down to play cards, choosing *what* to play and remembering *how* to play the game always takes too long. **DeckTionary** solves this by acting as both a quick-reference library and a personalized guide to picking a fitting game, learning or recalling the rules in seconds, and keeping track of what everyone enjoyed most.
### key features
**Smart Recommender**
- Tell DeckTionary how many people you’re playing with, how much time you have, and the level of complexity you want, and it instantly recommends card games that fit those parameters. Each suggestion includes a short summary and estimated playtime.
- Helps *Selectors* overcome decision fatigue and get to playing faster.
**Quick Rulesheets**
- Every game comes with a concise, standardized rule page outlining setup, key actions, and win conditions. These are written to be taught very quickly.
- Supports *explainers* and newer *players* by making teaching simple and consistent, even when months go by between game sessions.
**Personal Deck**
- Users can like, dislike, and bookmark games to build a personal catalog of favorites. DeckTionary’s recommendations adapt to preferences.
- Gives *players* continuity across sessions and helps rediscovers forgotten favorites without memorization.
## concept design
### concept specifications
**concept** Catalog
**purpose** maintain structured records of all card games and their attributes
**principle** every listed game has a consistent entry that can be looked up and filtered
**state**
	a set of Games with
		a name String
		a Rulesheet
		a minPlayers Number
		a maxPlayers Number
		a Difficulty
		a playTime Number
**actions**
	`register(name: string, rulesheet: Rulesheet, minPlayers: Number, maxPlayers: Number, difficulty: Difficulty, playTime: Number)`
		**requires** `name` to not already be associated with a Game
		**effects** adds a Game with the given attributes
	`lookup(name: String) : (game: Game)`
		**requires** `name` to be associated with an existing Game
		**effects** retrieves the game with the corresponding `name`
	`filter(playerCount: Number, playTime: Number, difficulty: Difficulty) : (games: Games[])`
		**requires** none
		**effects** returns all Games with matching criteria; that is, a game will not be filtered out `minPlayers <= playerCount <= maxPlayers`, its difficulty matches the one given, and does not take longer than `playTime` to play
	`remove(game: Game)`
		**requires** `game` exists
		**effects** removes the given `game` from the set of Games
		

**concept** Libraries\[User, Game\]
**purpose** let users record likes, dislikes, and bookmarks
**principle** every user has a library that reflects their experience with games
**state**
	a set of Libraries with
		a User
		a set of Games with
			a Rating (Like or Dislike or Undefined)
**actions**
	`bookmark(user: User, game: Game)`
		**requires** `game` not already be in the `user` Library
		**effects** adds the `game` to the Library of the given `user`
	`removeBookmark(user: User, game: Game)`
		**requires** `game` be in the Library of the given `user` (and the `user` to exist)
		**effects** removes the `game` from the Library of the given `user`
	`rate(user: User, game: Game, rating: Rating)`
		**requires** none
		**effects** adds this `game` to the `user`'s library if it's not already present, and applies the `rating`
	`removeRating(user: User, game: Game)`
		**requires** `game` be in the `user`'s library, with a defined rating (and `user` to exist)
		**effects** sets the `user`'s rating of the `game` to Undefined
	`getLikes(user: User) : (games: Game[])`
		**requires** `user` to exist
		effects returns all the games liked by this `user`
	`getBookmarks(user: User) : (games: Game[])`
		**requires** `user` to exist
		effects returns all the games in this `user`'s library
	`getDislikes(user: User) : (games: Game[])`
		**requires** `user` to exist
		effects returns all the games disliked by this `user`

**concept** Recommender\[Game]
**purpose** generate game suggestions given context and user data
**principle** allows users to request recommendations given likes and dislikes
**state**
	a set of Requests with
		a id String
		a User
		a playerCount Number
		a Difficulty
		a playTime Number
		a satisfied Bool
		a response String
**actions**
	`request(user: User, playerCount: Number, difficulty: Difficulty, playTime: Number) : (id: String)`
		**requires** none
		**effects** creates a Request with a unique id, a `False` satisfied flag, an empty response string, and the given fields; returns the request id
	`respond(id: String, likes: Games[], matches: Games[])`
		**requires** `id` be associated with a Request and the Request not be satisfied
		**effects** marks the corresponding request as satisfied and sets the response string
	`consumeResponse(id: String) : (response: String)`
		**requires** `id` be associated with a Request and the Request be satisfied
		**effects** returns the response associated with the corresponding Request
	`removeRequest(id: String)`
		**requires** `id` be associated with a Request
		**effects** deletes the Request

**concept** PasswordAuthentication
**purpose** limit access to known users
**principle** after a user registers with a username and a password,
    they can authenticate with that same username and password
    and be treated each time as the same user
**state**
	a set of Users with
		a username String
		a password String
**actions**
	`register(username: String, password: String) : (user: User)`
		**requires** `username` not be taken by another User
		**effects** creates and returns a User with the corresponding `username` and `password`
	`authenticate(username: String, password: String) : (user: User)`
		**requires** some User with `username` and `password` exist in the set of Users
		**effects** returns the corresponding User

**concept** ExpiringResource \[Resource]  
**purpose** expire resources automatically to manage costs  
**principle** after setting expiry for a resource, the system will expire it after the given time  
**state**  
	a set of Resources with  
		an expiry DateTime  
**actions**  
	`setExpiry (resource: Resource, seconds: Number)`  
	  **effect** associates an expiry seconds from now with resource  
	**system** `expireResource () : (resource: Resource)`  
		**requires** expiry for some resource is in the past  
		**effect** returns the resource, and removes it from the set
### some essential syncs
**sync** satisfyRequest
**when** 
	`Recommender.request (user, playerCount, playTime, difficulty) : (id)`
**then** 
	`Libraries.getLikes(user) : (likes)`
	`Catalog.filter(playerCount, playTime, difficulty) : (matches)`
	`Recommender.respond (id, likes, matches)`

**sync** responseRetrieval
**when** `Recommender.consumeResponse (id)`
**then** `ExpiringResource.setExpiry (id)`

**sync** requestCleanup
**when** 
	`expireResource () : (id)`
**then** `removeRequest (id)`
### a brief note

The concepts in this design divide the app’s functionality into independent layers.  
**PasswordAuthentication** governs user access and ensures that actions in other concepts are tied to known users.  
**Catalog** anchors the system by maintaining the canonical set of games and their attributes.  
**Libraries** stores each user’s personal interaction history—likes, dislikes, and bookmarks—so that preferences can be referenced without changing the catalog itself.  
**Recommender** handles dynamic request generation and response delivery, drawing on both **Catalog** and **Libraries** to produce personalized game suggestions.  
The optional **ExpiringResource** concept manages temporary state, ensuring that recommendation requests or cached responses are cleaned up after a short period.

Together, these concepts satisfy the core flows of _DeckTionary_: authenticated users browse structured game data, log their preferences, and request tailored recommendations that automatically expire once viewed.
## UI sketches

## user journey
