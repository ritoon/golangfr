# Go Slices: usages et fonctionnement interne

5 Janvier 2011

## Introduction

Le type slice dans **Go** offre un moyen pratique et efficace de travailler avec des séquences de données typées. Les slices sont analogues aux tableaux dans d'autres langues, mais ont quelques propriétés inhabituelles. Cet article se penchera sur ce que sont les slices et comment elles sont utilisées.

## Tableaux

Le type slice est une abstraction construite au-dessus du type tableau de **Go**, c'est pourquoi, avant de comprendre les slices, nous devons d'abord comprendre les tableaux.

Une définition du type tableau spécifie une longueur et un type d'élément. Par exemple, le type `[4]int` représente un tableau de quatre nombres entiers. La taille d'un tableau est fixe ; sa longueur est une partie de son type (`[4]int` et `[5]int` sont distincts et de types incompatibles). Les tableaux peuvent être indexés de la manière habituelle, de sorte que l'expression `s[n]` accède au nième élément, à partir de zéro.

```go
var a [4]int
a[0] = 1
i := a[0]
// i == 1
```

Les tableaux n'ont pas besoin d’être initialisé de manière explicite ; la valeur zéro d'un tableau est un tableau prêt à l'emploi, dont les éléments sont eux-mêmes mis à zéro :

```go
// a[2] == 0, la valeur zéro du type int
```

La représentation en mémoire de `[4]int` est à seulement quatre valeurs entières fixées séquentiellement :

