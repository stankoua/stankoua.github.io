---
title: "Les GADTs Par l'Exemple"
date: 2019-10-26T4:30:00+02:00
draft: false
description:
keywords:
  - GADT
  - GADTs
  - Generalized Algebraic Data Types
  - ADT
  - Algebraic Data Types
  - Functional Programming
  - Interface
  - Free Monad
---

Soyez les bienvenu·e·s! Cette session à le dessein de vous présenter
un outil de programmation très puissant. Alors que la plupart des
introductions sur le sujet commencent par une présentation de ses
fondements théoriques d'une manière très formelle, nous avons choisi
de vous le présenter à travers de courts exemples et des cas
d'utilisation concrets.

Cet atelier est composé de trois parties. La dernière présente trois
des cas d'utilisation des plus utiles. Ils forment les usages majeurs
en pratique. Mais ne vous y aventurez pas sans préparation! Cette partie
est la dernière pour une bonne raison: elle s'appuie massivement sur
les leçons des parties précédentes.
Commencez par  [Premier Contact](#premier-contact), elle vous exposera
, via les plus simples exemples, les idées clefs. Sont but est
d'ouvrir votre esprit à des manières d'utiliser les types et données
que vous n'avez vraisemblablement jamais soupçonnées.
Arpentez ensuite
[Cas d'Utilisation Simples et Pratiques: Relations entre Types](#easy-useful-use-cases-relations-on-types),
pour un premier défi devant un usage pratique.
Après cela seulement vous serez prêt pour 
[Cas d'Utilisation Plus Avancés](#more-advanced-use-cases).

Assurez vous de **lire [LISEZ-MOI](#readme)**, elle contient de précieuses
astuces pour faciliter votre parcours.

## Remerciements

Nous tenons à remercier [Laure Juglaret](http://www.laure-juglaret.fr/) pour
ses nombreuses relectures, ses précieuses remarques et corrections.

## LISEZ-MOI

Durant toute cette présentation, nous considèrerons que:

- `null` **n'existe pas!**
- **La réflexion au runtime n'existe pas!** (c.-à-d. `isInstanceOf`, `getClass`, etc)

Cette présentation considère que *ces fonctionnalités n'existent pas du tout!*.

**Leur utilisation n'amènera jamais à une réponse correcte aux questions plus bas.**.

Pour faire cette atelier vous devez disposez du nécessaire pour écrire, compiler et
exécuter rapidement du code  *Scala*. Le meilleur moyen est d'ouvrir une session 
interactive (*R.E.P.L.*). Si vous avez *Scala* d'installé sur votre système, vous 
pouvez facilement en démarrer une via la ligne de commande en exécutant le programme
`scala`:

```scala
system-command-line# scala
Welcome to Scala 2.13.1 (OpenJDK 64-Bit Server VM, Java 1.8.0_222).
Type in expressions for evaluation. Or try :help.

scala>
```

Pour rappel, dans une session interactive (*R.E.P.L.*),
la commande `:paste` permet de copier du code dans la session
et la commande `:reset` de repartir d'un environnement vierge.

Si vous n'avez pas *Scala* d'installé, **vous pouvez utiliser** le
site https://scastie.scala-lang.org/ .

## Échauffements

Cette section est un bref rappel de quelques définitions et propriétés sur
les types et les valeurs.

### *Valeurs* et *Types*?

Les **valeurs** sont les *données concrètes que vos programmes manipulent*
comme l'entier `5`, le booléen `true`, la chaîne `"Hello World!"`,
la fonction `(x: Double) => x / 7.5`, la liste `List(1,2,3)`, etc.
Il est souvent pratique de classer les valeurs en groupes. Ces groupes sont appelés
des **types**. Par exemple:

- `Int` est le groupe des valeurs entières, c.-à-d. les valeurs telles que `1`, `-7`, `19`, etc.
- `Boolean` est le groupe contenant exactement les valeurs
  `true` et `false` (ni plus, ni moins!).
- `String` est le groupe dont les valeurs sont `"Hello World!"`, `""`, `"J' ❤️ les GADTs"`, etc.
- `Double => Double` est le groupe dont les valeurs sont les fonctions prenant en argument
   n'importe quel `Double` et renvoyant également un double `Double`.

Pour indiquer que la valeur `v` appartient au type (c.-à-d. groupe of valeurs) `T`,
la notation est `v : T`. En *Scala*, tester si une valeur `v` appartient au type `T`
est très simple: il suffit de taper `v : T` dans la session interactive (*REPL*):

```scala
scala> 5 : Int
res7: Int = 5
```

Si *Scala* l'accepte, alors `v` appartient bien au type `T`. Si *Scala* râle,
ce n'est probablement pas le cas:

```scala
scala> 5 : String
       ^
       error: type mismatch;
        found   : Int(5)
        required: String
```

### Combien de types?

Créons maintenant quelques types et quelques unes de leurs valeurs (quand cela est possible!).

```scala
class UnType
```

- **Question 1:** Combien de types la ligne `class UnType` définit elle?

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

      Comme son nom le suggère, la ligne `class UnType` définit seulement un type, nommé `UnType`.
    </details>

Passons maintenant à:

```scala
class UnTypePourChaque[A]
```

- **Question 2:** Combien de types la ligne `class UnTypePourChaque[A]` définit elle?

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

      Comme son nom le suggère, chaque type concret `A` donne lieu à un type distinct `UnTypePourChaque[A]`.

      Par example, une liste d'entiers n'est ni une liste de booléens, ni une liste de châine de
      caractères, ni une lste de fonctions, ni ... En effet les types `List[Int]`, `List[Boolean]`,
      `List[Int => Int]`, etc sont tous des types distincts.

      la ligne `class UnTypePourChaque[A]` définit **un type distinct pour chaque type concret** `A`.
      Il y a une infinité de types concret `A`, donc une infinité de de types distincts `UnTypePourChaque[A]`.
    </details>

- **Question 3:** Donnez une valeur qui appartient à la fois aux types `UnTypePourChaque[Int]`
  et `UnTypePourChaque[Boolean]`.

    **Pour rappel, `null` n'existe pas!**

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    C'est en fait impossible. Chaque type concret `A` donne lieu
    à un type distinct `UnTypePourChaque[A]` qui n'a aucune valeur en commun
    avec les autres types de la forme `UnTypePourChaque[B]` avec `B ≠ A`.

    </details>

### Combien de valeurs?

En considérant le type suivant:

```scala
final abstract class PasDeValeurPourCeType
```

- **Question 1:** Donnez une valeur appartenant au type `PasDeValeurPourCeType`?
   Combien de valeurs appartiennent au type `PasDeValeurPourCeType`?

    <details>
      <summary>*Astuce (cliquer pour dévoiler)*</summary>
      - Qu'est ce qu'une classe `final`? En quoi est ce qu'elle diffère d'une classe normale (non finale)?
      - Qu'est ce qu'une classe `abstract`? En quoi est ce qu'elle diffère d'une classe concrète?
    </details>

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

      La classe `PasDeValeurPourCeType` est déclarée comme `abstract`. Cela signifie qu'
      il est interdit de créer des instances directes de cette classe:

      ```scala
      scala> new PasDeValeurPourCeType
             ^
             error: class PasDeValeurPourCeType is abstract; cannot be instantiated
      ```

      La seule manière de créer une instance d'une classe abstraite est de créer une
      une sous-classe concrète. Mais le mot clef `final` interdit la création de
      telles sous-classes:

      ```scala
      scala> class SousClasseConcrete extends PasDeValeurPourCeType
                                              ^
             error: illegal inheritance from final class PasDeValeurPourCeType
      ```

      Il n'existe aucun moyen de créer une instance pour `PasDeValeurPourCeType`.
    </details>


Prenons un autre exemple:

```scala
sealed trait ExactementUneValeur
case object LaSeuleValeur extends ExactementUneValeur
```

- **Question 2:** Donnez une valeur appartenant au type `ExactementUneValeur`?

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

      Par définition, `LaSeuleValeur` est une valeur du type `ExactementUneValeur`.
    </details>

- **Question 3:** Combien de valeurs appartiennent à `ExactementUneValeur`?

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

      Comme ci-dessus, `ExactementUneValeur`, étant un `trait`, est *abstrait*. Étant `sealed`,
      l'étendre en dehors de son fichier source est interdit.
      Donc `LaSeuleValeur` est la seule valeur du type `ExactementUneValeur`.
    </details>

## Premier Contact

Cette partie présente les idées clefs. Il y a en fait seulement
deux idées! Vous trouverez des exemples épurés illustrant chacune
de ces deux idées.

### Cas d'Utilisation: Preuve d'une propriété

Définissons un simple *sealed trait*:

```scala
sealed trait ATrait[A]
case object AValue extends ATrait[Char]
```

- **Question 1:** Donnez une valeur du type `ATrait[Char]`.

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

      Par dfinition, `AValue` est une valeur du type `ATrait[Char]`.
    </details>

- **Question 2:** Donnez une valeur du type `ATrait[Double]`.

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

      Il n'existe aucun moyen d'obtenir une instance du type `ATrait[Double]`.
      Il n'existe en fait aucun moyen d'obtenir une instance de `ATrait[B]` pour `B ≠ Char`
      parce que la seule valeur possible est `AValue` qui est de type `ATrait[Char]`.
    </details>

- **Question 3:** Que pouvez vous conclure sur le type `A` si vous avez une valeur
  `ev` de type `ATrait[A]` (c.-à-d. `ev: ATrait[A]`)?

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    La seule valeur possible est `AValue`, donc `ev == AValue`.
    De plus `AValue` est de type `ATrait[Char]` donc `A = Char`.

    </details>

- **Question 4:** Dans la session interactive (*REPL*), entrez le code suivant:

    ```scala
    def f[A](x: A, ev: ATrait[A]): Char =
      x
    ```

- **Question 5:** Essayez maintenant en utiliant un filtrage par motif (pattern matching) sur `ev: ATrait[A]`

    ```scala
    def f[A](x: A, ev: ATrait[A]): Char =
      ev match {
        case AValue => x
      }
    ```

    Le filtrage par motif (pattern-matching) est il exhaustif?

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

      Le filtrage par motif est exhaustif parce la seule et unique valeur possible
      pour `ev` est en fait `AValue`. De plus `AValue` est de type `ATrait[Char]` ce qui signifie 
      que `ev : ATrait[Char]` parce que `ev == AValue`. Donc `A = Char` et `x : Char`.
    </details>

- **Question 6:** Appelez `f` avec `x = 'w' : Char`.

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    ```scala
    scala> f[Char]('w', AValue)
    res0: Char = w
    ```
    </details>

- **Question 7:** Appelez `f` avec `x =  5.2 : Double`.

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    C'est impossible parce que cela demenderait de fournir une valeur
    `ev : ATrait[Double]` qui n'existe pas!

    ```scala
    scala> f[Double](5, AValue)
                        ^
              error: type mismatch;
                found   : AValue.type
                required: ATrait[Double]
    ```

    </details>

<details>
  <summary>**Remarque pour les personnes à l'aise en _Scala_** *(cliquer pour dévoiler)* </summary>

En utilisant toutes les chouettes fonctionnalités syntaxiques de *Scala*,
la version satisfaisante en production du code ci-dessus est:

```scala
sealed trait IsChar[A]
object IsChar {
  implicit case object Evidence extends IsChar[Char]

  def apply[A](implicit evidence: IsChar[A]): IsChar[A] =
    evidence
}

def f[A: IsChar](x: A): Char =
  IsChar[A] match {
    case IsChar.Evidence => x
  }
```

</details>

### Cas d'Utilisation: La seule chose que je sais, est qu'il existe

Que feriez vous si vous vouliez que votre application tienne un journal d’événements (c.-à-d. un *log*),
mais que vous vouliez être sur qu'elle ne dépende d'aucun détail d'implémentation de la méthode
de journalisation (c.-à-d. du *logger*) de telle manière que vous puissiez changer son implémentation
sans risquer de casser vôtre application?

En considérant le type suivant, `UnknownLogger`, des méthodes de journalisation:

```scala
sealed trait UnknownLogger
final case class LogWith[X](logs : X, appendMessage: (X, String) => X) extends UnknownLogger
```

La première méthode (c.-à-d. **logger*) que nous créons stocke les messages dans une `String`:

```scala
val loggerStr : UnknownLogger =
  LogWith[String]("", (logs: String, message: String) => logs ++ message)
```

La seconde méthode les stocke dans une `List[String]`:

```scala
val loggerList : UnknownLogger =
  LogWith[List[String]](Nil, (logs: List[String], message: String) => message :: logs)
```

La troisième méthode de journalisation imprime directement les messages sur la sortie standard:

```scala
val loggerStdout : UnknownLogger =
  LogWith[Unit]((), (logs: Unit, message: String) => println(message))
```

Notez que ces trois méthodes de journalisation ont toutes le même type (c.-à-d. `UnknownLogger`)
mais qu'elles stockent les messages en utilisant différents types `X` (`String`, `List[String]` et `Unit`).

- **Question 1:** Soit `v` une valeur de type `UnknownLogger`.
   Clairement `v` doit être une instance de la classe `LogWith[X]` pour un certain `X`.
   Que pouvez vous dire sur le type `X`? Pouvez vous deviner quel type concret est `X`?

    **Pour rappel, il est interdit d'utiliser la réflexion au runtime!** (c.-à-d. `isInstanceOf`, `getClass`, etc)

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    Nous ne savons presque rien sur `X`. La seule chose que nous avons est qu'il existe au
    moins une valeur (`v.logs`) de type `X`. À part cela, `X` peut être n'importe quel type.

    Ne pas savoir quel type concret est `X` est très utile pour garantir que le code qui utilisera
    `v : UnknownLogger` ne dépendra pas de la nature de `X`. Si ce code savait que `X` était
    `String` par exemple, il pourrait exécuter des opérations que nous voulons interdir comme inverser
    la liste, ne retenir que les *n* premiers caractères, etc. En cachant la nature de`X`, nous forçons
    notre application à ne pas dépendre du type concret derrière`X` mais de n'utiliser que la
    function fournie `v.appendMessage`.
    Ainsi changer l'implémentation réelle de la méthode de journalisation ne cassera aucun code.

    </details>

- **Question 2:** Écrivez la fonction `def log(message: String, v: UnknownLogger): UnknownLogger`
  qui utilise `v.appendMessage` pour ajouter le `message` au journal `v.logs`
  et retourne un nouvel `UnknownLogger` contenant le nouveau journal (c.-à-d. le nouveau `log`).

    Pour rappel, en *Scala*, le motif (c.-à-d. pattern) `case ac : AClass[t] =>`
    est possible dans les expressions de type `match/case` comme alternative au motif
    `case AClass(v) => `:

    ```scala
    final case class AClass[A](value : A)

    def f[A](v: AClass[A]): A =
      v match {
        case ac : AClass[t] =>
          // La variable `t` is une variable de type
          // Le type `t` est `A`
          val r : t = ac.value
          r
      }
    ```

    Son principal avantage est d'introduire la variable de type `t`.
    Les variables de type se comportent comme des variables de motif classiques (c.-à-d. pattern variables)
    à l'exception prés qu'elles représentent des types et non des valeurs.
    Avoir `t` sous la main nous permet d'aider le compilateur en donnant explicitement certains types
    (comme ci-dessus, expliciter que `r` est de type `t`).

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    ```scala
    def log(message: String, v: UnknownLogger): UnknownLogger =
      v match {
        case vx : LogWith[x] => LogWith[x](vx.appendMessage(vx.logs, message), vx.appendMessage)
      }
    ```

    </details>

- **Question 3:** Exécutez `log("Hello World", loggerStr)` et `log("Hello World", loggerList)`
  et `log("Hello World", loggerStdout)`

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    ```scala
    scala> log("Hello World", loggerStr)
    res0: UnknownLogger = LogWith(Hello World,$$Lambda$988/1455466014@421ead7e)

    scala> log("Hello World", loggerList)
    res1: UnknownLogger = LogWith(List(Hello World),$$Lambda$989/1705282731@655621fd)

    scala> log("Hello World", loggerStdout)
    Hello World
    res2: UnknownLogger = LogWith((),$$Lambda$990/1835105031@340c57e0)
    ```

    </details>

<details>
  <summary>**Remarque pour les personnes à l'aise en _Scala_** *(cliquer pour dévoiler)* </summary>

Encore une fois, en utilisant toutes les chouettes fonctionnalités syntaxiques de *Scala*,
la version satisfaisante en production du code ci-dessus est:

```scala
sealed trait UnknownLogger {
  type LogsType
  val logs : LogsType
  def appendMessage(presentLogs: LogsType, message: String): LogsType
}

object UnknownLogger {

  final case class LogWith[X](logs : X, appendMessage_ : (X, String) => X) extends UnknownLogger {
    type LogsType = X
    def appendMessage(presentLogs: LogsType, message: String): LogsType =
      appendMessage_(presentLogs, message)
  }

  def apply[X](logs: X, appendMessage: (X, String) => X): UnknownLogger =
    LogWith[X](logs, appendMessage)
}
```

</details>

### Conclusion Intermédiaire

Les *GADTs* ne sont en fait que ceci: de simples *sealed trait* avec quelques
*case object* (possiblement aucun) et quelques *final case class* (également possiblement aucune!).
Dans les parties suivantes, nous explorerons quelques un des cas d'utilisation
majeurs des *GADTs*.

## Cas d'utilisation simples et utiles: relations sur les types

Une faculté simple mais très utile des *GADTs* est l'expression de relations sur les types telles que:

- Le type `A` est-il égal au type `B`?
- Le type `A` est-il un sous-type de `B`?

Notez bien que, par définition, tout type `A` est sous-type de lui-même (c.-à-d. `A <: A`),
tout comme tout entier `x` est également inférieur ou égal à lui-même `x ≤ x`.

### Cas d'Utilisation: Témoin d'Égalité entre Types

```scala
sealed trait EqT[A,B]
final case class Evidence[X]() extends EqT[X,X]
```

- **Question 1:** Donnez une valeur de type `EqT[Int, Int]`

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    ```scala
    scala> Evidence[Int]() : EqT[Int, Int]
    res0: EqT[Int,Int] = Evidence()
    ```

    </details>

- **Question 2:** Donnez une valeur de type `EqT[String, Int]`

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

      La classe `Evidence` est l'unique sous-classe conctrète du trait `EqT` et il est
      impossible d'en créer une autre parce que `EqT` est `sealed`. Donc une valeur `v : EqT[A,B]`
      ne peut être qu'une instance de `Evidence[X]` pour un certain type `X`, qui elle-même est
      de type `EqT[X,X]`.
      Ainsi il n'y a aucun moyen d'obtenir une valeur de type `EqT[String, Int]`

    </details>

- **Question 3:** Soient `A` et `B` deux types (inconnus).
   Si je vous donne une valeur de type `EqT[A,B]`, que pouvez vous en déduire?

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

      Si je vous donne une valeur `v : EqT[A,B]`, alors vous savez que `v` est une instance
      de `Evidence[X]` pour un certain type `X` (inconnu) car la classe `Evidence`  est la seule
      et unique sous-classe concrète du `sealed trait` `EqT`. En fait, `Evidence[X]` est un sous-type de
      `EqT[X,X]`. Donc `v : EqT[X,X]`. Les types `EqT[A,B]` et `EqT[X,X]` n'ont aucune valeur en commun
      si `A ≠ X` ou `B ≠ X`, donc `A = X` et `B = X`. Et donc `A = B`. CQFD.

    </details>

<details>
  <summary>**Remarque pour les personnes à l'aise en _Scala_** *(cliquer pour dévoiler)* </summary>

En production, il est pratique de définir `EqT` de la manière suivante, qui est bien entendu équivalente:

```scala
sealed trait EqT[A,B]
object EqT {
  final case class Evidence[X]() extends EqT[X,X]

  implicit def evidence[X] : EqT[X,X] = Evidence[X]()

  def apply[A,B](implicit ev: EqT[A,B]): ev.type = ev
}
```

</details>

#### Passer d'un type égal à l'autre

Si `A` et `B` sont en fait le même type, alors `List[A]` est également le même
type que `List[B]`, `Option[A]` est également le même type que `Option[B]`,
etc. De manière générale, pour n'importe quel `F[_]`, `F[A]` est également le même type
que `F[B]`.

- **Question 4:** Écrivez la fonction `def coerce[F[_],A,B](eqT: EqT[A,B])(fa: F[A]): F[B]`.

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    ```scala
    def coerce[F[_],A,B](eqT: EqT[A,B])(fa: F[A]): F[B] =
      eqT match {
        case _ : Evidence[x] => fa
      }
    ```

    </details>

La bibliothèque standard de *Scala* définit déjà une classe, nommée `=:=[A,B]`
(en effet, son nom est bien `=:=`), représentant l'égalité entre types.
Je vous recommande vivement de jeter un œil
[à sa documentation (cliquez ici)](https://www.scala-lang.org/api/current/scala/$eq$colon$eq.html).
Fort heureusement, pour plus de lisibilité, *Scala* nous permet d'écrire `A =:= B` le type `=:=[A,B]`.

Étant donnné deux types `A` et `B`, avoir une instance (c.-à-d. objet)
de `A =:= B` prouve que `A` et `B` sont en réalité le même type,
tout comme pour `EqT[A,B]`.
Pour rappel, `A =:= B` n'est que du sucre syntaxique pour désigner le type `=:=[A,B]`.

Des instances de `A =:= B` peuvent êtres crées en utilisant la fonction
[`(<:<).refl[X]: X =:= X`
(cliquer pour voir la documentation)](https://www.scala-lang.org/api/current/scala/$less$colon$less$.html#refl[A]:A=:=A).
Le "symbole" `<:<` est en effet un nom d'objet valide.

- **Question 5:** En utilisant la fonction `coerce` ci-dessus,
  écrivez la fonction `def toScalaEq[A,B](eqT: EqT[A,B]): A =:= B`.

    <details>
      <summary>*Astuce (cliquer pour dévoiler)*</summary>

    ```scala
    def toScalaEq[A,B](eqT: EqT[A,B]): A =:= B = {
      /* Trouver une définition pour:
            - le constructeur de type `F`
            - la valeur `fa : F[A]`
      */
      type F[X] = ???
      val fa : F[A] = ???

      /* Telles que cet appel: */
      coerce[F,A,B](eqT)(fa) // soit de type `F[B]`
    }
    ```

    </details>


    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    ```scala
    def toScalaEq[A,B](eqT: EqT[A,B]): A =:= B = {
      type F[X] = A =:= X
      val fa: F[A] = (<:<).refl[A]
      coerce[F,A,B](eqT)(fa)
    }
    ```

    </details>

- **Question 6:** En utilisant la *méthode* `substituteCo[F[_]](ff: F[A]): F[B]` des
  objects de la classe `A =:= B`, dont la
  [documentation est ici](https://www.scala-lang.org/api/current/scala/$eq$colon$eq.html#substituteCo[F[_]](ff:F[From]):F[To]),
  écrivez la fonction `def fromScalaEq[A,B](scala: A =:= B): EqT[A,B]`.

    <details>
      <summary>*Astuce (cliquer pour dévoiler)*</summary>

    ```scala
    def fromScalaEq[A,B](scala: A =:= B): EqT[A,B] = {
      /* Trouver une définition pour:
            - le constructeur de type `F`
            - la valeur `fa : F[A]`
      */
      type F[X] = ???
      val fa : F[A] = ???

      /* Telles que cet appel: */
      scala.substituteCo[F](fa) // soit de type `F[B]`
    }
    ```

    </details>

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    ```scala
    def fromScalaEq[A,B](scala: A =:= B): EqT[A,B] = {
      type F[X] = EqT[A,X]
      val fa: F[A] = Evidence[A]()
      scala.substituteCo[F](fa)
    }
    ```

    </details>

### Cas d'Utilisation: Témoin de Sous-Typage

Dans cette section, nous voulons les types `SubTypeOf[A,B]` dont les valeurs prouvent que le type `A` est
un sous-type de `B` (c.-à-d. `A <: B`). Une classe similaire, *mais différente*, est déjà définit dans la
bibliothèque standard de *Scala*.
Il s'agit de la classe `<:<[A,B]`, qui est le plus souvent écrite `A <:< B`. Sa
[documentation est ici](https://www.scala-lang.org/api/current/scala/$less$colon$less.html).
Cette section étant dédiée à l'implémentation d'une variante de cette classe,
veuillez **ne pas utiliser** `<:<[A,B]` pour implémenter `SubTypeOf`.

- **Question 1:** En utilisant uniquement des *bornes supérieures* (c.-à-d. `A <: B`)
  ou *bornes inférieures* (c.-à-d. `A >: B`) et **aucune** *annotation de variance* (c.-à-d. `[+A]` et `[-A]`),
  créez le trait `SubTypeOf[A,B]` (et tout ce qui est nécessaire) tel que:

    > Il existe une valeur de type `SubType[A,B]` **si et seulement si**
    `A` est un sous-type de `B` (c.-à-d. `A <: B`).

    Pour rappel, par définition, un type `A` est un sous-type de lui-même (c.-à-d. `A <: A`).

    Pour rappel, n'utilisez pas la classe `<:<[A,B]`.

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    ```scala
    sealed trait SubTypeOf[A,B]
    final case class SubTypeEvidence[A <: B, B]() extends SubTypeOf[A,B]
    ```

    <details>
      <summary>**Remarque pour les personnes à l'aise en _Scala_** *(cliquer pour dévoiler)* </summary>

    En production, il est pratique de définir `SubTypeOf` de la manière équivalente suivante:

    ```scala
    sealed trait SubTypeOf[A,B]
    object SubTypeOf {
      final case class Evidence[A <: B, B]() extends SubTypeOf[A,B]

      implicit def evidence[A <: B, B]: SubTypeOf[A,B] = Evidence[A,B]()

      def apply[A,B](implicit ev: SubTypeOf[A,B]): ev.type = ev
    }
    ```

    </details>

    </details>

### Cas d'Utilisation: Éviter les messages d'erreur de *scalac* à propos des bornes non respectées

Dans cet exemple, nous voulons modéliser le régime alimentaire de certains animaux.
Commençons par définir le type `Food` (c.-à-d. nourriture) et quelques un de ces sous-types:

```scala
trait Food
class Vegetable extends Food
class Fruit extends Food
```

et maintenant la classe représentant les animaux mangeant de la nourriture de type `A`
(c.-à-d. `Vegetable`, `Fruit`, etc):

```scala
class AnimalEating[A <: Food]

val elephant : AnimalEating[Vegetable] =
  new AnimalEating[Vegetable]
```

Définissons une fonction comme il en existe tant en *Programmation Fonctionelle*
et passons lui `elephant` comme argument:

```scala
def dummy[F[_],A](fa: F[A]): String = "Ok!"
```

```scala
scala> dummy[List, Int](List(1,2,3))
res0: String = Ok!

scala> dummy[Option, Boolean](Some(true))
res1: String = Ok!

scala> dummy[AnimalEating, Vegetable](elephant)
            ^
       error: kinds of the type arguments (AnimalEating,Vegetable)
       do not conform to the expected kinds of the type parameters (type F,type A).

       AnimalEating's type parameters do not match type F's expected parameters:
       type A's bounds <: Food are stricter than
       type _'s declared bounds >: Nothing <: Any
```

- **Question 1:** Pourquoi *scalac* se plaint il?

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    La fonction `dummy` requiert que sont argument `F`, qui est un constructeur
    de type comme le sont `List`, `Option`, `Future`, etc, accepte n'importe quel type
    en argument afin qu'il soit toujours possible d'écrire `F[A]` pour n'importe quel type `A`.
    Hors `AnimalEating` impose que son argument soit un sous-type de `Food`.
    Donc `AnimalEating` ne peut être utilisé comme argument `F` de `dummy`.
    </details>

Le problème est que, en définissant `class AnimalEating[A <: Food]`,
nous avons imposé à `A` d'être un sous-type de `Food`. Donc *Scala*, tout comme *Java*,
nous interdit de donner à `AnimalEating`, en tant qu'argument `A`, autre chose
qu'un sous-type de `Food` (en incluant `Food` lui-même):

```scala
scala> type T1 = AnimalEating[Int]
                 ^
       error: type arguments [Int] do not conform
       to class AnimalEating's type parameter bounds [A <: Food]

scala> type T2 = AnimalEating[Food]
defined type alias T2
```

Nous sommes face à un dilemme: afin d'utiliser la fonction `dummy`, que nous tenons
beaucoup à utiliser parce c'est une fonction très utile, il nous faut supprimer la
contrainte `A <: Food` de la définition `class AnimalEating[A <: Food]`.
Mais nous tenons également au fait que les animaux ne mangent que de la nourriture (`Food`)
et pas des entiers, ni des booléens et encore moins des chaînes de caractères!

- **Question 2:** Comment pouvez vous adapter la définition de `AnimalEating` tel que:

  + Il soit possible d'appeler `dummy` avec comme argument `elephant`! Nous voulons:

    ```scala
    scala> dummy[AnimalEating, Vegetable](elephant)
    res0: String = Ok!
    ```

  + *Si* `A` **n'est pas un sous-type** de `Food` (`Food` lui-même inclus),
    *alors* il doit être **impossible** de créer une instance de `AnimalEating[A]`.

  + La classe `AnimalEating` **doit** rester une classe ouverte
    (c.-à-d. non `sealed` ou `final`)! Il doit toujours être possible pour n'importe qui,
    n'importe quand, de créer librement des sous-classes de `AnimalEating`.
    Bien évidemment, ces sous-classes doivent respecter les deux contraintes
    ci-dessus.

    <details>
      <summary>*Astuce (cliquer pour dévoiler)*</summary>

    En *Scala*, `Nothing` est un type ne contenant aucune valeur.
    Pouvez vous créer une valeur de type `(Nothing, Int)`? Pourquoi?

    </details>

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    Si, afin de créer une instance de `AnimalEating[A]`, nous forçons **chaque**
    méthode créant des valeurs à prendre un paramètre supplémentaire de type `SubTypeOf[A, Food]`,
    alors il sera uniquement possible de créer une instance de `AnimalEating[A]`
    quand `A` sera un sous-type de `Food`:

    ```scala
    class AnimalEating[A](ev : SubTypeOf[A, Food])

    val elephant : AnimalEating[Vegetable] =
      new AnimalEating[Vegetable](SubTypeEvidence[Vegetable, Food])
    ```

    Pour créer une valeur de type `AnimalEating[A]`, nous avons besoin d'appeler
    le constructeur d'`AnimalEating`. Pour appler ce constructeur,
    il nous faut fournir `ev : SubTypeOf[A, Food]`.

    Il nous est désormais possible d'appeler la fonction `dummy` sur `elephant`:

    ```scala
    scala> dummy[AnimalEating, Vegetable](elephant)
    res0: String = Ok!
    ```

    En pratique, en utilisant des implicites, le compilateur peut
    fournir de lui-même le paramètre `ev : SubTypeOf[A, Food]`.

    Notez qu'il est désormais possible d'écrire le type `AnimalEating[Int]`
    mais vous ne pourrez jamais créer une valeur de ce type.

    </details>

### Cas d'Utilisation: Fournir les bonnes données au bon diagramme

Ce cas d'utilisation traite des méthodes pour garantir, à la compilation,
que seulement les valeurs du bon type peuvent être données à une fonction donnée.
L'exemple choisi est celui de la conception d'une bibliothèque de graphiques.
Afin de simplifier l'exemple, nous considèrerons que notre bibliothèque n'implémente
que deux types de graphiques:
des [camemberts (c.-à-d. pie charts)](https://www.google.com/search?q=pie+chart&tbm=isch)
et des [graphiques dit XY (c.-à-d. XY charts)](https://www.google.com/search?q=xy+charts&tbm=isch).
Cela s'écrit en *Scala* via l'énumération:

```scala
sealed trait ChartType
case object PieChart extends ChartType
case object XYChart extends ChartType
```

Bien évidemment les camemberts (*Pie*) et graphiques *XY* s'appuient sur des jeux de données de
nature différente. Encore une fois, pour simplifier, nous considérerons que les deux types
de données sont `PieData` pour les camemberts et `XYData` pour les graphiques *XY*:

```scala
class PieData
class XYData
```

Un camembert (`PieChart`) n'affiche **que** des données `PieData`,
alors qu'un graphique *XY* (`XYChart`) n'affiche **que** des données `XYData`.
Voici, grandement simplifiée, la fonction d'affichage `draw`:

```scala
def draw[A](chartType: ChartType)(data: A): Unit =
  chartType match {
    case PieChart =>
      val pieData = data.asInstanceOf[PieData]
      // Faire des trucs pour tracer les données pieData
      ()
    case XYChart =>
      val xyData = data.asInstanceOf[XYData]
      // Faire des trucs pour tracer les données xyData
      ()
  }
```

Cette fonction repose sur l'hypothèse que l'utilisateur·rice n'appellera la
fonction `draw` que sur le bon type de données.
Quand `chartType` vaut `PieChart`, la fonction présuppose, via
`data.asInstanceOf[PieData]` que `data` est en fait du type `PieData`.
Et quand `chartType` vaut `XYChart`, elle présuppose que `data` est en fait
de type `XYData`.

Le problème est que ces suppositions reposent sur l'idée que les utilisateurs·rices et/ou
développeurs·euses s'assureront toujours que ces conditions soient bien remplies.
Mais **rien** n'empêche quelqu'un·e d'appeler `draw` sur un camembert (`PieChart`)
avec des données de type `XYData` (ou le contraire),
faisant planter le système misérablement en production!

```scala
scala> draw(PieChart)(new XYData)
java.lang.ClassCastException: XYData cannot be cast to PieData
  at .draw(<pastie>:11)
  ... 28 elided
```

En tant que développeurs·euses, nous savons que les erreurs, ça arrive!
Nous voulons un moyen d'empêcher ces bogues ennuyeux de survenir en production!
Nous voulons imposer, à la compilation, que seulement deux scenarii soit possibles:

- Quand `draw` est appelée avec `chartType == PieChart`:
  l'argument `data` doit être de type `PieData`
- Quand `draw` est appelée avec `chartType == XYChart`:
  l'argument `data` doit être de type `XYData`.

Pour rappel, ces deux contraintes doivent être vérifiées à la compilation!

- **Question 1:** Adaptez la définition de `ChartType`, `PieChart`, `XYChart` et `draw` tel que:

    + Tout scenario différent des deux ci-dessus fera échouer la compilation sur une erreur de type.

    + `ChartType` doit toujours être un `sealed trait`. Mais il est autorisé à prendre des *paramètres de type* (c.-à-d. *generics*).

    + `PieChart` et `XYChar` doivent toujours être des `case object`
      et ils doivent toujours étendre `ChartType`.

    + Les déclarations de `ChartType`, `PieChart` et `XYChar` ne doivent **pas** avoir **de corps** du tout
    (c.-à-d. il ne doit pas y avoir d'accolades `{ ... }` dans leurs déclarations);

    <details>
      <summary>*Astuce (cliquer pour dévoiler)*</summary>

    Le code ressemble à ceci:

    ```scala
    sealed trait ChartType[/*METTRE LES GENERICS ICI*/]
    case object PieChart extends ChartType[/*Il y a quelque chose à écrire ici*/]
    case object XYChart extends ChartType[/*Il y a quelque chose à écrire ici aussi*/]

    def draw[A](chartType: ChartType[/*Ecrire quelque chose ici*/])(data: A): Unit =
     chartType match {
        case PieChart =>
          val pieData : PieData = data
          ()
        case XYChart =>
          val xyData: XYData = data
          ()
        }
    ```

    </details>

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    ```scala
    sealed trait ChartType[A]
    case object PieChart extends ChartType[PieData]
    case object XYChart extends ChartType[XYData]

    def draw[A](chartType: ChartType[A])(data: A): Unit =
     chartType match {
        case PieChart =>
          val pieData : PieData = data
          ()
        case XYChart =>
          val xyData: XYData = data
          ()
        }
    ```

    </details>

Vous pouvez maintenant dormir sur vos deux oreilles avec l'assurance que
votre code en production ne plantera pas à cause d'une entrée non conforme
à cet endroit 😉

## Cas d'Utilisation Plus Avancés

Maintenant que vous avez vu ce que sont les *GADTs* et comment les utiliser dans
la vie de tous les jours, vous êtes prêt·e pour les cas d'utilisations plus conséquents
ci-dessous.
Il y en a trois. Chacun illustre une manière différente d'utiliser la puissance des *GADTs*.
Le [premier](#use-case-effects) traite de l'expression d'éffets,
ce qui est très largement utilisé dans chaque *IO monad* populaire ou effets algébriques.
Ne vous inquiétez pas de ne pas savoir ce que ces derniers sont, cette section l'expliquera.
Le [second](#use-case-ensuring-types-are-supported-by-the-database)
s'attache à montrer comment garantir des propriétés dans le système de types.
Ce point est illustré à travers l'exemple de l’accommodation des techniques issues
de la programmation fonctionnelle au contraintes issues des bases de données.
Le [troisième](#use-case-simplifying-implicits) offre une manière plus simple
de travailler avec des implicites.


### Cas d'Utilisation: Les Effets!

Ce qui est appelé un effet est parfois juste une interface déclarant quelques fonctions dépourvues
d'implémentation. Par exemple nous pouvons définir le `trait` ci-dessous. Notez qu'*aucune* de ses fonctions
ne fournit une implémentation.

```scala
trait ExampleEffectSig {
  def echo[A](value: A): A
  def randomInt : Int
  def ignore[A](value: A): Unit
}
```

Les implémentations de ces interfaces sont données ailleurs, et il peut en avoir beaucoup!
Cela est utile quand il est désirable de changer facilement d'implémentation:

```scala
object ExampleEffectImpl extends ExampleEffectSig {
  def echo[A](value: A): A = value
  def randomInt : Int = scala.util.Random.nextInt()
  def ignore[A](value: A): Unit = ()
}
```

Une manière équivalente de définir `ExampleEffectSig` est via un `sealed trait`
muni de quelques `final case class` (peut être aucune!) et/ou quelques `case object` (peut être aucun!):

```scala
sealed trait ExampleEffect[A]
final case class  Echo[A](value: A) extends ExampleEffect[A]
final case object RandomInt extends ExampleEffect[Int]
final case class  Ignore[A](value: A) extends ExampleEffect[Unit]
```

De nouveau, nous avons une déclaration ne fournissant aucune implémentation!
De nouveau, ses implémentations peuvent être fournies ailleurs et il peut en avoir beaucoup:

```scala
def runExampleEffect[A](effect: ExampleEffect[A]): A =
  effect match {
    case Echo(value) => value
    case RandomInt   => scala.util.Random.nextInt()
    case Ignore(_)   => ()
  }
```

Prenons un effet plus réaliste ainsi qu'une implémentation possible:

```scala
trait EffectSig {
  def currentTimeMillis: Long
  def printLn(msg: String): Unit
  def mesure[X,A](fun: X => A, arg: X): A
}

object EffectImpl extends EffectSig {
  def currentTimeMillis: Long =
    System.currentTimeMillis()

  def printLn(msg: String): Unit =
    println(msg)

  def mesure[X,A](fun: X => A, arg: X): A = {
    val t0 = System.currentTimeMillis()
    val r  = fun(arg)
    val t1 = System.currentTimeMillis()
    println(s"Took ${t1 - t0} milli-seconds")
    r
  }
}
```

- **Question 1:** Tout comme `ExampleEffect` est l'équivalent de `ExampleEffectSig`
  via la définition d'un `sealed trait` muni de quelques `final case class` et
  `case object`, écrivez l'équivalent de `EffectSig` de la même manière.
  Appelez cet trait `Effect`.

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    ```scala
    sealed trait Effect[A]
    final case object CurrentTimeMillis extends Effect[Long]
    final case class  PrintLn(msg: String) extends Effect[Unit]
    final case class  Mesure[X,A](fun: X => A, arg: X) extends Effect[A]
    ```

    </details>

- **Question 2:** Écrivez la fonction `def run[A](effect: Effect[A]): A` qui reproduit l'implémentation de
  `EffectImpl` tout comme `runExampleEffect` reproduit celle de `ExampleEffectImpl`.

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    ```scala
    def run[A](effect: Effect[A]): A =
      effect match {
        case CurrentTimeMillis =>
          System.currentTimeMillis()

        case PrintLn(msg) =>
          println(msg)

        case Mesure(fun, arg) =>
          val t0 = System.currentTimeMillis()
          val r  = fun(arg)
          val t1 = System.currentTimeMillis()
          println(s"Took ${t1 - t0} milli-seconds")
          r
      }
    ```

    </details>

Le type `Effect[A]` déclare des effets intéressants (`CurrentTimeMillis`,
`PrintLn` et `Mesure`) mais pour être réellement utile, il doit être possible
de chaîner ces effets! Pour ce faire, nous voulons pouvoir disposer des deux fonctions suivantes:

- `def pure[A](value: A): Effect[A]`
- `def flatMap[X,A](fx: Effect[X], f: X => Effect[A]): Effect[A]`

De nouveau, nous ne nous intéressons pas à leurs implémentations. Tout ce que nous
voulons pour le moment est déclarer ces deux opérations de la même manière que
nous avons déclaré `CurrentTimeMillis`, `PrintLn` et `Mesure`.

- **Question 3:** Ajoutez deux *final case classes*, `Pure` et `FlatMap`,
  à `Effect[A]` déclarant ces deux opérations.

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    ```scala
    sealed trait Effect[A]
    final case object CurrentTimeMillis extends Effect[Long]
    final case class  PrintLn(msg: String) extends Effect[Unit]
    final case class  Mesure[X,A](fun: X => A, arg: X) extends Effect[A]
    final case class  Pure[A](value: A) extends Effect[A]
    final case class  FlatMap[X,A](fx: Effect[X], f: X => Effect[A]) extends Effect[A]
    ```

    </details>

- **Question 4:** Adaptez la fonction `run` pour gérer ces deux nouveaux cas.

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    ```scala
    def run[A](effect: Effect[A]): A =
      effect match {
        case CurrentTimeMillis =>
          System.currentTimeMillis()

        case PrintLn(msg) =>
          println(msg)

        case Mesure(fun, arg) =>
          val t0 = System.currentTimeMillis()
          val r  = fun(arg)
          val t1 = System.currentTimeMillis()
          println(s"Took ${t1 - t0} milli-seconds")
          r

        case Pure(a) =>
          a

        case FlatMap(fx, f) =>
          val x  = run(fx)
          val fa : Effect[A] = f(x)
          run(fa)
      }
    ```

    </details>

- **Question 5:** Ajoutez les deux méthodes suivantes au trait `Effect[A]` pour obtenir:

    ```scala
    sealed trait Effect[A] {
      final def flatMap[B](f: A => Effect[B]): Effect[B] = FlatMap(this, f)
      final def map[B](f: A => B): Effect[B] = flatMap[B]((a:A) => Pure(f(a)))
    }
    ```

    Et exécutez le code suivant pour voir s'il fonctionne:

    ```scala
    val effect1: Effect[Unit] =
      for {
        t0 <- CurrentTimeMillis
        _  <- PrintLn(s"The current time is $t0")
      } yield ()

    run(effect1)
    ```

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    ```scala
    sealed trait Effect[A] {
      final def flatMap[B](f: A => Effect[B]): Effect[B] = FlatMap(this, f)
      final def map[B](f: A => B): Effect[B] = flatMap[B]((a:A) => Pure(f(a)))
    }
    final case object CurrentTimeMillis extends Effect[Long]
    final case class  PrintLn(msg: String) extends Effect[Unit]
    final case class  Mesure[X,A](fun: X => A, arg: X) extends Effect[A]
    final case class  Pure[A](value: A) extends Effect[A]
    final case class  FlatMap[X,A](fx: Effect[X], f: X => Effect[A]) extends Effect[A]

    def run[A](effect: Effect[A]): A =
      effect match {
        case CurrentTimeMillis =>
          System.currentTimeMillis()

        case PrintLn(msg) =>
          println(msg)

        case Mesure(fun, arg) =>
          val t0 = System.currentTimeMillis()
          val r  = fun(arg)
          val t1 = System.currentTimeMillis()
          println(s"Took ${t1 - t0} milli-seconds")
          r

        case Pure(a) =>
          a

        case FlatMap(fx, f) =>
          val x  = run(fx)
          val fa : Effect[A] = f(x)
          run(fa)
      }

    val effect1: Effect[Unit] =
      for {
        t0 <- CurrentTimeMillis
        _  <- PrintLn(s"The current time is $t0")
      } yield ()
    ```

    When we run `run(effect1)`:

    ```scala
    scala> run(effect1)
    The current time is 1569773175010
    ```

    </details>

Félicitations! Vous venez d'écrire votre première monade *IO*! Il y
a de nombreux noms scientifiques au `sealed trait Effect[A]`:
vous pouvez l'appeler un *effet algébrique*, une *monade libre*, une *IO*, etc.
Mais au bout du compte, ce n'est qu'un simple et banal `sealed trait` pour lequel
nous avons défini quelques `final case class` et `case object` afin de
représenter les fonctions dont nous voulions disposer sans fournir
leurs implémentations (`CurrentTimeMillis`, `PrintLn`, `Mesure`,
`Pure` et `FlatMap`). Vous pouvez les appeler des *méthodes virtuelles* si vous voulez.
**Ce qui importe réellement est d'avoir isolé la définition de ces fonctions de leurs implémentations.**
Rappelez vous qu'un `trait` est juste une interface après tout.

### Cas d'Utilisation: S'assurer que les types sont pris en charge par la Base De Données.

Les bases de données sont formidables. Nous pouvons y stocker des tables, des documents,
des paires clef/valeur, des graphes, etc.
Mais, pour n'importe quelle base de données, il y a malheureusement seulement un nombre limité
de types pris en charge.
Prenez la base de données que vous voulez, je suis sur de pouvoir trouver des types qu'elle
ne prend pas en charge.

Dans cette section, nous allons nous intéresser au cas des structures des données
et du code qui ne marche pas pour tout les types, mais seulement certains! Ce cas d'usage
ne se limite pas aux bases de données mais concerne chaque *interface de programmation* qui ne
supporte qu'un nombre limité de types (la vaste majorité des *interfaces de programmation*).
Comment s'assurer du respect de ces contraintes? Comment adapter les techniques que nous aimons
afin qu'elles travaille sous ces contraintes? Voilà ce dont il s'agit dans cette section.

Nous considérerons une base de données fictive qui ne prend en charge **que** les types suivants:

1. `String`
2. `Double`
3. `(A,B)` où `A` et `B` sont également des types pris en charge par la base de données.

Cela signifie que les valeurs stockées dans la base de données (dans des tables, des paires clef/valeur,
etc) **doivent** respecter les règles ci-dessus. Elle peut stocker `"Hello World"` parce que c'est une
`String`, qui est est un type pris en charge par la base de données en vertu de la règle *1*.
Pour les mêmes raisons, elle peut stocker `5.2` parce que c'est un `Double`,
mais elle ne peut **pas** stocker l'entier `5` parce que c'est un`Int`.
Elle peut stocker `("Hello World", 5.2)` grâce à la règle *3* ainsi que
`(("Hello World", 5.2) , 8.9)`, de nouveau grâce à la règle *3*.

- **Question 1:** Définissez le type `DBType[A]` tel que:

    > Il existe une valeur de type `DBType[A]` **si et seulement si** `A`
    est un type pris en charge par la base de données.

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    La version simple est:

    ```scala
    sealed trait DBType[A]
    final case object DBString extends DBType[String]
    final case object DBDouble extends DBType[Double]
    final case class  DBPair[A,B](first: DBType[A], second: DBType[B]) extends DBType[(A,B)]
    ```

    <details>
    <summary>**Remarque pour les personnes à l'aise en _Scala_** *(cliquer pour dévoiler)* </summary>

    En utilisant toutes les chouettes fonctionnalités syntaxiques de *Scala*,
    la version satisfaisante en production du code ci-dessus est:

    ```scala
    sealed trait DBType[A]
    object DBType {
      final case object DBString extends DBType[String]
      final case object DBDouble extends DBType[Double]
      final case class DBPair[A,B](first: DBType[A], second: DBType[B]) extends DBType[(A,B)]

      implicit val dbString : DBType[String] =
        DBString

      implicit val dbDouble : DBType[Double] =
        DBDouble

      implicit def dbPair[A,B](implicit first: DBType[A], second: DBType[B]): DBType[(A,B)] =
        DBPair(first, second)

      def apply[A](implicit ev: DBType[A]): ev.type = ev
    }
    ```

    </details>

    </details>

En utilisant `DBType`, nous pouvons coupler une valeur de type `A`
avec une valeur de type `DBType[A]`, fournissant ainsi la preuve que
le type `A` est pris en charge par la base de données:

```scala
final case class DBValue[A](value: A)(implicit val dbType: DBType[A])
```

Notez que le paramètre `dbType` n'a nullement besoin d'être implicite!
Ce qui compte est que pour créer une valeur de type `DBValue[A]`,
nous devions fournir une valeur de type `DBType[A]`
ce qui force `A` à être un type pris en charge par la base de données.

Un foncteur est, de manière informelle et approximative, un constructeur de type`F`,
comme `List`, `Option`, `DBValue`, etc,
pour lequel il est possible de fournir une instance du trait:

```scala
trait Functor[F[_]] {
  def map[A,B](fa: F[A])(f: A => B): F[B]
}
```

où `map(fa)(f)` applique la fonction `f` à chaque valeur de type `A` contenue dans `fa`. Par exemple:

```scala
implicit object OptionFunctor extends Functor[Option] {
  def map[A,B](fa: Option[A])(f: A => B): Option[B] =
    fa match {
      case Some(a) => Some(f(a))
      case None => None
    }
}
```

- **Question 2:** Écrivez une instance de `Functor[DBValue]`.

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    C'est en fait impossible! Si nous tentions de compiler le code suivant:

    ```scala
    object DBValueFunctor extends Functor[DBValue] {
      def map[A,B](fa: DBValue[A])(f: A => B): DBValue[B] =
        DBValue[B](f(fa.value))
    }
    ```

    *Scala* râlerait: `could not find implicit value for parameter dbType: DBType[B]`. En effet, les booléans
    ne sont pas un type pris en charge par la base de données:
    ils ne sont ni des chaînes de caractères, ni des nombres flottants, ni des pairs de types pris en charge.

    Supposons que nous puissions définir une instance de `Funcor` pour `DBValue`
    (c.-à-d. que nous puissions définir une fonction `map` pour `DBValue`), alors nous pourrions écrire:

    ```scala
    val dbValueString  : DBValue[String]  = DBValue("A")(DBString)
    val dbValueBoolean : DBValue[Boolean] = dbValueString.map(_ => true)
    val dbTypeBoooean  : DBType[Boolean]  = dbValueBoolean.dbType
    ```

    Nous obtiendrions une valeur (`dbTypeBoooean`) de type `DBType[Boolean]`
    ce qui signifirait que le type `Boolean` est pris en charge par la base de donnée.
    Mais il ne l'est pas! Par définition:

    > Il existe une valeur de type `DBType[A]` **si et seulement si** `A`
    est un type pris en charge par la base de donnée.

    Donc il est impossible d'obtenir une valeur de type `DBType[Boolean]`
    et donc il est impossible d'écrire une fonction `map` pout `DBValue`.
    Ainsi il n'y a aucun moyen de définir une instance de `Functor` pour `DBValue`. CQDF.

    </details>

Un *Foncteur Généralisé* est très similaire à un `Functor` classique, à la différence près
que la fonction `map` ne doit pas obligatoirement être applicable à n'importe quels
types `A` et `B` mais peut n'être applicable qu'à certains types `A` et `B` particuliers:

```scala
trait GenFunctor[P[_],F[_]] {
  def map[A,B](fa: F[A])(f: A => B)(implicit evA: P[A], evB: P[B]): F[B]
}
```

Par exemple, `Set` (plus précisément `TreeSet`) n'est pas un foncteur!
En effet il n'y a aucun moyen d'écrire une fonction `map` qui fonctionne
pour n'importe quel type `B` (parce qu'il est nécessaire d'avoir une relation d'ordre sur `B`).
Mais si l'on restreint `map` aux seuls types `B` disposant d'une relation d'ordre, alors il devient
possible d'écrire:

```scala
import scala.collection.immutable._
object TreeSetFunctor extends GenFunctor[Ordering, TreeSet] {
  def map[A,B](fa: TreeSet[A])(f: A => B)(implicit evA: Ordering[A], evB: Ordering[B]): TreeSet[B] =
    TreeSet.empty[B](evB) ++ fa.toSeq.map(f)
}
```

- **Question 3:** Écrivez une instance de `GenFunctor[DBType, DBValue]`r

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    ```scala
    object DBValueGenFunctor extends GenFunctor[DBType, DBValue] {
      def map[A,B](fa: DBValue[A])(f: A => B)(implicit evA: DBType[A], evB: DBType[B]): DBValue[B] =
        DBValue[B](f(fa.value))(evB)
    }
    ```

    </details>

Ce que nous avons fait ici avec `Functor` peut être fait avec de nombreuses structures de données et
techniques de programmation. Il est souvent possible de restreindre la plage des types sur lesquels
la structure de donnée ou la classe de types (*type class*) peut opérer en ajoutant un paramètre supplémentaire
comme `ev : DBType[A]` aux constructeurs et méthodes.

### Cas d'Utilisation: Simplifier les Implicites

Ce cas d'utilisation est l'un des plus intéressants, mais malheureusement, pas l'un des plus simples.
Il montre comment il est possible d'utiliser les *GADTs* pour simplifier la création de valeurs implicites.
Nous prendrons comme celui des listes de valeurs dont les éléments peuvent être de types différents.
Ces listes sont appelées *listes hétérogènes*. Elles sont généralement définies en *Scala* presque comme
les listes classiques:

```scala
final case class HNil() // La liste vide
final case class HCons[Head,Tail](head: Head, tail: Tail) // L'operation: `head :: tail`

val empty : HNil =
  HNil()

val oneTrueToto : HCons[Int, HCons[Boolean, HCons[String, HNil]]] =
  HCons(1, HCons(true, HCons("toto", HNil())))

val falseTrueFive: HCons[Boolean, HCons[Boolean, HCons[Int, HNil]]] =
  HCons(false, HCons(true, HCons(5, HNil())))
```

Comme vous pouvez le voir, il n'y a rien de vraiment spécial à propos de ces listes.
Nous voulons définir des relations d'ordre sur les listes hétérogènes.
Une relation d'ordre est une façon de comparer deux valeurs (**du même type!**):
elles peuvent êtres égales ou l'une peut être strictement plus petite que l'autre.
Ceci peut se définir en *Scala* via le trait `Order`:

```scala
trait Order[A] {
  // vrai si et seulement si a1 < a2
  def lesserThan(a1: A, a2: A): Boolean

  /* a1 et a2 sont égales si et seulement si
     aucune d'entre elles n'est strictement plus petite que l'autre
  */
  final def areEqual(a1: A, a2: A): Boolean = !lesserThan(a1, a2) && !lesserThan(a2, a1)

  // a1 > a2 si et seulement si a2 < a1
  final def greaterThan(a1: A, a2: A): Boolean = lesserThan(a2, a1)

  final def lesserThanOrEqual(a1: A, a2: A): Boolean = !lesserThan(a2, a1)

  final def greaterThanOrEqual(a1: A, a2: A): Boolean = !lesserThan(a1, a2)
}

object Order {
  def apply[A](implicit ev: Order[A]): ev.type = ev

  def make[A](lg_ : (A,A) => Boolean): Order[A] =
    new Order[A] {
      def lesserThan(a1: A, a2: A): Boolean = lg_(a1,a2)
    }
}

implicit val orderInt    = Order.make[Int](_ < _)
implicit val orderString = Order.make[String](_ < _)
```

Pour rappel, nous ne comparerons que des types de *même type*:

- Les listes de type `HNil` seront uniquement comparées aux listes de type `HNil`.
- Les listes de type `HCons[H,T]` seront uniquement comparées aux listes de type `HCons[H,T]`.

Comparer des listes de type `HNil` est trivial parce qu'il n'y a qu'une seule et unique valeur
de type `HNil` (la liste vide `HNil()`) mais il existe de nombreuses façon de comparer des listes
de type `HCons[H,T]`.
Voici deux relations d'ordre possibles (il en existe de nombreuses autres!):

- L'*ordre lexicographique* (c.-à-d. l'ordre du dictionnaire: de la gauche vers la droite)

    > `HCons(h1,t1) < HCons(h2,t2)` **si et seulement si**
    `h1 < h2` *ou* (`h1 == h2` *et* `t1 < t2` *par l'ordre lexicographique*).

    ```scala
    sealed trait Lex[A] {
      val order : Order[A]
    }

    object Lex {
      def apply[A](implicit ev: Lex[A]): ev.type = ev

      implicit val lexHNil: Lex[HNil] =
        new Lex[HNil] {
          val order = Order.make[HNil]((_,_) => false)
        }

      implicit def lexHCons[Head,Tail](implicit
          orderHead: Order[Head],
          lexTail: Lex[Tail]
        ): Lex[HCons[Head, Tail]] =
        new Lex[HCons[Head, Tail]] {
          val orderTail: Order[Tail] = lexTail.order

          val order = Order.make[HCons[Head, Tail]] {
            case (HCons(h1,t1), HCons(h2,t2)) =>
              orderHead.lesserThan(h1,h2) || (orderHead.areEqual(h1,h2) && orderTail.lesserThan(t1,t2))
          }
        }
    }
    ```

- L'*ordre lexicographique inversé* qui est la version à l'envers de l'ordre lexicographique
  (c.-à-d. de droite à gauche)

    > `HCons(h1,t1) < HCons(h2,t2)` **si et seulement si**
    (`t1 < t2` *par ordre lexicographique inversé*) *ou* (`t1 == t2` *and* `h1 < h2`).

    ```scala
    sealed trait RevLex[A] {
      val order : Order[A]
    }

    object RevLex {
      def apply[A](implicit ev: RevLex[A]): ev.type = ev

      implicit val revLexHNil: RevLex[HNil] =
        new RevLex[HNil] {
          val order = Order.make[HNil]((_,_) => false)
        }

      implicit def revLexHCons[Head,Tail](implicit
          orderHead: Order[Head],
          revLexTail: RevLex[Tail]
        ): RevLex[HCons[Head, Tail]] =
        new RevLex[HCons[Head, Tail]] {
          val orderTail: Order[Tail] = revLexTail.order

          val order = Order.make[HCons[Head, Tail]] {
            case (HCons(h1,t1), HCons(h2,t2)) =>
              orderTail.lesserThan(t1,t2) || (orderTail.areEqual(t1,t2) && orderHead.lesserThan(h1,h2))
          }
        }
    }
    ```

Comme dit plus haut, il est possible de définir d'avantages de relation d'ordre:

- **Question 1:** L'ordre `Alternate` est défini par:

    > `HCons(h1,t1) < HCons(h2,t2)` **si et seulement si**
    `h1 < h2` *ou* (`h1 == h2` *et* `t1 > t2` *par ordre* `Alternate`).

    En suivant la méthoe employée pour `Lex` and `RevLex`,
    implémentez l'ordre `Alternate`.

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    ```scala
    sealed trait Alternate[A] {
      val order : Order[A]
    }

    object Alternate {
      def apply[A](implicit ev: Alternate[A]): ev.type = ev

      implicit val alternateHNil: Alternate[HNil] =
        new Alternate[HNil] {
          val order = Order.make[HNil]((_,_) => false)
        }

      implicit def alternateHCons[Head,Tail](implicit
          orderHead: Order[Head],
          alternateTail: Alternate[Tail]
        ): Alternate[HCons[Head, Tail]] =
        new Alternate[HCons[Head, Tail]] {
          val orderTail: Order[Tail] = alternateTail.order

          val order = Order.make[HCons[Head, Tail]] {
            case (HCons(h1,t1), HCons(h2,t2)) =>
              orderHead.lesserThan(h1,h2) || (orderHead.areEqual(h1,h2) && orderTail.greaterThan(t1,t2))
          }
        }
    }
    ```

    </details>

Il existe de nombreuses manières de définir une relation d'ordre valide sur les listes hétérogènes!
Créer une classe de type (*type class*) comme `Lex`, `RevLex` et `Alternate` pour chaque relation
d'ordre voulues est fatiguant et propice aux erreurs. Nous pouvons faire bien mieux ...
avec un *GADT* 😉

```scala
sealed trait HListOrder[A]
object HListOrder {
  final case object HNilOrder extends HListOrder[HNil]

  final case class HConsOrder[Head,Tail](
      orderHead: Order[Head],
      hlistOrderTail: HListOrder[Tail]
    ) extends HListOrder[HCons[Head,Tail]]

  // Définitions des Implicites

  implicit val hnilOrder : HListOrder[HNil] =
    HNilOrder

  implicit def hconsOrder[Head,Tail](implicit
      orderHead: Order[Head],
      hlistOrderTail: HListOrder[Tail]
    ): HListOrder[HCons[Head,Tail]] =
    HConsOrder(orderHead, hlistOrderTail)

  def apply[A](implicit ev: HListOrder[A]): ev.type = ev
}
```

Il est à notez que la définition de ces implicites est du pur boilerplate. Leur seule
raison d'être est de passer leurs arguments au constructeur correspondant
(c.-à-d. `final case class` ou `case object`):
`hnilOrder` à `HListOrder` (O arguments) et `hconsOrder` à `HConsOrder` (2 arguments).

- **Question 2:** Écrivez une fonction `def lex[A](implicit v : HListOrder[A]): Order[A]`
  qui retourne l'ordre lexicographique à partir d'une valeur de type `HListOrder[A]`.

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    ```scala
    def lex[A](implicit v : HListOrder[A]): Order[A] =
      v match {
        case HListOrder.HNilOrder =>
          Order.make[HNil]((_,_) => false)

        case hc : HListOrder.HConsOrder[head,tail] =>
          val orderHead: Order[head] = hc.orderHead
          val orderTail: Order[tail] = lex(hc.hlistOrderTail)

          Order.make[HCons[head, tail]] {
            case (HCons(h1,t1), HCons(h2,t2)) =>
              orderHead.lesserThan(h1,h2) || (orderHead.areEqual(h1,h2) && orderTail.lesserThan(t1,t2))
          }
      }
    ```

    </details>

- **Question 3:** Écrivez une fonction `def revLex[A](implicit v : HListOrder[A]): Order[A]`
  qui retourne l'ordre lexicographique inversée à partir d'une valeur de type `HListOrder[A]`.

    <details>
      <summary>*Solution (cliquer pour dévoiler)*</summary>

    ```scala
    def revLex[A](implicit v : HListOrder[A]): Order[A] =
      v match {
        case HListOrder.HNilOrder =>
          Order.make[HNil]((_,_) => false)

        case hc : HListOrder.HConsOrder[head,tail] =>
          val orderHead: Order[head] = hc.orderHead
          val orderTail: Order[tail] = revLex(hc.hlistOrderTail)

          Order.make[HCons[head, tail]] {
            case (HCons(h1,t1), HCons(h2,t2)) =>
              orderTail.lesserThan(t1,t2) || (orderTail.areEqual(t1,t2) && orderHead.lesserThan(h1,h2))
          }
      }
    ```

    </details>

Cette approche a de nombreux avantages. Alors que l'approche initiale devait effectuer
une recherche d'implicites pour chaque relation d'ordre, l'approche par *GADT* n'a besoin
de faire cette recherche qu'une seule fois!
Sachant que la résolution d'implicites est une opération gourmande, la réduire signifie des
temps de compilation plus courts.
Lire le code des fonctions `lex` et `revLex` est également plus simple que comprendre
comment la résolution d'implicites fonctionne pour les traits `Lex` et `RevLex`.
De plus, ce ne sont que des fonctions, vous pouvez y utiliser tout ce que vous pouvez
programmer afin de construire les instances de `Order[A]`.

## Conclusion

Pas si trivial, n'est ce pas? 😉 En fait, une grande part de la complexité
à laquelle vous venez de faire face vient du triste fait que les techniques
de raisonnements sur les types et valeurs n'est presque jamais enseigné dans
les cours de programmation.
Ce que vous trouvez simple maintenant (API Web, Streaming, Bases De Données, etc)
terrifierait probablement la/le jeune programmeuse·eur que vous étiez à votre
premier "Hello World!".
Vous n'avez probablement pas appris tout ce que vous savez en programmation en
trois heures, donc n'attendez pas des techniques de raisonnement sur des programmes
d'êtres magiquement plus simples.

Cet atelier avait pour but de vous inspirer, d'ouvrir votre esprit à ce nouvel univers
de possibilités. Si vous trouvez ces cas d'utilisation intéressants, alors prenez le temps
de comprendre les techniques.

Amusez vous bien et prenez bien soin de vous ❤️
