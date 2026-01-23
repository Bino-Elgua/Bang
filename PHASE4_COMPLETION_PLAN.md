# Phase 4: Ralph Loop - Completion Plan

**Status:** 60% COMPLETE
**Timeline:** 1-2 weeks
**Estimated Effort:** 8-12 hours

---

## CURRENT STATE

### ✅ What's Working
- `services/ralphLoop.ts` (328 lines) — Core logic complete
  - `runRalphLoop()` — Main iteration loop
  - `parsePRDItems()` — Parses user input into PRD items
  - `loadCheckpoint()` — Resume from checkpoint
  - `exportCheckpoint()` / `importCheckpoint()` — Serialization
  
- `components/RalphLoopPanel.tsx` — UI component complete
  - PRD editor modal
  - Progress bar
  - Checkpoint history display
  
- `App.tsx` — Basic wiring done
  - Ralph loop toggle button
  - State variables: `ralphEnabled`, `prdItems`, `ralphIteration`, etc.
  - `runRalphLoopHandler()` function
  - `handleLoadRalphCheckpoint()` & `handleExportRalphCheckpoints()`

- Build passes (npm run build ✅)

### ❌ What Needs Completion

#### Issue 1: PRD Item Completion Detection (Critical)
**Problem:** Lines 179-189 in ralphLoop.ts use naive heuristic matching
```typescript
const completedIds = new Set<string>();
if (result.initialTeam) {
  result.initialTeam.forEach(agent => {
    remaining.slice(0, 3).forEach(item => {
      if (agent.description.toLowerCase().includes(item.description.slice(0, 20).toLowerCase())) {
        completedIds.add(item.id); // TOO SIMPLE!
      }
    });
  });
}
```

**Solution:** Implement semantic matching via embeddings or LLM classification

#### Issue 2: Missing RalphLoopPanel Integration
**Problem:** Panel exists but not wired into layout
**Solution:** Add to setup tab UI (2-3 hours)

#### Issue 3: Checkpoint Persistence
**Problem:** Checkpoints only in memory, lost on refresh
**Solution:** Wire localStorage save/load (1-2 hours)

#### Issue 4: Context Token Tracking
**Problem:** Simulated token counts (lines 251-253)
**Solution:** Track actual API token usage (2-3 hours)

#### Issue 5: Error Handling & Resume Logic
**Problem:** No retry mechanism for failed iterations
**Solution:** Add error recovery (2-3 hours)

---

## IMPLEMENTATION ROADMAP

### PHASE 4A: Smart PRD Item Completion (3-4 hours)

#### Step 1: Improve Completion Detection
Replace simple string matching with semantic analysis:

```typescript
// In ralphLoop.ts, replace lines 179-189

async function detectCompletedItems(
  agents: Agent[],
  remainingItems: PRDItem[],
  orchestratorConfig: IntelligenceConfig
): Promise<string[]> {
  const completedIds: string[] = [];
  
  // For each agent's output, use LLM to classify against PRD items
  for (const agent of agents) {
    const prompt = `
    Based on this agent output, identify which PRD items it completed:
    
    OUTPUT: ${agent.description}
    
    PRD ITEMS:
    ${remainingItems.map(p => `- [${p.id}] ${p.description}`).join('\n')}
    
    Return JSON: { "completedItemIds": ["prd-1", "prd-2"] }
    `;
    
    // Call LLM to get classification
    const response = await performAgentTask(prompt, agent, orchestratorConfig);
    const match = response.match(/\{[\s\S]*\}/);
    if (match) {
      const result = JSON.parse(match[0]);
      completedIds.push(...result.completedItemIds);
    }
  }
  
  return Array.from(new Set(completedIds));
}
```

#### Step 2: Add Priority-Based Item Ordering
Order items by priority so high-priority items complete first:

```typescript
const sortedRemaining = remaining
  .sort((a, b) => {
    const priorityMap = { high: 0, medium: 1, low: 2 };
    return priorityMap[a.priority] - priorityMap[b.priority];
  })
  .slice(0, 5); // Process top 5 items per iteration
```

#### Step 3: Update Progress Messaging
Show which specific items were completed each iteration:

