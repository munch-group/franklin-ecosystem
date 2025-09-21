# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Franklin is a Python-based educational platform for managing Jupyter notebook exercises in containerized environments. It consists of four packages in a monorepo:

- `franklin-cli/` - Core CLI tool providing Docker and Jupyter integration
- `franklin-educator/` - Plugin adding exercise creation/management commands
- `franklin-admin/` - Plugin adding user management commands
- `franklin-container/` - Jupyter magic for container environments
- `franklin-magic/` - Standalone magic utilities

## Essential Commands

### Testing
```bash
# Run tests for a package
cd franklin-cli && ./test.sh           # Uses python -m unittest
```

### Releases
```bash
# Create and push version tag (triggers CI/CD)
cd franklin-cli && ./release-tag.sh "Release message"
```

### Documentation
```bash
# Build Quarto documentation
cd franklin-cli/docs && ./docs-build-render.sh

# Execute notebooks for documentation
cd franklin-cli/docs && ./docs-run-notebooks.sh
```

### Development Setup
```bash
# Install dependencies using Pixi (preferred)
pixi install

# Alternative: pip install
pip install -e .
```

## Architecture

### Plugin System
The core franklin-cli package uses Click plugins with entry points discovery:

```python
# franklin-cli/src/franklin_cli/__init__.py
@with_plugins(entry_points().select(group='franklin_cli.plugins'))
@click.group(cls=AliasedGroup)
def franklin():
    pass
```

Plugins are registered via `pyproject.toml` entry points and extend the CLI with additional commands.

**Plugin Interface** (`franklin-cli/src/franklin_cli/plugin_interface.py`): Base class for plugins providing standardized methods for command registration and configuration.

**Facade Interfaces** (`franklin-cli/src/franklin_cli/interfaces.py`): Stable API surfaces for plugins:
- `GitLabInterface`: GitLab API operations
- `TerminalInterface`: Terminal formatting and interaction
- `DockerInterface`: Docker container management
- `JupyterInterface`: Jupyter server management
- `ConfigInterface`: Configuration access
- `UtilsInterface`: Common utilities

**Authentication** (`franklin-cli/src/franklin_cli/auth.py`): Centralized encryption and token management utilities.

### Key Integration Points

1. **GitLab API** (`franklin-cli/src/franklin_cli/gitlab.py`): Exercise distribution, user authentication, repository management
2. **Docker Management** (`franklin-cli/src/franklin_cli/docker.py`): Container lifecycle, volume management, Docker Desktop integration
3. **Jupyter Server** (`franklin-cli/src/franklin_cli/jupyter.py`): Notebook server management in containers
4. **Pixi Package Manager**: Modern Python dependency management for exercises
5. **Browser Automation** (`franklin-cli/src/franklin_cli/chrome.py`): Selenium-based UI automation

### Dependency Hierarchy
```
franklin-admin → franklin-educator → franklin-cli (core)
```

Students install only `franklin-cli`, educators add `franklin-educator`, administrators need `franklin-admin`.

**Note**: Plugins declare `franklin-cli` as a dependency in their `pyproject.toml` rather than duplicating all core dependencies.

### Exercise Workflow
1. Educators create exercises as GitLab repositories using `franklin exercise create`
2. Students download with `franklin download <exercise-url>`
3. Franklin manages Docker containers with proper dependencies via Pixi
4. Jupyter runs in containers with franklin-container magic for package management

## Development Guidelines

### Commit Convention
Use conventional commits as specified in the README:
- `fix:` for bug fixes (PATCH)
- `feat:` for new features (MINOR)
- `BREAKING CHANGE:` or `!` suffix for breaking changes (MAJOR)
- Other types: `build:`, `chore:`, `ci:`, `docs:`, `style:`, `refactor:`, `perf:`, `test:`

### Version Management
Version is stored in each package's `pyproject.toml`. The `release-tag.sh` script reads this version and creates matching git tags.

### Testing Approach
Currently uses Python's built-in `unittest`. Test files follow the pattern `test/test_*.py` in each package directory.

### Package Building
- Modern Python packaging with `pyproject.toml` and setuptools
- Conda packages built via GitHub Actions on version tags
- Dependencies managed through Pixi with fallback to pip

## Important Files and Locations

- Main CLI entry: `franklin-cli/src/franklin_cli/__init__.py`
- Docker integration: `franklin-cli/src/franklin_cli/docker.py`
- GitLab integration: `franklin-cli/src/franklin_cli/gitlab.py`
- Configuration: `franklin-cli/src/franklin_cli/config.py`
- Exercise templates: `franklin-cli/src/franklin_cli/data/templates/exercise/`
- Documentation source: `franklin-cli/docs/` (Quarto-based)

## Git Backend Interface Plan

### Overview
To make Franklin platform-agnostic and support multiple git hosting services (GitLab, GitHub, Bitbucket, etc.), we need to implement a general backend interface system.

### Architecture Components

#### 1. Abstract Backend Interface
Create a `GitBackend` abstract base class defining standard operations:
- Repository management (create, delete, fork, list)
- User management (create, get, list)
- Group/organization management
- File operations (read, write, update)
- Merge/pull request creation

Platform-agnostic data models:
- `Repository`: Standard repository representation
- `User`: Standard user representation  
- `Group`: Standard group/organization representation

#### 2. Platform Implementations
- **GitLabBackend**: Full GitLab API implementation
- **GitHubBackend**: GitHub API with org/team support
- **BitbucketBackend**: Bitbucket API implementation
- **GiteaBackend**: Self-hosted Gitea support

#### 3. Backend Factory & Configuration
```yaml
# ~/.franklin/backend.yaml
backend:
  type: gitlab  # or 'github', 'bitbucket', etc.
  settings:
    url: https://gitlab.com
    token: ${FRANKLIN_GIT_TOKEN}
  defaults:
    visibility: private
    init_readme: true
```

#### 4. Feature Compatibility Matrix

| Feature | GitLab | GitHub | Bitbucket |
|---------|--------|--------|-----------|
| Create Repository | ✅ | ✅ | ✅ |
| Fork Repository | ✅ | ✅ | ✅ |
| Create Users | ✅ | ❌ | ❌ |
| Create Groups | ✅ | ✅ (Orgs) | ✅ (Teams) |
| CI/CD | ✅ | ✅ (Actions) | ✅ |

### Implementation Phases

#### Phase 1: Abstraction Layer (Weeks 1-2)
- Create abstract base classes
- Implement adapter for current GitLab code
- Maintain backward compatibility

#### Phase 2: GitLab Backend (Weeks 3-4)
- Full GitLabBackend implementation
- Migrate existing code to new interface
- Comprehensive testing

#### Phase 3: GitHub Backend (Weeks 5-6)
- Implement GitHubBackend
- Handle GitHub-specific limitations
- Add GitHub Actions support

#### Phase 4: Integration (Weeks 7-8)
- CLI backend switching
- Migration tools
- Documentation

### Key Benefits
1. **Flexibility**: Institutions can use their preferred git platform
2. **Portability**: Easy migration between platforms
3. **Extensibility**: Simple to add new backend support
4. **Maintainability**: Clean separation of concerns

### Migration Strategy
- Provide tools to migrate exercises between backends
- Support gradual migration with backward compatibility
- Comprehensive testing at each phase

For full implementation details, see `GIT_BACKEND_INTERFACE_PLAN.md`