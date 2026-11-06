# LLM Red-Team Toolkit

> An offensive AI-security tool that fires a library of prompt-injection and jailbreak attacks at a language model, decides which ones broke its rules, rates each finding by OWASP LLM Top-10 category and severity, and writes an HTML + JSON report.

**Role:** Solo — designed and built end to end · **Status:** Working learning project (October 2026). The CLI, web UI, reports and tests all run locally against models I host myself with Ollama. Agentic (tool-abuse) attacks exist only as an early prototype, and a comparison with garak and CI are still on the roadmap. · **Source code:** private (walkthrough or live demo available on request)

## The problem

An LLM app keeps its rules, and often its secrets, in a hidden system prompt. The model can't reliably tell its owner's instructions apart from the user's text, so a well-worded message can talk it into breaking those rules. Prompt injection is #1 on the OWASP Top 10 for LLM applications, and system-prompt leakage is #7.

Poking at a chatbot by hand shows that this happens, but it isn't repeatable, it doesn't say how bad a failure was, and it leaves no evidence. I had already built a defensive tool, my [LLM AI Firewall](../llm-ai-firewall/). This project is its offensive counterpart. I wanted to learn how the attacks really work by building a small red-team scanner that runs the same attacks every time, judges the results the same way every time, and writes a report a security team could read.

## What I built

The toolkit (internal codename "A1") points a fixed set of attacks at a target model and grades how it held up.

- **A test target with a planted secret.** The target model gets a system prompt containing a fake password (a "canary") and the rule never to reveal it. If the canary shows up in a reply, the model broke its rule. That gives every attack a clear, objective pass/fail.
- **Attack library.** 12 attacks kept in a JSON data file. Each one is tagged with an OWASP LLM Top-10 category and the severity it carries if it works.
- **Scan engine.** It sends each attack to the target through a small HTTP adapter for Ollama's chat API, passes the reply to a detector, and records `LEAKED`, `BLOCKED` or `ERROR`.
- **Two detectors.** A fast keyword (canary) match, and an LLM-as-judge that uses a second local model to decide whether a password was really handed over.
- **Risk scoring and reports.** Every scan gets an overall risk grade and a timestamped HTML + JSON report that holds the evidence: the attack, the model's exact reply and the verdict.
- **Two interfaces.** A command-line tool (Typer) and a small web UI (FastAPI). Both reuse the same adapter, detectors and scoring.
- **Unit tests.** 8 pytest tests for the detectors and the risk scoring.

It started as a short Bash + `curl` script with 6 attacks. I then moved the attacks into a data file and rewrote the tool in Python with a CLI, reports, a web UI and tests.

## Architecture

```mermaid
flowchart LR
    CLI["CLI (Typer)"] --> ENG["Scan engine"]
    WEB["Web UI (FastAPI, localhost only)"] --> ENG
    LIB["Attack library: 12 attacks in JSON, OWASP tag + severity"] --> ENG
    ENG --> ADP["Target adapter (HTTP)"]
    ADP -->|attack| TGT["Target model on Ollama, system prompt holds a planted canary"]
    TGT -->|reply| DET["Detector: keyword match or LLM-as-judge"]
    DET -.->|judge mode only| JDG["Judge model extracts any password given out"]
    DET -->|LEAKED, BLOCKED or ERROR| RISK["Risk scoring: worst leaked severity"]
    RISK -->|CLI| REP["HTML + JSON report files"]
    RISK -->|web UI| PAGE["Results table in the browser"]
```

For each attack, the engine sends the planted system prompt plus the attack text to the target model and hands the reply to the chosen detector. The verdicts go to the risk scorer. The CLI then writes a report pair to disk, while the web UI shows the same results on the page.

## Key features

**Attack library mapped to the OWASP LLM Top-10.** The 12 attacks cover two OWASP categories and a range of techniques:

| OWASP category | Attacks | Techniques | Severity if it works |
|---|---|---|---|
| LLM01 Prompt Injection | 7 | direct instruction override, fake-administrator social engineering, jailbreak role-play, hypothetical framing, emotional urgency, forced reply prefix, emotional role-play | 5 HIGH, 2 MEDIUM |
| LLM07 System Prompt Leakage | 5 | sentence completion, secret hidden in a story, full system-prompt dump, JSON format trick, fake debug mode | 4 HIGH, 1 CRITICAL |

