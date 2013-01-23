title: Functional programming

## Functional programming

Let's start with some definitions: [(1)][1][(5)][5]

**Functor:**
    map ( A=>B ) => ( C[A]=>C[B] )   or   map C[A] => (A=>B) => C[B]

**Monad:**
    flatMap ( A=>C[B] ) => ( C[A]=>C[B] )

**Applicative:**
    apply ( C[A=>B] ) => ( C[A]=>C[B] )

    class Container[T](val value:T)

Implementation details:
=======================

// Functor [(1)][1][(2)][2]

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

or

    trait Functor[T[_]] {
      def fmap[A,B](f:A=>B)(ta:T[A]):T[B]
    }

// Monad

    def flatMap[A,B]( func:A=>Container[B] ) = (a:Container[A]) => func( a.value )

or 

    def flatMap[A,B](func:A=>Container[B]) = (a:Container[A]) => map(func)(a).value

or 

    def flatMap[A,B](func:A=>MyBox[B]) = (a:MyBox[A]) => flatten(map(func)(a))
    def flatten[B](m:MyBox[MyBox[B]]):MyBox[B] = m.value

or 

    trait Monad[M[_]] extends Applicative[M] {
      def >>=[A,B](ma:M[A])(f:A=>M[B]):M[B] // or bind
    }

// Applicative

    def apply[A,B] ( b:Container[A=>B] ) = (a:Container[A]) => new Container(b.value(a.value))

or

    trait Applicative[T[_]] extends Functor[T] {
      def pure[A](a:A):T[A]
      def <*>[A,B](tf:T[A=>B])(ta:T[A]):T[B] // or apply
    }

or [(6)][6]

    val applicative: F[Y => Z] => F[Y] => F[Z] = // [6]
      f => a => for {
        ff <- f
        aa <- a
      } yield ff(aa)

`fib.zip(fib.tail)` can be rewritten as `zip <*> tail`

Concrete use:
=============

Writer monad: [(3)][3]
-------------

    class LogBox[T](val value:T, val mesg:String="") {
        def map[B](f:T=>B) = new LogBox(f(value), mesg)
        def flatMap[B](f:T=>LogBox[B]) = flatten( map(f) )
        def flatten[B](m:LogBox[LogBox[B]]) = new LogBox( m.value.value, m.mesg+m.value.mesg+"\n")
    }
 

References:
===========

[1]: http://thedet.wordpress.com/2012/04/28/functors-monads-applicatives-can-be-so-simple/

[2]: http://thedet.wordpress.com/2012/05/04/functors-monads-applicatives-different-implementations/

[3]: http://thedet.wordpress.com/2013/01/21/functors-monads-applicatives-taking-monad-apart-draft/

[4]: http://gabrielsw.blogspot.de/2011/08/functors-applicative-functors-and.html

[5]: http://www.haskell.org/wikiupload/e/e9/Typeclassopedia.pdf

[6]: https://groups.google.com/forum/#!msg/scala-user/uh5w6N2eAHY/3Shf1295VpYJ

