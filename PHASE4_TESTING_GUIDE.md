# Phase 4: Ralph Loop Testing Guide

**Status:** Ready for testing
**Estimated Time:** 10-15 minutes per test scenario

---

## QUICK START (2 minutes)

```bash
cd /data/data/com.termux/files/home/SwarmIDE2
npm run dev
```

Navigate to `http://localhost:3000`

---

## TEST SCENARIO 1: Ralph Panel Visibility

**Objective:** Verify Ralph Loop panel is visible in setup tab

**Steps:**
1. App loads → default "setup" tab active ✓
2. Scroll down → see "Target Nodes" section ✓
3. Continue scrolling → see **"🔄 Ralph Loop: PRD-Driven Execution"** panel below Target Nodes ✓

**Expected:**
- Panel shows:
  - Progress bar (0% initially)
  - "✓ Completed" section (empty)
  - "⏳ Remaining" section (empty)
  - "+ Add PRD Items" button
  - "How Ralph Loop Works" info box

**Pass/Fail:** ______

---

## TEST SCENARIO 2: Manual PRD Entry

**Objective:** Test adding PRD items manually

**Steps:**
1. Click "+ Add PRD Items" button
2. Paste in textarea:
   ```
   1. Build REST API with Express
   2. Setup PostgreSQL database
   3. Create React authentication UI
   4. Deploy to Docker
   5. Write API documentation
   ```
3. Click "🚀 Start Ralph Loop" button

**Expected:**
- Button disappears → text area shows
- 5 items parsed correctly:
  - Category auto-detected (api, database, frontend, deployment, docs)
  - All marked as incomplete
  - Priority shows "medium"

**Pass/Fail:** ______

---

## TEST SCENARIO 3: First Iteration Starts

**Objective:** Verify first Ralph Loop iteration executes

**Setup:** Complete Test Scenario 2 first

**Steps:**
1. "🚀 Start Ralph Loop" button clicked
2. Watch progress bar increment from 0%
3. In terminal/logs, see:
   - `Ralph Iteration 1/5: Processing 5 items with fresh context...`
   - Agent orchestration begins

**Expected:**
- Progress bar moves (even slowly)
- Terminal shows iteration started
- Panel shows: "Iteration 1/5"

**Pass/Fail:** ______

---

## TEST SCENARIO 4: Checkpoint Creation

**Objective:** Verify checkpoint saved after iteration

**Setup:** Let Test Scenario 3 complete (iteration 1)

**Expected in Ralph Panel:**
- Section "📌 Checkpoints" appears
- Shows: `Iter 1: X%`
- Timestamp displayed
- Can click to select

**How to verify:**
- Look for "Checkpoints" section in panel
- Should show at least 1 checkpoint

**Pass/Fail:** ______

---

## TEST SCENARIO 5: Completion Detection

**Objective:** Verify items are marked as completed

**Setup:** Complete at least iteration 1 from Test Scenario 3

**Expected:**
- "✓ Completed" section shows items like:
  - "Build REST API with Express"
  - "Setup PostgreSQL database"
  - etc.
- "⏳ Remaining" shows what's left

**How to verify:**
- Check panel's two-column grid
- Green text = completed items
- Yellow text = remaining items

**Pass/Fail:** ______

---

## TEST SCENARIO 6: Checkpoint Persistence (localStorage)

**Objective:** Verify checkpoints survive page refresh

**Setup:** Complete Test Scenario 4 (at least 1 checkpoint created)

**Steps:**
1. Open browser DevTools (F12)
2. Go to "Application" → "Local Storage" → "localhost:3000"
3. Look for key: `ralph_checkpoints`
4. Should see JSON with checkpoint data:
   ```json
   [
     {
       "iteration": 1,
       "timestamp": "2026-01-23T...",
       "completedCount": 2,
       "remainingCount": 3,
       "completionRate": 0.4,
       "prdItems": [...]
     }
   ]
   ```

**Expected:**
- localStorage key exists
- Valid JSON
- Checkpoint data accurate

**Pass/Fail:** ______

---

## TEST SCENARIO 7: Page Refresh → Checkpoint Recovery

**Objective:** Verify checkpoints load after refresh

**Setup:** Complete Test Scenario 6 (checkpoints in localStorage)

**Steps:**
1. Press F5 to refresh page
2. App reloads
3. Check terminal logs → should see:
   `✓ Restored X Ralph Loop checkpoints from previous session`

**Expected:**
- Log message shows checkpoints restored
- Ralph panel displays checkpoints again

**Pass/Fail:** ______

---

## TEST SCENARIO 8: Export Checkpoints

**Objective:** Test JSON export of checkpoints

