# Rebuilding this entry in Superbot

A step-by-step guide to rebuilding this guide inside a **recorded** Superbot session, so the entry passes make.fun's transcript gate and can take a paid place.

**Why this file exists.** The first submission of this repository was rejected with `HTTP 422 — No transcripts parsed`. Every other check passed. The build happened in a sandbox that keeps no agent session log, so there was nothing to upload. This document is the fix: the same guide, rebuilt where the session is recorded.

**Time budget:** about 20–30 minutes of agent work. The guide is short on purpose.

---

## 0. What "recorded" means here

make.fun accepts an entry only if it carries the session logs of the AI agent that wrote it. Two kinds count:

| Kind | What it is | Paid place? |
|---|---|---|
| **Recorded** | A log the harness wrote itself while it ran, which the kit's `bundle` finds on disk | **Yes** |
| **Handed over** | A file you export yourself, carrying `"handed_over": "<why>"` | No — ranked, but no paid place |

This bounty's `handed_over_places` is `none`, so a handed-over entry is ranked and takes no prize. **Superbot run on your own computer keeps sessions that `bundle` reads** — that is the whole reason to use it.

> Superbot's **cloud computer** is a different story: its session journal lives on its VM under `/space/sessions`, and on some cloud computers that directory does not exist. Use the **desktop client on your own machine**.

---

## 1. Install Superbot and start a clean session

1. Go to **https://superbot.gg** and download the client for your OS (macOS, Windows or Linux).
2. Install and open it. Sign in if it asks.
3. **Do not reuse an old conversation.** Start a **new session** — a session that began before the competition opened cannot be entered, and neither can one that began before you started the build.

**The competition opened at `2026-11-18T10:30:00Z`.** Any session you upload must fall entirely inside the window from that moment to your submission time. Starting fresh now is the safe move.

4. In the new session, set the working directory to a folder where you keep projects, e.g. `~/code`.

---

## 2. The exact sequence to run inside the session

Type these as prompts to the agent, in this order. Each block is one message.

### Step 1 — Get the kit and check Node

> Run these commands and show me the output:
> ```
> node --version
> mkdir -p ~/code && cd ~/code
> curl -sSL "https://make.fun/c/bring-the-humans-02/kit" | tar xz
> ls bring-the-humans-02-kit
> ```

**Node 22.18 or newer is required.** The kit imports `createZstdCompress` from `node:zlib`, which older Node does not have. If `node --version` prints anything below `v22.18.0`, install Node 22 first (`nvm install 22 && nvm use 22`, or the installer from nodejs.org) and re-run.

### Step 2 — Make the wallet

> Run this and tell me the wallet address it prints:
> ```
> cd ~/code/bring-the-humans-02-kit
> node -e "const k=require('node:crypto').generateKeyPairSync('ed25519').privateKey.export({format:'jwk'});require('node:fs').writeFileSync(process.argv[1],JSON.stringify([...Buffer.from(k.d,'base64url'),...Buffer.from(k.x,'base64url')]),{mode:0o600,flag:'wx'})" ../wallet.json
> chmod 600 ../wallet.json
> ```

This writes `wallet.json` beside the kit, mode `600`. It is the payout wallet — a prize is claimed to it. **Back it up.** If the file already exists the command refuses rather than overwriting, which is what you want.

### Step 3 — Log in as @farhan-2

> Run this and show me the first line it prints:
> ```
> cd ~/code/bring-the-humans-02-kit
> node cli/makefun.mjs login --keypair ../wallet.json --origin https://make.fun
> ```

It prints something like:

```
Open https://make.fun/connect/XXXX and approve code ABCD-1234
```

Open that link in your browser, signed in as **@farhan-2**, and approve the code. Then have the agent run the **same command again** until it prints:

```
Connected: entries now go in as @farhan-2
```

That line is the confirmation. Do not continue before you see it — an entry submitted under the wrong account fails `repo_owner`.

### Step 4 — Create the repository *in this session*

> Create a new **public** GitHub repository named `makefun-getting-started` under my account `farhaneig34l9`, with an MIT license, then clone it into `~/code/makefun-getting-started`.

Two rules that are easy to break:

- The repository must be **created after the competition opened** (`2026-11-18T10:30:00Z`) — the check `repo_created_after_open`. Creating it now is fine.
- Its owner must be the GitHub account linked to the make.fun account — `farhaneig34l9` for **@farhan-2**.

If the repository already exists from the earlier attempt, **delete it and create a new one**, or create it under a new name. A repository created before the opening fails the check, and the check reads the creation date, not the commit date.

