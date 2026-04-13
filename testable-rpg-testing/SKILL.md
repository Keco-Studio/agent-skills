---
name: testable-rpg-testing
description: Testable RPG game engine testing workflow. Use this skill when developing features for the testable-rpg project to ensure tests are run before UI changes. This skill enforces a test-first workflow using the GameTestAPI harness, Vitest unit tests, and Playwright integration tests.
license: MIT
metadata:
  author: project-carp
  version: "1.0.0"
---

# Testable RPG Testing Workflow

A deterministic, automation-first RPG game engine with first-class testability. **Before writing any UI code, run the relevant tests to establish a baseline and verify your changes.**

## Core Testing Philosophy

1. **Test the engine, not the DOM** - Game logic lives in pure TypeScript systems (`src/engine/`) with zero UI dependencies
2. **Use the GameTestAPI** - The `window.__game` harness provides full read/control of game state for declarative testing
3. **Contract-driven development** - Each system has a `contracts/*.contract.md` file listing behavioral guarantees
4. **Quality layer validation** - Game feel and player experience validated through `fun/*.test.ts` tests

## Test Commands

```bash
cd testable-rpg

# Unit tests (Vitest) - PRIMARY for logic validation
npm test              # Run once
npm run test:watch     # Watch mode

# Playwright tests
npm run test:pw           # All Playwright tests
npm run test:visual        # Visual regression tests
npm run test:integration   # Integration tests
npm run test:e2e           # End-to-end gameplay tests

# AI Explorer (headless bug detection)
npm run explorer -- http://localhost:5173 42 200 reports/explorer.json
```

## GameTestAPI (`window.__game`)

Installed automatically in non-production mode. Provides full game state control:

### Read Methods
```typescript
window.__game.getScene()        // Current scene name
window.__game.getPlayer()        // Player stats snapshot
window.__game.getInventory()     // Array of {itemId, quantity}
window.__game.getQuestState()    // Quest state by ID
window.__game.getQuestLog()      // All quest states
window.__game.getDialogState()   // Active dialog info
window.__game.getMapPosition()   // {map, x, y}
window.__game.getBattleState()   // Active battle info
window.__game.getFlags()         // Story flags
```

### Control Methods
```typescript
window.__game.teleport(x, y, map?)           // Move player
window.__game.setPlayerStat(stat, value)     // Modify stats
window.__game.setHP(value)                   // Set player HP
window.__game.addItem(itemId, quantity?)     // Add to inventory
window.__game.removeItem(itemId, quantity?)  // Remove from inventory
window.__game.triggerDialog(npcId)           // Start NPC dialog
window.__game.choose(index)                  // Select dialog choice
window.__game.skipDialog()                   // Advance dialog
window.__game.startBattle(enemyIds)          // Initiate combat
window.__game.endBattle(outcome)             // End battle ('win'|'lose'|'flee')
window.__game.stepFrames(frames)             // Advance battle simulation
window.__game.changeScene(sceneName)         // Scene transition
window.__game.setFlag(key, value)            // Set story flag
window.__game.activateQuest(id)              // Start quest
window.__game.completeQuest(id)              // Complete quest
window.__game.saveGame(slot)                // Save to slot (0-2)
window.__game.loadGame(slot)                // Load from slot
window.__game.setSeed(seed)                  // Control RNG
```

### Scenario Runner
For declarative test scenarios:
```typescript
window.__game.runScenario({
  steps: [
    { action: 'setSeed', value: 42 },
    { action: 'addItem', itemId: 'sword' },
    { action: 'teleport', x: 50, y: 50 },
    { assert: { path: 'player.x', equals: 50 } },
    { action: 'startBattle', enemyIds: ['slime'] },
    { action: 'endBattle', outcome: 'win' },
    { assert: { path: 'battle.outcome', equals: 'win' } }
  ]
})
```

## Test File Locations

| System | Unit Tests | Contract |
|--------|-----------|----------|
| Combat | `src/engine/combat/__tests__/CombatSystem.test.ts` | `contracts/combat.contract.md` |
| Inventory | `src/engine/inventory/__tests__/InventorySystem.test.ts` | `contracts/inventory.contract.md` |
| Quest | `src/engine/quest/__tests__/QuestSystem.test.ts` | `contracts/quest.contract.md` |
| Dialog | `src/engine/dialog/__tests__/DialogSystem.test.ts` | `contracts/dialog.contract.md` |
| Save | `src/engine/save/__tests__/SaveSystem.test.ts` | `contracts/save.contract.md` |
| Storyline | `src/engine/storyline/__tests__/StorylineEngine.test.ts` | `contracts/storyline.contract.md` |
| SeededRNG | `src/engine/rng/__tests__/SeededRNG.test.ts` | - |
| Runtime | `src/runtime/__tests__/GameRuntime.test.ts` | - |
| Quality | `src/runtime/__tests__/fun-*.test.ts` | `fun/*.md` |

## Workflow: Test Before UI

### Step 1: Identify Affected Systems
When given a UI change, identify which engine systems are affected:
- Combat UI changes → run `CombatSystem.test.ts`
- Inventory UI changes → run `InventorySystem.test.ts`
- Dialog UI changes → run `DialogSystem.test.ts`

### Step 2: Run Baseline Tests
```bash
cd testable-rpg
npm test -- --run src/engine/<system>/__tests__/<System>.test.ts
```

### Step 3: Verify Contract Compliance
Check `contracts/<system>.contract.md` for behavioral guarantees. Ensure your UI changes don't violate any P1 (critical) invariants.

### Step 4: Write/Update Tests for UI Logic
If the UI requires new game behavior:
1. Write the test first using `GameTestAPI`
2. Verify it fails
3. Implement the game logic
4. Verify test passes
5. Only then implement the UI

### Step 5: Run Full Test Suite
```bash
npm test
npm run test:pw
```

## Quality Framework

Before modifying gameplay code, consult `fun/README.md` for 5 quality layers:

| Layer | Focus | Test File |
|-------|-------|-----------|
| Visual Clarity | Tile colors, NPC contrast, text readability | `fun-visual-clarity.test.ts` |
| World Feel | Map connectivity, zone variety, NPC placement | `fun-world-feel.test.ts` |
| Moment-to-Moment | Feedback, transitions, input response | `fun-moment-to-moment.test.ts` |
| Session Arc | Reward pacing, progression, anti-grind | `fun-session-arc.test.ts` |
| Story & Agency | Choices, consequences, faction divergence | `fun-story-agency.test.ts` |

## Browser Demo Pages

- `/` - Main playable canvas game
- `/playtest.html` - Runtime playtest dashboard with live controls
- `/game-storyline-demo.html` - Long storyline playtest for canvas game
- `/storyline-demo.html` - Runtime storyline validation

## Key Constraints

1. **Engine has no UI dependencies** - All game logic in `src/engine/` is pure TypeScript
2. **Tests must be deterministic** - Use `setSeed()` for reproducible RNG
3. **Contract violations are blocking** - No code change may break a P1 contract without tests updated first
4. **New features require contracts + tests** - No exceptions
