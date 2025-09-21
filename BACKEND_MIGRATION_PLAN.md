# Backend Migration Plan - Phase 1 Implementation

## Overview

Phase 1 of the Git Backend Interface has been implemented without modifying any existing Franklin code. This document describes how to integrate the new backend system with the existing codebase in future phases.

## What Has Been Implemented

### New Files Created

1. **`franklin/src/franklin/backends/__init__.py`**
   - Main module initialization
   - Exports all backend classes and utilities

2. **`franklin/src/franklin/backends/base.py`**
   - Abstract `GitBackend` base class
   - Data models: `Repository`, `User`, `Group`, `MergeRequest`
   - Capability system: `BackendCapabilities`
   - Custom exceptions for backend operations

3. **`franklin/src/franklin/backends/gitlab_backend.py`**
   - Complete GitLab implementation of `GitBackend`
   - Wraps python-gitlab library
   - Converts GitLab objects to generic models

4. **`franklin/src/franklin/backends/factory.py`**
   - `BackendFactory` class for creating backends
   - Configuration loading from YAML/JSON
   - Backend registration system
   - Environment variable expansion

5. **`franklin/src/franklin/backends/config.py`**
   - `BackendConfig` class for managing settings
   - Interactive configuration initialization
   - Configuration validation

## Migration Strategy for Phase 2

### Step 1: Create Adapter Layer (Week 1)

Create an adapter that makes the existing `gitlab.py` module use the new backend system:

```python
# franklin/src/franklin/gitlab_adapter.py
from franklin.backends import get_backend

class GitLabAdapter:
    """Adapter to make existing code work with new backend system."""
    
    def __init__(self):
        self.backend = get_backend('gitlab')
    
    # Existing function signatures mapped to backend methods
    def create_project(self, name, **kwargs):
        repo = self.backend.create_repository(name, **kwargs)
        return self._convert_to_legacy_format(repo)
    
    def get_project(self, project_id):
        repo = self.backend.get_repository(project_id)
        return self._convert_to_legacy_format(repo)
    
    # ... more adapter methods ...
```

### Step 2: Update Import Points (Week 1)

Identify all files that import from `franklin.gitlab` and update them to use the adapter:

**Files to Update:**
- `franklin/src/franklin/__init__.py` - Main CLI commands
- `franklin/src/franklin/docker.py` - Exercise downloading
- `franklin/src/franklin/config.py` - GitLab configuration
- `franklin_educator/src/franklin_educator/git.py` - Exercise management

**Update Pattern:**
```python
# Old
from franklin import gitlab

# New (Phase 2 - with adapter)
from franklin.gitlab_adapter import GitLabAdapter
gitlab = GitLabAdapter()

# Future (Phase 3 - direct backend usage)
from franklin.backends import get_backend
backend = get_backend()  # Auto-detects from config
```

### Step 3: Gradual Method Migration (Week 2)

Replace GitLab-specific calls with backend-agnostic calls:

```python
# Old: GitLab-specific
project = gl.projects.create({
    'name': name,
    'visibility': 'private'
})

# New: Backend-agnostic
repo = backend.create_repository(
    name=name,
    visibility='private'
)
```

### Step 4: Configuration Migration (Week 2)

Migrate existing Franklin configuration to new backend configuration:

```python
# Migration script: migrate_config.py
import yaml
from pathlib import Path

def migrate_franklin_config():
    """Migrate existing config to backend config."""
    old_config_path = Path.home() / '.franklin' / 'config.yaml'
    new_config_path = Path.home() / '.franklin' / 'backend.yaml'
    
    if old_config_path.exists():
        with open(old_config_path) as f:
            old_config = yaml.safe_load(f)
        
        # Extract GitLab settings
        new_config = {
            'backend': {
                'type': 'gitlab',
                'settings': {
                    'url': old_config.get('gitlab_url', 'https://gitlab.com'),
                    'token': old_config.get('gitlab_token'),
                },
                'defaults': {
                    'visibility': old_config.get('default_visibility', 'private'),
                }
            }
        }
        
        # Save new configuration
        with open(new_config_path, 'w') as f:
            yaml.dump(new_config, f)
```

## Integration Points

### 1. CLI Commands

Update Click commands to use backend factory:

```python
# franklin/src/franklin/__init__.py
from franklin.backends import get_backend

@click.command()
@click.option('--backend', type=click.Choice(['gitlab', 'github']), 
              help='Backend to use')
def download(url, backend):
    """Download exercise from repository."""
    backend_instance = get_backend(backend)
    # ... use backend_instance ...
```

### 2. Exercise Management

Update exercise creation/distribution:

