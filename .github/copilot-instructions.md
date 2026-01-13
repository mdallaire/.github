# Renovate Config Repository

This repository provides shareable Renovate Bot configuration presets for automated dependency management across multiple repositories.

## Architecture

**Configuration Structure**: Modular preset system where [renovate-config/default.json](renovate-config/default.json) serves as the main entry point, extending specialized configs using `github>mdallaire/renovate-config:` references. Each preset is independently consumable.

**Preset Organization**:
- [autoMerge.json5](renovate-config/autoMerge.json5) - Auto-merge rules for GitHub Actions, Mise tools, with trusted package allowlists
- [labels.json5](renovate-config/labels.json5) - PR labeling by update type (major/minor/patch/digest)
- [semantic-commits.json5](renovate-config/semantic-commits.json5) - Conventional commit formatting with datasource-specific scopes
- [generic-regex-manager.json](renovate-config/generic-regex-manager.json) - Custom YAML dependency extraction via inline comments
- Legacy presets: `automerge-docker-digest.json`, `automerge-github-actions.json` (superseded by autoMerge.json5)

## Configuration Patterns

**Semantic Commit Scoping**: Each dependency type maps to specific commit scopes and prefixes. Docker uses `chore(container)`, GitHub Actions use `ci(github-action)`, Helm uses `feat(helm)`. See [semantic-commits.json5](renovate-config/semantic-commits.json5) lines 24-54 for the complete mapping.

**Auto-merge Strategy**: Three-tier trust model:
1. Trusted vendors (actions/*, docker/*, renovatebot/*) - 1 minute age, immediate merge
2. Standard GitHub Actions - 3 days age for stability
3. Mise tools - minor/patch only, no major updates

All auto-merge uses `automergeType: "branch"` with `ignoreTests: true`.

**Custom Dependency Detection**: YAML files support inline Renovate directives:
```yaml
# renovate: datasource=github-releases depName=kubernetes/kubectl
version: "1.28.0"
```

## Development Workflow

**Testing Changes**: Update `.renovaterc.json5` in test repositories to extend modified presets. Use Renovate's dry-run mode via workflow_dispatch with `dryRun: true` input.

**File Format**: Use `.json5` for configs with comments (autoMerge, labels, semantic-commits). Use `.json` for schema-strict configs (generic-regex-manager).

**Schema Validation**: All files include `$schema` reference to `https://docs.renovatebot.com/renovate-schema.json` for IDE validation.

## Integration Points

**Consumption Pattern**: External repos reference via `extends: ["github>mdallaire/renovate-config"]` in their `.renovaterc.json5`. Specific presets can be cherry-picked: `extends: ["github>mdallaire/renovate-config:autoMerge.json5"]`.

**GitHub Actions Integration**: [.github/workflows/renovate.yaml](../workflows/renovate.yaml) runs every 2 hours with auto-discovery across all `mdallaire/*` repositories using GitHub App authentication for elevated permissions.

## Key Conventions

- **Version Display**: Use `{{currentVersion}} → {{newVersion}}` format in commit messages for clarity
- **Digest Updates**: Display shortened hashes: `{{currentDigestShort}} → {{newDigestShort}}`
- **Timezone**: All scheduling uses `America/Toronto` timezone
- **Branch Strategy**: `rebaseWhen: "auto"` with `:automergeBranch` for clean git history
