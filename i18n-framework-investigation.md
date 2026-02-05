# i18n Framework Investigation for WasmEdge Docs

## Current State

This repository is built with **Docusaurus v2.4.0**, which has **built-in i18n support** already partially configured:

- **Config**: `docusaurus.config.js` declares 3 locales: `en`, `zh`, `zh-TW`
- **English docs**: 194 Markdown files in `docs/`
- **Chinese (Simplified) translations**: ~192 translated files in `i18n/zh/docusaurus-plugin-content-docs/current/`
- **Theme translations**: `navbar.json` and `footer.json` exist but are **not actually translated** (still contain English values)
- **`zh-TW` locale**: Declared in config but has **no `localeConfigs` entry** and **no translation files**
- **Language switcher**: Already enabled in the navbar via `localeDropdown`
- **Search**: `@easyops-cn/docusaurus-search-local` supports both `en` and `zh` with `@node-rs/jieba` for Chinese segmentation

### Key Observation

The Docusaurus built-in i18n system is already the "framework" in use. It is file-system based and designed to be **decoupled from any translation SaaS**. The real question is: **which Translation Management System (TMS) or workflow tool should be layered on top** to manage ongoing translations at scale.

---

## Frameworks / Tools Evaluated

### 1. Crowdin (Recommended by Docusaurus)

**What it is**: A cloud-based Translation Management System (TMS) with official Docusaurus integration.

