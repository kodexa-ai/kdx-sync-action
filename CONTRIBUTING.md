# Contributing to Kodexa Sync Action

Thank you for your interest in contributing to the Kodexa Sync GitHub Action! We welcome contributions from the community.

## Ways to Contribute

- **Bug Reports**: Open an issue describing the problem, steps to reproduce, and expected behavior
- **Feature Requests**: Suggest new features or improvements via issues
- **Documentation**: Improve README, examples, or inline documentation
- **Code**: Submit pull requests for bug fixes or new features

## Development Setup

1. **Fork and clone the repository**:
   ```bash
   git clone https://github.com/YOUR-USERNAME/kdx-sync-action.git
   cd kdx-sync-action
   ```

2. **Create a feature branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```

3. **Make your changes**:
   - Update `action.yml` for action changes
   - Add/update examples in `examples/`
   - Update README.md if adding features

4. **Test your changes**:
   - Test the action locally using [act](https://github.com/nektos/act)
   - Or create a test workflow in a separate repository

## Testing Locally

### Using act (Recommended)

Install [act](https://github.com/nektos/act) and run:

```bash
# Test the basic workflow
act -W .github/workflows/test.yml

# Test on specific platform
act -P ubuntu-latest=ghcr.io/catthehacker/ubuntu:act-latest

# Test with dry-run mode
act -W .github/workflows/test.yml -s KODEXA_URL=https://test.kodexa.com -s KODEXA_TOKEN=test
```

### Using a Test Repository

1. Create a test repository
2. Reference your fork/branch:
   ```yaml
   uses: YOUR-USERNAME/kdx-sync-action@feature/your-feature-name
   ```
3. Run the workflow and verify results

## Pull Request Guidelines

1. **Describe your changes**: Provide a clear description of what and why
2. **Reference issues**: Link to any related issues
3. **Update documentation**: Update README or examples if needed
4. **Test thoroughly**: Ensure your changes work across platforms
5. **Keep it focused**: One feature/fix per PR

## Code Style

- Use clear, descriptive variable names
- Add comments for complex logic
- Follow existing formatting conventions
- Keep shell scripts POSIX-compatible where possible

## Commit Message Format

Use conventional commit messages:

```
feat: add support for custom metadata directories
fix: handle spaces in organization names correctly
docs: update examples for multi-environment setup
test: add test for Windows platform
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `test`: Test additions or modifications
- `chore`: Maintenance tasks
- `refactor`: Code refactoring

## Review Process

1. **Automated tests**: Your PR must pass all CI checks
2. **Code review**: Maintainers will review your changes
3. **Feedback**: Address any requested changes
4. **Merge**: Once approved, your PR will be merged

## Questions?

- Open a [discussion](https://github.com/kodexa-ai/kdx-sync-action/discussions)
- Email: support@kodexa.com

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
