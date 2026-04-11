% super(1)
%
% April 2026

# NAME

super - process data with SuperSQL queries

# SYNOPSIS

**super** [*options*] *command*

**super** [*options*] [**-c** *query*] [*file* ...]

# DESCRIPTION

**super** is the command-line tool for interacting with and managing SuperDB. It is organized as a hierarchy of sub-commands but can also be invoked directly to execute SuperSQL queries detached from any database storage layer.

When invoked without a sub-command and with either a query or input paths, **super** runs the SuperDB query engine standalone. Input may be specified as command-line paths, via HTTP/HTTPS/S3 URLs, or read from standard input using `-` as the path. Inputs may also be referenced directly within the query using a **from** operator or a SQL `FROM` clause.

When multiple input files are specified they are processed in order as if concatenated into a single **from** clause. If no input is specified, the query receives a single `null` value, enabling use as a calculator or for standalone expressions.

Format detection is automatic for files with recognized extensions (`.json`, `.parquet`, `.sup`, etc.) and for standard input **super** attempts to detect the format by reading a sample of the data. The `line` format cannot be auto-detected and requires explicit `-i line`. When `-i` is used, all input files must share the same format; without it, each file's format is determined independently. Parquet and CSUP require seekable input and cannot be read from standard input.

Output is written to standard output by default. When stdout is a terminal the default output format is SUP; otherwise it is BSUP. These defaults may be overridden with `-f`, `-s`, or `-S`. If no query is specified with `-c`, inputs are scanned and re-serialized in the requested output format, providing a convenient format-conversion path.

Certain output formats (Arrow, Parquet) are schema-rigid and require all values to conform to a single schema. To write heterogeneous super-structured data to these formats, use the **fuse** or **blend** operator to unify types, or use `-split` to write one file per distinct type.

Fatal errors such as missing files or full filesystems cause **super** to exit immediately. Runtime errors produced by the query itself do not halt execution; instead they appear as first-class error values interleaved with normal output and can be detected with the **is_error** function.

The `-C` flag compiles and displays a query in canonical form without executing it, which is useful for understanding how abbreviated SuperSQL syntax expands into full pipe-query form.

# OPTIONS

## Global Options

**-h**
:   Display help.

**-help**
:   Display help.

**-hidden**
:   Show hidden options (default "false").

**-version**
:   Print version and exit (default "false").

## Query Options

**-aggmem** *size*
:   Maximum memory used per aggregate function value in MiB, MB, etc (default "auto(1GiB)").

**-c** *query*
:   SuperSQL query to execute; may be used multiple times.

**-C**
:   Display parsed AST in a textual format without executing (default "false").

**-dynamic**
:   Disable static type checking of inputs (default "false").

**-e**
:   Stop upon input errors (default "true").

**-fusemem** *size*
:   Maximum memory used by **fuse** in MiB, MB, etc (default "auto(1GiB)").

**-I** *file*
:   Source file containing query text; may be used multiple times.

**-sam**
:   Execute query in sequential runtime (default "false").

**-sortmem** *size*
:   Maximum memory used by **sort** in MiB, MB, etc (default "auto(1GiB)").

**-stats**
:   Display search stats on stderr (default "false").

**-vam**
:   Execute query in vector runtime (default "false").

## Input Options

**-bsup.readmax** *size*
:   Maximum Super Binary read buffer size in MiB, MB, etc (default "auto(1GiB)").

**-bsup.readsize** *size*
:   Target Super Binary read buffer size in MiB, MB, etc (default "auto(512KiB)").

**-bsup.threads** *n*
:   Number of Super Binary read threads; 0 means GOMAXPROCS (default "0").

**-bsup.validate**
:   Validate format when reading Super Binary (default "false").

**-csv.delim** *char*
:   CSV field delimiter (default ",").

**-e**
:   Stop upon input errors (default "true").

**-i** *format*
:   Format of input data: `auto`, `arrows`, `bsup`, `csup`, `csv`, `json`, `jsup`, `line`, `parquet`, `sup`, `tsv`, `zeek` (default "auto").

**-samplesize** *n*
:   Values to read per input file to determine type; less than 1 means all (default "1000").

## Output Options

**-B**
:   Allow Super Binary to be sent to a terminal output (default "false").

**-bsup.compress**
:   Compress Super Binary frames (default "true").

**-bsup.framethresh** *bytes*
:   Minimum Super Binary frame size in uncompressed bytes (default "524288").

**-color**
:   Enable/disable color formatting for `-S` and db text output (default "true").

**-f** *format*
:   Format for output data: `arrows`, `bsup`, `csup`, `csv`, `db`, `json`, `jsup`, `line`, `parquet`, `sup`, `table`, `tsv`, `zeek` (default "bsup").

**-J**
:   Shortcut for `-f json -pretty 2`, i.e., multi-line JSON (default "false").

**-j**
:   Shortcut for `-f json -pretty 0`, i.e., line-oriented JSON (default "false").

**-noheader**
:   Omit header for CSV and TSV output (default "false").

**-o** *file*
:   Write data to output file.

**-persist** *regexp*
:   Regular expression to persist type definitions across the stream.

