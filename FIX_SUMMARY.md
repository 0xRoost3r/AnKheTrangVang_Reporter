# DCA Calculator Fix - Cumulative Rounding Logic

## Problem Identified
The JavaScript DCA calculator was producing different results than the MT5 code because it used a **direct calculation** approach instead of **cumulative rounding**.

### Original (Incorrect) Logic:
```javascript
const slotRaw = initSlot * Math.pow(multi, lineLevel);
const slot = Math.round(slotRaw * 100) / 100;
```

This calculates: `0.02 * 1.5^level` for each level independently.

### MT5 Actual Logic (Cumulative Rounding):
```
Level 1: 0.02 (initial)
Level 2: round(0.02 × 1.5) = 0.03
Level 3: round(0.03 × 1.5) = 0.05
Level 4: round(0.05 × 1.5) = 0.08
... and so on
```

Each level uses the **PREVIOUS rounded lot**, not the original calculation.

## Test Case: Init Lot 0.02, Multiplier 1.5, Max DCA 12

### MT5 Results (Actual):
```
Level 1:  0.02
Level 2:  0.03
Level 3:  0.05
Level 4:  0.08
Level 5:  0.12
Level 6:  0.18
Level 7:  0.27
Level 8:  0.41
Level 9:  0.62
Level 10: 0.93
Level 11: 1.40
Level 12: 2.10
```

### JavaScript Results (After Fix):
```
Level 1:  0.02 ✓
Level 2:  0.03 ✓
Level 3:  0.05 ✓
Level 4:  0.08 ✓
Level 5:  0.12 ✓
Level 6:  0.18 ✓
Level 7:  0.27 ✓
Level 8:  0.41 ✓
Level 9:  0.62 ✓
Level 10: 0.93 ✓
Level 11: 1.40 ✓
Level 12: 2.10 ✓
```

## Changes Made to index.html

### 1. Main Loop (Lines 237-258)
**Before:**
```javascript
const slotRaw = initSlot * Math.pow(multi, lineLevel);
const slot = Math.round(slotRaw * 100) / 100;
```

**After:**
```javascript
let currentLot = initSlot; // Track cumulative rounding

for (let i = 0; i < maxLine; i++) {
    let slot;
    if (i === 0) {
        slot = initSlot;
    } else {
        slot = Math.round(currentLot * multi * 100) / 100;
    }
    currentLot = slot; // Update for next iteration
}
```

### 2. Summary Calculation (Lines 295-308)
**Before:**
```javascript
const slot = Math.round(slotRaw * 100) / 100;
```

**After:**
```javascript
let currentLotSummary = initSlot;
for (let i = 0; i < maxLine; i++) {
    let slot;
    if (i === 0) {
        slot = initSlot;
    } else {
        slot = Math.round(currentLotSummary * multi * 100) / 100;
    }
    currentLotSummary = slot;
}
```

### 3. Risk Warning Function (Lines 401-406)
**Before:**
```javascript
const maxSlot = Math.round((initSlot * Math.pow(multi, maxLine - 1)) * 100) / 100;
```

**After:**
```javascript
let maxSlot = initSlot;
for (let i = 1; i < maxLine; i++) {
    maxSlot = Math.round(maxSlot * multi * 100) / 100;
}
```

## Result
✅ **JavaScript now produces IDENTICAL results to MT5!**

All calculations now match the MT5 behavior exactly, including:
- Individual lot sizes per level
- Cumulative slots
- P&L calculations
- Breakeven prices
- All derived metrics

