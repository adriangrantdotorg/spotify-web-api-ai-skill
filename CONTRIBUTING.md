# 🤝 Contributing to Spotify Web API Agentic Skill

First off, thank you for considering contributing to this project! It's people like you that make this skill a valuable resource for the AI developer community 🙌🏾

## 🌟 Ways to Contribute

There are many ways you can contribute to this project:

- 🐛 **Report bugs** - Found something that doesn't work? Let us know!
- 💡 **Suggest features** - Have an idea for improvement? We'd love to hear it!
- 📝 **Improve documentation** - Help make our docs clearer and more comprehensive
- 🔧 **Fix issues** - Pick up an open issue and submit a fix
- ✨ **Add examples** - Share your Spotify integration patterns with the community
- 🧪 **Write tests** - Help improve code coverage and reliability
- 🎨 **Enhance the skill** - Add new best practices or update existing ones

---

## 🚀 Getting Started

### Prerequisites

Before contributing, make sure you have:

- A **GitHub account**
- **Git** installed on your machine
- An **AI coding assistant** that supports Agent Skills (Claude Code, Cursor, etc.)
- **Spotify Developer credentials** (for testing)
- Basic knowledge of **Markdown** (for documentation)
- Familiarity with **Python/JavaScript** (for code examples)

### Setting Up Your Development Environment

1. **Fork the repository** on GitHub by clicking the "Fork" button

2. **Clone your fork locally**:
   ```bash
   git clone https://github.com/adriangrantdotorg/spotify-web-api-ai-skill.git
   cd spotify-web-api-skill
   ```

3. **Add the upstream remote**:
   ```bash
   git remote add upstream https://github.com/ORIGINAL-OWNER/spotify-web-api-skill.git
   ```

4. **Install the skill in your local environment** (choose your platform):
   ```bash
   # For Claude Code
   ln -s $(pwd) ~/.claude/skills/spotify-web-api
   
   # For OpenCode
   ln -s $(pwd) ~/.config/opencode/skills/spotify-web-api
   
   # For Cursor
   ln -s $(pwd) .cursor/skills/spotify-web-api
   ```

5. **Create a test project** to validate your changes:
   ```bash
   mkdir test-integration
   cd test-integration
   # Test your changes by asking your AI agent to use the skill
   ```

---

## 📝 Contribution Workflow

### 1. Create a Feature Branch

Always create a new branch for your changes:

```bash
# Update your main branch first
git checkout main
git pull upstream main

# Create your feature branch
git checkout -b feature/your-feature-name
```

**Branch naming conventions:**
- `feature/` - New features or enhancements
- `fix/` - Bug fixes
- `docs/` - Documentation updates
- `test/` - Test additions or improvements
- `refactor/` - Code refactoring

Examples:
- `feature/add-playlist-management-pattern`
- `fix/rate-limit-header-parsing`
- `docs/improve-installation-instructions`

### 2. Make Your Changes

#### Updating the Skill Definition (skill/spotify-web-api/SKILL.md)

The `skill/spotify-web-api/SKILL.md` file is the core of the skill. When modifying it:

- ✅ Keep instructions clear and actionable
- ✅ Include code examples with comments
- ✅ Follow the existing structure and formatting
- ✅ Test with your AI agent to ensure instructions work as intended
- ✅ Use consistent markdown formatting

**Example of a good addition:**

```markdown
## 6. Playlist Management Best Practices

### Batch Operations
When adding multiple tracks to a playlist, use batch operations to minimize API calls.

**Before (Inefficient):**
```python
for track_uri in track_uris:
    sp.playlist_add_items(playlist_id, [track_uri])  # Multiple API calls
```

**After (Optimized):**
```python
# Add up to 100 tracks per request
sp.playlist_add_items(playlist_id, track_uris)  # Single API call
```
```

#### Adding Examples

When contributing examples to the `examples/` directory:

1. **Create a descriptive folder name**: `examples/your-example-name/`
2. **Include a README.md** explaining what the example demonstrates
3. **Provide complete, runnable code** with all dependencies listed
4. **Add comments** explaining key concepts
5. **Include environment variable templates** (e.g., `.env.example`)

