# Exercices du TD n°3

## Objectifs
* Créer des conflits !
* Les résoudre
* Voir la différence entre un merge, un merge fast-forward, un merge squash
* Simuler une désynchro entre un dépôt local et un dépôt distant !

## Mise en place
> Comme l'autre jour, vous allez devoir sortir de ce répertoire (le répertoire contenant *mon* dépôt) et créer une hiérarchie avec vos différents dépôts.
> Je vous propose donc de créer avec une hiérachie ainsi faite :  

```
/TD_git_201                 #ce dépôt
    .git-completion.sh
    .git-prompt.sh
    Exercices_TD_01.md
    Exercices_TD_02.md
    Exercices_TD_03.md      #vous êtes ici !
    README.md
    rajouter_a_bashrc.md

/301                        #exercice 1 du TD 3
/302                        #etc
```

Lorsque c'est pertinent pour la compréhension, par exemple si on change de branche, je mettrai en début de ligne la branche.  
Par exemple :  
`(master) $ git log`
vous indique que vous êtes bien dans la branche master et que vous devez taper la commande `git log`

# Exercice n°1
## Mise en place
Vous êtes dans le répertoire 301.
```bash
git init
echo "Hello" > init.txt
git add .
git commit -m "Crée le fichier initial"
```

`(master) $ git log --oneline`  
devrait donc vous montrer une ligne similaire à  
`77f7daa (HEAD -> master) Crée le fichier initial`

## Ajout d'une branche et modification
```bash
git checkout -b dev
echo ' world !' >> init.txt
git add .
git commit -m "Change le message de bienvenue"
```
## Modification du fichier dans la branche principale (divergence!)
```bash
git switch master
echo ' everybody' >> init.txt
git add .
git commit -m "Change le message de bienvenue (bis)" 
```

> À ce stade on a un fichier init.txt qui existe en deux états divergents.
On peut comparer manuellement en basculant de branche, ou on peut regarder ce qui nous attend en faisant un `diff`.
```bash
(master) $ git diff dev
```
qui vous affichera un truc du genre :  
```
diff --git a/init.txt b/init.txt
index 1800480..cce14e9 100644
--- a/init.txt
+++ b/init.txt
@@ -1,2 +1,2 @@
 Hello
- world !
+ everybody
```
ce qui nous indique que la version ciblée de notre fichier a perdu "world !" mais gagné " everybody".

## Manufactured drama ! (on crée un conflit)
```bash
(master) $ git merge dev
```
Vous devriez vous retrouver avec un message d'erreur :  
```
Auto-merging init.txt
CONFLICT (content): Merge conflict in init.txt
Automatic merge failed; fix conflicts and then commit the result.
philippe@MEDEA:~/301 (master|MERGING) $
```

>Remarquez comme le prompt a changé en `(master|MERGING)`

```bash
cat init.txt
```
doit vous montrer votre fichier en conflit :  
```
 Hello
 <<<<<<< HEAD
  everybody
 =======
  world !
 >>>>>>> dev
```
On voit : 
* `<<<<<< HEAD`  
  indique la branche sur laquelle on se trouve.  
* le contenu en conflit (ici une seule ligne)
* `=======`  
  sépare les deux contenus en conflit
* le contenu venant de l'autre branche
* `>>>>>>> dev`  
  qui finit la zone de conflit et indique d'où vient le contenu (ici la branche `dev`)

## Résolution du conflit
```bash
(master|MERGING) $ echo 'Hello world, hello everybody !' > init.txt
(master|MERGING) $ git add .
(master|MERGING) $ git commit -m "Fusion de dev"
```

Ici on a discuté avec nos collègues et dans un souci de bienveillance, on a décidé de fusionner réellement les deux messages, puisqu'aucun n'est vraiment supérieur à l'autre et qu'on peut tout à fait les faire cohabiter.  
> La résolution de conflit n'est pas un jeu à somme nulle !

