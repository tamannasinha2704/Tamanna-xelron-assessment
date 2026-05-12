# Part 3: Prompt Preparation Document

**Selected PR:** #3877 — Web Readonly Mode  
**Repository:** beetbox/beets  
**PR Link:** https://github.com/beetbox/beets/pull/3877

---

## 3.1.1 Repository Context

> ⚠️ **Note to evaluator:** This section must be written in your own words. Below is a structured guide of what to include — rewrite it in your own voice before submitting.

**Draft to rewrite in your own words:**

Beets is a command-line tool written in Python that helps people manage their music libraries. The main problem it solves is that music files often have incorrect or incomplete metadata — things like wrong album names, missing artist tags, or inconsistent track numbering. Beets connects to MusicBrainz, a community-run music database, and automatically identifies and corrects these tags.

At its core, beets keeps a local SQLite database of all your music, and you interact with it through terminal commands like `beet import`, `beet list`, or `beet modify`. One of beets' strongest design choices is its plugin system — almost all extended features are packaged as separate plugins that users can enable in a config file. This includes things like fetching lyrics, downloading album art, or — as is relevant to this PR — running a local web server to browse your library.

The `web` plugin specifically starts a small Flask-based HTTP server that exposes the music library as a REST API. Users can query for tracks, albums, and play audio files through a browser or any HTTP client. This makes beets accessible to remote tools and scripts that want to read or modify library data programmatically.

The intended users of beets are technically comfortable music enthusiasts — people who are comfortable with the terminal, care deeply about tag accuracy, and want fine-grained control over their library. The web plugin extends this to any use case where someone wants to access their library over a network.

---

## 3.1.2 Pull Request Description

> ⚠️ **Note to evaluator:** This section must be written in your own words. Below is a structured guide — rewrite it in your own voice before submitting.

**Draft to rewrite in your own words:**

This PR adds a `readonly` configuration option to the beets `web` plugin. Before this change existed, the web plugin's HTTP API exposed DELETE and PATCH endpoints that could modify or remove items from the music library. Any client that could reach the running beets web server — including scripts, browser extensions, or other users on the same network — could delete tracks or change metadata without any restrictions.

