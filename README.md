# Claude Code Configurations

Modular configurations for Claude Code with specialized agents, skills, and commands for different tech stacks.

## Quick Start

```bash
# Clone the repo
git clone https://github.com/YOUR_USER/claude-code-configs.git

# Copy a stack to your project
cp -r stacks/react-tailwind/.claude /path/to/my-project/
cp stacks/react-tailwind/CLAUDE.md /path/to/my-project/
```

## Available Stacks

| Stack | Description | Key Technologies |
|-------|-------------|------------------|
| `react-tailwind` | Modern React SPA | React 19, TypeScript, Tailwind CSS v4, Vitest |
| `nextjs-fullstack` | Full-stack Next.js | Next.js 15, App Router, Server Actions, Prisma |
| `python-fastapi` | REST API backend | FastAPI, SQLAlchemy 2.0, Pydantic v2, pytest |
| `python-scripts` | Scripts & automation | Python 3.12+, Click, Rich, asyncio |
| `payload-cms` | Headless CMS | Payload 3.0, TypeScript, MongoDB/Postgres |
| `python-scraping` | Data pipeline API | FastAPI, OpenAI, PostgreSQL, WebSockets, Scraping |

## Structure

Each stack is self-contained with everything you need:

```
stacks/
├── react-tailwind/
│   ├── .claude/
│   │   ├── agents/           # All agents (stack + shared)
│   │   ├── skills/           # Framework skills
│   │   └── commands/         # Custom commands
│   └── CLAUDE.md             # Project context
│
├── nextjs-fullstack/
├── python-fastapi/
├── python-scripts/
├── payload-cms/
└── python-scraping/
```

## Agents

Each stack includes these agents:

### Core Agents
- **feature-developer** - Implements features from specs
- **code-reviewer** - Reviews code (read-only)
- **test-writer** - Writes comprehensive tests

### Shared Agents (included in all stacks)
- **git-commit** - Writes semantic commits (Conventional Commits)
- **docs-writer** - Technical documentation
- **refactor-agent** - Code improvement without behavior changes
- **security-reviewer** - Security audit (OWASP, vulnerabilities)
- **performance-optimizer** - Performance analysis and optimization

### Stack-Specific
- **component-builder** (react-tailwind) - UI components with Tailwind
- **api-developer** (nextjs-fullstack) - API routes specialist
- **pipeline-developer** (python-scraping) - Data pipeline specialist

## Skills

Skills provide domain knowledge that Claude loads automatically:

| Stack | Skills |
|-------|--------|
| react-tailwind | react19, tailwind-v4, tanstack-query |
| nextjs-fullstack | nextjs15, prisma |
| python-fastapi | fastapi, sqlalchemy-async |
| python-scripts | cli |
| payload-cms | payload |
| python-scraping | web-scraping, openai-extraction, websockets, background-tasks |

## Commands

Custom slash commands available in all stacks:

- `/review-changes` - Review git changes before commit/PR
- `/new-component` - Generate React component (react-tailwind)

## Usage Examples

### React Project
```
"Here's the feature spec for user authentication, implement it"
→ feature-developer creates components, hooks, types

"Create a modal component with animations"
→ component-builder creates polished UI

"Review the auth code before I commit"
→ code-reviewer analyzes without modifying

"Check for security issues"
→ security-reviewer audits for vulnerabilities
```

### Python API Project
```
"Add a new endpoint for user exports with background processing"
→ feature-developer creates route, service, models

"Write tests for the export functionality"
→ test-writer creates pytest tests with fixtures

"Optimize the database queries"
→ performance-optimizer identifies N+1 and suggests fixes
```

## Customization

### Adding Project-Specific Context

Edit `CLAUDE.md` in your project to add:
- Project-specific patterns
- Team conventions
- Environment details
- API documentation links

### Creating New Skills

```markdown
# .claude/skills/my-skill/SKILL.md
---
name: my-skill
description: What it does and when to trigger it
---

# Skill content here
```

### Creating New Agents

```markdown
# .claude/agents/my-agent.md
---
name: my-agent
description: When to use this agent
tools: Read, Write, Edit, Bash
model: sonnet
---

# Agent instructions here
```

## License

MIT