```typescript
const completedThisRound = newlyCompleted.map(id => 
  remaining.find(p => p.id === id)?.description
).filter(Boolean);

onProgress(
  `Ralph Iteration ${iteration}: Completed items: ${completedThisRound.join(', ')}`,
  completionRate
);
```

---

### PHASE 4B: UI Integration (2-3 hours)

#### Step 1: Add Ralph Loop Panel to Setup Tab

In `App.tsx` line 733, add RalphLoopPanel to the setup grid:

```typescript
{/* Ralph Loop Section - Bottom */}
<div className="col-span-12 rounded-lg border bg-black/30 overflow-hidden">
  <RalphLoopPanel
    prdItems={prdItems}
    isRunning={isRalphRunning}
    currentIteration={ralphIteration}
    maxIterations={ralphMaxIterations}
    completionRate={ralphCompletionRate}
    checkpoints={ralphCheckpoints}
    onStartRalphLoop={runRalphLoopHandler}
    onLoadCheckpoint={handleLoadRalphCheckpoint}
    onExportCheckpoints={handleExportRalphCheckpoints}
  />
</div>
```

#### Step 2: Add Ralph Loop Monitor Widget
Create mini status widget in header:

```typescript
// In top header bar
{ralphEnabled && (
  <div className="flex items-center space-x-2 text-[8px] bg-green-600/10 px-2 py-1 rounded border border-green-500/30">
    <i className="fa-solid fa-refresh-cw animate-spin text-green-400"></i>
    <span>Ralph Loop: {(ralphCompletionRate * 100).toFixed(0)}%</span>
    <span className="opacity-60">{ralphIteration}/{ralphMaxIterations}</span>
  </div>
)}
```

---

### PHASE 4C: Checkpoint Persistence (2-3 hours)

#### Step 1: Add localStorage Hooks

```typescript
// In App.tsx useEffect section
useEffect(() => {
  // Save checkpoints to localStorage
  if (ralphCheckpoints.length > 0) {
    localStorage.setItem('ralph_checkpoints', JSON.stringify(
      ralphCheckpoints.map(cp => ({
        iteration: cp.iteration,
        timestamp: cp.timestamp.toISOString(),
        completedCount: cp.completedItems.length,
        remainingCount: cp.remainingItems.length,
        completionRate: cp.completionRate
      }))
    ));
  }
}, [ralphCheckpoints]);

// Load checkpoints on mount
useEffect(() => {
  const saved = localStorage.getItem('ralph_checkpoints');
  if (saved) {
    try {
      const checkpoints = JSON.parse(saved);
      addLog(`Restored ${checkpoints.length} previous Ralph Loop checkpoints`);
    } catch (e) {
      console.error('Failed to load checkpoints', e);
    }
  }
}, []);
```

#### Step 2: Add Checkpoint Recovery Button
In RalphLoopPanel, add recovery UI:

```typescript
const previousCheckpoints = localStorage.getItem('ralph_checkpoints');
{previousCheckpoints && (
  <button 
    onClick={() => {
      const checkpoints = JSON.parse(previousCheckpoints);
      onLoadCheckpoint(checkpoints[checkpoints.length - 1]);
    }}
    className="w-full px-3 py-1 text-[8px] font-bold uppercase rounded bg-orange-600/20 border border-orange-500/50"
  >
    🔄 Recover Last Session
  </button>
)}
```

---

### PHASE 4D: Real Token Tracking (2-3 hours)

#### Step 1: Update Token Counting
Replace simulated token counts (line 252 in ralphLoop.ts):

```typescript
// Instead of: totalTokensUsed += 15000;
// Use actual token counting from API response

if (result.tokenUsage) {
  totalTokensUsed += result.tokenUsage.input + result.tokenUsage.output;
  totalCostUSD += result.tokenUsage.cost;
} else {
  // Fallback estimation
  totalTokensUsed += estimateTokenCount(remainingStr + refreshPrompt);
}
```

#### Step 2: Wire Cost Calculator
Integrate with Phase 1 cost calculator:

```typescript
import { calculateCostForTokens } from './services/costCalculator';

const iterationCost = calculateCostForTokens(
  totalTokensUsed,
  project.orchestratorConfig.model
);
```

---

### PHASE 4E: Error Handling & Resume (2-3 hours)

#### Step 1: Add Error Recovery
In `runRalphLoop()`, add try-catch with checkpoint saving:

