# Franklin Video Recording Guide

This guide lists all the screencasts needed for the Franklin documentation, organized by priority and estimated duration.

## Video Recording Setup

### Recommended Tools
- **macOS**: QuickTime Player (built-in) or OBS Studio
- **Windows**: OBS Studio or Windows Game Bar
- **Linux**: OBS Studio or SimpleScreenRecorder

### Recording Guidelines
- **Resolution**: 1920x1080 (1080p) minimum
- **Format**: MP4 (H.264 codec)
- **Audio**: Include narration for complex workflows
- **Duration**: Keep videos concise (2-5 minutes ideal)
- **Speed**: Normal speed for commands, can speed up waiting/loading

### File Naming Convention
- Use lowercase with hyphens: `install-python-overview.mp4`
- Place in `docs/pages/videos/` directory
- Use descriptive names matching the placeholders

## Priority 1: Essential Videos (For Beginners)

These videos are critical for new users who need visual guidance.

### 1. `install-python-overview.mp4` (2 min)
**Location**: Getting Started for Beginners
**Content**:
- Show Miniforge download page
- Download installer
- Run installation (both Windows and Mac if possible)
- Verify installation in terminal

### 2. `first-exercise-complete-workflow.mp4` (5 min)
**Location**: Getting Started for Beginners
**Content**:
- Open terminal
- Run `franklin download`
- Show course selection interface
- Show exercise selection
- Demonstrate download completion
- Run `franklin jupyter`
- Show JupyterLab opening

### 3. `working-in-jupyterlab.mp4` (3 min)
**Location**: Getting Started for Beginners
**Content**:
- JupyterLab interface tour
- Opening a notebook
- Running cells (Shift+Enter)
- Adding new cells
- Saving work
- Basic markdown and code cells

### 4. `troubleshoot-docker-not-running.mp4` (2 min)
**Location**: Troubleshooting Guide
**Content**:
- Show the error message
- Open Docker Desktop
- Wait for Docker to start
- Retry Franklin command
- Show it working

## Priority 2: Educator Videos

Essential for educators creating exercises.

### 5. `educator-create-first-exercise.mp4` (5 min)
**Location**: Educator Getting Started
**Content**:
- Run `franklin exercise new`
- Enter course name
- Enter exercise name
- Show created file structure
- Open exercise.ipynb in Jupyter
- Add simple content
- Run `pixi run test-notebook`
- Publish exercise

### 6. `exercise-types-overview.mp4` (4 min)
**Location**: Creating Exercises
**Content**:
- Show tutorial exercise example
- Show problem-solving exercise
- Show data analysis exercise
- Show debugging exercise
- Explain when to use each

### 7. `adding-interactive-widgets.mp4` (3 min)
**Location**: Creating Exercises
**Content**:
- Import ipywidgets
- Create slider widget
- Create interactive plot
- Show button interactions
- Demonstrate in student view

### 8. `setting-up-automated-tests.mp4` (4 min)
**Location**: Creating Exercises
**Content**:
- Create tests directory
- Write test_notebook.py
- Run pytest locally
- Show CI/CD configuration
- Demonstrate test failure and success

## Priority 3: Administrator Videos

For system administrators setting up Franklin.

### 9. `admin-complete-setup.mp4` (10 min)
**Location**: Admin Setup Guide
**Content**:
- System preparation
- Install dependencies
- Install Franklin Admin
- Initial configuration
- Create first admin user
- Verify installation

### 10. `gitlab-setup-configuration.mp4` (5 min)
**Location**: Admin Setup Guide
**Content**:
- Create GitLab token
- Configure Franklin for GitLab
- Create group structure
- Set up container registry
- Test connection

### 11. `user-management-overview.mp4` (3 min)
**Location**: User Management Guide
**Content**:
- Create individual user
- Assign roles
- Check user permissions
- Modify user
- Deactivate user

### 12. `bulk-user-import.mp4` (2 min)
**Location**: User Management Guide
**Content**:
- Create CSV file
- Show CSV format
- Run import command
- Verify users created
- Handle errors

## Priority 4: Advanced Videos

For experienced users wanting to leverage advanced features.

### 13. `advanced-workflows-demo.mp4` (5 min)
**Location**: Python Users Guide
**Content**:
- Custom Docker containers
- Batch processing exercises
- Git integration
- Performance optimization

