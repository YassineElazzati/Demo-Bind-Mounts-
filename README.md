# TP Docker : Maîtriser les Bind Mounts pour le Hot-Reload

Bienvenue dans ce TP conçu pour vous faire découvrir la puissance des **Bind Mounts** Docker, une technique essentielle pour le développement et le "hot-reload".

## Introduction

Dans le monde du développement moderne, la rapidité est reine. Attendre la reconstruction d'une image Docker après chaque petite modification de code est une perte de temps. C'est ici qu'interviennent les **Bind Mounts**.

Un Bind Mount est une "porte" que l'on ouvre entre le système de fichiers de votre machine (l'hôte) et l'intérieur d'un conteneur. Toute modification sur l'un est instantanément visible sur l'autre. Des géants comme Google utilisent ce principe pour le développement de leurs applications, permettant aux développeurs de voir leurs changements en temps réel sans jamais redémarrer leur conteneur.

Ce TP vous montrera comment mettre en place ce miroir parfait entre votre machine et un conteneur.

## Prérequis

*   Avoir Docker Desktop installé et fonctionnel sur votre machine (Mac, Windows ou Linux).
*   Avoir un terminal (ligne de commande) ouvert.

## Étapes pas à pas

### 1. Préparation de l'environnement local

Commençons par créer un dossier de travail et un fichier qui nous servira de source.

```bash
# Crée un dossier et navigue dedans
mkdir ~/demo-bind && cd ~/demo-bind

# Crée un fichier avec un message initial
echo 'Salut le futur expert DevOps !' > message.txt
```

### 2. Lancement du conteneur avec un Bind Mount

C'est ici que la magie opère. Nous allons lancer un conteneur `alpine` (une distribution Linux très légère) et lui monter notre dossier `demo-bind`.

```bash
docker run -d --name test-bind -v "$(pwd)":/app alpine sleep 1000
```

**Analyse de la commande :**
*   `docker run -d` : Lance un conteneur en mode "détaché" (en arrière-plan).
*   `--name test-bind` : Donne un nom facile à retenir à notre conteneur.
*   `-v "$(pwd)":/app` : La partie la plus importante !
    *   `-v` est l'abréviation de `--volume`.
    *   `"$(pwd)"` : C'est une commande shell qui est remplacée par le chemin absolu de votre dossier actuel (ex: `/Users/yassine/demo-bind`). **Les guillemets sont cruciaux** pour que cela fonctionne même si votre chemin contient des espaces (fréquent sur Windows ou macOS).
    *   `:` : C'est le séparateur entre le chemin de l'hôte et le chemin du conteneur.
    *   `/app` : C'est le dossier à l'intérieur du conteneur où notre dossier local sera "monté".
*   `alpine sleep 1000` : L'image que nous utilisons et la commande pour la maintenir en vie pendant 1000 secondes.

### 3. Vérification du miroir

Vérifions si le fichier `message.txt` de notre machine est bien visible à l'intérieur du conteneur.

```bash
docker exec test-bind cat /app/message.txt
```

Vous devriez voir s'afficher : `Salut le futur expert DevOps !`. Le miroir fonctionne !

### 4. Le Hot-Reload en action !

Maintenant, modifiez le fichier `message.txt` sur votre machine avec votre éditeur de code préféré (ou via la commande `echo 'Le Hot-Reload, c'est magique !' > message.txt`).

Puis, revérifiez le contenu dans le conteneur :

```bash
docker exec test-bind cat /app/message.txt
```

Le nouveau message apparaît instantanément, **sans avoir à reconstruire ou redémarrer le conteneur**. C'est ça, le hot-reload !

## Points de Vigilance & Bonnes Pratiques

### Conflits de Permissions (UID/GID)

Sur Linux (et parfois sur Mac), vous pourriez rencontrer des problèmes de permissions. Le processus à l'intérieur du conteneur tourne avec un certain `UserID` (UID). Si cet UID ne correspond pas à celui du propriétaire des fichiers sur votre machine hôte, le conteneur pourrait ne pas avoir le droit de lire ou d'écrire dans le dossier monté. C'est une problématique avancée, mais gardez-la en tête !

### Option avancée : Montage en lecture seule

Si vous voulez juste donner accès à des fichiers de configuration sans que le conteneur ne puisse les modifier, vous pouvez monter le volume en lecture seule en ajoutant `:ro` (read-only).

```bash
docker run -v "$(pwd)":/app:ro ...
```

## Exercice d'auto-évaluation

Pour être sûr d'avoir bien compris, répondez à ces deux questions :

1.  Si je supprime le conteneur `test-bind`, est-ce que mon fichier `message.txt` sur ma machine est supprimé ? Pourquoi ?
2.  Quelle est la principale différence de "gestion" entre un Bind Mount et un Volume Nommé ?

---

*Réponses : 

1. Non, le fichier n'est pas supprimé car il vit sur votre machine hôte; le Bind Mount n'est qu'un "lien". 

2. Un Bind Mount est géré par vous (le chemin existe sur votre machine), tandis qu'un Volume Nommé est entièrement géré par Docker (vous ne vous souciez pas de son emplacement physique).*