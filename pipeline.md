# The Go Blog

## Patternes de concurrence en Go: Pipelines et annulation

**13 March 2014**

###Introduction

Les primitives de concurrence en Go facilitent la construction de pipelines de données en flux continu qui utilisent efficacement les I/O et les processeurs multicores. Cet article présente des exemples de tels pipelines, souligne les subtilités qui surviennent lorsque les opérations échouent, et introduit des techniques pour traiter les défaillances proprement.

###Qu'est ce qu'un pipeline?

Il n'y a pas de définition officielle d'un pipeline en Go; C'est juste l'un des nombreux types de programmes simultanés. De façon informelle, un pipeline est une série d'étapes reliées par des canaux, où chaque étape est un groupe de goroutines exécutant la même fonction. A chaque étape, les goroutines

- recevoir des valeurs en amont via des canaux entrants
- effectuer une fonction sur ces données, produisant généralement de nouvelles valeurs
- envoyer des valeurs en aval via des canaux sortants

Chaque étage possède un nombre quelconque de canaux entrants et sortants, à l'exception du premier et du dernier étage, qui ne contiennent que des canaux sortants ou entrants, respectivement. La première étape est parfois appelée la source ou le producteur; La dernière étape, l'évier ou le consommateur.

Nous allons commencer par un simple exemple de pipeline pour expliquer les idées et les techniques. Plus tard, nous allons présenter un exemple plus réaliste.

###Nombres au carré

Considérons un pipeline à trois étapes.

La première étape **gen**, qui est une fonction qui convertit une liste d'entiers en un canal et émet les entiers dans la liste. La fonction **gen** démarre une goroutine qui envoie les entiers sur le canal et ferme le canal lorsque toutes les valeurs ont été envoyées :

```go
func gen(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        for _, n := range nums {
            out <- n
        }
        close(out)
    }()
    return out
}
```

La deuxième étape **sq**, qui reçoit des entiers d'un canal et renvoie un canal qui émet le carré de chaque entier reçu. Une fois que le canal entrant est fermé et que cette étape a envoyé toutes les valeurs en aval, il ferme le canal sortant :

```go
func sq(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        for n := range in {
            out <- n * n
        }
        close(out)
    }()
    return out
}
```

La fonction **main** configure le pipeline et exécute l'étape finale: elle reçoit les valeurs de la deuxième étape et imprime chacune d'elles, jusqu'à ce que le canal soit fermé :

```go
func main() {
    // Set up the pipeline.
    c := gen(2, 3)
    out := sq(c)

    // Consume the output.
    fmt.Println(<-out) // 4
    fmt.Println(<-out) // 9
}
```

Puisque **sq** a le même type pour ses canaux entrants et sortants, nous pouvons le composer n'importe quel nombre de fois. Nous pouvons également réécrire **main** comme une boucle de plage, comme les autres étapes :

```go
func main() {
    // Set up the pipeline and consume the output.
    for n := range sq(sq(gen(2, 3))) {
        fmt.Println(n) // 16 then 81
    }
}
```

###Fan-out, Fan-in

Plusieurs fonctions peuvent lire à partir du même canal jusqu'à ce que ce canal soit fermé; C'est ce qu'on appelle le **fan-out**. Cela permet de répartir le travail entre un groupe de workers pour paralléliser l'utilisation du processeur et les I/O.

Une fonction peut lire à partir de plusieurs entrées et continuer jusqu'à ce que toutes soient fermées en multiplexant les canaux d'entrée sur un seul canal fermé lorsque toutes les entrées sont fermées. C'est ce qu'on appelle le **fan-in**.

Nous pouvons changer notre pipeline pour exécuter deux instances de **sql**, chaque lecture du même canal d'entrée. Nous introduisons une nouvelle fonction, fusionner, pour ventiler dans les résultats :

```go
func main() {
    in := gen(2, 3)

    // Distribute the sq work across two goroutines that both read from in.
    c1 := sq(in)
    c2 := sq(in)

    // Consume the merged output from c1 and c2.
    for n := range merge(c1, c2) {
        fmt.Println(n) // 4 then 9, or 9 then 4
    }
}
```

La fonction **merge** convertit une liste de canaux en un canal unique en commençant une goroutine pour chaque canal entrant qui copie les valeurs au seul canal sortant. Une fois que toutes les goroutines de sortie ont été démarrées, le **merge** démarre une autre goroutine pour fermer le canal sortant après que tous les envois sur ce canal soient terminés.

Sur un canal fermé Go envoie un panique , il est donc important de s'assurer que tous les envois soient effectués avant d'appeler le **close**. Le type **sync.WaitGroup** fournit un moyen simple d'organiser cette synchronisation :


