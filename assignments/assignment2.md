# Assignment 2
[Link to PSET in google doc!](https://docs.google.com/document/d/e/2PACX-1vQKlq_e2QjrgSwl__TCxR8pp_3d1aUuvxnXqoYv2DsY7bPit33QEyH0NaeakanPV-3ILtYo2j8raChS/pub)

Exercise 1: Reading a concept

1. **Invariants**. What are two invariants of the state? (*Hint*: one is about aggregation/counts of items, and one relates requests and purchases). Say which one is more important and why; identify the action whose design is most affected by it, and say how it preserves it.

One invariant of the state is that all purchases are related to an existing request in the registry for their items.

Another invariant is that each gift can only be purchased once so the number of purchases are <= the number of requests in the registry.

I think the second is more important so that multiple people don’t end up purchasing the same gift. The purchase count is the action that the design is most affected by and keeping track of requests and purchases will ensure the invariant stays true.

1. **Fixing an action**. Can you identify an action that potentially breaks this important invariant, and say how this might happen? How might this problem be fixed?

Adding an item could break the first invariant if the item isn’t officially added when a user wants to make that purchase.

1. **Inferring behavior**. The operational principle describes the typical scenario in which the registry is opened and eventually closed. But a concept specification often allows other scenarios. By reading the specs of the concept actions, say whether a registry can be opened and closed repeatedly. What is a reason to allow this?

Yes, because an event might be canceled and then reopened for any reason for the user.

1. **Registry deletion**. There is no action to delete a registry. Would this matter in practice?

As a storage issue, it would be a bothersome item if a user couldn’t delete the registry, but in practice, no system issue would occur.

1. **Queries**. What are two common queries likely to be executed against the concept state? (*Hint*: one is executed by a registry owner, and one by a giver of a gift.)

Registry owner- who bought what items?

Giver- what can be bought?

1. **Hiding purchases**. A common feature of gift registries is to allow the recipient to choose not to see purchases so that an element of surprise is retained. How would you augment the concept specification to support this?

You could add a flag to make purchases and hide/show the purchases based on the value of the boolean.

1. **Generic types**. The User and Item types are specified as generic parameters. The Item type might be populated by SKU codes, for example. Explain why this is preferable to representing items with their names, descriptions, prices, etc.

Prices can fluctuate, names can change, but the SKU code will stay reliable and avoid ambiguity in the representation details (like ‘flour, 24oz’ vs the specific SKU for that specific item).






Exercise 2: Extending a familiar concept

**concept** PasswordAuthentication
**purpose** limit access to known users
**principle** after a user registers with a username and a password, they can authenticate with that same username and password and be treated each time as the same user **state**a set of Users with

Username: String,

passwordHash: String
**actions**register (username: String, password: String): (user: User)
 Requires that the username is unique from the past and is not taken

Effects: creates a new user with the username and with passwordHash

authenticate (username: String, password: String): (user: User)
 Requires that the user with username exists and that the password matches the hash for it

Effects is that the user is returned if correct, else nothing

1. Complete the definition of the concept state.

**state**a set of Users with

Username: String,

passwordHash: String

1. Write a requires/effects specification for each of the two actions. (*Hints*: The register action creates and returns a new user. The authenticate action is primarily a guard, and doesn’t mutate the state.)

**actions**register (username: String, password: String): (user: User)
 Requires that the username is unique from the past and is not taken

Effects: creates a new user with the username and with passwordHash

authenticate (username: String, password: String): (user: User)
 Requires that the user with username exists & password matches the hash for it

Effects is that the user is returned if correct, else nothing

1. What essential invariant must hold on the state? How is it preserved?

Invariant: each username is unique, and this is preserved since register will not allow a user to create an account with a duplicated username

1. One widely used extension of this concept requires that registration be confirmed by email. Extend the concept to include this functionality. (*Hints*: you should add (1) an extra result variable to the register action that returns a secret token that (via a sync) will be emailed to the user; (2) a new confirm action that takes a username and a secret token and completes the registration; (3) whatever additional state is needed to support this behavior.)

We could create another state that is ‘potential users,’ with username, password, email, and a boolean for verified, along with the forementioned token.

**Extra state:**
 confirmed: Boolean

**Extra action:**

confirmed(username: String, password: String, email: String, verified: Boolean, storedToken: token)

Requires matching user exists with the stored token

Effects will set confirmed as true for that user after successful confirmation







Exercise 3: Comparing concepts

**concept** PersonalAccessToken

**purpose** to provide a safe authentication alternative to passwords

**principle** a user generates one or more tokens linked to their account and each token will act like a password but access can be revoked or limited

**state**

a set of Users

a set of Tokens with

owner User

string value

scopes Set<Scope>

active Flag

**actions**

generate(user: User, scopes: Set<Scope>): (token: Token)

Requires a valid user and scope

Effects is that a token is created and linked to user

revoke(token: Token)

Requires valid token

Effects set active to false

authenticate(username: String, token: String): (user: User)

Requires the token to match

Effects is that it will return the corresponding user







Exercise 4: Defining familiar Concepts

**concept** URLShortener

**purpose** is to provide shorter links that redirect to longer URLs.

**principle** a user will provide the system with a long URL and will receive a shorter URL that redirects to the correct URL

**state**

a set of Mappings with

shortURL: String

longURL: String

**actions**

shorten(longURL: String): (shortURL: String)

Requires longURL is valid

Effects will create and return a new mapping with an auto-generated shorter URL

customize(longURL: String, shortURL: String)

Requires shortURL is not already taken

Effects creates a new mapping with this shortURL and longURL

redirect(shortURL: String): (longURL: String)

Requires mapping exists

Effects returns the mapped longURL

**Summary**

This is a very simple state & concept since it just maps short URLs to long ones. No collisions since we check URL uniqueness.








**concept** ConferenceRoomBooking

**purpose** allow users to reserve rooms for meetings without conflicts.

**principle** a user can see available rooms, create a booking for a specific time slot, and cancel bookings. No possible overlapping bookings for the same room during the same time slot

**state**

a set of Rooms

a set of Bookings with

room: Room

startTime: Time

endTime: Time

bookedBy: User

**actions**

book(room: Room, startTime: Time, endTime: Time, user: User): (booking: Booking)

Requires no existing booking overlaps with the same room and time slot

Effects creates a new booking

cancel(booking: Booking)

Requires booking exists and is active

Effects removes the booking

viewBookings(room: Room): (set of Bookings)

Requires room exists

Effects returns all active bookings for the room

**Summary**

We cannot have two bookings overlap in the same room, could extend so a user is only allowed to book one room at a time








**concept** ElectronicBoardingPass

**purpose** gives passengers with a digital boarding pass that can be updated

**principle** a boarding pass is issued when a passenger checks in for a flight

**state**

a set of Passes with

passenger: User

flight: Flight

seat: String

gate: String

boardingTime: Time

active: Flag

**actions**

issue(passenger: User, flight: Flight): (pass: Pass)

Requires a valid booking for passenger on flight

Effects is that it creates a pass and marks user active

update(pass: Pass, gate: String, boardingTime: Time, seat: String)

Requires that the pass exists and is valid/active

Effects updates fields with new information

invalidate(pass: Pass)

Requires pass is valid/exists

Effects is that active is set to false