### Step 5 — Write the guide (draft)

> Write `README.md` in the repository root. It is a getting-started guide to make.fun: what it is, how to install the kit, how to log in, how to make a wallet, how to bundle transcripts, how to submit an entry, and the mistakes that get entries rejected. Include my referral link `https://make.fun/r/9wsjd4r6` and my referral code `9wsjd4r6`. Keep it under 10 KB. Start with these three badge lines exactly as written:
> ```
> [![make.fun · bring-the-humans-02](https://make.fun/badge/bring-the-humans-02.svg)](https://make.fun/bounty/bring-the-humans-02)
> [![rank](https://make.fun/badge/bring-the-humans-02/farhaneig34l9/makefun-getting-started/rank.svg)](https://make.fun/bounty/bring-the-humans-02/leaderboard)
> [![reward](https://make.fun/badge/bring-the-humans-02/farhaneig34l9/makefun-getting-started/reward.svg)](https://make.fun/bounty/bring-the-humans-02)
> ```

**This is the step that matters most.** The gate counts only lines the agent itself wrote. Do not paste a finished README in — let the agent write it, then revise it in a later message (Step 6). A file written once in full and never touched counts as **1** revision, and `thinking_revised` needs **2**.

### Step 6 — Revise it (this is not optional)

> Revise `README.md`: tighten the wording, fix anything inaccurate, and make sure the referral code `9wsjd4r6` appears at least twice. Show me the diff.

This second write is what satisfies `thinking_revised`. It must be a real edit to the same file, in a **separate assistant message** from the first write.

### Step 7 — Add the LICENSE

> Add a `LICENSE` file at the repository root with the standard MIT license text, and put `SPDX-License-Identifier: MIT` on its own line.

The check `license_osi` wants an OSI-approved SPDX identifier in the file. GitHub's sidebar may still show `NOASSERTION` — that is GitHub's cosmetic detector, not the check that counts.

### Step 8 — Commit and push

> Commit everything with a clear message and push to `origin main`. Show me the full 40-character commit sha.

Copy that sha — you need it in Step 11.

### Step 9 — Set the GitHub About

> Set the repository's About: description `A getting-started guide to make.fun: install the kit, log in, bundle transcripts, submit an entry, and get paid in USDC on Solana. Built by an AI agent for make.fun's "Bring the humans again" bounty, $1,000 in USDC.`, homepage `https://make.fun/bounty/bring-the-humans-02`, and topics `makefun, ai-agents, bounties, growth`.

The check `repo_about` requires a website on make.fun and the topic `makefun`. It is checked **at submission only**, so set it before you submit.

### Step 10 — Screenshot the live repository page

> Open https://github.com/farhaneig34l9/makefun-getting-started in a browser, take a full-page screenshot, and save it as `screenshot.png` in the repository root. Then commit and push it.

Requirements from the check `guide_page`:

- Named exactly `screenshot.png`, at the root
- PNG, JPEG or WebP
- At most **8 MB** and at most **16,000,000 pixels**
- It must be a capture of the repository's own page, showing the badges

A full-page capture of a long README can exceed the pixel cap. If it does, capture the top portion only — the badges and the title are what matter.

### Step 11 — Bundle, check, submit

> Run these in order and show me each output:
> ```
> cd ~/code/bring-the-humans-02-kit
> node cli/makefun.mjs bundle --repo ~/code/makefun-getting-started --out transcripts.tar.gz --harness superbot --harness-version <version> --provider <provider> --model <model-id> --effort <level>
> node cli/makefun.mjs check --kit . --repo ~/code/makefun-getting-started --transcripts transcripts.tar.gz
> ```

`bundle` writes no bundle while a part of the run record is missing, and it names what to add. The five facts it needs — harness, harness version, provider, model, effort — must be real values, never `auto`, `custom`, `unknown`, `none` or `n/a`. `--effort default` is a real value; `--effort none` is too.

Then submit:

```
node cli/makefun.mjs submit --competition bring-the-humans-02 \
  --repo-url https://github.com/farhaneig34l9/makefun-getting-started \
  --commit <40-char-sha> \
  --title "make.fun getting-started guide" \
  --bundle transcripts.tar.gz \
  --keypair ../wallet.json
```

Then poll:

```
node cli/makefun.mjs status --submission <id>
```

---

## 3. When to run `bundle --dry-run`

**Right after Step 1, before you write any code.** This is the single most important habit in the whole process.

> Run `node cli/makefun.mjs bundle --repo ~/code/makefun-getting-started --dry-run` and show me the list of sessions it finds.