## Situation finale
```bash
git log --oneline --graph
```
va vous montrer la situation après nos opérations :  
```
*   136b491 (HEAD -> master) Fusion de dev
|\
| * 1452965 (dev) Change le message de bienvenue
* | 99cab71 Change le message de bienvenue (bis)
|/
* 77f7daa Crée le fichier initial
```

## Bonus 
On peut effacer la branch `dev` et pourtant ne pas perdre le commit venant d'elle. C'est parce que le commit de fusion contient une référence vers `1452965`!
```bash
git branch -d dev
git log --oneline --graph
```
vous le montre clairement :  
```
*   136b491 (HEAD -> master) Fusion de dev
|\
| * 1452965 Change le message de bienvenue
* | 99cab71 Change le message de bienvenue (bis)
|/
* 77f7daa Crée le fichier initial
```
Vous n'avez effacé qu'un pointeur appelé `dev` qui pointait sur `1452965`, rien de plus.  
Et d'ailleur vous pouvez absolument créer la branche à nouveau. Le nom est disponible et la branche nouvellement crée partira de votre commit en cours.  

# Exercice n°2
## Mise en place
```
/TD_git_201                 #le dépôt que vous avez téléchargé
    ...
/301                        #exercice 1 du TD 3
    /.git
    init.txt
/302                        #vous allez être ici
```
> Vous êtes dans le répertoire 302.
```bash
git init
echo "Hello" > init.txt
git add .
git commit -m "Crée le fichier initial"
git checkout -b fix/r-302
echo 'Hello world !' > init.txt
git add .
git commit -m "Change le message de bienvenue"
echo 'Fixed it!' > fix.txt
git add .
git commit -m "Effectue le fix, tant qu'on est là"
git switch master
echo 'Hello everybody :)' > init.txt
git add .
git commit -m "Change le message de bienvenue (bis)"
git diff fix/r-302
```
Vérifions qu'on a bien deux branches prêtes à entrer en conflit :  
```bash
diff --git a/fix.txt b/fix.txt
deleted file mode 100644
index 35171f1..0000000
--- a/fix.txt
+++ /dev/null
@@ -1 +0,0 @@
-Fixed it!
diff --git a/init.txt b/init.txt
index 3c59a58..c7c9ed6 100644
--- a/init.txt
+++ b/init.txt
@@ -1 +1 @@
-Hello world !
+Hello everybody :)
```
## Rebase depuis master
Pour expliquer la situation : on a un développement fait sur une branche de fix (`fix/r-302`) mais entre temps quelqu'un a fait une modification sur la branche dont on est parti (`master`).  
Notre politique d'équipe exige qu'avant qu'on puisse fusionner notre fix, on mette à jour notre travail à l'aune de ce qui a été effectué entre temps.  
Autrement dit : on veut rebaser notre fix sur la branche master. Ensuite seulement on pourra fusionner dans master. 
```bash
git rebase master fix/r-302
```
Hélas le changement sur la branche master entre en conflit avec notre travail :  
```
Auto-merging init.txt
CONFLICT (content): Merge conflict in init.txt
error: could not apply 6b2dc00... Change le message de bienvenue
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
hint: Disable this message with "git config advice.mergeConflict false"
Could not apply 6b2dc00... Change le message de bienvenue
philippe@MEDEA:~/302 (fix/r-302|REBASE 1/2) $
```
>Remarquez à nouveau comment le prompt change pour vous indiquer votre situation : `(fix/r-302|REBASE 1/2)`  
Vous êtes sur la branche en cours de ré-écriture, et vous en êtes au premier commit sur 2 au total.  

On va maintenant résoudre le conflit *après en avoir parlé avec nos collègues !*  
Un fix est généralement un travail prioritaire et urgent, mais cela ne dispense pas des bonnes pratiques.  
Sauf qu'ils ne sont pas là et que ça ne peut pas attendre...  
On va prendre une décision et la justifier dans notre message de commit, tant pis. Mais il faudra en discuter au plus tôt.  

