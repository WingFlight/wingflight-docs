# Curves

The Curves tab holds two independent pools of up to 8 reusable curves each.
You edit a curve here, then assign it *by number* wherever it's used -- a
[Mixer](mixer.md) rule, or a [Profiles](profiles.md#master-gain) axis --
rather than defining the shape inline there. That reuse is the point:
editing curve 3 here updates every rule or axis currently assigned to curve
3, so one curve can shape several things identically at once.

## Mixer Curves

**X**: the mixer rule's input value. **Y**: the value substituted in its
place once weighted onto the output. Both share the rule's own -1000..1000
scale. The default is a straight diagonal -- input passes straight through
unchanged -- so there's nothing to assign until you deliberately bend it:
more resolution near center, a dead zone, an asymmetric response, a
non-linear throttle-to-motor curve, and so on. Assign one to a rule from
the [Mixer](mixer.md) tab.

## Gain Curves

**X**: stick deflection, 0% (centered) to 100% (full throw) -- the same
curve applies in both directions, it doesn't distinguish left from right.
**Y**: a gain multiplier, 0-500%, where 100% means no change. The gain
actually applied in flight is Master Gain (set per-axis on
[Profiles](profiles.md#master-gain)) multiplied by this curve's value at
the current stick position -- so a flat line at 100% has no effect at all.
Drag a point below 100 to taper gain out as the stick moves that way (a
softer response out toward the ends of the stick), or above 100 to sharpen
it there.

## Editing a curve

- Drag a point to move it, click empty plot space to add one, or
  right-click a point to remove it -- or use the point table alongside the
  plot for exact numeric values. Mixer curves allow up to 9 points; Gain
  curves up to 6.
- The first and last point are pinned to the curve's domain extremes
  (-1000/1000 for Mixer, 0%/100% for Gain) and can't be deleted or dragged
  in X, only Y -- they define where the curve starts and ends, so at least
  two points always exist and always span the full range.
- **Reset Curve** returns the currently selected slot to a neutral straight
  line (no reshaping, no gain change).
