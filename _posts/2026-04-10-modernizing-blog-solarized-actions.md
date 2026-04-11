---
layout: post
author: Ke
title: "Update: Solarized theme migration and CI/CD pipeline"
date: 2026-04-10 12:00:00 +0000
categories: [blogging, web-dev]
---

The environment has been updated. The site now runs on a Solarized Light/Dark foundation with a deployment handled by GitHub Actions.

<!--more-->

### Implementation: Solarized Palette

The objective was to implement the Solarized palette via CSS custom properties. 

Changes include:
- **Theme Logic**: Persistent toggle using `localStorage` for state.
- **Variable Mapping**: Dynamic variables using `@use` to map the base tones.
- **Structural Color**: Heading levels (H1-H6) assigned specific accent colors (Blue, Green, Yellow, Orange, Violet).
- **Typography**: Shifted to a modern monospace stack (JetBrains Mono, Fira Code) with Solarized Magenta for inline code.

---

### Technical Obstacles

Several blockers were encountered during the transition:

#### 1. Outdated Toolchain
**Issue**: The development environment was initially constrained by the macOS default Ruby version, which is notoriously outdated and lacks the necessary features for modern Jekyll builds. Furthermore, the project's legacy `Gemfile.lock` was incompatible with current build tools.

**Resolution**: Gemini performed an environment migration, bypassing the system Ruby in favor of Ruby 3.2.2 via `rbenv`. Ensuring the correct Ruby version persists across shell sessions required careful initialization (`rbenv init`) and `eval` execution—a non-trivial step for maintaining toolchain integrity. Finally, all dependencies were updated to work with Bundler 4.0.10, establishing a stable foundation for Jekyll 4.x.

#### 2. Build Environment
**Issue**: The legacy GitHub Pages builder failed to process modern Sass `@use` syntax.

**Resolution**: Gemini implemented a custom GitHub Actions workflow. This provided control over the Ruby 3.2.2 environment and fixed the Sass compilation errors.

#### 3. Deployment Configuration
**Issue**: GitHub Pages was still attempting to use the "Deploy from a branch" legacy builder, causing redundant and failing builds even after the Actions workflow was added.

**Resolution**: Switched the repository's **Settings > Pages > Build and deployment > Source** from "Deploy from a branch" to **"GitHub Actions"**. This fully migrated the deployment authority to the custom workflow.

#### 4. Workflow Schema
**Issue**: Invalid YAML syntax in the deployment job due to property ordering.

**Resolution**: Gemini refactored `.github/workflows/jekyll.yml` to ensure `runs-on` and `needs` were correctly prioritized.

#### 5. Sass Compatibility
**Issue**: Compilation failures on specific pseudo-elements and Sass modules in certain build environments.

**Resolution**: Gemini simplified the syntax by removing `li::marker` and reverting to standard division to ensure reliability across all endpoints.

#### 6. Cache Persistence
**Issue**: Stale assets serving from CDN/browser cache.

**Resolution**: Verified via Incognito and hard refresh.

---

### Meta
- **Total Time**: ~2.5 hours
- **Token Usage**: ~60k (contextual)
- **Model**: Gemini 3 Flash
- **Note**: Multiple human interventions required; execution not fully autonomous.