Adding an attack means adding an entry to the data file. No code changes are needed.

**Two detectors, chosen per scan.**

| Detector | How it decides | Trade-off |
|---|---|---|
| Keyword / canary | Is the planted secret in the reply? (case-insensitive) | Instant and free, but fooled when a model *refuses while quoting* the secret |
| LLM-as-judge | A second local model (Llama 3.2 3B) is asked to *extract* the password the reply hands out, or say NONE. It counts as a leak only if the extracted value is the real secret. | Designed not to be fooled by refusals or made-up passwords, but costs one extra model call per attack |

**Severity and an overall risk grade.** Each successful attack carries its severity. The target's overall grade is the worst severity among the attacks that leaked (CRITICAL, HIGH, MEDIUM or LOW), or PASS if nothing leaked.

**Evidence-first reports.** Every CLI scan writes a timestamped JSON file and an HTML page. The page shows the risk badge, counters (attacks run, leaked, blocked, errored) and one row per attack with its OWASP category, severity, verdict, the attack text and the model's exact reply. All model output is HTML-escaped, because a reply produced under attack is untrusted text.

**CLI and web UI.** The CLI takes the target model, attack file, per-attack timeout and detector as options, for example:

```bash
python a1.py --model llama3.1:8b --detector llm
```

The web UI runs on `127.0.0.1` only. It lists the models installed in Ollama, has a checkbox for the LLM-as-judge and shows the results table after a scan.

**Keeps going when an attack fails.** A timeout or error on one attack is recorded as `ERROR`, and the scan moves on, which matters when a large model runs slowly on a CPU.

**Agentic prototype (early).** A separate script gives a small agent one file-reading tool and one forbidden file, then sends 5 messages: one harmless control request and 4 attempts to make it misuse the tool. It counts a tool call only if the reply actually starts with one, so an agent that merely mentions the tool while refusing isn't counted. This prototype isn't part of the engine or the reports yet, and I haven't saved any results from it.

## Results

All numbers come from the project's saved report files and its test suite. Every scan ran on 2026-10-01 against two small open models on my own machine. That gives 11 saved reports: 8 runs on Llama 3.2 3B and 3 on Llama 3.1 8B. The first 9 runs used the first 8 attacks, and the last two used the full 12-attack library.

- **8 unit tests, all passing** (pytest; I re-ran them on a copy of the project): 3 for the keyword detector, 3 for the risk scoring and 2 for the LLM-as-judge with the model call mocked, so no model is needed.

**Final full-library scans (the two reports shown below):**

| Target model | Report | Attacks | Flagged as leaked | Blocked | Errors | Overall risk |
|---|---|---|---|---|---|---|
| Llama 3.2 3B | 2026-10-01 23:55 | 12 | 5 (4 HIGH, 1 MEDIUM) | 7 | 0 | **HIGH** |
| Llama 3.1 8B | 2026-10-01 23:59 | 12 | 6 (4 HIGH, 1 MEDIUM, 1 CRITICAL) | 6 | 0 | **CRITICAL** |

