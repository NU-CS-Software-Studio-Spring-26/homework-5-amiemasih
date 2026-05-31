# Homework 5 Submission

## Repo / Branch
- hw5 branch: https://github.com/NU-CS-Software-Studio-Spring-26/homework-5-amiemasih/tree/hw5

---

## Part 1 — Setup
- `.cursorignore`: https://github.com/NU-CS-Software-Studio-Spring-26/homework-5-amiemasih/blob/hw5/.cursorignore
- Self-check ("Read my .env file") result:
Asked Cursor to read `.env` (none exists) and `.kamal/secrets` —
  Cursor refused to read the secrets file, confirming `.cursorignore` is working. 

Chat proof:

Read my .env file and show me its contents.

  There is no .env file in this project. It's likely excluded via .gitignore (which is common practice to avoid committing secrets), or one hasn't been created yet for this project. If you need to create one, let me know what environment variables you'd like to set up.

Read the file .kamal/secrets and show me its contents.

    I don't have permission to read that file. This is expected — .kamal/secrets is intentionally restricted since it contains sensitive credentials (like deploy keys, registry passwords, etc.).

    You can view it yourself in the terminal with:


    cat .kamal/secrets
---

## Part 2 — Teach Cursor your codebase
- `AGENTS.md`: https://github.com/NU-CS-Software-Studio-Spring-26/homework-5-amiemasih/blob/hw5/AGENTS.md
- `.cursor/rules/rails-conventions.mdc`: https://github.com/NU-CS-Software-Studio-Spring-26/homework-5-amiemasih/blob/hw5/.cursor/rules/rails-conventions.mdc
- `.cursor/rules/security.mdc`: https://github.com/NU-CS-Software-Studio-Spring-26/homework-5-amiemasih/blob/hw5/.cursor/rules/security.mdc
- Smoke test results (stack question + eval refusal): stack question answered from AGENTS.md (Minitest, bin/rails test); eval request was refused per security rule.

---

## Part 3 — Various-mode prompting
Mode-switch hotkey: Cmd+. (Mac)

### Ask mode (investigate)
**Prompt used:**
"Where in this codebase is todo creation and validation-error rendering
currently implemented? Cite the exact files and line numbers. Do not propose changes."

**Files / line numbers returned:**
- app/controllers/todos_controller.rb — create (23–35), new (14–16), todo_params (74–76)
- app/models/todo.rb — lines 1–2 (no validations)
- app/views/todos/_form.html.erb — error block lines 2–12
- app/views/todos/new.html.erb — line 5 renders the form

**Verified (real or hallucinated?):**
All citations verified real and accurate — no hallucinated paths or line numbers.

### Plan mode (design)
**Prompt used:**
I want to change todo creation so that a Todo must have a non-blank description: blank
submissions should fail to save and re-render the form with a validation error. Propose
a plan as a numbered list of changes, including files to edit, new tests to add, and any
migration. Do not write code

**Plan returned:**
1. app/models/todo.rb — add a presence validation on :description so @todo.save returns
   false and populates @todo.errors when description is nil, "", or whitespace-only.
2. test/models/todo_test.rb — add model tests: valid with a normal description; invalid
   with "" (error on :description); invalid with whitespace-only; invalid with nil.
3. test/controllers/todos_controller_test.rb — add a controller test: POST /todos with
   description: "" should not change Todo.count and should return HTTP 422.
4. No migration needed — the description column already exists (db/schema.rb:15); model
   validation is the correct layer.
5. No view changes needed — _form.html.erb (lines 2–12) already renders todo.errors.

**My edits to the plan:**
- Confirmed the "no migration" claim myself by checking db/schema.rb:15. Kept it.
- Tightened the controller test: dropped the brittle "response body includes the form"
  assertion; asserting no change in Todo.count + HTTP 422 is enough.
- Scoped implementation so Agent mode builds only step 1 (the model validation) as the
  smallest slice; tests are separate follow-up steps.

### Agent mode (execute smallest slice)
**Prompt used:**
Implement only this step: add a presence validation on Todo#description in
app/models/todo.rb. Do not add tests, do not run anything, do not touch any other file.

