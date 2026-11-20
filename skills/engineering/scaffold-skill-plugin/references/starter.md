# Starter files and verification

Use these examples for a new bucketed, single-plugin repository. Replace the sample identity and initial capability with the user's actual project. Keep the target's existing conventions when extending a repository.

## Smallest useful tree

```text
.claude-plugin/
  plugin.json
  marketplace.json
.github/workflows/check.yml
AGENTS.md
README.md
package.json
package-lock.json
scripts/check-plugin.mjs
skills/engineering/
  README.md
  explain-diff/
    SKILL.md
    agents/openai.yaml
docs/engineering/explain-diff.md
```

Create other buckets when they have content. Add release automation, a canonical install reference, and a router when they earn their maintenance cost. Keep all runtime material inside the skill directory, since the host may install only selected skill directories.

## Manifests

`.claude-plugin/plugin.json`:

```json
{
  "name": "team-skills",
  "version": "0.1.0",
  "description": "Shared engineering practices for the team",
  "author": { "name": "Team Tools Maintainers" },
  "skills": ["./skills/engineering/explain-diff"]
}
```

`.claude-plugin/marketplace.json`:

```json
{
  "name": "team-tools",
  "description": "Agent skills maintained by the team",
  "owner": { "name": "Team Tools Maintainers" },
  "plugins": [
    {
      "name": "team-skills",
      "source": "./",
      "description": "Shared engineering practices for the team"
    }
  ]
}
```

Plugin `name` and marketplace `name` have different jobs. The installation identifier is `team-skills@team-tools`. Replace both maintainer names with the actual owner; strict validation warns when author metadata or the marketplace description is missing. Add repository, license, and homepage only when their real values are known. Promoted paths are explicit directories; an array entry of `./skills/` would erase the lifecycle boundary.

For a new plugin's installation section, document registering its own repository as a marketplace followed by installing `<plugin-name>@<marketplace-name>`. Check the exact syntax in the user's CLI. An official marketplace listing requires separate acceptance. Keep automatic update claims specific to the tested agent and settings.

## Initial skill

`skills/engineering/explain-diff/SKILL.md`:

```markdown
---
name: explain-diff
description: Explain a git diff in terms of changed behaviour, evidence, and unresolved risks. Use when a user wants to understand what a patch changes.
---

# Explain Diff

Establish which diff the user means from their request and git state. Read the changed code and the callers needed to explain its observable effects.

Explain the problem it addresses, the resulting behaviour, and the evidence available from tests or local execution. Distinguish observed results from assumptions. Cite the relevant files and identify any uncertainty that changes the interpretation.

Done when the reader can identify the behavioural changes and the remaining uncertainty without reconstructing the diff themselves.
```

`agents/openai.yaml`:

```yaml
interface:
  display_name: "Explain Diff"
  short_description: "Explain behaviour and evidence in a git diff"
```

This is a functional example, not a mandatory first skill. Replace it with the requested capability. Use normal YAML parsing to check both files. Quote descriptions containing `: ` or other YAML indicator characters.

## Repository check

Copy the skill's `assets/check-plugin.mjs` into the target `scripts/` directory. It uses Node.js and the `yaml` package. For a new project, a minimal `package.json` is:

```json
{
  "name": "team-skills",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "scripts": {
    "check-plugin": "node scripts/check-plugin.mjs"
  },
  "devDependencies": {
    "yaml": "^2.9.1"
  }
}
```

Install dependencies and commit the resulting lockfile. In CI, install with `npm ci` and run `npm run check-plugin`. Add the platform validator in an environment with the target CLI installed. Do not commit a workflow that assumes an unavailable command exists on a clean runner.

The checker enforces this reference's conventions: parseable skill metadata, matching invocation policies, a unique global skill namespace, exact promoted-set membership, contained real paths, linked indexes, promoted docs pages, and package/plugin version equality. It is not a replacement for the platform validator or a runtime discovery test.

## Version authority

If `package.json` owns versions, a small `scripts/sync-plugin-version.mjs` can read that version and update `plugin.json` as part of the chosen release command:

```js
import { readFileSync, writeFileSync } from "node:fs";

const packageFile = new URL("../package.json", import.meta.url);
const pluginFile = new URL("../.claude-plugin/plugin.json", import.meta.url);
const { version } = JSON.parse(readFileSync(packageFile, "utf8"));
const plugin = JSON.parse(readFileSync(pluginFile, "utf8"));
plugin.version = version;
writeFileSync(pluginFile, JSON.stringify(plugin, null, 2) + "\n");
```

For Changesets, use `changeset version && node scripts/sync-plugin-version.mjs` as the version command, with private package versioning enabled. A release PR can then update the package, plugin manifest, lockfile, and changelog together. CI's checker detects drift before merge.

## Verification boundaries

1. Run `npm run check-plugin`: prove repository relationships and version equality.
2. Run `claude plugin validate . --strict`: prove the installed platform validator accepts the packaging. Capture warnings as well as errors. Root steering-file warnings can appear even when JSON fields are valid; fix a new scaffold or report a pre-existing target limitation accurately.
3. Use `claude --plugin-dir <absolute-plugin-directory>` for a session-local smoke test where supported. In that session, invoke a promoted skill such as `/team-skills:explain-diff` and inspect the available skills. Keep a temporary beta fixture outside the manifest and check that it is absent from discovery.
4. For every additional target agent, verify its supported loading method and invocation syntax before testing. Record CLI versions and results. A manifest accepted by one agent is not evidence that another agent loaded it.

Marketplace registration and persistent installation modify the user's configuration. Use isolated settings for an installation test, or leave the commands for the user unless they requested installation. Structural checks can finish while runtime checks remain explicitly unverified.