```go
func merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    // Start an output goroutine for each input channel in cs.  output
    // copies values from c to out until c is closed, then calls wg.Done.
    output := func(c <-chan int) {
        for n := range c {
            out <- n
        }
        wg.Done()
    }
    wg.Add(len(cs))
    for _, c := range cs {
        go output(c)
    }

    // Start a goroutine to close out once all the output goroutines are
    // done.  This must start after the wg.Add call.
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}
```

###Arrêt écourté

Il existe un modèle pour nos fonctions de pipeline :

- étapes qui ferment les canaux sortants lorsque toutes les opérations d'envoi sont effectuées.
- étapes qui continuent de recevoir des valeurs des canaux entrants jusqu'à ce que ces canaux soient fermés.

Ce modèle permet à chaque étape de réception d'être écrite comme une boucle de plage et assure que toutes les goroutines sortent une fois que toutes les valeurs ont été envoyées avec succès en aval.

Mais dans les pipelines réels, les étapes ne reçoivent pas toujours toutes les valeurs entrantes. Parfois, c'est par conception: le récepteur n'a besoin que d'un sous-ensemble de valeurs pour progresser. Plus souvent, une étape sort plus tôt car une valeur entrante représente une erreur dans une étape antérieure. Dans les deux cas, le récepteur ne devrait pas avoir à attendre que les valeurs restantes arrivent, et nous voulons que les étapes précédentes arrêtent de produire des valeurs dont les étapes ultérieures n'ont pas besoin.

Dans notre pipeline d'exemple, si une étape ne consomme pas toutes les valeurs entrantes, les goroutines essayant d'envoyer ces valeurs bloqueront indéfiniment :

```go
    // Consume the first value from output.
    out := merge(c1, c2)
    fmt.Println(<-out) // 4 or 9
    return
    // Since we didn't receive the second value from out,
    // one of the output goroutines is hung attempting to send it.
}
```
Il s'agit d'une fuite de ressources: les goroutines consomment de la mémoire et des ressources d'exécution, et les références de tas dans les piles de goroutine empêchent les données d'être collectées. Les goroutines ne sont pas garbage collectées; Elles doivent sortir seuls.

Nous devons prendre des dispositions pour que les étapes en amont de notre pipeline sortent même lorsque les étapes aval ne parviennent pas à recevoir toutes les valeurs entrantes. Une façon de faire est de changer les canaux sortants pour avoir un tampon. Un tampon peut contenir un nombre fixe de valeurs; Envoyer des opérations immédiatement si il y a de la place dans le tampon :

```go
c := make(chan int, 2) // buffer size 2
c <- 1  // succeeds immediately
c <- 2  // succeeds immediately
c <- 3  // blocks until another goroutine does <-c and receives 1
```

Lorsque le nombre de valeurs à envoyer est connu au moment de la création du canal, un buffer peut simplifier le code. Par exemple, nous pouvons réécrire **gen** pour copier la liste des entiers dans un canal bufferisé et ainsi éviter de créer une nouvelle goroutine :

```go
func gen(nums ...int) <-chan int {
    out := make(chan int, len(nums))
    for _, n := range nums {
        out <- n
    }
    close(out)
    return out
}
```

Si nous retournons les goroutines bloquées dans notre pipeline, nous pourrions envisager d'ajouter un buffer au canal sortant renvoyé par **merge** :

```go
func merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int, 1) // enough space for the unread inputs
    // ... the rest is unchanged ...
```    

Bien que cela corrige la goroutine bloquée dans ce programme, c'est un mauvais code. Le choix de la taille du buffer de 1 dépend ici du nombre de valeurs que la fusion va recevoir et du nombre de valeurs que les étapes en aval consommeront. Ceci est fragile: si nous transmettons une valeur supplémentaire à **gen**, ou si l'étape en aval lit moins de valeurs, nous aurons de nouveau des goroutines de bloqué.

Au lieu de cela, nous devons fournir un moyen pour que les étapes en aval puissent indiquer aux expéditeurs qu'ils cesseront d'accepter les données.

###Annulation explicite

Quand **main** décide de sortir sans recevoir toutes les valeurs de **out**, il doit dire aux goroutines dans les étapes en amont d'abandonner les valeurs qu'ils essaient d'envoyer. Il le fait en envoyant des valeurs sur un canal appelé **done**. Il envoie deux valeurs puisqu'il y a potentiellement deux expéditeurs bloqués :

