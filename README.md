<div align="center">
<img src="assets/social-preview.png" alt="leak-gate" width="860">
</div>

<h1 align="center">leak-gate</h1>

<p align="center"><b>Scan for internal detail before the doc leaves the building.</b><br><sub>one file · zero dependencies · stdlib only</sub></p>

<p align="center"><a href="https://github.com/chenhz01/leak-gate/actions/workflows/gate.yml"><img src="https://github.com/chenhz01/leak-gate/actions/workflows/gate.yml/badge.svg" alt="tests"></a> <img src="https://img.shields.io/badge/python-3.9%2B-3776AB?style=flat-square" alt="python"> <img src="https://img.shields.io/badge/dependencies-0-2ea44f?style=flat-square" alt="deps"> <img src="https://img.shields.io/badge/license-Apache--2.0%20%2B%20CC%20BY%204.0-blue?style=flat-square" alt="license"></p>

---

## The failure this catches

- The external proposal still carries a colleague's real email, a customer's full name, an internal IP.
- A screenshot had a ticket number in the corner; you noticed three days after sending it.
- The customer list is spread across a dozen docs and nobody knows which ones went out.

One command, on a file you already have:

```bash
python gate.py examples/docs --customer-names examples/names.txt
```

<details>
<summary><b>Real output</b> — produced by the bundled <code>examples/</code>, not a mock-up</summary>

```text
scanned 1 file(s), 4 finding(s)
warning NET1  examples/docs/plan.md:16 10.12.3.8 — private-range IP address
warning NET2  examples/docs/plan.md:15 corp.internal — internal hostname
warning MAIL1 examples/docs/plan.md:15 zhangsan@corp.internal — email address not on allowlist
warning CUST1 examples/docs/plan.md:15 Acme Foods — customer name mention
```

</details>

## What it does

- Static rules for internal emails, private IPs, ticket IDs, and your own customer list.
- Allowlist file, so public addresses stop being reported.
- JSON output; runs offline, nothing leaves the machine.

## What it does **not** do

- It is a word-list and regex scanner. Reworded leaks will pass.
- It does not read images or PDFs.
- Zero findings is not a guarantee of safety — it means these rules found nothing.

Being explicit here is cheaper than a wrong expectation later.

## Exit codes

`0` = no **error-level** finding (warnings are printed but do not fail the run). `1` = at least one secret-level finding. `2` = a path does not exist, or nothing was scannable. **Warnings alone never fail the run; a mistyped path never passes silently.**

## Install

```bash
git clone https://github.com/chenhz01/leak-gate
cd leak-gate
python gate.py --help
```

Python 3.9+. Nothing to `pip install` — stdlib only.

## Use it in CI

```yaml
- name: leak-gate
  run: python gate.py examples/docs --customer-names examples/names.txt
```

## Use it as a GitHub Action

`action.yml` is included, so you can call this repo directly as a composite action.

```yaml
# .github/workflows/gates.yml
- uses: actions/checkout@v4

- uses: chenhz01/leak-gate@v0.1.3
  with:
    # IMPORTANT: this path is inside YOUR repository.
    # The examples/ in this repo are NOT in your workspace — pass your own files.
    args: "path/to/docs --customer-names path/to/names.txt"
```

