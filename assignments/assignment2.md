# Assignment 2

   ## 1. Exercise 1

1. One invariant is that for all of the items in the Registry, the total number of those items purchased must not exceed the total number of those items requested. Another invariant is that for each request in the Registry, the total count of requested items must be $\geq 0$ at all times. The first invariant is more important because if the total number of some item purchased is greater than the number requested, then the purchaser spends extra money on items that the person making requests does not even need, and the person making requests will also have to deal with an excess of items, which is inconvenient for both parties. 
2. An action that breaks the above invariant is the purchase action, which could have synchronization issues that would lead to over-purchase of an item. For example, if two different purchasers check the count of a specific item at the exact same time, and both see that there is 1 left, then they could both purchase the item, which would result in 2 purchases when only 1 was needed. One possible fix is to not allow multiple purchases of the same item at the same time--for example, when a user begins to purchase an item, they can hold that item for 10 minutes during which no one else can purchase that item, then after that they either finish the purchase and the count is updated, or the purchase is cancelled and the count remains the same (this is similar to how concert ticket purchases function in real life). 
3. A registry can be opened and closed repeatedly. The open and close actions both only require that the registry exists and is currently in the opposite state as the action; there's no requirement on the open/close history (which is not tracked). A reason to allow this is that a user/registry owner may want to break the gift-giving into multiple phases--for example, open for a period before the event, then closed during the event, then open after the event for those who couldn't give gifts in time. 
4. It's a slight problem that there's no option to delete registries. If an event host made a registry by accident, or for an event that was ultimately cancelled, then that registry would still exist but be completely unused, therefore taking up unnecessary space. Also, if an event has passed and all thank-you notes have been sent, some users may not need their old registries anymore. Overall, the inability to delete registries leads to clutter, taking up unnecessary storage and creating confusion for users. However, this is not a major issue because the website would generally be built to support large amounts of storage, and most users probably won't have too many registries. 
5. A query is a read operation (one that accesses info but does not change the state). One possible query is by the registry owner to see which gifts were purchased, how many of each, and by whom (this can be done at any point, whether the registry is open or closed). Formally, this is retrieving a list of all purchases across all requests in a registry. Another query is by the gift giver to see which gifts are available for purchase and how many. Formally, this is retrieving a list of all requests (and remaining counts) where the remaining request count is > 0. 
6. To hide purchases from the registry owner, I would add a feature called a UserViewability flag to each Registry. This flag would have a Visible state (where the user can see purchases) and a Hidden state (where the user cannot see purchases). The corresponding actions would be hide(registry: Registry) and unhide(registry: Registry), each of which requires that the registry exists and is currently in the opposite state as the action. 
7. Specifying Items using SKU codes instead of descriptions such as names and prices is better for multiple reasons. First of all, the SKU code is a reference instead of a hard-coded object, so if the object's details change while the registry is active, the Item won't need to be updated on the Registry website since its SKU code will be the same. Also, this is an abstraction that reduces unnecessary data usage--instead of storing a bunch of parameters, the website can simply store an 8-character string for each item, which saves a lot of space. It also reduces visual clutter since each Item would have a much shorter representation. 

## 2. Exercise 2

1. The state is: 
   
   A set of Users with 
   
   * a username String

   * a password String

2. Requires and effects:
   1. register(username: String, password: String): (user: User)
      1. **requires:** username doesn't currently exist
      2. **effects:** creates and returns new User with username String and password String
   2. authenticate(username: String, password: String): (user: User)
      1. **requires:** username currently exists and (username, password) is a matching valid combination
3. The essential invariant on the state is that all users must have distinct usernames. This is preserved by the requires condition in the register action, which only allows the registration of a username if there don't exist any users with that username.
4. To add email confirmation, add/change the following things:
   1. create token String; state becomes:
      1. A set of Users with
         1. a username String
         2. a password String
         3. a token String
         4. a Confirmed Flag
   2. register function becomes register(username: String, password: String): (user: User)
      1. **requires:** username doesn't currently exist
      2. **effects:** creates and returns new user with username String, password String, token String, and Confirmed flag set to False
   3. authenticate function becomes authenticate(username: String, password: String): (user: User)
      1. **requires:** username currently exists, (username, password) is a matching valid combination, and Confirmed Flag is True
   4. action: confirm(username: String, token: String)
      1. **requires:** username exists and the token matches the one associated with the User with this username
      2. **effects:** changes Confirmed Flag of User to True

## 3. Exercise 3

* **concept** PersonalAccessToken [User]
* **purpose** manage access to a set of resources
* **principle** a user creates a token that's associated with a set of resources; when the user presents a token, they're authenticated to that set of resources
* **state**
  * a set of Users with
    * a set of Tokens
  * a set of Tokens with
    * an associated User
    * a name: String
    * a set of accessible resources: Resources
    * an isActive flag: Boolean
    * an ExpirationDate
