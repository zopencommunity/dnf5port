# Patches for dnf5 on z/OS

## PR1.patch

This patch contains z/OS-specific modifications for building dnf5:

### Key Changes:

1. **C++20 chrono::parse workaround** (`libdnf5-cli/session.cpp`):
   - Replaced `std::chrono::parse` with manual `strptime` parsing
   - `std::chrono::parse` is not available on z/OS even though the compiler supports C++20
   - Uses `mktime` for date conversion

2. **Compiler warnings** (`CMakeLists.txt`):
   - Removed `-Werror` flag to allow compilation with warnings
   - z/OS headers generate warnings that can't be easily fixed

3. **Platform-specific APIs**:
   - Added `__MVS__` guards for z/OS-specific code
   - Replaced `open_memstream` with custom `memstream_open`/`memstream_finish` functions
   - Replaced `fmemopen` with temp file-based implementation
   - Replaced `renameat2` with `rename` (RENAME_EXCHANGE not supported)
   - Set `RTLD_NODELETE` to 0 (not defined on z/OS)

4. **Filesystem operations** (`zos-filesystem.h`):
   - Created portable `portable_remove_all` wrapper for `std::filesystem::remove_all`
   - Works around potential issues with z/OS filesystem operations

5. **Date/time formatting**:
   - Custom `get_tz_offset` function for timezone formatting
   - `fmt::format` timezone specifier `%z` not fully supported on z/OS

6. **Standard I/O**:
   - Used `STDIN_FILENO`, `STDOUT_FILENO`, `STDERR_FILENO` instead of `fileno(stdin)` etc.
   - Changed subprocess result members from `stdout`/`stderr` to `out`/`err` (avoid reserved names)

7. **Library linking**:
   - Changed from shared to static library builds for libdnf5 and libdnf5-cli
   - Use `fmt::fmt-header-only` instead of shared fmt library
   - Fixed rpmbuild library name to `librpmbuild`

8. **Disabled features**:
   - Disabled automatic_plugin and needs_restarting_plugin (platform-specific issues)
   - Disabled some CMake linker flags not supported on z/OS

### Build Requirements:

- z/OS with C++20-capable compiler (clang)
- All dependencies from buildenv file installed via zopen
- zoslib for bridging z/OS LE runtime and POSIX APIs

### Known Issues:

- Some plugins disabled due to platform compatibility
- Tests may need adjustments for z/OS environment
- Date handling assumes local time rather than UTC (mktime limitation)