**-pretty** *n*
:   Tab size to pretty-print JSON and Super JSON output; 0 for newline-delimited output (default "2").

**-S**
:   Shortcut for `-f sup -pretty 2`, i.e., multi-line SUP (default "false").

**-s**
:   Shortcut for `-f sup -pretty 0`, i.e., line-oriented SUP (default "false").

**-split** *dir*
:   Split output into one file per data type in the specified directory (see also `-splitsize`).

**-splitsize** *size*
:   If greater than 0 and `-split` is set, split into files at least this large rather than by data type (default "0B").

**-unbuffered**
:   Disable output buffering (default "false").

# SUB-COMMANDS

**compile**
:   Compile a SuperSQL query for inspection and debugging; see https://superdb.org/command/compile.html.

**db**
:   Run database commands; see https://superdb.org/command/db.html.

**dev**
:   Run specified development tool; see https://superdb.org/command/dev.html.

# OUTPUT

Output is always produced in super-structured form and serialized to the format specified by `-f`. The supported output formats are: `arrows`, `bsup`, `csup`, `csv`, `db`, `json`, `jsup`, `line`, `parquet`, `sup`, `table`, `tsv`, and `zeek`.

When writing to a terminal the default format is SUP; otherwise BSUP. The `db` format pretty-prints lake metadata as produced by `super db` sub-commands.

The `line` format writes one value per line. String values are printed as-is with escape sequences rendered as their native characters; non-string values are formatted as SUP. The `line` format cannot be auto-detected on input and requires `-i line`.

SUP and JSON output may be pretty-printed with `-pretty`, `-S`, or `-J`. Colorization is enabled by default when writing to a terminal and can be disabled with `-color false`.

BSUP and CSUP are self-describing, stream-oriented, and pipeline-friendly. Super-structured output from one **super** invocation can be piped directly into another without schema registration.

For schema-rigid formats (Arrow, Parquet), heterogeneous data must be unified before writing. Use the **fuse** or **blend** operator to produce a single type, or use `-split` to write one file per distinct output type. Files written with `-split` are named using the `-o` value as a prefix with a `-<n>.<ext>` suffix.

# ERRORS

Fatal errors (e.g., file not found, filesystem full) are reported immediately and cause **super** to exit.

Runtime query errors do not halt execution. They appear as first-class error values interleaved with normal output in the data stream. These errors can be detected and filtered using the **is_error** function. Errors may be wrapped in one another, providing stack-trace-like debugging output alongside result data.

# EXAMPLES

Query a CSV, JSON, or Parquet file using SuperSQL:

```
super -c "SELECT * FROM file.[csv|csv.gz|json|json.gz|parquet]"
```

Run a SuperSQL query sourced from an input file:

```
super -I path/to/query.sql
```

Pretty-print a sample value as super-structured data:

```
super -S -c "limit 1" file.[csv|csv.gz|json|json.gz|parquet]
```

Compute a histogram of the "data shapes" in a JSON file:

```
super -c "count() by typeof(this)" file.json
```

Display a sample value of each "shape" of JSON data:

```
super -c "any(this) by typeof(this) | values any" file.json
```

Search Parquet files easily and efficiently without schema handcuffs:

```
super *.parquet > all.bsup
super -c "? search keywords | other pipe processing" all.bsup
```

Read a CSV from stdin, process with a query, and write to stdout:

```
cat input.csv | super -f csv -c <query> -
```

Fuse JSON data into a unified schema and output as Parquet:

```
super -f parquet -o out.parquet -c fuse file.json
```

Run as a calculator:

```
super -c "1.+(1/2.)+(1/3.)+(1/4.)"
```

Search all values in a database pool called logs for keyword "alert" and level >= 2:

```
super db -c "from logs | ? alert level >= 2"
```

Traverse nested data with recursive functions and re-entrant subqueries:

```
super -c '
fn walk(node, visit):
  case kind(node)
  when "array" then
    [unnest node | walk(this, visit)]
  when "record" then
    unflatten([unnest flatten(node) | {key,value:walk(value, visit)}])
  when "union" then
    walk(under(node), visit)
  else visit(node)
  end
fn addOne(node): case typeof(node) when <int64> then node+1 else node end
values 1, [1,2,3], [{x:[1,"foo"]},{y:2}]
| values walk(this, &addOne)
'
```

Handle and wrap errors in a SuperSQL pipeline:

```
... | super -c "
switch is_error(this) (
    case true ( values error({message:\"error into stage N\", on:this}) )
    default (
        <non-error processing here>
        ...
    )
)
"
| ...
```

Embed a pipe query search in SQL FROM clause:

```
super -c "
SELECT union(type) as kinds, network_of(srcip) as net
FROM ( from logs.json | ? example.com AND urgent )
WHERE message_length > 100
GROUP BY net
"
```

Or write this as a pure pipe query using SuperSQL shortcuts:

```
super -c "
from logs.json
| ? example.com AND urgent
| message_length > 100
| kinds:=union(type), net:=network_of(srcip) by net
"
```

# SEE ALSO

SuperDB documentation: https://superdb.org

`super` command reference: https://superdb.org/command/super.html

SuperSQL language reference: https://superdb.org/super-sql/intro.html
