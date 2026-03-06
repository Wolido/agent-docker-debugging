# Agent Docker Debugging

A collection of Claude Code / Kimi Code CLI skills for Docker container debugging and containerization workflows.

## Overview

This repository contains skills that help AI agents diagnose and fix Docker deployment issues, with a strong focus on the **"works on my machine"** problem.

## Skills

### agent-docker-debugging

The main skill for debugging Docker containers and containerized applications.

**Core Principle:**
> **"Works on my machine" means nothing. Verify in Docker before declaring completion.**

**Features:**
- **Pre-Deployment Checklist** - Mandatory verification steps to catch issues before production
- **Docker Compose Debugging** - Multi-service orchestration troubleshooting  
- **Language-Specific Guides** - Containerization patterns for Rust, Go, Python, and Node.js
- **Common Issues Reference** - Systematic troubleshooting for frequent Docker problems

**Installation:**

```bash
# For Claude Code
git clone https://github.com/YOUR_USERNAME/agent-docker-debugging.git
ln -s $(pwd)/agent-docker-debugging/skills/agent-docker-debugging \
  ~/.claude/skills/agent-docker-debugging

# For Kimi Code CLI
git clone https://github.com/YOUR_USERNAME/agent-docker-debugging.git
ln -s $(pwd)/agent-docker-debugging/skills/agent-docker-debugging \
  ~/.config/agents/skills/agent-docker-debugging
```

**Usage:**

This skill automatically activates when you mention:
- Docker container issues
- "Works locally but not in Docker"
- Docker Compose problems
- Pre-deployment verification needs
- Container debugging

## Repository Structure

```
agent-docker-debugging/
├── README.md                          # This file
└── skills/
    └── agent-docker-debugging/        # The skill
        ├── SKILL.md                   # Main skill entry point
        └── references/
            ├── pre-deployment-checklist.md
            ├── docker-compose-debugging.md
            ├── language-specific-issues.md
            ├── common-container-issues.md
            ├── docker-debugging-basics.md
            ├── debugging-checklist.md
            └── container-optimization.md
```

## Author

This extended version of the skill was created by **Kimi Code CLI (小顺子)** with guidance from **Wolido**.

Built with practical experience from real-world Docker deployment debugging scenarios.

## Acknowledgments

The `agent-docker-debugging` skill is based on the original [container-debugging skill](https://github.com/aj-geddes/useful-ai-prompts/tree/master/skills/container-debugging) by **aj-geddes**.

Key extensions and additions:
- Added comprehensive **Pre-Deployment Checklist** with automated verification script
- Added **Docker Compose Debugging** guide for multi-service architectures
- Added **Language-Specific Issues** with practical examples for Rust, Go, Python, and Node.js
- Expanded troubleshooting coverage for "Local Works, Docker Doesn't" scenarios
- Converted all reference tables to YAML format per skill best practices

Thank you to aj-geddes for creating the original skill and sharing it with the community.

## License

MIT License - See [LICENSE](LICENSE) file for details.

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.
