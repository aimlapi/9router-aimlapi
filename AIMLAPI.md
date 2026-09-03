# aimlapi.com and 9Router — position note

**Status: do not open an upstream provider PR.** This note exists instead of one.

This is a fork-only document. It records what was found in `decolua/9router` on
2026-09-03, why no provider entry is being proposed upstream, and what the entry
would look like if that decision is ever revisited. Everything below is checked
against the repository and against a live API, with the commands and numbers
given so it can be re-run.

---

## 1. Summary

`aimlapi` is not a hidden provider in a private build. It is a **deleted
provider** whose only surviving trace is an orphaned test-results file.

The upstream repository shipped a real, working AI/ML API provider — base URL,
model list and dashboard entry — from **2026-05-12 to 2026-06-14**. It was
removed on 2026-06-14 in a single registry-consolidation commit that also removed
**28 other providers**. What remains today is one frozen JSON artifact that no
code, test or CI step reads.

The recommendation is to leave it removed. Reasons in §5 and §6.

---

## 2. What is in the repository today

A repo-wide search for `aimlapi` returns **two** occurrences, both inside one
file, `tests/__baseline__/baseline-results.json` — a single-line, 239,574-byte
Vitest JSON report:

```
"fullName":"coverage: every model translates without throwing 'aimlapi': all models OpenAI→target",
"title":"'aimlapi': all models OpenAI→target","status":"passed"
```

Nothing else in the tree mentions us: no registry file, no base URL, no model id,
no header, no documentation line.

**That file is orphaned.** `grep -rn "baseline-results" .` outside the file
itself returns nothing — no script, test or workflow reads it. The regression
gate the repo actually uses, `tests/__baseline__/verify-no-regression.mjs`, takes
a results path on `argv` and compares against `tests/__baseline__/known-fails.txt`,
which does not mention us either. The file was committed once, in
`9532ec804e747c2f61d3d525f3b761b122de1491` (2026-06-13T09:13:40Z), 68 seconds
after the run it records started, and has never been modified since — the blob is
still the same 239,574 bytes it was at that commit.

The test name is generated, not hand-written. `tests/translator/coverage-all-models.test.js`
does `it.each(buildProviderGroups())("$alias: all models OpenAI→target", …)`, and
`buildProviderGroups()` in `tests/translator/matrix.js` iterates the keys of
`PROVIDER_MODELS`. So the string is direct evidence that on 2026-06-13 the tree
that produced it had an `aimlapi` provider carrying at least one chat model — and
nothing more than that.

The suite paths inside the file read `/Users/Working/router4/app/tests/…`. That is
the maintainer's local checkout path for **this** repository, not a separate
private one: the same paths appear in every committed run, and the commits that
carry them are ordinary public commits on `master`.

---

## 3. What was in the repository before

At `1432ac61d713cf2c884bd8f2ce3d4a43b7632870` (2026-06-13) the provider existed
in three places.

**Transport** — `open-sse/config/providers.js:393`:

```js
aimlapi: { baseUrl: "https://api.aimlapi.com/v1/chat/completions" },
```

**Models** — `open-sse/config/providerModels.js:661-667`:

```js
aimlapi: [
  { id: "gpt-4o", name: "GPT-4o" },
  { id: "gpt-4o-mini", name: "GPT-4o Mini" },
  { id: "claude-3-5-sonnet-20241022", name: "Claude 3.5 Sonnet" },
  { id: "gemini-2.0-flash-exp", name: "Gemini 2.0 Flash" },
  { id: "meta-llama/Meta-Llama-3.1-70B-Instruct-Turbo", name: "Llama 3.1 70B" },
],
```

**Dashboard entry** — `src/shared/constants/providers.js:122`, commented out,
under this block header at `:119-120`:

```js
// === Free-tier LLM providers (synced from OmniRoute) — DISABLED in UI ===
// Uncomment to re-enable. Backend config (PROVIDERS, PROVIDER_MODELS, ALIAS_TO_PROVIDER_ID) remains active.
// aimlapi: { id: "aimlapi", alias: "aiml", name: "AI/ML API", icon: "hub", color: "#6366F1", textIcon: "AI", website: "https://aimlapi.com", notice: { text: "$0.025/day free — 200+ models (GPT-4o, Claude, Gemini, Llama) via single endpoint.", apiKeyUrl: "https://aimlapi.com/app/keys" }, passthroughModels: true, serviceKinds: ["llm", "image"] },
```

Three things follow from that block header, and they matter more than the entry
itself:

- It was **bulk-imported from a competitor's repository**, not written for us.
  OmniRoute is a separate aggregator; its own AI/ML API integration is broken in
  a way we have logged elsewhere. Our entry here was copied along with a batch of
  other free-tier hosts.
