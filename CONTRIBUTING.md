# Welcome to the CodeCanvas Collective! 👋

First off, thank you for considering contributing to CodeCanvas. It's people like you that make this open-source startup possible. 

We are a community-led organization. Whether you are fixing a typo, adding a major feature, or updating documentation, your help is appreciated and recognized.

## 🌟 Quick Links
* **[Join the Discord]([LINK_TO_DISCORD](https://discord.gg/s5RfVbVV))** - Chat with us and ask questions.
* **[View the Roadmap](LINK_TO_PROJECT_BOARD)** - See what we are working on.
* **[Governance](GOVERNANCE.md)** - Learn how to become a Maintainer.

---

## 🛠 How to Contribute

### 1. Find an Issue
* Go to the **Issues** tab.
* Look for labels:
    * `good-first-issue`: Great for newcomers.
    * `help-wanted`: Tasks we need help with.
    * `bug`: Things that are broken.
* **Comment on the issue** before you start working! This prevents duplicate work. 
    * *Example: "I'd like to take this on!"*

### 2. Fork & Clone
1. Fork the repository to your own GitHub account.
2. Clone it to your local machine.
3. Install dependencies (check the `README.md` of the specific project you are working on).

### 3. Create a Branch
We use a standard naming convention for branches to keep things organized:
* `feat/short-description` (For new features)
* `fix/short-description` (For bug fixes)
* `docs/short-description` (For documentation)

```bash
git checkout -b feat/add-dark-mode
```

### 4. Make Your Changes

* Keep your changes focused. If you find yourself fixing three different things, split them into three different PRs.
* Write clear code comments where necessary.

### 5. Commit Your Changes

We prefer **Conventional Commits**. This helps us generate changelogs automatically.

* Structure: `type(scope): description`
* Examples:
* `feat(auth): add google login support`
* `fix(navbar): correct alignment on mobile`
* `docs(readme): update installation steps`

### 6. Submit a Pull Request (PR)

1. Push your branch to your fork.
2. Open a Pull Request against the `main` branch of the CodeCanvas repo.
3. Fill out the **PR Template** completely.
4. **Link the Issue:** In your PR description, type `Fixes #123` (where 123 is the issue number) to auto-close the issue when merged.

---

## 🧪 Testing

* Please ensure all tests pass before submitting.
* If you added a new feature, please add a test case for it (if applicable).

## 🎨 Style Guide

* We use **Prettier** and **ESLint** (or language equivalent) to enforce code style.
* Please run the linter before pushing: `npm run lint` (or equivalent).

## 🤝 Community & Recognition

* After your PR is merged, we will add you to our **All Contributors** list in the README!
* Consistent contributors will be invited to join as **Triagers** or **Maintainers**.

Thank you for building with us! 🚀
