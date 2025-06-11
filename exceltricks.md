# Excel: Keyboard-Only Formula Entry with F2

## Problem
When typing formulas like `=COUNT(` in Excel, cursor gets trapped in edit mode. Arrow keys won't navigate to select cell ranges, forcing mouse usage or manual range typing.

## Solution: F2 Toggle Technique

1. Start typing formula: `=COUNT(`
2. **Press F2** (toggles out of point mode)
3. Use arrow keys to navigate to desired cells
4. Use **Shift + Arrow keys** to select range
5. Press Enter to complete

## Key Insight
F2 toggles between two modes:
- **Point Mode**: Excel expects mouse clicks for cell selection
- **Edit Mode**: Keyboard navigation works normally

## Quick Tips
- Use **Ctrl + Shift + Arrow** for rapid range selection
- Combine with **Ctrl + G** for specific ranges
- Type ranges directly if known: `=COUNT(A1:A10)`

## Result
100% keyboard-driven formula entry without mouse dependency.
