# Exercices du TD n°1

## Mise en place
> Attention, si vous avez cloné ce dépôt, il faut maintenant en sortir pour créer un répertoire de travail dans lequel nous allons faire tous ces exercices !  

Vous devriez avoir une hiérachie de ce type :
````
/td02                       #ce dépôt
    .git                    #invisible
    Exercices_TD_01.md      
    Exercices_TD_02.md      #ce fichier
    README.md
/td02_exos                  #là où vous allez faire les exos suivants
    .git                    #une fois que vous avez initialisé
````

> Avant de commencer l'exercice n°1, 

Mettez-vous donc dans `/td02_exos` et exécutez
````
git init
````
## Exercice n°1 : reflog et HEAD@{n}
On va juste revenir sur cette notation relative, utile dans certains cas.

### Mise en place
````bash
echo 'Hello world' > main.txt
git add main.txt
git commit -m "Ajoute main"
echo '/secrets/**' > .gitignore
git add .gitignore
git commit -m "AJoute gitnore"
````
Oups ! Le message de commit était un peu de travers. Corrigeons !  
### 1. Correction du commit précédent
````bash
git commit -m "Ajoute .gitignore" --amend
````
### 2. Vérifiez que le commit corrigé existe encore
Vous pouvez voir que le commit corrigé "n'existe plus" :
````bash
git log --oneline
# L'option --oneline va écrire chaque commit sur une seule ligne
# Va vous afficher quelque chose comme :
1441231 (HEAD -> master) Ajoute .gitignore
699d3cd Ajoute main
````
Mais si vous regardez avec `reflog` : 
````bash
git reflog
# Va vous afficher un truc du genre :
1441231 (HEAD -> master) HEAD@{0}: commit (amend): Ajoute .gitignore
c73123d HEAD@{1}: commit: AJout gitnore
699d3cd HEAD@{2}: commit (initial): Ajoute main
````
On voit bien ici que notre commit avec son message mal écrit existe toujours.  
Remarquez au passage la notation absolue (les hash en début de ligne)  
mais aussi la notation relative `HEAD@{n}` qui par exemple vous permet ici de revenir en arrière _dans le temps_.  
````bash
git rev-parse HEAD~1
git rev-parse HEAD@{1}
````
>Comparez ces valeurs.  

`HEAD~1` va vous renvoyer le hash du commit parent de HEAD.  
Ici le commit initial `699d3cd`  
Au contraire, `HEAD@{1}` vous permet de cibler le commit `c73123d` qui a été corrigé, et normalement n'existe plus.  
Vous pouvez bien vérifier, il n'est pas présent dans `git log`.  

### 3. Créons un tag
````bash
git tag ex01
````
Ça nous permet de revenir à notre repo propre si on fait des bêtises par la suite.  

## Exercice n°2 : ORIG_HEAD
Bien pratique quand on fait une manipulation de l'historique et qu'on veut finalement revenir en arrière ! 
### Mise en place
Vous êtes toujours à la racine de votre dépôt, qui contient à ce stade :  
````
/td02_exos                  #Vous êtes ici
    .git
    .gitignore
    main.txt
````
Et votre historique contient deux commits :  
````
1441231 (HEAD -> master) Ajoute .gitignore
699d3cd Ajoute main
````
### 1. Créons un commit erroné
````bash
touch secrets.txt
git add .
git commit -m "Ajoute secrets.txt"
````
Remarquez que `secrets.txt` a bien été ajouté car vous l'avez créé dans le répertoire racine, pas dans un sous-répertoire `/secrets`, qui aurait été ignoré.    
Bien sûr c'est une erreur, donc faisons disparaitre ce commit !  
### 2. Revenons en arrière
````bash
git reset --hard HEAD~1
````
Le fichier `secrets.txt` doit avoir disparu.  
### 3. Revenons vers le futur !
Faisons réapparaitre le fichier `secrets.txt`.  
````bash
git reset --hard ORIG_HEAD
````
Et hop! Comme si rien ne s'était passé.
Vérifiez le log et constatez la présence du commit "Ajoute secrets.txt"
Regardez le reflog et constatez la différence: 
````git 
f977168 (HEAD -> master) HEAD@{0}: reset: moving to ORIG_HEAD
1441231 HEAD@{1}: reset: moving to HEAD~1
f977168 (HEAD -> master) HEAD@{2}: commit: Ajoute secrets.txt
1441231 HEAD@{3}: commit (amend): Ajoute .gitignore
c73123d HEAD@{4}: commit: AJout gitnore
699d3cd HEAD@{5}: commit (initial): Ajoute main
````
Voyez comme la première ligne ne vous donne pas le message d'un commit, mais plutôt l'action qui vous a amené à ce commit. Ici, votre `git reset`.

> Quelles autres manières de revenir en arrière auriez vous pu utiliser ?  
> Si vous faites des essais, ou des allers-retours, remarquez comme votre `git reflog` se remplit en conséquence.

Pour fini l'exercice, déplacez le fichier `secrets.txt` dans un sous-répertoire `/secrets` et assurez-vous d'avoir effacé le commit ajoutant `secrets.txt` à l'historique.