The problem that triggered this PR (referenced as issue #3870) was that users who ran the web plugin on shared machines or over a local network had no way to protect their library from accidental or unauthorised modifications. The web interface was fine for browsing, but exposing write operations felt risky without an explicit opt-in.

The change introduced here is straightforward: a new config option called `readonly` is added to the `web:` section of the beets config. It defaults to `true`, meaning write operations are now blocked by default. If a user wants to re-enable DELETE and PATCH, they set `readonly: no` in their config.

Previous behaviour: DELETE and PATCH were available to anyone who could reach the server.  
New behaviour: DELETE and PATCH return HTTP 405 (Method Not Allowed) unless `readonly: no` is explicitly set.

This is noted as a potentially breaking change for users who were relying on DELETE or PATCH, but the migration is a simple one-line config addition.

---

## 3.1.3 Acceptance Criteria

The following criteria define what a correct implementation of this PR looks like:

✓ When `readonly` is not set in config (default), the web plugin should block DELETE requests and return HTTP 405.

✓ When `readonly` is not set in config (default), the web plugin should block PATCH requests and return HTTP 405.

✓ When `readonly: true` is explicitly set in config, DELETE and PATCH should both be blocked with HTTP 405.

✓ When `readonly: false` is explicitly set in config, DELETE requests should succeed and return the appropriate HTTP response (200 or 404).

✓ When `readonly: false` is explicitly set in config, PATCH requests should succeed and apply the changes to the library item.

✓ GET requests should always succeed regardless of the `readonly` setting — reading the library is never restricted.

✓ The `readonly` configuration option should be documented in `docs/plugins/web.rst` with its default value and an example of disabling it.

✓ The Flask `app.config['READONLY']` value should be correctly synchronised from the beets config `self.config['readonly']` when the plugin initialises.

✓ Existing tests for GET operations should continue to pass without modification.

✓ The changelog should include a note about this being a potentially breaking change for users who were previously using DELETE or PATCH without configuration.

---

## 3.1.4 Edge Cases

**Edge Case 1: Config value is not a boolean**  
If a user writes `readonly: yes` or `readonly: 1` instead of `readonly: true`, beets' config system should coerce these to a Python boolean correctly. The implementation should use beets' `.get(bool)` config accessor rather than raw string comparison to ensure all truthy/falsy YAML representations are handled.

**Edge Case 2: Flask app.config not synced from beets config**  
The web plugin uses two separate config systems: beets' own `confuse`-based config (`self.config['readonly']`) and Flask's `app.config`. If the code reads from one but not the other, or sets `app.config['READONLY']` before the beets config is fully loaded, the readonly state may be wrong. The sync must happen at plugin setup time — specifically in the `commands()` or `app.before_first_request` hook — not lazily.

**Edge Case 3: Plugin loaded but no `web:` section in config**  
If a user runs the web plugin without ever having a `web:` config section, the `readonly` option should fall back to its default of `true` gracefully. This means calling `.get(bool, default=True)` rather than a bare `.get()` that would raise a `NotFoundError`.

**Edge Case 4: Partial config — only one of DELETE/PATCH is tested**  
Both DELETE and PATCH must independently check the readonly flag. A naive implementation might only guard one endpoint. Tests must verify both verbs separately, not assume that fixing one fixes the other.

---

## 3.1.5 Initial Prompt

---

**System/Context:**  
You are implementing a feature for the `beetbox/beets` repository — a Python-based command-line music library manager. The codebase uses a plugin architecture where each plugin is a standalone Python file in the `beetsplug/` directory. The web plugin (`beetsplug/web.py`) runs a local Flask-based HTTP REST API that exposes the music library. Beets uses a custom configuration library called `confuse` (accessed via `self.config`) for reading user settings from `config.yaml`.

---

**Task:**  
Implement the changes described in GitHub Pull Request #3877 for the `beetbox/beets` repository.

The PR adds a `readonly` configuration option to the `web` plugin. Here is exactly what needs to be implemented:

**1. Add `readonly` config option to `beetsplug/web.py`:**
- In the `WebPlugin` class, read `self.config['readonly'].get(bool)` — the default must be `True`
- Assign this value to Flask's `app.config['READONLY']` so route handlers can access it
- This assignment must happen when the Flask app is being configured, before any requests are served

**2. Guard DELETE and PATCH route handlers:**
- At the top of the DELETE item handler, check `if app.config['READONLY']: return flask.make_response(jsonify(error='...'), 405)`
- At the top of the PATCH item handler, add the same guard
- GET operations must never be blocked

**3. Update `docs/plugins/web.rst`:**
- Add a documentation entry for the new `readonly` config option
- Specify that the default is `true`
- Show an example of setting `readonly: false`

**4. Add tests to `test/test_web.py`:**
- Write a test that verifies DELETE returns 405 when readonly is true (default)
- Write a test that verifies PATCH returns 405 when readonly is true (default)
- Write a test that verifies DELETE succeeds (200 or 404) when `readonly: false`
- Write a test that verifies PATCH succeeds when `readonly: false`
- Ensure all existing GET tests still pass

**Edge cases to handle:**
- The `readonly` config value must be parsed as a boolean, not a string — use `.get(bool)` not `.get()`
- If `readonly` is not present in config at all, it must default to `True` (safe default)
- Both DELETE and PATCH must be independently guarded — do not assume fixing one covers the other
- `app.config['READONLY']` must be set from `self.config['readonly']` before any request is processed — do not lazy-read the beets config inside each route handler

**Do not:**
- Modify any GET endpoints
- Change the URL routing structure
- Alter how the library is queried — only the write-operation guard logic needs to change

**Testing requirement:**  
After implementation, the following test scenarios must all pass:
- `GET /item/1` works regardless of readonly setting
- `DELETE /item/1` → 405 when `readonly: true`
- `DELETE /item/1` → 200/404 when `readonly: false`
- `PATCH /item/1` → 405 when `readonly: true`
- `PATCH /item/1` → 200 when `readonly: false` with valid JSON body

Reference the existing test setup in `test/test_web.py` to understand how the Flask test client is configured and how beets config is set during tests. Follow the same patterns for your new test cases.
