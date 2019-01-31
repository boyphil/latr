# Latr

A scala macro for reasonable lazy semantics.

This library provides three annotation macros:

`@lazify`: Rewrites a `def` or `val` so that its evaluation is delayed and memoized. Does not synchronize or attempt to share the memo between threads. If two threads call the unevaluated expression at the same time, the evaluation will happen twice.

`@lazifyPessimistic`: Rewrites a `def` or `val` to something similar to a `lazy val`, except instead of locking the whole object while evaluating, it locks only while checking if evaluation has already happened. Essentially implements [SIP-20](https://docs.scala-lang.org/sips/improved-lazy-val-initialization.html) Version V2. Performs best if the expected thread contention is low.

`@lazifyOptimistic`: Rewrites a `def` or `val` to an atomic memory cell that uses _compare-and-swap_ to share the memo among multiple threads. Performs better than the pessimistic version when contention is high, but worse when contention is low. Essentially implements [SIP-20](https://docs.scala-lang.org/sips/improved-lazy-val-initialization.html) Version V4.

## Setup

Latr requires the [macro paradise compiler plugin](https://docs.scala-lang.org/overviews/macros/paradise.html). Follow the instructions there to set up the plugin. If you're using SBT, you should just need a line like this in your `build.sbt`:

``` scala
addCompilerPlugin("org.scalamacros" % "paradise" % "2.1.0" cross CrossVersion.full)
```

If you're using SBT 0.13.6 or better, you can get Latr from Bintray thusly: 

``` scala
resolvers += Resolver.bintrayRepo("runarorama", "maven")

libraryDependencies += "com.higher-order" %% "latr" % "0.3.0"
```

With older versions of SBT you'll need this instead:

``` scala
resolvers += Resolver.url("runarorama bintray", url("http://dl.bintray.com/runarorama/maven"))
```

## Usage

Wherever you would otherwise say something like this in Scala:

``` scala
lazy val x: Int = {
  println("I will only print once and block the world while doing it!")
  10
}
```

You can instead say:

``` scala
import latr._

@lazify val x: Int = {
  println("I may print multiple times under thread contention, but I'm super cheap!")
  10
}
```

Or:

``` scala
@lazifyOptimistic val x: Int = {
  println("I will only print once, using fancy atomic stuff!")
  10
}
```

Or:

``` scala
@lazifyPessimistic val x: Int = {
  println("I will only print once, using good old-fashioned synchronized blocks!")
  10
}
```

