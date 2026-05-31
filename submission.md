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


### One thing I verified against the Turbo Streams handbook / Rails source


### Self-check answers
- MIME type:
- Where the matching view file goes for `toggle_priority` on `TodosController`:

### Pull Request
- PR URL: <link>
