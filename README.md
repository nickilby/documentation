# IT Documentation Repository

This repository contains IT policies and how-to guides for both Technical IT staff and end users.

## 📚 Structure

- **[Policies](./policies/)** - IT policies and procedures for technical staff
- **[Guides](./guides/)** - How-to guides for both technical staff and end users

## 🌐 GitHub Pages

This documentation is published via GitHub Pages for easy accessibility.

**View the live documentation:** [https://nickilby.github.io/documentation/](https://nickilby.github.io/documentation/)

### Setting up GitHub Pages

1. Go to your repository settings on GitHub
2. Navigate to "Pages" in the left sidebar
3. Under "Source", select the branch (typically `main` or `master`)
4. Select the folder (typically `/ (root)`)
5. Click "Save"

The site will be available at `https://[username].github.io/documentation/` after a few minutes.

## 📖 Navigation

Each section contains an index file that lists all available documents and their status:
- ✅ **Written** - Document is complete and available
- 📝 **To Do** - Document is planned but not yet written

## 🛠️ Development Setup

### Pre-commit Hooks

This repository uses pre-commit hooks to automatically lint markdown files before commits.

**Initial Setup:**
```bash
# Install dependencies
npm install

# This will set up the git hooks automatically
npm run prepare
```

After setup, the pre-commit hook will automatically:
- Lint all staged markdown files
- Attempt to auto-fix common issues
- Prevent commits if unfixable errors are found

**Manual Linting:**
```bash
# Lint all markdown files
npm run lint

# Lint and auto-fix issues
npm run lint:fix
```

## 🤝 Contributing

When adding new policies or guides:
1. Create the document in the appropriate folder
2. Update the relevant index file to mark it as written
3. Follow the existing naming conventions and structure
4. The pre-commit hook will automatically check your markdown formatting

## 📄 License

This documentation is licensed under the MIT License - see the [LICENSE](./LICENSE) file for details.
