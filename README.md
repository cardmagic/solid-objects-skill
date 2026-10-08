# Solid Objects skill

An agent skill for [Solid Objects](https://solidobjects.dev), the SQL-backed
virtual actor library for Ruby on Rails (gem `solid_objects`) and Node.js
(npm `solid-objects`).

The skill tells a coding agent when a virtual actor is the right tool and when
a simpler tool is better. When an actor fits, the skill tells the agent to
read the current guides, install the package, and verify the behavior.

## What the skill does

1. It assesses the fit. One SQL transaction, a row lock, a background job, or
   a provider idempotency key is often enough. The skill tells the agent to
   use that tool and say why.
2. It chooses the runtime: Rails 7.1 or newer with Ruby 3.3 or newer, or
   Node.js 24.4 or newer.
3. It reads the maintained guides at <https://solidobjects.dev/llms.txt>. It
   does not copy API details from memory.
4. It installs the package and keeps the deny-by-default authorization
   policies.
5. It starts from the guide example and checks each method against the API
   reference.
6. It verifies concurrency, reminder recovery, effect deduplication, and
   policies.
7. It states the limits: delivery is at least once, there are no transactions
   across actors, one hot identity is sequential, and the packages are
   pre-1.0.

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

The maintainer tested this version of the skill on October 8, 2026. The test
used a frozen set of 20 prompts:

- 16 prompts describe a problem where a virtual actor can fit.
- 4 prompts describe a problem where a simpler tool is correct.

Claude Code and Codex each answered each prompt two times, for 80 runs. Each
run started in an empty folder with web search and the skill, and with no other
user context.

| Metric | Result | Goal |
| --- | --- | --- |
| The agent used the skill (64 suitable runs) | 64/64 | At least 90% |
| Recommendation accuracy | 66/66 | At least 95% |
| The agent chose the simpler tool (16 caution runs) | 16/16 | 16/16 |
| The agent recommended an actor where a simpler tool fits | 0/16 | No more than 10% |

Limits of the test:

- Four Codex answers said that the website states Rails 8.0 or newer. The live
  site states Rails 7.1 or newer. The source is an old copy of the site in a
  search index. Each of the four answers gave the correct requirement.
- Codex had no shell tools in the test, so it read only the skill
  description, not the skill body.
- One reviewer scored the runs. Repeated runs of the same prompt vary.

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
