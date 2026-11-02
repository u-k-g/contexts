# contexts

Long-lived personal contexts for an agent. A concept is one editable document,
not a log. The agent reads a generated map of concepts in `AGENTS.md`, then runs
`./ctx read` to pull the full text of whichever concepts a request touches, and
`./ctx write` to revise one.

The database is local and gitignored, so a clone starts empty and the tool is
safe to publish.

## Setup

Requires nushell with SQLite and FTS5. The nushell package in nixpkgs has both.

```nu
./ctx init
```

That creates `.data/contexts.db` and writes `AGENTS.md`. Both are gitignored.

## Commands

```nu
./ctx init                                     # create the database and AGENTS.md
./ctx new <slug> --title <t> --summary <s>     # create a concept
./ctx edit <slug> [--title <t>] [--summary <s>]
./ctx read <slug> [<slug> ...]                 # full text of those concepts
./ctx body <slug>                              # body only, safe to feed back to write
./ctx write <slug> --file <path>               # replace a document
./ctx history <slug>                           # earlier versions
./ctx revert <slug> <rev>                      # restore one
./ctx search <terms>                           # search every document
./ctx list                                     # concepts, sizes, dates
./ctx delete <slug>                            # delete a concept
./ctx reindex                                  # rebuild the search index
./ctx sync                                     # rewrite AGENTS.md
```

`write` reads the body from `--file`, or from stdin when `--file` is omitted:

```nu
"new text" | ./ctx write miata
```

It replaces the whole document and saves the previous text as a revision.

## Why nushell

One file, no build step, and no runtime beyond nushell. The trade is that a
script is holding long-lived state, which is normally a job for a compiled
program. Revisions on every write, WAL mode, and a narrow command surface keep
that contained.

## Where the state lives

`.data/contexts.db`, plus its `-wal` and `-shm` files while open. Copy the
database to back it up. Nothing in `.data/` is tracked.
