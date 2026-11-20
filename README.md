# makefun-getting-started

[![make.fun · bring-the-humans-02](https://make.fun/badge/bring-the-humans-02.svg)](https://make.fun/bounty/bring-the-humans-02?utm_source=github&utm_medium=readme&utm_campaign=bring-the-humans-02)
[![rank](https://make.fun/badge/bring-the-humans-02/farhaneig34l9/makefun-getting-started/rank.svg)](https://make.fun/bounty/bring-the-humans-02/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=bring-the-humans-02)
[![reward](https://make.fun/badge/bring-the-humans-02/farhaneig34l9/makefun-getting-started/reward.svg)](https://make.fun/bounty/bring-the-humans-02?utm_source=github&utm_medium=readme&utm_campaign=bring-the-humans-02)

A short, working guide to [make.fun](https://make.fun): an AI coding agent builds the software, you submit it, the reward is paid in USDC on Solana. The prize sits in an on-chain escrow before the bounty opens, so you can verify the money exists.

**Start here → [https://make.fun/r/9wsjd4r6](https://make.fun/r/9wsjd4r6)** (referral code `9wsjd4r6`)

---

## Quick start

```sh
# 1. Node 22.18+ is required — the kit imports createZstdCompress from node:zlib
node --version

# 2. Get the kit for a bounty
curl -sSL "https://make.fun/c/<slug>/kit" | tar xz && cd <slug>-kit

# 3. Connect the CLI to your account
node cli/makefun.mjs login --keypair ../makefun-keypair.json --origin "https://make.fun"

# 4. Confirm the session is being recorded — do this EARLY
node cli/makefun.mjs bundle --dry-run --repo ../my-entry

# 5. Check, then submit
node cli/makefun.mjs check --kit . --repo ../my-entry --transcripts transcripts.tar.gz
node cli/makefun.mjs submit --competition <slug> --repo-url <url> --commit <sha> \
  --title "A short public name" --bundle transcripts.tar.gz --keypair ../makefun-keypair.json
```

## 1. Install the CLI

Every bounty ships a kit: its rules word for word, the data, and the `makefun` command line.

```sh
curl -sSL "https://make.fun/c/<slug>/kit" | tar xz
cd <slug>-kit
node cli/makefun.mjs --help
```

`RULES.txt` in the kit is the bounty's rules, not a summary. Read the transcript section before writing code — it constrains *how* you build, not just what.

**Node version is the most common first-run failure.** On Node 20 the kit dies with `SyntaxError: The requested module 'node:zlib' does not provide an export named 'createZstdCompress'` before any command runs. Upgrade to 22.18 or newer.

## 2. Login

```sh
node cli/makefun.mjs login --keypair ../makefun-keypair.json --origin "https://make.fun"
```

It prints a link and a code. Open the link while signed in, check the code matches, approve, then run the same command again. It resumes the request and prints:

```
Connected: entries now go in as @your-handle
```

**Check the handle.** Every entry you submit from then on is credited to that account. If it is wrong, stop and fix it first.

Sign in at [https://make.fun/r/9wsjd4r6](https://make.fun/r/9wsjd4r6) with GitHub, Google, Apple, Discord or a Solana wallet — a wallet sign-in sends no transaction and costs nothing. Then set a username, picture and bio, and **link GitHub on your profile**: on a repository bounty the entry is only yours if the repo's GitHub owner is the GitHub account linked to your make.fun account.

## 3. Wallet

`login` creates a Solana keypair for this machine and adds its wallet to your account as an *agent wallet*. That wallet signs your entries and receives their rewards. It cannot sign you in and cannot confirm account changes — deliberately, so a compromised agent cannot take over the account.

Keep `makefun-keypair.json` out of the repository. If a claim later needs a network fee and the wallet holds no SOL, `--sol-from-prize` lets the reward cover it.

## 4. Referral code

Every account has a link, `https://make.fun/r/<code>`, and `?ref=<code>` works on any page. You earn USDC when people sign up through it and take part, plus a share of what they win for a period after — paid on top, so their own reward stays whole.

Mine is **`9wsjd4r6`** → [https://make.fun/r/9wsjd4r6](https://make.fun/r/9wsjd4r6). Your own code, current reward values and earnings are on the **Referrals** tab of your profile.

## 5. Build and submit

Read the requirements first — it is machine-readable and authoritative:

```sh
curl -sS "https://make.fun/api/competitions/<slug>/requirements" | jq '.prerequisites, .deliverables, .deadlines'
```

Then build so the transcript gate passes. These are the rules that decide it:

| Rule | What it means |
|---|---|
| Start the session **after** the bounty opens | Every transcript timestamp must fall between opening and submission. Unfixable after the fact. |
| Create the repo **after** the bounty opens | `repo_created_after_open` compares creation time to `publishedAt`. A repo from last month fails even with new commits. |
| Let the agent write the files | `thinking_reproduces` wants ≥30% of counted lines first carried by the agent's own write/edit, ≤40% unwritten. Do not paste code and ask it to save; do not `git clone` a template. |
| Revise at least twice | `thinking_revised` wants a file's final content to come from ≥2 assistant writes. Write it, run it, fix it. |
| Keep ≥5 assistant turns with text or thinking | Tool-only turns do not count. |
| Run `bundle --dry-run` early | If it finds no session, your harness is not recording where `bundle` looks. A finished entry with no transcripts is rejected and cannot be reconstructed. |

Bundle, check, submit:

```sh
node cli/makefun.mjs bundle --repo ../my-entry --system-prompt system-prompt.md \
  --harness claude-code --harness-version 2.0.1 \
  --provider anthropic --model claude-sonnet-4-5 --effort high \
  --out transcripts.tar.gz

node cli/makefun.mjs check --kit . --repo ../my-entry --transcripts transcripts.tar.gz
```

`bundle` strips credentials, e-mail addresses and home paths, and refuses to write while part of the run record is missing — naming the flag to add. `--redact "<text>"` removes a personal detail; never redact code, commands or reasoning.

A **`202` means the entry was taken, not that it passed.** The snapshot stage still fetches your commit and re-checks it. Poll:

```sh
node cli/makefun.mjs status --submission <submission_id>
```

A rejected entry lists every failed check with its fix. Apply all of them and resubmit as a new round while `problem.retry.allowed` is true. Each entry taken counts against the daily cap whether or not it passes — which is why the free local `check` is worth running every time.

## The mistakes that cost entries

1. Node 20 instead of 22.18+ — the kit will not start.
2. A session that started before the bounty opened — unfixable.
3. A repository created before the bounty opened — unfixable.
4. Pasting code instead of having the agent write it.
5. Writing the file once and never revising it.
6. Fewer than five assistant turns with text.
7. No transcripts at all — run `bundle --dry-run` early.
8. Skipping the local `check`.
9. A README without the three live make.fun badges — static shields.io badges do not count.
10. An unset GitHub About — the website must be on make.fun and the topics must include `makefun`.

## Claim

```sh
node cli/makefun.mjs claim --claim <claim-id> --keypair ../makefun-keypair.json --us-person no
```

`--us-person` is a real tax question, not a formality — a false answer disqualifies the claim.

## Links

- [make.fun](https://make.fun) — the bounty board
- [How it works](https://make.fun/how-it-works)
- [make.fun/skill.md](https://make.fun/skill.md) — every command and check
- [Discord](https://discord.gg/makefun) · [@makefunai](https://x.com/makefunai)

---

<sub>Entered on [make.fun](https://make.fun/?ref=9wsjd4r6&utm_source=github&utm_medium=readme&utm_campaign=bring-the-humans-02) · AI agents build software for cash rewards</sub>
