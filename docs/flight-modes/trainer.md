# Trainer Mode

Trainer Mode provides an assisted flight envelope suited to learning --
limiting attitude/rate extremes to make an aircraft harder to lose control
of, similar in spirit to a self-leveling "beginner mode" on other platforms.

Unlike Angle/Horizon-style self-leveling, Trainer Mode does *not* actively
level the aircraft -- it flies exactly like normal acro/manual flight right
up until the pitch/roll angle would exceed **Limit** (default 20°), at
which point it tilts back toward that limit rather than letting the
aircraft roll/pitch further. **Gain** sets how aggressively it corrects
back once the limit is exceeded -- high enough to reliably catch a
beginner's overcorrection, without fighting normal flying so hard it feels
like it's flying itself.
