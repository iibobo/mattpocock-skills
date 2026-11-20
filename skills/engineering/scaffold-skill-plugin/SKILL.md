---
name: scaffold-skill-plugin
description: Scaffold an agent-skills plugin repository with curated skill buckets, plugin and marketplace manifests, discovery docs, and release checks. Use when creating a skills collection plugin or applying this repository's organisation to another project. For browser extensions or ChatGPT app/MCP plugins, use the target platform's own workflow.
---

# Scaffold Skill Plugin

Build a working repository for distributing agent skills. Keep skill content, the shipped set, discovery, and release mechanics as separate concerns, connected by checks.

Read [references/project-organisation.md](references/project-organisation.md) for the source project's organisation and the reasons behind it. Read [references/starter.md](references/starter.md) when writing the manifests and verification commands. Both references travel with this skill; the source repository is not a runtime dependency.

## 1. Establish the target

Inspect the destination's existing instructions, files, manifests, and git status. Infer the plugin name, destination, audience, target agents, and initial skill from the user's request and repository. Ask only for missing choices that would change the result. If the initial capability is unknown, ask for one concrete task the plugin should handle while preparing the infrastructure.

This pattern is a single plugin whose repository is also its marketplace. For an existing repository, preserve its layout and release tooling where they already serve the same purpose. For another plugin ecosystem, establish that ecosystem's manifest and packaging rules before applying any examples here.

**Done when:** the destination, plugin identity, intended shipped skills, and target agents are known. Label untested target agents as unverified.

## 2. Create the content and packaging boundary

For a new repository, use `skills/engineering/` and `skills/productivity/` for promoted skills. Add `in-progress/`, `misc/`, and `deprecated/` only as they acquire content. Other domain names are fine if the user needs them; adapt the checker bucket constants accordingly.

Write at least one useful skill from the requested capability, with concrete completion criteria. Its directory name matches the frontmatter `name`. Add `agents/openai.yaml` with picker metadata. Automatic discovery is the default. If the user chooses explicit-only invocation, pair `disable-model-invocation: true` with `policy.allow_implicit_invocation: false`.

Keep supporting references, scripts, and assets inside the skill that uses them. Keep substantive conditional guidance behind pointers naming when to read it. If a skill dispatches subagents, tell it to start them with a fresh context window, without inheriting the parent conversation.

Create `.claude-plugin/plugin.json` with an explicit array of promoted skill directory paths, each starting with `./`. Create `.claude-plugin/marketplace.json` with a matching plugin entry whose `source` is `./`. Package real files inside the repository; local development symlinks are a separate convenience.

**Done when:** every manifest path resolves inside the plugin, and the listed set equals the promoted set. Drafts and retired skills stay outside that set.

## 3. Make the collection discoverable

- Top-level `README.md`: purpose, installation, and promoted skills grouped by invocation mode, with skill names linked to `SKILL.md`.
- Bucket `README.md`: every skill in that bucket, linked with a one-line description. Group promoted buckets by invocation mode; use flat lists for the others.
- `docs/<bucket>/<name>.md`: a human-facing page for each promoted skill. Explain what it does, when to use it, common questions, observable success, and where it fits. Keep the agent's procedure in `SKILL.md`.
- `AGENTS.md` or `CLAUDE.md`: maintenance invariants and pointers to shared authoring, invocation, and installation conventions. Root steering files guide repository development; a distributed skill carries its own runtime instructions.

For a collection that needs a router, add or update one to map each user-reachable skill and its neighbours. A router recommends explicit-only skills to the human; reusable dependencies should be model-invoked. A one-skill plugin can start with its README instead of an extra router.

Write installation instructions once, then reuse them verbatim. Substitute the new repository and marketplace identities. Use the new repository's own marketplace unless it has actually been accepted into an official marketplace. Verify commands against the installed CLI or current official documentation before claiming support for another agent.

**Done when:** shipped skills can be found through the README, bucket index, docs, and any router, with consistent invocation modes and no copied upstream identity.

## 4. Wire verification and releases

Copy [assets/check-plugin.mjs](assets/check-plugin.mjs) into the destination's `scripts/check-plugin.mjs`. It checks this bucketed, single-plugin convention, supplementing the platform validator. Add its `yaml` dependency and a package script, following the starter reference. Keep one version authority: if `package.json` owns the version, synchronise the plugin manifest during versioning and check equality in CI.

Add CI for the repository check and platform validation. Use the destination's existing release process, or add Changesets when release PRs and changelog automation are useful. A Changesets private package needs `privatePackages.version` enabled to receive version bumps. Initial scaffolding does not require tagging or publishing a release.

Run the repository check and `claude plugin validate . --strict`. If strict validation fails, distinguish new failures from existing warnings, fix in-scope failures, and report remaining limitations. A passing manifest check establishes structure; it does not establish runtime discovery or behaviour.

Smoke-test one initial skill in each available target agent using an isolated local configuration or a session-local plugin directory. Verify its invocation mode and that a non-promoted fixture is absent from discovery. If the target is unavailable, report that runtime check as unverified. Preserve the user's installed plugins and configuration.

**Done when:** checks have real results, the first skill has a usable local invocation, and runtime compatibility claims name what was actually tested.

## 5. Hand over the working repository

Summarise the generated layout, where to add the next skill, how promotion updates the manifest and discovery surfaces, the local verification commands, and any checks that could not run. Leave creation of a remote repository, marketplace registration, installation into user settings, and publication within the user's requested scope.
