# Klingon Dialect for Clang (kclang)

This is a small, end-to-end dialect of Clang that replaces C keywords with Klingon equivalents.

## Features

- **New driver name**: `kclang` - A symlink to `clang` that defaults to Klingon mode
- **Language switch**: Use `-x klingon` to explicitly enable Klingon mode
- **File extension**: Files with `.klingon` extension are automatically recognized
- **Minimal changes**: All changes are isolated behind a `LangOptions.Klingon` flag

## Keyword Mappings

See [files/SPEC.md](files/SPEC.md) for the complete list of keyword replacements.

### Examples

#### Control Flow
- `if` → `HIja` (yes condition)
- `else` → `ghobe` (no, otherwise)
- `for` → `vangqa` (repeatedly act)
- `while` → `tIq` (be enduring)
- `return` → `chegh` (return/retreat)

#### Types
- `int` → `mI` (number)
- `char` → `qIt` (symbol/character)
- `float` → `ghurtaH` (floating)
- `void` → `chIm` (be empty)

## Usage

### Using kclang driver

```bash
# Compile a Klingon source file
kclang myfile.klingon -o myprogram

# Or explicitly specify the language
kclang -x klingon myfile.c -o myprogram
```

### Using clang with -x klingon

```bash
# Compile with explicit language flag
clang -x klingon myfile.klingon -o myprogram
```

## Example Code

Here's a simple max function in Klingon:

```c
mI qoq(mI a, mI b) {
  HIja (a > b) {
    chegh a;
  } ghobe {
    chegh b;
  }
}

mI main() {
  mI x = 10;
  mI y = 20;
  mI result = qoq(x, y);
  chegh 0;
}
```

This is equivalent to the standard C code:

```c
int max(int a, int b) {
  if (a > b) {
    return a;
  } else {
    return b;
  }
}

int main() {
  int x = 10;
  int y = 20;
  int result = max(x, y);
  return 0;
}
```

## Implementation Details

The implementation consists of the following key changes:

1. **Language Support**:
   - Added `Klingon` to the `Language` enum in `LangStandard.h`
   - Added Klingon language option to `LangOptions.def`
   - Added Klingon language standard in `LangStandards.def`

2. **Input Type**:
   - Added `klingon` and `klingon-cpp-output` types to `Types.def`
   - Added file extension mapping for `.klingon` and `.ki` files

3. **Driver Support**:
   - Added `kclang` to driver suffixes in `ToolChain.cpp`
   - Added `kclang` symlink in `CMakeLists.txt`

4. **Keyword Translation**:
   - Modified `Preprocessor::LookUpIdentifierInfo()` to translate Klingon keywords to C keywords when `LangOpts.Klingon` is enabled
   - Translation happens before identifier lookup, ensuring full compatibility with C semantics

5. **Testing**:
   - Added driver test in `clang/test/Driver/klingon.klingon`
   - Added parser test in `clang/test/Parser/klingon_keywords.klingon`

## Building

To build LLVM/Clang with Klingon support:

```bash
mkdir build
cd build
cmake -G Ninja \
  -DCMAKE_BUILD_TYPE=Release \
  -DLLVM_ENABLE_PROJECTS="clang" \
  -DLLVM_TARGETS_TO_BUILD="X86" \
  ../llvm
ninja clang
```

After building, `kclang` will be available as a symlink to `clang`.

## Testing

Run the Klingon dialect tests:

```bash
# Run driver tests
ninja check-clang-driver

# Run parser tests
ninja check-clang-parser

# Or run all clang tests
ninja check-clang
```

## Design Principles

The implementation follows these principles:

1. **Minimal changes**: All Klingon-specific code is isolated behind the `LangOpts.Klingon` flag
2. **Zero impact on standard C**: When not in Klingon mode, there is no performance or behavioral impact
3. **Full C compatibility**: Klingon code is just C with different keywords - all C semantics are preserved
4. **Clean separation**: The translation happens at the lexer level, so the rest of the compiler sees standard C

## License

This is part of the LLVM Project and is available under the Apache License v2.0 with LLVM Exceptions.
See LICENSE.TXT for license information.
