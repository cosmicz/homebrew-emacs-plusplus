# homebrew-emacs-local

A local fork of [d12frosted/homebrew-emacs-plus](https://github.com/d12frosted/homebrew-emacs-plus)
that builds GNU Emacs 30.2 for macOS with two additional source patches
to the thread and process layers.  Everything under `Library/`, `scripts/`,
`community/` and the five cosmetic patches under `patches/emacs-30/` is
unmodified upstream code (MIT, see `LICENSE`).

## The two local patches

- `emacs-30.2-ebadf-retry-v3.patch` (`src/process.c`): `wait_reading_process_output`
  retries `select` on `EBADF` instead of aborting, and aborts only when a
  descriptor that is still registered with a process has been closed.  Fixes
  the reproducible abort when one thread deletes a process another thread is
  waiting on.  Includes the ERT test `process-tests/delete-process-other-thread-waiting`.
- `emacs-30.2-thread-last-error.patch` (`src/thread.c`, `src/thread.h`,
  `doc/lispref/threads.texi`): `thread-last-error` takes an optional `THREAD`
  argument and each thread retains its own last uncaught error, so an error in
  a non-main thread is no longer lost or overwritten by another thread.
  Arity becomes `(0 . 2)`.  Includes tests in `test/src/thread-tests.el`.

Both are candidates for upstream submission.  The formula pins each patch by
SHA-256.

## Install

```sh
brew tap cosmic/emacs-local https://github.com/cosmicz/homebrew-emacs-local
brew install --build-from-source cosmic/emacs-local/emacs-plus-local@30
```

The formula name `emacs-plus-local@30` keeps the install separate from
`emacs-plus@30`, so both kegs can coexist and the old one remains the
rollback.  Options are the same as the upstream `emacs-plus@30` formula.

## Provenance

The formula in the first commit is byte-identical to the one that built
the live keg on 2026-11-05 (binary SHA-256
`dc8984afe03cbbb547960ef14f274bf0be4048019ad335652af61ef135455ea1`,
ABI `30.2-8ac2a0dd`).  The block in `install` that rebuilds the
dependency environment exists because that build ran with
`--ignore-dependencies` against the already installed dependency set; it
is inert in a normal `brew install`.
