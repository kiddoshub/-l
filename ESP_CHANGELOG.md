ESP.java changelog (Minecraft 26.3/Fabric context notes included)

1. getFadeAlpha (line ~372-379): fixed double-squaring. fadeDist is used unsquared; alpha = sqrt(dist)/fadeDist when dist <= fadeDist*fadeDist. Threshold check remains.
2. drawBoundingBox (line ~233-258): early-return if getColor returns null to prevent inheriting previous entity colors. Colors only applied after null check.
3. getColor (line ~354-370): added overload getColor(Entity, Color out) writing into caller-supplied Color; existing signature preserved as wrapper using shared baseColor.
4. 2D mode (line ~262-317): pos1 initialized to +∞, pos2 to -∞; skip drawing if pos1.x > pos2.x. checkCorner now updates shared pos1/pos2.
5. Sorting (line ~217-261, ~262-317): snapshot entity list once per frame into ArrayList, sort far-to-near by squared distance to camera before iterating (3D and 2D).
6. Occlusion (new): added occlusion/strict-occlusion settings. New EntityOcclusion.java samples 8 corners + center, raycasts from camera eye; visibleFraction = visible/total. Skip if fraction <= 0.05, multiply alpha by fraction. strict-occlusion uses COLLIDER shape and treats non-full-cube/non-solid collision shapes as occluders.
7. Validation delay (new): validation-delay-ms setting + ConcurrentHashMap<UUID,Long> firstSeen; skip entities seen < delay. Cleared on activate/deactivate/world tick (onTick clears if mc.world null).
8. Snapshot once per frame: done in both onRender3D and onRender2D before iteration.
9. Item/projectile color split (new): itemsColor and projectilesColor settings added; getEntityTypeColor routes ItemEntity -> itemsColor, ProjectileEntity/EndCrystalEntity -> projectilesColor.

26.3 porting notes:
- Render events still use Meteor's Render3DEvent/Render2DEvent in this codebase; to move to Fabric WorldRenderEvents.AFTER_ENTITIES and post-GUI 2D stage, replace @EventHandler methods with Fabric event registrations (flagged for verification).
- Renderer2D.COLOR batching compatibility with renderpearl + Sodium/Iris should be verified (CheatUtils pattern reference); no changes made here.
- NametagUtils.to2D projection matrices need re-validation after SDL3/window refactor (unchanged call sites).
- Any new keybind must use SDL scancodes (no new keybinds added).
- No outbound packets/chunk requests added; only reads client data.

New files: src/main/java/meteordevelopment/meteorclient/systems/modules/render/EntityOcclusion.java
Modified: src/main/java/meteordevelopment/meteorclient/systems/modules/render/ESP.java
