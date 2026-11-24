# Contributing to prompt-edu 🛠

Thank you for considering contributing to our project! This document provides organization-wide contribution guidelines. For repository-specific details, please also read the `CONTRIBUTING.md` (or `AGENTS.md`) in the relevant repository.

## 🌟 Our Core Philosophy: The 3 Golden Rules

If you remember nothing else from this guide, keep these three rules in mind:

1. **🧹 Leave It Cleaner Than You Found It** — Every time you touch a piece of code, try to leave it in a better state. Tidy up as you go!
2. **🔍 Do It Right the First Time** — Aim for quality from the start. "Fixing it later" rarely happens, so let's avoid technical debt today.
3. **🤓 Take Pride in Your Work** — Write code that is a joy to read. Attention to detail is what separates good software from great software.

---

## 📦 Our Repositories

| Repository                                                               | What it is                                                                                |
| ------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------- |
| [prompt](https://github.com/prompt-edu/prompt)                           | Main PROMPT platform — React micro-frontends + Go microservices                           |
| [prompt-intro-course](https://github.com/prompt-edu/prompt-intro-course) | Standalone intro programming course component and server                                  |
| [prompt-lib](https://github.com/prompt-edu/prompt-lib)                   | Shared frontend libraries (`@tumaet/prompt-ui-components`, `@tumaet/prompt-shared-state`) |
| [prompt-sdk](https://github.com/prompt-edu/prompt-sdk)                   | Go SDK for auth, resolution helpers, and shared domain types                              |

Each repository contains its own detailed setup and development guide.

---

## 🆔 Identity Requirements

To maintain a trustworthy and transparent community, **all contributors** must:

- **Use their real full name** in their GitHub profile.
- **Upload an authentic profile picture** — a clear, professional photo. Avoid memojis, avatars, or comic-style images.

These requirements apply to both organization members and external contributors. Contributions that do not adhere to this policy may not be accepted. For full details, see our [Code of Conduct](./CODE_OF_CONDUCT.md#identity-and-transparency-).

---

## 🤝 How to Contribute

### Reporting Bugs

- Search the repository's [Issues](https://github.com/prompt-edu) first to see if the bug has already been reported.
- If not, open a new issue using the available bug report template.
- Include clear reproduction steps, your environment, and screenshots where applicable.

### Suggesting Enhancements

- We love new ideas! Open a feature request issue in the appropriate repository.
- For large architectural changes, discuss them on our [Discord](https://discord.gg/eybNUqD8gf) first.

---

## 🔄 Contribution Process

### For Members of the Organization 👤

1. **Create a feature branch** directly in the repository using a descriptive name:
   - `feature/your-feature-name`
   - `fix/your-bug-description`
   - `chore/task-description`
2. **Work on your changes** in that branch, following the code conventions in the relevant repo's guide.
3. **Open a pull request** against the `main` (or appropriate) branch with a [Conventional Commits](#conventional-commits) title and fill in the PR template completely.
4. **Respond to review feedback** and keep your branch up to date.

### For External Contributors 🌍

1. **Verify your identity**: Ensure your GitHub profile uses your real name and an authentic photo.
2. **Fork the repository** you want to contribute to.
3. **Create a feature branch** in your fork with a descriptive name.
4. **Make your changes**, following the coding standards and conventions defined in the repository.
5. **Open a pull request** from your fork to the upstream repository's `main` branch with a [Conventional Commits](#conventional-commits) title.
6. **Keep your branch up to date** with the upstream main branch before requesting review.

---

## Conventional Commits

All repositories use [Conventional Commits](https://www.conventionalcommits.org/). Pull requests are squash-merged, and **the PR title becomes the commit message on `main`**. The PR title therefore must follow the format below; a required check (`pr-title / conventional-commit`) validates it. The commits on your branch are not checked, but following the format there as well keeps the history readable.

### Format

```text
type(scope)!: subject
```

- **`type`** (required): one of the types listed below.
- **`scope`** (optional): the area of the code, e.g. `table`, `auth` or `deps`.
- **`!`** (optional): marks a breaking change.
- **`subject`** (required): a short summary in the imperative mood ("add", not "added").

Examples:

```text
feat(table): add column pinning
fix: keep the selected course after reload
feat(auth)!: remove the legacy token format
chore(deps): update biome to v2
```

### Types and Releases

Repositories with automated releases (e.g. prompt-lib, prompt-shared-state and prompt-sdk) use semantic-release, which creates a release on every merge to `main` that contains a releasing commit. The type decides the version bump:

| Type       | Use for                                                    | Release                     |
| ---------- | ---------------------------------------------------------- | --------------------------- |
| `feat`     | A new feature                                              | Minor (`1.2.0` → `1.3.0`)   |
| `fix`      | A bug fix                                                  | Patch (`1.2.0` → `1.2.1`)   |
| `perf`     | A performance improvement                                  | Patch                       |
| `docs`     | Documentation only                                         | None                        |
| `style`    | Formatting, no change in behavior                          | None                        |
| `refactor` | Code changes that neither fix a bug nor add a feature      | None                        |
| `test`     | Adding or updating tests                                   | None                        |
| `build`    | Build system or packaging                                  | None                        |
| `ci`       | CI configuration and workflows                             | None                        |
| `chore`    | Other maintenance, e.g. dev dependencies                   | None                        |
| `revert`   | Reverting an earlier commit                                | None (see below)            |

A `revert` releases a patch only if the commit message contains `This reverts commit <sha>.`.

**Breaking changes** create a major release (`1.2.0` → `2.0.0`), whatever the type. Mark them with `!` after the type or scope (preferred, since it is part of the PR title), or with a `BREAKING CHANGE: <description>` footer typed into the squash commit message in the merge dialog. Footers in branch commits or the PR description do not end up in the squash commit. Some repositories need extra steps for a major release (e.g. a `/v2` module path in Go modules); see the repository's README.

**Dependency updates** from Renovate follow the same rules: updates of runtime dependencies are titled `fix(deps): ...` and release a patch, while updates of dev dependencies and GitHub Actions are titled `chore(deps): ...` and release nothing.

Maintainers can find out how a repository adopts the shared PR title check and release workflow in [Reusable Workflows](./docs/reusable-workflows.md).

---

## 💻 Coding Standards

Each repository defines its own conventions — please read the relevant guide before starting:

| Repository            | Guide                                                                                |
| --------------------- | ------------------------------------------------------------------------------------ |
| `prompt`              | [`AGENTS.md`](https://github.com/prompt-edu/prompt/blob/main/AGENTS.md)              |
| `prompt-intro-course` | [`AGENTS.md`](https://github.com/prompt-edu/prompt-intro-course/blob/main/AGENTS.md) |
| `prompt-lib`          | [`README.md`](https://github.com/prompt-edu/prompt-lib/blob/main/README.md)          |
| `prompt-sdk`          | [`README.md`](https://github.com/prompt-edu/prompt-sdk/blob/main/README.md)          |

General cross-repository standards:

- **Frontend (TypeScript/React)**: PascalCase for components, camelCase for utilities, no `any` types, prefer `interface` over `type`.
- **Backend (Go)**: PascalCase for exported identifiers, camelCase for unexported, snake_case for DB columns/tables.
- Keep pull requests **small and focused** — one concern per PR.
- Update documentation if you change behavior.
- Add or update tests as appropriate.

---

## 🧪 Testing

- **Go services**: `go test ./...` from the relevant server directory. Tests use testcontainers for DB isolation.
- **Frontend**: `yarn lint` and `yarn build` from the relevant client directory.

Tests must pass before a PR will be merged.

---

## 💬 Community & Support

- **Discord**: [Join our server](https://discord.gg/eybNUqD8gf) for real-time discussions, help, and planning.
- **Documentation**: [prompt-edu.github.io/prompt](https://prompt-edu.github.io/prompt/)
- **Issues**: Use the GitHub Issues tracker in the relevant repository.

---

## 📜 Code of Conduct

By contributing, you agree to abide by our [Code of Conduct](./CODE_OF_CONDUCT.md). Please read it before participating.

---

Thank you for helping us build great open-source software for education! 🚀

> PROMPT is developed at the [Research Group Applied Education Technologies (AET)](https://aet.cit.tum.de/), Technical University of Munich.
