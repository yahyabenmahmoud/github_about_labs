# Rapport - Lab 2 : Git Branches

**Nom :** Yahya Ben Mahmoud
**Lab :** Lab 2 - Git Branches

---

## 1. Objectif du Lab

L'objectif de ce lab est de comprendre le travail avec les branches Git :
- Créer et gérer des branches
- Basculer entre branches (`checkout`)
- Fusionner des branches (`merge`)
- Tagger une version précise
- Créer et résoudre un conflit de fusion (merge conflict)

---

## 2. Configuration préalable

Ajout d'un alias pour un affichage compact des logs :

```bash
git config --global alias.lgone 'log --oneline --decorate'
```

Cet alias permet d'utiliser `git lgone` à la place de `git log --oneline --decorate`, pour un historique plus lisible sur une seule ligne par commit. Étant en `--global`, il s'applique à tous les dépôts de la machine.

---

## 3. Création du projet "website"

```bash
mkdir -p website/site-prod website/site-dev
cd website
git init
```

Deux dossiers sont créés :
- `site-prod` : le site en production (branche `master`)
- `site-dev` : les nouveautés en cours de développement

```bash
echo '<h1>Hello World!</h1>' > site-prod/index.html
git add -A
git commit -m "The beginnings of my web site."
```

Vérification :

```bash
git lgone
git branch
```

Résultat : un seul commit, une seule branche `master`.

---

## 4. Création et utilisation de la branche `new-features`

### Création de la branche

```bash
git branch new-features
```

`git branch <nom>` crée une nouvelle branche à partir du commit courant, **sans basculer dessus**.

### Basculer sur la branche

```bash
git checkout new-features
```

### Ajout de fichiers sur cette branche

```bash
echo 'Contact : yahyabenmahmoud2005@gmail.com' > site-dev/contact.html
git add -A
git commit -m "Added contact.html file"

echo 'Help : God bless you' > site-dev/help.html
git add -A
git commit -m "Added help.html file"
```

```bash
git lgone
```

Résultat : deux nouveaux commits visibles uniquement sur `new-features`.

### Retour sur master

```bash
git checkout master
git lgone
```

Les fichiers `contact.html` et `help.html` **n'existent pas** sur `master` : chaque branche possède son propre état de fichiers, isolé des autres.

---

## 5. Création de la branche `php-features`

```bash
git checkout -b php-features
```

`checkout -b` crée **et** bascule sur la nouvelle branche en une seule commande.

```bash
mkdir -p site-dev
cat > site-dev/index.php << 'EOF'
<!DOCTYPE html>
<html>
<body>
<h1>My first PHP page</h1>
<?php
echo "Hello World!";
?>
</body>
</html>
EOF

git add -A
git commit -m "Added index.php"
git lgone
```

---

## 6. Modifications sur master

Retour sur `master` pour ajouter un `README.md` et améliorer `index.html` :

```bash
git checkout master
echo "My First HTML Site" > README.md
```

`site-prod/index.html` est complété avec un squelette HTML complet (DOCTYPE, head, meta, title, body).

```bash
git add -A
git commit -m "Added html skeleton to index.html and README.md"
git lgone
```

---

## 7. Fusion de `new-features` dans `master`

```bash
git merge new-features
```

**Résultat obtenu :**

```
Merge made by the 'recursive' strategy.
 site-dev/contact.html | 1 +
 site-dev/help.html    | 1 +
 2 files changed, 2 insertions(+)
```

Aucun conflit : Git a automatiquement combiné les deux historiques car les fichiers modifiés étaient différents entre les deux branches.

```bash
git lgone --graph
```

Ceci affiche visuellement l'arborescence des branches et leur point de fusion.

### Suppression de la branche fusionnée

```bash
git branch -d new-features
```

Une fois fusionnée, la branche `new-features` n'est plus utile : `-d` la supprime en toute sécurité (Git refuse si elle n'est pas totalement fusionnée).

---

## 8. Tagger une version

```bash
git tag v0.1 7d18e96
```

Un tag associe un nom lisible (`v0.1`) à un commit précis — généralement utilisé pour marquer une version stable ou une release.

```bash
git lgone
```

Le tag `v0.1` apparaît désormais à côté du commit de fusion correspondant.

---

## 9. Fusion de `php-features` (sans conflit constaté)

```bash
git checkout php-features
echo "My First PHP Site" > README.md
git add -A
git commit -m "Added README.md file"

git checkout master
git merge php-features
```

**Résultat obtenu :**