| Aspect | Details |
|---|---|
| **Docusaurus support** | Official integration, documented at [docusaurus.io/docs/i18n/crowdin](https://docusaurus.io/docs/i18n/crowdin) |
| **Cost** | Free for qualifying open-source projects (unlimited projects, strings, members) |
| **File format support** | Markdown, MDX, JSON — all Docusaurus formats natively supported |
| **Workflow** | Upload sources via CLI/API, translators work in web UI, download translations back |
| **CI/CD** | `crowdin.yml` config + CLI for automated sync; can download translations at build time |
| **Git integration** | GitHub/GitLab/Bitbucket integrations; can auto-sync via branches |
| **Translation features** | Translation Memory, Glossary, Machine Translation, in-context editing, progress tracking |
| **Community features** | Invite community translators, track individual contributions |
| **Notable users** | Jest, Docusaurus itself, ReasonML, many CNCF projects |

**Pros**:
- First-class Docusaurus support with well-documented setup
- Free for open-source projects (WasmEdge likely qualifies as a CNCF sandbox project)
- Handles Markdown/MDX files natively, preserving formatting and links
- Web-based translation UI — non-developer translators can contribute
- Built-in Translation Memory reduces repeated work across similar docs
- Can keep translations out of Git (download at build time) or sync them in

**Cons**:
- SaaS dependency (translations hosted on Crowdin's servers)
- Open-source license requires contributing translations to Global Translation Memory
- Recent Markdown format changes caused link translation issues (links treated as plain text)
- Concurrent upload/download operations not well supported
- Quota limits have been reported even for OSS projects in edge cases

---

### 2. Weblate

**What it is**: An open-source, self-hostable web-based translation platform with Git integration.

| Aspect | Details |
|---|---|
| **Docusaurus support** | No official integration; requires manual setup |
| **Cost** | Free if self-hosted; paid cloud plans available |
| **File format support** | Markdown (since v5.0, still developing), JSON, PO/gettext |
| **Workflow** | Translations committed directly to your repo via PRs |
| **CI/CD** | Tight VCS integration; translations are Git commits |
| **Git integration** | Native — treats the repo as single source of truth |
| **Translation features** | Translation Memory, Glossary, consistency checks, automatic suggestions |
| **Self-hosting** | Yes, fully open source (GPLv3) |

**Pros**:
- Fully open-source and self-hostable — full data ownership
- Deep Git integration — translations come back as real commits/PRs
- Strong community and active development
- No vendor lock-in

**Cons**:
- **No turnkey Docusaurus integration** — significant custom glue work needed
- Markdown support is still developing (since v5.0); can't reliably round-trip translations
- May need intermediate tools like `po4a` to convert MD to PO format
- Self-hosting adds operational overhead
- Weblate is source-of-truth for translations (won't import changes from translated files)

---

### 3. GitLocalize

**What it is**: A GitHub-native translation management tool that uses PRs for translation delivery.

| Aspect | Details |
|---|---|
| **Docusaurus support** | Used by Yew project (Docusaurus v2 site) |
| **Cost** | Free for public repositories |
| **File format support** | Markdown, JSON |
| **Workflow** | Tracks file changes in repo, translators work in web UI, submits PRs |
| **CI/CD** | PR-based — fits naturally into GitHub workflows |
| **Git integration** | GitHub-native; repo is single source of truth |
| **Translation features** | Change tracking, segment-by-segment comparison, web editor |

**Pros**:
- GitHub-native workflow — translations arrive as PRs
- Free for public repos
- Tracks source file changes and highlights what needs re-translation
- Low barrier to entry for GitHub-familiar contributors
- Real-world Docusaurus usage (Yew project)

**Cons**:
- Less feature-rich than Crowdin (no Translation Memory, limited glossary)
- Smaller community and fewer integrations
- Less mature than alternatives
- Limited automation capabilities

---

### 4. Transifex

**What it is**: An enterprise-grade cloud TMS with API-first design.

| Aspect | Details |
|---|---|
| **Docusaurus support** | Via file format support (no dedicated integration) |
| **Cost** | Free tier for OSS; paid starts at ~$70/month |
| **File format support** | Markdown, JSON, YAML, and many others |
| **Workflow** | CLI/API sync, web-based translation editor |
| **CI/CD** | CLI + API for pipeline integration |
| **Git integration** | Via CLI/API (not as deep as Weblate) |
| **Translation features** | Translation Memory, Glossary, in-context review, MT |

**Pros**:
- Mature platform with strong API
- Free tier for open-source
- In-context review for non-technical translators
- Good CI/CD integration

**Cons**:
- Paid tier is expensive ($70/mo doesn't include Glossary, Source editing)
- No dedicated Docusaurus integration
- Cannot be self-hosted
- Overkill for a documentation-only project

---

### 5. AI-Powered Tools (docusaurus-i18n / Dicodocus)

**What they are**: CLI/editor tools that use LLMs (OpenAI) to auto-translate Docusaurus docs.

| Tool | Description |
|---|---|
| **[docusaurus-i18n](https://github.com/moonrailgun/docusaurus-i18n)** | CLI tool: `npx docusaurus-i18n` auto-translates all docs using OpenAI API |
| **[Dicodocus](https://dicodocus.com/)** | Open-source Docusaurus editor with one-click AI translation and missing translation analyzer |

**Pros**:
- Extremely fast initial translation of entire doc sets
- Low setup overhead — just needs an API key
- Can be used as a complement to any TMS (generate drafts, then human-review)
- Good for bootstrapping translations for new locales (e.g., `zh-TW`)

**Cons**:
- AI translations require human review for technical accuracy
- Ongoing API costs (OpenAI token usage)
- No community translation workflow
- No Translation Memory or Glossary
- Not a replacement for a TMS — best used as a supplement

---

### 6. Pure Git Workflow (Manual)

**What it is**: Docusaurus's built-in approach — manage translations as files in the repo, no external tool.

**Pros**:
- Zero external dependencies
- Full control over translation files
- Already partially in use in this repo

**Cons**:
- No tooling to track what's changed and needs re-translation
- No web UI for non-developer translators
- Doesn't scale well as doc count and language count grow
- Easy for translations to drift out of sync with source

---

## Comparison Matrix

| Criteria | Crowdin | Weblate | GitLocalize | Transifex | AI Tools | Git (Manual) |
|---|---|---|---|---|---|---|
| **Docusaurus integration** | Official | None | Community | None | Dedicated | Built-in |
| **Free for OSS** | Yes | Yes (self-host) | Yes | Limited | API costs | Yes |
| **Setup complexity** | Low | High | Low | Medium | Very Low | None |
| **Markdown/MDX support** | Native | Developing | Yes | Yes | Yes | N/A |
| **Community translation** | Excellent | Good | Good | Good | None | Poor |
| **Translation Memory** | Yes | Yes | No | Yes | No | No |
| **Change tracking** | Yes | Yes | Yes | Yes | No | Manual |
| **Self-hostable** | No | Yes | No | No | N/A | N/A |
| **CI/CD integration** | Strong | Strong | PR-based | Strong | Script-based | Manual |
| **Non-dev translator UX** | Excellent | Good | Good | Excellent | N/A | Poor |
| **Operational overhead** | Low (SaaS) | High (self-host) | Low | Low | Low | Medium |

---

## Recommendation

### Primary: **Crowdin** (Translation Management)

For WasmEdge docs, **Crowdin** is the strongest choice because:

1. **Official Docusaurus support** — the only TMS with documented, first-class integration
2. **Free for open-source** — WasmEdge (CNCF project with public repo) should qualify
3. **Handles the existing setup** — works directly with the `i18n/` directory structure already in place
4. **Community-friendly** — web UI enables non-developer contributors to translate
5. **Proven at scale** — used by Docusaurus itself, Jest, and many CNCF-adjacent projects
6. **Low migration effort** — existing `zh` translations can be uploaded as-is

### Supplement: **AI tools** (for bootstrapping)

Use `docusaurus-i18n` or similar to:
- Generate initial `zh-TW` translations from existing `zh` docs
- Quickly produce draft translations for new English docs
- These drafts should then be reviewed by human translators (in Crowdin or via PR)

### Implementation Steps

1. **Fix existing config issues**: Add `zh-TW` to `localeConfigs`, translate `navbar.json` and `footer.json`
2. **Apply for Crowdin OSS license**: [crowdin.com/page/open-source-project-setup-request](https://crowdin.com/page/open-source-project-setup-request)
3. **Set up `crowdin.yml`** in the repo root with source/translation file mappings
4. **Upload existing translations** to Crowdin to establish baseline
5. **Integrate with CI/CD**: Add Crowdin sync step before builds
6. **Bootstrap `zh-TW`**: Use AI tools to generate drafts, upload to Crowdin for review
7. **Invite community translators** via Crowdin's contributor system

---

## Sources

- [Docusaurus i18n Introduction](https://docusaurus.io/docs/i18n/introduction)
- [Docusaurus i18n - Using Crowdin](https://docusaurus.io/docs/i18n/crowdin)
- [Docusaurus i18n - Using Git](https://docusaurus.io/docs/i18n/git)
- [Crowdin Open Source License Request](https://crowdin.com/page/open-source-project-setup-request)
- [Crowdin Pricing](https://crowdin.com/pricing)
- [Crowdin Docusaurus Integration](https://store.crowdin.com/docusaurus)
- [Weblate Markdown Support](https://docs.weblate.org/en/weblate-5.10.3/formats/markdown.html)
- [GitLocalize](https://gitlocalize.com/)
- [docusaurus-i18n (AI auto-translate)](https://github.com/moonrailgun/docusaurus-i18n)
- [Dicodocus (AI-powered Docusaurus editor)](https://dicodocus.com/)
- [Docusaurus v2 i18n RFC](https://github.com/facebook/docusaurus/issues/3317)
