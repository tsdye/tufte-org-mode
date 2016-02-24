# Introduction to Tufte Org Mode

Tufte Org Mode is designed to provide an Org mode environment for
writing books and handouts in a style developed and made famous by
[Edward R. Tufte](http://www.edwardtufte.com/tufte/index).  A characteristic of Tufte's style is a page layout
with a wide margin on one side (typically the right side) in which
notes, references, small tables, and small figures are placed.  The
style is widely admired, and it was a matter of time until the LaTeX
community produced the [Tufte-LaTeX classes](https://tufte-latex.github.io/tufte-latex/) to typeset books and handouts in
Tufte's style.

Tufte Org Mode consists of two files:

-   an Org mode file, `tufte-latex.org`, that contains documentation and
    setups for the Tufte-LaTeX book and Tufte-LaTeX handout classes, and
-   an Emacs Lisp file, `ox-tufte-latex.el`, derived from the `ox-latex.el`
    exporter written by Nicolas Goaziou, that implements an Org mode
    export backend for the Tufte-LaTeX classes.

The Tufte-LaTeX classes implement several non-standard LaTeX commands to
achieve a Tufte style page layout, including `marginfigure`,
`margintable`, and `sidenote`.  `Ox-tufte-latex.el` implements these
commands and `tufte-latex.org` gives examples of their use.

## The Tufte-LaTeX package

Most standard LaTeX distributions ship an older version of the Tufte-LaTeX
package.  Tufte Org Mode requires version 3.5.2 of the package, which at
the time of this writing was the version available from [the
Tufte-LaTeX web site](https://tufte-latex.github.io/tufte-latex/).  In particular, the Tufte-LaTeX package must support the
`nobib` option, which instructs the package not to load `natbib` support.
The `natbib` support in older versions of the Tufte-LaTeX package clashed with
`biblatex`, which is very useful in a Tufte-style document.

You'll need to install the latest version of the Tufte-LaTeX classes where your
LaTeX distribution can find them.  Please consult the documentation
for your LaTeX distribution for the best place to install local
packages. If all else fails, put them in the folder with your Tufte
Org Mode document, which is typically the first place LaTeX will look
for them.

### Other Required LaTeX Packages

In addition to the LaTeX packages required by the Tufte-LaTeX package,
Tufte Org Mode tries to load the following packages:

-   `etex`
-   `biblatex`
-   `booktabs`
-   `graphicx`
-   `microtype`
-   `hyphenat`
-   `marginfix`
-   `amsmath`
-   `morefloats`
-   `xparse`  (distributed as part of the l3packages bundle)
-   `xpatch`

These packages are all loaded in `#+LATEX_HEADER` lines.

### Patches to `biblatex`

The Tufte-LaTeX package was written while `biblatex` was under development and it
was not yet clear that it would be able to replace the venerable
`bibtex` package.  Accordingly, the developers of Tufte-LaTeX chose to base the
package's citation handling on the popular `natbib` package, which is
based on `bibtex`.

However, since that time `biblatex` has emerged as a more powerful and
flexible alternative to `bibtex`, and its facilities are very useful for
the humanities style citations used by Tufte.

If the Tufte-LaTeX classes are loaded with the `nobib` option, and `biblatex` is
also loaded, then the `footcite` command defined by `biblatex` can be used
out of the box to place citations in the document margin.  For many
documents, especially those with sparse marginal material, this might
represent a complete solution.  However, when there are many citations
or an abundance of other marginal material, items in the margin might
be placed incorrectly, leading most often to collisions where one item
is typeset over another.

A clever piece of code that addresses this problem was posted to the
[StackExchange TeX-LaTeX community](http://tex.stackexchange.com/questions/238661/is-it-possible-to-fine-tune-the-citation-positions-in-tufte-biblatex-combination?lq=1) by moewe.  It defines a `sidecite`
command that takes an optional parameter that can be used to shift a
citation up or down in the margin.  This code is loaded by
`tufte-latex.org` using `#+LATEX_HEADER:` lines:

    #+LATEX_HEADER: \usepackage{xparse}
    #+LATEX_HEADER: \usepackage{xpatch}
    #+LATEX_HEADER: 
    #+LATEX_HEADER: \makeatletter
    #+LATEX_HEADER: \xpatchcmd{\@footnotetext}%
    #+LATEX_HEADER:       {\color@begingroup}
    #+LATEX_HEADER:       {\color@begingroup\toggletrue{blx@footnote}}
    #+LATEX_HEADER:       {}
    #+LATEX_HEADER:       {}
    #+LATEX_HEADER: \makeatother
    #+LATEX_HEADER: 
    #+LATEX_HEADER: \DeclareCiteCommand{\sidecitehelper}
    #+LATEX_HEADER:   {\usebibmacro{prenote}}
    #+LATEX_HEADER:   {\usebibmacro{citeindex}%
    #+LATEX_HEADER:    \usebibmacro{cite}}
    #+LATEX_HEADER:   {\multicitedelim}
    #+LATEX_HEADER:   {\usebibmacro{cite:postnote}}
    #+LATEX_HEADER: 
    #+LATEX_HEADER: \ExplSyntaxOn
    #+LATEX_HEADER: \NewDocumentCommand\sidecite{D<>{}O{}om}{%
    #+LATEX_HEADER:   \iftoggle{blx@footnote}
    #+LATEX_HEADER:     {\cs_set_protected_nopar:Npn \__sct_wrapper:nn ##1 ##2 {\mkbibparens{##2}}}
    #+LATEX_HEADER:     {\cs_set_protected_nopar:Npn \__sct_wrapper:nn ##1 ##2 {\sidenote[][##1]{##2}}}
    #+LATEX_HEADER:     {\IfNoValueTF{#3}
    #+LATEX_HEADER:       {\__sct_wrapper:nn{#1}{\sidecitehelper[#2]{#4}}}
    #+LATEX_HEADER:       {\__sct_wrapper:nn{#1}{\sidecitehelper[#2][#3]{#4}}}}
    #+LATEX_HEADER: }
    #+LATEX_HEADER: \ExplSyntaxOff

## The `ox-tufte-latex.el` Exporter

The `ox-tufte-latex.el` exporter is currently under review as a
contribution to Org mode.  If it passes muster, then it will be
distributed with Org mode in the `contrib` folder and Emacs will be able
to find it in the same way it finds other files in `contrib`.

In the event `ox-tufte-latex.el` does not pass muster, then you will
have to make certain that Emacs can find it.  Typically, this means
that your installation location must appear in the list of directories
in the `load-path` variable.  To add your installation location to
`load-path` you will need to execute a command something like the
following example, perhaps in an initialization file:

    (add-to-list 'load-path "path/to/installation/location")

### Other Emacs Packages

The `tufte-latex.org` examples require two Emacs packages &#x2013; the `[[http://joostkremers.github.io/ebib/][Ebib`
package]] by Joost Kremers for managing a `biblatex` database, and [the
`ox-extra` package](http://orgmode.org/cgit.cgi/org-mode.git/plain/contrib/lisp/ox-extra.el) by Aaron Ecay to control which Org mode headlines are
exported.

The `Ebib` package provides a facility for formatting text that can be
populated with information from a `biblatex` database and inserted into
an Org mode buffer.  This facility is leveraged by Tufte Org Mode to
insert Org mode links that are exported as `biblatex` citation commands.
Six of these are [defined](#orgheadline1) &#x2013; `footcite`, `sidecite`, `cite`, `textcite`,
`parencite`, and `multicite`.  In practice, the Tufte Org Mode user calls
the function `ebib-insert-bibtex-key`, selects an entry from the
bibliographic database and a citation type, and then answers three or
four prompts.  When the final prompt is answered, `Ebib` inserts the
`biblatex` key into the Org mode buffer.

The `ox-extra` package implements a headline tag, `:ignore:`, that
activates a filter to remove the headline from export.  The `:ignore:`
tag is especially handy when writing a Tufte book that lacks Parts,
which are associated with first level headlines in Org Mode.  In this
case, the user simply tags first level headlines with `:ignore:`

    * This headline will not be exported                                 :ignore:

These two packages are loaded with `# eval:` lines in the Emacs local
variables list.

### Other Emacs Lisp Source Code Blocks

Three other Emacs Lisp source code blocks are evaluated as Emacs local
variables &#x2013; `user-entities`, `pdf-process-bibtex`, and `jk-keywords`.

The `user-entities` source code block adds entities commonly used in my
work to the `org-entities-user` list.  These are unlikely to be useful
for many users and they are included here as an example.  

The `pdf-process-bibtex` source code block defines the sequence of
commands that LaTeX will use to process the Tufte Org Mode document.
An alternative, `pdf-process-biber`, is also provided, in case you would
like to use [the modern `biber` package](http://biblatex-biber.sourceforge.net/), which intends to be a
replacement for `bibtex` that offers support for UTF-8, remote data
sources, and many other sophisticated facilities.

The `jk-keywords` source code block contains [two functions contributed
by John Kitchin](http://kitchingroup.cheme.cmu.edu/blog/2013/05/05/Getting-keyword-options-in-org-files/) that are used to retrieve keyword options in Org mode
files.

# Keywords

Tufte Org Mode defines several keywords that it uses primarily to
construct the front matter of a book:

-   **FULLNAME:** the full name of the copyright holder;
-   **PUBLISHER:** the book publisher;
-   **PRINT-DATE:** the month and date of printing;
-   **COPYRIGHT-DATE:** the copyright year; and
-   **WEB-SITE:** the URL for the book.

In addition, the keyword `MARGIN-NOTE-FONT` can be used to select a font
for margin notes, which are unnumbered notes that can appear in the
margin. 

# Macros

Org mode macros are most useful for small bits of text because they
don't work across line breaks.  Macros that potentially deal with
longer pieces of text have counterparts among the [links](#orgheadline2), which are
capable of handling text with line breaks.

Tufte Org mode defines several Org mode macros for convenience.

-   **newthought:** The first few words of each section are identified as
    a new thought and typeset in small caps.
-   **sidenote:** This macro takes three arguments:
    -   the text of the note
    -   optionally, a number for the note (if none is given, one will be assigned)
    -   an offset expressed as a LaTeX length, where positive values move
        the note down in the margin and negative values move it up.
-   **marginnote:** this macro puts an unnumbered note in the margin, and
    takes two arguments:
    -   the text of the note
    -   an offset expressed as a LaTeX length, where positive values move
        the note down in the margin and negative values move it up.
-   **tl:** a convenience macro that will result in a properly typeset
    package name, Tufte-LaTeX.

# Links

I think Org mode links are an outstanding feature.  They make it very
easy to extend Org mode and at least [one user has proposed to make
them more extensible](https://lists.gnu.org/archive/html/emacs-orgmode/2010-08/msg00404.html).  This document uses links for citations and for
inserting arbitrary LaTeX commands.

## Citation Links

Citation links are inserted by Ebib.  The `footcite` link will place a
citation in the margin at a location chosen by LaTeX.  You won't be
able to move it if it collides with something else.  [The `sidecite` link](http://tex.stackexchange.com/questions/238661/is-it-possible-to-fine-tune-the-citation-positions-in-tufte-biblatex-combination?lq=1)
has an `offset` option that lets you move the citation up or down as you
wish.  The other citation links &#x2013; `cite`, `textcite`, and `parencite` &#x2013;
are most useful in notes.

The biblatex package offers a `footcites` command with an unusual
syntax.  This is implemented in Tufte Org mode by placing the ƒ
character (alt-f on my keyboard) in the text where the footnote number
should be placed and following it with two or more `multicite` links.
If you use the ƒ character in your work, then you'll want to change
the character used in the filter.

One current limitation of Tufte Org mode is the [lack of an analogous
`sidecites` command](http://tex.stackexchange.com/questions/290446/sidecites-command-for-biblatex-and-tufte-latex?lq=1).  In practical terms, this limitation means that if
you are citing multiple works, each with pre- and/or post-notes, then
you'll need to use `footcites` and hope for the best.  Alternatively,
the `sidecite` link can handle multiple bibliography keys, you just
won't be able to add pre- or post-notes to them individually.

## Links for LaTeX commands

There are two general purpose links that can be used for inserting
arbitrary LaTeX commands.  The `latex` link is useful for commands
without optional arguments.  For example, it can be used for the
`newthought` command, which introduces the first few words of the first
paragraph in a section:

    [[latex:newthought][The first few words]] of the sentence.

There is also a `newthought` macro that accomplishes the same thing.

The `latex-opt` link is useful for commands that have one optional
argument.  The optional argument is taken from the description part of
the link.  It is separated by a semi-colon from the required argument
that starts the description.  This is useful for things like long
marginnotes that would break an Org mode macro:

    [[latex-opt:marginnote][This is a long margin note that is going to babble on and on until it
    extends past the point that it could be handled easily by an Org mode
    macro.;1in]]

Note that the use of `;` to separate the arguments means that this
character shouldn't appear in the note.  If your notes need
semi-colons, then you'll want to edit the link definition to use some
other separator character.

Note, too, that Org mode will recognize LaTeX fragments, so it is
possible to enter the raw LaTeX directly, rather than relying on
links.  I like to use links because the buffer looks cleaner and less
cluttered, which helps me concentrate on the text and flow of an
argument. 

# Headings

The [Tufte book class](../tufte-latex.md) defines headings for Part, Chapter, Section,
Subsection, and Paragraph.  Part maps to first level Org mode headlines,
Chapter to second level Org mode headlines, and so on.

If you don't want a book with Parts, then you can use the `:ignore:` tag
implemented in the `ox-extra` library by Aaron Ecay with your first
level headers:

    * First level headline is a Part                                     :ignore:
    ** Second level headline is a Chapter
    *** Third level headline is a Section
    **** Fourth level headline is a Subsection
    ***** Fifth level headline is a Paragraph

The [Tufte handout class](../tufte-latex.md) defines headings for Section and Subsection
and these are mapped to first and second level Org mode headings,
respectively. 

    * First level headline is a Section
    ** Second level headline is a Subsection

# Text

It is sometimes the case that you'll want a block of text to run wider
than usual and extend into the margin.  This is done with the
`fullwidth` environment, which you can create with the standard Org mode
solution of a `#+begin_fullwidth` `#+end_fullwidth` pair.

    #+begin_fullwidth
    Some long text that you want to run into the margin.
    #+end_fullwidth

# Tables

The Tufte LaTeX classes support three table sizes: one that fits in
the text block, another that fits in the margin, and a third that
spans the text block and the margin.  Which kind you get is determined
by the `:float` attribute.

This example shows a table that will be placed in the text block.

    #+name: tab:text-block
    #+caption[Example in-text table]: Example table in the text.
    #+caption: Note that the caption is placed in the margin.
    #+attr_latex: :font \footnotesize
    | First | Second | Third | Fourth | Fifth | Sixth |
    |-------+--------+-------+--------+-------+-------|
    | One   | Two    | Three | Four   | Five  | Stop  |
    | Six   | Seven  | Eight | Nine   | Ten   | Here  |

Here is an example of a table placed in the margin.  Note `:float
margin` in the `#+attr_latex:` line.

    #+name: tab:marginal
    #+caption[Example marginal table]: Example marginal table.
    #+caption: Note that the table and the caption are placed in the margin.
    #+attr_latex: :booktabs nil :font \footnotesize :float margin :offset -2in
    | First | Second | Third |
    |-------+--------+-------|
    | One   | Two    | Three |
    | Six   | Seven  | Eight |

Here is an example of a table that can span the text block and
margin.  Note `:float multicolumn` in the `#+attr_latex:` line.

    #+name: tab:full-width
    #+caption[Example full width table]: Example full width table.
    #+caption: Note that the caption is placed in the margin.
    #+attr_latex: :font \footnotesize :float multicolumn
    | First | Second | Third | Fourth | Fifth | Sixth |
    |-------+--------+-------+--------+-------+-------|
    | One   | Two    | Three | Four   | Five  | Stop  |
    | Six   | Seven  | Eight | Nine   | Ten   | Here  |

# Figures

Figures also come in three widths, just like tables.  However, figures
have two additional attributes that adjust the alignment of the
caption: `vertical-alignment` and `horizontal-alignment`.  The
`vertical-alignment` attribute can be set to `t`, to align the caption
with the top of the figure, or `b`, to align it with the bottom.
Sometimes, a figure will be placed on one page and its caption will
appear on another.  In this case, the `horizontal-alignment` attribute
can be set to `l`, to make the float verso, or `r`, to make the float
recto.

The default text-width figure is 4.375 in. wide.

    #+name: fig:text-block
    #+caption[Hilbert curves]: Hilbert curves of various degrees /n/.  
    #+caption: Note that this figure only takes up the main text block width.
    #+caption: Note also that the caption in the margin is aligned with the bottom of the image.
    #+attr_latex: :vertical-alignment b
    [[file:hilbertcurves.pdf]]

A figure that spans the text block and the margin is 6.75 in. wide.
Note `:float multicolumn` in the `#+attr_latex:` line.

    #+name: fig:full-width
    #+caption[Sine wave]: This graph shows a sine wave.
    #+caption: Note that the figure takes up the full page width.
    #+attr_latex: :float multicolumn
    [[file:sine.pdf]]

A figure placed in the margin can be 2 in. wide.  A margin figure is
specified by `:float margin` in the `#+attr_latex:` line.  The position of
the figure in the margin can be adjusted up or down using the `:offset`
attribute, which takes a LaTeX length.  A negative length will move
the figure up in the margin and a positive length will move the figure
down.

    #+name: fig:marginal
    #+caption[Helix in the margin]: Helix in the margin.
    #+caption: Note that this figure fits in the margin.
    #+attr_latex: :float margin :width 2in :offset -2.5in 
    [[file:helix.pdf]]
