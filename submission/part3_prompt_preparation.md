# Part 3: Prompt Preparation

## Selected Pull Request

- **Repository:** beetbox/beets  
- **Selected PR:** #3877 — Web Readonly Mode  
- **Repository Link:** https://github.com/beetbox/beets  
- **PR Link:** https://github.com/beetbox/beets/pull/3877

---

# 3.1.1 Repository Context

Beets is an open-source Python application used for organising and managing music libraries. It works mainly through the command line and helps users automatically correct music metadata such as artist names, album names, song titles, genres, and track numbers.

The repository mainly focuses on solving problems related to music library organisation and metadata management. Many music collections contain incorrect or inconsistent tags, and manually fixing them can take a long time. Beets solves this by connecting with online databases like MusicBrainz to identify songs and fetch accurate metadata automatically.

Another important feature of beets is its plugin system. Users can enable plugins to add extra functionality like lyrics support, album art downloads, duplicate detection, replay gain calculation, and web access. The repository is designed in a modular way, so plugins can add new features without changing the main application heavily.

The `web` plugin is relevant for this pull request because it creates a Flask-based web server that exposes the music library through an HTTP API. Users can browse their library, access music information, and interact with the library from browsers or scripts.

The main users of beets are music enthusiasts, collectors, and users who manage large music libraries and want better automation and organisation.

---

# 3.1.2 Pull Request Description

This pull request adds a new `readonly` configuration option to the beets web plugin.

Before this change, the web plugin allowed users to send DELETE and PATCH requests through the HTTP API. These requests could modify or delete music library data directly. Any user or device with access to the running web server could potentially make changes to the library.

This behaviour created a security and safety concern, especially for users running the web plugin on shared systems or local networks. Some users only wanted read-only access for browsing their music collection and did not want editing features enabled by default.

To solve this issue, the PR introduces a `readonly` setting inside the web plugin configuration. The setting is enabled by default.

When readonly mode is active:
- DELETE requests are blocked
- PATCH requests are blocked
- The server returns HTTP 405 (Method Not Allowed)

If users want editing support, they can manually disable readonly mode in the config file:

```yaml
web:
  readonly: false
```

Previous behaviour:
- DELETE and PATCH requests worked normally

New behaviour:
- DELETE and PATCH requests are blocked unless readonly mode is disabled manually

This change improves safety while still allowing advanced users to enable write operations if needed.

---

# 3.1.3 Acceptance Criteria

✓ When the `readonly` setting is missing from the config file, the system should use `true` as the default value.

✓ When readonly mode is enabled, DELETE requests should return HTTP 405.

✓ When readonly mode is enabled, PATCH requests should return HTTP 405.

✓ When readonly mode is disabled using `readonly: false`, DELETE requests should work normally.

✓ When readonly mode is disabled using `readonly: false`, PATCH requests should successfully update library items.

✓ GET requests should continue working regardless of readonly mode.

✓ The `readonly` option should be added to the plugin documentation with an example configuration.

✓ The implementation should correctly update Flask's `app.config['READONLY']` value from the beets configuration.

✓ Tests should be added for both readonly enabled and disabled behaviour.

✓ Existing GET request tests should continue passing after implementation.

✓ The changelog should include a note explaining the new readonly behaviour.

---

# 3.1.4 Edge Cases

## Edge Case 1 — Missing Config Value

If the user does not add the `readonly` setting in the config file, the system should automatically use `true` as the default value instead of failing.

---

## Edge Case 2 — Incorrect Boolean Handling

The config system should correctly handle values like `yes`, `no`, `true`, and `false` and convert them into proper boolean values.

---

## Edge Case 3 — Only One Route Protected

The implementation should make sure that both DELETE and PATCH requests are checked separately. Protecting only one route could create inconsistent behaviour.

---

## Edge Case 4 — GET Requests Accidentally Blocked

Readonly mode should only block write operations. GET requests for viewing library data should continue working normally.

---

## Edge Case 5 — Flask Config Not Updated Properly

The readonly value from the beets config should correctly update Flask's `app.config['READONLY']` before any request is processed.

---

# 3.1.5 Initial Prompt

You are working on the `beetbox/beets` repository, which is a Python command-line music library manager. The repository uses a plugin-based architecture, and the `web` plugin provides a Flask-based HTTP API for accessing the music library.

Implement the changes described in PR #3877 related to readonly mode for the web plugin.

## Main Goal

Add a new `readonly` configuration option that prevents users from modifying library data through the web API unless readonly mode is manually disabled.

---

## Requirements

### 1. Add Readonly Configuration Support

Add a new config option called `readonly` inside the web plugin configuration.

Requirements:
- The default value should be `true`
- Read the value using the beets config system
- Store the value inside Flask's `app.config['READONLY']`

The readonly value should be available before any requests are handled.

---

### 2. Protect DELETE and PATCH Requests

When readonly mode is enabled:
- DELETE requests should return HTTP 405
- PATCH requests should return HTTP 405

GET requests should continue working normally.

If users disable readonly mode using:

```yaml
web:
  readonly: false
```

then DELETE and PATCH requests should work normally again.

---

### 3. Update Documentation

Update the web plugin documentation to include:
- Explanation of the new `readonly` option
- Default value information
- Example configuration using `readonly: false`

---

### 4. Add Tests

Add tests for:
- DELETE blocked when readonly mode is enabled
- PATCH blocked when readonly mode is enabled
- DELETE allowed when readonly mode is disabled
- PATCH allowed when readonly mode is disabled
- GET requests still working normally

Existing GET request tests should continue passing.

---

## Edge Cases to Consider

- If the config value is missing, readonly should default to `true`
- The config value should be handled as a boolean
- DELETE and PATCH routes must both be protected
- GET requests must not be blocked
- Flask config should be updated before requests are processed

---

## Expected Behaviour

- Readonly mode should prevent modification requests
- Users should still be able to browse the music library normally
- Existing functionality outside the web plugin should remain unchanged
