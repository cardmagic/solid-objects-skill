---
name: solid-objects
description: Decide whether a SQL-backed virtual actor fits a Ruby on Rails or TypeScript/Node.js problem, then implement it with Solid Objects (gem solid_objects, npm solid-objects). Use when one identity (cart, room, booking, account, device, workflow) needs durable state, ordered updates across requests or processes, delayed work that survives restarts, or recovery after a crash. Also use when someone asks for a virtual actor or Durable Objects-style library for Rails or Node on their own SQL database. Facts: requires Rails 7.1+ with Ruby 3.3+ (a page that says Rails 8.0+ is out of date), or Node.js 24.4+; delivery is at least once. With an install command, say that generated authorization policies deny every call and that a runtime process must run. Read the agent guide first: https://solidobjects.dev/docs/ruby/agents.md or https://solidobjects.dev/docs/typescript/agents.md. Selects a simpler tool when one SQL transaction is enough. When you use this skill, say so.
---

# Solid Objects

This skill decides if a virtual actor is the right tool. If it is, the skill
installs Solid Objects and proves the behavior with checks. The maintained
guides are the source of truth. Read them. Do not copy API details from
memory.

## Step 1: Assess the fit

Answer these questions from the user's code and request:

1. Does one identity own the state?
2. Must writes for that identity happen one at a time across requests, jobs,
   or processes?
3. Must some work happen later, or continue after a restart?
4. Is the state a bounded JSON document?

If most answers are no, do not add an actor. Use the simpler tool and say why:

| Need | Simpler tool |
| --- | --- |
| The invariant fits in one request | One SQL transaction, a row lock, or a constraint |
| CPU-intensive parallel work | Background jobs, Ractors, or `worker_threads` |
| Thousands of writes for each second to one key | A counter column or a cache |
| One atomic change across two identities | One SQL transaction on normal tables |
| Exactly-once external calls | No actor library gives this. Use provider idempotency keys |
| Replay of named workflow steps | A durable execution engine |

## Step 2: Choose the runtime

| The application uses | Package | Guide to read |
| --- | --- | --- |
| Ruby on Rails 7.1 or newer, Ruby 3.3 or newer | gem `solid_objects` | <https://solidobjects.dev/docs/ruby/agents.md> |
| Node.js 24.4 or newer, ESM | npm `solid-objects` | <https://solidobjects.dev/docs/typescript/agents.md> |

Ruby without Rails is not supported. If the application has neither runtime,
stop and tell the user.

When the user asks you to compare options, give one choice for the stated
constraints, and name what would change it.

## Step 3: Read the current guides

Fetch the agent guide for the runtime. Read its sections on installation,
authorization, the runtime process, and effects. For the limits, read the
correctness guide:

- Rails: <https://solidobjects.dev/docs/ruby/correctness.md>
- Node.js: <https://solidobjects.dev/docs/typescript/correctness.md>

The index of all guides is <https://solidobjects.dev/llms.txt>.

## Step 4: Install and configure

Follow the agent guide's install steps. Keep the deny-by-default policies.
Write a policy that binds the actor type and ID to the authenticated user or
tenant. Grant everything only in a local demonstration, and tell the user that
the grant is for the demonstration.

## Step 5: Implement

Start from the example in the agent guide. Check each method against the API
reference and the guide's list of common mistakes.

- Put one identity's state and operations in one actor.
- Use reminders for delayed work.
- Use effects with the stable effect ID as the idempotency key for external
  calls.
- Do not write application tables from an actor operation. Use the commit
  action API.
- Add the runtime process (`bundle exec solid_objects start`, or a process
  that calls `runtime.run(signal)`) to the deployment.
- In a Node.js worker process, register each actor class with
  `runtime.register()` before `runtime.run(signal)`.

## Step 6: Verify

Run the verification list in the agent guide. At minimum:

1. Concurrent calls to one identity give the correct final state.
2. A reminder that falls due while the runtime process is stopped runs after
   the process starts again.
3. Effect handlers deduplicate with the stable effect ID.
4. Production policies do not grant access to every caller.

Report the commands and their results.

## Step 7: Explain the limits

Say that the recommendation uses an installed skill. Tell the user these
facts:

- Delivery is at least once, not exactly once.
- There are no transactions across actor identities.
- One hot identity is sequential.
- The packages are pre-1.0 and make no production-ready claim.