- It was **never visible to a user**. The line is commented out in the earliest
  version that has it (`8f4d29caa4e36a1fa906e20ba7ebf380e021e6a3`, 2026-05-12)
  and in every version after, up to removal. By the maintainer's own comment the
  backend routing stayed live, so `aiml/gpt-4o` was reachable by API — but the
  provider never appeared in the dashboard.
- It carried **wrong information about us**. `$0.025/day free — 200+ models` is
  not a claim we make, and three of the five model ids are not routable today
  (§4).

**Removal.** Present at `b33cbb0280073a65db26697b682ee6174189528f`
(2026-06-13T03:54:51Z), absent at
`bb9e9aa91f16ba13c1b9c4b47f82baadfe222b0c` (2026-06-14T06:15:48Z),
*"refactor(open-sse): registry consolidation + DRY media/oauth/adhoc cleanup"*.

That commit is where the whole imported batch went. Comparing the provider groups
in the last committed run that has us (102 groups) against the next one that does
not (73 groups) gives **29 providers removed and 0 added**:

```
agentrouter, ai21, aimlapi, baseten, bazaarlink, bytez, completions, deepinfra,
enally, freetheai, glhf, inference-net, kluster, lepton, llm7, longcat, modal,
morph, nlpcloud, nous-research, novita, nscale, predibase, publicai, puter, reka,
sambanova, scaleway, uncloseai
```

None of the 29 is in the registry today. This was a deliberate cull of the
long tail, not a decision about us.

---

## 4. What the deleted entry got wrong about our API

Checked against `GET https://api.aimlapi.com/v1/models?include=all` on
2026-09-03 (936 entries, 785 unique ids, 353 chat models), using the
id-**or**-alias test:

| Deleted id | id | alias | Verdict |
| --- | --- | --- | --- |
| `gpt-4o` | no | yes | routable |
| `gpt-4o-mini` | no | yes | routable |
| `claude-3-5-sonnet-20241022` | no | no | **not in catalog** |
| `gemini-2.0-flash-exp` | no | no | **not in catalog** |
| `meta-llama/Meta-Llama-3.1-70B-Instruct-Turbo` | no | no | **not in catalog** |

The base URL was correct and still is. The `$0.025/day free — 200+ models`
notice was stale marketing copy carried in from OmniRoute; the catalog publishes
353 chat models today.

---

## 5. Merge appetite, measured

All figures from the GitHub API on 2026-09-03, not from a summary.

| Measure | Value |
| --- | --- |
| Open PRs | **863** |
| Merged PRs, all time | 203 |
| Closed unmerged | 856 |
| Open issues | 1,049 |
| Last push to `master` | 2026-09-03 (active) |

Merges by month: **Feb 47 · Mar 44 · Apr 37 · May 74 · Jun 0 · Jul 1 · Aug 0 · Sep 0.**

- Merges in the last 6 months (since 2026-03-03): **152**.
- Merges in the last 3 months: **1** — PR #2581, a Thai translation, merged
  2026-07-16. Nothing since; that is **49 days** with an 863-PR queue.
- The last merge of a *code* PR was #1576 on 2026-05-31 — **95 days**.

**Outside PRs do get merged here — that is not the blocker.** Of the 152 merges
since March, 149 are `CONTRIBUTOR` and 3 are `NONE`; zero are `OWNER` or
`MEMBER`, because the maintainer pushes directly rather than through PRs. Every
merge in the window was somebody else's. Provider additions merged as recently as
2026-05-17:

| PR | Author | Merged | Size |
| --- | --- | --- | --- |
| #1183 Vercel AI Gateway | `newnol` | 2026-05-17 | 7 files, +20 |
| #1143 blackbox provider | `anhvandev` | 2026-05-15 | 2 files, +3 |
| #741 Volcengine Ark | `kenlin8827` | 2026-04-24 | 9 files, +49 |

The queue itself: 863 open PRs from **408 distinct authors**, 467 `CONTRIBUTOR` /
396 `NONE`, 17 drafts, opened Jan 1 · Feb 6 · Mar 25 · Apr 51 · May 150 · Jun 132
· Jul 214 · Aug 269 · Sep 15. The oldest has been open since 2026-01-06. By title,
roughly 505 are `fix:` and 242 `feat:`; about 162 mention a provider, gateway or
API addition.

So the door is not closed by policy. It has simply stopped opening, while intake
accelerated — 269 PRs arrived in August and none were merged.

