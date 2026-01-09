---
layout: post
title: "Understanding Agent Skills: From Concept to Implementation"
date: 2026-01-09 00:17:21 +0000
categories: [AI, developer-tools]
tags: [AI, agentic-coding, developer-tools]
---

# Understanding Agent Skills: From Concept to Implementation

![Terminal with code](/assets/images/skills/terminal-code-hero.jpg)

At the beginning of 2025, developers were growing accustomed to AI chatbots appearing in their [IDE](/tags/2026/01/09/ide.html)s. By year's end, a fundamental shift had occurred—we moved from passive suggestion to [agentic-coding](/tags/2026/01/09/agentic-coding.html), where [AI](/tags/2026/01/09/ai.html) agents don't just respond but actively plan, execute, verify, and iterate across entire codebases directly from the terminal.

This evolution brought a critical challenge: while [Claude-Code](/tags/2026/01/09/claude-code.html), [OpenCode](/tags/2026/01/09/opencode.html), and [Vibe-CLI](/tags/2026/01/09/vibe-cli.html) are remarkably capable general-purpose agents, they lack domain-specific expertise. An agent might write excellent Python but struggle with your organization's specific coding standards, security protocols, or deployment workflows.

**Agent Skills** solve this problem. They're organized folders of instructions, scripts, and resources that agents can discover and load dynamically—transforming general-purpose assistants into specialized experts for any domain.

In this guide, I'll walk through what Agent Skills are, how they work architecturally, their implementation across major tools, and how to build your first skill from scratch.

---

## What Are Agent Skills?

Agent Skills are modular, markdown-based capabilities that extend an [AI](/tags/2026/01/09/ai.html) agent's functionality. Think of them as **organized instruction manuals** that an agent can consult when relevant to your request.

At their simplest, a skill is a directory containing a `SKILL.md` file with YAML frontmatter and markdown instructions:

```markdown
---
name: code-review
description: Review code changes using company standards and security guidelines.
  Use when reviewing PRs, code changes, or before merging.
---

# Code Review Skill

When reviewing code, always check:

1. **Security**: SQL injection, XSS, authentication issues
2. **Performance**: N+1 queries, unnecessary loops
3. **Style**: Follow company coding standards
4. **Tests**: Verify test coverage exists

See [SECURITY.md](SECURITY.md) for detailed security checklist.
```

### Skills vs. Slash Commands

It's worth distinguishing Agent Skills from slash commands, as they serve different purposes:

| Feature | Agent Skills | Slash Commands |
|---------|-------------|----------------|
| **Invocation** | Auto-discovered based on context | Explicit `/command` required |
| **Complexity** | Multi-file with supporting resources | Usually single-file prompts |
| **Discovery** | Agent reads descriptions, applies contextually | User must know command exists |
| **Use Case** | Domain expertise, workflows | Quick templates, reminders |

The key difference: skills activate **automatically** based on natural language, while slash commands require manual invocation. When you ask "Can you review this code?", a properly configured code-review skill activates without you typing `/review`.

---

## The Architecture: Progressive Disclosure

![Code editor](/assets/images/skills/code-editor.jpg)

The architecture behind Agent Skills uses a principle called **progressive disclosure**—loading information only as needed. This keeps context windows lean while allowing effectively unbounded skill complexity.

### Three Levels of Information Loading

**Level 1: Metadata (Always Loaded)**

The agent loads only the skill's name and description at startup. This costs approximately 100 tokens per skill:

```yaml
---
name: pdf-processing
description: Extract text and tables from PDF files, fill forms, merge documents.
  Use when working with PDFs, forms, or document extraction.
---
```

This metadata helps the agent decide when to apply each skill without consuming context for unused capabilities.

**Level 2: Instructions (Loaded When Triggered)**

When your request matches a skill's description, the agent asks permission and loads the full `SKILL.md` body—typically 5,000 tokens or less:

```markdown
# PDF Processing

## Quick Start

Extract text using pdfplumber:

```python
import pdfplumber
with pdfplumber.open("document.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

For form filling workflows, see [FORMS.md](FORMS.md).
```

**Level 3: Resources (Loaded As Needed)**

Supporting files, scripts, and reference materials load dynamically only when referenced:

```
pdf-processing/
├── SKILL.md          # Overview - always loaded when triggered
├── FORMS.md          # Form-filling guide - loaded only if needed
├── REFERENCE.md      # API documentation - loaded only if needed
└── scripts/
    ├── validate.py   # Runs without loading into context
    └── fill_form.py  # Executed directly
```

### Why This Matters

The progressive disclosure model means:

1. **Unbounded Context**: Skills can bundle unlimited documentation—agents only load what's relevant
2. **Efficient Loading**: Main conversation stays clean; skill details load silently
3. **Filesystem Navigation**: Agents treat skills like a well-organized manual, accessing exactly what each task requires

A PDF processing skill might contain 50 pages of documentation and 10 utility scripts. Without progressive disclosure, this would consume your entire context window. With it, the agent loads only the quick-start section unless form-filling is specifically requested.

---

## Implementation Across Tools

The Agent Skills format has been adopted across multiple [CLI-tools](/tags/2026/01/09/cli-tools.html), each with slight variations but compatible core structures.

### Claude Code: The Original Implementation

[Claude-Code](/tags/2026/01/09/claude-code.html) pioneered the skills architecture. Skills live in two locations:

```bash
# Personal skills (all projects)
~/.claude/skills/skill-name/SKILL.md

# Project skills (shared with team via git)
.claude/skills/skill-name/SKILL.md
```

The YAML frontmatter supports several fields:

```yaml
---
name: security-review         # Required: lowercase, hyphens only
description: |                # Required: max 1024 chars
  Analyze code for security vulnerabilities and OWASP Top 10 violations.
  Use when reviewing security-sensitive code.
allowed-tools: Read, Grep     # Optional: restrict tool access
model: claude-opus-4-5        # Optional: specific model for this skill
context: fork                 # Optional: run in sub-agent context
---
```

### OpenCode: Open Source Alternative

[OpenCode](/tags/2026/01/09/opencode.html) is a Go-based, model-agnostic [open-source](/tags/2026/01/09/open-source.html) alternative that maintains compatibility with the Claude Code skills format:

```bash
# OpenCode-specific locations
.opencode/skill/*/SKILL.md
~/.config/opencode/skill/*/SKILL.md

# Claude Code compatibility
.claude/skills/*/SKILL.md
~/.claude/skills/*/SKILL.md
```

OpenCode comes with built-in agents (Build, Plan, General, Explore) and supports any AI model provider, giving developers flexibility without configuration complexity.

### Vibe CLI: Mistral's Approach

[Vibe-CLI](/tags/2026/01/09/vibe-cli.html), Mistral's Python-based terminal agent, takes a different approach with its agent loop pattern:

```
interpret intent → plan steps → execute → verify → iterate
```

Commands like `vibe scan` index your codebase, while `vibe approve` and `vibe iterate` control the agent loop. A key differentiator: Vibe can run entirely locally with Devstral models—your code never leaves your machine.

![Developer workflow](/assets/images/skills/developer-workflow.jpg)

---

## The Open Standard: agentskills.io

On December 18, 2025, Anthropic released Agent Skills as an [open-source](/tags/2026/01/09/open-source.html) specification at [agentskills.io](https://agentskills.io). The specification is, as Simon Willison noted, "deliciously tiny—you can read the entire thing in just a few minutes."

### Industry Adoption

The standard has been adopted by major players:

- **Microsoft** and **GitHub** integrated skills into Copilot
- **OpenAI** launched Skills in Codex using the same standard
- **Cursor**, **Amp**, and **VS Code** provide native support
- **Partner skills** from Canva, Stripe, Notion, and Zapier are available at launch

### Relationship with MCP

Agent Skills complement Anthropic's Model Context Protocol (MCP). The distinction:

- **MCP is the plumbing**: Defines how agents connect to databases, APIs, and external systems
- **Agent Skills is the manual**: Teaches agents how to navigate those connections effectively

Together, they create what some have called "the USB-C for [AI](/tags/2026/01/09/ai.html)"—a universal standard that lowers the barrier to building specialized agents from months of integration work to implementing a single specification.

---

## Lowering the Barrier of Entry

One of the most significant impacts of Agent Skills is [democratization](/tags/2026/01/09/democratization.html) of AI capabilities.

### The Current Challenge

Traditional AI development creates a bottleneck. The Claude Agent SDK requires programming expertise—understanding code architecture, managing deployments, handling errors, and configuring infrastructure. This excludes potential users who could benefit from AI agents but lack technical skills.

Consider: only **0.03% of the global population** has programming skills. Skills help bridge this gap by encapsulating expert knowledge in accessible formats.

### How Skills Democratize AI Capabilities

**Knowledge Encapsulation**

Expert knowledge from senior team members gets encoded in a skill and automatically applied by the agent for everyone:

```yaml
---
name: brand-compliance
description: Ensure marketing content follows brand guidelines, tone, and visual standards.
  Use when creating or reviewing marketing materials.
---

# Brand Compliance

## Voice and Tone
- Professional but approachable
- Active voice preferred
- Avoid jargon unless necessary

## Visual Standards
- Primary color: #2563EB
- Font: Inter for headings, system fonts for body
- Logo minimum size: 24px height

See [BRAND_GUIDE.md](BRAND_GUIDE.md) for complete guidelines.
```

A marketing coordinator uses this skill without learning the technical details—they just create content, and the agent automatically applies brand standards.

**Natural Language Triggers**

Users don't need to memorize commands. They describe what they need:

- "Review this for brand compliance" → skill activates
- "Can you check the PR?" → code review skill activates
- "Extract text from this PDF" → PDF skill activates

**Consistent Standards Without Learning**

New team members get the same quality guidance as veterans. Instead of a 50-page onboarding document, skills encode standards that apply automatically.

### The Remaining Gap

Multi-agent orchestration and advanced workflows still require significant technical expertise. As Gergely Orosz observed, "parallel agent work demands skills typically honed by experienced tech leads." Skills help narrow this gap but don't eliminate it entirely—yet.

---

## Building Your First Skill

Let's build a practical skill: a commit message generator that follows conventional commits.

### Step 1: Create the Directory

```bash
mkdir -p ~/.claude/skills/commit-generator
```

### Step 2: Write the SKILL.md

Create `~/.claude/skills/commit-generator/SKILL.md`:

```yaml
---
name: commit-generator
description: Generate clear, conventional commit messages from staged changes.
  Use when writing git commits or needing help with commit message format.
allowed-tools: Bash(git diff:*), Bash(git status:*)
---

# Commit Message Generator

Generate commit messages following [Conventional Commits](https://www.conventionalcommits.org/).

## Process

1. Run `git diff --staged` to review changes
2. Identify the type of change:
   - `feat`: New feature
   - `fix`: Bug fix
   - `docs`: Documentation
   - `style`: Formatting
   - `refactor`: Code restructuring
   - `test`: Tests
   - `chore`: Maintenance

3. Generate message with format:
   ```
   type(scope): subject (under 50 chars)

   Body explaining what and why (not how)

   Footer with issue references
   ```

## Example

For changes adding password reset:

```
feat(auth): add password reset functionality

- Implement email-based reset flow
- Add security token validation
- Include rate limiting for abuse prevention

Closes #123
```

## Best Practices

- Use present tense: "add feature" not "added feature"
- Be specific: "fix login error delay" not "fix bug"
- Explain intent in body, not implementation details
- Reference issues: "Closes #123" or "Refs #456"
```

### Step 3: Test the Skill

Ask your agent a question matching the skill's description:

```
Can you help me write a commit message for my staged changes?
```

The agent will request permission to use the skill, then:
1. Run `git diff --staged`
2. Analyze the changes
3. Generate a properly formatted message following conventional commits

### Best Practices Checklist

When creating skills, follow these guidelines:

- [ ] **Keep skills focused**: One capability per skill
- [ ] **Write specific descriptions**: Include keywords users naturally say
- [ ] **Use progressive disclosure**: Put essentials in SKILL.md, details in supporting files
- [ ] **Bundle validation scripts**: Complex logic → tested scripts, not token generation
- [ ] **Keep SKILL.md under 500 lines**: Use supporting files for comprehensive docs
- [ ] **Restrict tool access**: Use `allowed-tools` to limit capabilities
- [ ] **Test with team members**: Ensure descriptions trigger correctly

---

## Conclusion

Agent Skills represent a fundamental shift in how we extend [AI](/tags/2026/01/09/ai.html) capabilities. By providing a standardized, filesystem-based approach to packaging domain expertise, they transform general-purpose agents into specialized assistants that can navigate complex workflows with the same fluency as experienced team members.

The progressive disclosure architecture keeps interactions efficient while allowing unbounded complexity. The open standard ensures portability across [Claude-Code](/tags/2026/01/09/claude-code.html), [OpenCode](/tags/2026/01/09/opencode.html), [Vibe-CLI](/tags/2026/01/09/vibe-cli.html), and the growing ecosystem of [agentic-coding](/tags/2026/01/09/agentic-coding.html) tools.

For developers, skills offer a way to encode hard-won expertise into reusable, shareable assets. For organizations, they provide consistent standards without extensive training. For non-technical users, they lower the barrier to leveraging sophisticated AI capabilities.

The specification at [agentskills.io](https://agentskills.io) is intentionally minimal—you can read it in minutes. The barrier to creating your first skill is equally low: a directory, a markdown file, and a clear description of when it should activate.

Start with something small: a commit message generator, a code review checklist, or a documentation template. As you see skills activate contextually and improve your workflow, you'll find more opportunities to encode expertise—turning individual knowledge into organizational capability.

---

**Further Reading:**
- [Agent Skills Specification](https://agentskills.io/specification) - Official standard documentation
- [Anthropic Engineering Blog](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) - Deep dive on the design
- [OpenCode Documentation](https://opencode.ai/docs/skills/) - Open source implementation
- [Simon Willison's Analysis](https://simonwillison.net/2025/Dec/19/agent-skills/) - Community perspective

*Images from [Unsplash](https://unsplash.com) under the Unsplash License.*

---

## Related Tags

- [agentic-coding](/tags/2026/01/09/agentic-coding.html)
- [AI](/tags/2026/01/09/ai.html)
- [Claude-Code](/tags/2026/01/09/claude-code.html)
- [CLI-tools](/tags/2026/01/09/cli-tools.html)
- [democratization](/tags/2026/01/09/democratization.html)
- [IDE](/tags/2026/01/09/ide.html)
- [open-source](/tags/2026/01/09/open-source.html)
- [OpenCode](/tags/2026/01/09/opencode.html)
- [Vibe-CLI](/tags/2026/01/09/vibe-cli.html)
