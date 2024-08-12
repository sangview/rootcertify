---
title: "Runspace Communication"
description: ""
summary: "La communication entre des runspaces est essentielle lorsque vous souhaitez exécuter des tâches en parallèle dans PowerShell."
date: 2023-09-07T16:21:44+02:00
lastmod: 2023-09-07T16:21:44+02:00
draft: false
weight: 15
categories: ["Scripts"]
tags: ["Powershell", "Script", "Windows"]
contributors: ["Adama SANGARE"]
pinned: false
homepage: false
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

# Communication entre des runspaces dans PowerShell
Dans ce tutoriel, nous allons explorer 2 méthodes pour mettre en place une communication efficace entre des runspaces en utilisant des files d'attente (queues) et les hashtables pour échanger des données.

## 1 - Création d'une file d'attente partagée

Une file d'attente partagée est un moyen efficace de transférer des données entre des runspaces. Voici comment vous pouvez la créer et l'utiliser :

```powershell
# Runspace 1 : Création
$runspace1 = [runspacefactory]::CreateRunspace()
$runspace1.Open()

# Runspace 2 : Création
$runspace2 = [runspacefactory]::CreateRunspace()
$runspace2.Open()
```
### Création d'une file d'attente et partage
```powershell

$Queue = [System.Collections.Queue]::Synchronized((New-Object System.Collections.Queue))

# Partage de la file d'attente avec les runspaces
$runspace1.SessionStateProxy.SetVariable("Queue", $Queue)
$runspace2.SessionStateProxy.SetVariable("Queue", $Queue)

```

{{< callout context="tip" title="Affichage dans un runspace?" icon="outline/rocket" >}}
Pouvoir afficher des messages reçus dans la file d'attente depuis le runspace sur l'interface de notre console ISE (runspace parent), on introduit la variable d'envrionnement $host dans notre runspace
{{< /callout >}}

```powershell
$Affichage = $host
$runspace2.SessionStateProxy.SetVariable("Affichage", $Affichage)
```

### Lecture et écriture d'une file d'attente dans un runspace
```powershell
# Runspace 1 : Envoi des données vers le runspace2 via la file d'attente
$powershell1 = [powershell]::Create()
$powershell1.Runspace = $runspace1
$powershell1.AddScript({
    $data = "Donnée de Runspace 1"
    $Queue.Enqueue($data)
}) | Out-Null
$handle = $powershell1.BeginInvoke()
```

```powershell
# Runspace 2 : Reception des données depuis le runspace1 via la file d'attente
$powershell2 = [powershell]::Create()
$powershell2.Runspace = $runspace2
$powershell2.AddScript({
    $data = $Queue.Dequeue()
    Write-Host "Donnée reçue : $data"
    $Affichage.ui.WriteLine("Donnée reçue : $data")
}) | Out-Null
$handle = $powershell2.BeginInvoke()
```
{{< callout context="note" title="Note" icon="outline/info-circle" >}}
 La commande Write-Host seule ne peut pas afficher de message sur la console courante (runspace parent) depuis un runspace.
{{< /callout >}}


## 2 - Utilisation de variables partagées :
Il est également possible de partager des variables entre les runspaces, mais une attention particulière doit être observée pour éviter les conflits de données.

```powershell
# Création des runspaces
$runspace1 = [runspacefactory]::CreateRunspace()
$runspace1.Open()
$runspace2 = [runspacefactory]::CreateRunspace()
$runspace2.Open()
```
### Création d'une variable partagée

```powershell
# Création d'une variable partagée

$personnage = [hashtable]::Synchronized(@{})
$personnage.Name = "Qui ?"
$personnage.Jeton = 5
```
On introduit une variable de contrôle pour arrêter ou démarrer les runspaces

```powershell
$personnage.Continue = $true  # changer à $false à tout moment pour arrêter l'exécution des runspaces
```

{{< callout context="tip" title="Affichage dans un runspace?" icon="outline/rocket" >}}
Comme précédemment, on introduit la variable d'envrionnement $host dans notre variable partagé pour afficher des messages sur l'interface du runspace parent
{{< /callout >}}

```powershell
$personnage.Affichage = $host
```

Partage des variables dans les sessions de nos runspaces
```powershell
$runspace1.SessionStateProxy.SetVariable("personnage", $personnage)
$runspace2.SessionStateProxy.SetVariable("personnage", $personnage)
```
### Lecture et écriture d'une variable partagée depuis des runspaces
```powershell
# Runspace 1 : Exécution
$powershell1 = [powershell]::Create()
$powershell1.Runspace = $runspace1
$powershell1.AddScript({
      while($personnage.Continue){

        $personnage.Name = "Alice"
        $personnage.Affichage.ui.WriteLine(" Name : $($personnage.Name) et Jetons : $($personnage.Jeton)")
        $personnage.Jeton = $personnage.Jeton + 2

        Start-Sleep -Seconds 3
       }
})  | Out-Null
$handle1 = $powershell1.BeginInvoke()
```
Dans le runspace suivant, on va lire la variable en modification dans le runspace1
```powershell
# Runspace 2 : Exécution
$powershell2 = [powershell]::Create()
$powershell2.Runspace = $runspace2
$powershell2.AddScript({
      while($personnage.Continue){

        $personnage.Name = "Bob"
        $personnage.Affichage.ui.WriteLine(" Name : $($personnage.Name)   et Jetons : $($personnage.Jeton)")

        Start-Sleep -Seconds 3
       }
})  | Out-Null
$handle = $powershell2.BeginInvoke()
```
Une fois exécuté, on remarque que le nombre de jetons est incrémenté par le runspace 1 et le runspace 2 affiche en temps réél cette modification. Pour arrêter l'exécution des runspaces, il suffit de briser la boucle grace à l'élément `Continue`

{{< figure
  src="images/site-runspaceCom-Continue.png"
  alt="ISE - Arrêt des runspaces"
  caption="ISE - Arrêt des runspaces"
>}}

L'utilisation de la file d'attente permet une communication fluide entre les runspaces. Assurez-vous de gérer les cas où la file d'attente est vide ou pleine pour éviter les erreurs potentielles.

## Conclusion

La communication entre des runspaces est essentielle pour tirer parti de la puissance de l'exécution parallèle en PowerShell. En utilisant des files d'attente, vous pouvez échanger des données de manière sécurisée entre les runspaces et améliorer l'efficacité de vos scripts.