Pin the tag (`@v0.1.3`) rather than `@main` — `main` moves.
Check [Releases](https://github.com/chenhz01/leak-gate/releases) for the newest tag; `v0.1.0` predates the
fix that makes a mistyped path fail instead of silently passing.

If you omit `args`, the action only prints help and exits 0. It raises a warning
annotation — quoting this gate's own correct usage — so a green check is never
mistaken for "it inspected my files".


## 中文速览

**泄露门** — 文件出门之前，先扫一遍内部信息。

- 对外方案里留着同事的真实邮箱、客户全名和内网 IP。
- 截图边角有个工单号，发出去三天后才有人提醒你。
- 客户名单散在十几份文档里，谁也不知道哪份已经被发出去过。

一行命令：`python gate.py examples/docs --customer-names examples/names.txt`

边界：It is a word-list and regex scanner. Reworded leaks will pass.

### 互推：只装一个的话，装哪个

十五个链接是迷宫，一个症状是门。按上表挑一个，其余九个在下面 [The gates](#the-gates)。
每个仓都能单独活：复制 `gate.py`，一条命令跑完，不共享运行时、不装包、不加配置。
有星就给这一个，没有也不影响任何仓的可用性——没有任何门是「先给星才能用」。

### 热门前置：保证结构，不保证排名

本页承诺的是 **15 项可机器复核的结构特征**，当前 **15/15**（2026-10-20 复核）。
GitHub 从未公开 Trending 的排序算法，所以任何「保证上热门」的说法都在描述一件它不可能知道的事；
本仓不这么说。跑 `python tools/hotness.py` 可自行复算每一条。

## Start here (if you install only one)

Fifteen links is a maze; one symptom is a door. Find your symptom, install that gate,
ignore the rest until it hurts.

| what you are seeing | install this |
|---|---|
| the bill is 10x normal and the trace is a wall of the same step | **[loop-gate](https://github.com/chenhz01/loop-gate)** |
| the agent can delete prod and nobody signed off on that | **[permission-gate](https://github.com/chenhz01/permission-gate)** |
| answers get worse after truncation and nobody knows what got dropped | **[context-gate](https://github.com/chenhz01/context-gate)** |
| two rules in the prompt contradict each other; the model obeys neither | **[instruction-gate](https://github.com/chenhz01/instruction-gate)** |
| AI text ships with fabricated links and empty phrases | **[output-gate](https://github.com/chenhz01/output-gate)** |
| half the repo might be dead and nobody can prove which half | **[deadcode-gate](https://github.com/chenhz01/deadcode-gate)** |

A gate you did not need is a gate you uninstall, so nothing here says "install all
fifteen". Each repo stands alone: copy `gate.py`, run one command, no shared runtime,
no package manager, no config file to grow.

**On stars.** If this one saved you an incident, star it — that is the only signal I get
about which gate to build next. There is no exchange: starring one repo does not owe me
a star on another, and no gate here is unlocked by a star. Star trading breaks GitHub's
rules and would poison the single honest metric this project has.

## Hot-ready guarantee

What is guaranteed is **structure**, not ranking. Fifteen properties, each one
machine-checked against this repo — current score **15/15** (2026-10-20).

Plain version: instead of asking you to trust a badge, I list what I actually checked,
name the script that checks it, and let you re-run it. If one of these lines is false,
that is a bug in my repo, not a matter of taste.

- Falsifiable side: `python tools/hotness.py` re-runs every check and prints the evidence line
  for each one. You do not have to trust this paragraph.
- Non-falsifiable side: GitHub has never published how Trending sorts repositories, so
  any repo promising you a guaranteed spot is telling you something it cannot know.
  This one does not make that promise.
- If a claim on this page stops being true, that is a bug: open an issue and it gets
  fixed in the same week, not in the next launch.

What actually carries this repo:

- the command in this README is byte-for-byte the command CI runs <sub>(57-char argv, exact match)</sub>
- the output block is a real run of `examples/`, not a mock-up <sub>(Real output section)</sub>
- CI asserts the exit code, so the badge can turn red — and it has <sub>(asserts exit=0)</sub>
- every fixture is committed, so a fresh clone reproduces the output <sub>(2 files under examples/ tracked in git)</sub>
- `* -text` in `.gitattributes`: same bytes on Windows and Linux <sub>(.gitattributes * -text)</sub>

## The gates

| # | gate | what it stops |
|---|---|---|
| 02 | **[loop-gate](https://github.com/chenhz01/loop-gate)** | Find where your agent looped, before it burns another run |
| 03 | **[permission-gate](https://github.com/chenhz01/permission-gate)** | Score how much more your agent can do than it should |
| 04 | **[context-gate](https://github.com/chenhz01/context-gate)** | Measure what truncation deletes, before the model starts guessing |
| 05 | **[instruction-gate](https://github.com/chenhz01/instruction-gate)** | Two of your instructions contradict each other. This finds them |
| 06 | **[output-gate](https://github.com/chenhz01/output-gate)** | Before AI-written text ships, run the four checks it always fails |
| 07 | **[memory-expiry-gate](https://github.com/chenhz01/memory-expiry-gate)** | Give every memory a TTL, then prove the expiry semantics actually hold |
| 08 | **[drift-gate](https://github.com/chenhz01/drift-gate)** | When embeddings drift, retrieval quietly rots. Fingerprint it |
| 09 | **[price-gate](https://github.com/chenhz01/price-gate)** | Your cost model is only as good as the price table under it |
| 10 | **[complexity-ceiling](https://github.com/chenhz01/complexity-ceiling)** | Per-file complexity budgets that fail the build instead of the reviewer |
| 11 | **[deadcode-gate](https://github.com/chenhz01/deadcode-gate)** | Quantify how much of the repo is dead, instead of guessing |
| 12 | **[docrot-gate](https://github.com/chenhz01/docrot-gate)** | Docs don't die at once. They rot one stale command at a time |
| 13 | **leak-gate** ← you are here | Scan for internal detail before the doc leaves the building |
| 14 | **[backup-gate](https://github.com/chenhz01/backup-gate)** | A backup you have never verified is a rumor |
| 15 | **[trail-gate](https://github.com/chenhz01/trail-gate)** | Hash-chained agent audit trail — and it verifies its own chain |

## License

- Code: **Apache-2.0** — see [LICENSE](LICENSE)
- Docs and example data: **CC BY 4.0** — see [LICENSE-SPEC](LICENSE-SPEC)

Attribution required when you redistribute the documentation; code can be used commercially
without a separate agreement. See [NOTICE](NOTICE).
