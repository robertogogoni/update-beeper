# Contributing to update-beeper

Thanks for your interest in contributing! This project aims to be simple and focused.

## Ways to Contribute

### Bug Reports

If something isn't working:

1. Check existing [issues](https://github.com/robertogogoni/update-beeper/issues)
2. Include your Arch Linux version and Beeper version
3. Run `beeper-version` and include the output
4. Include any error messages

### Feature Requests

Open an issue describing:
- What problem you're trying to solve
- Why the current solution doesn't work for you

Keep in mind the project's philosophy: **simple, focused, reliable**.

### Code Contributions

1. Fork the repo
2. Make your changes
3. Test locally: `bash -n update-beeper && ./update-beeper --check`
4. Run shellcheck: `shellcheck update-beeper beeper-version`
5. Submit a PR

## Code Style

- Use `bash` (not `sh`)
- Use `[[ ]]` for conditionals, not `[ ]`
- Quote variables: `"$VAR"` not `$VAR`
- Use meaningful variable names
- Add comments for non-obvious logic
- Keep functions focused and small

## Testing

Before submitting:

```bash
# Check syntax
bash -n update-beeper
bash -n beeper-version

# Run shellcheck
shellcheck update-beeper beeper-version

# Test check mode (safe, doesn't modify anything)
./update-beeper --check

# Test version display
./beeper-version
```

## Commit Messages

- Use present tense: "Add feature" not "Added feature"
- First line: short summary (50 chars or less)
- Body: explain what and why, not how

## Questions?

Open an issue or reach out!