```go
func main() {
    in := gen(2, 3)

    // Distribute the sq work across two goroutines that both read from in.
    c1 := sq(in)
    c2 := sq(in)

    // Consume the first value from output.
    done := make(chan struct{}, 2)
    out := merge(done, c1, c2)
    fmt.Println(<-out) // 4 or 9

    // Tell the remaining senders we're leaving.
    done <- struct{}{}
    done <- struct{}{}
}
```

Les goroutines émetteurs remplacent leur opération d'envoi par une instruction **select** qui se déroule soit lorsque l'envoi se produit soit quand ils reçoivent une valeur de **done**. Le type de valeur de **done** est la structure vide parce que la valeur n'a pas d'importance: c'est l'événement **receive** qui indique que le **send** sur **out** doit être abandonné. Les goroutines de sortie continuent de boucler sur leur canal entrant, **c**, de sorte que les étages amont ne sont pas bloqués. (Nous discuterons dans un instant comment permettre à cette boucle de revenir plus tôt.)

```go
func merge(done <-chan struct{}, cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    // Start an output goroutine for each input channel in cs.  output
    // copies values from c to out until c is closed or it receives a value
    // from done, then output calls wg.Done.
    output := func(c <-chan int) {
        for n := range c {
            select {
            case out <- n:
            case <-done:
            }
        }
        wg.Done()
    }
    // ... the rest is unchanged ...
```

Cette approche a un problème: chaque récepteur en aval doit connaître le nombre d'expéditeurs en amont potentiellement bloqués et prendre des dispositions pour signaler ces expéditeurs au retour anticipé. Garder la trace de ces dénombrements est fastidieux et sujet à erreurs.

Nous avons besoin d'une façon de dire à un nombre inconnu et illimité de goroutines d'arrêter d'envoyer leurs valeurs en aval. Dans Go, nous pouvons le faire en fermant un canal, car une opération de réception sur un canal fermé peut toujours procéder immédiatement, ce qui donne la valeur zéro du type d'élément.

Cela signifie que **main** peut débloquer tous les expéditeurs simplement en fermant le canal **done**. Cette fermeture est effectivement un signal de diffusion pour les expéditeurs. Nous étendons chacune de nos fonctions de pipeline pour accepter comme un paramètre et nous organisons pour que la fin se produise via une instruction de report, de sorte que tous les chemins de retour à partir de **main** signalent les étapes de pipeline à la sortie.

```go
func main() {
    // Set up a done channel that's shared by the whole pipeline,
    // and close that channel when this pipeline exits, as a signal
    // for all the goroutines we started to exit.
    done := make(chan struct{})
    defer close(done)

    in := gen(done, 2, 3)

    // Distribute the sq work across two goroutines that both read from in.
    c1 := sq(done, in)
    c2 := sq(done, in)

    // Consume the first value from output.
    out := merge(done, c1, c2)
    fmt.Println(<-out) // 4 or 9

    // done will be closed by the deferred call.
}
```

Chacun de nos stades de pipeline est maintenant libre de revenir dès que le **done** est fermé. La routine de sortie **merge** peut retourner sans drainer son canal entrant, puisqu'elle connaît l'expéditeur en amont, **sq**, cessera d'essayer d'envoyer quand il est fermé. La sortie garantit que **wg.Done** est appelé sur tous les chemins de retour via une instruction **defer** :

```go
func merge(done <-chan struct{}, cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    // Start an output goroutine for each input channel in cs.  output
    // copies values from c to out until c or done is closed, then calls
    // wg.Done.
    output := func(c <-chan int) {
        defer wg.Done()
        for n := range c {
            select {
            case out <- n:
            case <-done:
                return
            }
        }
    }
    // ... the rest is unchanged ...
```

De même, **sq** peut retourner dès que **done** est fermé. **Sq** assure que son canal de sortie est fermé sur tous les chemins de retour via une instruction **defer** :

```go
func sq(done <-chan struct{}, in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            select {
            case out <- n * n:
            case <-done:
                return
            }
        }
    }()
    return out
}
```

Voici les directives pour la construction de pipelines:

- les étapes ferment leurs canaux sortants lorsque toutes les opérations d'envoi sont effectuées.
- les étapes continuent de recevoir des valeurs des canaux entrants jusqu'à ce que ces canaux soient fermés ou que les expéditeurs soient débloqués.

Les pipelines débloquent les expéditeurs soit en veillant à ce qu'il y ait suffisamment de mémoire tampon pour toutes les valeurs envoyées, soit en signalant explicitement l'expéditeur lorsque le récepteur peut abandonner le canal.

###Digérer un tree

