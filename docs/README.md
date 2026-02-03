# Documentation

This project uses Sphinx with Nix flakes for reproducible documentation builds.

## Local Development

From the `docs/` directory:

```bash
# Build HTML documentation
nix run .#make -- html

# Build PDF documentation
nix run .#make -- latexpdf

# Build EPUB documentation
nix run .#make -- epub

# Enter development shell with all tools available
nix develop
```

## Building Packages

From the repository root:

```bash
# Build HTML package
nix build ./docs#html

# Build PDF package
nix build ./docs#pdf
```

Build output will be in `result/share/doc/`.

## GitLab CI

The CI pipeline automatically:

- Builds and validates docs on merge requests
- Publishes to GitLab Pages on pushes to main branch
