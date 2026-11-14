# Agent harness from scratch

A minimal coding agent harness in one Python file, built step by step in [Building an agent harness from scratch to pass Ian Goodfellow's intelligence test](https://notesbylex.com/building-an-agent-harness-from-scratch) on notesbylex.com.

It has the parts most coding harnesses share: an agent loop, a model wrapper (GPT-6 Luna through the OpenAI Responses API), four tools (read, write, edit and bash), context compaction, a safety classifier, and support for `AGENTS.md` context files and skills. All of it is in `harness.py`, under 500 lines including comments.

Not really intended for serious work. Read the article for how each piece works.

## Running it

You need [uv](https://docs.astral.sh/uv/) and an OpenAI API key. uv installs the OpenAI SDK listed in the header of `harness.py`.

```bash
git clone https://github.com/lextoumbourou/agent-harness-from-scratch.git
cd agent-harness-from-scratch
export OPENAI_API_KEY=...
uv run harness.py "Say hello and tell me what you can do"
```

The working directory defaults to `agent-harness-working/`. Pass a different one as the first argument:

```bash
uv run harness.py path/to/workdir "your task"
```

`--web-search` turns on the Responses API's built-in web search, which is billed separately from model tokens. Each run stops after 30 turns or $1.00 of model tokens.

**The bash tool can run any command on your machine.** The safety classifier asks before risky commands, but it is not a security boundary, so run the harness in a sandbox for anything real.

## Running it in a Docker Sandbox

This is how the article's CIFAR-10 run worked, with the paths adjusted for this repository. On a Mac, the one-time setup is:

```bash
brew trust docker/tap
brew install docker/tap/sbx
sbx login
sbx policy init balanced
```

Then, from the root of this repository:

```bash
name=cifar-$(date +%s)
mkdir "/private/tmp/$name"
sbx create --name "$name" shell "/private/tmp/$name" "$PWD:ro"
printf '%s' "$OPENAI_API_KEY" | sbx secret set openai --sandbox "$name"
sbx policy allow network --sandbox "$name" www.cs.toronto.edu,cave.cs.toronto.edu
sbx exec -it -e OPENAI_API_KEY=proxy-managed "$name" \
  uv run "$PWD/harness.py" --web-search "/private/tmp/$name" \
  "Build a CIFAR-10 image classifier from scratch. Find the dataset, train a model, and show predictions on images it was not trained on. Report its accuracy on held-out test images and leave the code and trained model in this directory. You may download data and install Python packages."
```

The OpenAI key stays on the host. Docker's proxy adds it to API requests, while the Python client inside the sandbox sees only a placeholder.

## What's here

| Path | What it is |
|---|---|
| `harness.py` | The whole harness |
| `test_harness.py` | Tests that never touch the network: `uv run test_harness.py` |
| `agent-harness-working/` | The example workspace: an `AGENTS.md` context file, a small CSV and a hello-world skill |
| `cifar10-first-run/` | The training script and results from the article's CIFAR-10 run (81.41% on the held-out test set) |
