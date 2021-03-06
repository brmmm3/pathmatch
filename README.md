`pathmatch`: a better `fnmatch`
====

This is an experiment to develop an improved alternative to [POSIX `fnmatch` function](http://pubs.opengroup.org/onlinepubs/9699919799/functions/fnmatch.html). It can be used, for example, when implementing `.gitignore`-like functionality.

The module interface is currently a single global function, without extra options:

    pub fn pathmatch(pattern: &str, pathstring: &str) -> bool

Note: this implementation does not exactly match actual `.gitignore` and contains more functionality (e.g. `{}` patterns).

A quick taste of the idea
----

`*.txt` matches file names without path, if they end with “.txt”:

    one.txt
    two.txt

`**.txt` matches any path ending with “.txt”:

    one.txt
    two.txt
    foo/3.txt
    foo/bar/4.txt

`**/build/**` would match any path containing “build” folder anywhere in it, including the bare string `build` itself.

`**/build/{Debug,Release}` matches paths whose two last folders are either `build/Debug` or `build/Release`:

    build/Debug
    build/Release
    subproject1/build/Debug
    subproject1/build/Release

In other words, `**/` may match start of the path string, `/**` may match end of the path string, and `/**/` may match single path separator (`/`). To disable this behavior, explicitly require something to be present before/after: `*/**/build/{Debug,Release}`, `*/**/build/**/*`.

The `?` wildcard is also supported, matching exactly one character excluding path separator.

`pmfind` command line tool
----

You can use the included `pmfind` tool to test `pathmatch()` in real world:

      $ ./pmfind '{*.rs,Makefile}' '**/{logs,master}'
    pmfind.rs
    pathmatch.rs
    Makefile
    .git/refs/remotes/origin/master
    .git/refs/heads/master
    .git/logs
    .git/logs/refs/remotes/origin/master
    .git/logs/refs/heads/master

Usage:

    Usage:
            ./pmfind [options] [pattern ...]

    Options:
        -C --dir dir        change directory to this before starting
        -h --help           print help and exit

If a pattern starts with `!`, `pmfind` excludes paths matching it from output, otherwise those paths are printed. Patterns are applied in the order they appear on the command line, so the latest match decides whether the path will be printed. Paths that match no patterns are never printed. Example:

      $ ./pmfind '**/master'
    .git/refs/remotes/origin/master
    .git/refs/heads/master
    .git/logs/refs/remotes/origin/master
    .git/logs/refs/heads/master

      $ ./pmfind '**/master' '!**/logs/**'
    .git/refs/remotes/origin/master
    .git/refs/heads/master

      $ ./pmfind '**/master' '!**/logs/**' '**/logs/refs/heads/master'
    .git/refs/remotes/origin/master
    .git/refs/heads/master
    .git/logs/refs/heads/master

