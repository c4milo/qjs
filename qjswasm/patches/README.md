# QuickJS Patches

This directory contains patches that are automatically applied to the QuickJS submodule during the build process.

## How It Works

1. **Build Process**: When you run `make build` or `make build-debug`, the Makefile automatically:
   - Applies all `.patch` files from this directory to the QuickJS submodule
   - Builds the WASM binary with the patched code
   - Cleans up by restoring the original files

2. **Patch Application**: Patches are applied in alphabetical order using `git apply`.

3. **Manual Control**:
   ```bash
   make apply-patches   # Apply all patches manually
   make clean-patches   # Remove applied patches (restore originals)
   ```

## Current Patches

### quickjs-wasm-stack-overflow.patch

**Purpose**: Fixes MaxStackSize heuristic to handle byte values correctly

**What it changes**:
- Removes arbitrary `< 100000` check that broke 256KB+ values
- Always calculates frame depth from stack_size when > 0
- Caps at MAX_SAFETY_DEPTH (100000) for safety

**Why needed**:
- Previous logic failed when MaxStackSize was set to 256KB (262144 bytes)
- This caused it to use default 1000 frames instead of calculated 256 frames

**Result**: MaxStackSize option now works correctly for all valid values

**Related documentation**: See `/WASM_STACK_OVERFLOW.md` for full details

## Adding New Patches

To add a new patch:

1. Make your changes to files in the `qjswasm/quickjs/` submodule
2. Create a patch file:
   ```bash
   cd qjswasm/quickjs
   git diff > ../patches/my-new-patch.patch
   ```
3. Test that it applies cleanly:
   ```bash
   make clean-patches  # Clean existing patches
   make apply-patches  # Should apply all patches including yours
   ```
4. Commit the patch file to the repository

## Patch Naming Convention

Use descriptive names that indicate what the patch does:
- `quickjs-<feature>-<description>.patch`
- Example: `quickjs-wasm-stack-overflow.patch`

## Troubleshooting

**Patch fails to apply:**
- Check that the patch was created from the correct base commit
- Verify the submodule is at the expected commit (check `.gitmodules`)
- Try applying manually to see the exact error:
  ```bash
  cd qjswasm/quickjs
  git apply ../patches/problematic-patch.patch
  ```

**Build fails after adding patch:**
- Ensure the patch doesn't introduce syntax errors
- Check that all modified files are restored by `make clean-patches`
- Test building without the patch to isolate the issue