```typescript
try {
  const result = await orchestrateTeam(refreshPrompt, currentPRD.slice(0, 5), aiConfig);
  // ... process result
} catch (error) {
  // Save checkpoint before failing
  const failureCheckpoint: RalphCheckpoint = {
    iteration,
    timestamp: new Date(),
    completedItems: currentPRD.filter(p => p.completed),
    remainingItems: currentPRD.filter(p => !p.completed),
    completionRate: currentPRD.filter(p => p.completed).length / currentPRD.length,
    outputs: allOutputs,
    agents: [],
    errors: [error instanceof Error ? error.message : String(error)]
  };
  
  checkpoints.push(failureCheckpoint);
  throw new Error(`Iteration ${iteration} failed. Checkpoint saved for recovery.`);
}
```

#### Step 2: Add Retry Logic
Allow resuming from last successful checkpoint:

```typescript
const handleResumeFromFailure = async (checkpoint: RalphCheckpoint) => {
  setPrdItems([...checkpoint.completedItems, ...checkpoint.remainingItems]);
  setRalphIteration(checkpoint.iteration);
  addLog(`Resumed from iteration ${checkpoint.iteration}. Will retry from here.`);
  
  // Trigger another run starting at checkpoint
  await runRalphLoopHandler(checkpoint.remainingItems);
};
```

---

## TESTING CHECKLIST

### Unit Tests
- [ ] `parsePRDItems()` handles numbered, bulleted, and markdown lists
- [ ] `detectCompletedItems()` correctly identifies completed PRD items
- [ ] `runRalphLoop()` reaches 95% completion on 5-10 item PRD
- [ ] Checkpoints serialize/deserialize correctly
- [ ] Token counting matches expected ranges

### Integration Tests
- [ ] Ralph Loop button toggle works
- [ ] PRD input → parsing → first iteration works
- [ ] Progress bar updates in real-time
- [ ] Checkpoints save and load from localStorage
- [ ] Resume from checkpoint continues correctly
- [ ] Export checkpoints downloads valid JSON

### E2E Tests
- [ ] Complete 5-item PRD in Ralph Loop (5-10 iterations)
- [ ] Refresh page, resume from last checkpoint
- [ ] Export, share, re-import checkpoints
- [ ] Verify token counts and costs accurate

---

## SUCCESS CRITERIA

| Criterion | Target |
|-----------|--------|
| **Completion Rate** | 100% PRD items marked complete by iteration 5 |
| **Detection Accuracy** | 90%+ correct item completion identification |
| **Token Accuracy** | ±10% vs. actual API usage |
| **Checkpoint Recovery** | 100% success rate |
| **Performance** | <5 sec between iterations |
| **UI Responsiveness** | No freezes during iteration |
| **Error Handling** | Graceful recovery with checkpoint save |

---

## DELIVERABLES

**By End of Phase 4:**

```
SwarmIDE2-Phase4-Complete/
├── ✅ services/ralphLoop.ts (updated with smart completion detection)
├── ✅ components/RalphLoopPanel.tsx (integrated into layout)
├── ✅ App.tsx (checkpoint persistence wired)
├── ✅ Checkpoint recovery from localStorage
├── ✅ Real token tracking + cost integration
├── ✅ Error recovery with auto-checkpoint
├── ✅ All 8 test scenarios passing
└── ✅ Ready for Phase 5 (Advanced Features)
```

---

## NEXT IMMEDIATE STEPS

### DAY 1 (Today - 4 hours)
1. Implement `detectCompletedItems()` function with LLM classification
2. Update progress messaging in `runRalphLoop()`
3. Add priority-based item sorting

### DAY 2 (4 hours)
1. Integrate RalphLoopPanel into setup tab layout
2. Add header status widget
3. Test UI responsiveness

### DAY 3 (3-4 hours)
1. Add localStorage checkpoint persistence
2. Implement recovery button
3. Test checkpoint save/load cycle

### DAY 4 (2 hours)
1. Wire real token tracking
2. Integrate cost calculator
3. Update progress logging

### DAY 5 (2-3 hours)
1. Add error recovery logic
2. Test failure scenarios
3. Complete documentation

---

**Target Completion Date:** Feb 9, 2026

**Status:** 🟡 Ready to start Phase 4A (Smart PRD Detection)