Considérons un pipeline plus réaliste.

**MD5** est un algorithme message-digest qui est utile comme un fichier **checksum**. L'utilitaire de ligne de commande **md5sum** imprime des valeurs de digest pour une liste de fichiers.

```sh
% md5sum *.go
d47c2bbc28298ca9befdfbc5d3aa4e65  bounded.go
ee869afd31f83cbb2d10ee81b2b831dc  parallel.go
b88175e65fdcbc01ac08aaf1fd9b5e96  serial.go
```

Notre exemple de programme est comme **md5sum** mais prend plutôt un seul répertoire comme un argument et imprime les valeurs de digest pour chaque fichier sous ce répertoire, triés par nom de chemin.

```sh
% go run serial.go .
d47c2bbc28298ca9befdfbc5d3aa4e65  bounded.go
ee869afd31f83cbb2d10ee81b2b831dc  parallel.go
b88175e65fdcbc01ac08aaf1fd9b5e96  serial.go
```

La fonction principale de notre programme invoque une fonction d'assistance **MD5All**, qui renvoie une map du nom de chemin à la valeur de digest, puis trie et imprime les résultats :

```go
func main() {
    // Calculate the MD5 sum of all files under the specified directory,
    // then print the results sorted by path name.
    m, err := MD5All(os.Args[1])
    if err != nil {
        fmt.Println(err)
        return
    }
    var paths []string
    for path := range m {
        paths = append(paths, path)
    }
    sort.Strings(paths)
    for _, path := range paths {
        fmt.Printf("%x  %s\n", m[path], path)
    }
}
```

La fonction **MD5All** est au centre de notre discussion. Dans **serial.go**, l'implémentation n'utilise pas de simultanéité et lit et additionne simplement chaque fichier pendant qu'il parcourt l'arborescence.

```go
// MD5All reads all the files in the file tree rooted at root and returns a map
// from file path to the MD5 sum of the file's contents.  If the directory walk
// fails or any read operation fails, MD5All returns an error.
func MD5All(root string) (map[string][md5.Size]byte, error) {
    m := make(map[string][md5.Size]byte)
    err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
        if err != nil {
            return err
        }
        if !info.Mode().IsRegular() {
            return nil
        }
        data, err := ioutil.ReadFile(path)
        if err != nil {
            return err
        }
        m[path] = md5.Sum(data)
        return nil
    })
    if err != nil {
        return nil, err
    }
    return m, nil
}
```

###Digestion en parallèle

En parallèle, nous avons divisé **MD5All** en un pipeline à deux étages. Le premier étage, **sumFiles**, parcourt l'arbre, digère chaque fichier dans une nouvelle goroutine et envoie les résultats sur un canal avec le résultat par type de valeur :

```go
type result struct {
    path string
    sum  [md5.Size]byte
    err  error
}
```

**sumFiles** renvoie deux canaux: un pour les résultats et un autre pour l'erreur renvoyée par **filepath.Walk**. La fonction **walk** démarre une nouvelle goroutine pour traiter chaque fichier, puis vérifie. S'il est fermé, **walk** s'arrête immédiatement :

```go
func sumFiles(done <-chan struct{}, root string) (<-chan result, <-chan error) {
    // For each regular file, start a goroutine that sums the file and sends
    // the result on c.  Send the result of the walk on errc.
    c := make(chan result)
    errc := make(chan error, 1)
    go func() {
        var wg sync.WaitGroup
        err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
            if err != nil {
                return err
            }
            if !info.Mode().IsRegular() {
                return nil
            }
            wg.Add(1)
            go func() {
                data, err := ioutil.ReadFile(path)
                select {
                case c <- result{path, md5.Sum(data), err}:
                case <-done:
                }
                wg.Done()
            }()
            // Abort the walk if done is closed.
            select {
            case <-done:
                return errors.New("walk canceled")
            default:
                return nil
            }
        })
        // Walk has returned, so all calls to wg.Add are done.  Start a
        // goroutine to close c once all the sends are done.
        go func() {
            wg.Wait()
            close(c)
        }()
        // No select needed here, since errc is buffered.
        errc <- err
    }()
    return c, errc
}
```

**MD5All** reçoit les valeurs de digest de **c.MD5All** retourne au début de l'erreur, la ferme **done** via un **defer** :

```go
func MD5All(root string) (map[string][md5.Size]byte, error) {
    // MD5All closes the done channel when it returns; it may do so before
    // receiving all the values from c and errc.
    done := make(chan struct{})
    defer close(done)

    c, errc := sumFiles(done, root)

    m := make(map[string][md5.Size]byte)
    for r := range c {
        if r.err != nil {
            return nil, r.err
        }
        m[r.path] = r.sum
    }
    if err := <-errc; err != nil {
        return nil, err
    }
    return m, nil
}
```

