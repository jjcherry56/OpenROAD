# OpenROAD Build Instructions (macOS with Homebrew)

## Fix for Abseil Link Error and ABC cmake_info

### Root Cause
1. **Abseil version mismatch**: Using abseil 20250814 while `/opt/homebrew/include` points to 20260107 caused `lts_20260107` undefined symbol errors.
2. **ABC Makefile overwritten**: Running `cmake ..` from inside `build/` caused CMake to overwrite `third-party/abc/Makefile` with a generated file that lacks the `cmake_info` target.

### Solution

**Always use `cmake -S . -B build`** (never `cd build && cmake ..`). This keeps the build out-of-source and preserves the ABC Makefile.

### Build Steps

# 0. apply docs/BuildMacos.patch or use macos branch

```bash
# 1. Restore ABC Makefile if it was overwritten
git -C third-party/abc checkout Makefile

# 2. Configure with Homebrew's abseil 20260107 (matches OpenROAD and /opt/homebrew)
cmake -S . -B build -Dabsl_ROOT=/opt/homebrew/Cellar/abseil/20260107.1

# 3. Build
cd build && make -j$(sysctl -n hw.ncpu)
```

### Why abseil 20260107.1?
- OpenROAD's DependencyInstaller expects abseil 20260107
- Homebrew's default (`/opt/homebrew/include/absl`) points to 20260107.1
- Using 20250814 caused header/library mismatch: code compiled with 20260107 headers but linked 20250814 libs

### If ABC Makefile Gets Overwritten Again
Run: `git -C third-party/abc checkout Makefile`

Then reconfigure with step 2 above.
