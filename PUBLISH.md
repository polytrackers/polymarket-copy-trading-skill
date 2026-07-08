# PUBLISH.md — human runbook for the PolyTrackers skill repo

This directory (`distribution/skill-repo/`) is the ready-to-push payload for the
public skill repository. A human with the right accounts pushes these files to a
fresh public repo under the `polytrackers` GitHub org and publishes to ClawHub.
The coding agent does **not** create GitHub orgs, push repos, or run `clawhub`.

## Suggested repo name and description

- **Repo name (keyword-rich):** `polymarket-copy-trading-skill`
  - Alternative: `polytrackers-agent-skill`
- **Repo description (mirror of README first line / SEO surface):**

  > Copy trading & paper trading for Polymarket — built for AI agents. MCP server, whale tracking, $10k mock wallets. Install: npx skills add polytrackers/polymarket-copy-trading-skill

## Files in this payload

- `SKILL.md` — Agent Skills spec skill (spec frontmatter + verbatim body of
  `public/skill.md`).
- `README.md` — SEO payload + install one-liners + MCP config blocks.
- `LICENSE` — MIT license for the public skill payload.
- `PUBLISH.md` — this runbook.

## Human prerequisites (do these first; some need lead time)

1. **Create the `polytrackers` GitHub org** and a dedicated machine account
   (fresh email) as its owner. Set org/member visibility to private; make commits
   with the machine account's `noreply` email only.
2. **ClawHub account age gate:** ClawHub requires the publishing GitHub account to
   be **≥ 1 week old**. Create the machine account immediately so it ages while the
   rest of this is prepared.
3. Install the CLIs you'll use:
   - `npx skills` (Agent Skills CLI) — no install needed, run via `npx`.
   - `clawhub` — run via `npx clawhub`.
4. Reserve/control the `@polytrackers` npm scope and publish
   `@polytrackers/mcp-stdio` before promoting stdio examples broadly. Verify
   ownership from a clean shell with `npm view @polytrackers/mcp-stdio`; the
   command must return PolyTrackers-controlled package metadata, not a 404 or
   third-party package.

## Publish steps

1. Create a new **public** repo under the `polytrackers` org using the name and
   description above.
2. Copy the contents of `distribution/skill-repo/` to the repo root:

   ```sh
   # from a checkout of this repo
   cp distribution/skill-repo/SKILL.md   /path/to/new-repo/SKILL.md
   cp distribution/skill-repo/README.md  /path/to/new-repo/README.md
   cp distribution/skill-repo/LICENSE    /path/to/new-repo/LICENSE
   cp distribution/skill-repo/PUBLISH.md /path/to/new-repo/PUBLISH.md
   ```

3. Commit and push as the machine account (noreply email).
4. **Validate the skill locally before publishing** (if `npx skills` supports a
   local install/validate in your environment):

   ```sh
   npx skills add /path/to/new-repo        # local path install / validation
   ```

   Confirm the skill's frontmatter (`name`, `description`, `compatible_agents`)
   parses and the skill is listed.

5. **Publish to ClawHub — dry run first:**

   ```sh
   npx clawhub skill publish ./ --dry-run \
     --slug polymarket-copy-trading \
     --name "Polymarket Copy Trading (Polytrackers)" \
     --version 1.0.0

   npx clawhub skill publish ./ \
     --slug polymarket-copy-trading \
     --name "Polymarket Copy Trading (Polytrackers)" \
     --version 1.0.0
   ```

6. Confirm the install one-liner works from a clean machine:

   ```sh
   npx skills add polytrackers/polymarket-copy-trading-skill
   npx clawhub install polymarket-copy-trading
   ```

Once installs happen, the skill auto-lists on skills.sh (no submission step) and
gets scraped into aggregator directories.

## Keeping SKILL.md in sync (IMPORTANT)

`SKILL.md` in this payload is generated so that its **frontmatter differs** from
`public/skill.md` (Agent Skills spec fields) but its **body is byte-identical**
to the body of `public/skill.md`. A unit test enforces this:
`app/api/mcp/catalog-skill-sync.test.ts` (`distribution SKILL.md body stays in
sync with public/skill.md`). Editing `public/skill.md` without regenerating this
copy fails CI.

To regenerate after any change to `public/skill.md`, run from the repo root:

```sh
node -e '
const fs = require("fs");
const raw = fs.readFileSync("public/skill.md", "utf8");
const m = raw.match(/^---\n[\s\S]*?\n---\n/);
if (!m) throw new Error("public/skill.md has no frontmatter");
const body = raw.slice(m[0].length);
const frontmatter =
  "---\n" +
  "name: polytrackers-agent-skill\n" +
  "description: >-\n" +
  "  Copy trading and paper (mock) trading for Polymarket, built for AI agents.\n" +
  "  Use this skill for Polymarket market intelligence, whale tracking, anomaly\n" +
  "  detection, copy-trading review, and $10K mock/paper-trading experiments via\n" +
  "  the PolyTrackers MCP server and REST API. Covers Agent API Key setup,\n" +
  "  tier/scope guidance, trade preflight safety, and mock wallet workflows.\n" +
  "compatible_agents:\n" +
  "  - claude\n" +
  "  - cursor\n" +
  "  - codex\n" +
  "  - openclaw\n" +
  "metadata:\n" +
  "  stable_url: https://polytrackers.com/skill.md\n" +
  "  source: public/skill.md\n" +
  "---\n";
fs.writeFileSync("distribution/skill-repo/SKILL.md", frontmatter + body);
console.log("regenerated distribution/skill-repo/SKILL.md");
'
```

Then re-run the sync test:

```sh
pnpm run test:unit
```

If you change the frontmatter block above, keep it in step with the frontmatter
assertions in `app/api/mcp/catalog-skill-sync.test.ts`.
