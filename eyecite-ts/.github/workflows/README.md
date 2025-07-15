# GitHub Actions Workflows

This repository uses several GitHub Actions workflows to automate testing and publishing:

## 1. Test Workflow (`test.yml`)

- **Triggers**: On every push and pull request to main/master/develop branches
- **Purpose**: Runs tests across multiple OS and Node versions
- **Matrix**: Ubuntu, macOS, Windows × Node 18.x, 20.x, 21.x
- **Uses**: Bun for package management and test running

## 2. Manual Publish Workflow (`publish.yml`)

- **Triggers**: When a version tag is pushed (e.g., `v1.0.0`)
- **Purpose**: Publishes to npm when you manually create a version tag
- **Requirements**: 
  - Set `NPM_TOKEN` secret in repository settings
  - Tag version must match package.json version

### Usage:
```bash
# Update version in package.json
bun version patch  # or minor/major

# This creates a tag and pushes it, triggering the publish workflow
git push origin v2.7.7
```

## 3. Semantic Release Workflow (`release.yml`)

- **Triggers**: On every push to main/master/next/beta/alpha branches
- **Purpose**: Automatically determines version and publishes based on commit messages
- **Features**:
  - Analyzes commits using conventional commit format
  - Automatically bumps version
  - Generates changelog
  - Creates GitHub releases
  - Publishes to npm

### Commit Message Format:
- `feat: new feature` → minor version bump
- `fix: bug fix` → patch version bump
- `feat!: breaking change` or `BREAKING CHANGE:` → major version bump
- `chore:`, `docs:`, `style:`, `refactor:`, `test:` → no release

### Requirements:
- Set `NPM_TOKEN` secret in repository settings
- Use conventional commit messages

## Setting up Secrets

1. Go to Settings → Secrets and variables → Actions
2. Add `NPM_TOKEN`:
   - Get token from npm: https://www.npmjs.com/settings/[username]/tokens
   - Create "Automation" token with publish permissions
   - Add as repository secret

## Choosing a Publishing Strategy

### Option 1: Manual Publishing (recommended for getting started)
- Use the `publish.yml` workflow
- You control when versions are released
- Update version manually with `bun version`
- Push tags to trigger publish

### Option 2: Automated Semantic Release
- Use the `release.yml` workflow
- Versions determined automatically from commits
- Requires discipline with commit messages
- Great for active projects with multiple contributors

## Dependabot

The repository is configured to automatically update dependencies weekly:
- npm dependencies
- GitHub Actions versions
- Creates PRs with updates