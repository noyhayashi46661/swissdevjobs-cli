<div align="center">

# 🇨🇭 swissdevjobs-cli

**Search, filter, and apply across 9 job boards in 8 countries — ~4,700 tech jobs with salary data, plus all ~50,000 Swiss postings on jobs.ch & jobup.ch and ~9,000 Singapore IT postings on MyCareersFuture — without leaving your terminal.**

🇨🇭 Switzerland · 🇩🇪 Germany · 🇬🇧 UK · 🇺🇸🇨🇦 US & Canada · 🇳🇱 Netherlands · 🇫🇷 France · 🇸🇬 Singapore

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Python](https://img.shields.io/badge/python-3.9%2B-3776AB.svg?logo=python&logoColor=white)](https://www.python.org/)
[![Dependencies](https://img.shields.io/badge/dependencies-zero-brightgreen.svg)](pyproject.toml)
[![Boards](https://img.shields.io/badge/boards-9_boards_·_8_countries-orange.svg)](#boards)
[![Claude Code plugin](https://img.shields.io/badge/Claude%20Code-plugin-8A63D2.svg)](#install)
[![MCP](https://img.shields.io/badge/MCP-server%20included-6E56CF.svg)](#mcp-server)
[![CI](https://github.com/Stupidoodle/swissdevjobs-cli/actions/workflows/ci.yml/badge.svg)](https://github.com/Stupidoodle/swissdevjobs-cli/actions/workflows/ci.yml)

</div>

---

Every posting on [swissdevjobs.ch](https://swissdevjobs.ch) — and its five sister
boards covering Germany, the UK, the US & Canada, the Netherlands, and France — is
required to publish a salary range. That makes them the rare job boards where you can
filter by pay before you click. Since v0.5 the tool also searches
[jobs.ch](https://www.jobs.ch) and [jobup.ch](https://www.jobup.ch) — Switzerland's
two biggest boards, every industry, ~50,000 postings. It also searches
[MyCareersFuture](https://www.mycareersfuture.gov.sg), Singapore's government
job portal, scoped to its ~9,000 Information Technology postings. One search,
nine boards, JSON-first so an LLM agent can drive it.

It also remembers what you've already applied to, across every board, so the same
job never shows up twice.

```console
$ sdj list --tech Kubernetes --remote --min-salary 90000 --sort salary
57 shown · 57 match filters · 5521 in feed · 3 hidden (already applied)
----------------------------------------------------------------------------------------
6a60ec82…  devitjobs-us    p=2026-07-22  Senior Maximo Systems Developer/Integrator  TEKsystems c/o Allegis Gr  Ottawa   USD 208'000–270'400  remote  API, Ansible, Bash, ESB, GIS, IBM
6a6cc9bd…  devitjobs-us    p=2026-07-31  Senior Solutions Architect                  TEKsystems c/o Allegis Gr  Calgary  USD 187'200–228'800  remote  AI, API, Architect, Cloud, Docker
```

## Contents

- [Why](#why) · [Please be reasonable](#please-be-reasonable) · [Install](#install) · [Boards](#boards) · [Configure](#configure) · [Commands](#commands)
- [How applying works](#how-applying-works) · [Filtering](#filtering)
- [MCP server](#mcp-server) · [Under the hood](#under-the-hood) · [Cloudflare](#cloudflare)
- [Claude Code skill](#claude-code-skill) · [Development](#development)

---

## Why

| | |
|---|---|
| 🌍 **Nine boards, one tool** | The all-IT devitjobs family across six countries, plus jobs.ch & jobup.ch for every industry in Switzerland and MyCareersFuture for Singapore IT roles — one search, per-board currencies |
| 💰 **Salary is a first-class filter** | `--min-salary 130000` — no more opening 40 tabs to find the range |
| 📅 **Real posting dates** | The site re-stamps `activeFrom` when it bumps a listing. This decodes the true creation time from the MongoDB ObjectId, so a "new" job that's actually four months old can't fool you |
| 🧠 **Remembers where you applied** | Local SQLite. Applied jobs vanish from `list` automatically |
| 🤖 **Agent-native** | An [MCP server](#mcp-server) plus `--json` on every command; duplicates come back as *data*, not errors |
| 📦 **Zero dependencies** | Python stdlib only. No `requests`, no `pydantic`, no supply chain |
| 🎯 **Refuses to black-hole your application** | Detects postings the site can't actually deliver and tells you where to apply instead |

---

## Please be reasonable

This talks to somebody else's website, built by a small team who chose to make salary
transparency mandatory. Keep your request volume human. Don't strip the caching. Don't
fire off applications to postings you haven't read — that wastes a real recruiter's
afternoon and poisons the well for everyone using the board honestly.

Read swissdevjobs.ch's terms before you automate anything on top of this.

---

## Install

### In Claude Code — two commands

```sh
/plugin marketplace add Stupidoodle/swissdevjobs-cli
/plugin install swissdevjobs@swissdevjobs
```

That's it. Claude Code prompts for your name, email, and CV path, then starts
the MCP server with `uvx` — nothing to install first, and no shell profile is
touched. Ask it *"find me senior Python roles in Zurich over 140k"* and go.

Requires [uv](https://docs.astral.sh/uv/) on your PATH
(`brew install uv`, or `curl -LsSf https://astral.sh/uv/install.sh | sh`).

### As a CLI

```sh
uv tool install git+https://github.com/Stupidoodle/swissdevjobs-cli
```

<details>
<summary>Other ways</summary>

```sh
# pipx
pipx install git+https://github.com/Stupidoodle/swissdevjobs-cli

# from a checkout, editable
git clone https://github.com/Stupidoodle/swissdevjobs-cli
cd swissdevjobs-cli
uv tool install -e .     # or: pipx install -e .

# no install at all — run it once
uvx --from git+https://github.com/Stupidoodle/swissdevjobs-cli sdj list --remote
```

Not published to PyPI.
</details>

Installs two equivalent binaries: **`sdj`** and **`swissdevjobs`**. Python 3.9+.

---

## Boards

Three platforms, nine boards, one tool. The devitjobs family shares one
backend across six countries — identical API, identical apply flow — the
JobCloud platform (jobs.ch, jobup.ch) covers **every** industry in
Switzerland, and MyCareersFuture is Singapore's government job portal, scoped
to IT and publishing salary:

| | Board | Country | Scope | Currency | Salary data | Direct apply |
|---|---|---|---|---|---|---|
| 🇨🇭 | [swissdevjobs.ch](https://swissdevjobs.ch) | Switzerland | IT | CHF | always | ✅ |
| 🇩🇪 | [germantechjobs.de](https://germantechjobs.de) | Germany | IT | EUR | always | ✅ |
| 🇬🇧 | [devitjobs.uk](https://devitjobs.uk) | United Kingdom | IT | GBP | always | ✅ |
| 🇺🇸🇨🇦 | [devitjobs.com](https://devitjobs.com) | US & Canada | IT | USD | always | ✅ |
| 🇳🇱 | [devitjobs.nl](https://devitjobs.nl) | Netherlands | IT | EUR | always | ✅ |
| 🇫🇷 | [devitjobs.fr](https://devitjobs.fr) | France | IT | EUR | always | ✅ |
| 🇨🇭 | [jobs.ch](https://www.jobs.ch) | Switzerland | **all industries** | CHF | none | 🌐 via ATS |
| 🇨🇭 | [jobup.ch](https://www.jobup.ch) | Switzerland (Romandie) | **all industries** | CHF | none | 🌐 via ATS |
| 🇸🇬 | [MyCareersFuture](https://www.mycareersfuture.gov.sg) | Singapore | IT | SGD | always | 🌐 on the posting page |

All boards are searched by default. A selector is a board id (just that one)
or a country code (every board there) — `--board`, with `--source` and
`--country` as accepted aliases:

```sh
sdj list --board jobsch                   # just jobs.ch
sdj list --board de --board uk            # just Germany + UK, this once
sdj config --boards ch,de                 # persist: all CH boards + Germany
sdj config --boards swissdevjobs          # persist: one board only
sdj config --boards all                   # back to everything
```

`sdj boards` (and the MCP tool `list_boards`) prints all of this as data —
no need to memorize which board does what:

```console
$ sdj boards
9 boards — select with --board <id|country>, persist with `sdj config --boards`
----------------------------------------------------------------------------------------------------
swissdevjobs     ch   SwissDevJobs        CHF  enabled  it · salary · unfilterable: workload
germantechjobs   de   GermanTechJobs      EUR  enabled  it · salary · unfilterable: workload
devitjobs-uk     uk   DevITjobs UK        GBP  enabled  it · salary · unfilterable: workload
devitjobs-us     us   DevITjobs US/CA     USD  enabled  it · salary · unfilterable: workload
devitjobs-nl     nl   DevITjobs NL        EUR  enabled  it · salary · unfilterable: workload
devitjobs-fr     fr   DevITjobs FR        EUR  enabled  it · salary · unfilterable: workload
jobsch           ch   jobs.ch             CHF  enabled  all-industries · no-salary · search-driven · no-native-apply · unfilterable: salary, remote, visa, level, tech, language · categories: it
jobup            ch   jobup.ch            CHF  enabled  all-industries · no-salary · search-driven · no-native-apply · unfilterable: salary, remote, visa, level, tech, language · categories: it
mycareersfuture  sg   MyCareersFuture     SGD  enabled  it · salary · search-driven · no-native-apply · unfilterable: visa, level, workload
```

**jobs.ch and jobup.ch are search-driven.** Their ~50k-job inventory can't be
mirrored (the API serves up to 200 rows per page and stops paging at 100 pages), so
they answer your query server-side, newest first — pass free text for real
coverage, and `--category it` to stay in tech:

```console
$ sdj list "pflegefachfrau" --board jobsch --limit 2      # any industry, server-side
2 shown · 100 match filters · 100 in feed
----------------------------------------------------------------------------------------
40b2f214…  jobsch  p=2026-08-25  Dauernachtwache - Dipl. Pflegefachperson HF / FH  Stiftung entero      Niederlenz  —
b5367506…  jobsch  p=2026-08-25  Dipl. Pflegefachfrau HF/FH mit Fachverantwortung  Spital Männedorf AG  Männedorf   —

$ sdj list "python" --board ch --category it              # CH tech across all three boards
```

They publish no salary data (rendered honestly as `—`) and have **no native
apply** — every posting routes to the company's own ATS, so `direct-apply`
refuses with the real apply URL instead of pretending.

**MyCareersFuture is search-driven too.** Singapore's government job portal
(Workforce Singapore) carries ~9,000 active Information Technology postings
and its API caps `limit` at 100 rows per page, so queries pass server-side,
newest first. Unlike jobs.ch it *does* publish salary — monthly SGD on the
wire, annualized by the adapter — and skills, so `--tech` and `--remote`
filter normally there. `--visa`, `--level`, and `--workload` are unavailable
by design: no sponsorship-eligibility field exists on the wire (Singapore's
employer applies for a specific work pass after the fact, so there is no
"will they sponsor" flag to read); the board's own position levels
("Professional", "Middle Management", "Fresh/entry level", …) have no honest
mapping onto the tool's Junior/Regular/Senior/Principal/CLevel enum, so they
stay in `job.raw` rather than being guessed at; and no workload-percentage
field exists at all. There is **no native apply** either — applying happens
on the posting page, through the portal's own flow, so `direct-apply`
refuses and hands back that page's URL.

Native postings on every devitjobs board publish a salary range. Syndicated listings
(marked `isPartner` by the boards — the majority outside Switzerland) sometimes
carry no range or a single-point figure; the tool renders those honestly and
**refuses to native-apply to them**, handing you the real ATS URL instead —
their pages have no native apply form, so a native submission would silently
vanish. The applied-jobs ledger is shared — apply to a role on one board and
the same company+role is hidden on all of them.

Want a board outside the family? [Open a board request](https://github.com/Stupidoodle/swissdevjobs-cli/issues/new?template=board_request.yml).

---

## Configure

Reading jobs needs no configuration at all. Applying needs to know who you are.

```sh
sdj config --init     # writes ~/.config/swissdevjobs-cli/.env (chmod 600)
sdj config            # show what's resolved, and from where
```

Then edit the file:

```dotenv
SDJ_NAME="Your Name"
SDJ_EMAIL="you@example.com"
SDJ_CV="/absolute/path/to/cv.pdf"

# Optional: which boards to search (default: all)
# Board ids and/or country codes: jobsch, jobup, swissdevjobs,
# germantechjobs, devitjobs-*, ch, de, uk, us, nl, fr
# SDJ_BOARDS=ch,de
```

### Where settings come from

`.env` files are read stdlib-only — no `python-dotenv` dependency. Anything already
exported in your shell always wins, so nothing on disk can silently shadow it.

```mermaid
flowchart TD
    A["1 · Command-line flag<br/>--name / --email / --cv"]
    B["2 · Real environment<br/>SDJ_NAME=… sdj …"]
    C["3 · $SDJ_ENV_FILE"]
    D["4 · ./.env<br/>walking up to /"]
    E["5 · ~/.config/swissdevjobs-cli/.env"]
    F["direct-apply refuses<br/>run: sdj config --init"]

    A -->|"not set"| B
    B -->|"not set"| C
    C -->|"not set"| D
    D -->|"not set"| E
    E -->|"still not set"| F

    classDef win fill:#c7f0d8,stroke:#1a7f45,color:#0b3d22
    classDef mid fill:#dbe7ff,stroke:#2a5db0,color:#12233f
    classDef low fill:#f0f0f4,stroke:#8a8a99,color:#2a2a33
    classDef bad fill:#ffd6d6,stroke:#c0392b,color:#4a1210

    class A win
    class B mid
    class C,D,E low
    class F bad
```

Highest priority at the top. A project-local `.env` beats the global one, which is
handy if you keep a separate identity per job search.

<details>
<summary>Full variable reference</summary>

| variable | purpose |
|---|---|
| `SDJ_NAME` | applicant full name for `direct-apply` |
| `SDJ_EMAIL` | applicant email for `direct-apply` |
| `SDJ_CV` | default CV path, so you can omit `--cv` |
| `SDJ_ENV_FILE` | explicit `.env` location, checked first |
| `SDJ_CONFIG_DIR` | override `~/.config/swissdevjobs-cli` (cookie jar, `.env`) |
| `SDJ_CACHE_DIR` | override `~/.cache/swissdevjobs-cli` (SQLite database) |
| `SDJ_APPLICATIONS_LOG` | markdown application log to import on first run |
| `SDJ_JOBCLOUD_PAGES` | pages fetched per jobs.ch/jobup.ch search (default 5 → 100 rows/board) |

</details>

---

## Commands

```mermaid
flowchart TD
    START(["sdj"]) --> DISCOVER["🔍 Discover"]
    START --> ACT["✉️ Act"]
    START --> TRACK["📊 Track"]

    DISCOVER --> L["list<br/>search and filter the feed"]
    DISCOVER --> S["show<br/>full posting text"]
    DISCOVER --> T["tech<br/>most-wanted tech tags"]
    DISCOVER --> O["open<br/>posting in your browser"]

    ACT --> A["apply<br/>how do I apply to this one?"]
    ACT --> DA["direct-apply<br/>submit through the site's form"]
    ACT --> AU["auth<br/>clear a Cloudflare challenge"]

    TRACK --> AP["applications<br/>everything you have sent"]
    TRACK --> ST["stats<br/>cache and application counts"]
    TRACK --> CF["config<br/>resolved settings and paths"]

    classDef root fill:#8A63D2,stroke:#5b3fa0,color:#ffffff
    classDef group fill:#dbe7ff,stroke:#2a5db0,color:#12233f
    classDef leaf fill:#f5f6fa,stroke:#9aa0b5,color:#22262f

    class START root
    class DISCOVER,ACT,TRACK group
    class L,S,T,O,A,DA,AU,AP,ST,CF leaf
```

Every command takes `--json`.

<details open>
<summary><b>Discover</b></summary>

```sh
sdj list                                            # everything active
sdj list --tech Python --tech Kubernetes --remote   # any of those tags, remote/hybrid
sdj list --min-salary 130000 --location Zurich --sort salary
sdj list "platform engineer" --level Senior --visa  # free text + visa sponsorship
sdj list --company Google --include-applied         # include ones you've done

sdj show 686f2a1c57370f0152e4950e                   # by id
sdj show senior-platform-engineer-acme              # …or by slug, or a substring
sdj show acme --json                                # machine-readable

sdj open acme                                       # launch the posting
sdj boards                                          # every board, as data
sdj tech --limit 20                                 # what the market wants
```

`list --json` returns compact summary rows in an envelope — capped at 50
unless you pass `--limit` (`0` = uncapped), with `boards_searched` and a
coverage `note` when a search-driven board ran without a query. Empty fields
are omitted and salary is numeric (`salary_from`/`salary_to` + `currency`).
The pre-0.6 raw wire rows are still there behind `--raw`:

```console
$ sdj list "python" --board swissdevjobs --json --limit 2
{
  "total_in_feed": 187,
  "total_after_filters": 28,
  "hidden_already_applied": 3,
  "returned": 2,
  "boards_searched": ["swissdevjobs"],
  "jobs": [
    {
      "job_id": "6a8d83d641e56340faa426c0",
      "title": "Senior Solutions Engineer Real Estate | Data & BIM",
      "company": "Rockstar Recruiting AG",
      "city": "Zurich",
      "salary_from": 110000,
      "salary_to": 130000,
      "currency": "CHF",
      "workplace": "hybrid",
      "contract": ["permanent"],
      "language": "German",
      "technologies": ["BIM", "CAFM", "Embedded", "ERP", "Mobile", "Python", "AI"],
      "posted_at": "2026-08-25T12:00:22+00:00",
      "country": "ch",
      "source": "swissdevjobs",
      "url": "https://swissdevjobs.ch/jobs/Rockstar-Recruiting-AG-Senior-Solutions-Engineer-Real-Estate--Data--BIM"
    },
    …
  ]
}
```

`list` columns: **id · board · dates · title · company · city · salary · workplace · tags**

On jobs.ch/jobup.ch rows the salary column reads `—` (the platform publishes
none) and tags are usually empty — their coverage comes from server-side
`query` search, not client-side tag filters.

The date column carries two values, and the difference matters:

| | |
|---|---|
| `p=` | **posted** — real creation time, decoded from the ObjectId. Immutable. |
| `a=` | **active** — `activeFrom`, which the site re-stamps every time it bumps a listing back to the top |

A row reading `p=2026-04-02 a=2026-08-22` is a four-month-old job wearing a fresh coat
of paint. Sort by `--sort posted` (the default) to see through it.
</details>

<details open>
<summary><b>Act</b></summary>

```sh
sdj apply <id> --json                     # what route does this posting use?
sdj apply <id> --open                     # …and open the ATS while you're at it

sdj direct-apply <id> --motivation ./letter.txt
sdj direct-apply <id> --cv ./cv_de.pdf --motivation "Sehr geehrte Damen und Herren, …"
sdj direct-apply <id> --lang-skills fluent --not-eu

sdj apply <id> --complete email           # you sent it yourself — record it
sdj apply <id> --complete browser --notes "answered 3 screening questions"
```

`--motivation` takes inline text **or** a file path — it checks whether the string is
an existing file. The letter must not contain `<` or `>`; the site rejects them.
</details>

<details open>
<summary><b>Track</b></summary>

```sh
sdj applications                          # newest first
sdj applications --json --limit 500
sdj stats                                 # cached jobs, applications, db path
sdj config                                # identity + paths + which .env loaded
```
</details>

---

## How applying works

Three postings on the same board can need three completely different actions. `sdj apply`
tells you which, and `direct-apply` refuses the cases it knows would vanish.

```mermaid
flowchart TD
    START(["sdj apply JOB_ID"]) --> Q1{"redirectJobUrl points at<br/>talent.com or jometer?"}

    Q1 -->|yes| AGG["🚫 aggregator_posting<br/>exit code 2"]
    Q1 -->|no| Q2{"candidateContactWay?"}

    Q2 -->|"Email, with<br/>an address"| DIRECT["✅ direct<br/>the site forwards it"]
    Q2 -->|"CompanyWebsite,<br/>no address"| CW["🚫 company_website_posting<br/>exit code 2"]

    AGG --> BROWSER["🌐 Go apply on the ATS<br/>Recruitee · Workday · Greenhouse<br/>Lever · Personio · SmartRecruiters"]
    CW --> BROWSER

    DIRECT --> POST["POST /api/jobApply<br/>multipart: name, email,<br/>motivation, CV PDF"]
    POST --> OK{"HTTP 200?"}
    OK -->|yes| MARK["💾 recorded in SQLite<br/>hidden from future list"]
    OK -->|no| ERR["❌ raised with the response body"]

    BROWSER -.->|"after you submit"| COMPLETE["sdj apply JOB_ID --complete browser"]
    COMPLETE --> MARK

    classDef start fill:#8A63D2,stroke:#5b3fa0,color:#ffffff
    classDef