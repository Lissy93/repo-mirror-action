# Contributing

Contributions of any kind are welcome (and much appreciated!)

## Reporting Issues

When opening an issue, please include:
- A clear description of the problem with steps to reproduce
- Mention the actual vs expected behavior, with examples if possible
- Link to your workflow code
- Link to your workflow run

## Pull Requests

**Before submitting:**
1. Test your changes with the test workflow (`.github/workflows/mirror.yml`)
2. Update docs if needed
3. Keep changes focused and atomic

**Guidelines:**
- Write clear commit messages
- Follow existing code style (shellcheck clean, proper quoting)
- Add examples for new features
- Update README if adding inputs/outputs

## Testing

Run the test workflow manually:
```bash
gh workflow run mirror.yml
```

Or push to your fork and test with:
```yaml
uses: your-username/mirror@your-branch
```
