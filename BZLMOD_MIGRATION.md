# Bzlmod Migration Status

## Overview

This document tracks the progress of migrating from WORKSPACE to bzlmod (MODULE.bazel) while maintaining compatibility with both systems.

## Current Status

### MODULE.bazel (bzlmod mode) ✅ Mostly Complete

All dependencies have been added to MODULE.bazel with versions aligned as closely as possible with WORKSPACE:

| Dependency | MODULE.bazel Version | WORKSPACE Version | Status |
|------------|---------------------|-------------------|---------|
| abseil-cpp | 20230802.1 | c2435f8342 (May 2023) | ✅ Aligned |
| protobuf | 29.3 | 315ffb5be89 (May 2023) | ⚠️ Different (see below) |
| googletest | 1.14.0 | f8d7d77c06 (v1.14.0) | ✅ Aligned |
| google_benchmark | 1.8.3 | 1.7.0 | ⚠️ Close (1.8.3 vs 1.7.0) |
| nlohmann_json | 3.11.3 | 3.11.3 | ✅ Aligned |
| googleapis | 0.0.0-20250826-a92cee39 | 1d5522ad10 (Sept 2020) | ⚠️ Different |
| googleapis-cc | 1.0.0 | N/A | ✅ Added for bzlmod |
| proto-converter | 2458ed8 (mmorel-35) | 2458ed8 (mmorel-35) | ✅ Aligned |
| rules_cc | 0.1.1 | Provided by rules_fuzzing | ⚠️ Different |

### Known Issues

#### 1. Proto-converter Compilation Error in Bzlmod Mode

**Status:** Blocked on upstream fix

**Error:**
```
error: testing if a concept-id is a valid expression; add 'requires' to check satisfaction [-Werror=missing-requires]
```

**Root Cause:** The proto-converter code in commit 3f393212 has a gcc compilation warning that is treated as an error. This is a compiler compatibility issue with newer gcc versions.

**Location:** `external/proto-converter~/src/google/protobuf/util/converter/utility.cc`

**Resolution:** This needs to be fixed in the mmorel-35/proto-converter repository, either by:
1. Fixing the code to satisfy the gcc warning
2. Adjusting compiler flags to not treat this warning as an error
3. Updating to a newer commit that has this fix

#### 2. Protobuf Version Mismatch

**Status:** Acceptable difference

**Explanation:**
- **MODULE.bazel uses protobuf 29.3** (October 2025): Required for bzlmod compatibility, includes modern Bazel infrastructure
- **WORKSPACE uses protobuf from May 2023**: The old version has BUILD file incompatibilities with Bazel 7.6 (exec_tools attribute issues), but newer versions (28.x, 29.x) require bzlmod infrastructure (rules_python, bazel_features) that creates circular dependencies in WORKSPACE mode

**Impact:** This is a fundamental limitation. Protobuf 29.x is designed for bzlmod and doesn't work well in legacy WORKSPACE mode.

**Recommendation:** Focus on bzlmod migration as the primary build system once proto-converter is fixed.

## Build Instructions

### Bzlmod Mode (Recommended)

```bash
# Build with bzlmod enabled (requires proto-converter fix)
bazel build --enable_bzlmod //src:all
```

### WORKSPACE Mode (Legacy, has compatibility issues)

```bash
# Build with WORKSPACE (currently has protobuf/Bazel 7.6 compatibility issues)
bazel build --noenable_bzlmod //src:all
```

## Next Steps

1. **Priority:** Fix proto-converter compilation error in mmorel-35/proto-converter repository
2. Once proto-converter is fixed, bzlmod mode should work fully
3. Consider deprecating WORKSPACE mode in favor of bzlmod, given the fundamental protobuf version incompatibility
4. Add CI/CD validation for both bzlmod and WORKSPACE modes (once both work)

## Dependencies on External Repositories

- **proto-converter (mmorel-35/proto-converter):** Needs compilation fix for gcc warning
  - Branch: bzlmod
  - Commit: 2458ed8ea405b47c1960f0b0af211efdf0e057a0

## References

- [Bazel bzlmod documentation](https://bazel.build/external/migration)
- [proto-converter bzlmod branch](https://github.com/mmorel-35/proto-converter/tree/bzlmod)
