# Mots-clefs

  

-   COM
    
-   Maquette de logiciel
    
-   Bus de traitement * :  Le bus informatique est la réunion des parties matérielles et immatérielles qui permet la transmission de données entre les composants participants. Ici on parle donc de la totalité des traitement.
-   Instancier
    
-   Compilation
    
-   Crash mémoire
    
-   Libère les objets
    
-   Ordre d’exécution
    
-   IDisposable * : Provides a mechanism for releasing unmanaged resources.
    
-   Librairies
    
-   Chaîne de valeur
    
-   Garbage Collector *: is a form of automatic"Memory management"


# Plan d’action

-   Etudier
    

## IDisposable
   - Fournit un mécanisme pour relâcher les ressources inutilisés
   - Interface
- Permet de compléter le garbage collector car :
	- On ne sait pas quand il va se déclancher
	- Le GB n'a aucune idée de certaines ressources comme des "windows handles", des fichiers ouverts, ou des diffusions.
````C#
public class SampleClass : IDisposable
{
    // Use volatile because finalizers are called by a separate Thread
    private volatile bool isDisposed;  

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    // Call this method from a subclass when you implement a finalizer (destructor).
    protected void Dispose(bool disposing)
    {
        if (isDisposed) { return; }
        isDisposed = true;

        OnDispose(disposing);
        if (disposing)
        {
            // Cleanup and call Dispose on objects for which this class is responsible 
            // here ...
        }
        // Free unmanaged resources here ...
    }

    // Override this method in a subclass to free, release or reset any resources.
    protected virtual void OnDispose(bool disposing) { }
}
````

##  Garbage collector

 - JVM / CLR a la responsabilité d'allouer de la mémoire pour stocker l'objet en question dans une zone réservée : "tas" ou "heap".
-   Le rôle d'un Garbage Collector sera de bâtir des graphes d'objets en partant des références racines (référence permettant au programme d'atteindre l'objet) et en parcourant récursivement toutes les références rencontrées. Dès qu'un objet est supprimé, il doit être effacé du graphe, et sa **mémoire récupérée**.

Algorithmes :

-   **Mark and Sweep** :
    -   Plus simple et plus connu
    -   Lors exécution, tous les objets sont marqués ==> Accéssible
    -   Lors du deuxième tour, il joue du balayeur (sweep), en parcourant tous les objets non marqués pour les supprimer.
    -   PRO/CONS
        -   Temps exécution proportionnel à la taille du tas
        -   Laisse le tas fragmenté
        -   Fonctionne mal si les objets n'ont pas une taille globale uniforme
-   **Collecte par recopie (stop & copy)** :
    -   Découper l'espace d'allocation mémoire (tas), en deux parties égales.
    -   Les objets référencés dans le premier espace sont copiés dans le second espace
    -   On met à jour les références de l'application pour pointer vers le deuxième espace.
    -   Tous les objets "vivants" sont copiés vers le premier espace.
    -   PRO/CONS
        -   Compacter automatiquement le tas (no fragmentation)
        -   Problème entre données pointeurs et entières
        -   Copie systématique (et inutile) d'objets qui ont une durée de vie importante
        -   Pause bloquantes et potentiellement longue **3 Familles de CG**
-   Conservatifs : "dans le doute s'abstenir" pour éviter les problèmes mémoire contenants un pointeur
-   Incrémentaux : Partitionner en plusieurs parties cohérentes, que l'on peut déclencher indépendamment des unes et des autres. ⇒ Pas bloquer lors de recopie, et applications plus fluides
-   Générationnels : On met une zone spéciale pour les objets "immortels" ou trop volumineux, et une autre pour les objets qui sont crées dynamiquement (voués à disparaître rapidement).

**Le GC .NET**

-   Compromis entre le Mark & Sweep et la collecte par recopie : "Mark and Compact"
-   A chaque collecte, le CG bâtit les graphes d'objets atteignables à partir de chaque référence racine
-   Tour : Parcourt le tas linéairement objet par objet. Chaque fois qu'il trouve un objet non référencé, sa mémoire est "libre". Les objets qui ne sont pas touchés dans le second tour, sont recopiés vers le bas du tas, pour écraser la zone mémoire libre, et compacter le tout. Les références sont mises à jours. (Sauf si +20 Ko ⇒ Stockés dans un endroit du tas et pratiquement jamais recopié)

/!\ Si mémoire est pleine, le cycle de collecte ne s'exécute pas.


