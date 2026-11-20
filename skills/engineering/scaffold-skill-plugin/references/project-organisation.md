# How this project is organised

This is a reusable summary of `mattpocock/skills`, based on its checked-in structure. Paths below describe the source project; they are not dependencies that a generated repository must fetch. The key design is four concerns with different sources of truth.

| Concern | Source of truth | What it controls |
| --- | --- | --- |
| Behaviour | Each skill's `SKILL.md` and its supporting files | What an agent does after invocation |
| Distribution | `.claude-plugin/plugin.json` | Exactly which skills ship as the plugin |
| Discovery | Root and bucket READMEs, docs pages, `ask-matt` | How humans choose a skill and see its neighbours |
| Releases | `package.json`, Changesets, version sync script | Which version is shipped and when it changes |

## Content and lifecycle

```text
skills/
  engineering/<skill>/
    SKILL.md
    agents/openai.yaml
    references/, scripts/, assets/   # only when useful
  productivity/<skill>/
  in-progress/<skill>/
  misc/<skill>/
  deprecated/<skill>/
```

The top-level buckets combine two classifications. Engineering and productivity describe the domain of promoted skills. The other buckets describe lifecycle and maintenance status.

| Bucket | Meaning | Plugin and root skill index | Docs page | Local development links |
| --- | --- | --- | --- | --- |
| `engineering` | Daily code work | Included | Yes | Included |
| `productivity` | Daily non-code workflows | Included | Yes | Included |
| `in-progress` | Public beta, feedback wanted | Excluded | No | Included |
| `misc` | Rarely used, unmaintained | Excluded | No | Excluded |
| `deprecated` | Retired skills | Excluded | No | Excluded |

Every bucket has its own linked inventory. Promoted bucket indexes and the root skill index split by invocation mode. Non-promoted indexes stay flat. An archived page for a promoted skill removed outright can remain at its old URL, labelled archived.

The important distinction is that being present in the repository, linked for local development, and shipped in a plugin are three different sets.

## Invocation is independent of the bucket

| Mode | `SKILL.md` frontmatter | `agents/openai.yaml` | Role |
| --- | --- | --- | --- |
| Model-invoked | Omit `disable-model-invocation` | Omit restrictive policy | Automatically discoverable and manually usable |
| User-invoked | `disable-model-invocation: true` | `policy.allow_implicit_invocation: false` | The human chooses when to start it |

User-invoked skills often orchestrate work. Model-invoked skills hold reusable practices that other skills can use. The router helps a human choose; it does not bypass the invocation policy. Description wording follows that role: trigger conditions for model discovery, a concise human-facing summary for explicit-only entrypoints.

## Packaging and installation

`.claude-plugin/plugin.json` contains metadata and an explicit `skills` array of promoted directories. `.claude-plugin/marketplace.json` makes the repository a single-plugin marketplace with `source: "./"`. The plugin definition and marketplace listing serve different purposes, so component paths stay in the plugin manifest.

`.agents/install-block.md` owns installation wording. The source project's official Claude Code listing is an acquired distribution route, not a capability a new fork inherits. Its current checked-in instructions also describe managed routes for other agents. Verify each target's current support instead of assuming identical behaviour from a common manifest.

`scripts/link-skills.sh` is a maintainer convenience: it links working-tree skills into local harness directories. These development links can include beta skills. They are not the distributed plugin, and symlink behaviour in an installer must be tested separately.

## Documentation and routing

The documentation has distinct audiences:

- `SKILL.md` tells the agent what to do, with supporting material disclosed only when relevant.
- The root and bucket READMEs provide linked inventories.
- `docs/engineering/` and `docs/productivity/` explain each promoted skill to a human. Pages cover **What it does**, **When to reach for it**, **Common questions**, **It's working if**, and **Where it fits**. They also state prerequisites when needed.
- `skills/engineering/ask-matt/SKILL.md` maps the main workflow, on-ramps, reusable disciplines, and standalone tools.
- `CLAUDE.md`, exposed through the `AGENTS.md` symlink, states repository maintenance rules. `.agents/` holds detailed authoring conventions, canonical installation wording, and ADRs.
- `GLOSSARY.md` owns domain vocabulary. `SCOPE.md`, `CONTRIBUTING.md`, and `.out-of-scope/` record the upstream project's contribution and curation decisions.

The source site's URL scheme, author identity, issue labels, contribution restrictions, and prose preferences are local conventions. A new project chooses its own. Preserve the separation of responsibilities without copying upstream policy as a universal requirement.

## Versions and checks

`package.json` is private because this project distributes through plugins rather than npm. Changesets still versions it: `.changeset/config.json` enables versioning and tagging private packages. The release workflow creates a version PR, and `npm run version` runs Changesets followed by `scripts/sync-plugin-version.mjs`.

That script copies the package version into `plugin.json`; its `--check` mode detects drift. The marketplace has no separate version. `scripts/check-frontmatter.mjs` parses every skill's YAML using the same `yaml` library as the installer. The checked-in Check workflow runs the frontmatter check. Other maintenance rules, including exact promoted-set coverage, currently live in instructions rather than that CI job.

The scaffold skill adds a reusable checker to make those publication and discovery invariants executable in new projects. That is an improvement on the source pattern, not a claim about checks the source project already runs.

## A change crosses these boundaries

| Change | Coordinated updates |
| --- | --- |
| Add a promoted skill | Skill and picker metadata, plugin array, root index, bucket index, docs page, router if present, changeset |
| Add a beta skill | Skill and picker metadata, beta bucket index, optional local development link |
| Promote a beta skill | Move the directory, remove its old bucket entry, add every promoted discovery surface and manifest path |
| Rename or move a promoted skill | Directory and frontmatter name where renamed, manifest paths, indexes, docs path, router and references |
| Remove a promoted skill outright | Remove packaging and navigation entries; preserve its docs page with an archive notice |
| Release | Version authority, plugin version synchronisation, changelog and tag through the chosen release process |

## Evidence and platform references

The summary comes from the source project's `CLAUDE.md`, `.agents/invocation.md`, `.agents/writing-docs.md`, `.agents/install-block.md`, ADR 0002 including its later updates, manifests, scripts, and workflows. Historical ADR constraints can be superseded by later updates.

For platform-specific fields and behaviour, consult the current [Claude Code manifest reference](https://code.claude.com/docs/en/plugins-reference) and [marketplace guide](https://code.claude.com/docs/en/plugin-marketplaces). Treat the examples here as a repository pattern and validate them with the target CLI.
