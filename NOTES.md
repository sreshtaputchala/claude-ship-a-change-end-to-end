# Notes: `PUT /users/:id`

## The plan
The plan added an `updateUser(id, { name, email })` helper to `db/store.js` and a `PUT /users/:id` route in `routes/users.js` that follows the existing GET/POST pattern. Both `name` and `email` are required, because PUT replaces the user. Missing, blank or non-string fields and a malformed email return 400. An unknown id, including a non-numeric one, returns 404 with the same `"User not found"` message as GET. POST was deliberately left unchanged to keep the change focused. I approved the plan as written, without edits.

## Model
I chose **Claude Opus 5.5**. The endpoint itself is small, but shipping it was a multi-step process: planning, building in commits, debugging and fixing. A mistake in any step carries into the next, so I wanted the strongest model for reasoning across all of them:

- **Planning:** the plan had to fit the existing code, not just pass the tests. That meant reusing `getUserById` instead of duplicating it, matching GET's 404 message and POST's 400 message, and deciding up front how PUT should treat blank fields, malformed emails, non-numeric ids and an `id` in the body.
- **Debugging:** some failures weren't in the feature code. PowerShell 5.1 mangled multi-line arguments, so commit messages and PR descriptions failed until the cause was found and worked around with message files. Earlier in the course, the PR also had to go to the course repo rather than my fork.
- **Review and fixing:** the tests were already green when the review found the untrimmed-value bug, which no provided test covers. Catching what tests miss takes careful reasoning about edge cases, and that's why I used the stronger model.

A smaller, faster model could likely write this route. Opus 5.5 was worth it for the planning, debugging and self-review around it.

## Commits
1. **Add updateUser helper to the in-memory store**: the data-access piece on its own, since routes go through the store.
2. **Add PUT /users/:id endpoint with validation and 404**: the route, which is the commit that turns the update-user tests green.
3. **Trim name and email before validating and storing**: a separate fix for what the review found, so its reason stays visible in the history.
4. **Add NOTES.md**: this write-up.

Each commit is one logical change that can be understood from its message alone.

## Review
I sent edge cases to the running app (unknown and non-numeric ids, missing, blank and non-string fields, bad emails, empty and array bodies, and a body trying to change `id`) and checked each response.

- **Caught and fixed:** validation trimmed values to reject blanks, but stored them untrimmed, so `"  Ada  "` was saved with its spaces and `" alan@example.com "` was wrongly rejected. Values are now trimmed once and used for both validation and storage.
- **Confirmed fine:** every 400 and 404 path; `id` in the body is ignored; nothing crashes.
- **Accepted:** `/users/1.0` resolves to user 1, and an unknown id with an invalid body returns 400 (validation runs first). GET behaves the same way.
- **Out of scope:** malformed JSON gets Express's default HTML 400 page. That comes from `express.json()` and affects every route, so it belongs in a separate change.
