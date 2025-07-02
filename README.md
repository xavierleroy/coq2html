# Rocqnavi: an HTML documentation generator for Rocq prover

## Overview

Rocqnavi is a fork of [coq2html](https://github.com/xavierleroy/coq2html). It generates HTML documentation from Rocq files. Proof scripts are hidden by default but can be revealed by clicking ([folding in action](https://compcert.org/doc/html/compcert.common.Memory.html#Mem.valid_access_dec)).

The initial motivation for this fork it to provide a documentation tool for [MathComp-Analysis](https://github.com/math-comp/analysis).

**Warning:** To produce cross-references, rocqnavi reads `.glob` files produced by Rocq. Inaccuracies in `.glob` files sometimes cause rendering issues (examples: [Rocq issue 18516](https://github.com/rocq-prover/rocq/issues/18516), [coq-elpi issue 575](https://github.com/LPCIC/coq-elpi/issues/575)). The format of `.glob` files is [documented](https://github.com/coq/coq/blob/master/interp/dumpglob.mli). `.glob` files exists since Coq 7.3 (2002-05) but their format has been changing silently between major releases of Coq; it was first documented officially in 2021.

This fork extends coq2html with:
* generation of indexes (like `coqdoc`)
* clickable notations (like `coqdoc`)
* an option `-Q <dir> <coqdir>`
* Markdown and LaTeX notations in comments
* darkmode
* a sidebar that displays the modules tree
* design for mobile phone (wip)

### Examples of documentation generated using this fork of coq2hml:

* [MathComp-Analysis](https://github.com/math-comp/analysis): https://yoshihiro503.github.io/rocqnavi/analysis/
* [Infotheo](https://github.com/affeldt-aist/infotheo): https://yoshihiro503.github.io/rocqnavi/infotheo/
* TODO: [Monae](https://yoshihiro503.github.io/rocqnavi/monae/): https://yoshihiro503.github.io/rocqnavi/monae/
* TODO: [CompCert](https://compcert.org/): https://yoshihiro503.github.io/rocqnavi/compcert/

## Usage

```
          rocqnavi [options] file.glob ... file.v ...
```

Option                     | Summary
---------------------------|----------------------------
`-base` _COQDIR_           | Set the name space for the modules being processed
`-coqlib` _URL_            | Set base URL for Rocq standard library
`-d` _DIR_                 | Output files to directory _DIR_ (default: current directory)
`-external` _URL_ _COQDIR_ | Set base URL for linking references whose names start with _COQDIR_
`-no-css`                  | Do not add rocqnavi.css to the output directory
`-redirect`                | Generate redirection files
`-short-names`             | Use short, unqualified module names in the output
`-title` _TITLE_           | Set the title of the index page
`-Q` _DIR_ _COQDIR_        | Map the directory _DIR_ to correspond to the module name _COQDIR_ (similar to `coqc`)
`-hierarchy-graph` _FILE_  | Show the hierarchy graph of <dot-file> on the index.html (You need Graphviz command line tool)
`-index-blacklist` _FILE_  | Exclude specified items from the index

### Usage example

Let us assume that the project [MathComp-Analysis](https://github.com/math-comp/analysis) is in the directory `analysis` and that the `rocqnavi` binary is installed in the directory `rocqnavi` with the following file hierarchy:
```tree
.
├── rocqnavi/
└── analysis/
```
Then the following command generates the documentation of MathComp-Analysis [TODO: update]:
```console
../rocqnavi/rocqnavi \
  -title "MathComp-Analysis" \
  -d html/ -base mathcomp -Q theories analysis \
  -coqlib https://coq.inria.fr/doc/V8.18.0/stdlib/ \
  -external https://math-comp.github.io/htmldoc/ mathcomp.ssreflect \
  -external https://math-comp.github.io/htmldoc/ mathcomp.algebra \
  $(find classical/ theories/ -name "*.v" -or -name "*.glob")
```

### HTML generation

rocqnavi takes one or several Rocq source files with extension `.v`, pretty-prints their contents, and saves the generated HTML text to files with extension `.html`.

By default all files are generated in the current working directory.  The `-d` _DIR_ option makes rocqnavi generate files in the given directory instead.  The directory _DIR_ must exist before rocqnavi is started.

In addition to HTML files, rocqnavi also produces two auxiliary files in the output directory given by `-d` or by default in the current directory:
* rocqnavi.css: the style sheet for the generated HTML files;
* rocqnavi.js: auxiliary JavaScript code that implements proof folding.

The `-no-css` option suppresses the generation of the rocqnavi.css file.  Users of this option are expected to provide their own style sheet.  It must be named rocqnavi.css and it must reside in the directory where rocqnavi generates its files.

For a source file `F.v` in the current directory and if no `-base` option is given, the generated HTML file is named `F.html`.  If the source file is in a subdirectory, or if the `-base` option is given, a Rocq-Style fully-qualified name is used as the name of the generated HTML file, as shown below:

`-base` option     |  Source file name     |   Generated HTML file name
-------------------|-----------------------|---------------------------
none               | `F.v`                 | `F.html`
none               | `D/E/F.v`             | `D.E.F.html`
`-base A.B.C`      | `F.v`                 | `A.B.C.F.html`
`-base A.B.C`      | `D/E/F.v`             | `A.B.C.D.E.F.html`

As strange as it looks, this file naming convention is compatible with that of coqdoc and ensures that file names are globally unique across multiple Rocq libraries and packages.

The fully-qualified name is also used to produce the title of the generated HTML page.  In the last example above, the fully-qualified name is A.B.C.D.E.F and the title of the page will be **Module A.B.C.D.E.F**.  If the `-short-names` option is given, the unqualified, local name is used instead, giving **Module F** as title.

Earlier versions of rocqnavi used short names `F.html` instead of fully-qualified names to name the generated HTML files.  For backward compatibility, the `-redirect` option can be given.  It causes an additional `F.html` file to be generated, containing a HTTP redirection to the file `A.B.C.D.E.F.html`.  The `-redirect` option is silently ignored if no `-base` option is given.

### Cross-referencing

rocqnavi can generate cross-references as hyperlinks from uses to definitions of Rocq names, provided the appropriate `.glob` files are given on the command-line.

A cross-reference file `F.glob` is generated by the Rocq compiler coqc when it processes the `F.v` source file.  (Unless the `-no-glob` option is passed to coqc; don't do that.)  When giving `F.v` as argument to rocqnavi, also give `F.glob` as argument, so that rocqnavi can use those cross-references to produce hyperlinks.

**Important:** if the source files are compiled within a Rocq namespace, you must give a `-base` option to rocqnavi indicating this namespace.  For example, if you compile with
```
        coqc -R A.B.C .
```
meaning that the current directory means namespace A.B.C, then you must invoke rocqnavi with
```
        rocqnavi -base A.B.C
```

By default, cross-references are generated if the referenced definition is
* in one of the `.glob` files given to rocqnavi on the command line, or
* in the Rocq standard library.

For this reason, to get better cross-referencing, you should either do a single run of rocqnavi with all the `.v` and `.glob` files of your Rocq development, or give all the `.glob` files of your library to every run of rocqnavi on every `.v` file.

The cross-references to the Rocq standard library use the online version of this library at https://coq.inria.fr/library/, which corresponds to the latest release of Rocq.  If you wish to reference a specific version of the standard library, use the `-coqlib` option, e.g.
```
        rocqnavi -coqlib https://coq.inria.fr/doc/V8.14.1/stdlib
```
for the 8.14.1 version of the standard library.

Using the `-external` option, you can add cross-references to other external libraries whose coqdoc or rocqnavi-generated documentation is accessible online.  For example,
```
        rocqnavi -external https://math-comp.github.io/htmldoc_1_12_0 mathcomp ...
```
should produce cross-references to the `mathcomp` library modules.  (Warning: untested feature.)

## Special handling of proof scripts

Proof scripts are Rocq text (outside comments) that
* starts with `Proof` or `Next Obligation` at the beginning of a line;
* ends with `Qed.` or `Defined.` or `Save.` or `Admitted.` or `Abort.` at the end of a line.

A proof script can start and end on the same line, e.g. `Proof. auto. Qed.`

Proof scripts are formatted in a smaller font and folded by default, leaving only the starting line (`Proof`, etc) visible.  Clicking on this first line displays the whole proof script.

The syntax of proof scripts is strict.  In particular, after stating a `Theorem` or `Lemma`, it does not work to omit the `Proof` keyword and start the script immediately, nor to abort it immediately with `Admitted.` or `Abort.`.  Likewise, the dot `.` must follow `Qed`, `Defined`, etc, without spaces.  For example, the following proof scripts won't be properly formatted:
```
Lemma x:...
auto. Qed.                      (* No "Proof." to mark the beginning of the script. *)

Lemma x:...
Admitted.                       (* No "Proof." to mark the beginning of the script. *)

Lemma x:...
Proof. auto. Defined .          (* Whitespace between "Defined" and "." *)

Lemma x:...  Proof. Admitted.   (* "Proof" must start a new line. *)
```

## Documentation comments

Documentation comments start with
- `(** ` (two stars followed by a space),
- `(**r ` (two stars, the "r" character, one space), or
- `(**md ` (two stars, the "md" string, one space).
```
     (** This is a documentation comment. *)
     (**r This is a right-aligned documentation comment. *)
     (**md This is a Markdown comment. *)
     (* This is a regular comment. *)
```
Regular, non-documentation comments are removed from the HTML output, except within proof scripts, where they are kept as is.

Documentation comments of the `(**` kind are formatted as described next, then inserted as a paragraph in the HTML output.  Right-aligned documentation comments of the `(**r` kind are formatted likewise, but do not start a paragraph.  Instead, they hang to the right of the Rocq code on the same line.  Example:
```
     (** This is the type of lists. *)

     Inductive list (A: Type) : Type :=
     | nil                         (**r the empty list *)
     | cons (hd: A) (tl: list A).  (**r the  nonempty list [hd::tl] *)
```

Documentation comments of the `(**md` kind are interpreted as Mardown input with possibly LaTeX mathematical notations (between `$` and `$`) inserted.

In addition, comments formatted as follows (80 characters wide)
```
     (**md***************************************************************************)
     (*                                                                             *)
     (* Some Mardown + LaTeX comment.                                               *)
     (*                                                                             *)
     (*******************************************************************************)
```
are displayed inside a frame.

## Markup language for rocqnavi

Inside documentation comments that are not Markdown, one can use the following features to improve the rendering of comments.
Many of them are subsumed by the Mardown language available in Mardown + LaTeX documentation comments which
should be preferred when possible.

### Inline Rocq text

Within a documentation comment, text within square brackets `[...]` is taken to be Rocq text and is formatted in monospace font.  You can write `[x + S y]` and will get `x + S y`.  Square brackets nest properly, hence `[[x;y]]` gives `[x;y]`.

### Inline HTML

Within a documentation comment, text between hash signs `#...#` is treated as pure HTML and copied verbatim to the output, without escaping of HTML special characters.  Hence, `#...#` escapes are the only way to insert HTML tags.  Example:
```
    (** 32-bit integers are less than #2<sup>32</sup># *)
```

### Verbatim text

Verbatim text starts with `<<` on a line by itself and ends with `>>` on a line by itself.  The lines between those two markers are copied to the output, in typewriter font, respecting newlines.  Example:
```
(** This is normal documentation text.
<<
        This is verbatim text.
        Second line of verbatim text.
>>
    This is normal text again. *)
```

Warning: `<<` and `>>` must be at the beginning of a line, without any space before.

### Sections

Section titles are denoted by one to four `*` characters at the beginning of a documentation comment:
```
    (** * Section title *)
    (** ** Subsection *)
    (** *** Sub-sub section *)
    (** **** Sub-sub-sub section *)
```
The section title extends until the end of the documentation comment.

### Lists of items

Lists start with a dash `-` at the beginning of the line.  Subsequent lines starting with a dash are items in the list.  A blank line terminates the list.  Example:
```
(** A list is either:
-     [nil], denoting the empty list;
-     [cons h t], also written [h :: t], denoting the nonempty list
      with head [h] and tail [t].

  If we remove the [nil] case and declare a [CoInductive], we get infinite
  streams instead of lists. *)
```
Nested lists are built using two, three or four dashes instead of one:
```
- Outer list, item #1
-- Inner list, item #1
-- Inner list, item #2
- Outer list, item #2
```

### Special handling of proof scripts

Proof scripts are Rocq text (outside comments) that
* starts with `Proof` or `Next Obligation` at the beginning of a line;
* ends with `Qed.` or `Defined.` or `Save.` or `Admitted.` or `Abort.` at the end of a line.

A proof script can start and end on the same line, e.g. `Proof. auto. Qed.`

Proof scripts are formatted in a smaller font and folded by default, leaving only the starting line (`Proof`, etc) visible.  Clicking on this first line displays the whole proof script.

The syntax of proof scripts is strict.  In particular, after stating a `Theorem` or `Lemma`, it does not work to omit the `Proof` keyword and start the script immediately, nor to abort it immediately with `Admitted.` or `Abort.`.  Likewise, the dot `.` must follow `Qed`, `Defined`, etc, without spaces.  For example, the following proof scripts won't be properly formatted:
```
Lemma x:...
auto. Qed.                      (* No "Proof." to mark the beginning of the script. *)

Lemma x:...
Admitted.                       (* No "Proof." to mark the beginning of the script. *)

Lemma x:...
Proof. auto. Defined .          (* Whitespace between "Defined" and "." *)

Lemma x:...  Proof. Admitted.   (* "Proof" must start a new line. *)
```

## Known limitations

- Cross-referencing (HTML links on identifiers that jump to the definition of the identifiers) is implemented for identifiers bound at the top-level of a Rocq source file (by `Definition`, `Fixpoint`, `Inductive`, `Module`, `Variable`, etc), but not for identifiers bound within Rocq terms (by `fun`, `match`, `forall`, etc). There is no cross-referencing for user-defined notations either.

- All non-ASCII Unicode characters are treated as being parts of identifiers.  Hence, `A ⊕ B` is read as three identifiers, `A`, `⊕`, and `B`, and links are correctly added to `A` and `B` if they are bound at top-level.  However, `A⊕B` is read as a single identifier, which has no corresponding definition, therefore no link is added.

- The formatting of right-hanging documentation comments `(**r` is inflexible and not appropriate for narrow display windows or long source lines.