Comme dans l'exercice 1, on peut constater que c'est bien le fichier init.txt qui est en conflit, et qui a été modifié en conséquence.  
On peut aussi regarder `git status` et remarquer qu'il est un peu différent d'habitude.  

Résolvons le problème :  
```bash
echo 'Hello one and all !' > init.txt
git add .
git rebase --continue
```
> Notez-bien la différence avec le conflit de l'exercice 1.  
> Ici on a bien eu un `git add .` mais pas de `git commit`.  
> On signifie la résolution en utilisant `git rebase --continue`

Vous devriez vous retrouver dans votre éditeur par défaut, car rebase vous propose l'option de modifier le message de commit que vous venez de modifier. C'est l'occasion de justifier votre choix de résolution !  
>Remarquez que vous avez fini l'opération `rebase` sur la branche déplacée, même si vous étiez ailleurs (`master`) au départ.

```bash
(fix/r-302) $ git log --oneline
dc3f58b (HEAD -> fix/r-302) Effectue le fix, tant qu'on est là
db13440 Change le message de bienvenue (conflit résolu)
b504d8e (master) Change le message de bienvenue (bis)
85b08ae Crée le fichier initial
```
Vous voyez comme le message de commit a bien changé (si vous l'avez fait!) et la branche `fix/r-302` est maintenant bien directement à la suite de la branche `master` dans l'historique.  

## Fusion sur master, avec écrasement
Maintenant si nous voulons fusionner notre fix, nous savons qu'il est prêt et les conflits ont déjà été résolus.  Mais pas besoin de garder tous nos commits, on va juste fusionner un commit unique de fix.  
```bash
git switch master
git merge fix/r-302 --squash
```
Comme expliqué pendant le cours, cette fusion ne fait que mettre les choses en place, et laisse à l'utilisateur le choix de faire le commit.  

```
Updating b504d8e..dc3f58b
Fast-forward
Squash commit -- not updating HEAD
 fix.txt  | 1 +
 init.txt | 2 +-
 2 files changed, 2 insertions(+), 1 deletion(-)
 create mode 100644 fix.txt
 ```
 `git status` nous montre que les fichiers ont été modifiés et sont prêts pour le commit :  
 ```
 On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   fix.txt
        modified:   init.txt
 ```
 ```bash
 git commit -m "🛠️ FIX : Répare le bug R-302"
 git log --graph --oneline master fix/r-302
 ```
 Et vous devriez voir les deux branches distinctes :  
 ```
 * 7bbd472 (HEAD -> master) 🛠️ FIX : Répare le bug R-302
| * dc3f58b (fix/r-302) Effectue le fix, tant qu'on est là
| * db13440 Change le message de bienvenue (conflit résolu)
|/
* b504d8e Change le message de bienvenue (bis)
* 85b08ae Crée le fichier initial
```
Vous voyez que vous avez maintenant un seul commit `7bbd472` qui représente le travail effectué dans les deux commits de la branche de fix.  
Comme dans l'exercice 1, on peut effacer la branche `fix/r-302` par contre ici elle disparaitra entièrement du log.  
```bash
git branch -d fix/r-302
```
Mais git voyant que la branche n'a jamais été fusionné, vous prévient !  
```
error: the branch 'fix/r-302' is not fully merged
hint: If you are sure you want to delete it, run 'git branch -D fix/r-302'
hint: Disable this message with "git config advice.forceDeleteBranch false"
```

À vous de décider si la branche a une utilité. Cela dépend de votre méthode de travail, de la longueur de la branche, etc.  
Et bien sûr même si vous l'effacez, les commits seront encore présents longtemps dans votre reflog.
Pour ma part je force la suppression avec :  
```bash 
git branch -D fix/r-302
```