```python
# franklin_educator/src/franklin_educator/git.py
from franklin.backends import get_backend

class ExerciseManager:
    def __init__(self):
        self.backend = get_backend()
    
    def create_exercise(self, name):
        repo = self.backend.create_repository(
            name=f"exercise-{name}",
            visibility='private'
        )
        # Add exercise files
        self.backend.create_file(
            repo_id=repo.id,
            file_path='README.md',
            content=self.get_exercise_template(),
            message='Initial exercise setup'
        )
```

### 3. User Management

Update user/group operations:

```python
# franklin_admin/src/franklin_admin/users.py
from franklin.backends import get_backend

def create_course_group(course_name, students):
    backend = get_backend()
    
    # Create group
    group = backend.create_group(
        name=course_name,
        description=f"Group for {course_name}"
    )
    
    # Add students
    for student_email in students:
        user = backend.get_user(student_email)
        backend.add_user_to_group(group.id, user.id)
```

## Testing Strategy

### Unit Tests

Create tests for each backend implementation:

```python
# tests/test_backends.py
import pytest
from unittest.mock import Mock, patch
from franklin.backends import GitLabBackend, BackendFactory

class TestGitLabBackend:
    def test_create_repository(self):
        backend = GitLabBackend(url='https://gitlab.com')
        with patch('gitlab.Gitlab') as mock_gitlab:
            # Test repository creation
            pass

class TestBackendFactory:
    def test_create_backend(self):
        backend = BackendFactory.create_backend('gitlab')
        assert isinstance(backend, GitLabBackend)
```

### Integration Tests

Test with real GitLab instance (optional):

```python
# tests/integration/test_gitlab_integration.py
@pytest.mark.integration
def test_gitlab_operations():
    backend = GitLabBackend(url=TEST_GITLAB_URL, token=TEST_TOKEN)
    backend.authenticate({'token': TEST_TOKEN})
    
    # Create test repository
    repo = backend.create_repository('test-repo')
    assert repo.name == 'test-repo'
    
    # Cleanup
    backend.delete_repository(repo.id)
```

## Rollback Plan

If issues arise during migration:

1. **Keep existing code intact** - Don't delete `gitlab.py` until fully migrated
2. **Feature flag** - Add environment variable to toggle between old and new:
   ```python
   if os.environ.get('USE_NEW_BACKEND'):
       from franklin.backends import get_backend
       backend = get_backend()
   else:
       from franklin import gitlab
       backend = gitlab  # Use old module
   ```
3. **Gradual rollout** - Migrate one command at a time
4. **Comprehensive testing** - Ensure each migrated component works before proceeding

## Phase 3: Adding GitHub Support

Once GitLab migration is complete:

1. **Implement GitHub backend**:
   ```python
   # franklin/src/franklin/backends/github_backend.py
   from github import Github
   from .base import GitBackend
   
   class GitHubBackend(GitBackend):
       # Implementation
   ```

2. **Register with factory**:
   ```python
   BackendFactory.register_backend('github', GitHubBackend)
   ```

3. **Add backend switching command**:
   ```python
   @click.command()
   def switch_backend():
       """Switch between git backends."""
       # Interactive backend selection
   ```

## Benefits of This Approach

1. **No breaking changes** - Existing code continues to work
2. **Gradual migration** - Can be done incrementally
3. **Testing at each step** - Ensures stability
4. **Rollback capability** - Can revert if issues arise
5. **Clean architecture** - Clear separation of concerns

## Next Steps

1. **Review and approve** this migration plan
2. **Create test suite** for backend implementations
3. **Begin Phase 2** - Create adapter layer
4. **Document changes** for users
5. **Plan Phase 3** - GitHub implementation

## Timeline

- **Week 1**: Create adapter layer and update imports
- **Week 2**: Migrate methods and configuration
- **Week 3**: Testing and bug fixes
- **Week 4**: Documentation and user communication
- **Week 5-6**: GitHub backend implementation
- **Week 7-8**: Final testing and release

## Risks and Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Breaking existing functionality | High | Keep old code, use adapter pattern |
| Performance degradation | Medium | Profile and optimize hot paths |
| Configuration complexity | Low | Provide migration tools |
| User confusion | Medium | Clear documentation and examples |

## Success Criteria

- [ ] All existing Franklin commands work with new backend
- [ ] No performance regression (< 5% slower)
- [ ] Configuration migration tool works
- [ ] Tests pass with > 90% coverage
- [ ] Documentation updated
- [ ] GitHub backend functional

## Conclusion

Phase 1 has successfully created a robust backend abstraction layer without modifying any existing Franklin code. The migration plan provides a clear path to integrate this new system while maintaining backward compatibility and minimizing risk.