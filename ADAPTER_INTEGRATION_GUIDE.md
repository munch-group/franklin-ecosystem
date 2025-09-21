# Adapter Layer Integration Guide

This guide explains how to integrate the new backend system with existing Franklin code using the adapter layer, without modifying any existing files.

## Overview

The adapter layer provides a bridge between the existing Franklin GitLab code and the new backend system. It allows the current codebase to work with multiple git platforms (GitLab, GitHub, etc.) without any modifications.

## Components Created

### 1. Adapter Module (`backends/adapter.py`)

Provides compatibility classes that mimic the existing GitLab interface:

- **`GitLabCompatibilityAdapter`** - Drop-in replacement for GitLab client
- **`ExerciseAdapter`** - Exercise management with any backend
- **Module-level functions** - Direct replacements for existing functions

### 2. CLI Wrapper (`backends/cli_wrapper.py`)

New CLI commands that work with any backend:

- **`franklin backend`** - Manage backend configuration
- **`franklin exercise`** - Backend-aware exercise commands
- **`franklin repo`** - Backend-aware repository commands

### 3. Migration Utilities (`backends/migration.py`)

Tools to help transition to the new system:

- **`ConfigMigrator`** - Migrate old Franklin config to backend format
- **`RepositoryMigrator`** - Migrate repos between backends
- **`CodeMigrator`** - Update Python imports automatically

## Integration Methods

### Method 1: Import Replacement (Minimal Change)

Replace GitLab imports with adapter imports in your code:

```python
# Old import
from franklin import gitlab
from franklin.gitlab import create_project

# New import (using adapter)
from franklin.backends.adapter import gitlab
from franklin.backends.adapter import create_project
```

The adapter provides the same interface, so no other code changes are needed.

### Method 2: Environment Variable Override

Set an environment variable to redirect imports (requires import hook):

```python
# In your main script or __init__.py
import os
if os.environ.get('USE_BACKEND_ADAPTER'):
    import sys
    sys.modules['franklin.gitlab'] = __import__('franklin.backends.adapter')
```

### Method 3: Monkey Patching (No Code Changes)

Replace the GitLab module at runtime:

```python
# In franklin/__init__.py (when ready to integrate)
def _setup_backend_adapter():
    """Replace gitlab module with adapter if configured."""
    import os
    if os.environ.get('FRANKLIN_USE_BACKEND', '').lower() == 'true':
        from franklin.backends.adapter import GitLabCompatibilityAdapter
        
        # Replace the gitlab module
        import sys
        sys.modules['franklin.gitlab'] = GitLabCompatibilityAdapter()

# Call during initialization
_setup_backend_adapter()
```

## Usage Examples

### Using the Adapter Directly

```python
from franklin.backends.adapter import GitLabCompatibilityAdapter

# Create adapter (uses configured backend)
gitlab = GitLabCompatibilityAdapter()

# Use exactly like the old GitLab module
project = gitlab.create_project("test-project", visibility="private")
user = gitlab.get_user()
projects = gitlab.list_projects(owned=True)
```

### Using Backend-Aware CLI Commands

```bash
# Configure backend (one-time setup)
franklin backend init --type github

# Use backend-aware commands
franklin repo create my-repo --visibility private
franklin repo list --owned
franklin exercise create homework-1
franklin exercise download https://github.com/prof/homework-1
```

### Using with Different Backends

```python
from franklin.backends import GitLabBackend, GitHubBackend
from franklin.backends.adapter import GitLabCompatibilityAdapter

# Use GitLab backend
gitlab_backend = GitLabBackend(url="https://gitlab.com", token="...")
adapter = GitLabCompatibilityAdapter(backend=gitlab_backend)
repo = adapter.create_project("gitlab-repo")

# Switch to GitHub backend
github_backend = GitHubBackend(token="...")
adapter = GitLabCompatibilityAdapter(backend=github_backend)
repo = adapter.create_project("github-repo")  # Creates on GitHub!
```

## Migration Path

### Step 1: Test Compatibility (No Changes)

Test that the adapter works with your existing code:

```python
# Temporary test file
from franklin.backends.adapter import gitlab

# Your existing code should work unchanged
project = gitlab.create_project("test", visibility="private")
```

### Step 2: Configure Backend

Create backend configuration:

```bash
# Interactive configuration
franklin backend init

# Or create manually at ~/.franklin/backend.yaml
```