**Example structure:**
```
examples/
└── react-polling-dashboard/
    ├── README.md                # What this example demonstrates
    ├── package.json             # Dependencies
    ├── .env.example             # Environment variable template
    ├── src/
    │   ├── App.jsx              # Main application
    │   └── hooks/
    │       └── useSpotify.js    # Custom hook with rate limiting
    └── public/
        └── index.html
```

#### Writing Tests

If adding tests to `tests/`:

- Use descriptive test names that explain what's being tested
- Test both success and failure cases
- Mock external API calls to Spotify
- Document any setup requirements

### 3. Test Your Changes

Before submitting, ensure:

- ✅ **The skill works with your AI agent** - Install locally and test with real prompts
- ✅ **Examples run successfully** - Test all code examples
- ✅ **Documentation is accurate** - No broken links or outdated information
- ✅ **Markdown renders correctly** - Preview on GitHub or locally
- ✅ **Tests pass** (if applicable)

**Testing with your AI agent:**

```
# Ask your agent:
"Use the Spotify Web API skill to create a dashboard that polls my currently playing track"

# Verify the agent:
- Uses proper rate limit handling
- Implements the Page Visibility API pattern
- Sets appropriate polling intervals
- Extracts Retry-After headers
```

### 4. Commit Your Changes

Write clear, descriptive commit messages:

```bash
# Good commit messages
git commit -m "Add: playlist batch operations pattern to skill/spotify-web-api/SKILL.md"
git commit -m "Fix: rate limit header parsing in Python example"
git commit -m "Docs: clarify OAuth scope requirements"

# Bad commit messages
git commit -m "update"
git commit -m "fix bug"
git commit -m "changes"
```

**Commit message format:**
```
<Type>: <Short description (50 chars max)>

<Optional detailed explanation (if needed)>
```

**Types:**
- `Add:` - New features, examples, or documentation
- `Fix:` - Bug fixes
- `Update:` - Updates to existing content
- `Remove:` - Removal of deprecated content
- `Docs:` - Documentation-only changes
- `Test:` - Test additions or improvements

### 5. Push and Create a Pull Request

```bash
# Push your branch to your fork
git push origin feature/your-feature-name
```

Then, on GitHub:

1. Navigate to your fork
2. Click **"New Pull Request"**
3. Select your feature branch
4. Fill out the PR template (see below)
5. Click **"Create Pull Request"**

---

## 📋 Pull Request Guidelines

### PR Title Format

Use the same format as commit messages:

```
Add: Support for podcast episode tracking
Fix: Incorrect polling interval in JavaScript example
Docs: Update installation instructions for Cursor IDE
```

### PR Description Template

```markdown
## Description
Brief description of what this PR does.

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Documentation update
- [ ] Example addition
- [ ] Test improvement

## Changes Made
- Bullet point list of specific changes
- Another change
- Yet another change

## Testing
Describe how you tested these changes:
- [ ] Tested with Claude Code
- [ ] Tested with Cursor
- [ ] Ran example code successfully
- [ ] Verified documentation renders correctly

## Checklist
- [ ] I have read the CONTRIBUTING.md guidelines
- [ ] My code follows the project's style
- [ ] I have tested my changes with an AI agent
- [ ] I have updated documentation (if needed)
- [ ] I have added tests (if applicable)
- [ ] All examples run successfully

## Screenshots (if applicable)
Add screenshots or terminal output if relevant.

## Additional Context
Any other information that reviewers should know.
```

---

## 💅 Code Style Guidelines

### Skill Definition Formatting

- Use `##` for main sections, `###` for subsections
- Keep line length reasonable (80-100 characters for prose)
- Use code fences with language identifiers: ` ```python `, ` ```javascript `
- Add comments to code examples
- Use **bold** for emphasis on critical points
- Use `inline code` for variable names, functions, and short code snippets

### Python Code Style