**Setup:** Complete at least iteration 2 (multiple checkpoints)

**Steps:**
1. Look for "💾 Export All Checkpoints" button in Ralph panel
2. Click it
3. JSON file downloads (e.g., `ralph-checkpoints-2026-01-23.json`)
4. Open file in text editor
5. Verify structure:
   ```json
   [
     { "iteration": 1, "timestamp": "...", "completedCount": 2, ... },
     { "iteration": 2, "timestamp": "...", "completedCount": 3, ... }
   ]
   ```

**Expected:**
- File downloads
- Valid JSON
- All iterations included

**Pass/Fail:** ______

---

## TEST SCENARIO 9: Full Completion

**Objective:** Run Ralph Loop until 95%+ complete

**Setup:** Start with fresh 5-item PRD

**Steps:**
1. Add 5 PRD items
2. Click "Ralph Loop"
3. Monitor progress bar → should reach ~95-100%
4. Watch iteration counter: should finish in 3-5 iterations
5. Terminal shows: `✓ Ralph Loop: ALL PRD ITEMS COMPLETE!`

**Expected:**
- Progress bar reaches 95%+ 
- All items move to "✓ Completed" section
- Zero items in "⏳ Remaining"
- Iteration stops (doesn't continue past 95%)

**Success Criteria:**
- Completion rate: ≥95%
- Iterations: ≤5
- Terminal message: ALL PRD ITEMS COMPLETE

**Pass/Fail:** ______

---

## TEST SCENARIO 10: Error Handling (Attempt)

**Objective:** Verify graceful handling if iteration fails

**Prerequisite:** Have no API key set (to force an error)

**Steps:**
1. Remove/blank GEMINI_API_KEY from .env
2. Start Ralph Loop with 3 items
3. Watch what happens

**Expected (Current State):**
- Error appears in terminal
- Iteration stops gracefully
- No crash/freeze
- Checkpoint still saved (even with error)

**Known:** Phase 4E will add better error recovery

**Pass/Fail:** ______

---

## SUMMARY SCORECARD

| Test | Scenario | Status |
|------|----------|--------|
| 1 | Panel Visibility | ☐ PASS ☐ FAIL |
| 2 | Manual PRD Entry | ☐ PASS ☐ FAIL |
| 3 | First Iteration | ☐ PASS ☐ FAIL |
| 4 | Checkpoint Creation | ☐ PASS ☐ FAIL |
| 5 | Completion Detection | ☐ PASS ☐ FAIL |
| 6 | localStorage Write | ☐ PASS ☐ FAIL |
| 7 | Checkpoint Recovery | ☐ PASS ☐ FAIL |
| 8 | Export Checkpoints | ☐ PASS ☐ FAIL |
| 9 | Full Completion | ☐ PASS ☐ FAIL |
| 10 | Error Handling | ☐ PASS ☐ FAIL |

**Total Passing:** ___/10

**Overall Status:** 
- ✅ READY FOR PHASE 4B (if 8/10+)
- 🟡 NEEDS FIXES (if 5-7/10)
- 🔴 CRITICAL ISSUES (if <5/10)

---

## DEBUGGING TIPS

### Ralph Panel Not Visible
- Check: Did you scroll all the way down in setup tab?
- Try: Refresh page (F5)
- Check browser console: `F12` → Console tab

### Ralph Loop Button Doesn't Work
- Check: Did you enter a mission prompt?
- Check: Is "Ralph" toggle enabled (green)?
- Try: Click different button

### Checkpoints Not Saving
- Check: localStorage enabled in browser settings
- Verify: F12 → Application → Local Storage → key exists
- Try: Export checkpoints → verify JSON valid

### Progress Bar Stuck at 0%
- Check: API key configured (.env)
- Check: Network tab in F12 → any failed requests?
- Try: Open browser console → any errors?

### Items Not Marked as Completed
- This is normal for custom/unusual PRD items
- Completion detection uses keyword matching
- Works best with standard tech terms (api, database, auth, etc.)

---

## NEXT PHASES AFTER TESTING

Once ≥8/10 tests pass:

1. **Phase 4B:** Real token tracking (2-3 hours)
2. **Phase 4C:** Cost integration (2 hours)
3. **Phase 4D:** Error recovery (2-3 hours)
4. **Phase 4E:** Documentation (1-2 hours)

**Estimated Full Completion:** Jan 26, 2026 ✅

---

## CONTACT / NOTES

**Build Command:** `npm run build`
**Dev Server:** `npm run dev` (localhost:3000)
**Env File:** `.env.local` (copy from `.env.local.example`)

Good luck! 🚀
