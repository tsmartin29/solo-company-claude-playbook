# Running a one-person software company on 20+ Claude Code sessions

A practical guide: how to organise, protect and pace a pile of parallel AI coding sessions when you are the only human in the company.

Made by Tommy Martin. Built while running [ClubPlugins](https://clubplugins.com).

**Honest status.** These are lessons from a few weeks of running 20 to 35 Claude Code sessions at a time on one 24 GB Mac for a solo software company. Almost every rule below exists because something went wrong first (a crash, a lost file, a "done" that was not done). They are one setup's lessons, not laws; the numbers are examples. The tools linked at the end are small and were built for our own use; two of them (the task router and the safety hooks) are new and lightly used so far.

## Contents

1. [The shape of the problem](#1-the-shape-of-the-problem)
2. [Roles: give sessions jobs, not just tasks](#2-roles-give-sessions-jobs-not-just-tasks)
3. [Memory files: how the sessions stay consistent](#3-memory-files-how-the-sessions-stay-consistent)
4. [Safety rules for agents that can delete things](#4-safety-rules-for-agents-that-can-delete-things)
5. [Guarding the machine](#5-guarding-the-machine)
6. [Pacing your Claude usage](#6-pacing-your-claude-usage)
7. [Testing discipline](#7-testing-discipline)
8. [Habits that kept the human sane](#8-habits-that-kept-the-human-sane)
9. [Tools](#9-tools)
10. [Starter checklist](#10-starter-checklist)

## 1. The shape of the problem

When you have one session, you are the manager. When you have twenty, you are the bottleneck, and three new problems appear:

- **Coordination.** Sessions do not know about each other. Two will edit the same thing, or both will say they own a decision.
- **Resources.** The machine, the disk and your usage limit are all shared, and nobody is watching them.
- **Trust.** Each session is confident. Fast, confident, parallel work means mistakes arrive faster too, and some of them are irreversible.

Everything below is an answer to one of those three.

## 2. Roles: give sessions jobs, not just tasks

A task ends. A role keeps going and keeps context. We ended up with a small org chart made of sessions, each with a one-paragraph charter in a shared folder:

| Role | What it does | What it must not do |
|---|---|---|
| **Operator (COO)** | Runs on a schedule (every couple of hours) and in one live chat you talk to. Reads the inbox, checks every team against one goal-level critical path, decides, unblocks, reports to you. | Execute long work itself. A long-lived chat re-reads its whole history every turn, so coordination and decisions only. |
| **Heads** (finance, IT/infra, strategy, secretary) | One standing session each that keeps its area's files true after every change, and refreshes only before a report or when asked (to save usage). The secretary tracks every promise and open decision with evidence. | Act outside their area. |
| **Owning sessions** | One per product/repo. They hold the context and make the changes. Follow-ups go back to them. | Run heavy jobs without the queue. |
| **Cleanup manager** | A daily disk pass. Removes rebuildable things, moves everything to the Trash, reports in three lines (GB freed, GB free now, decisions needed). | Touch anything on its never-touch list: release folders, the guard's own state, training data, your real libraries. Empty the Trash. |
| **Task router** | Decides who should do each task: itself, an owning session, a cheaper model, a new session, or you. Replies with a short routing card. | Start sessions on its own. |
| **Testers** | A separate team whose only job is to use the product like a customer and report honestly what was and was not tested. | Share the builder's blind spots: do not let the builder grade their own work. |

Rules that made the org chart work:

- **One shared inbox.** Anything a session wants the operator or you to read goes into one append-only `INBOX.md`: a heading with date and task name, what was done, files written, decisions needed. Never delete entries. The operator reads it first, every pass, acts on each item, then moves handled items to `INBOX_read.md`. Cheap-mode runs (section 6) write there too, so nothing is lost when your usage runs out.
- **One goal-level critical path.** Every pass, the operator asks "what is the one chain of dependencies that decides whether the main goal is met?" and checks each team against it, instead of reporting on everything equally.
- **Route cheapest-first.** Matters only you can decide go to you. Follow-ups go to the session that already holds the context. Mechanical work goes to the smallest model, scoped work to a mid model, judgment to the top model. The router writes a card: `TO / MODEL / WHY / BRIEF / CONTEXT / COST / ACTION NEEDED`.
- **Only the human starts new standing sessions.** Agents propose; you approve.
- **Estimate in agent hours, with your test round as the gate.** We kept quoting human-developer days for work that took agents about two hours. The real time goes to verification and to you trying it. Say "built tonight, ready for your test", not "about a week".
- **Pair every blocker with its cheapest workaround.** "We would need a Windows machine" is a bad report; "we would need a Windows machine, or a free cloud CI runner for about nothing" lets you decide with the full picture.
- **Measure against independent truth.** If you grade your own output against another tool's output, you are only measuring agreement. Score everyone, including yourself, on the same independent answer key.

## 3. Memory files: how the sessions stay consistent

Claude Code can keep a folder of small markdown memory files plus an index that every session loads. We treat it as the company's institutional memory. What worked:

- **One rule per file**, with a short front-matter `name` and `description`, then three parts: the rule, **Why** (the incident or preference that caused it), **How to apply** (when it kicks in). The "why" matters: it lets a future session judge edge cases instead of following the rule blindly.
- **An index with one line per file, under ~200 characters.** The index is loaded into every session, so a bloated index quietly eats context and gets truncated. We hit the size limit and lost the last entries. Keep detail in the topic file.
- **Write the rule the moment you correct a session.** "No, never do that" is the highest-value sentence you will say all week. If it is not written down, the next session will do it again.
- **Supersede, do not accumulate.** When a decision changes, edit the old file and say it is superseded; contradictory memories are worse than none.
- **Keep status files small** (about 20 KB) and archive old entries, because every message re-reads them.
- **Do not store secrets or other people's data** in memory files. They are plain text and every session reads them.

A template is in `templates/memory-file.md`.

## 4. Safety rules for agents that can delete things

An agent that can run shell commands will, sooner or later, run a bad one. Our rules, each with the scar behind it:

1. **No permanent deletes of user files. Ever.** Move to the Trash (`mv path ~/.Trash/`). Deleting is fine only in temp and build areas (`/tmp`, build folders, `node_modules`, virtualenvs, caches). Prompts get forgotten; enforce it with a hook ([claude-safety-hooks](https://github.com/tsmartin29/claude-safety-hooks)).
2. **Dated backup before overwriting a tested file**, next to the file in the project, named `<file>.<YYYY-MM-DD>_<reason>.bak`. Not in `/tmp`, not in a scratch folder: we once kept the only good copy in scratch, a cleanup removed it, and a rollback was impossible. Also enforced by a hook, one backup per file per day. Keep the `.bak`s out of version control and out of cleanup passes.
3. **Never hand a human a destructive script with a guessable default path.** A copy-to-USB script with `rm -rf "$VOL/..."` and `VOL=/Volumes/USB` wiped a real drive that happened to have that common name. Rule: no `rm -rf`, `dd`, `mkfs`, or overwrite-in-place in a script you give someone with a default or guessable target. Make the user type the exact target, echo it back, and have the script refuse unless the target is clearly the right kind of place. Prefer the tool's own safe export over a shell script.
4. **Look before you delete.** List the target and show what will be removed before any deletion, and treat drives that could hold real libraries as irreversible.
5. **Delete your own scratch, and only your own.** Once a task is verified, remove the build folders, test outputs and scratch files you created, without asking (disks fill up and a full disk produces silent corruption). Never remove other sessions' work-in-progress, and never the folders your own scripts depend on.
6. **Never touch what is running.** Before cleanup, check no process is using the folder (`lsof +D`) and that no queued job references it.
7. **Frozen things stay frozen.** Release folders, shipped products and anything marked frozen are changed only by their owner or the operator, and nobody builds into a folder another session is shipping.
8. **A hook is a seatbelt, not a sandbox.** Ours cannot catch a delete hidden inside a script. Keep the rules in the prompt as well.

## 5. Guarding the machine

With 35 sessions open (a few GB just for them), plus one session running twenty parallel jobs, plus your normal apps, a 24 GB Mac ran out of memory and crashed five times in a few days. What fixed it:

- **One heavy job at a time, machine-wide, by default.** "Heavy" means over about 1.5 GB or a long CPU run: training, batch analysis, renders, browser recordings, full test suites, big builds. One worker, a few threads.
- **A queue, not a convention.** Agents run heavy jobs through `heavyrun "<session>: <job>" <command>`: first come first served, waits for enough free memory, runs at low priority (`nice -n 15`). A light lane exists for short small jobs (under ~15 minutes and ~1.5 GB) so quick builds do not wait behind a training run. Ranked order lives in a plain text file the operator edits.
- **A guard that does not rely on obedience.** A small launchd job every 15 seconds that only touches processes started by agent sessions: pause the biggest when memory is low, kill it when critical, resume one by one when memory recovers, and stop the lowest-ranked paused job if memory stays low too long (paused jobs keep their memory). It never touches browsers, the Claude app or system processes. ([memguard](https://github.com/tsmartin29/memguard))
- **Watch disk and swap, not just RAM.** A near-miss came from swap growing to ~10 GB while the disk had 2.7 GB free: swap lives on the same disk, so a full disk is a crash. New heavy jobs are held while the disk is low or swap is high.
- **Measure real footprint, not RSS.** GPU/Metal memory and re-parented background jobs hide from naive checks. Use footprint and walk the process tree.
- **Label jobs with their real peak memory.** A job labelled 0.7 GB that used 3.3 GB froze the queue.
- **Lane by size and time, then audit the lanes.** A multi-hour job sitting in the light lane blocked two-minute jobs for an hour. Someone (the operator) should check each queued job's lane against its real size and time.
- **Installs, copies, signing, a quick test are not heavy.** Do not queue them; over-queuing is its own failure mode.
- **Parallelism is a budget.** More slots open only when memory allows (we used 1 slot under ~10 GB available, 2 up to ~14 GB, 3 above), and workers per job stay at one unless the job is the top slot with lots of free memory.
- **Sessions must not spawn sessions to run more heavy work in parallel.** Queue it.

## 6. Pacing your Claude usage

Usage limits are the real budget. What we learned:

- **Know your numbers every pass.** Have the operator read 5-hour %, weekly %, per-model weekly %, and extra-usage % before starting sessions or sub-agents. ([claude-local-first](https://github.com/tsmartin29/claude-local-first) includes a menu-bar usage meter.)
- **Three bands.** Green: work normally. Amber (5-hour at 80 %+, weekly at 85 %+, extra usage at 90 %+): hand routine work (copy, reports, research, sorting files) to a cheaper route, run sub-agents only on small/mid models, start no new top-model sessions for routine work. Red (5-hour or extra usage at 100 %): the best model is for release-critical work only.
- **Never cheap mode for** builds, releases, installers, native or realtime code, or anything that needs connectors (cheap mode has none). Those wait for Claude.
- **A separate per-model allowance is free headroom.** If one model has its own weekly allowance, use it first and fall back to the normal model when it is used up or unavailable. Do not stall work waiting for it.
- **Cheap mode, properly isolated.** A launcher (part of [claude-local-first](https://github.com/tsmartin29/claude-local-first)) points Claude Code at OpenRouter with a clean environment (`env -i`), because otherwise your Claude login could be sent to the other endpoint. Keys live in the Keychain, never in files.
- **A relay that fires only when usage is empty.** A tiny no-AI script notices that usage is empty, checks the machine has spare RAM, and starts one cheap run (at most once an hour) that summarises what each chat was doing into the inbox and works through a folder of handoff files. When Claude is back, you read the inbox first.
- **Handoff files.** A task for a cheaper model needs: the task, the files, the rules, and a "done when". Cheaper models follow written instructions well and guess badly.
- **What actually burned usage.** For us it was long sessions re-reading near-million-token histories plus sub-agent swarms. Scheduled tasks were under 1 %. So: one task per session; when a task is done or the session passes ~200k tokens, write a handoff note and start fresh; at most two sub-agents at once; run the free scripted checks before asking a model to check anything; open screenshots only for steps that failed or need a visual call, at half scale.
- **Automate the routing.** claude-task-router (coming soon) encodes the bands and rules above as a config file and a small CLI.

## 7. Testing discipline

- **Passing unit tests is not "works".** We had a run where many products were reported "built, tested and installed" on the strength of unit tests and a plug-in validator, and then failed immediately for the human: features did nothing, windows grew and could not shrink. The rule since: before saying built, tested or ready, use it as a real user would, on the installed artifact, with realistic sessions rather than one action in a loop, and say honestly what was and was not tried by hand.
- **Do not steal the human's screen.** Automated UI tests must never take focus or the keyboard. Run them in the background or off-screen, or ask first.
- **Look at the rendered result**, not just assertions.
- **Never let the builder grade the build.** Separate testers, separate session.
- **Make the nag automatic.** A Stop hook that notices "done/fixed/built" without any test run in the turn, and asks once, catches most of it. It must never loop, and it must fail open.
- **Name the gate.** Plans should say which human test round decides, not which calendar day.
- **Features freeze before launch.** Pick a date after which only fixes go in. Sessions will otherwise keep improving things you are about to ship.

## 8. Habits that kept the human sane

- **Talk to one page.** One live chat with the operator; everything you decide there is written into a decisions file so scheduled runs pick it up.
- **Plain labels in reports.** Internal shorthand is invisible to you after a week. Heads' reports should be scannable and name things the way you do.
- **Put the choice in the chat, not only in a document.** When you must decide, list the options in the reply.
- **Steps for the human are click-by-click.** If you must do something yourself (sign in, make an account), the session should give literal steps with the exact text to paste, inline.
- **Report like a boss would read it.** Per item: what is done, what is blocked and by what, what you need to decide. Short.
- **Proactive beats idle.** When you are away, the operator should use spare capacity on the critical path, report to the inbox, and never spend money you did not authorise.

## 9. Tools

All small, MIT licensed, macOS first, standard-library Python or shell.

| Repo | What it is |
|---|---|
| duet (coming soon) | A terminal and Mac app that hosts a turn-by-turn conversation between Claude and ChatGPT using normal subscription logins (no API keys). |
| [memguard](https://github.com/tsmartin29/memguard) | The RAM / swap / disk guard and the one-at-a-time heavy-job queue. |
| [claude-local-first](https://github.com/tsmartin29/claude-local-first) | Runs recurring work at the cheapest tier that can do it (local script, local model, cheap mode, then Claude), with a menu-bar usage meter whose suggestions you can decline. Includes cheap mode and the relay. |
| [claude-safety-hooks](https://github.com/tsmartin29/claude-safety-hooks) | Hooks: no permanent deletes, dated backups, "test before done" reminder. |
| claude-task-router (coming soon) | Rules engine: which model tier, effort and lane each task gets, from current usage. |

Guides: [problem-first-framework](https://github.com/tsmartin29/problem-first-framework) (build only what solves a real, sourced pain) and [claude-business-launch](https://github.com/tsmartin29/claude-business-launch) (setting up a software business with Claude).

Templates in this repo: `templates/claude-md-rules.md` (a block to paste into your global `CLAUDE.md`), `templates/routing-card.md`, `templates/handoff-file.md`, `templates/inbox-entry.md`, `templates/memory-file.md`.

## 10. Starter checklist

If you only do five things:

1. Install the delete/backup hooks and paste the rules block into `CLAUDE.md`.
2. Run the memory guard and make heavy jobs go through the queue.
3. Start one inbox file and one operator session that reads it first.
4. Watch your usage and write down what you do in amber and red *before* you are in red.
5. Write the memory file the next time you say "no, never do that".

## Licence

MIT. See `LICENSE`.
