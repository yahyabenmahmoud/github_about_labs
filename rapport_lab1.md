# Rapport - Lab 1 : Git Init

**Nom :** Yahya Ben Mahmoud
**Module :**Programmation réseau 
**Lab :** Lab 1 - Git Init

---

## 2. Configuration de Git

Avant de commencer, on configure l'identité qui sera attachée à chaque commit :

```bash
Yahya@fedora ~/Networking_Programming %  git config --global user.name "YahyaBenMahmoud"
Yahya@fedora ~/Networking_Programming %  git config --global user.email "yahyabenmahmoud2005@gmail.com"
```

On active également la coloration dans le terminal (optionnel, juste pour la lisibilité) :

```bash
Yahya@fedora ~/Networking_Programming % git config --global color.diff auto
Yahya@fedora ~/Networking_Programming %  git config --global color.status auto
git config --global color.branch auto
```

Vérification de la configuration :

```bash
Yahya@fedora ~/Networking_Programming % git config --list
```

---

## 3. Création du dépôt local

```bash
Yahya@fedora ~/Networking_Programming/Lab1-GitInit %  mkdir Lab1-GitInit
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % cd Lab1-GitInit
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % git init
```

`git init` transforme le dossier courant en dépôt Git en créant un dossier caché `.git/` qui va stocker tout l'historique du projet.

---

## 4. Création des fichiers et suivi (tracking)

```bash
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % echo "My first Git Prject" > file1.txt
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % echo "Hello World" > file2.txt
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % git status
```

`git status` montre que les deux fichiers sont **untracked** (non suivis) : Git les voit, mais ne les surveille pas encore.

---

## 5. Staging Area (`git add`)

```bash
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % git add file1.txt
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % git status
```

`git add` place le fichier dans la **zone d'index (staging area)** : c'est une "salle d'attente" avant le commit définitif. `file1.txt` apparaît maintenant comme prêt à être commité.

### Correction avant commit

```bash
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % echo "My first Git Project" > file1.txt
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % git diff file1.txt
```

`git diff` affiche la différence exacte entre le contenu indexé et le contenu actuel du fichier de travail (`-` = supprimé, `+` = ajouté). Cela a permis de repérer et corriger la faute "Prject" → "Project".

---

## 6. Ajout de tous les fichiers en une fois

```bash
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % touch private.txt
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % git add -A
```

`git add -A` indexe **tous** les changements du dépôt en une seule commande. Ici, cela a inclus par erreur `private.txt`, un fichier qu'on ne voulait pas suivre.

### Retirer un fichier de l'index

```bash
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % git rm --cached private.txt
```

`--cached` retire le fichier de la zone d'index **sans le supprimer physiquement** du disque.

### Ignorer ce fichier définitivement

```bash
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % echo private.txt >> .gitignore
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % git add -A
```

Le fichier `.gitignore` liste les fichiers/patterns que Git doit toujours ignorer (mots de passe, fichiers temporaires, etc.).

---

## 7. Premier commit

```bash
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % git commit -m "First commit"
```

Le commit enregistre de façon permanente l'état actuel des fichiers indexés, avec un message descriptif. C'est un "point de sauvegarde" du projet.

---

## 8. Modifier le dernier commit (`--amend`)

```bash
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % echo "Line erreur" >> file2.txt
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % git add file2.txt
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % git commit --amend -m "j'ai ajouté une ligne erreur"
```

`--amend` remplace le dernier commit au lieu d'en créer un nouveau. Utile pour corriger un message ou ajouter un oubli, mais uniquement si ce commit n'a pas encore été partagé (pushé).

---

## 9. Historique des commits

```bash
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % git log
```

**Résultat obtenu :**

```
commit 4aebe8cb97ad588a5bef4042e635e7978b25e873 (HEAD -> master)
Author: YahyaBenMahmoud <yahyabenmahmoud2005@gmail.com>
Date:   Thu Sep 24 09:58:38 2026 +0100

    j'ai ajouté une ligne erreur

commit 814f63626392271abb36fa9af4b2af56de450abb
Author: YahyaBenMahmoud <yahyabenmahmoud2005@gmail.com>
Date:   Thu Sep 24 09:55:56 2026 +0100

    First commit
```

`git log` liste tous les commits (du plus récent au plus ancien), chacun avec son identifiant unique (hash), son auteur, sa date et son message.

---

## 10. Annuler un commit erroné (`git reset --soft`)

Le commit "j'ai ajouté une ligne erreur" contenait une erreur volontaire dans `file2.txt`. Pour la corriger proprement :

```bash
Yahya@fedora ~/Networking_Programming/Lab1-GitInit %  cat file2.txt
```
```
Hello World
Line erreur
```

Correction du texte avec `sed` :

```bash
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % sed -i 's/Line erreur/Ligne corrigée/' file2.txt
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % git status
```

Retour au commit précédent ("First commit") tout en conservant les modifications en cours :

```bash
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % git reset --soft 814f636
```

`--soft` déplace le pointeur de branche vers un commit antérieur **sans perdre les changements** : ils repassent dans la zone d'index. Cela permet d'effacer le commit erroné de l'historique tout en gardant le travail effectué.

```bash
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % git status
```
→ `file2.txt` apparaît maintenant comme modifié et prêt à être commité (staged).

---

## 11. Nouveau commit propre

```bash
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % git add file2.txt
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % git commit -m "Ajout ligne corrigée"
```

---

## 12. Vérification finale

```bash
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % git log
```

**Résultat obtenu (historique final propre) :**

```
commit b9537c29b952f2a653f10e71e75a6f3c499aa7a5 (HEAD -> master)
Author: YahyaBenMahmoud <yahyabenmahmoud2005@gmail.com>
Date:   Sat Sep 26 16:38:11 2026 +0100

    Ajout ligne corrigée

commit 814f63626392271abb36fa9af4b2af56de450abb
Author: YahyaBenMahmoud <yahyabenmahmoud2005@gmail.com>
Date:   Thu Sep 24 09:55:56 2026 +0100

    First commit
```

Le commit erroné a bien disparu de l'historique.

```bash
Yahya@fedora ~/Networking_Programming/Lab1-GitInit % git show b9537c2
```

**Résultat :**

```
commit b9537c29b952f2a653f10e71e75a6f3c499aa7a5 (HEAD -> master)
Author: YahyaBenMahmoud <yahyabenmahmoud2005@gmail.com>
Date:   Sat Sep 26 16:38:11 2026 +0100

    Ajout ligne corrigée

diff --git a/file2.txt b/file2.txt
index 557db03..0f2276f 100644
--- a/file2.txt
+++ b/file2.txt
@@ -1 +1,2 @@
 Hello World
+Ligne corrigée
```

`git show <hash>` affiche le diff complet d'un commit précis. Ce résultat confirme que `file2.txt` contient bien "Hello World" et "Ligne corrigée", et que l'erreur a été corrigée.

---