## Exercice n°3 : Première branche, première erreur
### Mise en place
Vous êtes toujours dans votre répertoire principal qui contient désormais :
````
/td02_exos                  #Vous êtes ici
    .git
    .gitignore
    main.txt
    /secrets
        secrets.txt
````
Votre historique doit avoir seulement deux commits :
````
1441231 (HEAD -> master) Ajoute .gitignore
699d3cd Ajoute main
````
### 1. Créons une branche et faisons des modifications
````bash
git branch -c dev
mkdir window
cd window
echo "Ma première feature !" >> main.txt
git add .
git commit -m "Ajoute une fenêtre"
````
Super ! Sauf que si vous vérifiez maintenant votre historique avec `git log`, vous pouvez constater que votre commit (et donc le code qui va avec) est en fait dans la branche master !  
### 2. Corrigeons
On veut que le sous-répertoire n'existe que lorsqu'on est dans la branche `dev`.  
> Vous pouvez exécuter les commandes ci-dessous dans le sous-répertoire `window`, mais comme on va faire disparaitre le sous-répertoire en question de la branche dev, vous aurez une erreur `fatal: Unable to read current working directory: No such file or directory` après avoir fait votre premier `git reset`. Si ça arrive, remontez à la racine de votre dépôt.

````bash
git switch dev
git reset --hard master
#les deux branches pointent maintenant au même endroit
git switch master
git reset --hard dev
#votre branche master est bien revenue au propre.
````
Vérifiez bien le log de vos branches. Si vous n'êtes pas dans la branche dont vous voulez le log, vous pouvez tapez son nom en argument.  
````bash
git log dev
````
> Quelles autres références pouvions nous utiliser pour faire nos reset ?

## Exercice n°4 : `rebase` - les choses sérieuses commencent !
À ce stade votre répertoire de travail contient :
````bash
# /main.txt
# /secrets/secrets.txt
# si vous êtes sur la branche dev vous avez également 
# /window/main.txt
````

### 1. Rajoutons quelques commits pour notre exercice
````bash
#on travaille sur notre feature
git switch dev
echo 'Les choses avancent bien pour notre nouvelle fenêtre' >> window/main.txt
git add . 
git commit -m "Avance la feature window"

#on doit basculer sur autre chose d'important direcement sur master
git switch master
echo 'On modifie aussi la branche principale pour dautres raisons' > main.txt
git add . 
git commit -m "Hotfix sur master"

#on modifie un peu tant qu'on y est
echo 'Utilise la feature Window1 pour afficher le message message.txt' >> main.txt
echo 'hello world!' > message.txt
git add . 
git commit -m "Refacto sur master"

#on revient sur notre feature
git switch dev
echo 'Ça avançait tellement bien ! Jai perdu le fil maintenant.' >> window/main.txt
git add . 
git commit -m "Évolution sur la feature"
````
### 2. Déplaçons des commits
On décide de déplacer les commits fait sur master dans la feature, en fait.  
En fait dans master on a utilisé la feature qui n'existe pas encore !  
Donc on va tout déplacer.  
````bash
git rebase dev master
````
> Vous allez probablement avoir des conflits car /main.txt a bien changé et git n'arrive pas à deviner quelle version est la bonne.
````bash
CONFLICT (content): Merge conflict in main.txt
error: could not apply 9c5f253... Hotfix sur master
Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".
````
Pour résoudre le conflit, suivez les instructions !  
Il va falloir éditer le fichier coupable. Pour les besoins de l'exercice, vous pouvez simplement écraser son contenu en faisant par exemple :  
````bash
echo 'affiche window1 avec message.txt' > main.txt
git add .
git rebase --continue
````
Regardez vos logs pour voir les commits qui y sont incorporés
````bash
git log master
git log dev
````

### 3. Déplaçons nos pointeurs de branche
````bash
git switch dev
git reset --hard master
git switch master
git reset --hard ex02
````
> Normalement ORIG_HEAD nous permet d'annuler la dernière grosse modification, comme un rebase. Mais ici comme nous avons ensuite fait deux reset, ORIG_HEAD ne nous permet pas d'annuler le rebase !  
Sauriez-vous rétablir la situation initiale ?

## Exercice n°5 - Déplaçons une sous-branche !
En pratique, on va avoir une branche master intouchable,  
une branche de dev où tout le monde pousse ses modifs une fois stables  
et chacun se crée une branche de feature pour travailler dans son coin.
Typiquement, on merge un feature dans `dev` et `dev` dans `master`.  
Mais il peut arriver, pour une raison X ou Y qu'on veuille migrer un commit de feature directement sur `master`. `Let's rebase !`  

### Mise en place
Ajoutons quelques commits et une branche.
Revenez à la **racine** du projet puis :
````bash
git switch master
touch some_stuff.txt
git add .
git commit -m "Ajout de stuff"

echo 'This is really useful' > some_stuff.txt
git commit -a -m "Modifie stuff"

git switch dev

git branch -c feat/win
git switch feat/win
echo 'Une feature qui en fait va servir à tout le monde' > toto.txt
git add .
git commit -m "Création d'un utilitaire toto"

git switch dev
echo 'On ajoute des options à notre feature window' >> window/main.txt
git commit -a -m "Développe la feature window"
````

À ce stade on a trois branches, chacune un peu avancée dans son coin
>déplacez la branche feat/win directement sur master