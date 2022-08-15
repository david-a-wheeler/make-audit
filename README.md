# make-audit

This is a simple, easy-to-use tool for auditing makefiles.
It will report when an execution of `make` either reads or changes
files in ways that are inconsistent with its Makefile.

## Prerequisites

`make-audit` requires also installing these OSS components before it can work:

* `pmaudit`, an "auditor" tool, in your PATH. Please
  download and install [boyski's pmaudit](https://github.com/boyski/pmaudit).
* GNU make (installed and run as `make`).
* `python3`  (this tool is primarily implemented in Python version 3.X).

## Running

To run `make-audit`, simply enter:

~~~~
make-audit [*MAKE-OPTIONS*]
~~~~

This will run make (which we presume is GNU make)
with the specified options

If you see no error `*** Error` output at the end,
everything is fine!!

Here are the errors it can report:

* `*** Error: Target TARGET : unreported prerequisites: SET` :
  The make recipe for creating TARGET is reading from the
  prerequisites in SET, but the makefile fails to report them as dependencies.
  You may want to add SET to the prerequisites of TARGET.
* `*** Error: Target TARGET : claimed but unused prerequisites: SET` :
  The make recipe for creating TARGET claims that it depends on SET,
  but the items in SET were never read.
  You may want to remove SET from the prerequisites of TARGET.
* `*** Error: Target TARGET : unreported target: SET`
  The make recipe for updating TARGET also modifies the files in SET
  but this is not reported. The easy solution is to use GNU make's
  grouped targets feature that it added in version 4.3,
  e.g., `TARGET SET &: PREREQUISITES`.
  If you don't want to use grouped targets, you can instead use a "marker
  file" to simulate grouped targets; just have TARGET and everything else
  depend on some marker file (`TARGET SET: MARKER_FILE`), then create
  a separate rule to create the marker file
  (`MARKER_FILE: PREREQUISITES`).
  Attach all commands to this latter rule that creates the marker file,
  and be sure to update the marker file (e.g., `touch MARKER_FILE`).
* `*** Error: Target TARGET : unmodified reported target: SET`
  The make recipe for updating TARGET does not appear to actually
  write to TARGET.
  You may need to add a missing command to actually modify TARGET.

## Auditor

To work, this depends on having an "auditor" tool that can
report the files that were read and the files that were written to
by an execution, as reported in a JSON file of a particular format.

We expect that will be pmaudit (poor man's audit).
That tool records the access times (atimes) and modification times (mtimes)
before and after an execution and reports the differences.

The `pmaudit` tool only notices file changes in the current directory
and below by default, so those are also the only changes we notice.

## TODO

NOTE: This is an *extremely* early version.
Much needs fixing, e.g.:

* This needs an easy-to-use install method (at least one).

* For example, this doesn't properly handle grouped targets or
empty commands.

* It should handle makefiles with their own SHELL and .ONESHELL values.

* I don't think it handles multi-line make commands exactly correctly
(it's close but not quite right).

* Lots more options are needed.
You should be able to control which make (make? gmake?), which auditor,
what directories to watch/exclude, etc.

* There should be a way to easily override false positives.
That is, a way to say "Report XXX specifically about YYY is not true".

* Lots more tests are needed.

## Related/similar OSS tools

[Checkmake](https://github.com/mrtazz/checkmake)
is an experimental tool for linting and checking Makefiles.

[make-booster](https://github.com/david-a-wheeler/make-booster)
provides utility routines intended to greatly simplify data processing
(particularly a data pipeline) using GNU make.

## License

This is released under the [MIT license](./LICENSE.md), a
well-known and widely-used open source software license.
This depends on other tools, in particular `pmaudit` and `GNU make`,
which have their own licenses.
