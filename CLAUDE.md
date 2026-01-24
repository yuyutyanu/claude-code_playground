# CLAUDE.md

**Documentation for AI Assistants working on claude-code_playground**

Last Updated: 2026-01-24

---

## Repository Overview

**Repository:** yuyutyanu/claude-code_playground
**Purpose:** Development playground and experimentation sandbox
**Current State:** Early stage / minimal setup
**Primary Goal:** Enable flexible development experimentation (including mobile-friendly workflows)

This is a sandbox repository designed for experimental development work. The repository name and README suggest a focus on convenient, accessible development workflows.

---

## Current Repository Structure

```
claude-code_playground/
├── .git/                 # Git repository data
├── README.md            # Project introduction (Japanese)
└── CLAUDE.md           # This file - AI assistant documentation
```

### Key Files

- **README.md**: Brief project description in Japanese ("寝ながらスマホぽちぽちで開発したい所存" - expressing desire to develop conveniently via mobile)
- **CLAUDE.md**: This documentation for AI assistants

---

## Development Workflow

### Git Branching Strategy

**Branch Naming Convention:**
- All feature branches MUST follow the pattern: `claude/<description>-<session-id>`
- Example: `claude/add-feature-abc123`
- Current working branch: `claude/add-claude-documentation-m0esa`

**Critical Git Rules:**
1. Always develop on designated feature branches (never directly on main)
2. Branch names must start with `claude/` and end with a matching session ID
3. Pushing to incorrectly named branches will fail with 403 error

### Git Operations Best Practices

**Pushing Changes:**
```bash
git push -u origin claude/<branch-name>
```
- Always use `-u` flag for initial push
- Retry up to 4 times with exponential backoff (2s, 4s, 8s, 16s) on network errors
- Never force push to main/master without explicit permission

**Fetching/Pulling:**
```bash
git fetch origin <branch-name>
git pull origin <branch-name>
```
- Prefer fetching specific branches
- Apply same retry logic for network failures

**Commit Messages:**
- Use clear, descriptive commit messages
- Focus on "why" rather than "what"
- Follow conventional commit format when possible:
  - `feat:` for new features
  - `fix:` for bug fixes
  - `docs:` for documentation
  - `refactor:` for code refactoring
  - `test:` for tests
  - `chore:` for maintenance

---

## Coding Conventions

### General Principles

Since this is an early-stage playground, establish conventions as the codebase grows:

1. **Simplicity First**: Avoid over-engineering
2. **Minimal Abstractions**: Don't create abstractions for one-time operations
3. **Direct Changes**: No backwards-compatibility hacks unless explicitly needed
4. **Clean Deletions**: Remove unused code completely (no commented-out code)
5. **Security Conscious**: Watch for vulnerabilities (XSS, SQL injection, command injection, etc.)

### File Organization

As the project grows, consider organizing by:
- **Type** (components/, utils/, services/)
- **Feature** (auth/, api/, ui/)
- **Domain** (user/, product/, order/)

Choose the pattern that best fits the project's needs.

### Documentation

- Keep README.md updated with project overview
- Document complex logic inline only when non-obvious
- Maintain this CLAUDE.md as the codebase evolves
- Add API documentation when external interfaces are created

---

## Working with This Repository

### For AI Assistants

**Before Making Changes:**
1. ✓ Read existing files before proposing modifications
2. ✓ Understand the current structure and patterns
3. ✓ Check git status and current branch
4. ✓ Use TodoWrite tool for multi-step tasks

**When Implementing Features:**
1. Keep solutions simple and focused
2. Only make changes that are directly requested
3. Don't add unnecessary features or refactoring
4. Validate at system boundaries (user input, external APIs)
5. Avoid premature optimization

**Code Review Checklist:**
- [ ] Security: No injection vulnerabilities
- [ ] Simplicity: Minimal necessary complexity
- [ ] Testing: Works as intended
- [ ] Documentation: Complex logic explained
- [ ] Cleanup: No unused code or comments

**Task Management:**
- Use TodoWrite tool for complex multi-step tasks
- Mark tasks in_progress when starting
- Mark completed immediately upon finishing
- Keep only one task in_progress at a time

### Repository Evolution

As this repository grows, update this document with:
- New directory structure patterns
- Technology stack decisions
- Build and deployment processes
- Testing conventions
- Code style guides
- API documentation links

---

## Technology Stack

**Current:** Not yet established

**To be documented as the project grows:**
- Programming languages
- Frameworks and libraries
- Build tools
- Testing frameworks
- Deployment targets

---

## Development Environment

### Prerequisites

To be documented as project requirements emerge.

### Setup

```bash
# Clone the repository
git clone <repository-url>
cd claude-code_playground

# Create feature branch
git checkout -b claude/your-feature-$(date +%s)

# (Additional setup steps to be added as needed)
```

---

## Testing

Testing strategy to be established as codebase grows.

Recommended approach:
- Unit tests for business logic
- Integration tests for API endpoints
- E2E tests for critical user flows

---

## Common Tasks

### Adding New Features

1. Create feature branch: `git checkout -b claude/feature-name-<session-id>`
2. Implement changes with clear commits
3. Test thoroughly
4. Push: `git push -u origin claude/feature-name-<session-id>`
5. Create pull request if needed

### Updating Dependencies

To be documented when package managers are added.

### Running the Project

To be documented when project structure is established.

---

## Troubleshooting

### Common Issues

**403 Error on Push:**
- Check branch name follows `claude/<description>-<session-id>` pattern
- Verify you're not pushing to protected branches

**Network Errors:**
- Retry with exponential backoff (2s, 4s, 8s, 16s)
- Check connectivity and proxy settings

---

## Notes for AI Assistants

### Repository Context
- **Type:** Playground/Experimental
- **Maturity:** Early stage
- **Flexibility:** High - conventions are being established
- **Language Context:** Japanese developer (consider bilingual documentation)

### Communication Style
- Be concise and direct
- Avoid over-the-top validation or excessive praise
- Focus on technical accuracy
- Use objective guidance over agreement

### Special Considerations
1. This is a playground - balance structure with experimentation freedom
2. Mobile development workflow may be a consideration
3. Incremental improvements preferred over large refactors
4. Document decisions as patterns emerge

### When in Doubt
- Ask for clarification rather than making assumptions
- Prefer simpler solutions
- Keep changes minimal and focused
- Update this documentation as you learn more about the project

---

## Resources

### Internal Documentation
- README.md - Project overview

### External Resources
- Git documentation: https://git-scm.com/doc
- GitHub guides: https://guides.github.com

---

## Version History

- **2026-01-24**: Initial CLAUDE.md creation
  - Documented minimal repository structure
  - Established git workflow conventions
  - Created framework for future documentation

---

## Contributing

As this is a playground repository:
1. Follow the git branching conventions
2. Keep changes focused and simple
3. Update documentation as patterns emerge
4. Prioritize working code over perfect code

---

**Remember:** This is a living document. Update it as the repository evolves and new patterns emerge.
