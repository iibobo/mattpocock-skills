## What it does

`scaffold-skill-plugin` builds a repository for sharing a curated collection of agent skills. It connects the skill folders, plugin and marketplace manifests, discovery pages, and version checks so that the shipped set stays explicit as the collection grows.

Only promoted skills enter the plugin. A draft can live in the same repository and be used during local development without becoming part of the published bundle.

## When to reach for it

Type `/scaffold-skill-plugin`, or the agent reaches for it automatically when the task fits.

| Situation | Use it for |
| --- | --- |
| You want to share a team skills collection | A working plugin repository with an initial useful skill |
| You are adapting this repository's organisation | The separation between content, packaging, discovery, and releases |
| Your collection has drafts mixed with stable skills | An explicit publication boundary and a check that enforces it |

This skill targets repositories using `.claude-plugin` manifests. A browser extension or a ChatGPT app needs its own platform's workflow.

## Prerequisites

A destination directory and an initial task the plugin should help with. Existing files and release conventions are inspected before changes are made. Structural checks use Node.js and the `yaml` package; platform and runtime checks require the relevant target CLI.

## The publication boundary

The plugin manifest lists skill directories individually. README entries, bucket indexes, human-facing pages, and any router describe that same promoted set. A reusable checker catches missing paths, accidental draft inclusion, invocation policy mismatches, and package/plugin version drift.

The repository can start with one useful skill. Extra lifecycle buckets, release automation, and a router arrive when the collection needs them. The organisation summary and starter examples travel with the skill, so using it does not require a checkout of the original repository.

## Common questions

**Can a new project use the official marketplace installation command?**

Only after it has its own accepted listing. A new repository starts with its own marketplace identity. Copying an upstream installation command would install the upstream plugin.

**Does validation prove the plugin works in every coding agent?**

No. Repository checks establish consistency, and platform validation checks packaging. Runtime discovery and skill behaviour are tested separately in each available target agent. Untested targets are reported as unverified.

## It's working if

- You can locate the initial skill from the repository README and its bucket index.
- The manifest lists the stable skills you intended to ship, while a draft stays outside discovery in the local smoke test.
- The final handoff gives actual check results and identifies any target agents that were unavailable.
- Adding the next skill has a clear path through packaging, documentation, and versioning.

## Where it fits

This is a reach-for-it-anytime standalone for creating the repository that distributes a skills collection. [writing-for-agents](https://aihero.dev/skills-writing-for-agents) helps shape the instructions inside its skills; [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) configures a consuming project's issue tracker and docs conventions. [ask-matt](https://aihero.dev/skills-ask-matt) routes you when you are choosing among the collection's workflows.
