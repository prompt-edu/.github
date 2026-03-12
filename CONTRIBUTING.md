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
3. **Open a pull request** against the `main` (or appropriate) branch and fill in the PR template completely.
4. **Respond to review feedback** and keep your branch up to date.

### For External Contributors 🌍

1. **Verify your identity**: Ensure your GitHub profile uses your real name and an authentic photo.
2. **Fork the repository** you want to contribute to.
3. **Create a feature branch** in your fork with a descriptive name.
4. **Make your changes**, following the coding standards and conventions defined in the repository.
5. **Open a pull request** from your fork to the upstream repository's `main` branch.
6. **Keep your branch up to date** with the upstream main branch before requesting review.

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
- **Documentation**: [ls1intum.github.io/prompt2](https://ls1intum.github.io/prompt2/)
- **Issues**: Use the GitHub Issues tracker in the relevant repository.

---

## 📜 Code of Conduct

By contributing, you agree to abide by our [Code of Conduct](./CODE_OF_CONDUCT.md). Please read it before participating.

---

Thank you for helping us build great open-source software for education! 🚀

> PROMPT is developed at the [Research Group Applied Education Technologies (AET)](https://www.aet.cit.tum.de/), Technical University of Munich.
