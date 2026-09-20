# Code Guidelines and Testing

Guidance for firmware changes. For layout and formatting rules, see
[Coding Style](coding-style.md).

## General principles

1. Name everything well.
2. Strike a balance between simplicity and not repeating code.
3. Functions that start with `find` can return null. Functions that start
   with `get` should not.
4. Keep functions short. It makes them easier to test.
5. Don't be afraid of moving code to a new file. It reduces test
   dependencies.
6. Avoid noise words in names, like `data` or `info`. Think about what you are
   naming, and don't be afraid to rename anything.
7. Avoid comments that describe what the code is doing. The code should
   describe itself. Comments are useful for the big picture and for
   documenting what a variable holds.
8. Document a variable at its declaration. Don't copy the comment to the
   `extern` usage, since it will rot.
9. Seek advice from other developers.
10. Be professional. Humor, or criticism of existing code, in the code itself
    is not helpful to whoever has to change it next.
11. There is always more than one way to do something, and code is never
    final, but it does have to work.

## Changing flight-control code

Mixer, mode and PID changes are safety-critical. Before you change them, read
the [Flight Dynamics](https://github.com/WingFlight/wingflight-firmware/blob/master/docs/FlightDynamics.md)
notes in the firmware repository. They record the signal chain, sign
conventions and the design rationale, and list known defects that can look
intentional.

- Do not renumber the permanent mode (BOX) IDs. Keep MSP and CLI compatible
  where you can.
- Saved settings live in parameter groups. Follow the
  [parameter group rules](tech/parameter-groups.md), and bump the group version
  when you change its layout.

## Unit testing

Ideally, any new code has a test. The codebase is old and was not designed for
testing, so this can be hard. Focus on the smallest change that lets you add a
test.

Tests live in `src/test/unit` and use GoogleTest. They are written in C++,
linked with the firmware's C code, and built and run natively on your
development machine, so you need no board. To run them from the root of the
firmware repository:

```
make test
```

`make junittest` does the same. Each `*_unittest.cc` file becomes one
executable in `obj/test`. `make -k test` carries on with the next test after a
failure. Test reports are also written by the run.

Some test files are disabled by their `.cc.txt` extension, for example the
mixer, IMU and failsafe tests, and need reviving before they can run. Several
areas have no tests at all: the hold engine, Auto Hover, Attitude Hold, the
thrust-vector loop, leveling, airborne detection, servos and GPS navigation.
Tests for those are welcome.

## Pull requests

- Keep a pull request to one thing. It is easier to review and test.
- Fork the repository, create a branch for each change, and open the pull
  request against `master`. Never open a pull request from your own `master`.
- Don't rewrite history on a branch after you have opened the pull request.
- Firmware pull requests should build with `make unified`.
- If your change affects what a user sees, update the matching page on this
  site.
