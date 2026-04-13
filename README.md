# Agent Skills

Collection of reusable skills for Claude Code agents.

## Available Skills

### testable-rpg-testing

Game engine testing workflow for the testable-rpg project. Provides GameTestAPI harness, Vitest unit tests, and Playwright integration tests.

**When to use:** When developing features in the testable-rpg project, run tests before writing UI code.

---

## Installation

### Option 1: Manual Installation (Recommended)

Each developer manually creates the skill directory in their project:

```bash
# 1. Create the skills directory in your project root
mkdir -p .agents/skills/testable-rpg-testing

# 2. Download the SKILL.md file into that directory
#    Clone this repo or download just the file from:
#    https://github.com/Keco-Studio/agent-skills/blob/main/testable-rpg-testing/SKILL.md

# 3. Verify the file exists
ls .agents/skills/testable-rpg-testing/SKILL.md
```

### Option 2: Clone the Repository

```bash
# Clone this repo anywhere, then copy the skill folder to your project
git clone https://github.com/Keco-Studio/agent-skills.git
cp -r agent-skills/testable-rpg-testing /path/to/your/project/.agents/skills/
```

---

## Usage

After installation, you can invoke the skill in Claude Code:

```
/skill testable-rpg-testing
```

Or simply mention "testable-rpg" in your conversation and Claude will use the skill automatically.

---

## For testable-rpg Developers

This skill enforces a **test-first workflow**:

1. **Identify** which engine systems your change affects
2. **Run** `npm test` to establish a baseline
3. **Write/update** tests for any new game logic
4. **Implement** the game logic
5. **Verify** all tests pass
6. **Then** write UI code

See [SKILL.md](./testable-rpg-testing/SKILL.md) for full documentation.
