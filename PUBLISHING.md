# PyPI Publishing Guide

This guide walks you through publishing the `actig-lambda-deployer` package to PyPI so anyone can install it with `pip install actig-lambda-deployer`.

## Prerequisites

1. **PyPI Account**: Create accounts on both:
   - [PyPI](https://pypi.org/account/register/) (production)
   - [TestPyPI](https://test.pypi.org/account/register/) (testing)

2. **Install build tools**:
   ```bash
   pip install --upgrade build twine
   ```

## Step-by-Step Publishing

### 1. Clean Previous Builds
```bash
# Remove any existing build artifacts
rm -rf dist/ build/ src/*.egg-info/
```

### 2. Test the Package Locally
```bash
# Install in development mode
pip install -e .

# Test the CLI
actig-lambda-deployer --help

# Verify imports work
python -c "import actig_lambda_deployer; print('Import successful')"
```

### 3. Update Version (if needed)
Edit `pyproject.toml` and update the version:
```toml
version = "0.1.1"  # Increment as needed
```

### 4. Build the Package
```bash
# Build source distribution and wheel
python -m build

# Verify the build created files
ls dist/
# Should show:
# actig_lambda_deployer-0.1.0-py3-none-any.whl
# actig_lambda_deployer-0.1.0.tar.gz
```

### 5. Test on TestPyPI First

#### Upload to TestPyPI
```bash
# Upload to TestPyPI
python -m twine upload --repository testpypi dist/*

# You'll be prompted for:
# Username: your TestPyPI username
# Password: your TestPyPI password (or API token)
```

#### Test Installation from TestPyPI
```bash
# Create a new virtual environment for testing
python -m venv test_env
source test_env/bin/activate  # On Windows: test_env\Scripts\activate

# Install from TestPyPI
pip install --index-url https://test.pypi.org/simple/ actig-lambda-deployer

# Test the installation
actig-lambda-deployer --help

# Clean up
deactivate
rm -rf test_env
```

### 6. Publish to Production PyPI

#### Upload to PyPI
```bash
# Upload to production PyPI
python -m twine upload dist/*

# You'll be prompted for:
# Username: your PyPI username  
# Password: your PyPI password (or API token)
```

#### Verify Publication
```bash
# Check your package page
open https://pypi.org/project/actig-lambda-deployer/

# Test installation from PyPI
pip install actig-lambda-deployer
```

## Using API Tokens (Recommended)

Instead of passwords, use API tokens for better security:

### 1. Create API Tokens
- **PyPI**: Go to [Account Settings](https://pypi.org/manage/account/) → API tokens → Add API token
- **TestPyPI**: Go to [Account Settings](https://test.pypi.org/manage/account/) → API tokens → Add API token

### 2. Configure Twine with Tokens
Create/edit `~/.pypirc`:
```ini
[distutils]
index-servers =
    pypi
    testpypi

[pypi]
username = __token__
password = pypi-YOUR_PRODUCTION_TOKEN_HERE

[testpypi]
repository = https://test.pypi.org/legacy/
username = __token__
password = pypi-YOUR_TEST_TOKEN_HERE
```

### 3. Upload with Tokens
```bash
# TestPyPI
twine upload --repository testpypi dist/*

# Production PyPI
twine upload dist/*
```

## Automated Publishing with GitHub Actions

Create `.github/workflows/publish.yml`:

```yaml
name: Publish to PyPI

on:
  release:
    types: [published]

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'
    
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install build twine
    
    - name: Build package
      run: python -m build
    
    - name: Publish to PyPI
      env:
        TWINE_USERNAME: __token__
        TWINE_PASSWORD: ${{ secrets.PYPI_API_TOKEN }}
      run: twine upload dist/*
```

Add your PyPI API token as a GitHub secret named `PYPI_API_TOKEN`.

## Version Management

### Semantic Versioning
Follow [SemVer](https://semver.org/):
- `0.1.0` → `0.1.1` (patch: bug fixes)
- `0.1.0` → `0.2.0` (minor: new features)
- `0.1.0` → `1.0.0` (major: breaking changes)

### Version Bump Commands
```bash
# Manual version update in pyproject.toml
# Then rebuild and republish

# For automated versioning, consider using bump2version:
pip install bump2version
bump2version patch  # 0.1.0 → 0.1.1
bump2version minor  # 0.1.0 → 0.2.0
bump2version major  # 0.1.0 → 1.0.0
```

## Post-Publication Checklist

### 1. Verify Installation
```bash
# Test installation from PyPI
pip install actig-lambda-deployer

# Test CLI functionality
actig-lambda-deployer --help
```

### 2. Update Documentation
- [ ] Update README badges with PyPI version
- [ ] Add installation instructions
- [ ] Update changelog

### 3. Create Git Release
```bash
# Tag the release
git tag v0.1.0
git push origin v0.1.0

# Create GitHub release with release notes
```

### 4. Monitor Usage
- Check PyPI download statistics
- Monitor GitHub issues for user feedback
- Update documentation based on user questions

## Troubleshooting

### Common Issues

**"Package already exists"**
- Increment the version number in `pyproject.toml`
- You cannot reuse version numbers on PyPI

**"Invalid credentials"**
- Verify your username/password or API token
- Check the `~/.pypirc` configuration

**"Distribution not found"**
- Ensure you've run `python -m build` first
- Check that files exist in the `dist/` directory

**"Repository not found"**
- For TestPyPI, ensure you're using the correct repository URL
- Use `--repository testpypi` flag

### Package Validation
```bash
# Check package metadata
twine check dist/*

# Validate package structure
tar -tzf dist/actig_lambda_deployer-0.1.0.tar.gz

# Test wheel contents
unzip -l dist/actig_lambda_deployer-0.1.0-py3-none-any.whl
```

## Maintenance

### Regular Tasks
1. **Update dependencies**: Keep `pyproject.toml` dependencies current
2. **Security updates**: Monitor for vulnerabilities in dependencies  
3. **Python version support**: Test with new Python releases
4. **Documentation**: Keep README and examples up to date

### Yanking Releases
If you need to remove a problematic release:
```bash
# Yank a specific version (doesn't delete, just hides from pip install)
twine upload --repository pypi --yank "Reason for yanking" dist/package-version.tar.gz
```

## Resources

- [PyPI Packaging Guide](https://packaging.python.org/tutorials/packaging-projects/)
- [Twine Documentation](https://twine.readthedocs.io/)
- [Python Build Documentation](https://build.pypa.io/)
- [Semantic Versioning](https://semver.org/)

---

**Ready to publish?** Follow the steps above and your package will be available for anyone to install with `pip install actig-lambda-deployer`!