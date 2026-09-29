# ecs-logging-audit — free Claude Code skill

Audit any app's logging before an ELK rollout, in one command. This skill
inventories every log call, classifies each one (keep / enrich / drop), and
flags secrets or personal data being written to logs. Read-only: it changes
nothing.

## Why

Getting an existing app's logs into Elasticsearch properly starts with
knowing what you have. Most codebases fail that audit in interesting ways:
`console.log(dto)` calls that dump whatever a client sends, error handlers
that log a bare `err` with no context, and entire modules where failures
never get logged at all.

On the first real app I ran this against, the audit found that the app's
only production error log was dead code: NestJS's default `abortOnError`
exits the process before `bootstrap().catch()` ever runs. Full story:
https://dev.to/erickcm/getting-a-nestjs-apps-logs-into-elasticsearch-without-writing-grok-453m

## Install

Copy `skills/ecs-logging-audit/` into your project's `.claude/skills/`
(or `~/.claude/skills/` for all projects), open Claude Code, and say:

> audit my logging

You get a table like this:

| File:line | Current call | Level | Verdict | Why |
|---|---|---|---|---|
| `src/main.ts:37` | `Logger.error('Failed to start...')` | error | keep | convert format only |
| `key-value.service.ts:26` | `console.error(err)` | error | enrich | no context, bare error |
| `key-value.service.ts:47` | `console.log(updateKeyValueDto)` | — | drop | dumps full request DTO |

plus a summary of risky calls, unlogged modules, and what a full ECS
retrofit would involve for your stack.

## Works with

Any language Claude Code can read. The verdict conventions follow ECS
(Elastic Common Schema) and Elastic Stack 8.x.

## The full version

This is the audit-only edition. The full **ECS Logging Retrofit** skill
does the rest of the job: converts every call to ECS JSON using the official
formatter for your existing logger, rewrites messages so variables become
queryable fields, generates a working `filebeat.ecs.yml` for your runtime,
and verifies real output lines before declaring done. It ships with example
output from a production run.

**$9 → https://erickcmdev.gumroad.com/l/ecs-logging**

## License

MIT for everything in this repo.