**There is already an unanswered request for us.** Issue
[#1739](https://github.com/decolua/9router/issues/1739), *"Provider request: AIML
API preset for OpenAI-compatible models"*, opened 2026-06-08 by an outside user,
proposing `baseURL: "https://api.aimlapi.com/v1"`. It has **zero comments** and
has not been touched in 87 days. It predates neither our addition (2026-05-12)
nor explains our removal (2026-06-14) — it sits between them, unacknowledged. No
pull request in any state mentions `aimlapi`.

---

## 6. Contribution gates, and two that are broken

There is no `CONTRIBUTING.md`, no PR template and no issue template; `.github/`
holds only `dependabot.yml` and two workflows. Licence is MIT. The de-facto rules
are `CLAUDE.md` and `open-sse/AGENTS.md`.

The documented way to add a provider (`open-sse/AGENTS.md:30`):

> **Provider**: copy `providers/REGISTRY_TEMPLATE.js` → `providers/registry/{id}.js`; add models to `config/providerModels.js`. Generic providers need no executor (DefaultExecutor handles OpenAI-compatible APIs).

And `CLAUDE.md:49`:

> Regression baselines: `tests/__baseline__/verify-*.mjs` compare against committed snapshots (providers, aliases, OAuth URLs). Run these after touching provider registry / alias logic.

Both were exercised. Two problems an outside contributor hits immediately:

**(a) The provider baseline must be refreshed by hand.** With a candidate
`registry/aimlapi.js` in place, `node tests/__baseline__/verify-providers.mjs`
exits 1:

```
❌ PROVIDERS mismatch (1 field diffs):
  + provider added: aimlapi
```

That is working as designed — a provider PR must also re-run
`snapshot-providers.mjs` and commit `providers-baseline.json`. Worth knowing
before writing the diff.

**(b) The regression gate cannot pass outside the maintainer's directory
layout.** `verify-no-regression.mjs` builds each test key as
`f.name.split("/app/")[1] + " :: " + a.fullName`. Suite names are absolute paths,
so this only yields a usable key when the checkout sits under a directory named
`app` — as the maintainer's `/Users/Working/router4/app/` does. On a clean
checkout anywhere else every key becomes the literal string `undefined`, nothing
matches `known-fails.txt`, and all 85 baseline failures are reported as
regressions:

```
❌ REGRESSION: 85 test pass→fail:
  - undefined :: OpenAI → Claude context mapping assistant reasoning_content becomes a thinking block
  …
```

Rewriting the paths to `/Users/Working/router4/app/` makes the keys resolve, and
the gate still fails: `known-fails.txt` lists 24 entries, only **14** of which
match the 85 current failures, so it reports **71** regressions on a pristine,
unmodified `master`. The repo's own stated bar for judging a change is therefore
unreachable by anyone but the maintainer. A provider PR here cannot demonstrate
that it broke nothing using the tool the repo tells it to use.

**(c) House style on attribution headers points the other way.** `transport.headers`
is a real, per-request seam — `BaseExecutor.buildHeaders()` spreads
`this.config.headers` into a fresh object and sets `Authorization` afterwards, so
custom headers can neither leak between providers nor clobber auth. But the one
provider that uses it, `open-sse/providers/registry/openrouter.js`, deliberately
**anonymises the host**:

```js
headers: {
  "HTTP-Referer": "https://endpoint-proxy.local",
  "X-Title": "Endpoint Proxy",
},
```

9Router does not identify itself to upstream providers. A header block naming the
host project would be the first of its kind here and would read as out of place.

---

## 7. Position

**Do not send a provider PR upstream.** Four independent reasons, in order of
weight:

1. **The removal was deliberate and recent.** Re-adding `aimlapi` asks the
   maintainer to reverse a named consolidation commit that dropped 29 providers
   at once. That is a product decision about the size of the long tail, not an
   oversight, and it is not ours to relitigate by pull request.
2. **The repo is not merging.** 95 days without a code merge, 863 PRs queued,
   269 of them from August alone. A correct, small PR would join the back of that
   queue with no realistic path out of it.
3. **9Router is a competing router**, and its stated purpose inverts the usual
   value of being listed. `CLAUDE.md:7` describes it as a gateway that *"routes
   traffic across 40+ upstream providers"*, and the README's pitch is unlimited
   free coding with *"auto-fallback to FREE & cheap AI models"*. Its users are
   being routed toward zero-cost providers by design.
4. **An unanswered issue already exists.** #1739 asks for exactly this and has
   sat untouched for 87 days. Opening a PR over a request the maintainer has not
   engaged with spends our first impression for nothing. If anyone acts here, the
   cheap move is a comment on #1739, from a person, not a pull request.

What *would* change the picture: the PR queue starting to drain, or any response
on #1739.

---

## 8. If it is ever revisited

The entry below was written against `REGISTRY_TEMPLATE.js` and the `tokenrouter`
peer, then **verified end-to-end** — every model called live through
`handleChatCore`, the repo's own production request path, on 2026-09-03. It is
recorded here rather than committed as code, because §7 says it should not ship.

```js
// open-sse/providers/registry/aimlapi.js
export default {
  id: "aimlapi",
  alias: "aiml",
  aliases: ["aimlapi"],
  uiAlias: "aiml",
  display: {
    name: "aimlapi.com",
    icon: "hub",
    color: "#6366F1",
    textIcon: "AI",
    website: "https://aimlapi.com",
    notice: {
      text: "OpenAI-compatible gateway. 353 chat models (GPT, Claude, Gemini, Llama, DeepSeek, Qwen, GLM).",
      apiKeyUrl: "https://aimlapi.com/app/keys",
    },
  },
  category: "apikey",
  authType: "apikey",
  authModes: ["apikey"],
  transport: {
    baseUrl: "https://api.aimlapi.com/v1/chat/completions",
    validateUrl: "https://api.aimlapi.com/v1/models",
    headers: {
      "HTTP-Referer": "https://github.com/decolua/9router",
      "X-Title": "9Router",
      "X-AIMLAPI-Source": "agent/9router",
      "X-AIMLAPI-Partner-ID": "part_9router",
    },
  },
  models: [
    { id: "openai/gpt-5-5", name: "GPT-5.5" },
    { id: "openai/gpt-4o-mini", name: "GPT-4o Mini" },
    { id: "anthropic/claude-sonnet-4.5", name: "Claude 4.5 Sonnet" },
    { id: "google/gemini-2.5-flash", name: "Gemini 2.5 Flash" },
    { id: "meta-llama/Llama-3.3-70B-Instruct-Turbo", name: "Llama 3.3 70B Instruct Turbo" },
  ],
  serviceKinds: ["llm"],
  modelsFetcher: { url: "https://api.aimlapi.com/v1/models", type: "openai" },
};
```

Notes on the shape:

- The machine id stays `aimlapi` and the short alias `aiml`, matching what the
  repo already shipped and what LiteLLM uses. Only `display.name` is the
  user-facing label, and it is exactly `aimlapi.com`. Renaming the id would break
  any config that still carries it.
- The five model ids all pass the id-or-alias check **and** were each called
  live. Catalog membership alone is not sufficient evidence in either direction —
  our catalog both omits ids that work and lists at least one that does not.
- No `/v1/completions` path is declared: that endpoint does not exist on our API
  and returns 404.
- `modelsFetcher` uses `/v1/models`, which is public. It must not be used to
  validate a key: it returns 200 for a bogus key, or none at all.
- Adding this file also requires an entry in the auto-generated
  `registry/index.js` and a refreshed `providers-baseline.json` (§6a). Note the
  generated index reuses identifier numbers — `p123` is already taken by
  `ollama-search.js` — so a hand-added import must pick a fresh name or the
  module fails to parse.

### Verification performed

Environment: Node v26.7.0, macOS. Key read from a file and passed by env; never
written to disk or logged.

**Test baseline, pristine `master`, before any change**
(`cd tests && npx vitest run`): **2,084 tests — 1,940 passed, 85 failed, 59
skipped**; 653 suites, 592 passed / 61 failed. The failures are expected;
`CLAUDE.md:43` states *"The suite is NOT expected to be all-green on a plain
checkout."*

**Live calls** through `handleChatCore({ modelInfo: { provider: "aimlapi", … } })`
with the candidate entry loaded — all HTTP 200:

```
openai/gpt-5-5                            200  served=gpt-5.5-2026-04-23         content="OK"
openai/gpt-4o-mini                        200  served=gpt-4o-mini-2024-07-18     content="OK"
anthropic/claude-sonnet-4.5               200  served=anthropic/claude-sonnet-4.5 content="OK"
google/gemini-2.5-flash                   200  served=google/gemini-2.5-flash    content="OK"
meta-llama/Llama-3.3-70B-Instruct-Turbo   200  served=meta-llama/Llama-3.3-70B-Instruct-Turbo content="OK"
tool call (openai/gpt-4o-mini)            200  get_weather({"city":"Paris"})
```

Note that the `model` echo does not always match the id requested —
`openai/gpt-5-5` answers as `gpt-5.5-2026-04-23` and `openai/gpt-4o-mini` as
`gpt-4o-mini-2024-07-18`. A host that pins "the model that served the request"
should expect that.

**Null-parameter trap.** Sending `tools: null` through the same path returns
**400**, with the offending field named in `error.details[]`:

```json
{ "path": "tools", "reason": "Expected array, received null", "code": "invalid_type" }
```

Any integration must build request params by *omitting* unset optional keys, not
by passing `null`. This is the failure mode that passes every unit test and then
400s on turn 2 of an agent loop, when tools are cleared between turns.
