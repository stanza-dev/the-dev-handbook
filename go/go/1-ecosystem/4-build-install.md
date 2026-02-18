---
source_course: "go"
source_lesson: "go-build-install"
---

# Building & Installing Go Programs

## Introduction

Go's compiler produces single, statically-linked binaries that run anywhere—no runtime dependencies, no DLL hell, no interpreter needed. This simplicity revolutionizes deployment: copy one file, and it runs. This lesson covers the build process, cross-compilation, and production optimizations.

## Key Concepts

**Static Binary**: An executable containing all dependencies, requiring no external libraries at runtime.

**Cross-Compilation**: Building binaries for different operating systems and architectures from a single machine.

**Build Tags**: Conditional compilation directives that include or exclude files based on constraints.

**ldflags**: Linker flags that inject values at build time or strip debugging information.

## Real World Context

Docker images for Go applications can be as small as 5MB (using `scratch` base images) because the binary is self-contained. CI/CD pipelines can build Linux, Windows, and macOS binaries in parallel from a single codebase. Companies distribute CLI tools as single executables without asking users to install Go.

## Deep Dive

### Basic Building

```bash
go build                    # Build current package
go build -o myapp           # Specify output name
go build ./cmd/server       # Build specific package
```

### Installing Binaries

```bash
go install                           # Install current module
go install github.com/user/tool@v1.0 # Install remote tool
```

Binaries go to `$GOBIN` (default: `$GOPATH/bin`).

### Cross-Compilation

Set `GOOS` and `GOARCH` environment variables:

```bash
# Common targets
GOOS=linux   GOARCH=amd64 go build -o app-linux
GOOS=darwin  GOARCH=arm64 go build -o app-mac-m1
GOOS=windows GOARCH=amd64 go build -o app.exe
```

### Production Build Flags

```bash
# Strip debug info (smaller binary)
go build -ldflags="-s -w" -o myapp

# Embed version at build time
go build -ldflags="-X main.version=1.0.0" -o myapp

# Enable race detector (development only)
go build -race -o myapp
```

### Build Tags

```go
//go:build linux
// +build linux

package main
// This file only compiles on Linux
```

## Common Pitfalls

1. **Using `-race` in production**: The race detector adds significant overhead (10x slower, 10x memory). It's for development only.

2. **Forgetting CGO implications**: Cross-compilation with CGO enabled requires a C cross-compiler. Set `CGO_ENABLED=0` for pure Go builds.

3. **Not stripping binaries**: Production binaries include debug symbols by default, making them 30-50% larger than necessary.

## Best Practices

- Use `-ldflags="-s -w"` for production to reduce binary size.
- Embed version information using `-ldflags="-X main.version=$(git describe)"`.
- Create a Makefile or build script for reproducible multi-platform builds.
- Test cross-compiled binaries on target platforms before release.

## Summary

Go produces statically-linked binaries using `go build`. Cross-compilation is achieved by setting `GOOS` and `GOARCH`. Use `go install` for tools and `-ldflags` for production optimizations. Build tags enable platform-specific compilation.

## Code Examples

**Cross-compilation**

```bash
# Cross-compile for multiple platforms
GOOS=darwin GOARCH=arm64 go build -o myapp-mac-arm64
GOOS=linux GOARCH=amd64 go build -o myapp-linux-amd64
GOOS=windows GOARCH=amd64 go build -o myapp.exe
```


## Resources

- [Go Build Command](https://pkg.go.dev/cmd/go#hdr-Compile_packages_and_dependencies) — Official documentation for go build
- [Build Constraints](https://pkg.go.dev/go/build#hdr-Build_Constraints) — Guide to using build tags

---

> 📘 *This lesson is part of the [Go Programming](https://stanza.dev/courses/go) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*