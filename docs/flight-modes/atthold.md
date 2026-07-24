# Attitude Hold

Attitude Hold (ATTHOLD) holds the aircraft's commanded attitude when you
release the sticks, rather than the aircraft continuing to fly whatever
attitude it was last commanded (typical for a fixed-wing aircraft with no
self-leveling). This gives a self-leveling-like behavior on demand,
independent of full autopilot features.

Unlike [Auto Hover](auto-hover.md), Attitude Hold doesn't bound stick
authority to any particular orientation -- it freezes and holds whatever
attitude the aircraft happens to be in the instant every stick returns
inside **Deadband** (percent stick deflection, on any of roll/pitch/yaw)
at once, at any orientation, not just vertical or level. Above the
deadband on any stick, you get full unmodified authority and the mode
continuously re-captures its target, so the next freeze is seamless
whenever the sticks settle again. **Gain** sets how aggressively it
corrects back to that frozen attitude, and **Max Rate** caps how fast it's
allowed to rotate the aircraft while doing so.