##   Appels dynamiques
    
### Les erreurs en classe dynamique :
```C#
    ExampleClass ec = new ExampleClass();
    // The following call to exampleMethod1 causes a compiler error 
    // if exampleMethod1 has only one parameter. Uncomment the line
    // to see the error.
    //ec.exampleMethod1(10, 4);

    dynamic dynamic_ec = new ExampleClass();
    // The following line is not identified as an error by the
    // compiler, but it causes a run-time exception.
    dynamic_ec.exampleMethod1(10, 4);

    // The following calls also do not cause compiler errors, whether 
    // appropriate methods exist or not.
    dynamic_ec.someMethod("some argument", 7, null);
    dynamic_ec.nonexistentMethod();
```
###  Lambda
    
- Fonction anonyme utilisé pour :
	- Créer des delegates
	- Des types d'expression d'arbre
- On peut écrire des fonctions locales, qui peuvent être passés en arguments, ou retournés comme valeurs dans une fonction.
- Très utile pour écrire des LINQ query

- On écrit les opérateurs sur le gauche, on met '=>" et à droite, on met le retour.
 
 ````C#
delegate int del(int i);  
static void Main(string[] args)  
{  
    del myDelegate = x => x * x;  
    int j = myDelegate(5); //j = 25  
}  
`````

````C#
double division = Calcul((a, b) =>
{
    return (double)a / (double)b;
}, 4, 5);

````

###  Méthodes anonymes
   
   - Introduite en C# dans la version 3.0
   - Instruction 'inline' qui peut être utilisée partout où un type délégué est attendu.
   - n'a pas de nom, n'a que de vie à cet endroit là
   - Très utile pour simplifier la synthaxe des délégués :
  ```C#
    //Déclaration du type délégué AfficheHelloWorld_Delegate
    delegate void AfficheHelloWorld_Delegate();
    
    AfficheHelloWorld_Delegate del_Hw = null;
/* [........] */
        // Ajoute directement au délégué le code à exécuter
	del_Hw = delegate { MessageBox.Show("Hello World !"); };
	// à la place de :
	```
del_Hw = new AfficheHelloWorld_Delegate(AfficheHelloWorld);
```

    
## Les délégués
 - Base des évènements
- Permet de créer des pointeurs vers des méthodes
- Définit uns "signature de méthode", et on va pouvoir pointer vers n'importe quel méthode qui respecte la signature.
- On va les déclarer avec le type delegate.
- Une delegate peut-être multicast : Pointer sur plusieurs méthodes. En en invoquant une, ça invoque la deuxième automatiquement. (pratique hein)

##  Actions <T>
  
- Action est un délégué qui permet de pointer vers une méthode void, et qui peut accepter jusqu’à 16 types différents.
- Equivaut à créer un délégué qui ne renvoi rien, et qui prend  un type en paramètre.

````C#
public class TrieurDeTableau
{
    private void TrierEtAfficher(int[] tableau, Action<int[]> methodeDeTri)
    {
        methodeDeTri(tableau);
        foreach (int i in tableau)
        {
            Console.WriteLine(i);
        }
    }

    public void DemoTri(int[] tableau)
    {
        TrierEtAfficher(tableau, delegate(int[] leTableau)
        {
            Array.Sort(leTableau);
        });

        Console.WriteLine();

        TrierEtAfficher(tableau, delegate(int[] leTableau)
        {
            Array.Sort(leTableau);
            Array.Reverse(leTableau);
        });
    }
}
````

## Func(T1, T2)
 
 - Délégué qui permet de pointer vers une fonction. Le dernier paramètre générique sera du type du retour du délégué    

````C#
public class Operations
{
    public void DemoOperations()
    {
        double division = Calcul(delegate(int a, int b)
        {
            return (double)a / (double)b;
        }, 4, 5);

        double puissance = Calcul(delegate(int a, int b)
        {
            return Math.Pow((double)a, (double)b);
        }, 4, 5);

        Console.WriteLine("Division : " + division);
        Console.WriteLine("Puissance : " + puissance);
    }

    private double Calcul(Func<int, int, double> methodeDeCalcul, int a, int b)
    {
        return methodeDeCalcul(a, b);
    }
}
````
  

-   Réaliser
    

-   Workshop ([https://moodle-exia.cesi.fr/course/view.php?id=820](https://moodle-exia.cesi.fr/course/view.php?id=820))