- Follow [PEP 8](https://pep8.org/) guidelines
- Use meaningful variable names
- Add docstrings to functions
- Include type hints where appropriate
- Maximum line length: 88 characters (Black formatter standard)

**Example:**
```python
def handle_rate_limit(exception: spotipy.exceptions.SpotifyException) -> int:
    """
    Extract the Retry-After value from a Spotify 429 response.
    
    Args:
        exception: The SpotifyException with HTTP 429 status
        
    Returns:
        int: Seconds to wait before retrying (default: 5)
    """
    if exception.http_status == 429:
        return int(exception.headers.get('Retry-After', 5))
    return 5
```

### JavaScript Code Style

- Use modern ES6+ syntax
- Prefer `const` over `let`, avoid `var`
- Use meaningful variable names
- Add JSDoc comments for functions
- Format with Prettier (2 spaces, single quotes)

**Example:**
```javascript
/**
 * Adjusts polling interval based on page visibility
 * @param {boolean} isVisible - Whether the page is currently visible
 * @returns {number} Polling interval in milliseconds
 */
const getPollInterval = (isVisible) => {
  return isVisible ? 10000 : 60000; // 10s active, 60s hidden
};
```

---

## 🎯 Specific Contribution Ideas

### Beginner-Friendly

- Fix typos in documentation
- Improve existing code examples with better comments
- Add more "Usage Examples" prompts to README.md
- Create a troubleshooting FAQ
- Test installation on a new platform and document the process

### Intermediate

- Add new code examples (e.g., playlist management, search functionality)
- Implement additional best practices in skill/spotify-web-api/SKILL.md
- Create integration tests for examples
- Write a blog post or tutorial using the skill
- Improve error handling patterns

### Advanced

- Add support for advanced Spotify features (Markets, Recommendations)
- Create comprehensive test suite with mocked API responses
- Build a skill analyzer tool that validates skill/spotify-web-api/SKILL.md format
- Develop CI/CD workflows for automated testing
- Research and document performance optimizations

---

## 📞 Communication Channels

- **GitHub Issues** - For bug reports and feature requests
- **GitHub Discussions** - For questions and general discussion
- **Pull Requests** - For code contributions

### How to Report Bugs

When reporting bugs, please include:

1. **Description**: Clear description of the issue
2. **Platform**: Which AI assistant you're using (Claude Code, Cursor, etc.)
3. **Steps to Reproduce**: Detailed steps to reproduce the behavior
4. **Expected Behavior**: What you expected to happen
5. **Actual Behavior**: What actually happened
6. **Prompt Used**: The prompt you gave to the AI agent
7. **Code Sample**: Relevant code snippets or screenshots
8. **Environment**: OS, Python/Node version, spotipy version, etc.

### How to Suggest Features

When suggesting features, please include:

1. **Problem Statement**: What problem does this solve?
2. **Proposed Solution**: How would you implement this?
3. **Alternatives Considered**: Other approaches you thought about
4. **Use Cases**: Real-world scenarios where this would be useful
5. **Examples**: Code examples or mockups if applicable

---

## ⚖️ Code of Conduct

We are committed to providing a welcoming and inclusive environment for all contributors.

### Our Standards

**We encourage:**
- ✅ Using welcoming and inclusive language
- ✅ Being respectful of differing viewpoints
- ✅ Accepting constructive criticism gracefully
- ✅ Focusing on what's best for the community
- ✅ Showing empathy towards others

**We do not tolerate:**
- ❌ Harassment or discriminatory language
- ❌ Trolling or insulting comments
- ❌ Personal or political attacks
- ❌ Publishing others' private information
- ❌ Unethical or unprofessional conduct

### Enforcement

Violations of the Code of Conduct may result in:
1. A warning from maintainers
2. Temporary ban from the project
3. Permanent ban from the project

Report violations to [INSERT EMAIL].

---

## 🏆 Recognition

All contributors will be recognized in our project! We maintain:

- **Contributors List** in README.md
- **Release Notes** crediting contributors for each version
- **Hall of Fame** for significant contributions

Your contributions, no matter how small, are valued and appreciated!

---

## 📜 License

By contributing to this project, you agree that your contributions will be licensed under the same **MIT License** that covers the project.

---

## ❓ Questions?

If you have questions about contributing, please:

1. Check existing GitHub Issues and Discussions
2. Review this CONTRIBUTING.md guide
3. Open a new Discussion thread
4. Reach out to maintainers via GitHub

---

## 🙏 Thank You!

Your contributions make this project better for everyone in the AI developer community. We appreciate your time, effort, and expertise!

Happy contributing! 🎉

---

<div align="center">
  <sub>Made with ❤️ by contributors like you</sub>
</div>