![](http://blog.golang.org/go-slices-usage-and-internals_slice-array.png)

Les tableaux dans **Go** sont des valeurs. Une variable tableau représente l'ensemble du tableau ; il n'est pas un pointeur vers le le tableau (comme ce serait le cas en C). Cela signifie que lorsque vous affectez ou passer en paramètre un tableau, vous faites une copie de son contenu (pour éviter la copie, vous pouvez passer un pointeur vers le tableau, mais qui est un pointeur vers un tableau, pas un tableau). Une façon de penser les tableaux est de le voir comme un type de structure composé de champs indexés plutôt que des champs nommés : une taille fixe de valeur composite.

Un tableau littéral peut être spécifié comme ceci :

```go
b := [2]string{"Penn", "Teller"}
```

Ou, vous pouvez laisser le compilateur compter les éléments du tableau pour vous :

```go
b := [...]string{"Penn", "Teller"}
```

Dans les deux cas, le type de `b` est `[2]string`.

## slices

Les tableaux ont leur place, mais ils sont un peu rigides, de sorte que vous ne les voyez pas trop souvent dans du code **Go**. Les slices, quant à elles, sont partout. Elles sont construites par dessus les tableaux pour fournir une grande puissance et ajoute de la souplesse d'utilisation.

La spécification de type pour une slice est `[]T` , où `T` est le type des éléments de la slice. Contrairement à un type tableau, un type de slice n'a pas de longueur spécifiée.

Une slice littérale est déclarée comme un tableau littéral, à l'exception que vous laissez vide le nombre d'éléments :

```go
letters := []string{"a", "b", "c", "d"}
```

Une slice peut être créée avec la fonction intégrée de Go appelée `make` qui a la signature suivante :

```go
func make([]T, len, cap) []T
```

Où `T` représente le type de la slice de l'élément à créer. La fonction `make` prend un type, une longueur et une capacité optionnelle. Lorsqu'elle est appelée, `make` alloue un tableau et renvoie une slice qui fait référence à ce tableau.

```go
var s []byte
s = make([]byte, 5, 5)
// s == []byte{0, 0, 0, 0, 0}
```
Lorsque l'argument de la capacité est omis, par défaut la longueur la spécifie. Voici une version plus succincte du même code :

```go
s := make([]byte, 5)
```

La longueur et la capacité d'une slice peuvent être inspectées en utilisant les fonctions intégrées `len` et `cap`.

```go
len(s) == 5
cap(s) == 5
```

### Les deux sections suivantes traitent de la relation entre la longueur et de la capacité.

La valeur zéro d'une slice est `nil`. Les fonctions de `len` et `cap` retournent toutes deux `0` pour une slice à `nil`.

Une slice peut également être formée par "slicing" d'une slice ou d'un tableau existant. Le slicing est réalisé en spécifiant une plage semi-ouverte avec deux indices séparés par deux points. Par exemple, l'expression `b[1:4]` crée une slice comprenant des éléments 1 à 3 de `b` (les éléments de la slice résultant seront de 0 à 2).

```go
b := []byte{'g', 'o', 'l', 'a', 'n', 'g'}
// b[1:4] == []byte{'o', 'l', 'a'}, partageant la même mémoire que b
```

Les indices de début et de fin d'une expression de slice sont facultatifs ; elles sont par défaut à zéro et la longueur de la slice respectivement :

```go
// b[:2] == []byte{'g', 'o'}
// b[2:] == []byte{'l', 'a', 'n', 'g'}
// b[:] == b
```
Ceci est également la syntaxe pour créer une slice donnée d'un tableau :

```go
x := [3]string{"Лайка", "Белка", "Стрелка"}
s := x[:] // une slice faisant référence au stockage de x
```

## Fonctionnement interne des slices 

Une slice est un descripteur d'un segment de tableau. Il se compose d'un pointeur sur le tableau, la longueur du segment et de sa capacité (la longueur maximale du segment).

![](http://blog.golang.org/go-slices-usage-and-internals_slice-struct.png)


Notre variable `s`, créée précédemment par `make([]byte, 5)`, est structurée comme suit :

![](http://blog.golang.org/go-slices-usage-and-internals_slice-1.png)

La longueur est le nombre d'éléments référencés par la slice. La capacité est le nombre d'éléments dans le tableau sous-jacent (à partir de l'élément visé par le pointeur de la slice). La distinction entre la longueur et la capacité deviendra claire après les quelques exemples suivants.

Si nous sliçons `s` , observez les changements dans la structure de données de slice et leur relation avec le tableau sous-jacent :

```go
s = s[2:4]
```

![](http://blog.golang.org/go-slices-usage-and-internals_slice-2.png)

Slicer ne copie pas les données de la slice. **Go** crée une nouvelle valeur de slice qui pointe vers le tableau d'origine. Cela rend les opérations de slice aussi efficaces que la manipulation d'indices de tableau. Par conséquent, la modification des éléments (non la slice elle-même) d'une slice modifie les éléments de la slice d'origine :


```go
d := []byte{'r', 'o', 'a', 'd'}
e := d[2:]
// e == []byte{'a', 'd'}
e[1] = 'm'
// e == []byte{'a', 'm'}
// d == []byte{'r', 'o', 'a', 'm'}
```
[play](https://play.golang.org/p/ACsYFbgzo_)

Précédemment, nous avons slicé `s` sur une longueur plus courte que sa capacité. Nous pouvons aussi agrandir la capacité de `s` par un nouveau slicing :


```go
s = s[:cap(s)]
```

![](http://blog.golang.org/go-slices-usage-and-internals_slice-3.png)

Une slice ne peut s'agrandir au-delà de sa capacité. Tenter de le faire va provoquer une panique du runtime, tout comme lors de l'indexation en dehors des limites d'une slice ou d'un tableau. De même, les slices ne peuvent pas être re-slicées en dessous de zéro pour accéder aux éléments précédents dans le tableau.

## Agrandir une slice (les fonctions copy et append)

Pour augmenter la capacité d'une slice on doit d'abord en créer une nouvelle plus grande puis copier le contenu de la slice originelle en elle. Cette technique permet de savoir comment fonctionnent en arrière-plan les tableaux dynamiques. L'exemple suivant double la capacité de `s` en faisant une nouvelle slice `t`, copie le contenu de `s` en `t` puis attribue la valeur de slice `t` à `s` :

```go
t := make([]byte, len(s), (cap(s)+1)*2) // +1 dans le cas où cap(s) == 0
for i := range s {
        t[i] = s[i]
}
s = t
```

Cette opération commune de boucle est facilitée par la fonction intégrée `copy`. Comme son nom l'indique, `copy` copie les données à partir d'une slice source vers une slice de destination. Elle renvoie le nombre d'éléments copiés.

```go
func copy(dst, src []T) int
```
La fonction `copy` prend en charge la copie entre des slices de longueur différente (elle copie seulement jusqu'au plus petit nombre d'éléments). En outre, la copie peut gérer la source et la destination des slices qui partagent le même tableau sous-jacent et gère le problème de chevauchement des slices.

Avec l'utilisation de `copy`, nous pouvons simplifier le code extrait précédemment :

```go
t := make([]byte, len(s), (cap(s)+1)*2)
copy(t, s)
// s = t
```

Une opération commune consiste à ajouter des données à la fin d'une slice. La fonction `AppendByte` ajoute des éléments d'octets à une slice d’octets (bytes),
augmente la taille de la slice si nécessaire, et renvoie la valeur de la slice mise à jour :

```go
func AppendByte(slice []byte, data ...byte) []byte {
    m := len(slice)
    n := m + len(data)
    if n > cap(slice) { // if necessary, reallocate
        // allocate double what's needed, for future growth.
        newSlice := make([]byte, (n+1)*2)
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[0:n]
    copy(slice[m:n], data)
    return slice
}
```

On pourrait utiliser `AppendByte` comme ceci :

```go
p := []byte{2, 3, 5}
p = AppendByte(p, 7, 11, 13)
// p == []byte{2, 3, 5, 7, 11, 13}
```

Des fonctions comme `AppendByte` sont utiles, car elles offrent un contrôle complet sur la façon dont les slices sont agrandies. Selon les caractéristiques du programme, il peut être souhaitable d'allouer en morceaux plus petits ou plus grands, ou de mettre un plafond sur la taille d'une réaffectation.

Mais la plupart des programmes ne nécessitent pas un contrôle si complet, c'est pourquoi **Go** fournit une fonction intégrée `append` qui est suffisante pour la plupart des cas; elle a la signature suivante :

```go
func append(s []T, x ...T) []T
```

La fonction `append` ajoute les éléments `x` à la fin de la slice `s` et agrandit la slice si une plus grande capacité est nécessaire.

```go
a := make([]int, 1)
// a == []int{0}
a = append(a, 1, 2, 3)
// a == []int{0, 1, 2, 3}
```

Pour combiner une slice à une autre, utilisez `...` pour élargir le second argument à une liste d'arguments.

```go
a := []string{"John", "Paul"}
b := []string{"George", "Ringo", "Pete"}
a = append(a, b...) // equivalent to "append(a, b[0], b[1], b[2])"
// a == []string{"John", "Paul", "George", "Ringo", "Pete"}
```

Puisque la valeur zéro d'une slice (`nil`) agit comme une slice de longueur `0`, vous pouvez déclarer une variable de slice, puis ajouter les éléments dans une boucle :

```go
// Filter retourne une nouvelle slice tenant uniquement
// les éléments de s qui satisfont f()
func Filter(s []int, fn func(int) bool) []int {
    var p []int // == nil
    for _, v := range s {
        if fn(v) {
            p = append(p, v)
        }
    }
    return p
}
```

## Le possible "piège"

Comme mentionné précédemment, slicer une slice ne fait pas une copie du tableau sous-jacent. Le tableau complet sera gardé en mémoire jusqu'à ce qu'il ne soit plus référencé. De temps en temps, cela peut pousser le programme à maintenir toutes les données en mémoire lorsque seulement un petit morceau de celui-ci est nécessaire.

Par exemple, cette fonction `FindDigits` charge un fichier en mémoire et recherche le premier groupe de chiffres numériques consécutifs et les retourne dans une nouvelle slice.

```go
var digitRegexp = regexp.MustCompile("[0-9]+")

func FindDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    return digitRegexp.Find(b)
}
```

Ce code se comporte comme souhaité, mais le retour de la fonction `[]byte` se réfère à un tableau contenant l’intégralité du fichier. Tant que la slice fait référence au tableau original, et aussi longtemps que la slice est maintenue, le ramasse miette ne peut pas libérer la mémoire du tableau ; par conséquent, les quelques octets utiles du fichier conservent tout son contenu.
 

Pour résoudre ce problème, on peut copier les données utiles dans une nouvelle slice avant de la retourner :

```go
func CopyDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    b = digitRegexp.Find(b)
    c := make([]byte, len(b))
    copy(c, b)
    return c
}
```

Nous pourrions réaliser une version plus concise de cette fonction en utilisant `append`. Ceci est laissé comme un exercice pour le lecteur.

## Lectures complémentaires

[**Efficace Go**](https://golang.org/doc/effective_go.html) contient des explications en profondeur des [slices](http://golang.org/doc/effective_go.html#slices) et des [tableaux](http://golang.org/doc/effective_go.html#arrays) et la [spécification du langage](http://golang.org/doc/go_spec.html) **Go** définit les slices et leurs fonctions d'assistance associées.

By Andrew Gerrand

Traduit le 22 Juillet 2016 par Henri Lepic à partir du [Blog Golang.org](https://blog.golang.org/go-slices-usage-and-internals).