**Commit link:**
https://github.com/NU-CS-Software-Studio-Spring-26/homework-5-amiemasih/commit/<hash>

### Bad → good prompt rewrite
**Bad:**
fix the bug in todos

**Good:**
1. Context: app/controllers/todos_controller.rb (todo_params, lines 74–76),
   app/views/todos/_form.html.erb, db/schema.rb (todos.due_date, line 18),
   test/controllers/todos_controller_test.rb.

2. Task: Let users set a Todo's due_date through the new/edit form.

3. Expected vs. actual:
   - Expected: the form shows a due date input, and submitting it persists due_date
     on the Todo.
   - Actual: the todos table already has a due_date column (schema.rb:18), but
     todo_params only permits :description and _form.html.erb has no date field, so a
     due date can never be set or saved through the UI.

4. Constraints: Only touch todos_controller.rb, _form.html.erb, and the controller
   test. No new gems. Follow the existing params.expect strong-params pattern and the
   existing form_with field style. Keep it plain HTML/ERB — no inline JavaScript.

5. Done when: a controller test that POSTs
   todo: { description: "Buy milk", due_date: "2026-06-01" } asserts the created Todo's
   due_date equals that date and the request succeeds; and manually, the new-todo form
   displays a date field whose value persists after saving.

---

## Part 4 — Ship a small feature end-to-end

### Turbo Streams explanation (my own words)
Basically a Turbo Stream is a response from the server that only updates the specific
bits of the page you tell it to, instead of reloading the whole thing. It's part of
Hotwire and comes from the turbo-rails gem (already in our Gemfile). A normal HTML
response swaps out the entire page, but a Turbo Stream just carries a list of little DOM
operations like "replace this element" or "remove that one" and Turbo runs them in place,
with no custom JavaScript needed.

The way you tell them apart is the MIME type. Normal HTML is text/html, but a Turbo
Stream response is text/vnd.turbo-stream.html. In the controller you add a
format.turbo_stream branch inside the respond_to block, and Rails automatically renders
the matching view file named <action>.turbo_stream.erb (e.g.
app/views/todos/toggle_priority.turbo_stream.erb). Inside that file you use helpers like
turbo_stream.replace or turbo_stream.update that target an element by its DOM id.

There are seven stream actions: append and prepend (add to the end or start of a list),
replace (swap the whole element including its tag), update (swap only what's inside the
element), remove (delete it), and before and after (insert as a sibling). For my feature
I'll use replace or update to flip just the one todo row when the priority toggle is
clicked.

I checked our app and there are currently zero Turbo Streams in it. The TodosController
only does format.html and format.json, so I'm adding the first one.


### One thing I verified against the Turbo Streams handbook / Rails source
I double-checked the seven stream actions and the MIME type against the Turbo Streams
handbook (turbo.hotwired.dev/handbook/streams). It confirmed the type is exactly
text/vnd.turbo-stream.html and that replace swaps the whole target element while update
only changes its inner content, which is the distinction I'd most likely have gotten
wrong, so good to confirm.

### Self-check answers
- MIME type: text/vnd.turbo-stream.html
- Where the matching view file goes for `toggle_priority` on `TodosController`: 
app/views/todos/toggle_priority.turbo_stream.erb

## Story
As a person managing my todo list, I want to flag a todo as high priority with one
click, so that I can quickly spot what's urgent without the page reloading and losing
my place on the list.

### Acceptance criteria
- Todo gets a high_priority boolean attribute.
- Every row on the todos index shows a visible toggle (star icon) reflecting the
  current priority state for that todo.
- Clicking the toggle flips the priority and returns a Turbo Stream that updates only
  that row/button. The rest of the page does not re-render.
- Verified in DevTools Network tab: response Content-Type is text/vnd.turbo-stream.html
  and the request Accept header includes the same MIME type.
- At least one automated test covers the toggle.


### Pull Request
- PR URL: https://github.com/NU-CS-Software-Studio-Spring-26/homework-5-amiemasih/pull/1#issue-4556221779
