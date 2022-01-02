## Clang

[![CI](https://github.com/JuliaInterop/Clang.jl/actions/workflows/ci.yml/badge.svg)](https://github.com/JuliaInterop/Clang.jl/actions/workflows/ci.yml)
[![TagBot](https://github.com/JuliaInterop/Clang.jl/actions/workflows/TagBot.yml/badge.svg)](https://github.com/JuliaInterop/Clang.jl/actions/workflows/TagBot.yml)
[![codecov](https://codecov.io/gh/JuliaInterop/Clang.jl/branch/master/graph/badge.svg)](https://codecov.io/gh/JuliaInterop/Clang.jl)
[![docs-stable](https://img.shields.io/badge/docs-stable-blue.svg)](https://JuliaInterop.github.io/Clang.jl/stable)
[![docs-dev](https://img.shields.io/badge/docs-dev-blue.svg)](https://JuliaInterop.github.io/Clang.jl/dev)
[![GitHub Discussions](https://img.shields.io/github/discussions/JuliaInterop/Clang.jl)](https://github.com/JuliaInterop/Clang.jl/discussions)

This package provides a Julia language wrapper for libclang: the stable, C-exported
interface to the LLVM Clang compiler. The [libclang API documentation](http://clang.llvm.org/doxygen/group__CINDEX.html)
provides background on the functionality available through libclang, and thus
through the Julia wrapper. The repository also hosts related tools built
on top of libclang functionality.

## Installation

```
pkg> add Clang
```

If you'd like to use the old generator(Clang.jl v0.13), please checkout [this branch](https://github.com/JuliaInterop/Clang.jl/tree/old-generator) for the documentation. Should you have any questions on how to upgrade the generator script, feel free to submit a post/request in the [Discussions](https://github.com/JuliaInterop/Clang.jl/discussions) area.

## Binding Generator

Clang.jl provides a module `Clang.Generators` for auto-generating C library bindings for Julia language from C headers.

### Quick start

Write a config file `generator.toml`:
```
[general]
library_name = "libclang"
output_file_path = "./LibClang.jl"
module_name = "LibClang"
jll_pkg_name = "Clang_jll"
export_symbol_prefixes = ["CX", "clang_"]
```

and a Julia script `generator.jl`:
```julia
using Clang.Generators
using Clang.LibClang.Clang_jll  # replace this with your jll package

cd(@__DIR__)

include_dir = normpath(Clang_jll.artifact_dir, "include")
clang_dir = joinpath(include_dir, "clang-c")

options = load_options(joinpath(@__DIR__, "generator.toml"))

# add compiler flags, e.g. "-DXXXXXXXXX"
args = get_default_args()
push!(args, "-I$include_dir")

headers = [joinpath(clang_dir, header) for header in readdir(clang_dir) if endswith(header, ".h")]
# there is also an experimental `detect_headers` function for auto-detecting top-level headers in the directory
# headers = detect_headers(clang_dir, args)

# create context
ctx = create_context(headers, args, options)

# run generator
build!(ctx)
```

Please refer to [this toml file](https://github.com/JuliaInterop/Clang.jl/blob/master/gen/generator.toml) for a full list of configuration options.

### Examples

The generator is currently used by several projects and you can take them as examples.

- [JuliaInterop/Clang.jl](https://github.com/JuliaInterop/Clang.jl): the thin low-level LibClang.jl is generated by this package itself
- [JuliaMultimedia/CSFML.jl](https://github.com/JuliaMultimedia/CSFML.jl): config multiple library names
- [JuliaGraphics/FreeType.jl](https://github.com/JuliaGraphics/FreeType.jl): select function-like macros
- [maleadt/LLVM.jl](https://github.com/maleadt/LLVM.jl): cross-platform support with patches
<!-- - [JuliaWeb/LibCURL.jl](https://github.com/JuliaWeb/LibCURL.jl): cross-platform build -->
- [scipopt/SCIP.jl](https://github.com/scipopt/SCIP.jl): applying custom patches by using `prologue.jl`&`epilogue.jl`
- [JuliaLang/SuiteSparse.jl](https://github.com/JuliaLang/SuiteSparse.jl): config compiler definitions for cross-platform
- [JuliaGPU/VulkanCore.jl](https://github.com/JuliaGPU/VulkanCore.jl): cross-platform build with third-party JLL dependencies
- [JuliaGPU/oneAPI.jl](https://github.com/JuliaGPU/oneAPI.jl): format the generated code with [JuliaFormatter.jl](https://github.com/domluna/JuliaFormatter.jl)

<!-- - [JuliaGPU/CUDA.jl](https://github.com/JuliaGPU/CUDA.jl/tree/master/res/wrap): old generator
- [SciML/Sundials.jl](https://github.com/SciML/Sundials.jl/blob/master/scripts/wrap_sundials.jl): old generator -->
