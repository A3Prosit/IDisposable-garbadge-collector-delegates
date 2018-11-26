# Prosit 7 Aller : IDisposable et Garbage collector

Lien du prosit : https://moodle-exia.cesi.fr/course/view.php?id=429

## Team
-	Animateur : Pierre Maz
-	Secrétaire : Etienne
-	Gestionnaire : Fantou
-	Scribe : John

## Mots-clefs

-	COM
-	Maquette de logiciel
-	Bus de traitement *: c'est pas un bus de transmission, contexte surement dire une suite de traitement
-	Instancier
-	Compilation
-	Crash mémoire
-	Libère les objets
-	Ordre d’exécution
-	IDisposable *: interface servant à libérer des ressouces non managées
-	Librairies
-	Chaîne de valeur
-	Garbage Collector *: mécanisme de gestion automatique des donnée (mémoire)

(* : à définir)

## Contexte
Quoi ?
↪Réaliser une maquette logiciel
Comment ?
↪ En utilisant des composants COM
Pourquoi ?
↪ Résoudre les erreurs et les problèmes de crash


## Contraintes

Utiliser COM client

## Problématiques

-	Comment éviter des crashs mémoires en utilisant des composants COM ?
-	Comment acquérir et libérer la mémoire en .NET ?

## Généralisation
-	Optimisation
-	Gestion des ressources

## Hypothèses
-	Le garbage collector réorganise la mémoire.
-	Le garbage collector ne supprimera un objet que s’il n’y a plus de références dans l’objet.
-	IDisposable est une interface qui permet de libérer les ressources non utilisées.
-	IDisposable permet de défragmenter les objets.
-	Using est un mot clef qui permet de libérer automatiquement les ressources.
-	Using utilise IDisposable.
-	Il existe une équivalence aux foncteurs en C#.

## Plan d’action
##	Etudier 
-	IDisposable

fournit un mécanisme pour libérer des ressources non gérées

La principale utilisation de cette interface est de libérer les ressources non managées.  Le garbage collector libère automatiquement la mémoire allouée à un objet managé lorsque cet objet n’est plus utilisé.  Toutefois, il n’est pas possible de prédire quand le garbage collection se produit.  En outre, le garbage collector n’a aucune connaissance des ressources non managées, telles que les handles de fenêtre, ou ouvreture des fichiers et flux.

la méthode dispose est appellée par le consomateur de la ressource, on doit l'encapsuler dans une SafeHandle (recommandé) ou substituez Object.Finalize pour libérer les ressources non managées dans le cas où le consommateur oublie d’appeler Dispose

**utilisation:** on le place dans un try, finalize ou avec un using
ex: using (StreamReader sr = new StreamReader(filename)) { txt = sr.ReadToEnd(); }

à la compil using est remplacé par un try/finalize


try { sr = new StreamReader(filename); txt = sr.ReadToEnd(); } finally { if (sr != null) sr.Dispose(); }


on a 2 méthode dispose:
un qui ne prends pas de param appelée par les consomateurs et un avec un bool que Dispose() appelle

```
class BaseClass : IDisposable
{
   // Flag: Has Dispose already been called?
   bool disposed = false;
   // Instantiate a SafeHandle instance.
   SafeHandle handle = new SafeFileHandle(IntPtr.Zero, true);
   
   // Public implementation of Dispose pattern callable by consumers.
   public void Dispose()
   { 
      Dispose(true);
      GC.SuppressFinalize(this);           
   }
   
   // Protected implementation of Dispose pattern.
   protected virtual void Dispose(bool disposing)
   {
      if (disposed)
         return; 
      
      if (disposing) {
         handle.Dispose();
         // Free any other managed objects here.
         //
      }
      
      disposed = true;
   }
}

```

modèle à suivre si on substitue le finalizer 
```
using System;

class BaseClass : IDisposable
{
   // Flag: Has Dispose already been called?
   bool disposed = false;
   
   // Public implementation of Dispose pattern callable by consumers.
   public void Dispose()
   { 
      Dispose(true);
      GC.SuppressFinalize(this);           
   }
   
   // Protected implementation of Dispose pattern.
   protected virtual void Dispose(bool disposing)
   {
      if (disposed)
         return; 
      
      if (disposing) {
         // Free any other managed objects here.
         //
      }
      
      // Free any unmanaged objects here.
      //
      disposed = true;
   }

   ~BaseClass()
   {
      Dispose(false);
   }
}
```


-	Garbage collector



-	Appels dynamiques = délégués
-	Lambda

(var1, var2) => {calcul }


pk lambda? parceque evenements



-	Méthodes anonymes

défini la foncion là où on l'invoque

```
TrierEtAfficher(tableau, delegate(int[] leTableau)
        {
            Array.Sort(leTableau);
        });
```	

elle n'a pas de nom et ne vie qu'a ce moment là 

- Les délégués

variable pointant vers une méthode
équivalent d'un pointeur de fonction mais on sait exactement ce qu'on appelle car le c# est un langage fortement typé
on s'en sert généralement pour passer une méthode en paramtre d'une autre méthode
```
public class TrieurDeTableau{
	private delegate void DelegateTri(int[] tableau);

	public void DemoTri(int[] tableau)
	{
	    DelegateTri tri = TriAscendant;
	    tri(tableau);
	}
}
```
```
static void Main(string[] args)
{
    int[] tableau = new int[] { 4, 1, 6, 10, 8, 5 };
    new TrieurDeTableau().DemoTri(tableau);
}
```

en s'en servant pour passer des fonctions:
```
private void TrierEtAfficher(int[] tableau, DelegateTri methodeDeTri)
{
    methodeDeTri(tableau);
    foreach (int i in tableau)
    {
        Console.WriteLine(i);
    }
}
```
```
public void DemoTri(int[] tableau)
{
    TrierEtAfficher(tableau, TriAscendant);
    Console.WriteLine();
    TrierEtAfficher(tableau, TriDescendant);
}
```


-	Actions <T> et Func(T1, T2)

il s'agit de délégués génériques 
Action permet de pointer vers un délégué qui ne renvoie rien et peut accepter jusqu'à 16 types différents

à la compil l'Action sera remplacée par un délégué qui ne renvoie rien et penant en paramètre ceux de l'Action

lorsque la méthode renvoie quelque chose on peut utiliser Func<T> sachant que le dernier paramètre générique est celui du type de retour
```
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
```



##	Réaliser
-	Workshop (https://moodle-exia.cesi.fr/course/view.php?id=820)
