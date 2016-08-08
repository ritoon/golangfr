# Réussir avec Go

* [Introduction](#Introduction)
 - [Exemples](#Examples)                                                        
* [Formattage](#Formatting)
* [Commentaire](#Commentary)
* [Nommage](#Names)
 - [Les noms de Package](#Packagenames)
 - [Getteurs](#Getters)
 - [Les noms d'interface](#Interfacenames)
 - [MélangeDeCasses](#MixedCaps)
* [Les points-virgules](#Semicolons)
* [Les structures de controle](#Controlstructures)
 - [If](#If)
 - [Redéclaration et réaffectation](#Redeclarationandreassignment)
 - [For](#For)
 - [Switch](#Switch)
 - [Type switch](#Typeswitch)
* [Functions](#Functions)
 - [Retour multiple de valeurs](#Multiplereturnvalues)
 - [Paramètres de retour nommés](#Namedresultparameters)
 - [Defer](#Defer)
* [Data](#Data)
 - [Allocation with new](#Allocationwithnew)
 - [Constructeurs and composite littéraux](#Constructorsandcompositeliterals)
 - [Allocation avec make](#Allocationwithmake)
 - [Arrays](#Arrays)
 - [Slices](#Slices)
 - [Slices à deux dimentions](#Two-dimensionalslices)
 - [Maps](#Maps)
 - [Printing](#Printing)
 - [Append](#Append)
* [Initialisation](#Initialization)
 - [Constantes](#Constants)
 - [Variables](#Variables)
 - [La function init](#Theinitfunction)
* [Méthodes](#Methods)
 - [Pointers vs. Valeurs](#Pointersvs.Values)
* [Interfaces et autres types](#Interfacesandothertypes)
 - [Interfaces](#Interfaces)
 - [Conversions](#Conversions)
 - [Interface conversions et assertions de type](#Interfaceconversionsandtypeassertions)
 - [Généralité](#Generality)
 - [Interfaces et méthodes](#Interfacesandmethods)
* [L'identifiant vide](#Theblankidentifier)
 - [L'identifiant vide dans une assignation multiple](#Theblankidentifierinmultipleassignment)
 - [Variables et imports non utilisés](#Unusedimportsandvariables)
 - [Importation pour des effets de bord](#Importforsideeffect)
 - [Validation d'interface](#Interfacechecks)
* [Incorporation](#Embedding)
* [Concurrence](#Concurrency)
 - [Partagez en communiquant](#Sharebycommunicating)
 - [Les goroutines](#Goroutines)
 - [Les canaux](#Channels)
 - [Les canaux de canaux](#Channelsofchannels)
 - [Parallélisation](#Parallelization)
 - [Fuite mémoire dans le buffer](#Aleakybuffer)
* [Errors](#Errors)
 - [Panic](#Panic)
 - [Recover](#Recover)
* [Serveur web](#Awebserver)


## Introduction <a id="Introduction"></a>

**Go** est un nouveau langage. Bien qu'il emprunte des idées à partir d'autres existants, il a des propriétés inhabituelles qui lui permet de réaliser des programmes éfficaces mais avec un caractère différent des programmes écrits de ses pairs. Une simple traduction d'un programme **C++** ou **Java** en **Go** est peu susceptible de produire un résultat satisfaisant. Les programmes Java sont écrits en **Java**, pas en **Go**. D'un autre coté, réfléchir aux problèmes avec une vision **Go** permet de réaliser un programme réussi mais en tous cas assez différent. En d'autres termes pour écrire du **Go**, il est essentiel d'avoir compris ses propriétés et ses idiomes. Il est également important de connaître les conventions établies pour la programmation en **Go**, comme la dénomination, le formatage, la construction du programme, et ainsi de suite, de sorte que les programmes que vous écrivez seront facile à comprendre pour les autres programmeurs **Go**.

Ce document donne des conseils pour l'écriture de code propre et idiomatique en **Go**. Il complète la [spécification du langage](https://golang.org/ref/spec), le [Tour de Go](https://tour.golang.org/) et [Comment écrire du code Go](https://golang.org/doc/code.html), tout ce dont vous devriez lire en premier.

### Examples <a id="Examples"></a>

[Les sources des paquets Go](https://golang.org/src/) sont destinés à servir non seulement comme la bibliothèque de base, mais aussi comme des exemples de la façon d'utiliser le language. En outre, beaucoup de paquets contiennent de travail, autonome exemples exécutables que vous pouvez exécuter directement depuis le site Web de [golang.org](https://golang.org/), comme [celui-ci](https://golang.org/pkg/strings/#example_Map) (si nécessaire, cliquez sur le mot «Exemple» pour l'ouvrir ). Si vous avez une question sur la façon d'aborder un problème ou comment quelque chose pourrait être mise en œuvre, la documentation, le code et les exemples dans la bibliothèque peuvent fournir des réponses, des idées et dévoiler les coulisses.


## Formatage <a id="Formatting"></a>

les questions de mise en forme sont les plus litigieuses, mais aussi les moins conséquentes. Les gens peuvent s'adapter à différents styles de mise en forme, mais il est préférable qu'elles n'aient pas à le faire et que le moins de temps soit consacré à la question si tout le monde adhère au même style. Le problème est de savoir comment aborder cette Utopie sans une longue charte descriptive de style.

Avec **Go** nous prenons une approche inhabituelle et laissons la machine prendre soin de la plupart des problèmes de formatage. Le programme **gofmt** (également disponible en tant que ``go fmt`` , qui fonctionne au niveau du paquet plutôt qu'au niveau du fichier source) lit un programme **Go** et émet la source dans un style standard d'indentation d'alignement vertical, de retenue et si nécessaire reformate les commentaires. Si vous voulez savoir comment gérer un nouveau type de mise en page,  exécutez **gofmt** ; si la réponse ne semble pas juste, réorganisez votre programme (ou déposer un bug à propos de **gofmt** ), mais ne passez pas plus de temps autour du problème.

A titre d'exemple , il n'y a pas besoin de passer du temps en alignant les commentaires sur les champs d'une structure. ``gofmt`` va le faire pour vous. Compte tenu de la déclaration :


```go
type T struct {
    name string // name of the object
    value int // its value
}
```
**gofmt** va aligner les colonnes :

```go
type T struct {
    name    string // name of the object
    value   int    // its value
}
```

Tout le code **Go** dans les paquets standard a été formaté avec ``gofmt``.


Certains détails de mise en forme demeurent. Très brièvement :

**Indentation**

Nous utilisons des onglets pour l'indentation et ``gofmt`` les émet par défaut. Utilisez des espaces uniquement si vous devez.

**Longueur de ligne**

**Go** n'a pas de limite de longueur de ligne. Ne vous inquiétez pas de déborder sur votre carte perforée ! Si une ligne paraît trop longue, vous pouvez faire un retour à la ligne et une tabulation en supplémentaire.


**Parentheses**

**Go** a besoins de moins de parenthèses que le **C** et le **Java** : les structures de contrôle ( ``if``, ``for``, ``switch``) ne disposent pas de parenthèses dans leur syntaxe. En outre, la hiérarchie des opérateurs de priorité est plus courte et plus claire, par conséquent


```go
x<<8 + y<<16
```

l'espacement est signifiant, contrairement aux autres languages.

## Les commentaires <a id="Commentary"></a>

**Go** fournit des blocs de commentaires ```/ * * /``` au même titre que le **C** et le **C++** et des commentaires en ligne ```//```. Les commentaires de ligne sont la norme ; les commentaires de bloc eux apparaissent surtout comme des commentaires du paquet, mais sont aussi utiles pour désactiver de larges pans de code.

Le programme et le serveur web **GoDoc** permet d'extraire des fichiers sources la documentation sur le contenu du paquet. Les commentaires qui apparaissent avant les déclarations de haut niveau, sans retour de ligne intermédiaires, sont extraits avec la déclaration pour servir de texte explicatif pour la documentation. La nature et le style de ces commentaires détermine la qualité de la documentation **GoDoc** produit.

Chaque paquet doit avoir son commentaire de paquet, un bloc de commentaire précéde la clause de paquet. Pour les paquets multi-fichiers, les commentaires du paquet doit être présent dans un seul fichier, ainsi tous en bénéficieront. Les commentaires de paquet devront introduire le paquet et fournir des informations pertinentes au paquet dans son ensemble. Il apparaît d'abord sur la page **GoDoc** et devrait introduire la documentation détaillée qui suit.


```go
/*
Package regexp implements a simple library for regular expressions.

The syntax of the regular expressions accepted is:

    regexp:
        concatenation { '|' concatenation }
    concatenation:
        { closure }
    closure:
        term [ '*' | '+' | '?' ]
    term:
        '^'
        '$'
        '.'
        character
        '[' [ '^' ] character-ranges ']'
        '(' regexp ')'
*/
package regexp
```

Si le paquet est simple, les commentaires du paquet peuvent être bref.

```go
// Package path implements utility routines for
// manipulating slash-separated filename paths.
```

Les commentaires ne nécessitent pas de mise en forme supplémentaire tels que des bannières publicitaires. La sortie générée ne peut même pas être présenté dans une police à largeur fixe, donc ne dépend pas de l'espacement pour l'alignement **godoc**, comme **gofmt**, prend soin de cela. Les commentaires sont sans interprétation en texte brut, de sorte que HTML et d'autres annotations telles que ```_this_``` seront reporduites textuellement c'est pourquoi elles ne doivent pas être utilisé. Un seul réglage dans **godoc** possible est ​​d'afficher une police à largeur fixe avec l'intendation du texte, adapté aux extraits du programme. Les commentaires du paquet fmt utilise à bon escient cette fonctionnalité.

Selon le contexte, **godoc** pourrait même ne pas reformater commentaires. C'est pourquoi, il vaut mieux directement utiliser la bonne orthographe, la ponctuation, la strucuture de phrase, découper les longues lignes... etc pour s'assurer du résultat produit.

Dans un package, tous commentaires précédent immédiatement une déclaration de haut niveau est utilisé pour la documentation. Toutes déclaration exportés (première lettre en majuscule) dans un paquet doit avoir son commentaire de documentation.

Les commentaires de la doc marchent mieux avec des phrases complètes, ce qui permet une plus large palette de présentation automatisé. La première phrase doit, en principe, être une seule phrase permettant de résumer l'objet de la déclaration et doit commencer par son nom.

```go
// Compile parses a regular expression and returns, if successful, a Regexp
// object that can be used to match against text.
func Compile(str string) (regexp *Regexp, err error) {
```

Si le nom de package commence toujours par son commentaire, l'export dans **godoc** en sera plus partique via grep. Immaginez que vous ne vous rappelez plus du nom "Compile" mais que vous être entrain de chercher pour une fonction de parsing avec les expressions régulières, vous pouvez lancer la commande suivante :

```sh
$ godoc regexp | grep parse
```

Si tous les commentaires de la doc commançaient par : "This function...", ``grep`` ne pourrait pas aider pour se rappeler du nom. Mais parce que les paquets commencent chacuns par le commentaire avec le nom, vous verrez quelque chose de famillier, ce qui vous rappellera le mot que vous être entrain de chercher.

```sh
$ godoc regexp | grep parse
    Compile parses a regular expression and returns, if successful, a Regexp
    parsed. It simplifies safe initialization of global variables holding
    cannot be parsed. It simplifies safe initialization of global variables
$
```

**Go** permet de réaliser des groupement de déclaration. Un seul commentaire permet d'introduire un groupe de constantes ou variables associés. À partir du moment que ce commentaire groupé a été réalisé, les commentaires suivants peuvent être plus sommaires.

```go
// Error codes returned by failures to parse an expression.
var (
    ErrInternal      = errors.New("regexp: internal error")
    ErrUnmatchedLpar = errors.New("regexp: unmatched '('")
    ErrUnmatchedRpar = errors.New("regexp: unmatched ')'")
    ...
)
```

Grouper permet aussi d'indiquer une relation entre des éléments, comme le fait qu'un ensemble de variables est protégé par un mutex.

```go
var (
    countLock   sync.Mutex
    inputCount  uint32
    outputCount uint32
    errorCount  uint32
)
```

## Nommage <a id="Names"></a>

Les noms sont aussi imporants en **Go** que dans tous autres langages. Il y a même un effet sémantique : la visibilité d'une déclaration à l'extérieur du paquet est déterminé en fonction de sa première lettre en capitale. C'est pouquoi nous pouvons passer un peut de temps sur le sujet à savoir quels sont les conventions de nommage dans un programme **Go** ?

### Les noms de paquets <a id="Packagenames"></a>

Quand un paquet est importé, le nom du paquet devient un accésseur de son contenu. Après :

```go
import "bytes"
```

Le paquet importateur peut parler du **bytes.Buffer**. C'est beaucoup plus pratique si les développeurs utilisant les même paquets peuvent utiliser les même noms pour faire référence aux même contenus. Le nom du paquet doit par conséquent être parfait : court, concis, évocateur. Par convention, les paquets sont en minuscules, les noms d'un seul mot; il devrait y avoir aucun besoin de tiret ou d'une notation en casseMélangée. **Err** est un exemple pour sa  brièveté, puisque tout le monde en utilisant ce paquet écrira ce nom. Et ne vous inquiétez pas des collisions a priori. Le nom du paquet est seulement le nom par défaut pour les importations ; il n'a pas besoin d'être unique dans l'ensemble du code source, et dans des rares cas d'une collision, le paquet importateur peut choisir un nom différent à utiliser localement. Dans tous les cas, la confusion est rare parce que le nom de fichier dans l'importation détermine simplement quel paquet est utilisé.

Une autre convention est que le nom du paquet est le nom de base pour son dossier source ; le paquet dans **src/encoding/base64** est importé tel que **encoding/base64** mais a comme nom **base64** et pas **encoding_base64** ou encore **encodingBase64**.

L'importateur d'un package utilisera le nom pour faire référence à son contenu, par conséquent, le nommage des déclarations à l'intérieur du paquet doivent tennir en compte ce fait afin d'éviter tous phénomènes de répétition. Évitez  la notation d'importation ``.``, ce qui peut simplifier les tests qui doivent fonctionner en dehors du package qu'ils testent. Par exemple, le type de reader buffurisé dans le package **bufio** est appelé **Reader**, pas **BufReader**, parce les utilisateurs considèrent **bufio.Reader** comme un nom clair et concis. En outre, parce que les entités importées sont toujours traitées avec leur nom de package, **bufio.Reader** n'est pas en conflit avec **io.Reader**. De même, la fonction permettant de créer de nouvelles instances de **ring.Ring**, ce qui est la définition d'un constructeur dans Go serait normalement appelé **NewRing**, mais sachant que **Ring** est le seul type  exporté par le paquet et puisque le paquet est appelé **ring**, il est appelé simplement **New**, ainsi les utilisateurs du paquet voient **ring.New**. Utilisez la structure de paquet pour vous aider à choisir les bons noms.

Un dernier cours exemple est **once.Do** ; **once.Do(setup)** se lit facilement et le fait d'écrire **once.DoOrWaitUntilDone(setup)** ne permettra pas automatiquement d'être plus compréhensible. La documentation permet souvent d'être plus riche que des noms à rallonge.

### Getteurs <a id="Getters"></a>

**Go** ne donne pas automatiquement des getteurs et setteurs. Il y a rien de mal à les créer soi-même et il semble normal d'en créer, mais il n'est pas idiomatique ni nécessaire d'ajouter **Get** dans le nom du getteur. Si vous avez un champs appelé **owner** (bas de casse : non exporté), la méthode getteur serait **Owner** (haut de casse : exporté) et non **GetOwner**. L'utilisation de la casse pour l'exportation permet de distinguer le champs de la méthode. Si nous avons besoin d'une fonction setteur, nous pourrions l'appeler **SetOwner**. Les deux noms se lisent facilement dans la pratique :

```go
owner := obj.Owner()
if owner != user {
    obj.SetOwner(user)
}
```

### Les noms d'interfaces <a id="Interfacenames"></a>

Par convention, les interfaces de méthodes sont appelés par le nom de la méthode avec en plus le suffix **-er** ou une modification similaire pour créer un nom d'agent : **Reader**, **Writer**, **Formatter**, **CloseNotifier**... etc.

Il y a un certain nombre de ces noms et il est productif d'honorer cette règle pour permettre de distinguer les noms des fonctions. **Read**, **Write**, **Close**, **Flush**, **String** et ainsi de suite ont des signatures canoniques et des significations propre. Afin d'éviter toute confusion, ne donnez pas à votre méthode un de ces noms, sauf si elle a la même signature et signification. A l'inverse, si votre type implémente une méthode avec le même sens comme une méthode sur un type connu, donnez lui le même nom et la signature ; appelez votre méthode de conversion de chaîne **String** et non **ToString**.

### MélangesDeCasses <a id="MixedCaps"></a>

Pour finir, la convention en Go est d'utiliser le **CamelCase** ou **camelCase** au lieux des tirets afin d'écrire des expressions de mots composés.

## Les points virgules <a id="Semicolons"></a>

Tous comme le **C**, **Go** utilise dans sa grammaire formelle le point virgule afin de finir une déclaration, mais contrairement au C, les points virgules n'apparaissent pas dans le code source. À la place le **lexer** utilise une règle simple permettant d'en insérer automatiquement au moment du balayage des sources, c'est pourquoi le texte exporté en est pratiquement viege.

La règle est la suivante : si le mot clée avant une nouvelle ligne est un identificateur (ce qui inclut les mots tel que ``int`` et ``float64``), une donnée littérale tel un nombre, une constante de chaîne ou un de ces mot clée :

```go
break continue fallthrough return ++ -- ) }
```

Le **lexer** insérera un point virgule à la fin du mot clée. Ce qui pourrait se résumer par : "si une nouvelle ligne viens après un mot clée qui pourrait être une déclaration, insère un point virgule".

Un point virgule peut être aussi omit tout de suite après une accolade fermante. Ainsi la déclaration suivante n'a pas besoin de point virgules :

```go
    go func() { for { dst <- <-src } }()
```

Les programmes idiomatiques en **Go** n'ont des points virugles que dans les boucles, afin de séparer l'initialiseur, la condition et l'élément de continuité. Ils sont aussi necessaire afin de permettre de séparer des déclarations multiple sur une seule ligne (non recommandé).

Une des conséquences des règles d'insertion des points virgules est qu'on ne peut pas mettre un retour de ligne avant une accolade ouvrante dans une structure de constrôle (``if``, ``for``, ``switch`` ou ``select``). Si vous le faites, un point virgule sera inséré juste après l'accolade, ce qui pourrait causer des effets non voulus.

Il faut écrire de cette façon :

```go
if i < f() {
    g()
}
```

et pas comme ceci :

```go
if i < f()  // wrong!
{           // wrong!
    g()
}
```

## Les structures de control  <a id="Controlstructures"></a>

Les structures de contrôle en **Go** ressemblent à celles du **C** mais diffèrent dans de nombreux cas. Il n'y a pas de boucle ``while``, seulement une généralisation de la boucle ``for``, qui est plus fléxible. ``if`` et ``switch`` accèptent un paramètre optionnel d'initialisation tel que celui du ``for``. Les déclarations ``break`` et ``continue`` peuvent prendre un paramètre optionnel permettant de définir ce qui ``break`` et ``continue``. Il y a aussi une nouvelle structure de contrôle qui est un nouveau type de ``switch``, le ``select`` qui permet de multiplexer des communications multidirectionnel. La syntaxe est aussi un peut différente : il n'y a pas de parenthèses et le corp de la structure de contrôle doit toujours être délimité par des accolades.

### If <a id="If"></a>

Dans Go un simple ``if`` ressemble a ça :

```go
if x > 0 {
    return y
}
```

L'obligation des accolades du ``if`` encouragent l'écriture de déclaration simple sur plusieurs lignes. C'est de toute façon un style plus lisible et ce tous particulièrement quand le corp de la structure de contrôle comporte des déclaration comme ``return`` ou ``break``.

Sachant que ``if`` et ``switch`` acceptent tous deux une déclaration d'initialisation, il est commun les voir avec une création d'une variable local.

```go
if err := file.Chmod(0664); err != nil {
    log.Print(err)
    return err
}
```

Dans la librairie standard de **Go**, vous verrez que quand une instruction ``if`` ne circule pas dans le corp de la déclaration et qu'elle se termine en ``break``, ``continue``, ``goto``, ou ``return`` le ``else`` est omis.


```go
f, err := os.Open(name)
if err != nil {
    return err
}
codeUsing(f)
```

Ceci est un exemple d'une situation commune où le code doit se prémunir contre une séquence de conditions d'erreur. Le code se lit bien si le déroulement réussi jusqu'en en bas de page, en éliminant les cas d'erreur à mesure qu'ils surviennent. Comme les cas d'erreurs ont tendance à se terminer dans des déclarations de retour, le code résultant n'a pas besoin de déclarations ``else``.

```go
f, err := os.Open(name)
if err != nil {
    return err
}
d, err := f.Stat()
if err != nil {
    f.Close()
    return err
}
codeUsing(f, d)
```

### Redeclaration et réaffectation <a id="Redeclarationandreassignment"></a>

Nous pouvons préciser que le dernier exemple de la section précédente montre un détail de la façon dont une déclaration courte ``:=`` est mise en œuvres. La déclaration qui appelle ``os.Open`` lit :

```go
f, err := os.Open(name)
```

Cette déclaration déclare deux variables, ``f`` et ``err``. Quelques lignes plus tard, l'appel de ``f.Stat`` lit :

```go
d, err := f.Stat()
```

qui ressemble comme si elle déclare ``d`` et ``err``. Remarquez, cependant, que ``err`` apparaît dans les deux déclarations. Cette duplication est légal : ``err`` est créé par la première déclaration, mais est seulement réaffecté dans la seconde. Cela signifie que l'appel à ``f.Stat`` utilise la variable ``err`` existante déclarée ci-dessus, et juste lui donne une nouvelle valeur.

Dans une déclaration ``:=`` une variable ``v`` peut apparaître même si elle a déjà été déclarée, à condition :

- que cette déclaration est dans la même portée que la déclaration existante de ``v`` (si ``v`` est déjà déclarée dans une portée externe, la déclaration va créer un nouvelle variable [1](#1)),
- la valeur correspondante dans l'initialisation est assignable à ``v``,
- et il y a au moins une autre variable dans la déclaration qui est déclarée à nouveau.

Cette propriété inhabituelle est pur pragmatisme, ce qui rend facile à utiliser une valeur ``err`` unique, par exemple, dans une longue suite d' ``if else``, vous la verrez souvent utilisé.

<a id="1">1 - </a>Il est intéressant de noter ici que, la portée d'une fonction les paramètres et les valeurs de retour sont les même que le corps de la fonction, même si elles apparaissent lexicalement en dehors des accolades qui entourent le corps.

### For <a id="For"></a>

La boucle ``for`` de **Go** et celle du **C** sont similaire mais pas identique. Elle unifie ``for`` et ``while``, et il n'y a pas de ``do-while``. Elle existe sous trois aspects, seulement une a des points virugles.

```go
// Like a C for
for init; condition; post { }

// Like a C while
for condition { }

// Like a C for(;;)
for { }
```

La déclaration courte permet la déclaration facile d'indexe de variable dirrectement dans la boucle.

```go
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```

Si vous bouclez dans un **tableau**, une **slice** (tranche), une **string**, une **map** ou que vous lisez dans une **channel** (canal), la clause ``range`` peut vous permettre de gérer la boucle.

```go
for key, value := range oldMap {
    newMap[key] = value
}
```

Si vous avez juste besoin du premier élément renvoyé par ``range`` (clée ou index), vous pouvez omettre le segond.

```go
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}
```

Si vous avez uniquement besoin du segond, vous pouvez utiliser l'identifiant vide (``_``), afin de passer sous silence le premier :

```go
sum := 0
for _, value := range array {
    sum += value
}
```

L'identifiant vide a beaucoup d'usages, nous reviendrons desssus dans une prochaine section.

Pour les chaînes de caractères, ``range`` fait plus de travail pour vous, brisant les points de code **Unicode** individuels en analysant l'**UTF-8**. codages erronés consomment un octet et produisent la ``rune`` de remplacement **U+FFFD**. (Le nom (avec le type builtin associé) ``rune`` est une terminologie de **Go** pour un seul point de code Unicode. Voir la spécification du langage pour les détails.) La boucle :

```go
for pos, char := range "日本\x80語" { // \x80 is an illegal UTF-8 encoding
    fmt.Printf("character %#U starts at byte position %d\n", char, pos)
}
```
affiche :

```sh
character U+65E5 '日' starts at byte position 0
character U+672C '本' starts at byte position 3
character U+FFFD '�' starts at byte position 6
character U+8A9E '語' starts at byte position 7
```

Enfin, **Go** n'a pas opérateur virgule deplus ``++`` et ``--`` sont des déclarations non des expressions. Ainsi, si vous voulez exécuter plusieurs variables en une vous devez utiliser l'affectation parallèle (bien que cela exclut ``++`` et ``--``).

```go
// Reverse a
for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
    a[i], a[j] = a[j], a[i]
}
```

### Switch <a id="Switch"></a>

Le ``switch`` de **Go** est plus ouvert que celui du **C**. Les expressions ne sont pas obligatoirement des constantes ou même des nombres entiers, les cas sont évalués de haut en bas jusqu'à ce qu'une correspondance soit trouvée. Si le ``switch`` n'a pas d'expression trouvée, il commute sur ``true``. Il est donc possible et idiomatique d'écrire une suite d'``if-else-if-else`` avec un ``switch``.

```go
func unhex(c byte) byte {
    switch {
    case '0' <= c && c <= '9':
        return c - '0'
    case 'a' <= c && c <= 'f':
        return c - 'a' + 10
    case 'A' <= c && c <= 'F':
        return c - 'A' + 10
    }
    return 0
}
```

Il n'y a pas de sortie automatique, mais des cas peuvent être présentés dans des listes séparées par des virgules.

```go
func shouldEscape(c byte) bool {
    switch c {
    case ' ', '?', '&', '=', '#', '+', '%':
        return true
    }
    return false
}
```

Bien qu'ils ne sont pas aussi courant dans **Go** comme dans certains autres languages proches du **C**, la déclaration ``break`` peut être utilisés pour mettre fin à un ``switch`` de manière précoce. Cependant parfois, il est nécessaire de sortir d'une boucle parente, ce qui peut être accompli en mettant une étiquette sur la boucle et ainsi «casser» la boucle à cette étiquette. Cet exemple montre les deux utilisations :

```go
Loop:
	for n := 0; n < len(src); n += size {
		switch {
		case src[n] < sizeOne:
			if validateOnly {
				break
			}
			size = 1
			update(src[n])

		case src[n] < sizeTwo:
			if n+1 >= len(src) {
				err = errShortInput
				break Loop
			}
			if validateOnly {
				break
			}
			size = 2
			update(src[n] + src[n+1]<<shift)
		}
	}
```

Bien sûr, l'instruction ``continue`` accepte également une étiquette facultative mais elle s'applique uniquement aux boucles.

Pour fermer cette section, voici une routine de comparaison pour les slices (tranches) d'octets qui utilise deux états de commutation :

```go
// Compare returns an integer comparing the two byte slices,
// lexicographically.
// The result will be 0 if a == b, -1 if a < b, and +1 if a > b
func Compare(a, b []byte) int {
    for i := 0; i < len(a) && i < len(b); i++ {
        switch {
        case a[i] > b[i]:
            return 1
        case a[i] < b[i]:
            return -1
        }
    }
    switch {
    case len(a) > len(b):
        return 1
    case len(a) < len(b):
        return -1
    }
    return 0
}
```

### Type switch <a id="Typeswitch"></a>

Un ``switch`` peut également être utilisé pour découvrir le type dynamique d'une variable d'interface. Un tel ``switch`` de type utilise la syntaxe d'une assertion de type avec le type de mot-clé à l'intérieur des parenthèses. Si le ``switch`` déclare une variable dans l'expression, la variable aura le type correspondant dans chaque clause. Il est également idiomatique de réutiliser le nom dans de tels cas. Ce qui a pour effet de déclarer un nouvelle variable avec le même nom mais un type qui peut varier à chaque fois.

```go
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("unexpected type %T\n", t)     // %T prints whatever type t has
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool
case int:
    fmt.Printf("integer %d\n", t)             // t has type int
case *bool:
    fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
case *int:
    fmt.Printf("pointer to integer %d\n", *t) // t has type *int
}
```

## Les fonctions <a id="Functions"></a>

### Retours de multiples valeurs <a id="Multiplereturnvalues"></a>

L'une des caractéristiques inhabituelles de **Go** est que les fonctions et les méthodes peuvent renvoyer des valeurs multiples. Ce type de signature peut être utilisé pour améliorer quelques idiomes maladroits dans les programmes écrits en **C** : retours d'erreurs en bande tels que **-1** pour **EOF** et la modification d'un argument transmis par adresse.

En **C**, une erreur d'écriture est signalée par un nombre négatif avec le code d'erreur sécrétée loin dans un endroit instable. Dans **Go**, ``Write`` peut renvoyer un nombre et une erreur: "Oui, vous avez écrit quelques octets, mais pas tous d'entre eux parce que vous avez rempli l'équipement". La signature de la méthode ``Write`` sur les fichiers de package ``os`` est :

```go
func (file *File) Write(b []byte) (n int, err error)
```

et comme le dit la documentation, elle renvoie le nombre d'octets écrits et une erreur non nul lorsque ``n != len(b)``. Ceci est un style commun; voir la section sur la gestion des erreurs pour plus d'exemples.

Une approche similaire évite la nécessité de passer un pointeur vers une valeur de retour pour simuler un paramètre de référence. Voici une fonction simpliste  pour prendre un certain nombre d'une position dans une tranche d'octets, retournant le nombre et la position suivante.

```go
func nextInt(b []byte, i int) (int, int) {
    for ; i < len(b) && !isDigit(b[i]); i++ {
    }
    x := 0
    for ; i < len(b) && isDigit(b[i]); i++ {
        x = x*10 + int(b[i]) - '0'
    }
    return x, i
}
```

Vous pouvez l'utiliser pour analyser les nombres dans une entrée tranche ``b`` comme ceci :

```go
    for i := 0; i < len(b); {
        x, i = nextInt(b, i)
        fmt.Println(x)
    }
```

### Les paramètres de retour nommés <a id="Namedresultparameters"></a>

Le retour ou résultat "paramètres" d'une fonction **Go** peuvent être nommés et utilisés en tant que variables régulières, tout comme les paramètres entrants. Lorsque les retours sont nommés, ils sont initialisés aux valeurs zéro pour leurs types lorsque la fonction commence; si la fonction exécute une instruction de retour sans argument, les valeurs actuelles des paramètres de résultat sont utilisés comme les valeurs retournées.

Les noms ne sont pas obligatoires, mais ils peuvent rendre le code plus court et plus clair : ils sont la documentation. Si nous nommons les retours de ``nextInt`` il devient évident de savoir qui est qui.

```go
func nextInt(b []byte, pos int) (value, nextPos int) {
```

Parce que les résultats nommés sont initialisés et sans fioritures, ils peuvent simplifier ainsi que clarifier la lecture du code. Voici une version de ``io.ReadFull`` qui les utilise bien :

```go
func ReadFull(r Reader, buf []byte) (n int, err error) {
    for len(buf) > 0 && err == nil {
        var nr int
        nr, err = r.Read(buf)
        n += nr
        buf = buf[nr:]
    }
    return
}
```

### Defer <a id="Defer"></a>

La déclaration ``defer`` de **Go** permet de planifier un appel de fonction (fonction différée) qui est executé immédiatement avant le retour de la fonction le contenant. C'est un moyen inhabituelle mais efficace pour faire face à des situations tel que des ressources qui doivent être libérés, peu importe quel chemin la fonction a comme retour. Les cas d'usage les plus courants sont par exemple le déverrouillage d'un mutex ou la fermeture d'un fichier.

```go
// Contents returns the file's contents as a string.
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // f.Close will run when we're finished.

    var result []byte
    buf := make([]byte, 100)
    for {
        n, err := f.Read(buf[0:])
        result = append(result, buf[0:n]...) // append is discussed later.
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err  // f will be closed if we return here.
        }
    }
    return string(result), nil // f will be closed if we return here.
}
```

Différer un appel de fonction tel que ``Close`` a deux avantages. Tout d'abord, elle garantit de jamais oublier de fermer le fichier, une erreur qui est si facile à faire si vous modifiez ultérieurement la fonction. Deuxièmement, cela signifie que la fermeture se trouve à **proximité** de l'ouverture, ce qui est beaucoup plus clair que de le placer à la fin de la fonction.

Les arguments de la fonction différée (qui comprennent le récepteur si la fonction est une méthode) sont évaluées lorsque le ``defer`` s'exécute, pas quand l'appel s'exécute. En plus d'éviter les soucis concernant les variables qui changent de valeurs lorsque la fonction s'exécute, ``defer`` permet d' exécuter une cascade de fonctions. Voici un exemple simpliste :


```go
for i := 0; i < 5; i++ {
    defer fmt.Printf("%d ", i)
}
```

Les fonctions différés sont exécutées dans l'ordre LIFO (last in first out), de sorte que ce code affichera 4 3 2 1 0 lors du retour de la fonction. Un exemple plus plausible est un moyen simple de tracer l'exécution de la fonction dans le programme. Nous pourrions écrire quelques routines de traçage simples comme ceci :


```go
func trace(s string)   { fmt.Println("entering:", s) }
func untrace(s string) { fmt.Println("leaving:", s) }

// Use them like this:
func a() {
    trace("a")
    defer untrace("a")
    // do something....
}
```

Nous pouvons faire mieux en exploitant le fait que les arguments des fonctions différées sont évaluées lorsque le ``defer`` s'exécute. La routine de traçage peut mettre en place l'argument à la routine ``untrace``. Tel que :

```go
func trace(s string) string {
    fmt.Println("entering:", s)
    return s
}

func un(s string) {
    fmt.Println("leaving:", s)
}

func a() {
    defer un(trace("a"))
    fmt.Println("in a")
}

func b() {
    defer un(trace("b"))
    fmt.Println("in b")
    a()
}

func main() {
    b()
}
```

affiche :

```sh
entering: b
in b
entering: a
in a
leaving: a
leaving: b
```

Pour les programmeurs habitués à des blocages au niveau la gestion des ressources dans d'autres langues, ``defer`` peut sembler étrange. Mais ses applications les plus intéressantes et les plus puissants viennent précisément du fait que cela ne repose pas sur un blocage, mais sur une fonction. Dans la section sur  ``panic`` et ``recover``, nous allons voir un autre exemple de ses  possibilités.

## Data <a id="Data"></a>

### L'Allocation avec new <a id="Allocationwithnew"></a>

**Go** a deux primitives d'allocation, les fonctions intégrées ``new`` et ``make``. Elles font des choses différentes et s'appliquent à différents types, ce qui peut être source de confusions, mais les règles sont simples. Parlons de ``new`` pour commencer, c'est une fonction intégrée qui alloue de la mémoire ; mais contrairement à ses homonymes dans d'autres langues ; il n'initialise pas la mémoire, il donne juste la valeur zéros du type. Autrement dit, ``new(T)`` alloue un stockage pour un nouvel élément de type ``T``, lui donne sa valeur zéro et retourne son adresse, une valeur de type ``*T``. Nous pouvons dire dans la terminologie **Go** : il renvoie un pointeur sur une valeur nouvellement allouée **nul** de type ``T``.

À partir du moment où ``new`` renvoie une valeur initialisé à sa valeur zéro, il est utile d'organiser la conception de vos structures de données avec une valeur zéro de chaque type qui peut être utilisé sans plus de travail d'initialisation. Cela signifie qu'un utilisateur de la structure de données peut créer une variable avec ``new`` qui est prêt à l'emploie. Par exemple, la documentation ``bytes.Buffer`` indique que "la valeur zéro pour le tampon est un tampon vide prêt à l'emploi." De même, ``sync.Mutex`` n'a pas de constructeur explicite ou méthode ``Init``. Au lieu de cela, la valeur zéro pour un ``sync.Mutex`` est défini comme étant un mutex déverrouillé.

La valeur zéro est une utile propriété qui fonctionne de manière transitive. Considérez cette déclaration de type :

```go
type SyncedBuffer struct {
    lock    sync.Mutex
    buffer  bytes.Buffer
}
```

Les valeurs de type ``SyncedBuffer`` sont également immédiatement prêtes à l'utilisation après l'attribution ou juste la déclaration. Dans le prochain extrait, ``p`` et ``v`` fonctionneront correctement sans autre arrangement.

```go
p := new(SyncedBuffer)  // type *SyncedBuffer
var v SyncedBuffer      // type  SyncedBuffer
```

### Constructeur et composite littéreaux <a id="Constructorsandcompositeliterals"></a>

Parfois, la valeur zéro n'est pas suffisante et un constructeur initialisant est nécessaire, comme dans cet exemple dérivé de package **os**.

```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := new(File)
    f.fd = fd
    f.name = name
    f.dirinfo = nil
    f.nepipe = 0
    return f
}
```

Il y a beaucoup de petites tâches dans la fonction ``NewFile``. Nous pouvons la simplifier avec l'utilisation d'une littérale composite, qui est une expression qui crée une nouvelle instance à chaque fois qu'elle est évalué.

```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := File{fd, name, nil, 0}
    return &f
}
```

Notez que, contrairement à **C**, il est parfaitement **OK** de renvoyer l'adresse d'une variable locale ; le stockage associé à la variable survit après le retour de la fonction. En fait, en prenant l'adresse d'un littéral composite, **Go** alloue une nouvelle instance à chaque fois qu'il l'évalue, donc nous pouvons combiner ces deux dernières lignes :

```go
return &File{fd, name, nil, 0}
```

Les champs d'un littéral composite sont disposées en ordre et doivent tous être présents. Cependant, en étiquetant les éléments explicitement comme des paires ``champ:valeurs``, les initialiseurs peuvent apparaître dans un ordre quelconque. Nous pouvons, de plus, omettre un champ qui prend sa valeur zéro par défaut. Ainsi pourrait-on dire :

```go
return &File{fd: fd, name: name}
```

Comme un cas limite, si un littéral composite ne contient pas de champs du tout, il crée une valeur zéro pour le type. Les expressions ``new(File)`` et de ``&File{}`` sont équivalentes.

Composite literals can also be created for arrays, slices, and maps, with the field labels being indices or map keys as appropriate. In these examples, the initializations work regardless of the values of Enone, Eio, and Einval, as long as they are distinct.

Les composites littéraux peuvent également être créés pour les **tableaux**, les **slices** et les **maps**, avec les étiquettes de champ étant des indices ou des clés de la carte, le cas échéant. Dans ces exemples, les initialisations fonctionnent quelles que soient les valeurs de énone, EIO, EINVAL, tant qu'ils sont distincts.

```go
a := [...]string   {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
s := []string      {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
m := map[int]string{Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
```

### Allocation avec make <a id="Allocationwithmake"></a>

Retour à l'allocation. La fonction intégré ``make(T, args)`` a des objectifs différents de ``new(T)``. Elle permet de créer des slices (tranches), des maps et des chanels (canaux) seulement et elle retourne une valeur initialisée (non mis à zéro) de type ``T`` (pas ``*T``). La raison de cette distinction est que ces trois types représentent, sous des couvertures, des références à des structures de données qui doivent être initialisées avant utilisation. Une slice, par exemple, est un descripteur de trois élément contenant un pointeur vers les données (dans un tableau), la longueur et la capacité. Jusqu'à ce que ces éléments soient initialisés, la slice est **nil**. Pour les slices, les maps et les chanels, ``make`` initialise la structure de données interne et prépare la valeur pour l'utilisation. Par exemple :

```go
make([]int, 10, 100)
```

alloue un tableau de 100 ints puis crée une structure de tranche de longueur 10 et une capacité de 100 pointant sur les 10 premiers éléments du tableau (Lorsque vous faites une slice, la capacité peut être omise. Voir la section sur les tranches pour plus d'informations). En revanche, ``new([] int)`` renvoie un pointeur sur une structure de tranche initialisé à sa valeur zéro nouvellement allouée, qui est un pointeur vers une valeur de tranche **nil**.

Ces exemples illustrent la différence entre les ``new`` et ``make``.

```go
var p *[]int = new([]int)       // allocates slice structure; *p == nil; rarely useful
var v  []int = make([]int, 100) // the slice v now refers to a new array of 100 ints

// Unnecessarily complex:
var p *[]int = new([]int)
*p = make([]int, 100, 100)

// Idiomatic:
v := make([]int, 100)
```

Rappelez-vous que ``make`` s'applique uniquement aux maps, des slices et aux  chanels et ne retourne pas un pointeur. Pour obtenir un pointeur explicite allouez à nouveau ou prennez l'adresse d'une variable explicitement.

### Tableaux <a id="Arrays"></a>

Les tableaux sont utiles lors d'une planification méticuleuse de la mémoire et peuvent parfois aider à éviter des allocations inutiles. mais surtout ils sont un bloc de construction pour les tranches (slices), le sujet de la prochaine section. Pour jeter les bases de ce sujet, voici quelques mots sur les tableaux.

Il existe des différences majeurs entre les façons dont les tableaux fonctionnent entre le **Go** et le **C**. Dans **Go** :

- Les tableaux sont des valeurs. l'Affectation d'un tableau à un autre copie tous les éléments.
- En particulier, si vous passez un tableau à une fonction, il recevra une copie du tableau, pas un pointeur vers elle.
- La taille d'un tableau fait partie de son type. Les types ``[10]int`` et ``int[20]`` sont distincts.
- La propriété de la valeur peut être utile, mais aussi coûteuse ; si vous voulez le comportement et l'efficacité du **C**, vous pouvez passer un pointeur vers le tableau.


```go
func Sum(a *[3]float64) (sum float64) {
    for _, v := range *a {
        sum += v
    }
    return
}

array := [...]float64{7.0, 8.5, 9.1}
x := Sum(&array)  // Note the explicit address-of operator
```

Mais même ce style est pas idiomatique en **Go**. Utilisez des tranches (slices) à la place.

### Tranches (slices) <a id="Slices"></a>

Une **slice** enveloppent un tableau pour donner une interface plus générale, puissante et pratique à des séquences de données. À l'exception des éléments de dimension explicite telles que les matrices de transformation, la plupart des programmes de collection de données dans **Go** sont fait avec des tranches plutôt que de simples tableaux.

Les **slices** détiennent les références à un tableau sous-jacent et si vous affectez une tranche à l'autre, les deux se réfèrent à la même matrice. Si une fonction prend un argument de **slice**, les modifications qu'elle apporte à des éléments de la tranche seront visible à l'appelant, ce qui est analogue à passer un pointeur vers le tableau sous-jacent. Une fonction ``Read`` peut donc accepter un argument de **slice** plutôt qu'un pointeur et un compteur ; la longueur dans la tranche fixe une limite supérieure de la quantité de données à lire. Voici la signature de la méthode ``Read`` du type de ``File`` dans le package ``os`` :

```go
func (f *File) Read(buf []byte) (n int, err error)
```

La méthode retourne le nombre d'octets lus et une valeur d'erreur, le cas échéant. Pour lire dans les 32 premiers octets d'un plus grand tampon buf, tranche (ici utilisé comme un verbe) le buffer (tampon).

```go
n, err := f.Read(buf[0:32])
```

Ce **"slicing"** (découpage ou tranchage) est commune et efficace. En fait, en laissant l'efficacité de côté, l'extrait suivant lirait également les 32 premiers octets de la mémoire tampon.

```go
    var n int
    var err error
    for i := 0; i < 32; i++ {
        nbytes, e := f.Read(buf[i:i+1])  // Read one byte.
        if nbytes == 0 || e != nil {
            err = e
            break
        }
        n += nbytes
    }
```

La longueur d'une tranche peut être modifiée aussi longtemps qu'elle se glisse dans les limites de la matrice sous-jacente ; il faut juste assigner à une **slice** à elle-même. La capacité d'une tranche, accessible par la fonction intégrée ``cap()``, rapporte la longueur maximale que la tranche peut assumer. Voici une fonction pour ajouter des données à une tranche. Si les données dépassent la capacité, la tranche est réattribué. La tranche résultante est renvoyée. La fonction utilise le fait que ``len`` et ``cap`` sont légaux lorsqu'elle est appliquée à une **slice** ``nil`` et retourne ainsi ``0``.

```go
func Append(slice, data []byte) []byte {
    l := len(slice)
    if l + len(data) > cap(slice) {  // reallocate
        // Allocate double what's needed, for future growth.
        newSlice := make([]byte, (l+len(data))*2)
        // The copy function is predeclared and works for any slice type.
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[0:l+len(data)]
    for i, c := range data {
        slice[l+i] = c
    }
    return slice
}
```

Nous devons retourner la **slice** ensuite parce que, bien que ``Append`` peut modifie les éléments de la **slice**, la **slice** elle-même (la structure de données d'exécution tenant le pointeur, la longueur et la capacité) est passé par valeur.

L'idée d'ajouter des éléments dans une tranche est si utile, que **Go** a une fonction intégrée ``append``. Pour comprendre la conception de cette fonction, cependant, nous avons besoin d'un peu plus d'informations, donc nous y reviendrons plus tard.

### Les tranches (slices) à deux dimensions <a id="Two-dimensionalslices"></a>

Go's arrays and slices are one-dimensional. To create the equivalent of a 2D array or slice, it is necessary to define an array-of-arrays or slice-of-slices, like this:

```go
type Transform [3][3]float64  // A 3x3 array, really an array of arrays.
type LinesOfText [][]byte     // A slice of byte slices.
```

Because slices are variable-length, it is possible to have each inner slice be a different length. That can be a common situation, as in our LinesOfText example: each line has an independent length.

```go
text := LinesOfText{
	[]byte("Now is the time"),
	[]byte("for all good gophers"),
	[]byte("to bring some fun to the party."),
}
```

Sometimes it's necessary to allocate a 2D slice, a situation that can arise when processing scan lines of pixels, for instance. There are two ways to achieve this. One is to allocate each slice independently; the other is to allocate a single array and point the individual slices into it. Which to use depends on your application. If the slices might grow or shrink, they should be allocated independently to avoid overwriting the next line; if not, it can be more efficient to construct the object with a single allocation. For reference, here are sketches of the two methods. First, a line at a time:

```go
// Allocate the top-level slice.
picture := make([][]uint8, YSize) // One row per unit of y.
// Loop over the rows, allocating the slice for each row.
for i := range picture {
	picture[i] = make([]uint8, XSize)
}
```

And now as one allocation, sliced into lines:

```go
// Allocate the top-level slice, the same as before.
picture := make([][]uint8, YSize) // One row per unit of y.
// Allocate one large slice to hold all the pixels.
pixels := make([]uint8, XSize*YSize) // Has type []uint8 even though picture is [][]uint8.
// Loop over the rows, slicing each row from the front of the remaining pixels slice.
for i := range picture {
	picture[i], pixels = pixels[:XSize], pixels[XSize:]
}
```

### Les cartes (maps) <a id="Maps"></a>

Maps are a convenient and powerful built-in data structure that associate values of one type (the key) with values of another type (the element or value) The key can be of any type for which the equality operator is defined, such as integers, floating point and complex numbers, strings, pointers, interfaces (as long as the dynamic type supports equality), structs and arrays. Slices cannot be used as map keys, because equality is not defined on them. Like slices, maps hold references to an underlying data structure. If you pass a map to a function that changes the contents of the map, the changes will be visible in the caller.

Maps can be constructed using the usual composite literal syntax with colon-separated key-value pairs, so it's easy to build them during initialization.

```go
var timeZone = map[string]int{
    "UTC":  0*60*60,
    "EST": -5*60*60,
    "CST": -6*60*60,
    "MST": -7*60*60,
    "PST": -8*60*60,
}
```

Assigning and fetching map values looks syntactically just like doing the same for arrays and slices except that the index doesn't need to be an integer.

```go
offset := timeZone["EST"]
```

An attempt to fetch a map value with a key that is not present in the map will return the zero value for the type of the entries in the map. For instance, if the map contains integers, looking up a non-existent key will return 0. A set can be implemented as a map with value type bool. Set the map entry to true to put the value in the set, and then test it by simple indexing.

```go
attended := map[string]bool{
    "Ann": true,
    "Joe": true,
    ...
}

if attended[person] { // will be false if person is not in the map
    fmt.Println(person, "was at the meeting")
}
```

Sometimes you need to distinguish a missing entry from a zero value. Is there an entry for "UTC" or is that the empty string because it's not in the map at all? You can discriminate with a form of multiple assignment.

```go
var seconds int
var ok bool
seconds, ok = timeZone[tz]
```

For obvious reasons this is called the “comma ok” idiom. In this example, if tz is present, seconds will be set appropriately and ok will be true; if not, seconds will be set to zero and ok will be false. Here's a function that puts it together with a nice error report:

```go
func offset(tz string) int {
    if seconds, ok := timeZone[tz]; ok {
        return seconds
    }
    log.Println("unknown time zone:", tz)
    return 0
}
```

To test for presence in the map without worrying about the actual value, you can use the blank identifier (_) in place of the usual variable for the value.

```go
_, present := timeZone[tz]
```

To delete a map entry, use the delete built-in function, whose arguments are the map and the key to be deleted. It's safe to do this even if the key is already absent from the map.

```go
delete(timeZone, "PDT")  // Now on Standard Time
```

### L'impression <a id="Two-Printing"></a>

Formatted printing in Go uses a style similar to C's printf family but is richer and more general. The functions live in the fmt package and have capitalized names: fmt.Printf, fmt.Fprintf, fmt.Sprintf and so on. The string functions (Sprintf etc.) return a string rather than filling in a provided buffer.

You don't need to provide a format string. For each of Printf, Fprintf and Sprintf there is another pair of functions, for instance Print and Println. These functions do not take a format string but instead generate a default format for each argument. The Println versions also insert a blank between arguments and append a newline to the output while the Print versions add blanks only if the operand on neither side is a string. In this example each line produces the same output.

```go
fmt.Printf("Hello %d\n", 23)
fmt.Fprint(os.Stdout, "Hello ", 23, "\n")
fmt.Println("Hello", 23)
fmt.Println(fmt.Sprint("Hello ", 23))
```

The formatted print functions fmt.Fprint and friends take as a first argument any object that implements the io.Writer interface; the variables os.Stdout and os.Stderr are familiar instances.

Here things start to diverge from C. First, the numeric formats such as %d do not take flags for signedness or size; instead, the printing routines use the type of the argument to decide these properties.

```go
var x uint64 = 1<<64 - 1
fmt.Printf("%d %x; %d %x\n", x, x, int64(x), int64(x))
```
prints

```sh
18446744073709551615 ffffffffffffffff; -1 -1
```

If you just want the default conversion, such as decimal for integers, you can use the catchall format %v (for “value”); the result is exactly what Print and Println would produce. Moreover, that format can print any value, even arrays, slices, structs, and maps. Here is a print statement for the time zone map defined in the previous section.

```go
fmt.Printf("%v\n", timeZone)  // or just fmt.Println(timeZone)
```

which gives output

```sh
map[CST:-21600 PST:-28800 EST:-18000 UTC:0 MST:-25200]
```

For maps the keys may be output in any order, of course. When printing a struct, the modified format %+v annotates the fields of the structure with their names, and for any value the alternate format %#v prints the value in full Go syntax.

```go
type T struct {
    a int
    b float64
    c string
}
t := &T{ 7, -2.35, "abc\tdef" }
fmt.Printf("%v\n", t)
fmt.Printf("%+v\n", t)
fmt.Printf("%#v\n", t)
fmt.Printf("%#v\n", timeZone)
```

prints

```go
&{7 -2.35 abc   def}
&{a:7 b:-2.35 c:abc     def}
&main.T{a:7, b:-2.35, c:"abc\tdef"}
map[string] int{"CST":-21600, "PST":-28800, "EST":-18000, "UTC":0, "MST":-25200}
```

(Note the ampersands.) That quoted string format is also available through %q when applied to a value of type string or []byte. The alternate format %#q will use backquotes instead if possible. (The %q format also applies to integers and runes, producing a single-quoted rune constant.) Also, %x works on strings, byte arrays and byte slices as well as on integers, generating a long hexadecimal string, and with a space in the format (% x) it puts spaces between the bytes.

Another handy format is %T, which prints the type of a value.

```go
fmt.Printf("%T\n", timeZone)
```

prints

```go
map[string] int
```

If you want to control the default format for a custom type, all that's required is to define a method with the signature String() string on the type. For our simple type T, that might look like this.

```go
func (t *T) String() string {
    return fmt.Sprintf("%d/%g/%q", t.a, t.b, t.c)
}
fmt.Printf("%v\n", t)
```

to print in the format
```sh
7/-2.35/"abc\tdef"
```

(If you need to print values of type T as well as pointers to T, the receiver for String must be of value type; this example used a pointer because that's more efficient and idiomatic for struct types. See the section below on pointers vs. value receivers for more information.)

Our String method is able to call Sprintf because the print routines are fully reentrant and can be wrapped this way. There is one important detail to understand about this approach, however: don't construct a String method by calling Sprintf in a way that will recur into your String method indefinitely. This can happen if the Sprintf call attempts to print the receiver directly as a string, which in turn will invoke the method again. It's a common and easy mistake to make, as this example shows.

```go
type MyString string

func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", m) // Error: will recur forever.
}
```

It's also easy to fix: convert the argument to the basic string type, which does not have the method.

```go
type MyString string
func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", string(m)) // OK: note conversion.
}
```

In the initialization section we'll see another technique that avoids this recursion.

Another printing technique is to pass a print routine's arguments directly to another such routine. The signature of Printf uses the type ...interface{} for its final argument to specify that an arbitrary number of parameters (of arbitrary type) can appear after the format.

```go
func Printf(format string, v ...interface{}) (n int, err error) {
```

Within the function Printf, v acts like a variable of type []interface{} but if it is passed to another variadic function, it acts like a regular list of arguments. Here is the implementation of the function log.Println we used above. It passes its arguments directly to fmt.Sprintln for the actual formatting.

```go
// Println prints to the standard logger in the manner of fmt.Println.
func Println(v ...interface{}) {
    std.Output(2, fmt.Sprintln(v...))  // Output takes parameters (int, string)
}
```

We write ... after v in the nested call to Sprintln to tell the compiler to treat v as a list of arguments; otherwise it would just pass v as a single slice argument.

There's even more to printing than we've covered here. See the godoc documentation for package fmt for the details.

By the way, a ... parameter can be of a specific type, for instance ...int for a min function that chooses the least of a list of integers:

```go
func Min(a ...int) int {
    min := int(^uint(0) >> 1)  // largest int
    for _, i := range a {
        if i < min {
            min = i
        }
    }
    return min
}
```

### Append <a id="Append"></a>

Now we have the missing piece we needed to explain the design of the append built-in function. The signature of append is different from our custom Append function above. Schematically, it's like this:

func append(slice []T, elements ...T) []T
where T is a placeholder for any given type. You can't actually write a function in Go where the type T is determined by the caller. That's why append is built in: it needs support from the compiler.

What append does is append the elements to the end of the slice and return the result. The result needs to be returned because, as with our hand-written Append, the underlying array may change. This simple example

```go
x := []int{1,2,3}
x = append(x, 4, 5, 6)
fmt.Println(x)
prints [1 2 3 4 5 6].
```

So append works a little like Printf, collecting an arbitrary number of arguments.

But what if we wanted to do what our Append does and append a slice to a slice? Easy: use ... at the call site, just as we did in the call to Output above. This snippet produces identical output to the one above.

```go
x := []int{1,2,3}
y := []int{4,5,6}
x = append(x, y...)
fmt.Println(x)
```

Without that ..., it wouldn't compile because the types would be wrong; y is not of type int.

## Initialisation <a id="Initialization"></a>

Although it doesn't look superficially very different from initialization in C or C++, initialization in Go is more powerful. Complex structures can be built during initialization and the ordering issues among initialized objects, even among different packages, are handled correctly.

### Constantes <a id="Constants"></a>

Constants in Go are just that—constant. They are created at compile time, even when defined as locals in functions, and can only be numbers, characters (runes), strings or booleans. Because of the compile-time restriction, the expressions that define them must be constant expressions, evaluatable by the compiler. For instance, 1<<3 is a constant expression, while math.Sin(math.Pi/4) is not because the function call to math.Sin needs to happen at run time.

In Go, enumerated constants are created using the iota enumerator. Since iota can be part of an expression and expressions can be implicitly repeated, it is easy to build intricate sets of values.

```go
type ByteSize float64

const (
    _           = iota // ignore first value by assigning to blank identifier
    KB ByteSize = 1 << (10 * iota)
    MB
    GB
    TB
    PB
    EB
    ZB
    YB
)
```

The ability to attach a method such as String to any user-defined type makes it possible for arbitrary values to format themselves automatically for printing. Although you'll see it most often applied to structs, this technique is also useful for scalar types such as floating-point types like ByteSize.

```go
func (b ByteSize) String() string {
    switch {
    case b >= YB:
        return fmt.Sprintf("%.2fYB", b/YB)
    case b >= ZB:
        return fmt.Sprintf("%.2fZB", b/ZB)
    case b >= EB:
        return fmt.Sprintf("%.2fEB", b/EB)
    case b >= PB:
        return fmt.Sprintf("%.2fPB", b/PB)
    case b >= TB:
        return fmt.Sprintf("%.2fTB", b/TB)
    case b >= GB:
        return fmt.Sprintf("%.2fGB", b/GB)
    case b >= MB:
        return fmt.Sprintf("%.2fMB", b/MB)
    case b >= KB:
        return fmt.Sprintf("%.2fKB", b/KB)
    }
    return fmt.Sprintf("%.2fB", b)
}
```

The expression YB prints as 1.00YB, while ByteSize(1e13) prints as 9.09TB.

The use here of Sprintf to implement ByteSize's String method is safe (avoids recurring indefinitely) not because of a conversion but because it calls Sprintf with %f, which is not a string format: Sprintf will only call the String method when it wants a string, and %f wants a floating-point value.

### Variables <a id="Variables"></a>

Variables can be initialized just like constants but the initializer can be a general expression computed at run time.

```go
var (
    home   = os.Getenv("HOME")
    user   = os.Getenv("USER")
    gopath = os.Getenv("GOPATH")
)
```

### La fonction init <a id="Theinitfunction"></a>

Finally, each source file can define its own niladic init function to set up whatever state is required. (Actually each file can have multiple init functions.) And finally means finally: init is called after all the variable declarations in the package have evaluated their initializers, and those are evaluated only after all the imported packages have been initialized.

Besides initializations that cannot be expressed as declarations, a common use of init functions is to verify or repair correctness of the program state before real execution begins.

```go
func init() {
    if user == "" {
        log.Fatal("$USER not set")
    }
    if home == "" {
        home = "/home/" + user
    }
    if gopath == "" {
        gopath = home + "/go"
    }
    // gopath may be overridden by --gopath flag on command line.
    flag.StringVar(&gopath, "gopath", gopath, "override default GOPATH")
}
```

## Les méthodes <a id="Methods"></a>

### Pointers vs. Valeurs <a id="Pointersvs.Values"></a>

As we saw with ByteSize, methods can be defined for any named type (except a pointer or an interface); the receiver does not have to be a struct.

In the discussion of slices above, we wrote an Append function. We can define it as a method on slices instead. To do this, we first declare a named type to which we can bind the method, and then make the receiver for the method a value of that type.

```go
type ByteSlice []byte

func (slice ByteSlice) Append(data []byte) []byte {
    // Body exactly the same as above
}
```

This still requires the method to return the updated slice. We can eliminate that clumsiness by redefining the method to take a pointer to a ByteSlice as its receiver, so the method can overwrite the caller's slice.

```go
func (p *ByteSlice) Append(data []byte) {
    slice := *p
    // Body as above, without the return.
    *p = slice
}
```

In fact, we can do even better. If we modify our function so it looks like a standard Write method, like this,

```go
func (p *ByteSlice) Write(data []byte) (n int, err error) {
    slice := *p
    // Again as above.
    *p = slice
    return len(data), nil
}
```

then the type *ByteSlice satisfies the standard interface io.Writer, which is handy. For instance, we can print into one.

```go
var b ByteSlice
fmt.Fprintf(&b, "This hour has %d days\n", 7)
```

We pass the address of a ByteSlice because only *ByteSlice satisfies io.Writer. The rule about pointers vs. values for receivers is that value methods can be invoked on pointers and values, but pointer methods can only be invoked on pointers.

This rule arises because pointer methods can modify the receiver; invoking them on a value would cause the method to receive a copy of the value, so any modifications would be discarded. The language therefore disallows this mistake. There is a handy exception, though. When the value is addressable, the language takes care of the common case of invoking a pointer method on a value by inserting the address operator automatically. In our example, the variable b is addressable, so we can call its Write method with just b.Write. The compiler will rewrite that to (&b).Write for us.

By the way, the idea of using Write on a slice of bytes is central to the implementation of bytes.Buffer.

## Les interfaces et les autres types <a id="Interfacesandothertypes"></a>

### Les interfaces <a id="Interfaces"></a>

Interfaces in Go provide a way to specify the behavior of an object: if something can do this, then it can be used here. We've seen a couple of simple examples already; custom printers can be implemented by a String method while Fprintf can generate output to anything with a Write method. Interfaces with only one or two methods are common in Go code, and are usually given a name derived from the method, such as io.Writer for something that implements Write.

A type can implement multiple interfaces. For instance, a collection can be sorted by the routines in package sort if it implements sort.Interface, which contains Len(), Less(i, j int) bool, and Swap(i, j int), and it could also have a custom formatter. In this contrived example Sequence satisfies both.

```go
type Sequence []int

// Methods required by sort.Interface.
func (s Sequence) Len() int {
    return len(s)
}
func (s Sequence) Less(i, j int) bool {
    return s[i] < s[j]
}
func (s Sequence) Swap(i, j int) {
    s[i], s[j] = s[j], s[i]
}

// Method for printing - sorts the elements before printing.
func (s Sequence) String() string {
    sort.Sort(s)
    str := "["
    for i, elem := range s {
        if i > 0 {
            str += " "
        }
        str += fmt.Sprint(elem)
    }
    return str + "]"
}
```

### Les conversions <a id="Conversions"></a>

The String method of Sequence is recreating the work that Sprint already does for slices. We can share the effort if we convert the Sequence to a plain []int before calling Sprint.

```go
func (s Sequence) String() string {
    sort.Sort(s)
    return fmt.Sprint([]int(s))
}
```

This method is another example of the conversion technique for calling Sprintf safely from a String method. Because the two types (Sequence and []int) are the same if we ignore the type name, it's legal to convert between them. The conversion doesn't create a new value, it just temporarily acts as though the existing value has a new type. (There are other legal conversions, such as from integer to floating point, that do create a new value.)

It's an idiom in Go programs to convert the type of an expression to access a different set of methods. As an example, we could use the existing type sort.IntSlice to reduce the entire example to this:

```go
type Sequence []int

// Method for printing - sorts the elements before printing
func (s Sequence) String() string {
    sort.IntSlice(s).Sort()
    return fmt.Sprint([]int(s))
}
```

Now, instead of having Sequence implement multiple interfaces (sorting and printing), we're using the ability of a data item to be converted to multiple types (Sequence, sort.IntSlice and []int), each of which does some part of the job. That's more unusual in practice but can be effective.

### Conversion d'interface et assertion de type <a id="Interfaceconversionsandtypeassertions"></a>

Type switches are a form of conversion: they take an interface and, for each case in the switch, in a sense convert it to the type of that case. Here's a simplified version of how the code under fmt.Printf turns a value into a string using a type switch. If it's already a string, we want the actual string value held by the interface, while if it has a String method we want the result of calling the method.

```go
type Stringer interface {
    String() string
}

var value interface{} // Value provided by caller.
switch str := value.(type) {
case string:
    return str
case Stringer:
    return str.String()
}
```

The first case finds a concrete value; the second converts the interface into another interface. It's perfectly fine to mix types this way.

What if there's only one type we care about? If we know the value holds a string and we just want to extract it? A one-case type switch would do, but so would a type assertion. A type assertion takes an interface value and extracts from it a value of the specified explicit type. The syntax borrows from the clause opening a type switch, but with an explicit type rather than the type keyword:

```go
value.(typeName)
```

and the result is a new value with the static type typeName. That type must either be the concrete type held by the interface, or a second interface type that the value can be converted to. To extract the string we know is in the value, we could write:

```go
str := value.(string)
```

But if it turns out that the value does not contain a string, the program will crash with a run-time error. To guard against that, use the "comma, ok" idiom to test, safely, whether the value is a string:

```go
str, ok := value.(string)
if ok {
    fmt.Printf("string value is: %q\n", str)
} else {
    fmt.Printf("value is not a string\n")
}
```

If the type assertion fails, str will still exist and be of type string, but it will have the zero value, an empty string.

As an illustration of the capability, here's an if-else statement that's equivalent to the type switch that opened this section.

```go
if str, ok := value.(string); ok {
    return str
} else if str, ok := value.(Stringer); ok {
    return str.String()
}
```

### Generalités <a id="Generality"></a>

If a type exists only to implement an interface and has no exported methods beyond that interface, there is no need to export the type itself. Exporting just the interface makes it clear that it's the behavior that matters, not the implementation, and that other implementations with different properties can mirror the behavior of the original type. It also avoids the need to repeat the documentation on every instance of a common method.

In such cases, the constructor should return an interface value rather than the implementing type. As an example, in the hash libraries both crc32.NewIEEE and adler32.New return the interface type hash.Hash32. Substituting the CRC-32 algorithm for Adler-32 in a Go program requires only changing the constructor call; the rest of the code is unaffected by the change of algorithm.

A similar approach allows the streaming cipher algorithms in the various crypto packages to be separated from the block ciphers they chain together. The Block interface in the crypto/cipher package specifies the behavior of a block cipher, which provides encryption of a single block of data. Then, by analogy with the bufio package, cipher packages that implement this interface can be used to construct streaming ciphers, represented by the Stream interface, without knowing the details of the block encryption.

The crypto/cipher interfaces look like this:

```go
type Block interface {
    BlockSize() int
    Encrypt(src, dst []byte)
    Decrypt(src, dst []byte)
}

type Stream interface {
    XORKeyStream(dst, src []byte)
}
```

Here's the definition of the counter mode (CTR) stream, which turns a block cipher into a streaming cipher; notice that the block cipher's details are abstracted away:

```go
// NewCTR returns a Stream that encrypts/decrypts using the given Block in
// counter mode. The length of iv must be the same as the Block's block size.
func NewCTR(block Block, iv []byte) Stream
```

NewCTR applies not just to one specific encryption algorithm and data source but to any implementation of the Block interface and any Stream. Because they return interface values, replacing CTR encryption with other encryption modes is a localized change. The constructor calls must be edited, but because the surrounding code must treat the result only as a Stream, it won't notice the difference.

### Les interfaces et les méthodes <a id="Interfacesandmethods"></a>

Since almost anything can have methods attached, almost anything can satisfy an interface. One illustrative example is in the http package, which defines the Handler interface. Any object that implements Handler can serve HTTP requests.

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

ResponseWriter is itself an interface that provides access to the methods needed to return the response to the client. Those methods include the standard Write method, so an http.ResponseWriter can be used wherever an io.Writer can be used. Request is a struct containing a parsed representation of the request from the client.

For brevity, let's ignore POSTs and assume HTTP requests are always GETs; that simplification does not affect the way the handlers are set up. Here's a trivial but complete implementation of a handler to count the number of times the page is visited.

```go
// Simple counter server.
type Counter struct {
    n int
}

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ctr.n++
    fmt.Fprintf(w, "counter = %d\n", ctr.n)
}
```

(Keeping with our theme, note how Fprintf can print to an http.ResponseWriter.) For reference, here's how to attach such a server to a node on the URL tree.

```go
import "net/http"
...
ctr := new(Counter)
http.Handle("/counter", ctr)
```

But why make Counter a struct? An integer is all that's needed. (The receiver needs to be a pointer so the increment is visible to the caller.)

```go
// Simpler counter server.
type Counter int

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    *ctr++
    fmt.Fprintf(w, "counter = %d\n", *ctr)
}
```

What if your program has some internal state that needs to be notified that a page has been visited? Tie a channel to the web page.

```go
// A channel that sends a notification on each visit.
// (Probably want the channel to be buffered.)
type Chan chan *http.Request

func (ch Chan) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ch <- req
    fmt.Fprint(w, "notification sent")
}
```

Finally, let's say we wanted to present on /args the arguments used when invoking the server binary. It's easy to write a function to print the arguments.

```go
func ArgServer() {
    fmt.Println(os.Args)
}
```

How do we turn that into an HTTP server? We could make ArgServer a method of some type whose value we ignore, but there's a cleaner way. Since we can define a method for any type except pointers and interfaces, we can write a method for a function. The http package contains this code:

```go
// The HandlerFunc type is an adapter to allow the use of
// ordinary functions as HTTP handlers.  If f is a function
// with the appropriate signature, HandlerFunc(f) is a
// Handler object that calls f.
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(c, req).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, req *Request) {
    f(w, req)
}
```

HandlerFunc is a type with a method, ServeHTTP, so values of that type can serve HTTP requests. Look at the implementation of the method: the receiver is a function, f, and the method calls f. That may seem odd but it's not that different from, say, the receiver being a channel and the method sending on the channel.

To make ArgServer into an HTTP server, we first modify it to have the right signature.

```go
// Argument server.
func ArgServer(w http.ResponseWriter, req *http.Request) {
    fmt.Fprintln(w, os.Args)
}
```

ArgServer now has same signature as HandlerFunc, so it can be converted to that type to access its methods, just as we converted Sequence to IntSlice to access IntSlice.Sort. The code to set it up is concise:

```go
http.Handle("/args", http.HandlerFunc(ArgServer))
```

When someone visits the page /args, the handler installed at that page has value ArgServer and type HandlerFunc. The HTTP server will invoke the method ServeHTTP of that type, with ArgServer as the receiver, which will in turn call ArgServer (via the invocation f(c, req) inside HandlerFunc.ServeHTTP). The arguments will then be displayed.

In this section we have made an HTTP server from a struct, an integer, a channel, and a function, all because interfaces are just sets of methods, which can be defined for (almost) any type.

## L'identifiant vide <a id="Theblankidentifier"></a>

We've mentioned the blank identifier a couple of times now, in the context of for range loops and maps. The blank identifier can be assigned or declared with any value of any type, with the value discarded harmlessly. It's a bit like writing to the Unix /dev/null file: it represents a write-only value to be used as a place-holder where a variable is needed but the actual value is irrelevant. It has uses beyond those we've seen already.

### L'identifiant vide avec plusieurs valeurs d'affectation <a id="Theblankidentifierinmultipleassignment"></a>

The use of a blank identifier in a for range loop is a special case of a general situation: multiple assignment.

If an assignment requires multiple values on the left side, but one of the values will not be used by the program, a blank identifier on the left-hand-side of the assignment avoids the need to create a dummy variable and makes it clear that the value is to be discarded. For instance, when calling a function that returns a value and an error, but only the error is important, use the blank identifier to discard the irrelevant value.

```go
if _, err := os.Stat(path); os.IsNotExist(err) {
	fmt.Printf("%s does not exist\n", path)
}
```

Occasionally you'll see code that discards the error value in order to ignore the error; this is terrible practice. Always check error returns; they're provided for a reason.

```go
// Bad! This code will crash if path does not exist.
fi, _ := os.Stat(path)
if fi.IsDir() {
    fmt.Printf("%s is a directory\n", path)
}
```

### Imports et variables non utilisés <a id="Unusedimportsandvariables"></a>

It is an error to import a package or to declare a variable without using it. Unused imports bloat the program and slow compilation, while a variable that is initialized but not used is at least a wasted computation and perhaps indicative of a larger bug. When a program is under active development, however, unused imports and variables often arise and it can be annoying to delete them just to have the compilation proceed, only to have them be needed again later. The blank identifier provides a workaround.

This half-written program has two unused imports (fmt and io) and an unused variable (fd), so it will not compile, but it would be nice to see if the code so far is correct.

```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
}
```

To silence complaints about the unused imports, use a blank identifier to refer to a symbol from the imported package. Similarly, assigning the unused variable fd to the blank identifier will silence the unused variable error. This version of the program does compile.

```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

var _ = fmt.Printf // For debugging; delete when done.
var _ io.Reader    // For debugging; delete when done.

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
    _ = fd
}
```

By convention, the global declarations to silence import errors should come right after the imports and be commented, both to make them easy to find and as a reminder to clean things up later.

### Les éffets de bord d'import <a id="Importforsideeffect"></a>

An unused import like fmt or io in the previous example should eventually be used or removed: blank assignments identify code as a work in progress. But sometimes it is useful to import a package only for its side effects, without any explicit use. For example, during its init function, the net/http/pprof package registers HTTP handlers that provide debugging information. It has an exported API, but most clients need only the handler registration and access the data through a web page. To import the package only for its side effects, rename the package to the blank identifier:

import _ "net/http/pprof"
This form of import makes clear that the package is being imported for its side effects, because there is no other possible use of the package: in this file, it doesn't have a name. (If it did, and we didn't use that name, the compiler would reject the program.)

### Validation d'interface <a id="Interfacechecks"></a>

As we saw in the discussion of interfaces above, a type need not declare explicitly that it implements an interface. Instead, a type implements the interface just by implementing the interface's methods. In practice, most interface conversions are static and therefore checked at compile time. For example, passing an *os.File to a function expecting an io.Reader will not compile unless *os.File implements the io.Reader interface.

Some interface checks do happen at run-time, though. One instance is in the encoding/json package, which defines a Marshaler interface. When the JSON encoder receives a value that implements that interface, the encoder invokes the value's marshaling method to convert it to JSON instead of doing the standard conversion. The encoder checks this property at run time with a type assertion like:

```go
m, ok := val.(json.Marshaler)
```


If it's necessary only to ask whether a type implements an interface, without actually using the interface itself, perhaps as part of an error check, use the blank identifier to ignore the type-asserted value:

```go
if _, ok := val.(json.Marshaler); ok {
    fmt.Printf("value %v of type %T implements json.Marshaler\n", val, val)
}
```

One place this situation arises is when it is necessary to guarantee within the package implementing the type that it actually satisfies the interface. If a type—for example, json.RawMessage—needs a custom JSON representation, it should implement json.Marshaler, but there are no static conversions that would cause the compiler to verify this automatically. If the type inadvertently fails to satisfy the interface, the JSON encoder will still work, but will not use the custom implementation. To guarantee that the implementation is correct, a global declaration using the blank identifier can be used in the package:

```go
var _ json.Marshaler = (*RawMessage)(nil)
```

In this declaration, the assignment involving a conversion of a *RawMessage to a Marshaler requires that *RawMessage implements Marshaler, and that property will be checked at compile time. Should the json.Marshaler interface change, this package will no longer compile and we will be on notice that it needs to be updated.

The appearance of the blank identifier in this construct indicates that the declaration exists only for the type checking, not to create a variable. Don't do this for every type that satisfies an interface, though. By convention, such declarations are only used when there are no static conversions already present in the code, which is a rare event.

## Incorporation <a id="Embedding"></a>

Go does not provide the typical, type-driven notion of subclassing, but it does have the ability to “borrow” pieces of an implementation by embedding types within a struct or interface.

Interface embedding is very simple. We've mentioned the io.Reader and io.Writer interfaces before; here are their definitions.

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}
```

The io package also exports several other interfaces that specify objects that can implement several such methods. For instance, there is io.ReadWriter, an interface containing both Read and Write. We could specify io.ReadWriter by listing the two methods explicitly, but it's easier and more evocative to embed the two interfaces to form the new one, like this:

```go
// ReadWriter is the interface that combines the Reader and Writer interfaces.
type ReadWriter interface {
    Reader
    Writer
}
```

This says just what it looks like: A ReadWriter can do what a Reader does and what a Writer does; it is a union of the embedded interfaces (which must be disjoint sets of methods). Only interfaces can be embedded within interfaces.

The same basic idea applies to structs, but with more far-reaching implications. The bufio package has two struct types, bufio.Reader and bufio.Writer, each of which of course implements the analogous interfaces from package io. And bufio also implements a buffered reader/writer, which it does by combining a reader and a writer into one struct using embedding: it lists the types within the struct but does not give them field names.

```go
// ReadWriter stores pointers to a Reader and a Writer.
// It implements io.ReadWriter.
type ReadWriter struct {
    *Reader  // *bufio.Reader
    *Writer  // *bufio.Writer
}
```

The embedded elements are pointers to structs and of course must be initialized to point to valid structs before they can be used. The ReadWriter struct could be written as

```go
type ReadWriter struct {
    reader *Reader
    writer *Writer
}
```

but then to promote the methods of the fields and to satisfy the io interfaces, we would also need to provide forwarding methods, like this:

```go
func (rw *ReadWriter) Read(p []byte) (n int, err error) {
    return rw.reader.Read(p)
}
```

By embedding the structs directly, we avoid this bookkeeping. The methods of embedded types come along for free, which means that bufio.ReadWriter not only has the methods of bufio.Reader and bufio.Writer, it also satisfies all three interfaces: io.Reader, io.Writer, and io.ReadWriter.

There's an important way in which embedding differs from subclassing. When we embed a type, the methods of that type become methods of the outer type, but when they are invoked the receiver of the method is the inner type, not the outer one. In our example, when the Read method of a bufio.ReadWriter is invoked, it has exactly the same effect as the forwarding method written out above; the receiver is the reader field of the ReadWriter, not the ReadWriter itself.

Embedding can also be a simple convenience. This example shows an embedded field alongside a regular, named field.

```go
type Job struct {
    Command string
    *log.Logger
}
```

The Job type now has the Log, Logf and other methods of *log.Logger. We could have given the Logger a field name, of course, but it's not necessary to do so. And now, once initialized, we can log to the Job:

```go
job.Log("starting now...")
```

The Logger is a regular field of the Job struct, so we can initialize it in the usual way inside the constructor for Job, like this,

```go
func NewJob(command string, logger *log.Logger) *Job {
    return &Job{command, logger}
}
```

or with a composite literal,

```go
job := &Job{command, log.New(os.Stderr, "Job: ", log.Ldate)}
```

If we need to refer to an embedded field directly, the type name of the field, ignoring the package qualifier, serves as a field name, as it did in the Read method of our ReaderWriter struct. Here, if we needed to access the *log.Logger of a Job variable job, we would write job.Logger, which would be useful if we wanted to refine the methods of Logger.

```go
func (job *Job) Logf(format string, args ...interface{}) {
    job.Logger.Logf("%q: %s", job.Command, fmt.Sprintf(format, args...))
}
```

Embedding types introduces the problem of name conflicts but the rules to resolve them are simple. First, a field or method X hides any other item X in a more deeply nested part of the type. If log.Logger contained a field or method called Command, the Command field of Job would dominate it.

Second, if the same name appears at the same nesting level, it is usually an error; it would be erroneous to embed log.Logger if the Job struct contained another field or method called Logger. However, if the duplicate name is never mentioned in the program outside the type definition, it is OK. This qualification provides some protection against changes made to types embedded from outside; there is no problem if a field is added that conflicts with another field in another subtype if neither field is ever used.

## Concurrence <a id="Concurrency"></a>

### Partager en communquant <a id="Sharebycommunicating"></a>

Concurrent programming is a large topic and there is space only for some Go-specific highlights here.

Concurrent programming in many environments is made difficult by the subtleties required to implement correct access to shared variables. Go encourages a different approach in which shared values are passed around on channels and, in fact, never actively shared by separate threads of execution. Only one goroutine has access to the value at any given time. Data races cannot occur, by design. To encourage this way of thinking we have reduced it to a slogan:

Do not communicate by sharing memory; instead, share memory by communicating.
This approach can be taken too far. Reference counts may be best done by putting a mutex around an integer variable, for instance. But as a high-level approach, using channels to control access makes it easier to write clear, correct programs.

One way to think about this model is to consider a typical single-threaded program running on one CPU. It has no need for synchronization primitives. Now run another such instance; it too needs no synchronization. Now let those two communicate; if the communication is the synchronizer, there's still no need for other synchronization. Unix pipelines, for example, fit this model perfectly. Although Go's approach to concurrency originates in Hoare's Communicating Sequential Processes (CSP), it can also be seen as a type-safe generalization of Unix pipes.

Goroutines

They're called goroutines because the existing terms—threads, coroutines, processes, and so on—convey inaccurate connotations. A goroutine has a simple model: it is a function executing concurrently with other goroutines in the same address space. It is lightweight, costing little more than the allocation of stack space. And the stacks start small, so they are cheap, and grow by allocating (and freeing) heap storage as required.

Goroutines are multiplexed onto multiple OS threads so if one should block, such as while waiting for I/O, others continue to run. Their design hides many of the complexities of thread creation and management.

Prefix a function or method call with the go keyword to run the call in a new goroutine. When the call completes, the goroutine exits, silently. (The effect is similar to the Unix shell's & notation for running a command in the background.)
```go
go list.Sort()  // run list.Sort concurrently; don't wait for it.
```

A function literal can be handy in a goroutine invocation.

```go
func Announce(message string, delay time.Duration) {
    go func() {
        time.Sleep(delay)
        fmt.Println(message)
    }()  // Note the parentheses - must call the function.
}
```

In Go, function literals are closures: the implementation makes sure the variables referred to by the function survive as long as they are active.

These examples aren't too practical because the functions have no way of signaling completion. For that, we need channels.

### Les canaux <a id="Channels"></a>

Like maps, channels are allocated with make, and the resulting value acts as a reference to an underlying data structure. If an optional integer parameter is provided, it sets the buffer size for the channel. The default is zero, for an unbuffered or synchronous channel.

```go
ci := make(chan int)            // unbuffered channel of integers
cj := make(chan int, 0)         // unbuffered channel of integers
cs := make(chan *os.File, 100)  // buffered channel of pointers to Files
```

Unbuffered channels combine communication—the exchange of a value—with synchronization—guaranteeing that two calculations (goroutines) are in a known state.

There are lots of nice idioms using channels. Here's one to get us started. In the previous section we launched a sort in the background. A channel can allow the launching goroutine to wait for the sort to complete.

```go
c := make(chan int)  // Allocate a channel.
// Start the sort in a goroutine; when it completes, signal on the channel.
go func() {
    list.Sort()
    c <- 1  // Send a signal; value does not matter.
}()
doSomethingForAWhile()
<-c   // Wait for sort to finish; discard sent value.
```

Receivers always block until there is data to receive. If the channel is unbuffered, the sender blocks until the receiver has received the value. If the channel has a buffer, the sender blocks only until the value has been copied to the buffer; if the buffer is full, this means waiting until some receiver has retrieved a value.

A buffered channel can be used like a semaphore, for instance to limit throughput. In this example, incoming requests are passed to handle, which sends a value into the channel, processes the request, and then receives a value from the channel to ready the “semaphore” for the next consumer. The capacity of the channel buffer limits the number of simultaneous calls to process.

```go
var sem = make(chan int, MaxOutstanding)

func handle(r *Request) {
    sem <- 1    // Wait for active queue to drain.
    process(r)  // May take a long time.
    <-sem       // Done; enable next request to run.
}

func Serve(queue chan *Request) {
    for {
        req := <-queue
        go handle(req)  // Don't wait for handle to finish.
    }
}
```

Once MaxOutstanding handlers are executing process, any more will block trying to send into the filled channel buffer, until one of the existing handlers finishes and receives from the buffer.

This design has a problem, though: Serve creates a new goroutine for every incoming request, even though only MaxOutstanding of them can run at any moment. As a result, the program can consume unlimited resources if the requests come in too fast. We can address that deficiency by changing Serve to gate the creation of the goroutines. Here's an obvious solution, but beware it has a bug we'll fix subsequently:

```go
func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1
        go func() {
            process(req) // Buggy; see explanation below.
            <-sem
        }()
    }
}
```

The bug is that in a Go for loop, the loop variable is reused for each iteration, so the req variable is shared across all goroutines. That's not what we want. We need to make sure that req is unique for each goroutine. Here's one way to do that, passing the value of req as an argument to the closure in the goroutine:

```go
func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1
        go func(req *Request) {
            process(req)
            <-sem
        }(req)
    }
}
```

Compare this version with the previous to see the difference in how the closure is declared and run. Another solution is just to create a new variable with the same name, as in this example:

```go
func Serve(queue chan *Request) {
    for req := range queue {
        req := req // Create new instance of req for the goroutine.
        sem <- 1
        go func() {
            process(req)
            <-sem
        }()
    }
}
```

It may seem odd to write

```go
req := req
```

but it's legal and idiomatic in Go to do this. You get a fresh version of the variable with the same name, deliberately shadowing the loop variable locally but unique to each goroutine.

Going back to the general problem of writing the server, another approach that manages resources well is to start a fixed number of handle goroutines all reading from the request channel. The number of goroutines limits the number of simultaneous calls to process. This Serve function also accepts a channel on which it will be told to exit; after launching the goroutines it blocks receiving from that channel.

```go
func handle(queue chan *Request) {
    for r := range queue {
        process(r)
    }
}

func Serve(clientRequests chan *Request, quit chan bool) {
    // Start handlers
    for i := 0; i < MaxOutstanding; i++ {
        go handle(clientRequests)
    }
    <-quit  // Wait to be told to exit.
}
```

### Les canaux de canaux

One of the most important properties of Go is that a channel is a first-class value that can be allocated and passed around like any other. A common use of this property is to implement safe, parallel demultiplexing.

In the example in the previous section, handle was an idealized handler for a request but we didn't define the type it was handling. If that type includes a channel on which to reply, each client can provide its own path for the answer. Here's a schematic definition of type Request.

```go
type Request struct {
    args        []int
    f           func([]int) int
    resultChan  chan int
}
```

The client provides a function and its arguments, as well as a channel inside the request object on which to receive the answer.

```go
func sum(a []int) (s int) {
    for _, v := range a {
        s += v
    }
    return
}

request := &Request{[]int{3, 4, 5}, sum, make(chan int)}
// Send request
clientRequests <- request
// Wait for response.
fmt.Printf("answer: %d\n", <-request.resultChan)
```

On the server side, the handler function is the only thing that changes.
```go
func handle(queue chan *Request) {
    for req := range queue {
        req.resultChan <- req.f(req.args)
    }
}
```

There's clearly a lot more to do to make it realistic, but this code is a framework for a rate-limited, parallel, non-blocking RPC system, and there's not a mutex in sight.

### La parallelisation  <a id="Parallelization"></a>

Another application of these ideas is to parallelize a calculation across multiple CPU cores. If the calculation can be broken into separate pieces that can execute independently, it can be parallelized, with a channel to signal when each piece completes.

Let's say we have an expensive operation to perform on a vector of items, and that the value of the operation on each item is independent, as in this idealized example.

```go
type Vector []float64

// Apply the operation to v[i], v[i+1] ... up to v[n-1].
func (v Vector) DoSome(i, n int, u Vector, c chan int) {
    for ; i < n; i++ {
        v[i] += u.Op(v[i])
    }
    c <- 1    // signal that this piece is done
}
```

We launch the pieces independently in a loop, one per CPU. They can complete in any order but it doesn't matter; we just count the completion signals by draining the channel after launching all the goroutines.

```go
const numCPU = 4 // number of CPU cores

func (v Vector) DoAll(u Vector) {
    c := make(chan int, numCPU)  // Buffering optional but sensible.
    for i := 0; i < numCPU; i++ {
        go v.DoSome(i*len(v)/numCPU, (i+1)*len(v)/numCPU, u, c)
    }
    // Drain the channel.
    for i := 0; i < numCPU; i++ {
        <-c    // wait for one task to complete
    }
    // All done.
}
```

Rather than create a constant value for numCPU, we can ask the runtime what value is appropriate. The function runtime.NumCPU returns the number of hardware CPU cores in the machine, so we could write

```go
var numCPU = runtime.NumCPU()
```

There is also a function runtime.GOMAXPROCS, which reports (or sets) the user-specified number of cores that a Go program can have running simultaneously. It defaults to the value of runtime.NumCPU but can be overridden by setting the similarly named shell environment variable or by calling the function with a positive number. Calling it with zero just queries the value. Therefore if we want to honor the user's resource request, we should write

```go
var numCPU = runtime.GOMAXPROCS(0)
```

Be sure not to confuse the ideas of concurrency—structuring a program as independently executing components—and parallelism—executing calculations in parallel for efficiency on multiple CPUs. Although the concurrency features of Go can make some problems easy to structure as parallel computations, Go is a concurrent language, not a parallel one, and not all parallelization problems fit Go's model. For a discussion of the distinction, see the talk cited in this blog post.

### Fuite mémoire sur un buffer <a id="Aleakybuffer"></a>

The tools of concurrent programming can even make non-concurrent ideas easier to express. Here's an example abstracted from an RPC package. The client goroutine loops receiving data from some source, perhaps a network. To avoid allocating and freeing buffers, it keeps a free list, and uses a buffered channel to represent it. If the channel is empty, a new buffer gets allocated. Once the message buffer is ready, it's sent to the server on serverChan.

```go
var freeList = make(chan *Buffer, 100)
var serverChan = make(chan *Buffer)

func client() {
    for {
        var b *Buffer
        // Grab a buffer if available; allocate if not.
        select {
        case b = <-freeList:
            // Got one; nothing more to do.
        default:
            // None free, so allocate a new one.
            b = new(Buffer)
        }
        load(b)              // Read next message from the net.
        serverChan <- b      // Send to server.
    }
}
```

The server loop receives each message from the client, processes it, and returns the buffer to the free list.

```go
func server() {
    for {
        b := <-serverChan    // Wait for work.
        process(b)
        // Reuse buffer if there's room.
        select {
        case freeList <- b:
            // Buffer on free list; nothing more to do.
        default:
            // Free list full, just carry on.
        }
    }
}
```

The client attempts to retrieve a buffer from freeList; if none is available, it allocates a fresh one. The server's send to freeList puts b back on the free list unless the list is full, in which case the buffer is dropped on the floor to be reclaimed by the garbage collector. (The default clauses in the select statements execute when no other case is ready, meaning that the selects never block.) This implementation builds a leaky bucket free list in just a few lines, relying on the buffered channel and the garbage collector for bookkeeping.

## Errors <a id="Errors"></a>

Library routines must often return some sort of error indication to the caller. As mentioned earlier, Go's multivalue return makes it easy to return a detailed error description alongside the normal return value. It is good style to use this feature to provide detailed error information. For example, as we'll see, os.Open doesn't just return a nil pointer on failure, it also returns an error value that describes what went wrong.

By convention, errors have type error, a simple built-in interface.

```go
type error interface {
    Error() string
}
```

A library writer is free to implement this interface with a richer model under the covers, making it possible not only to see the error but also to provide some context. As mentioned, alongside the usual *os.File return value, os.Open also returns an error value. If the file is opened successfully, the error will be nil, but when there is a problem, it will hold an os.PathError:

```go
// PathError records an error and the operation and
// file path that caused it.
type PathError struct {
    Op string    // "open", "unlink", etc.
    Path string  // The associated file.
    Err error    // Returned by the system call.
}

func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```

PathError's Error generates a string like this:

```sh
open /etc/passwx: no such file or directory
```

Such an error, which includes the problematic file name, the operation, and the operating system error it triggered, is useful even if printed far from the call that caused it; it is much more informative than the plain "no such file or directory".

When feasible, error strings should identify their origin, such as by having a prefix naming the operation or package that generated the error. For example, in package image, the string representation for a decoding error due to an unknown format is "image: unknown format".

Callers that care about the precise error details can use a type switch or a type assertion to look for specific errors and extract details. For PathErrors this might include examining the internal Err field for recoverable failures.

```go
for try := 0; try < 2; try++ {
    file, err = os.Create(filename)
    if err == nil {
        return
    }
    if e, ok := err.(*os.PathError); ok && e.Err == syscall.ENOSPC {
        deleteTempFiles()  // Recover some space.
        continue
    }
    return
}
```

The second if statement here is another type assertion. If it fails, ok will be false, and e will be nil. If it succeeds, ok will be true, which means the error was of type *os.PathError, and then so is e, which we can examine for more information about the error.

### Panic <a id="Panic"></a>

The usual way to report an error to a caller is to return an error as an extra return value. The canonical Read method is a well-known instance; it returns a byte count and an error. But what if the error is unrecoverable? Sometimes the program simply cannot continue.

For this purpose, there is a built-in function panic that in effect creates a run-time error that will stop the program (but see the next section). The function takes a single argument of arbitrary type—often a string—to be printed as the program dies. It's also a way to indicate that something impossible has happened, such as exiting an infinite loop.

```go
// A toy implementation of cube root using Newton's method.
func CubeRoot(x float64) float64 {
    z := x/3   // Arbitrary initial value
    for i := 0; i < 1e6; i++ {
        prevz := z
        z -= (z*z*z-x) / (3*z*z)
        if veryClose(z, prevz) {
            return z
        }
    }
    // A million iterations has not converged; something is wrong.
    panic(fmt.Sprintf("CubeRoot(%g) did not converge", x))
}
```

This is only an example but real library functions should avoid panic. If the problem can be masked or worked around, it's always better to let things continue to run rather than taking down the whole program. One possible counterexample is during initialization: if the library truly cannot set itself up, it might be reasonable to panic, so to speak.

```go
var user = os.Getenv("USER")

func init() {
    if user == "" {
        panic("no value for $USER")
    }
}
```

### Recover <a id="Recover"></a>

When panic is called, including implicitly for run-time errors such as indexing a slice out of bounds or failing a type assertion, it immediately stops execution of the current function and begins unwinding the stack of the goroutine, running any deferred functions along the way. If that unwinding reaches the top of the goroutine's stack, the program dies. However, it is possible to use the built-in function recover to regain control of the goroutine and resume normal execution.

A call to recover stops the unwinding and returns the argument passed to panic. Because the only code that runs while unwinding is inside deferred functions, recover is only useful inside deferred functions.

One application of recover is to shut down a failing goroutine inside a server without killing the other executing goroutines.

```go
func server(workChan <-chan *Work) {
    for work := range workChan {
        go safelyDo(work)
    }
}

func safelyDo(work *Work) {
    defer func() {
        if err := recover(); err != nil {
            log.Println("work failed:", err)
        }
    }()
    do(work)
}
```

In this example, if do(work) panics, the result will be logged and the goroutine will exit cleanly without disturbing the others. There's no need to do anything else in the deferred closure; calling recover handles the condition completely.

Because recover always returns nil unless called directly from a deferred function, deferred code can call library routines that themselves use panic and recover without failing. As an example, the deferred function in safelyDo might call a logging function before calling recover, and that logging code would run unaffected by the panicking state.

With our recovery pattern in place, the do function (and anything it calls) can get out of any bad situation cleanly by calling panic. We can use that idea to simplify error handling in complex software. Let's look at an idealized version of a regexp package, which reports parsing errors by calling panic with a local error type. Here's the definition of Error, an error method, and the Compile function.

```go
// Error is the type of a parse error; it satisfies the error interface.
type Error string
func (e Error) Error() string {
    return string(e)
}

// error is a method of *Regexp that reports parsing errors by
// panicking with an Error.
func (regexp *Regexp) error(err string) {
    panic(Error(err))
}

// Compile returns a parsed representation of the regular expression.
func Compile(str string) (regexp *Regexp, err error) {
    regexp = new(Regexp)
    // doParse will panic if there is a parse error.
    defer func() {
        if e := recover(); e != nil {
            regexp = nil    // Clear return value.
            err = e.(Error) // Will re-panic if not a parse error.
        }
    }()
    return regexp.doParse(str), nil
}
```

If doParse panics, the recovery block will set the return value to nil—deferred functions can modify named return values. It will then check, in the assignment to err, that the problem was a parse error by asserting that it has the local type Error. If it does not, the type assertion will fail, causing a run-time error that continues the stack unwinding as though nothing had interrupted it. This check means that if something unexpected happens, such as an index out of bounds, the code will fail even though we are using panic and recover to handle parse errors.

With error handling in place, the error method (because it's a method bound to a type, it's fine, even natural, for it to have the same name as the builtin error type) makes it easy to report parse errors without worrying about unwinding the parse stack by hand:

```go
if pos == 0 {
    re.error("'*' illegal at start of expression")
}
```

Useful though this pattern is, it should be used only within a package. Parse turns its internal panic calls into error values; it does not expose panics to its client. That is a good rule to follow.

By the way, this re-panic idiom changes the panic value if an actual error occurs. However, both the original and new failures will be presented in the crash report, so the root cause of the problem will still be visible. Thus this simple re-panic approach is usually sufficient—it's a crash after all—but if you want to display only the original value, you can write a little more code to filter unexpected problems and re-panic with the original error. That's left as an exercise for the reader.

## Un serveur web <a id="Awebserver"></a>

Let's finish with a complete Go program, a web server. This one is actually a kind of web re-server. Google provides a service at http://chart.apis.google.com that does automatic formatting of data into charts and graphs. It's hard to use interactively, though, because you need to put the data into the URL as a query. The program here provides a nicer interface to one form of data: given a short piece of text, it calls on the chart server to produce a QR code, a matrix of boxes that encode the text. That image can be grabbed with your cell phone's camera and interpreted as, for instance, a URL, saving you typing the URL into the phone's tiny keyboard.

Here's the complete program. An explanation follows.

```go
package main

import (
    "flag"
    "html/template"
    "log"
    "net/http"
)

var addr = flag.String("addr", ":1718", "http service address") // Q=17, R=18

var templ = template.Must(template.New("qr").Parse(templateStr))

func main() {
    flag.Parse()
    http.Handle("/", http.HandlerFunc(QR))
    err := http.ListenAndServe(*addr, nil)
    if err != nil {
        log.Fatal("ListenAndServe:", err)
    }
}

func QR(w http.ResponseWriter, req *http.Request) {
    templ.Execute(w, req.FormValue("s"))
}

const templateStr = `
<html>
<head>
<title>QR Link Generator</title>
</head>
<body>
{{if .}}
<img src="http://chart.apis.google.com/chart?chs=300x300&cht=qr&choe=UTF-8&chl={{.}}" />
<br>
{{.}}
<br>
<br>
{{end}}
<form action="/" name=f method="GET"><input maxLength=1024 size=70
name=s value="" title="Text to QR Encode"><input type=submit
value="Show QR" name=qr>
</form>
</body>
</html>
`
```

The pieces up to main should be easy to follow. The one flag sets a default HTTP port for our server. The template variable templ is where the fun happens. It builds an HTML template that will be executed by the server to display the page; more about that in a moment.

The main function parses the flags and, using the mechanism we talked about above, binds the function QR to the root path for the server. Then http.ListenAndServe is called to start the server; it blocks while the server runs.

QR just receives the request, which contains form data, and executes the template on the data in the form value named s.

The template package html/template is powerful; this program just touches on its capabilities. In essence, it rewrites a piece of HTML text on the fly by substituting elements derived from data items passed to templ.Execute, in this case the form value. Within the template text (templateStr), double-brace-delimited pieces denote template actions. The piece from {{if .}} to {{end}} executes only if the value of the current data item, called . (dot), is non-empty. That is, when the string is empty, this piece of the template is suppressed.

The two snippets {{.}} say to show the data presented to the template—the query string—on the web page. The HTML template package automatically provides appropriate escaping so the text is safe to display.

The rest of the template string is just the HTML to show when the page loads. If this is too quick an explanation, see the documentation for the template package for a more thorough discussion.

And there you have it: a useful web server in a few lines of code plus some data-driven HTML text. Go is powerful enough to make a lot happen in a few lines.

Build version go1.6.3.
Except as noted, the content of this page is licensed under the Creative Commons Attribution 3.0 License, and code is licensed under a BSD license.
Terms of Service | Privacy Policy
