# CI/CD Pipeline Documentation

## Overview

This document describes the automated CI/CD (Continuous Integration/Continuous Deployment) pipeline for the SpeedyHighway project. The pipeline is implemented using GitHub Actions and automatically builds, tests, and deploys the game whenever code changes are pushed to the repository.

## Workflow File

The workflow is defined in `.github/workflows/ci-cd.yml`

## Trigger Events

The workflow is triggered by the following events:

### Push Events
- **main branch**: Triggers build, test, and deployment
- **develop branch**: Triggers build and test only
- **release/** branches**: Triggers build and test

### Pull Request Events
- **Pull requests to main**: Triggers build and test
- **Pull requests to develop**: Triggers build and test

## Jobs

The CI/CD pipeline consists of three parallel jobs:

### 1. Build and Test Job

**Runs on**: Windows (windows-latest)

**Steps**:
1. **Checkout code**: Retrieves the repository code
2. **Set up Python**: Installs Python 3.12
3. **Install dependencies**: Installs pygame, pyinstaller, and development tools
4. **Lint with flake8**: Checks for Python syntax errors and code quality issues
5. **Check code quality with pylint**: Additional code quality checks
6. **Verify project structure**: Validates project metadata (version, author, description)
7. **Build executable**: Uses PyInstaller to create SpeedyHighway.exe
8. **Verify build artifacts**: Confirms the executable was created successfully
9. **Upload build artifacts**: Stores the executable for 30 days
10. **Upload data directory**: Stores game data files for 7 days

**Artifacts Produced**:
- `SpeedyHighway-Windows-{commit-sha}`: The game executable
- `game-data-{commit-sha}`: Game data files

### 2. Deploy Job

**Runs on**: Ubuntu (ubuntu-latest)

**Conditions**: 
- Only runs on pushes to the main branch
- Only runs after the build job succeeds

**Steps**:
1. **Checkout code**: Retrieves the repository code
2. **Download build artifacts**: Gets the executable from the build job
3. **Get version**: Extracts version number from source code
4. **Create Release**: Creates a GitHub release (only for release commits)
5. **Upload Release Asset**: Attaches the executable to the release
6. **Deployment notification**: Provides status information

**Release Creation**:
- Releases are only created for commits starting with "Release" or "Version"
- Release tags follow the format: `v{version}-{build-number}`
- Release includes build information and the executable

### 3. Code Quality Report Job

**Runs on**: Ubuntu (ubuntu-latest)

**Runs in parallel** with the build job

**Steps**:
1. **Checkout code**: Retrieves the repository code
2. **Set up Python**: Installs Python 3.12
3. **Install analysis tools**: Installs radon, bandit, and safety
4. **Analyze code complexity**: Calculates cyclomatic complexity
5. **Check maintainability**: Calculates maintainability index
6. **Security scan**: Scans for security vulnerabilities using bandit

**Output**: All results are added to the GitHub Actions step summary

## Requirements

### Repository Requirements
- Python 3.12 compatible code
- `requirements.txt` file with project dependencies
- `project/SpeedyHighway.spec` file for PyInstaller build configuration

### Secret Requirements
- `GITHUB_TOKEN`: Automatically provided by GitHub Actions (no setup needed)

## Viewing Results

### Build Status
1. Go to the repository on GitHub
2. Click on the "Actions" tab
3. Select the latest workflow run
4. View the status of each job

### Build Artifacts
1. Go to a completed workflow run
2. Scroll to the "Artifacts" section at the bottom
3. Download `SpeedyHighway-Windows-{commit-sha}`

### Release Assets
1. Go to the repository's "Releases" section
2. Find the latest release
3. Download `SpeedyHighway.exe` from the assets

## Workflow Configuration

### Python Version
The workflow uses Python 3.12 as specified in the `env.PYTHON_VERSION` variable. This can be updated if needed.

### Linting Configuration
- **flake8**: Configured to check for syntax errors and basic issues
  - Checks for: E9, F63, F7, F82 errors
  - Maximum line length: 127 characters
  - Maximum complexity: 10
- **pylint**: Configured with specific disables for game code patterns
  - Disabled checks: C0103, C0114, C0115, C0116, R0902, R0912, R0913, R0914, R0915

### Artifact Retention
- **Build artifacts**: Retained for 30 days
- **Game data**: Retained for 7 days

## Troubleshooting

### Build Failures

**PyInstaller fails**:
- Check that `project/SpeedyHighway.spec` is present and valid
- Verify all dependencies are in `requirements.txt`
- Check the PyInstaller logs in the workflow output

**Linting errors**:
- Review the flake8 and pylint output
- Fix syntax errors (linting warnings won't fail the build)

**Artifact upload fails**:
- Verify the executable was created in the correct location
- Check file permissions

### Deployment Failures

**Release creation fails**:
- Ensure the commit message starts with "Release" or "Version"
- Check that the tag doesn't already exist
- Verify GITHUB_TOKEN permissions

**Asset upload fails**:
- Ensure the build job completed successfully
- Verify the artifact download step succeeded

## Best Practices

1. **Test locally first**: Run `python -m py_compile car.py` before pushing
2. **Check dependencies**: Ensure `requirements.txt` is up to date
3. **Use semantic versioning**: Update `__version__` in car.py when making releases
4. **Write descriptive commit messages**: Especially for release commits
5. **Review workflow logs**: Check for warnings even if the build succeeds

## Extending the Workflow

### Adding Tests
To add automated tests to the pipeline:

1. Create a `tests/` directory with test files
2. Add test framework to `requirements.txt` (e.g., pytest)
3. Add a test step to the build job:
   ```yaml
   - name: Run tests
     run: pytest tests/
   ```

### Adding More Platforms
To build for additional platforms (Linux, macOS):

1. Create a matrix strategy in the build job:
   ```yaml
   strategy:
     matrix:
       os: [windows-latest, ubuntu-latest, macos-latest]
   runs-on: ${{ matrix.os }}
   ```
2. Adjust the build and artifact steps for each platform

### Adding Deployment Targets
To deploy to additional platforms (e.g., itch.io, Steam):

1. Add deployment credentials as GitHub secrets
2. Add deployment steps to the deploy job
3. Use appropriate GitHub Actions for each platform

## References

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [PyInstaller Documentation](https://pyinstaller.org/)
- [Python GitHub Actions](https://github.com/actions/setup-python)
- [GitHub Releases](https://docs.github.com/en/repositories/releasing-projects-on-github)

## Support

For issues with the CI/CD pipeline:
1. Check the workflow run logs in the Actions tab
2. Review this documentation
3. Submit an issue on GitHub with the workflow run URL

---

**Version**: 1.0  
**Last Updated**: 2026-02-02  
**Maintainer**: RonitARK
