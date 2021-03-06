[![Clojars Project](http://clojars.org/poc.cml/latest-version.svg)](http://clojars.org/poc.cml)

# poc.cml

Proof-of-Concept CML-style composable first-class events on top of
[core.async](https://github.com/clojure/core.async/).

## Why and what is CML?

Concurrent ML is a concurrent programming language developed by
[John Reppy](http://people.cs.uchicago.edu/~jhr/) in the late 1980's and early
1990's.  It is based on ideas, namely synchronous message passing and
non-deterministic choice, that you can find in
[CSP](http://en.wikipedia.org/wiki/Communicating_sequential_processes) and
[Pi-calculus](http://en.wikipedia.org/wiki/%CE%A0-calculus).  CML then extends
the idea of non-deterministic choice over synchronous channel operations to
first-class composable events with negative acknowledgments (nacks).

With plain core.async, one can express a non-deterministic choice over a linear
sequence of synchronous get and put operations:

```clojure
(alt!
  <channel>           ([<result>] <action>) ;; get operation
  [<channel> <value>] ([<sent>] <action>)   ;; put operation
  ...)
```

With CML, one has the following combinators (and more) for expressing
first-class events:

```clojure
(gete <channel>)             ;; An event to take a value on a channel
      <channel>              ;;   and alternative syntax for gete
(pute <channel> <value>)     ;; An event to give a value on a channel
     [<channel> <value>]     ;;   and alternative syntax for pute
(choose <event> ...)         ;; Non-deterministic choice over events
(wrap <event> <action-fn>)   ;; An event with a post synchronization action
(guard <event-thunk>)        ;; An event with a pre synchronization action
(with-nack <nack->event-fn>) ;; An event with a pre sync action given a nack
```

Compared to plain core.async, non-deterministic choices can be easily nested and
actions can be attached both before and after synchronization.  Furthermore,
there is a combinator that provides negative acknowledgment in case an event
wasn't ultimately chosen.

The plain core.async `alt!` grammar can be expressed using just a subset of the
combinators

```clojure
(sync!
  (choose
    (wrap (gete <channel>)         (fn [<result>] <action>))
    (wrap (pute <channel> <value>) (fn [<sent>] <action>))
    ...))
```

where `sync!` is an operation that synchronizes on a given event and returns its
result.  For convenience, `sync!` has an implicit `choose`, so the `choose` can
be dropped, and we can also use the alternative syntax for `gete` and `pute`:

```clojure
(sync!
  (wrap <channel>           (fn [<result>] <action>))
  (wrap [<channel> <value>] (fn [<sent>] <action>))
  ...)
```

Written this way, the `sync!` expression is just slightly more verbose than the
`alt!` expression, and we could certainly add further sugar to make the `sync!`
version more concise, but the key here is that the combinators `choose`, `wrap`,
`pute` and `gete` are just ordinary functions that return values that can be
further manipulated with other combinators, stored in data structures and even
passed through channels.

BTW, note that `gete` and `pute` don't end with a bang `!`.  That is because all
the effects are actually performed by `sync!` and the values returned by the
event combinators are immutable representations of selective synchronous
operations.

See the [examples](examples) for further documentation.

The book
[Concurrent Programming in ML](http://www.cambridge.org/us/academic/subjects/computer-science/distributed-networked-and-mobile-computing/concurrent-programming-ml)
is the most comprehensive introduction to Concurrent ML style programming.

## Next steps

What is interesting is that CML style events do not require significantly more
complicated machinery than what core.async already provides.  This
proof-of-concept library hopefully makes that clear.  For production use, you'd
want to implement the CML mechanisms directly.  Also note the one exception
raised in the implemention [here](src/poc/cml.cljc#L45).  The `alts!` operation
of core.async does not make it possible to distinguish between multiple
different operations on a single channel, which breaks composability.

## License

Copyright © 2015 Vesa Karvonen

Distributed under the Eclipse Public License either version 1.0 or (at your
option) any later version.
