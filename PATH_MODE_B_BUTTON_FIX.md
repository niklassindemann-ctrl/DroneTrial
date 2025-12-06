# Path Mode B Button Fix

**Date:** December 2025  
**Issue:** In path mode, pressing B button to adjust ghost depth was unintentionally removing points from the route, causing the path line to disappear.

---

## Problem Description

When in path building mode:
- Pressing **B button** was supposed to move the ghost preview **closer** (depth adjustment)
- However, it was also calling `UndoLastPoint()` which removed the last point from the route
- This caused the cyan path line to disappear while the numbered badges (1-2-3-4) remained visible
- Points were not actually deleted from the scene, but removed from the route, breaking the path visualization

**Important:** This bug only occurred in **path mode**, not in normal point creation mode.

---

## Root Cause

In `PathModeController.cs`, the `HandlePathModeInput()` method was handling the B button to perform undo functionality:

```csharp
// Handle B button for undo
bool bButton = ReadButton(_rightHand, CommonUsages.secondaryButton);
if (EdgePressed(bButton, ref _bButtonPrev))
{
    _pathManager.UndoLastPoint();
    ProvideHapticFeedback(_hapticAmplitude * 0.5f, _hapticDuration);
}
```

Meanwhile, `RayDepthController` was also handling B button for depth adjustment. This created a conflict where pressing B:
1. Moved the ghost closer (from RayDepthController)
2. Removed the last point from route (from PathModeController)

---

## Solution

Remove the undo functionality from B button in path mode. B button should only adjust ghost depth, not remove points.

---

## Files to Modify

### File: `Assets/Scripts/Points/PathModeController.cs`

#### Change 1: Remove B Button Undo Handling

**Location:** `HandlePathModeInput()` method (around lines 109-127)

**OLD CODE:**
```csharp
private void HandlePathModeInput()
{
    // Handle trigger for adding points to route
    bool trigger = ReadButton(_rightHand, CommonUsages.triggerButton);
    if (EdgePressed(trigger, ref _triggerPrev))
    {
        HandleTriggerInput();
    }

    // Handle B button for undo
    bool bButton = ReadButton(_rightHand, CommonUsages.secondaryButton);
    if (EdgePressed(bButton, ref _bButtonPrev))
    {
        _pathManager.UndoLastPoint();
        ProvideHapticFeedback(_hapticAmplitude * 0.5f, _hapticDuration);
    }

    // A button is used for depth control in normal mode - not used in path mode
```

**NEW CODE:**
```csharp
private void HandlePathModeInput()
{
    // Handle trigger for adding points to route
    bool trigger = ReadButton(_rightHand, CommonUsages.triggerButton);
    if (EdgePressed(trigger, ref _triggerPrev))
    {
        HandleTriggerInput();
    }

    // B button is handled by RayDepthController for depth adjustment (move ghost closer)
    // A button is also handled by RayDepthController for depth adjustment (move ghost further)
    // Removed undo functionality - B button should only adjust ghost depth in path mode
```

#### Change 2: Remove Unused Variables

**Location:** Class field declarations (around lines 21-26)

**OLD CODE:**
```csharp
private InputDevice _rightHand;
private InputDevice _leftHand;
private bool _rightGripPrev;
private bool _bButtonPrev;
private bool _aButtonPrev;
private bool _triggerPrev;
```

**NEW CODE:**
```csharp
private InputDevice _rightHand;
private InputDevice _leftHand;
private bool _rightGripPrev;
private bool _triggerPrev;
```

---

## Expected Behavior After Fix

In **path mode**:
- **A button** = Move ghost further away (depth adjustment) ✅
- **B button** = Move ghost closer (depth adjustment) ✅ (NO LONGER removes points)
- **Trigger** = Add point to route ✅
- **Right Grip** = Toggle path mode ✅

The path line will remain visible when adjusting ghost depth with A/B buttons.

---

## Testing Checklist

After applying the fix, verify:
- [ ] B button in path mode moves ghost closer without removing points
- [ ] A button in path mode moves ghost further without removing points
- [ ] Path line (cyan) remains visible when pressing A/B buttons
- [ ] Numbered badges (1-2-3-4) remain visible
- [ ] Points are only removed when using left trigger to delete
- [ ] All functionality works in normal point creation mode (non-path mode)

---

## Notes

- This fix removes the undo functionality from B button in path mode
- If undo functionality is needed in the future, use a different button combination
- The `RayDepthController` already handles A/B buttons for depth adjustment in all modes
- No changes needed to `RayDepthController.cs` - it already correctly handles depth adjustment

---

## Quick Reference

**Button Controls in Path Mode (After Fix):**
- Right Trigger = Add point to route
- Right Grip = Toggle path mode on/off
- A Button = Move ghost further (depth +)
- B Button = Move ghost closer (depth -)
- Left Trigger = Delete point (handled by RayDepthController)