### 14. `developer-setup-walkthrough.mp4` (6 min)
**Location**: Developer Guide
**Content**:
- Clone repository
- Set up development environment
- Install in editable mode
- Run tests
- Make a simple change
- Submit pull request

### 15. `franklin-overview-demo.mp4` (4 min)
**Location**: Main Index Page
**Content**:
- High-level platform overview
- Show student workflow
- Show educator workflow
- Show admin interface
- Demonstrate key features

## Recording Script Templates

### Script for `first-exercise-complete-workflow.mp4`

```
"Hi, I'll show you how to download and work on your first Franklin exercise.

First, I'll open my terminal. On Windows, this is the Miniforge Prompt, on Mac it's Terminal.

Now I'll type 'franklin download' and press Enter.

Franklin shows me available courses. I'll use the arrow keys to select 'Introduction to Python' and press Enter.

Now I see the available exercises. I'll select 'Week 1 - Getting Started'.

Great! Franklin has downloaded the exercise to my computer and shows me exactly where it is.

Now let's open it. I'll type 'franklin jupyter'.

I need to select the same course and exercise again.

Franklin is now setting up everything I need - installing packages, configuring the environment...

And here we go! JupyterLab has opened in my browser. I can see my exercise notebook here.

Let me click on it to open... and now I can start working on the exercise!"
```

### Script for `troubleshoot-docker-not-running.mp4`

```
"If you see the error 'Docker is not running', here's how to fix it.

First, I need to start Docker Desktop. On Windows, I'll find it in the Start Menu. On Mac, it's in Applications.

I'll click to open Docker Desktop.

Now I need to wait for Docker to fully start. You'll see this whale icon when it's ready.

The status should change to 'Docker Desktop is running'.

Now let's go back to our terminal and try the Franklin command again.

Perfect! It's working now. Remember, Docker needs to be running whenever you use Franklin."
```

## Video Storage and Optimization

### File Organization
```
franklin/docs/pages/videos/
├── beginner/
│   ├── install-python-overview.mp4
│   ├── first-exercise-complete-workflow.mp4
│   └── working-in-jupyterlab.mp4
├── educator/
│   ├── educator-create-first-exercise.mp4
│   ├── exercise-types-overview.mp4
│   └── adding-interactive-widgets.mp4
├── admin/
│   ├── admin-complete-setup.mp4
│   ├── gitlab-setup-configuration.mp4
│   └── user-management-overview.mp4
└── advanced/
    ├── advanced-workflows-demo.mp4
    └── developer-setup-walkthrough.mp4
```

### Video Compression
```bash
# Compress videos using ffmpeg
ffmpeg -i input.mp4 -c:v libx264 -preset slow -crf 23 -c:a aac -b:a 128k output.mp4

# Batch compress all videos
for f in *.mp4; do
    ffmpeg -i "$f" -c:v libx264 -preset slow -crf 23 -c:a aac -b:a 128k "compressed_$f"
done
```

### Alternative: YouTube/Vimeo Embedding

If hosting videos locally is not feasible, you can use YouTube or Vimeo:

```markdown
{{< video https://www.youtube.com/embed/VIDEO_ID >}}
{{< video https://player.vimeo.com/video/VIDEO_ID >}}
```

## Checklist

### High Priority (Complete First)
- [ ] install-python-overview.mp4
- [ ] first-exercise-complete-workflow.mp4
- [ ] working-in-jupyterlab.mp4
- [ ] troubleshoot-docker-not-running.mp4
- [ ] franklin-overview-demo.mp4

### Medium Priority (Educators)
- [ ] educator-create-first-exercise.mp4
- [ ] exercise-types-overview.mp4
- [ ] adding-interactive-widgets.mp4
- [ ] setting-up-automated-tests.mp4

### Lower Priority (Admins & Advanced)
- [ ] admin-complete-setup.mp4
- [ ] gitlab-setup-configuration.mp4
- [ ] user-management-overview.mp4
- [ ] bulk-user-import.mp4
- [ ] advanced-workflows-demo.mp4
- [ ] developer-setup-walkthrough.mp4

## Notes

- Focus on the high-priority videos first as they help the most users
- Keep videos concise - users prefer short, focused content
- Consider creating animated GIFs for very simple operations
- Update videos when UI changes significantly
- Add captions/subtitles for accessibility if possible