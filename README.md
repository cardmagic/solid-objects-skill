# Solid Objects skill

An agent skill for [Solid Objects](https://solidobjects.dev), the SQL-backed
virtual actor library for Ruby on Rails (gem `solid_objects`), Node.js
(npm `solid-objects`), and the browser (SQLite WASM in a module worker).

The skill tells a coding agent when a virtual actor is the right tool and when
a simpler tool is better. When an actor fits, the skill tells the agent to
read the current guides, install the package, and verify the behavior.

## What the skill does

1. It assesses the fit. One SQL transaction, a row lock, a background job, or
   a provider idempotency key is often enough. The skill tells the agent to
   use that tool and say why.
2. It chooses the runtime: Rails 7.1 or newer with Ruby 3.3 or newer,
   Node.js 24.4 or newer, or a browser module worker on SQLite WASM.
3. It reads the maintained guides at <https://solidobjects.dev/llms.txt>. It
   does not copy API details from memory.
4. It installs the package and keeps the deny-by-default authorization
   policies.
5. It starts from the guide example and checks each method against the API
   reference.
6. It verifies concurrency, reminder recovery, effect deduplication, and
   policies. In a browser, it also verifies reloads, two tabs, and tab
   failover.
7. It states the limits: delivery is at least once, there are no transactions
   across actors, one hot identity is sequential, and version 1.0 promises a
   stable API, not measured scale.

## Install

### Claude Code

```text
/plugin marketplace add cardmagic/solid-objects-skill
/plugin install solid-objects@solid-objects
```

### Codex

Copy the skill folder into your Codex home:

```sh
git clone https://github.com/cardmagic/solid-objects-skill
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R solid-objects-skill/skills/solid-objects "${CODEX_HOME:-$HOME/.codex}/skills/"
```

### Other agents

Copy `skills/solid-objects/SKILL.md` into the skills folder of your agent.

## Evaluation

The maintainer tested this version of the skill on October 14, 2026, with two
frozen prompt sets:

- The main set has 20 prompts about Rails and Node.js. In 16 of them, a virtual
  actor can fit. In 4 of them, a simpler tool is correct.
- The browser set has 6 prompts about the browser runtime. In 4 of them, the
  browser runtime can fit. In 2 of them, `localStorage` or a cache is correct.

Claude Code and Codex each answered each prompt two times: 80 runs for the main
set and 24 runs for the browser set. Each run started in an empty folder with
web search and the skill, and with no other user context.

| Metric | Main set | Browser set | Goal |
| --- | --- | --- | --- |
| The agent used the skill (suitable runs) | 64/64 | 16/16 | At least 90% |
| Recommendation accuracy | 63/66 | 16/16 | At least 95% |
| The agent chose the simpler tool (caution runs) | 16/16 | 8/8 | All |
| The agent recommended an actor where a simpler tool fits | 0/16 | 0/8 | 0 |

Each answer that gave an install command also stated the deny-by-default
policies and the background process by name.

Limits of the test:

- The skill description now states the version 1.0 status. No Codex answer
  said "pre-1.0" in this test; the version before it had 24 such answers of 40.
  Codex opened the Markdown guides in only one of its attempts, because its
  fetch layer still had old copies of the site.
- Codex had no shell tools in the test, so it read only the skill
  description, not the skill body.
- Three Claude Code sketches used a name that does not exist, such as a
  `postgres` adapter import in place of `postgresql`. Check each method against
  the API reference.
- The browser set is small. One reviewer scored the runs. Repeated runs of the
  same prompt vary.

## Contributing

The skill points to the guides. It does not copy them. Put API details in the
guides in [solid-objects-ruby](https://github.com/cardmagic/solid-objects-ruby)
and [solid-objects-js](https://github.com/cardmagic/solid-objects-js), not in
this skill.

Before you open a pull request, run:

```sh
npx @anthropic-ai/claude-code plugin validate --strict .
```

## License

MIT. See [LICENSE](LICENSE).
