[perf](https://tonyday567.github.io/perf/index.html)
====================================================

[![Build
Status](https://travis-ci.org/tonyday567/perf.svg)](https://travis-ci.org/tonyday567/perf)
[![Hackage](https://img.shields.io/hackage/v/perf.svg)](https://hackage.haskell.org/package/perf)
[![lts](https://www.stackage.org/package/perf/badge/lts)](http://stackage.org/lts/package/perf)
[![nightly](https://www.stackage.org/package/perf/badge/nightly)](http://stackage.org/nightly/package/perf)

[repo](https://github.com/tonyday567/perf)

Performance experiments using the
[rdtsc](https://en.wikipedia.org/wiki/Time_Stamp_Counter) register on
x86.

Benchmarks
==========

The code for these benchmark runs can be found in
[examples/examples.hs](examples/examples.hs).

Benchmarks are based on:

    number of runs:         1.0e3
    accumulate to:          1.0e3
    function:               foldl' (+) 0

1 cycle = 0.38 𝛈s (Based on my 2.6GHz machine, by definition).

tick\_
------

    one tick_: 224 cycles
    next 10: [22,24,24,24,24,24,24,22,24,24]
    average over 1m: 19.29 cycles
    99.999% perc: 32,441
    99.9% perc: 54.06
    99th perc:  25.16
    40th perc:  17.18
    [min, 10th, 20th, .. 90th, max]:
     1.2000e1 1.5244e1 1.5883e1 1.6522e1 1.7181e1 1.7897e1 1.8613e1 1.9713e1 2.1372e1 2.3428e1 8.0698e4

tick
----

    sum to 1000
    first measure: 1954 cycles
    second measure: 2730 cycles

ticks
-----

    sum to 1000 n = 1000 prime run: 1.954e3
    run                       first     2nd     3rd     4th     5th  40th %
    ticks                    2.53e3  1.59e3  1.55e3  1.55e3  1.57e3 1.61e3 cycles
    ticks (lambda)           2.00e3  1.67e3  1.57e3  1.58e3  1.63e3 1.54e3 cycles
    ticks (poly)             2.32e3  1.68e3  1.63e3  1.64e3  1.63e3 1.61e3 cycles
    ticksIO                  1.92e3  1.66e3  1.59e3  1.55e3  1.54e3 1.59e3 cycles
    ticksIO (lambda)         1.69e3  1.57e3  1.55e3  1.55e3  1.54e3 1.60e3 cycles
    ticksIO (poly)           1.87e3  1.62e3  1.65e3  1.65e3  1.66e3 1.60e3 cycles

ticks cost
----------

Looking for hidden computation costs:

    n = 1.000e0 outside: 1.787e5 inside: 3.391e4 gap: 1.448e5
    n = 1.000e1 outside: 1.178e5 inside: 5.839e4 gap: 5.941e4
    n = 1.000e2 outside: 3.282e5 inside: 2.691e5 gap: 5.904e4
    n = 1.000e3 outside: 2.253e6 inside: 2.187e6 gap: 6.562e4

tickns
------

Multiple runs summing to a series of numbers.

    sum to's [1,10,100,1000]
    ns (ticks n fMono) as:  3.311e1 5.766e1 2.465e2 1.910e3
    (replicateM n . tick fMono) <$> as:  2.374e1 2.849e1 2.079e2 9.660e2

vector
------

    sum to 1000
    ticks list               2.71e4  2.06e4  1.98e4  1.96e4  2.03e4 1.59e4 cycles
    ticks boxed              7.39e3  6.88e3  6.86e3  6.83e3  6.83e3 6.96e3 cycles
    ticks storable           2.18e3  1.81e3  1.75e3  1.76e3  1.74e3 1.58e3 cycles
    ticks unboxed            2.58e3  2.10e3  2.05e3  2.01e3  2.00e3 2.02e3 cycles

whnf
----

    sum to 1000
    tick                      1.35e3 cycles
    tickWHNF                  1.86e3 cycles
    ticks                    2.68e3  1.40e3  1.35e3  1.33e3  1.33e3 1.40e3 cycles
    ticksWHNF                6.00e1  1.80e1  1.80e1  1.80e1  2.00e1 2.11e1 cycles
    tickIO                    2.34e3 cycles
    tickWHNFIO                7.20e1 cycles
    ticksIO                  2.12e3  1.40e3  1.36e3  1.36e3  1.43e3 1.64e3 cycles
    ticksWHNFIO              1.38e2  4.00e1  2.00e1  2.00e1  1.80e1 1.88e1 cycles

R&D, To Do
==========

metal speed
-----------

The average cycles per (+) operation can get down to 0.7, and there are
about 4 cache registers per cycle, so 2.8 low level instructions per
(+). Is this close to the metal speed?

unboxed versus boxed
--------------------

[ghc user
guide](https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/glasgow_exts.html?highlight=inline#unboxed-type-kinds)

strictness
----------

[All about
strictness](https://www.fpcomplete.com/blog/2017/09/all-about-strictness)

sharing
-------

-   turn on no-full-laziness and no-cse
-   add the inline pragma to `ticks`

comparative results

-   no-full-laziness & no-cse in Perf.Cycle (mono function)

<!-- -->

    ticks non-memo 1.28e5
    ticks inline memo
    ticks noinline no-memo
    ticks inlinable non-memo
    ticksIO non-memo 6.48e4

    no-full-laziness in Perf.cycle

    ticks non-memo 6.51e4
    ticks inline memo
    ticksIO non-memo 1.28e5

    no-cse

    ticks memo 7.66e4
    ticks inline memo
    ticksIO nomemo 6.47e4

polymorphism
------------

Can slow a computation down by 2x

lambda expressions
------------------

Can really slow things down

workflow
========

    stack build --test --exec "$(stack path --local-install-root)/bin/perf-examples" --exec "$(stack path --local-bin)/pandoc -f markdown -i other/header.md examples/bench.md other/footer.md -t html -o index.html --filter pandoc-include --mathjax" --exec "$(stack path --local-bin)/pandoc -f markdown -i examples/bench.md -t markdown -o readme.md --filter pandoc-include --mathjax" --file-watch

solo experiments:

    stack exec "ghc" -- -O2 -rtsopts examples/summing.lhs
    ./examples/summing +RTS -s -RTS --runs 10000 --sumTo 1000 --chart --chartName other/sum1e3.svg --truncAt 4

references
==========

-   [rts cheat
    sheet](https://www.cheatography.com/nash/cheat-sheets/ghc-and-rts-options/)

-   [ghc tips](http://ghc.readthedocs.io/en/8.0.2/sooner.html)

time
----

-   [Optimising haskell for a tight inner
    loop](http://neilmitchell.blogspot.co.uk/2014/01/optimising-haskell-for-tight-inner-loop.html)

-   [Tools for analysing
    performance](http://stackoverflow.com/questions/3276240/tools-for-analyzing-performance-of-a-haskell-program/3276557#3276557)

-   [Write haskell as fast as
    c](https://donsbot.wordpress.com/2008/05/06/write-haskell-as-fast-as-c-exploiting-strictness-laziness-and-recursion/)

-   [Reading ghc
    core](http://stackoverflow.com/questions/6121146/reading-ghc-core)

space
-----

-   [Chasing space leaks in
    shake](http://neilmitchell.blogspot.com.au/2013/02/chasing-space-leak-in-shake.html)

-   [Space leak zoo](http://blog.ezyang.com/2011/05/space-leak-zoo/)

-   [Anatomy of a thunk
    leak](http://blog.ezyang.com/2011/05/anatomy-of-a-thunk-leak/)

-   [An insufficiently lazy
    map](http://blog.ezyang.com/2011/05/an-insufficiently-lazy-map/)

-   [Pinpointing space leaks in big
    programs](http://blog.ezyang.com/2011/06/pinpointing-space-leaks-in-big-programs/)

memoization
-----------

http://okmij.org/ftp/Haskell/\#memo-off

cache cycle estimates
---------------------

  Cache               Cycles
  ------------------- ----------------
  register            4 per cycle
  L1 Cache access     3-4 cycles
  L2 Cache access     11-12 cycles
  L3 unified access   30 - 40
  DRAM hit            195 cycles
  L1 miss             40 cycles
  L2 miss             &gt;600 cycles

A performance checklist
-----------------------

1.  compile with rtsopts flag

<!-- -->

    stack ghc -- --make examples/examples.hs -rtsopts -fforce-recomp

2.  check GC `examples/examples +RTS -s`

3.  enabling profiling

-   a normal ghc:
    `stack ghc -- --make examples/examples.hs -rtsopts -fforce-recomp`
-   profile enabled automatically:
    `stack ghc -- --make examples/examples.hs -rtsopts -fforce-recomp -prof -auto -auto-all`
-   if template haskell:
    `stack ghc -- --make examples/examples.hs -rtsopts -fforce-recomp -prof -auto -auto-all -osuf p_o`

4.  create an examples.prof on execution:
    `time examples/examples +RTS -p`

5.  space

<!-- -->

    examples/examples +RTS -p -hc
    hp2ps -e8in -c examples/examples.hp
    hp2ps -e8in -c examples/examples.hy # types
    hp2ps -e8in -c examples/examples.hp # constructors

6.  check strictness pragmas

7.  space leaks

<!-- -->

    examples/examples +RTS -s - additional memory
    examples/examples +RTS -xt -hy

8.  read core

<!-- -->

        stack exec ghc-core -- --no-cast --no-asm --no-syntax examples/simplest.hs >> other/simplest.core

        alias ghci-core="stack ghci -- -ddump-simpl -dsuppress-idinfo -dsuppress-coercions -dsuppress-type-applications -dsuppress-uniques -dsuppress-module-prefixes"

simplest core dump (cleaned up)

    ==================== Tidy Core ====================
    Result size of Tidy Core = {terms: 22, types: 31, coercions: 9}

    -- RHS size: {terms: 2, types: 0, coercions: 0}
    $trModule2 :: TrName
    $trModule2 = TrNameS "main"#

    -- RHS size: {terms: 2, types: 0, coercions: 0}
    $trModule1 :: TrName
    $trModule1 = TrNameS "Main"#

    -- RHS size: {terms: 3, types: 0, coercions: 0}
    $trModule :: Module
    $trModule = Module $trModule2 $trModule1

    -- RHS size: {terms: 4, types: 7, coercions: 0}
    main1
      :: State# RealWorld -> (# State# RealWorld, () #)
    main1 =
      \ (s_a1o2 [OS=OneShot] :: State# RealWorld) ->
        (# s_a1o2, () #)

    -- RHS size: {terms: 1, types: 0, coercions: 3}
    main :: IO ()
    main = main1 `cast` ...

    -- RHS size: {terms: 2, types: 1, coercions: 3}
    main2
      :: State# RealWorld -> (# State# RealWorld, () #)
    main2 = runMainIO1 @ () (main1 `cast` ...)

    -- RHS size: {terms: 1, types: 0, coercions: 3}
    :main :: IO ()
    :main = main2 `cast` ...

simplest core dump (full)

    [1 of 1] Compiling Main             ( examples/simplest.hs, examples/simplest.o )

    ==================== Tidy Core ====================
    Result size of Tidy Core = {terms: 22, types: 31, coercions: 9}

    -- RHS size: {terms: 2, types: 0, coercions: 0}
    $trModule2 :: TrName
    [GblId,

     Unf=Unf{Src=<vanilla>, TopLvl=True, Value=True, ConLike=True,
             WorkFree=True, Expandable=True, Guidance=IF_ARGS [] 30 20}]
    $trModule2 = TrNameS "main"#

    -- RHS size: {terms: 2, types: 0, coercions: 0}
    $trModule1 :: TrName
    [GblId,

     Unf=Unf{Src=<vanilla>, TopLvl=True, Value=True, ConLike=True,
             WorkFree=True, Expandable=True, Guidance=IF_ARGS [] 30 20}]
    $trModule1 = TrNameS "Main"#

    -- RHS size: {terms: 3, types: 0, coercions: 0}
    $trModule :: Module
    [GblId,

     Unf=Unf{Src=<vanilla>, TopLvl=True, Value=True, ConLike=True,
             WorkFree=True, Expandable=True, Guidance=IF_ARGS [] 10 30}]
    $trModule = Module $trModule2 $trModule1

    -- RHS size: {terms: 4, types: 7, coercions: 0}
    main1
      :: State# RealWorld -> (# State# RealWorld, () #)
    [GblId,
     Arity=1,

     Unf=Unf{Src=InlineStable, TopLvl=True, Value=True, ConLike=True,
             WorkFree=True, Expandable=True,
             Guidance=ALWAYS_IF(arity=1,unsat_ok=True,boring_ok=False)
             Tmpl= \ (s_a1o2 [Occ=Once, OS=OneShot]
                        :: State# RealWorld) ->
                     (# s_a1o2, () #)}]
    main1 =
      \ (s_a1o2 [OS=OneShot] :: State# RealWorld) ->
        (# s_a1o2, () #)

    -- RHS size: {terms: 1, types: 0, coercions: 3}
    main :: IO ()
    [GblId,
     Arity=1,

     Unf=Unf{Src=InlineStable, TopLvl=True, Value=True, ConLike=True,
             WorkFree=True, Expandable=True,
             Guidance=ALWAYS_IF(arity=0,unsat_ok=True,boring_ok=True)
             Tmpl= main1 `cast` ...}]
    main = main1 `cast` ...

    -- RHS size: {terms: 2, types: 1, coercions: 3}
    main2
      :: State# RealWorld -> (# State# RealWorld, () #)
    [GblId,
     Arity=1,

     Unf=Unf{Src=<vanilla>, TopLvl=True, Value=True, ConLike=True,
             WorkFree=True, Expandable=True, Guidance=IF_ARGS [] 20 60}]
    main2 = runMainIO1 @ () (main1 `cast` ...)

    -- RHS size: {terms: 1, types: 0, coercions: 3}
    :main :: IO ()
    [GblId,
     Arity=1,

     Unf=Unf{Src=InlineStable, TopLvl=True, Value=True, ConLike=True,
             WorkFree=True, Expandable=True,
             Guidance=ALWAYS_IF(arity=0,unsat_ok=True,boring_ok=True)
             Tmpl= main2 `cast` ...}]
    :main = main2 `cast` ...

    Linking examples/simplest ...