```
Merge made by the 'ort' strategy.
 README.md          | 1 +
 site-dev/index.php | 9 +++++++++
 2 files changed, 10 insertions(+)
```

Contrairement à l'exemple du lab, cette fusion s'est effectuée **sans conflit**, car dans mon historique le fichier `README.md` créé sur `php-features` ne partageait pas de commit ancêtre commun avec celui créé sur `master` (la divergence des branches n'était pas exactement la même que dans l'énoncé). Git a donc traité `README.md` comme un nouveau fichier à ajouter plutôt qu'un fichier à fusionner ligne par ligne.

Afin de tout de même illustrer la gestion d'un vrai conflit de fusion (objectif pédagogique du lab), un conflit a été provoqué volontairement (section suivante).

---

## 10. Provoquer et résoudre un vrai conflit de fusion

### Création du conflit

Sur `master` :
```bash
echo "Version A" > conflict-test.md
git add -A
git commit -m "Add conflict-test.md on master"
```

Sur une nouvelle branche partant d'un commit antérieur commun :
```bash
git checkout -b conflict-demo b1f394e
echo "Version B" > conflict-test.md
git add -A
git commit -m "Add conflicting version of conflict-test.md"
```

### Tentative de fusion

```bash
git checkout master
git merge conflict-demo
```

**Résultat :**
```
CONFLICT (add/add): Merge conflict in conflict-test.md
Automatic merge failed; fix conflicts and then commit the result.
```

Le fichier `conflict-test.md` a été créé différemment dans les deux branches : Git est incapable de choisir automatiquement quelle version garder.

### Inspection du conflit

```bash
git status
```

Le fichier apparaît comme "both added" (non fusionné).

Contenu de `conflict-test.md` après la tentative de fusion :
```
<<<<<<< HEAD
Version A
=======
Version B
>>>>>>> conflict-demo
```

- Tout ce qui est entre `<<<<<<< HEAD` et `=======` correspond à la version de la branche courante (`master`).
- Tout ce qui est entre `=======` et `>>>>>>> conflict-demo` correspond à la version de la branche entrante.

### Résolution manuelle

Le fichier est édité manuellement pour ne garder que le contenu final voulu, en supprimant les marqueurs `<<<<<<<`, `=======`, `>>>>>>>` :
```
Version A and B merged
```

### Finalisation de la fusion

```bash
git add -A
git status
```

→ "All conflicts fixed but you are still merging."

```bash
git commit
```

Le commit de fusion est créé, avec un message par défaut généré par Git (ou personnalisé).

---

## 11. Nettoyage des branches

```bash
git branch -d conflict-demo
```

La branche est supprimée car son contenu est désormais intégré dans `master`. `-d` refuse la suppression si la branche n'est pas totalement fusionnée, ce qui protège contre une perte de travail.

```bash
git branch
```

Vérification finale des branches restantes.

---

## 12. Historique final

```bash
git lgone --graph
```

Cette commande affiche l'arbre complet du projet : le commit initial, les branches créées, les fusions (avec et sans conflit), le tag `v0.1`, et la résolution du conflit — le tout dans un seul historique cohérent.

---

## 13. Difficultés rencontrées

- Le dossier `site-dev` n'était pas apparu correctement après le premier `mkdir -p website/{site-prod,site-dev}` (comportement d'expansion d'accolades différent sous zsh). Il a été recréé manuellement avec `mkdir site-dev`.
- L'alias `git lgone` n'a d'abord pas fonctionné car la commande `git config --global alias.lgone '...'` n'avait pas encore été exécutée.
- L'utilisation de `< >` comme espace réservé dans des commandes (`git tag v0.1 <hash>`, `git show <hash>`) a provoqué une erreur de syntaxe sous zsh — il faut toujours remplacer ces chevrons par la vraie valeur (hash de commit).
- La fusion de `php-features` ne s'est pas comportée exactement comme dans l'énoncé (pas de conflit automatique), à cause d'une divergence d'historique légèrement différente. Un conflit a donc été recréé manuellement pour couvrir cet objectif du lab.

---

## 14. Conclusion

Ce lab a permis de mettre en pratique les concepts fondamentaux du travail collaboratif avec Git :
- Créer, basculer et supprimer des branches
- Isoler du travail en développement du code en production
- Fusionner des branches, avec ou sans conflit
- Résoudre manuellement un conflit de fusion en éditant les marqueurs `<<<<<<<` / `=======` / `>>>>>>>`
- Utiliser les tags pour repérer une version précise du projet

La maîtrise des branches et des fusions est essentielle pour travailler en équipe sur un même projet sans écraser le travail des autres.