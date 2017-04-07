# Diesel [![Build Status](https://travis-ci.org/lloydmeta/diesel.svg?branch=master)](https://travis-ci.org/lloydmeta/diesel)

Boilerplate free Finally Tagless DSL annotation.

WIP

## General idea

This plugin provides an annotation that expands a given trait into an object
holding a Tagless Final Algebra and DSL wrapper methods.

Example:

```scala
// Declare your DSL
@diesel
trait Maths[F[_]] {
  def int(i: Int): F[Int]
  def add(l: F[Int], r: F[Int]): F[Int]
}

// Write your interpreter
type Id[A] = A
val interpreter = new Maths.Algebra[Id] {
  def int(i: Int) = i
  def add(l: Id[Int], r: Id[Int]) = l + r
}

// Use your stuff.
import Maths._
int(3)(interpreter) shouldBe 3
add(int(3), int(10))(interpreter) shouldBe 13
```

## How it works

```scala
@diesel
trait Maths[F[_]] {
  def int(i: Int): F[Int]
  def add(l: F[Int], r: F[Int]): F[Int]
}
```

is expanded into

```scala
object Maths {

  import scala.language.higherKinds
  import _root_.diesel.Dsl

  trait Algebra[F[_]] {
    def int(i: Int): F[Int]

    def add(l: F[Int], r: F[Int]): F[Int]
  }

  def int(i: Int): Dsl[Algebra, Int] = new Dsl[Algebra, Int] {
    def apply[F[_]](implicit interpreter: Algebra[F]): F[Int] = interpreter.int(i)
  }

  def add(l: Dsl[Algebra, Int], r: Dsl[Algebra, Int]): Dsl[Algebra, Int] = new Dsl[Algebra, Int] {
    def apply[F[_]](implicit interpreter: Algebra[F]): F[Int] = interpreter.add(l.apply[F], r.apply[F])
  }
}
```

## Sbt

Currently published as a SNAPSHOT on Sonatype

```scala
libraryDependencies += "com.beachape" %% "diesel-core" % "0.1.0-SNAPSHOT"


// Additional ceremony for using Scalameta macro annotations

resolvers += Resolver.url(
  "scalameta",
  url("http://dl.bintray.com/scalameta/maven"))(Resolver.ivyStylePatterns)

// A dependency on macro paradise is required to both write and expand
// new-style macros.  This is similar to how it works for old-style macro
// annotations and a dependency on macro paradise 2.x.
addCompilerPlugin(
  "org.scalameta" % "paradise" % "3.0.0-M7" cross CrossVersion.full)

scalacOptions += "-Xplugin-require:macroparadise"

```

# Credit
1. [Free vs Tagless final talk](https://github.com/cb372/free-vs-tagless-final)
2. [Alternatives to GADTS in Scala](https://pchiusano.github.io/2014-05-20/scala-gadts.html)
3. [Quark talk](https://www.slideshare.net/jdegoes/quark-a-purelyfunctional-scala-dsl-for-data-processing-analytics)