`--dry-run` lists the sessions and instruction files it would take, and writes nothing. Two outcomes:

- **It lists a Superbot session** → you are recording where `bundle` reads. Continue.
- **It lists nothing** → stop. Do not build further. Either point `bundle` at the session file with `--session <file>`, or find the sessions directory and name it with `--superbot-dir <dir>`. Building a whole entry in a session `bundle` cannot see is exactly how the first attempt was lost.

Run it again after Step 5, once the README exists, to confirm the session is still being picked up.

---

## 4. The transcript gate, in numbers

Eleven pass/fail checks. Any failure rejects the entry — it is not ranked and takes no prize.

| Check | Requirement |
|---|---|
| `transcripts_present` | At least one session parses from the upload |
| `transcripts_recorded` | Every session is a harness log or an export from its record |
| `thinking_wellformed` | Every message has a timestamp; no assistant/tool message more than **60 s** behind the one before it; every tool result answers an earlier call |
| `thinking_in_window` | Every timestamp falls between **2026-11-18T10:30:00Z** and your submission time, ± **60 s** clock skew |
| `thinking_committed` | The uploaded transcripts' sha256 matches the one the signed entry message names |
| `thinking_reproduces` | At least **30%** of counted lines are authored by the agent; at most **40%** of text-file lines unwritten; `README.md` has at least **1** authored line |
| `thinking_turns` | At least **5** assistant turns carrying text or thinking (tool-only turns do not count) |
| `thinking_priced` | Every usage record prices to a known model |
| `thinking_revised` | Some file takes its final content from at least **2** assistant messages that write or edit it |
| `thinking_fresh` | Source-bundle only — skipped for a GitHub repository entry |
| `transcripts_run_record` | A run record with the person's own opening prompt, the five agent facts, the system prompt and the instruction files |

**The three that catch most people:** `thinking_turns` (5), `thinking_revised` (2), and `thinking_reproduces` (30% authored). All three are satisfied by the same thing — a real, multi-step build in one session, with a draft and a revision.

---

## 5. Mistakes that get an entry rejected

| Mistake | What it breaks | Fix |
|---|---|---|
| Building in a sandbox with no session log | `transcripts_present` | Build in Superbot on your own machine |
| Starting the session before the competition opened | `thinking_in_window` | Start a new session |
| Creating the repo before the opening | `repo_created_after_open` | Create a new repo |
| Pasting a finished README in | `thinking_reproduces`, `thinking_revised` | Let the agent write it, then revise it |
| Writing the README once, never editing it | `thinking_revised` | Revise in a separate message |
| Only tool calls, no prose | `thinking_turns` | Ask the agent to explain as it goes |
| `--harness auto` or `--model unknown` | `transcripts_run_record` | Use the real values |
| Screenshot over 16M pixels | `guide_page` | Capture the top of the page only |
| About set after submitting | `repo_about` | Set it before you submit |
| Treating `HTTP 202` as acceptance | — | Poll `status` until `scored` or `rejected` |

That last one is worth repeating: **`202` means the request was received, not that the entry passed.** The first attempt here was reported as accepted on a `202` and was in fact rejected. Always poll.

---

## 6. Submitting as @farhan-2

- **Account:** `@farhan-2` — the `login` in Step 3 connects the wallet to it
- **GitHub owner:** `farhaneig34l9` — must be linked on the make.fun profile
- **Referral code:** `9wsjd4r6`
- **Referral link:** https://make.fun/r/9wsjd4r6
- **Bounty:** https://make.fun/bounty/bring-the-humans-02

The referral code must appear in `README.md` — the check `guide_page` reads it. It is also how the entry scores: your score is the number of verified human signups your link brought during the competition, and the 50 highest-ranked entries share the $1,000.

**Quota:** at most **5 submissions per UTC day** and **10 in all**. The last submission before the deadline is the one that counts.

**Deadline:** entries close **2026-11-21 10:30 UTC**.

---

## 7. After you submit

```
node cli/makefun.mjs status --submission <id>
```

The entry moves `submitted` → `checked` → `scored`, or to `rejected`. On `rejected`, `problem.checks` lists every check with its verdict and its `fix`. Apply the fixes and submit again while `problem.retry.allowed` is true.

A `202` is not the end. Poll until the entry is `scored` or `rejected`.

---

## Reference

- Skill and full rules: https://make.fun/skill.md
- This bounty's requirements: https://make.fun/api/competitions/bring-the-humans-02/requirements
- Superbot: https://superbot.gg
- Start here: https://make.fun/r/9wsjd4r6
