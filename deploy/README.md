# Internal privacy-hardened build

English | 内部使用版本，收敛所有外发隐私数据。

This directory carries the overlay that turns a stock `dsh` launch into the internal privacy-hardened build. It does not fork the product: it is a single Cordis patch overlay applied on top of any shipped profile.

## Use

```sh
dsh --profile web --patch deploy/internal-privacy.patch.yml
# or headless / sdk / acp — any profile that stacks dsh-base:
dsh --profile headless --patch deploy/internal-privacy.patch.yml "task"
```

To make it permanent for one installation, copy the file into the profile's own layer at `$DSH_HOME/profiles/<name>/cordis.patch.yml` (append the same rows; the profile layer applies after every bundle).

## What it disables

| Channel | Row patched | Effect |
|---|---|---|
| `dsh_session_log` session-log suffix on model requests | `session-log-deepseek` | `enabled: false` — no cwd, prompts, conversation, tool arguments/results leave with requests (default was on) |
| `dsh_plugin_packages` plugin inventory on model requests | `plugin-package-inventory-deepseek` | `enabled: false` |
| OTel session telemetry upload | `session-telemetry-otel` | `mode: DISABLED` — even explicit feedback is not uploaded; no collector URL |
| `x-deepseek-harness-user-id` anonymous install UUID header | `llm-deepseek`, `llm-deepseek-account` | `sendAnonymousUserId: false` (new config field, default `true` upstream) |
| Files API image retention | `llm-deepseek`, `llm-deepseek-account` | `fileExpiresAfterSeconds: 3600` (provider minimum, was 7 days) |
| DeepSeek Search / Fetch API web providers | `web-search-deepseek`, `web-deepseek-api` | disabled; `web.fetchProvider: http` keeps local anonymous fetch |

Remaining identifiers on the wire: the `user-agent` header still carries `deepseek-harness/<version>` plus `dsh-profile/<profile>` (set by the launcher via `DSH_USER_AGENT_SUFFIX`; unset it in the launcher environment for a fully neutral agent string), and `x-deepseek-harness-session-id` still carries the request's session id whenever a Session is live.

Remaining local-only channels: browser→host attachment upload (`/api/session/uploadFileBinary`, never leaves the machine), and the session log itself, which stays in `$DSH_HOME` on disk.

## Upstream delta

One code change versus upstream master: `sendAnonymousUserId` config field on the DeepSeek adapter (`packages/llm/llm-deepseek`). Everything else is configuration only and survives upstream rebases as long as the row ids above exist.
