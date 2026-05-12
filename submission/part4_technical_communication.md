# Part 4: Technical Communication

### Task 4.1: Scenario Response

I selected PR #3877 (Web Readonly Mode) because it was the easiest PR for me to understand clearly compared to the other pull requests available in the repository.

The main reason I chose this PR is that the problem statement is simple and practical. The PR mainly focuses on adding a readonly mode to the web plugin so users cannot accidentally modify or delete music library data through the web API. Since the issue is related to HTTP requests and configuration settings, it was easier for me to follow the purpose of the change and understand how the implementation works.

Some of the other PRs involved more complex concepts related to music metadata handling, plugin interactions, or advanced shell integrations. Compared to those, this PR had a smaller scope.

My current technical background also made this PR more suitable for me. I have beginner-level experience with Python and APIs, and configuration files. Because of this, I was able to understand concepts like HTTP methods, request handling, config values, and HTTP status codes without too much difficulty.

One challenge I faced while understanding this PR was the Flask configuration system used inside the web plugin. I have only worked with small beginner-level Flask projects before, so understanding how the beets configuration system connects with Flask's `app.config` took some time for me.

At first, I found it confusing that the repository was using two different configuration systems together — the beets config system and Flask configuration. I had difficulty understanding where the readonly value was being loaded and how it became available inside the request handlers.

To overcome these challenges, I would first spend more time reading the existing `web.py` plugin structure and understanding how Flask routes and configuration values are currently implemented. I would also review the existing tests in `test/test_web.py` to understand how requests are tested in the repository and then follow similar patterns while adding tests for readonly mode.

---

### Declaration

"I declare that all written content in this assessment is my own work, created without the use of AI language models or automated writing tools. All technical analysis and documentation reflects my personal understanding and has been written in my own words."
