# Sketcher reference-image visibility — investigation and acceptance plan

## Problem

When editing a room-layout sketch on top of a scanned apartment floor plan, the reference image appears darker than outside sketch edit mode. Fine details become difficult to trace while sketch lines and constraints must remain legible.

**Status:** investigation only; the rendering cause is not yet confirmed, and no runtime behavior has been changed.

## Existing controls to test first

1. In **Edit → Preferences → Sketcher → General**, test **Disable shading in edit mode** both enabled and disabled. This is an existing option; it does not specifically promise to control image brightness.
2. Check the sketch's **Section View** setting and the **View section** command, since the edit lifecycle calls `Show.TempoVis.sketchClipPlane(...)`.
3. For an ImagePlane used to trace a sketch, put the image slightly behind the sketch plane (for an XY sketch, try a small negative Z offset such as -0.1 mm). Be mindful of attachment orientation.
4. Compare image brightness at identical camera orientation and zoom outside and inside Sketcher, and check whether an overlay, occlusion, or image transparency is responsible.

## Source map (branch sketcher-improvements)

- `src/Mod/Sketcher/Gui/ViewProviderSketch.cpp`: `setEdit()` invokes visibility automation, shows attachment support and linked geometry, configures a sketch clip plane, and hides the edited sketch; `unsetEdit()` restores previous visibility through `TempoVis`.
- `src/Mod/Sketcher/Gui/ViewProviderSketch.cpp`: `setEditViewer()` adjusts camera orientation and activates viewer editing.
- `src/Gui/View3DInventorViewer.cpp`: `setEditing(bool)` changes edit state and cursor; no direct image dimming is evident in that method.
- `src/Mod/Sketcher/Gui/SketcherSettings.cpp`: Sketcher display and appearance preference loading/saving.
- `src/Mod/Sketcher/Gui/EditModeCoinManager.cpp`: Sketcher axes and geometry visual properties; axis transparency is separate from reference-image brightness.

The `SoSketchFaces` transparency value is associated with sketch-face rendering, **not evidence** of reference-image opacity control.

## Proposed behavior after root-cause confirmation

- Keep reference ImagePlane appearance consistent when entering or leaving edit mode, unless the user explicitly selects a dimming effect.
- If an intentional edit overlay causes the reduction in brightness, expose a configurable **reference-image dimming amount** with a backward-compatible default, including a 0% (off) option.
- Do not change Sketcher geometry, constraint, grid, solver, other-object visibility, or section-clipping semantics.
- Restore all temporary view state on normal exit and canceled edit.

## Acceptance checks

- ImagePlane on XY with a sketch slightly in front: inspect image contrast before, during, and after editing.
- Repeat with a scanned floor plan containing thin dark lines, and with dark/light application themes.
- Test a regular bitmap ImagePlane versus a textured model face to distinguish image-specific behavior.
- Verify constraints, selection/preselection, sketch faces, external geometry, and section view.
- Check entering/exiting edit repeatedly and canceling an edit.
- Compare with and without **Disable shading in edit mode** and with section view toggled.

## Implementation gate

Before changing C++ rendering paths, reproduce using a minimal FCStd project and record the FreeCAD version, image object type, view settings, screenshot, and exact reproduction steps. This prevents inadvertently weakening visibility or making shaded solids incorrect.
