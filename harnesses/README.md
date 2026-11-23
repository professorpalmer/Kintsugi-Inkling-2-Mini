# Use Inkling 2 Mini with a coding harness

The server works with four common coding harnesses: Claude Code, Codex CLI, Pi and OpenCode. This page gives the
version that we tested, the result, and the minimum settings for each harness. (In the bundle, this page is
`HARNESSES.md`.)

## Before you start

1. Start the server (see README.md). Wait for the line `ready  http://127.0.0.1:8080/v1`.
2. Use these values in the harness:
   - Address: `http://127.0.0.1:8080` (some harnesses need `/v1` at the end; see each harness below).
   - API key: any text, for example `any-key`. The server does not check the key, unless you set
     `INKLING_KEY_FILE`. If you set a key file, use the key from that file.
   - Model name: any text, for example `inkling`. The server has one model.
3. Thinking effort: the launcher default is `high` (the model's own default). A harness that sends its own effort
   (Codex: `model_reasoning_effort`) overrides it for its requests. For fast agent turns, try `medium`.

4. **Send the earlier thinking back.** Inkling 2 Mini needs its own earlier reasoning in the conversation. The
   server returns it as `reasoning_content`; the harness must send that field back with each earlier assistant
   turn. If a harness drops it, users report much worse multi-turn and agent results (we did not measure that
   case), and the server must also read the whole context again on each turn (slow at long context; measured). The four harnesses below send it back in the
   versions we tested, and so does the AppWorld agent we used for our scores. Check this first if a harness or
   your own client gives poor results.

## What we tested

We gave each harness the same 6 small tasks in a small Python repository. The tasks were: read a file, fix a function
and run its test, make a new file, search a word in many files, a task of more than 5 tool calls, and a question about
a file of about 42,000 tokens. Windows 11, RTX 4070 12 GB, our engine (two runs: before and after the repetition guard
was added; the same result in both), launcher defaults: every harness talked to the saved-chat layer (port 8080), which
is what you get when you start the bundle.

| Harness | Version tested | Tasks passed | Server errors, malformed calls, leaked tags | Loops |
|---|---|---|---|---|
| Claude Code | 2.1.330 | 5 of 6 | none | none |
| Codex CLI | 0.171.0 | 5 of 6 | none | none |
| Pi | 0.73.1 | 5 of 6 | none | none |
| OpenCode | 1.19.6 | 5 of 6 | none | none |

The same task failed in all four harnesses: "Which files call legacy_connect( ?". The model lists every file that the
search finds, also the one that defines the function. This is an answer of the model; Unsloth's build of the model
misses the same task in 3 of the 4 harnesses.

In an earlier run (with the 8-bit cache), OpenCode once looped: on Windows, OpenCode runs shell commands in PowerShell, the model wrote bash syntax,
and after the quoting failed it repeated trivial shell commands. This is the model's choice (it puts 96-100 % of its
probability on the shell tool at those turns), not a fault of the server: in 10 more tries of the same task (through the
layer, as above), the task passed 10 times with no loop, as it did with Unsloth's build. On Windows, OpenCode runs shell commands in PowerShell; the model sometimes writes bash syntax there and can get stuck repeating trivial commands.

The server's repetition guard (on by default) is for this case: when the model repeats nearly the same tool call, the
server adds one short note to that request. In replays of that loop, at two stuck points it broke the loop in 6 and 7
of 10 tries, against 0 of 10 without it. It does not stop every loop: if the model still repeats, stop the turn and ask
it to use the edit tool. To turn the guard off, see the bundle README ("Repetition guard").

## Claude Code

Set these variables before you start Claude Code (PowerShell):

```
$env:ANTHROPIC_BASE_URL = "http://127.0.0.1:8080"
$env:ANTHROPIC_API_KEY = "any-key"
$env:ANTHROPIC_DEFAULT_OPUS_MODEL = "inkling"
$env:ANTHROPIC_DEFAULT_SONNET_MODEL = "inkling"
$env:ANTHROPIC_DEFAULT_HAIKU_MODEL = "inkling"
$env:CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC = "1"
claude --model inkling
```

Claude Code uses the Anthropic API (`/v1/messages`). No login is necessary when `ANTHROPIC_API_KEY` is set. Claude
Code can show the note "unrecognized model": you can ignore it.

## Codex CLI

Codex reads `config.toml` in its home folder (`%USERPROFILE%\.codex\config.toml`). Add these lines:

```
model = "inkling"
model_provider = "inkling"

[model_providers.inkling]
name = "Inkling 2 Mini"
base_url = "http://127.0.0.1:8080/v1"
env_key = "INKLING_API_KEY"
wire_api = "responses"
```

Then, in PowerShell:

```
$env:INKLING_API_KEY = "any-key"
codex
```

Codex uses the Responses API (`/v1/responses`). Codex 0.171.0 does not accept `wire_api = "chat"` any more. Codex can
show the note "Model metadata for inkling not found": you can ignore it.

## Pi

Pi reads custom models from `%USERPROFILE%\.pi\agent\models.json`. Put this text in that file:

```
{
  "providers": {
    "inkling": {
      "baseUrl": "http://127.0.0.1:8080/v1",
      "api": "openai-completions",
      "apiKey": "any-key",
      "models": [
        { "id": "inkling", "name": "Inkling 2 Mini (local)", "reasoning": true, "contextWindow": 262144, "maxTokens": 24576 }
      ]
    }
  }
}
```

Then start Pi with:

```
pi --provider inkling --model inkling
```

## OpenCode

OpenCode reads `%USERPROFILE%\.config\opencode\opencode.json`. Put this text in that file:

```
{
  "$schema": "https://opencode.ai/config.json",
  "model": "inkling/inkling",
  "small_model": "inkling/inkling",
  "provider": {
    "inkling": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Inkling 2 Mini (local)",
      "options": { "baseURL": "http://127.0.0.1:8080/v1", "apiKey": "any-key" },
      "models": { "inkling": { "name": "Inkling 2 Mini", "limit": { "context": 262144, "output": 24576 } } }
    }
  }
}
```

Then start OpenCode with `opencode` (the `model` line in the file selects Inkling). For one task without the screen,
use `opencode run -m inkling/inkling "your task"`.

At its first start, OpenCode downloads the package `@ai-sdk/openai-compatible` from npm.

## On Linux

The settings are the same. Use `export NAME=value` in place of `$env:NAME = "value"`, and the files are in your home
folder (`~/.codex/config.toml`, `~/.pi/agent/models.json`, `~/.config/opencode/opencode.json`). Pi needs Node.js 20 or
newer.

## If a harness does not work

- Make sure that the server shows `ready` and that the harness uses port 8080.
- Look in `logs` in the bundle folder: `server-8080.log` and `layer-8080.log`.
- Please open a GitHub issue with the harness name, its version, and the last lines of those two logs.
