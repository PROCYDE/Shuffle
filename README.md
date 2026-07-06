# PROCYDE Shuffle Fork

This is a fork of [Shuffle](https://github.com/shuffle/Shuffle) managed by PROCYDE.
It is used for PROCYDE-internal Shuffle deployments and as a base for upstream pull requests.

## FAQ

### Can I contribute to this fork?

No. This fork is for PROCYDE-internal usage only. Please contribute to the upstream [Shuffle](https://github.com/shuffle/Shuffle) repository instead.

### Can you host or manage Shuffle for us?

No. Check out [Shuffle Cloud](https://shuffler.io) for managed Shuffle.

## What's different from upstream

The upstream [Shuffle](https://github.com/shuffle/Shuffle) project spans several repositories (`Shuffle`, `shuffle-shared`, `shuffle-apps`, `orborus`, etc.). This fork brings them together:

- `shuffle-shared/` — included as a **git submodule** pointing to [PROCYDE/shuffle-shared](https://github.com/PROCYDE/shuffle-shared)
- `orborus/` — included as a **git submodule** pointing to [PROCYDE/shuffle-orborus](https://github.com/PROCYDE/shuffle-orborus)

A `devenv` and `go.work` is provided for easier development.

## Local development setup

### Prerequisites

- Nix
- Devenv
- Docker

### Clone

```bash
git clone --recurse-submodules git@github.com:PROCYDE/Shuffle.git
cd shuffle
```

If you already cloned without `--recurse-submodules`, initialize the submodule:

```bash
git submodule update --init
```

### Go workspace

This repo includes a `go.work` file at the root. This means Go resolves `import "github.com/shuffle/shuffle-shared"` from the local `shuffle-shared/` directory — no manual `replace` edits needed.

### Running locally

**Infrastructure** (Opensearch):

```bash
docker compose up -d opensearch
```

**Backend:**

```bash
go run ./backend/go-app
```

The backend listens on port `5001` by default.

**Frontend:**

```bash
cd frontend
npm install --legacy-peer-deps
npm run dev
```

The Vite dev server runs on port `3000` and proxies `/api` requests to `http://localhost:5001`.

Open `http://localhost:3000` in a browser.

## Making changes to shuffle-shared

The `shuffle-shared/` directory is a git submodule pointing to [PROCYDE/shuffle-shared](https://github.com/PROCYDE/shuffle-shared). When you make changes inside it:

```bash
cd shuffle-shared

# Check out a branch (you're in detached HEAD by default)
git checkout -b my-feature

# Make changes, commit, and push to the PROCYDE fork
git push procyde HEAD:my-feature

# Create a pull request on github.com/PROCYDE/shuffle-shared

# When the PR merges, update the submodule reference in the main repo
cd ..
git add shuffle-shared
git commit -m "bump shuffle-shared to include <feature>"
```

After pushing changes to the PROCYDE fork, update `backend/go-app/go.mod` to reference the new commit if you want Docker/CI builds to pick them up:

```bash
cd backend/go-app
go mod tidy
```

### Adding upstream remote for contributions

When you want to contribute changes back to the upstream `shuffle/shuffle-shared` repository, add it as a remote inside the submodule:

```bash
cd shuffle-shared
git remote add upstream https://github.com/shuffle/shuffle-shared.git

# Push your feature branch to your fork
git push procyde HEAD:my-feature

# Open a pull request from PROCYDE/shuffle-shared:my-feature → upstream shuffle/shuffle-shared:main
```

## Merging upstream changes

### Main repo (PROCYDE/Shuffle)

Sync with upstream `Shuffle/Shuffle`:

```bash
git fetch upstream
git merge upstream/main   # or upstream/nightly
```

Resolve any conflicts and commit.

### Submodule (shuffle-shared)

Update the submodule to the latest upstream `shuffle-shared`:

```bash
cd shuffle-shared

# Fetch from PROCYDE fork and upstream
git fetch --all

# Merge upstream changes into the PROCYDE main branch
git checkout main
git merge upstream/main
git push procyde main

cd ..
git add shuffle-shared
git commit -m "sync shuffle-shared with upstream"
```

## Upstream links

- [Shuffle](https://github.com/shuffle/Shuffle) — main upstream repository
- [shuffle-shared](https://github.com/shuffle/shuffle-shared) — Go shared library
- [orborus](https://github.com/shuffle/orborus) — workflow execution distribution
- [python-apps](https://github.com/shuffle/python-apps) — app definitions
- [Documentation](https://shuffler.io/docs)
