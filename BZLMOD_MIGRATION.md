# Bzlmod Migration Status

## Overview

This document tracks the progress of migrating from WORKSPACE to bzlmod (MODULE.bazel) while maintaining compatibility with both systems.

## Current Status

### MODULE.bazel (bzlmod mode) ✅ Mostly Complete

All dependencies have been added to MODULE.bazel with versions aligned as closely as possible with WORKSPACE:

| Dependency | MODULE.bazel Version | WORKSPACE Version | Status |
|------------|---------------------|-------------------|---------|
| abseil-cpp | 20240116.2 | 4a2c63365e (LTS 20240116.0) | ✅ Aligned |
| protobuf | 29.3 | b407e8416e (v29.3) | ✅ Aligned |
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

**Root Cause:** The proto-converter code in commit 2458ed8 may have compilation warnings that are treated as errors. This is a compiler compatibility issue with newer gcc versions.

**Location:** `external/proto-converter~/src/google/protobuf/util/converter/utility.cc`

**Resolution:** This needs to be fixed in the mmorel-35/proto-converter repository, either by:
1. Fixing the code to satisfy the gcc warning
2. Adjusting compiler flags to not treat this warning as an error
3. Updating to a newer commit that has this fix

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
