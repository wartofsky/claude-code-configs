# Skills Maintenance

This repo stores curated stack-level skills under:

```text
stacks/<stack>/.claude/skills/<skill>/SKILL.md
```

Project-specific `CLAUDE.md` / `AGENTS.md` files are intentionally not stored in stack templates.

## Curated Installation Workflow

Never install a marketplace skill directly into a stack without reviewing it first.

1. Search:

```bash
npx skills find "react performance"
npx skills find "fastapi pytest"
npx skills find "nestjs postgres"
```

2. Stage in temp:

```bash
rm -rf /tmp/skills-staging
mkdir -p /tmp/skills-staging
cd /tmp/skills-staging
npx skills add <owner/repo> --skill <skill-name> --copy -y
```

3. Inspect:

```bash
sed -n '1,220p' /tmp/skills-staging/.claude/skills/<skill-name>/SKILL.md
find /tmp/skills-staging/.claude/skills/<skill-name> -maxdepth 3 -type f
```

4. Copy only if approved:

```bash
rm -rf stacks/<stack>/.claude/skills/<skill-name>
cp -R /tmp/skills-staging/.claude/skills/<skill-name> \
  stacks/<stack>/.claude/skills/<skill-name>
```

5. Update `docs/skills-matrix.md` with source, rationale, and assignment.

## Review Checklist

- Source is trusted or high-install and low-risk.
- Skill is current for the stack's framework versions.
- Skill does not impose incompatible assumptions.
- Skill frontmatter is valid.
- Skill does not introduce secrets, binaries, scripts, or unexpected generated files.
- Skill complements rather than duplicates better custom stack guidance.

## Useful Commands

```bash
# List skills in a stack
find stacks/<stack>/.claude/skills -maxdepth 2 -name SKILL.md -print

# Validate basic frontmatter
python3 - <<'PY'
from pathlib import Path
bad=[]
for p in Path('stacks').glob('*/.claude/skills/*/SKILL.md'):
    text=p.read_text(errors='replace')
    if not text.startswith('---') or 'name:' not in text.split('---',2)[1] or 'description:' not in text.split('---',2)[1]:
        bad.append(str(p))
print('OK' if not bad else '\n'.join(bad))
PY
```