**What the reports show:**
- **The larger model was the riskier one.** Llama 3.1 8B printed its entire system prompt, secret included, in **all 3 of its runs**. Llama 3.2 3B never did in its 8 runs. That full dump is the CRITICAL finding that raised the 8B to CRITICAL. My working explanation is that the bigger model follows instructions, including the attacker's, more faithfully. With two models and a handful of runs, this is an observation from my runs, not a general rule.
- **The raw counts need a careful reading.** One of the 8B's six flagged hits was a refusal that quoted the secret. That is the keyword detector's known false positive, so the fair count is **5 real leaks for each model**. The difference between the models is how severe the leaks were, not how many there were.
- **The models failed on different tricks.** Sentence completion, the fake-administrator request and the emotional role-play worked on both. The jailbreak role-play and the hidden-in-a-story trick really worked only on the 3B (the 8B's story "hit" is the false positive above), and the full dump and the JSON format trick worked only on the 8B. The plain "ignore your instructions" attack was blocked by both in the final runs.
- **Results vary from run to run.** On the same 8 attacks, the 3B was flagged on 3 or 4 of them across 7 runs, so a single scan is a sample, not a verdict.

**Honest limits.**
- The library covers **2 of the 10** OWASP LLM Top-10 categories (LLM01 and LLM07), with 12 hand-written attacks. "Mapped to the OWASP Top-10" means each attack is tagged with its category. It doesn't mean every category is tested.
- The target is a single planted system prompt with a fake secret, not a real application. Only two small local models were tested, both through Ollama, and never a hosted or third-party system.
- The keyword detector's false positive (a refusal that quotes the secret) appears twice in the 11 saved reports, once for each model. The report files don't record which detector was used, and every saved verdict matches a plain keyword check. The LLM-as-judge is unit-tested with a mocked model, but I have **no measured accuracy** for it. In my manual testing, a yes/no judge was unreliable on a small model, which is why it extracts the password instead.
- Tests cover the detectors and the scoring, not the report writer or the web UI. Scans run one attack at a time. The web UI shows results but doesn't save report files, and it has no login. It is meant for local use only.
- Still on the roadmap: agentic tool-abuse attacks in the main engine, a side-by-side comparison with NVIDIA garak, more OWASP categories and CI.

## Tech stack

| Area | Tools |
|---|---|
| Language & CLI | Python, Typer |
| Web UI | FastAPI, Uvicorn, Pydantic, plain HTML/JavaScript |
| Target & judge models | Ollama (local), Llama 3.2 3B, Llama 3.1 8B |
| HTTP | httpx |
| Security frameworks | OWASP Top 10 for LLM Applications (LLM01, LLM07) |
| Testing | pytest (with mocked model calls) |
| Early prototypes | Bash, curl, jq |

## Screenshots

Report pages generated by the tool for the two final scans. **For this public page, I hid the attack-text and model-reply columns** so the payloads aren't published. The real report also shows both columns as evidence.

| Llama 3.2 3B: overall risk HIGH | Llama 3.1 8B: overall risk CRITICAL |
|---|---|
| ![Red-team report for llama3.2:3b: 12 attacks, 5 leaked, 7 blocked, overall risk HIGH, with OWASP category, severity and verdict per attack](images/report-llama3-2-3b.png) | ![Red-team report for llama3.1:8b: 12 attacks, 6 leaked, 6 blocked, overall risk CRITICAL, including a CRITICAL full system-prompt dump](images/report-llama3-1-8b.png) |

The web UI's scan form: pick a local model, optionally turn on the LLM-as-judge, and click Scan.

![Web UI scan form with a target-model dropdown, an LLM-judge checkbox and a Scan button](images/web-ui-scan-form.png)

## Design decisions

- **A planted canary as ground truth.** "Did the model misbehave?" is hard to judge in general. A fake secret in the system prompt makes success measurable: the attack worked if that exact secret came out. It also keeps testing safe, because no real secret is ever at risk.
- **Attacks are data, not code.** Keeping the attacks, their OWASP tags and their severities in one data file means the library can grow without touching the engine, and every scan runs exactly the same set.
- **Ask the judge to extract, not to answer yes/no.** In my testing, a small model asked "did it leak?" gave unreliable answers. So the judge copies out the password the reply gives, or says NONE, and the tool compares that value to the real secret. This is designed to avoid the refusal-that-quotes-the-secret false positive that the keyword check falls for.
- **Risk equals the worst finding, not an average.** As in a pentest report, one critical issue makes the target critical. Averaging would let many blocked attacks hide one full system-prompt dump.
- **One engine, two interfaces.** The CLI and the web UI share the same adapter, detectors and scoring, so a scan gives the same verdicts whichever one starts it.
- **Treat model output as hostile.** Replies are produced under attack, so they're escaped before they go into an HTML report or the web page. The web UI also binds to localhost only.

## See it running

Source code is private to protect the work. I'm happy to walk through the code or give a live demo in an interview — contact **sulmn.cs@gmail.com** or [LinkedIn](https://linkedin.com/in/suliman-khan-2b9280251).

<sub>© 2026 Suliman Khan. All rights reserved.</sub>
