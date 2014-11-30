# frisby

Linear time composable parser for PEG grammars.

frisby is a parser library that can parse arbitrary PEG grammars in
linear time. Unlike other parsers of PEG grammars, frisby need not be
supplied with all possible rules up front, allowing composition of
smaller parsers.

PEG parsers are never ambiguous and allow infinite lookahead with no
backtracking penalty. Since PEG parsers can look ahead arbitrarily,
they can easily express rules such as the maximal munch rule used in
lexers, meaning no separate lexer is needed.

In addition to many standard combinators, frisby provides routines to
translate standard regex syntax into frisby parsers.

PEG based parsers have a number of advantages over other parsing strategies:

* PEG parsers are never ambiguous
* PEG is a generalization of regexes, they can be though of as
  extended regexes with recursion, predicates, and ordered choice
* you never need a separate lexing pass with PEG parsers, since you
  have arbitrary lookahead there is no need to break the stream into
  tokens to allow the limited LALR or LL lookahead to work.
* things like the maximal munch and minimal munch rules are trivial to
  specify with PEGs, yet tricky with other parsers
* since you have ordered choice, things like the if then else
  ambiguity are nonexistent.
* parsers are very very fast, guaranteeing time linear in the size of
  the input, at the cost of greater memory consumption
* the ability to make local choices about whether to accept something
  lets you write parsers that deal gracefully with errors very easy to
  write, no more uninformative parse error messages
* PEG parsers can be fully lazy, only as much of the input is read as
  is needed to satisfy the demand on the output, and once the output
  has been processed, the memory is immediately reclaimed since a PEG
  parser never backtracks
* PEG parsers can deal with infinite input, acting in a streaming
  manner
* PEG parsers support predicates, letting you decide what rules to
  follow based on whether other rules apply, so you can have rules
  that match only if another rule does not match, or a rule that
  matches only if two other rules both match the same input.
* Traditionally, PEG parsers have suffered from two major flaws:

A global table of all productions must be generated or written by
hand, disallowing composable parsers implemented as libraries and in
general requiring the use of a parser generator tool like pappy

Although memory consumption is linear in the size of the input, the
constant factor is very large.

frisby attempts to address both these concerns.

frisby parsers achieve composability by having a compilation pass,
recursive parsers are specified using the recursive do notation 'mdo'
which builds up a description of your parser where the recursive calls
for which memoized entries must be made are explicit. then runPeg
takes this description and compiles it into a form that can be
applied, during this compilation step it examines your composed
parser, and collects the global table of rules needed for a packrat
parser to work.

Memory consumption is much less of an issue on modern machines; tests
show it is not a major concern, however frisby uses a couple of
techniques for reducing the impact. First it attempts to create
parsers that are as lazy as possible -- this means that no more of the
file is read into memory than is needed, and more importantly, memory
used by the parser can be reclaimed as you process its output.

frisby also attempts to optimize your parser, using specialized
strategies when allowed to reduce the number of entries in your
memoization tables.

frisby attempts to be lazy in reading the results of parsers, parsers
tend to work via sending out 'feeler' predicates to get an idea of
what the rest of the file looks like before deciding what pass to
take, frisby attempts to optimize these feeler predicates via extra
lazyness such that they do not cause the actual computation of the
results, but rather just compute enough to determine whether a
predicate would have succeeded or not.

(It is interesting to note that the memory efficiency of frisby
depends vitally on being as lazy as possible, in contrast to
traditional thoughts when it comes to memory consumption)

frisby is a work in progress, it has a darcs repo at
http://repetae.net/repos/frisby which may be browsed at
http://repetae.net/dw/darcsweb.cgi?r=frisby;a=summary

And its homepage is at http://repetae.net/computer/frisby

to learn more about PEG parsers, see this paper
http://pdos.csail.mit.edu/~baford/packrat/popl04 and Bryan Ford's
packrat parsing page http://pdos.csail.mit.edu/~baford/packrat/