```yaml
backend:
  type: gitlab  # or github
  settings:
    url: https://gitlab.com
    token: ${GITLAB_TOKEN}
  defaults:
    visibility: private
```

### Step 3: Update Imports (Gradual Migration)

Update imports file by file as convenient:

```bash
# Use migration tool
franklin migrate code path/to/your/code.py

# Or manually update imports
# from franklin import gitlab → from franklin.backends.adapter import gitlab
```

### Step 4: Test Both Backends

Verify your code works with different backends:

```bash
# Test with GitLab
franklin backend switch gitlab
python your_script.py

# Test with GitHub
franklin backend switch github
python your_script.py
```

## Compatibility Matrix

| Feature | Old GitLab Module | Adapter (GitLab) | Adapter (GitHub) |
|---------|------------------|------------------|------------------|
| create_project | ✅ | ✅ | ✅ |
| get_project | ✅ | ✅ | ✅ |
| list_projects | ✅ | ✅ | ✅ |
| fork_project | ✅ | ✅ | ✅ |
| create_user | ✅ | ✅ | ❌* |
| create_group | ✅ | ✅ | ✅** |
| clone_project | ✅ | ✅ | ✅ |
| create_file | ✅ | ✅ | ✅ |
| create_merge_request | ✅ | ✅ | ✅*** |

\* GitHub doesn't support user creation via API
\** GitHub groups are organizations
\*** GitHub merge requests are pull requests

## Environment Variables

The adapter respects these environment variables:

- `GITLAB_TOKEN` - GitLab personal access token
- `GITHUB_TOKEN` - GitHub personal access token
- `FRANKLIN_USE_BACKEND` - Set to "true" to use adapter
- `FRANKLIN_BACKEND_TYPE` - Override backend type (gitlab/github)

## Troubleshooting

### Import Errors

If you get import errors, ensure the backends module is in your Python path:

```python
import sys
from pathlib import Path
sys.path.insert(0, str(Path(__file__).parent / 'franklin' / 'src'))
```

### Authentication Issues

The adapter tries to auto-authenticate using available tokens:

```python
# Manual authentication if needed
adapter = GitLabCompatibilityAdapter()
adapter.backend.authenticate({'token': 'your-token'})
```

### Backend Detection

Check which backend is being used:

```python
from franklin.backends.adapter import gitlab

print(f"Backend type: {type(gitlab.backend).__name__}")
print(f"Backend URL: {gitlab.url if hasattr(gitlab.backend, 'url') else 'N/A'}")
```

## Advanced Usage

### Custom Backend Configuration

```python
from franklin.backends import BackendFactory
from franklin.backends.adapter import GitLabCompatibilityAdapter

# Create custom backend
backend = BackendFactory.create_backend(
    'github',
    token='your-token',
    base_url='https://github.enterprise.com'  # For GitHub Enterprise
)

# Use with adapter
adapter = GitLabCompatibilityAdapter(backend=backend)
```

### Handling Backend Differences

```python
from franklin.backends.adapter import GitLabCompatibilityAdapter

adapter = GitLabCompatibilityAdapter()

# Check capabilities before using features
if adapter.backend.capabilities.create_users:
    user = adapter.create_user("newuser", "user@example.com", "New User")
else:
    print("Current backend doesn't support user creation")
```

### Parallel Operations with Multiple Backends

```python
from franklin.backends import GitLabBackend, GitHubBackend
from franklin.backends.adapter import GitLabCompatibilityAdapter

# Create adapters for both backends
gitlab_adapter = GitLabCompatibilityAdapter(GitLabBackend(token="..."))
github_adapter = GitLabCompatibilityAdapter(GitHubBackend(token="..."))

# Mirror repository to both
repo_name = "my-project"
gitlab_repo = gitlab_adapter.create_project(repo_name)
github_repo = github_adapter.create_project(repo_name)
```

## Benefits of Using the Adapter

1. **No Code Changes Required** - Existing code works unchanged
2. **Multi-Platform Support** - Same code works with GitLab and GitHub
3. **Gradual Migration** - Update at your own pace
4. **Backward Compatible** - Can always fall back to direct GitLab
5. **Future-Proof** - Easy to add more backends (Bitbucket, Gitea, etc.)

## Next Steps

1. **Test the adapter** with a small part of your code
2. **Configure your preferred backend** using `franklin backend init`
3. **Gradually migrate** imports as convenient
4. **Report issues** if you find incompatibilities

The adapter layer ensures a smooth transition to the multi-backend system without disrupting existing functionality.