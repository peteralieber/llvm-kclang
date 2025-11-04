# Copilot Instructions for llvm-kclang

## Project Overview

This repository implements a Klingon dialect for Clang (kclang), which replaces C keywords with Klingon language equivalents. This is an educational/experimental project that demonstrates how to extend LLVM/Clang with custom language features while maintaining full C compatibility.

Key features:
- `kclang` driver - symlink to `clang` that defaults to Klingon mode
- `.klingon` file extension support
- Keyword translation at the lexer level (transparent to rest of compiler)
- Apostrophe (') treated as a letter (Klingon glottal stop), not punctuation

## Technology Stack

- **Language:** C++ (LLVM codebase standards)
- **Build System:** CMake + Ninja
- **Testing:** LLVM's lit testing framework
- **Linting:** clang-format, clang-tidy (see `.clang-format` and `.clang-tidy`)
- **Base Project:** LLVM/Clang compiler infrastructure

## Coding Standards

### General Guidelines
- Follow LLVM coding standards: https://llvm.org/docs/CodingStandards.html
- Use existing `.clang-format` configuration for code formatting
- All code changes should be minimal and surgical
- Isolate all Klingon-specific code behind `LangOpts.Klingon` flag

### Klingon-Specific Code
- All Klingon dialect code MUST be conditional on `LangOpts.Klingon`
- Never modify standard C/C++ behavior when not in Klingon mode
- Keyword translation happens in `Preprocessor::LookUpIdentifierInfo()`
- Apostrophe support is handled in `Lexer.cpp`

### File Organization
- Klingon language definitions: `clang/include/clang/Basic/LangStandard.h`, `LangStandards.def`
- Driver integration: `clang/lib/Driver/ToolChain.cpp`
- Lexer modifications: `clang/lib/Lex/Preprocessor.cpp`, `clang/lib/Lex/Lexer.cpp`
- Keyword mappings: See `files/SPEC.md` for complete reference

## Critical Code Review Guidelines

When performing a code review, pay close attention to code modifying a function's
control flow. Could the change result in the corruption of performance profile
data? Could the change result in invalid debug information, in particular for
branches and calls?

### Additional Review Focus Areas
- Ensure zero overhead when `LangOpts.Klingon` is false
- Verify apostrophe handling doesn't break standard C character literals
- Check that keyword translation is complete and consistent with SPEC.md
- Confirm debug information accuracy is preserved
- Validate that performance profiling remains unaffected

## Testing Requirements

### Test Framework
- Use LLVM's `lit` testing framework
- Tests are in `clang/test/` directory
- Run with: `ninja check-clang` or `ninja check-clang-parser`

### Test Coverage Requirements
- All new Klingon keywords MUST have test coverage in `clang/test/Parser/klingon_keywords.klingon`
- Driver changes require tests in `clang/test/Driver/klingon.klingon`
- Use RUN lines with FileCheck for validation
- Tests should use `expected-no-diagnostics` when no errors are expected

### Testing Commands
```bash
# Build and test
ninja clang
ninja check-clang-driver
ninja check-clang-parser
ninja check-clang
```

## Building the Project

```bash
mkdir build && cd build
cmake -G Ninja \
  -DCMAKE_BUILD_TYPE=Release \
  -DLLVM_ENABLE_PROJECTS="clang" \
  -DLLVM_TARGETS_TO_BUILD="X86" \
  ../llvm
ninja clang
```

## Security Considerations

- Never hardcode credentials or sensitive data
- Be cautious with string handling and buffer operations
- Validate all input in lexer modifications
- Ensure apostrophe handling doesn't introduce injection vulnerabilities

## Documentation

- Update `KLINGON_README.md` for user-facing changes
- Update `IMPLEMENTATION_NOTES.md` for technical changes
- Update `files/SPEC.md` when adding new keyword mappings
- All public APIs should have clear documentation

## Klingon Language Specifics

### Apostrophe as a Letter
The apostrophe (') is a LETTER in Klingon, representing a glottal stop. This has important implications:
- Keywords can contain apostrophes: `HIja'` (if), `ghobe'` (else), `mI'` (int)
- Identifiers can contain apostrophes: `pa'` (room), `Qapla'` (success)
- Character literals (`'a'`) are NOT supported in Klingon mode
- Use integer literals instead: `65` instead of `'A'`
- String literals with double quotes work normally: `"Hello"`

### Keyword Translation
- Translation happens at lexer level before identifier lookup
- See `files/SPEC.md` for complete keyword mapping
- Translation is transparent to parser, semantic analysis, and codegen

## Design Principles

1. **Minimal Changes:** All modifications should be as small as possible
2. **Zero Impact:** No performance or behavioral impact on standard C when not in Klingon mode
3. **Full Compatibility:** Klingon code is just C with different keywords
4. **Clean Separation:** Translation at lexer level keeps rest of compiler unchanged
5. **Authentic Klingon:** Use proper Klingon spelling with apostrophes

## Common Pitfalls to Avoid

- Don't modify behavior when `LangOpts.Klingon` is false
- Don't break character literal handling for standard C/C++
- Don't add keywords without updating SPEC.md and tests
- Don't modify hot paths without ensuring zero overhead for non-Klingon code
- Don't change AST representation, symbol tables, or debug info format

## Resources

- Keyword mappings: `files/SPEC.md`
- Implementation details: `IMPLEMENTATION_NOTES.md`
- User guide: `KLINGON_README.md`
- LLVM Contributing Guide: https://llvm.org/docs/Contributing.html
- LLVM Coding Standards: https://llvm.org/docs/CodingStandards.html