* **actions**
  * addUser(user: User): (user: User)
    * **effect:** adds User with no Tokens
  * createToken(user: User, name: String, resources: Resources, ExpirationDate): Token
    * **requires:** User exists
    * **effect:** creates and returns Token with owner User, name String, accessible Resources, isActive flag set to True, and ExpirationDate; adds Token to User's set of Tokens
  * revokeToken(user: User, token: Token)
    * **requires:** Token exists and has owner User
    * **effect:** sets isActive flag to False
  * authenticate(user: User, token: Token): (Resources)
    * **requires:** User exists, Token is in User's list of Tokens, Token's isActive flag is True and time is before ExpirationDate
    * **effects:** grants User access to Token's Resources; return Resources

PasswordAuthentication and PersonalAccessToken are different in many ways. First of all, PasswordAuthentication grants access to exactly all of one user's resources, while PersonalAccessToken grants access to any set of resources (can be part of one user's resources, can contain multiple users' resources, etc.). Also, one can't force a user to delete their account or password, meaning that removing PasswordAuthentication and revoking all of a user's permissions is not possible, while it's possible to revoke a PersonalAccessToken and therefore a user's access to a certain set of resources. Finally, a user can choose their own password, while tokens are generated for them.

The GitHub page is overall very clear and concise, but one minor improvement is they could clarify that tokens and passwords are different, and explicitly explain the difference. The statement "Personal access tokens are like passwords" is helpful for commanding the user to maintain secrecy, but it does not fully clarify the differences between tokens and passwords. 

## 4. Exercise 4

1. Billable Hours Tracking:
   * Name: TrackWorkHours
   * Purpose: enables employees to track time spent at work
   * Operational principle: an employee begins new session by selecting a project and providing a description; system records start time; employee ends session when done, and system records end time. 
   * State: 
     * set of Users with
       * a set of Sessions
     * set of Sessions with
       * user: User working on session
       * project: Project being worked on
       * description: String describing project
       * startTime: logged start time
       * endTime: logged end time
   * Actions: 
     * addEmployee(user: User): (user: User)
       * effects: add User associated with empty set of Sessions
     * removeEmployee(user: User)
       * requires: User exists
       * effects: removes User from set of Users; removes all Sessions associated with User from set of Sessions
     * startSession(user: User, project: Project, description: Description): (session: Session)
       * requires: User exists
       * effects: if there are any currently active (no logged end time) sessions, log their end times; starts and returns new Session with user, project, and description; adds Session to sessions associated with User; logs start time
     * endSession(user: User, session: Session)
       * requires: User exists and is associated with Session
       * effects: logs end time
     * getSessions(user: User)
       * requires: User exists
       * effects: returns list of all Sessions associated with User

2. Conference Room Booking:
   * Name: BookRooms [User]
   * Purpose: enables users to reliably reserve rooms for events
   * Operational principle: owner of rooms makes slots available, people reserve a slot, and are guaranteed access to that room during that time
   * State: 
     * set of Slots with
       * a Time
       * a Location
     * set of Bookings with
       * a User
       * a Slot
   * Actions:
     * createSlot(t: Time, l: Location): (s: Slot)
       * **effects:** creates and returns new Slot with time Time and location Location
     * deleteSlot(s: Slot):
       * **requires:** Slot currently exists
       * **effects:** removes Slot and removes all Bookings with Slot
     * bookRoom(user: User, s: Slot): (b: Booking)
       * **requires:** Slot is not currently a part of any Bookings
       * **effects:** create and return Booking with user User and in slot Slot
     * cancelRoom(b: Booking)
       * **requires:** Booking exists
       * **effects:** remove Booking

3. Electronic Boarding Pass:
   * Name: MobileBoardingPass [User, FlightInfo]
   * Purpose: Provide realtime flight-related updates for users boarding a flight
   * Operational principle: Users download boarding passes onto their mobile devices; any realtime updates to their flight are reflected in the mobile boarding pass
   * State:
     * set of Users with
       * a set of BoardingPasses
     * set of BoardingPasses with
       * an associated User
       * a FlightInfo
       * a Used Flag
   * Actions:
     * addPass(user: User, FlightInfo): (BoardingPass)
       * **effects:** if User doesn't already exist, create new User and add to set of Users; adds BoardingPass with FlightInfo and Used Flag set to False to User's set of BoardingPasses
     * updateInfo(FlightInfo)
       * **effects:** looks up all BoardingPasses with FlightInfo and updates each one
     * usePass(user: User, BoardingPass)
       * **requires:** BoardingPass is in User's set of BoardingPasses
       * **effects:** changes Used Flag of BoardingPass to True