###Parallélisme encadré

La mise en œuvre d'**MD5All** en **parallel.go** démarre une nouvelle goroutine pour chaque fichier. Dans un répertoire avec de nombreux fichiers volumineux, cela peut allouer plus de mémoire que celle disponible sur la machine.

Nous pouvons limiter ces allocations en limitant le nombre de fichiers lus en parallèle. Dans **bounded.go**, nous le faisons en créant un nombre fixe de goroutines pour lire des fichiers. Notre pipeline a maintenant trois étapes: parcourir l'arbre, lire et digérer les dossiers, et collecter les digests.

La première étape, le parcours des fichiers, émet les chemins des fichiers réguliers dans l'arbre :

```go
func walkFiles(done <-chan struct{}, root string) (<-chan string, <-chan error) {
    paths := make(chan string)
    errc := make(chan error, 1)
    go func() {
        // Close the paths channel after Walk returns.
        defer close(paths)
        // No select needed for this send, since errc is buffered.
        errc <- filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
            if err != nil {
                return err
            }
            if !info.Mode().IsRegular() {
                return nil
            }
            select {
            case paths <- path:
            case <-done:
                return errors.New("walk canceled")
            }
            return nil
        })
    }()
    return paths, errc
}
```

Le stade intermédiaire commence un nombre fixe de goroutines et **digesteur**  reçoie des noms de fichiers à partir de chemins et envoie des résultats sur le canal **c** :

```go
func digester(done <-chan struct{}, paths <-chan string, c chan<- result) {
    for path := range paths {
        data, err := ioutil.ReadFile(path)
        select {
        case c <- result{path, md5.Sum(data), err}:
        case <-done:
            return
        }
    }
}
```

Contrairement aux exemples précédents, le **digesteur** ne ferme pas son canal de sortie, car plusieurs goroutines envoient un canal partagé. Au lieu de cela, le code dans **MD5All** fait en sorte que le canal soit fermé lorsque tous les **digesteurs** soient terminés :

```go
    // Start a fixed number of goroutines to read and digest files.
    c := make(chan result)
    var wg sync.WaitGroup
    const numDigesters = 20
    wg.Add(numDigesters)
    for i := 0; i < numDigesters; i++ {
        go func() {
            digester(done, paths, c)
            wg.Done()
        }()
    }
    go func() {
        wg.Wait()
        close(c)
    }()
```

Nous pourrions, plutôt que d'avoir des **digesters**, créer et retourner un propre canal de sortie, mais alors nous aurions besoin de goroutines supplémentaires pour **fan-in** les résultats.

L'étape finale reçoit tous les résultats de **c** puis vérifie l'erreur de **errc**. Cette vérification ne peut pas se produire plus tôt, car avant ce point, **walkFiles** peut bloquer les valeurs d'envoi en aval :

```go
    m := make(map[string][md5.Size]byte)
    for r := range c {
        if r.err != nil {
            return nil, r.err
        }
        m[r.path] = r.sum
    }
    // Check whether the Walk failed.
    if err := <-errc; err != nil {
        return nil, err
    }
    return m, nil
}
```

###Conclusion

Cet article a présenté des techniques pour la construction de pipelines de données en streaming dans Go. Le traitement des défaillances dans ces pipelines est délicat, car chaque étape du pipeline peut bloquer la tentative d'envoi de valeurs en aval et les étapes aval peuvent ne plus se soucier des données entrantes. Nous avons montré comment la fermeture d'un canal peut diffuser un signal "done" à toutes les goroutines démarré par un pipeline et défini des lignes directrices pour la construction des pipelines correctement.

Autres lectures:

- [Go Concurrency Patterns](http://talks.golang.org/2012/concurrency.slide#1) ([video](https://www.youtube.com/watch?v=f6kdp27TYZs)) Présente les bases des primitives de la concurrence Gps et plusieurs façons de les appliquer.
- [Advanced Go Concurrency Patterns](http://blog.golang.org/advanced-go-concurrency-patterns) ([video](http://www.youtube.com/watch?v=QDDwwePbDtw)) Couvre des utilisations plus complexes des primitives Gps, particulièrement select.
- Douglas McIlroy's paper [Squinting at Power Series](http://swtch.com/~rsc/thread/squint.pdf) Montre comment la concurrence simultanée fournit un support élégant pour les calculs complexes.

**Par Sameer Ajmani**

