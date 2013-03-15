title: Functional programming

## Functional programming

Let's start with some definitions: [(1)][1][(5)][5][(8)][8]

**Covariant Functor** — defines the operation commonly known as map or fmap.

    map ( A=>B ) => ( C[A]=>C[B] )   or   map C[A] => (A=>B) => C[B]

**Monad:** — defines the operation commonly known as bind, flatMap or =<<.

    flatMap ( A=>C[B] ) => ( C[A]=>C[B] )
    insert A => C[A]

Monads also define an identity operation, insert

**Applicative Functor** — defines the operation commonly known as apply or <*>.

    apply ( C[A=>B] ) => ( C[A]=>C[B] )
    insert A => C[A]

Applicative functors also define an identity operation, insert

**Exponential Functors** — defines the operation commonly known as xmap. Also known as invariant functors.

    xmap ( (A => B, B => A) ) => ( F[A] => F[B] )

**Contravariant Functor** — defines the operation commonly known as contramap.

    contramap ( B=>A ) => ( C[A]=>C[B] )

**Comonad** — defines the operation commonly known as extend, coflatMap or <<=.

    coflatMap ( F[A] => B ) => ( F[A] => F[B] )
    extract F[A] => A

Comonads also define an identity operation, extract

Implementation details:
=======================

Let's assume that we have

    class Container[T](val value:T)

// Covariant Functor [(1)][1][(2)][2][(3)][3]

    def map[A,B] ( rawfunc:A=>B ) = (a:Container[A]) => new Container( rawfunc(a.value) )

or

    def map[A,B](a:Container[A]): (A=>B)=>Container[B] = (rawfunc:A=>B) => new Container( rawfunc(a.value) )

or

    class Container[A](val value:A) {
      def map[A,B](rawfunc:A=>B) = new Container( rawfunc(this.value) )
    }

or

    implicit class ContainerWrapper[T](w:Container[T]) {
      def flatMap[R](f:T=>Container[R]) = map(f).value // monad
      def map[R](f:T=>R) = new Container(f(w.value)) // functor
    }

or [(5)][5]

    trait Functor[T[_]] {
      def fmap[A,B](f: A=>B)(ta: T[A]): T[B]
    }

// Monad [(3)][3][(4)][4]

    def flatMap[A,B]( func:A=>Container[B] ) = (a:Container[A]) => func( a.value )

or 

    def flatMap[A,B](func:A=>Container[B]) = (a:Container[A]) => map(func)(a).value

or 

    def flatMap[A,B](func:A=>MyBox[B]) = (a:MyBox[A]) => flatten(map(func)(a))
    def flatten[B](m:MyBox[MyBox[B]]):MyBox[B] = m.value

or [(5)][5]

    trait Monad[M[_]] extends Applicative[M] {
      def >>=[A,B](ma:M[A])(f:A=>M[B]):M[B] // or bind
    }

// Applicative [(3)][3]

    def apply[A,B] ( b:Container[A=>B] ) = (a:Container[A]) => new Container(b.value(a.value))

or [(5)][5]

    trait Applicative[T[_]] extends Functor[T] {
      def pure[A](a:A):T[A] // or identity
      def <*>[A,B](tf:T[A=>B])(ta:T[A]):T[B] // or apply
    }

or [(7)][7]

    val applicative: F[Y => Z] => F[Y] => F[Z] =
      f => a => for {
        ff <- f
        aa <- a
      } yield ff(aa)

`fib.zip(fib.tail)` can be rewritten as `zip <*> tail`

// Exponential functor [(8)][8]

    trait Exponential[F[_]] {
      def xmap[A, B](f: (A => B, B => A)): F[A] => F[B]
    }

Concrete use:
=============

Writer monad: [(4)][4]
-------------

    class LogBox[T](val value:T, val mesg:String="") {
        def map[B](f:T=>B) = new LogBox(f(value), mesg)
        def flatMap[B](f:T=>LogBox[B]) = flatten( map(f) )
        def flatten[B](m:LogBox[LogBox[B]]) = new LogBox( m.value.value, m.mesg+m.value.mesg+"\n")
    }
 

References:
===========

* [Functors, Monads, Applicatives – can be so simple][1]
* [Functors, Monads, Applicatives – different implementations][2]
* [Functors, Monads, Applicatives – playing with map (Functor)][3]
* [Functors, Monads, Applicatives – taking Monad apart][4]
* [Functors, Applicative Functors, and Monads aren't that scary][5]
* [Haskell Typeclassopedia][6]
* [Case study of fib.zip(fib.tail) by Tony Morris][7]
* [Functors and things using Scala][8]

[1]: http://thedet.wordpress.com/2012/04/28/functors-monads-applicatives-can-be-so-simple/ "Functors, Monads, Applicatives – can be so simple"

[2]: http://thedet.wordpress.com/2012/05/04/functors-monads-applicatives-different-implementations/ "Functors, Monads, Applicatives – different implementations"

[3]: http://thedet.wordpress.com/2012/05/20/functors-monads-applicatives-playing-with-map-functor/ "Functors, Monads, Applicatives – playing with map (Functor)"

[4]: http://thedet.wordpress.com/2013/01/21/functors-monads-applicatives-taking-monad-apart-draft/ "Functors, Monads, Applicatives – taking Monad apart"

[5]: http://gabrielsw.blogspot.de/2011/08/functors-applicative-functors-and.html "Functors, Applicative Functors, and Monads aren't that scary"

[6]: http://www.haskell.org/wikiupload/e/e9/Typeclassopedia.pdf "Haskell Typeclassopedia"

[7]: https://groups.google.com/forum/#!msg/scala-user/uh5w6N2eAHY/3Shf1295VpYJ "Case study of fib.zip(fib.tail) by Tony Morris"

[8]: http://blog.tmorris.net/posts/functors-and-things-using-scala/index.html "Functors and things using Scala"
