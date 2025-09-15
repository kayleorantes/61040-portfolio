# Assignment 2
[Assignment 2 (AKA Problem Set 1: Reading and Writing Concepts)](https://docs.google.com/document/d/e/2PACX-1vQKlq_e2QjrgSwl__TCxR8pp_3d1aUuvxnXqoYv2DsY7bPit33QEyH0NaeakanPV-3ILtYo2j8raChS/pub)





````markdown
# Problem Set 1: Reading and Writing Concepts

---

## Exercise 1: Reading a concept

**Invariants.**  
What are two invariants of the state? (Hint: one is about aggregation/counts of items, and one relates requests and purchases). Say which one is more important and why; identify the action whose design is most affected by it, and say how it preserves it.

- One invariant of the state is that all purchases are related to an existing request in the registry for their items.  
- Another invariant is that each gift can only be purchased once, so the number of purchases ≤ the number of requests in the registry.  

I think the second is more important so that multiple people don’t end up purchasing the same gift. The `purchase` action is most affected by this design, and keeping track of requests and purchases ensures the invariant stays true.

**Fixing an action.**  
Can you identify an action that potentially breaks this important invariant, and say how this might happen? How might this problem be fixed?

- Adding an item could break the first invariant if the item isn’t officially added when a user wants to make that purchase.

**Inferring behavior.**  
The operational principle describes the typical scenario in which the registry is opened and eventually closed. But a concept specification often allows other scenarios. By reading the specs of the concept actions, say whether a registry can be opened and closed repeatedly. What is a reason to allow this?

- Yes, because an event might be canceled and then reopened for any reason for the user.  

**Registry deletion.**  
There is no action to delete a registry. Would this matter in practice?

- As a storage issue, it would be a bothersome item if a user couldn’t delete the registry, but in practice, no system issue would occur.  

**Queries.**  
What are two common queries likely to be executed against the concept state? (Hint: one is executed by a registry owner, and one by a giver of a gift.)

- Registry owner → *Who bought what items?*  
- Giver → *What can be bought?*  

**Hiding purchases.**  
A common feature of gift registries is to allow the recipient to choose not to see purchases so that an element of surprise is retained. How would you augment the concept specification to support this?

- Add a flag to purchases and hide/show them based on a boolean value.  

**Generic types.**  
The User and Item types are specified as generic parameters. The Item type might be populated by SKU codes, for example. Explain why this is preferable to representing items with their names, descriptions, prices, etc.

- Prices can fluctuate, names can change, but the SKU code stays reliable and avoids ambiguity (e.g., “flour, 24oz” vs. the SKU for that exact item).

---

## Exercise 2: Extending a familiar concept

```text
concept PasswordAuthentication
purpose limit access to known users
principle after a user registers with a username and a password, they can authenticate with that same username and password and be treated each time as the same user
````

**State**

```
a set of Users with 
  Username: String,
  passwordHash: String
```

**Actions**

* `register(username: String, password: String): (user: User)`

  * Requires: username is unique and not taken
  * Effects: creates a new user with the username and password hash

* `authenticate(username: String, password: String): (user: User)`

  * Requires: username exists and password matches its hash
  * Effects: returns the user if correct, else nothing

**Invariant**
Each username is unique. Preserved because `register` will not allow duplicate usernames.

**Extension (Email Confirmation)**

Add:

* **Extra State**:

  * Potential users with `(username, password, email, verified: Boolean, token)`
* **Extra Action**:

  * `confirm(username: String, token: String)`

    * Requires: user exists with stored token
    * Effects: sets `verified = true` for that user

---

## Exercise 3: Comparing concepts

```text
concept PersonalAccessToken
purpose provide a safe authentication alternative to passwords
principle a user generates one or more tokens linked to their account and each token will act like a password but access can be revoked or limited
```

**State**

```
a set of Users
a set of Tokens with
  owner: User
  value: String
  scopes: Set<Scope>
  active: Flag
```

**Actions**

* `generate(user: User, scopes: Set<Scope>): (token: Token)`

  * Requires: valid user and scope
  * Effects: creates a token linked to user

* `revoke(token: Token)`

  * Requires: valid token
  * Effects: sets `active = false`

* `authenticate(username: String, token: String): (user: User)`

  * Requires: token matches
  * Effects: returns corresponding user

---

## Exercise 4: Defining familiar concepts

### Concept: URLShortener

```text
concept URLShortener
purpose provide shorter links that redirect to longer URLs
principle user provides a long URL and receives a shorter URL that redirects to the correct one
```

**State**

```
a set of Mappings with
  shortURL: String
  longURL: String
```

**Actions**

* `shorten(longURL: String): (shortURL: String)`

  * Requires: longURL is valid
  * Effects: creates and returns a new mapping with an auto-generated short URL

* `customize(longURL: String, shortURL: String)`

  * Requires: shortURL not already taken
  * Effects: creates new mapping with this short/long pair

* `redirect(shortURL: String): (longURL: String)`

  * Requires: mapping exists
  * Effects: returns the longURL

**Summary**
Simple mapping from short → long URLs. No collisions since uniqueness is enforced.

---

### Concept: ConferenceRoomBooking

```text
concept ConferenceRoomBooking
purpose allow users to reserve rooms for meetings without conflicts
principle a user can see available rooms, book a room for a time slot, and cancel; no overlapping bookings allowed
```

**State**

```
a set of Rooms
a set of Bookings with
  room: Room
  startTime: Time
  endTime: Time
  bookedBy: User
```

**Actions**

* `book(room: Room, startTime: Time, endTime: Time, user: User): (booking: Booking)`

  * Requires: no overlapping booking in same room/time
  * Effects: creates new booking

* `cancel(booking: Booking)`

  * Requires: booking exists and active
  * Effects: removes booking

* `viewBookings(room: Room): (set of Bookings)`

  * Requires: room exists
  * Effects: returns all active bookings for that room

**Summary**
No overlapping bookings per room. Could be extended to enforce “one active booking per user.”

---

### Concept: ElectronicBoardingPass

```text
concept ElectronicBoardingPass
purpose give passengers a digital boarding pass that can be updated
principle a boarding pass is issued when a passenger checks in for a flight
```

**State**

```
a set of Passes with
  passenger: User
  flight: Flight
  seat: String
  gate: String
  boardingTime: Time
  active: Flag
```

**Actions**

* `issue(passenger: User, flight: Flight): (pass: Pass)`

  * Requires: valid booking for passenger on flight
  * Effects: creates a pass and marks it active

* `update(pass: Pass, gate: String, boardingTime: Time, seat: String)`

  * Requires: pass exists and is active
  * Effects: updates pass info

* `invalidate(pass: Pass)`

  * Requires: pass exists and valid
  * Effects: sets `active = false`

