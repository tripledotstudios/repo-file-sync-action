# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a GitHub Action that keeps files like Action workflows or entire directories in sync between multiple repositories. It works by running in your main repository and using a `sync.yml` config file to determine which files to sync where.

## Development Commands

- `npm run lint` - Run ESLint on the source code
- `npm run start` - Run the Action locally for testing
- `npm run build` - Build production version with @vercel/ncc into the `dist/` folder

## Architecture

### Core Components

- **src/index.js** - Main entry point that orchestrates the sync process
- **src/config.js** - Handles configuration parsing and environment variable processing  
- **src/git.js** - Git operations class that handles cloning, commits, PRs, and GitHub API interactions
- **src/helpers.js** - Utility functions for file operations, templating with Nunjucks, and async iteration

### Key Flow

1. Parse configuration from `.github/sync.yml` (or custom path)
2. For each target repository:
   - Clone the repo locally to a temp directory
   - Create a PR branch (unless SKIP_PR is true)
   - Sync specified files/directories with template rendering support
   - Commit changes (either per file or batched)
   - Push to GitHub and create/update PR
   - Add labels, assignees, reviewers as configured

### Configuration System

The action supports both Personal Access Tokens (GH_PAT) and GitHub App Installation Tokens (GH_INSTALLATION_TOKEN). Configuration is loaded from environment variables via the `action-input-parser` library.

### File Sync Features

- Individual files or entire directories
- Template rendering with Nunjucks (Jinja-style syntax)
  - Repository metadata automatically injected into template context via `repo` object
  - Available properties: `repo.name`, `repo.user`, `repo.fullName`, `repo.uniqueName`, `repo.host`, `repo.branch`
  - Allows per-repository customization using Nunjucks conditionals and filters
- Exclude patterns for directory syncing
- Delete orphaned files option
- File deletion with `delete: true` option
- Custom destination paths
- Fork-based workflow support

### GitHub Integration

The `git.js` class handles all GitHub API interactions including:
- Repository cloning and pushing
- PR creation/updates with custom titles and bodies
- Labels, assignees, and reviewer management
- Verified commits via GitHub API when using installation tokens
- Branch management and force pushing

## Testing

There are no automated tests in this codebase currently. Local testing is done via `npm run start`.