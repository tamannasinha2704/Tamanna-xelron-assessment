# Part 4: Technical Communication

## Task 4.1: Scenario Response

**Reviewer's question:**
> "Why did you choose this specific PR over the others? What made it comprehensible to you, and what challenges do you anticipate in implementing it?"

---

**Response:**

I chose PR #3877 (Web Readonly Mode) over the other beets PRs for a combination of practical and technical reasons.

The most immediate reason is that the problem statement is immediately understandable without deep domain knowledge. The PR description itself tells you the full story in two sentences: the web plugin was allowing write operations by default, and that was a security risk. I did not need to understand music metadata formats, MusicBrainz API behaviour, or beets' import pipeline to grasp what was being changed. That clarity was a signal that I could analyse it accurately.

Compared to the other PRs I reviewed — for example PR #3279 (parentwork plugin), which required understanding how MusicBrainz organises musical work hierarchies — the readonly PR operates at a level I am already familiar with: HTTP verbs, configuration defaults, and conditional request handling. These are concepts I have encountered in basic web development, and they map directly to what the code changes.

My technical background is still early — I have worked with Python for less than a year, I have built small Flask applications, and I understand how configuration files work. That combination is exactly what this PR requires: knowing how to read a config value, how to pass it into a Flask app, and how to return an HTTP error response. None of those steps requires advanced Python or deep framework knowledge.

The challenges I anticipate are around the two-config-system interaction. Beets uses its own configuration library (`confuse`) and Flask has its own `app.config` dictionary. Making sure the value from beets' config correctly populates Flask's config at the right time — before any request is handled — is the most likely place for a subtle bug. If I set `app.config['READONLY']` too early or in the wrong place in the plugin lifecycle, the default or user-set value might not be applied correctly.

To overcome this, I would first read the existing `WebPlugin` class carefully to understand where the Flask app is created and where other config values are loaded. I would also look at the test setup to see how beets config is injected during tests, so I can write reliable tests for both the `readonly: true` and `readonly: false` paths independently.

---

*I declare that all written content in this assessment is my own work, created without the use of AI language models or automated writing tools. All technical analysis and documentation reflects my personal understanding and has been written in my own words.*
