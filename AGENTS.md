# AGENTS.md

One-page brief for AI coding agents working in this repo. Everything here is verified
against the actual code (Gemfile, schema, controllers), not assumed.

## Stack
- **Rails 8.1.3** on **Ruby 3.4.1**.
- **Database:** SQLite3 (`sqlite3` gem, >= 2.1). Asset pipeline is **Propshaft**.
- **View layer:** ERB + **Hotwire** (`turbo-rails`, `stimulus-rails`) with **importmap-rails** for JS. JSON responses use **jbuilder**. No CSS framework — plain CSS in `app/assets/stylesheets/`.
- **Test framework:** Minitest (Rails default) under `test/`; system tests use Capybara + Selenium. **Not RSpec.**
- **Background jobs / infra:** Solid Queue (Active Job), Solid Cache, Solid Cable. Deploy via Kamal/Docker (Puma + Thruster). Lint: rubocop-rails-omakase; security scan: Brakeman.

## Commands
- Setup: `bin/setup`
- Run dev server: `bin/dev` (or `bin/rails server`)
- Run tests: `bin/rails test` (unit/controller) and `bin/rails test:system` (system)
- Run a single test file: `bin/rails test test/controllers/todos_controller_test.rb`
- Lint: `bin/rubocop`
- Security scan: `bin/brakeman`
- DB migrate: `bin/rails db:migrate`

## Conventions
- Standard Rails MVC + naming. The single model is `Todo` (`app/models/todo.rb`); attributes: `description`, `due_date`.
- **Controllers respond with `respond_to` blocks** handling `format.html` and `format.json` (see `app/controllers/todos_controller.rb`). JSON is rendered through jbuilder views (`*.json.jbuilder`). There are no Turbo Stream responses yet.
- **Strong params** use the Rails 8 `params.expect(...)` style, e.g. `params.expect(todo: [:description])`.
- **Shared partials** live alongside their views in `app/views/todos/` (e.g. `_form.html.erb`, `_todo.html.erb`).
- **No authentication or authorization exists** in this app today. Do not assume Devise/Pundit/etc.; if a feature needs authz, flag it rather than inventing a framework.
- Generate boilerplate with `bin/rails generate` rather than hand-writing it; keep migrations reversible.

## Don'ts
- **No new gems** without explicit approval.
- **No inline JavaScript in ERB** — use Stimulus controllers under `app/javascript/controllers/`.
- **Do not disable CSRF** (`skip_before_action :verify_authenticity_token`) or mass-assignment protection.
- **Do not switch the test framework to RSpec** — this project uses Minitest.
- **Do not add Bootstrap or any CSS framework** — styling is plain CSS via Propshaft.
- **Do not seed real or fake user data** anywhere except `db/seeds.rb`.
