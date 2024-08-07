---
title: "Runspace Communication"
description: "La communication entre des runspaces est essentielle lorsque vous souhaitez exécuter des tâches en parallèle dans PowerShell."
summary: "You can use blog posts for announcing product updates and features."
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

Dans ce tutoriel, nous allons explorer comment mettre en place une communication efficace entre des runspaces en utilisant des files d'attente (queues) pour échanger des données.

## Étape 1 : Création d'une file d'attente partagée

Une file d'attente partagée est un moyen efficace de transférer des données entre des runspaces. Voici comment vous pouvez la créer et l'utiliser :

```powershell
# Créez une file d'attente partagée
$Queue = [System.Collections.Queue]::Synchronized((New-Object System.Collections.Queue))

# Runspace 1 : Ajoutez des données à la file d'attente
$runspace1 = [runspacefactory]::CreateRunspace()
$runspace1.Open()
$script1 = {
    $data = "Donnée de Runspace 1"
    $Queue.Enqueue($data)
}
$runspace1.SessionStateProxy.SetVariable("Queue", $Queue)
$runspace1.SessionStateProxy.SetVariable("Script", $script1)
$runspace1.Invoke()
$runspace1.Close()

# Runspace 2 : Lisez les données de la file d'attente
$runspace2 = [runspacefactory]::CreateRunspace()
$runspace2.Open()
$script2 = {
    $data = $Queue.Dequeue()
    Write-Host "Donnée reçue : $data"
}
$runspace2.SessionStateProxy.SetVariable("Queue", $Queue)
$runspace2.SessionStateProxy.SetVariable("Script", $script2)
$runspace2.Invoke()
$runspace2.Close()
```
## Utilisation de variables partagées :
Vous pouvez également partager des variables entre les runspaces, mais vous devez prendre des précautions pour éviter les conflits de données.

```powershell
# Créez une variable partagée
$SharedVariable = [ref]@{}

# Runspace 1 : Modifiez la variable partagée
$runspace1 = [runspacefactory]::CreateRunspace()
$runspace1.Open()
$script1 = {
    $SharedVariable.Value = "Valeur de Runspace 1"
}
$runspace1.SessionStateProxy.SetVariable("SharedVariable", $SharedVariable)
$runspace1.SessionStateProxy.SetVariable("Script", $script1)
$runspace1.Invoke()
$runspace1.Close()

# Runspace 2 : Lisez la variable partagée
$runspace2 = [runspacefactory]::CreateRunspace()
$runspace2.Open()
$script2 = {
    $data = $SharedVariable.Value
    Write-Host "Donnée reçue : $data"
}
$runspace2.SessionStateProxy.SetVariable("SharedVariable", $SharedVariable)
$runspace2.SessionStateProxy.SetVariable("Script", $script2)
$runspace2.Invoke()
$runspace2.Close()
```
L'utilisation de la file d'attente permet une communication fluide entre les runspaces. Assurez-vous de gérer les cas où la file d'attente est vide ou pleine pour éviter les erreurs potentielles.

## Conclusion

La communication entre des runspaces est essentielle pour tirer parti de la puissance de l'exécution parallèle en PowerShell. En utilisant des files d'attente, vous pouvez échanger des données de manière sécurisée entre les runspaces et améliorer l'efficacité de vos scripts.
