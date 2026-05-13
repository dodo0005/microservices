# GameHub — Understanding the system

10 questions to test your understanding of the data flow and architecture.
Work through them in order: read the code first, then run the app, then try to break things.

---

## How to investigate

You will need three things:

**1. Read the source code**
Start with `models.py` (the schema), then `seed.py` (the data), then `app.py` (the logic).
Many questions are answered entirely by reading carefully.

**2. Run the app and interact with it**
Use the UI at `http://localhost:5000` or send requests with curl or Postman.
Observe what actually happens — don't just reason about it.

```bash
# Example: log an activity for nova (id=1) on Hollow Knight (id=1)
curl -X POST http://localhost:5000/activities \
  -H "Content-Type: application/json" \
  -d '{"user_id": 1, "game_id": 1, "action": "started"}'
```

**3. Query the database directly**
Open `gamehub.db` with a SQLite tool and inspect the actual rows.

```bash
sqlite3 gamehub.db
.tables
SELECT COUNT(*) FROM notifications;
SELECT * FROM notifications WHERE user_id = 1;
```

Or use a GUI: **DB Browser for SQLite** (free, recommended).

---

## Suggested approach

| Phase | Questions | What you are doing |
|-------|-----------|-------------------|
| Read first | 1, 4, 8, 10 | Understand the code before touching anything |
| Then run it | 3, 6, 9    | Observe actual behaviour                     |
| Then break it | 2, 5, 7  | Try things, hit walls, reason about why      |

---

## Questions

**1.** When a user logs a new activity, how many database tables are written to?
List them and explain why each one is affected.
the activities table and the notifications table,
activities is affected because it needs to register the activity and the notification is affected because it is needed to notify your friend about what you did 

---

**2.** You call `DELETE FROM users WHERE id = 3` directly in SQLite.
What happens, and why? What would you need to do instead?
the commmand succeeded and deleted user id 3 , it shows the foreign key enforcement was not enabled, since normally it would fail with a foreing key constraint error 

---

**3.** User `nova` changes her username to `nova_2`.
She then checks her friends' notification feeds.
What do they see — the old name or the new one? Why?
they see the new username the notification field uses thhe variable username so it changes when changed 

---

**4.** Trace the full journey of a `POST /activities` request.
Starting from the HTTP call, list every operation that happens before the response is returned.
inserting into the activities table 
then pulling the user that created that activity then we get their username and then we find what game they did it on
after doing all that we can find the friends of this user and formulate a notification for those friends and we describe who did it what triggered it and in what game 

---

**5.** `pixel_queen` opts out of activity tracking.
A teammate adds an `opted_out` boolean column to the `users` table and updates the `POST /activities` API route to check it.
Is the feature fully implemented? What did they miss?
no only the api route was updated there's also seperate html routes that create activities and notification directly 

---

**6.** How many rows are created in the database when `nova` logs one activity, given the current seed data?
Show your working.
one for the activities table to insert her activity and then three in the notifications table to send her three friends notifications to know what her activity is 

---

**7.** You need to delete `maya_r`.
In what order must you delete rows across the tables, and why does the order matter?
notifications
activities
user_games
friends
users
we go in order of what variable is being used in each one so no one refernces notfications and it refences multiples ones below it so it's the first one to go, and then the same goes for activities as it needs the games friends and the users, and then so on and so forth
---

**8.** The `notifications` table has a foreign key pointing to `activities`.
What happens if you try to delete an activity that has notifications attached to it?

sqlite will reject the deletion because there is no cascade

---

**9.** A bug is found in the game catalog — wrong genre for one game.
You fix it and restart the app to ship the change.
What else just went down, and for how long?
everything being hosted goes down too, and it stays down for the duration of the restart, the things that were in example are :
users, activities, notifications, API routes

---

**10.** A teammate says: *"let's just move the notification logic into its own function in `app.py`"*.
Does that solve the problem described in Task 4?
What is the actual architectural issue?
no it doesn't, that only improves code origanization the real issue is that the route mixes multiple responsiblities in one place 