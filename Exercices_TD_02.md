# Exercices du TD n°1

## Exercice n°1 : reflog HEAD@{n} et
On va juste revenir sur cette notation relative, utile dans certains cas.

### Mise en place
````bash
mkdir td02ex01
cd td02ex01
git init
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
### 2. Vérifiez que vous pouvez bien encore voir le commit précédent
````bash
git reflog
# Va vous afficher un truc du genre :
b5b728d (HEAD -> master) HEAD@{0}: commit (amend): Ajoute .gitignore
2f5abb2 HEAD@{1}: commit: AJoute gitnore
````
Remarquez ici la notation absolue (les hash en début de ligne)  
mais aussi la notation relative `HEAD@{1}` qui par exemple vous permet ici de revenir en arrière _dans le temps_.  
````bash
git rev-parse HEAD~1
git rev-parse HEAD@{1}
````
Comparez ces valeurs. `HEAD~1` devrait échouer si vous n'avez pas de commit parent. `HEAD@{1}` vous permet de cibler le commit corrigé, qui en fait existe toujours dans votre dépôt.  

## Exercice n°2 : ORIG_HEAD
Bien pratique quand on fait une manipulation de l'historique et qu'on veut finalement revenir en arrière !  
````bash
touch secrets.txt
git add .
git commit -m "Ajoute secrets.txt"
````
Remarquez que secrets.txt a bien été ajouté car vous l'avez créé dans le répertoire racine, pas dans `/secrets`  
Bien sûr c'est une erreur, donc faisons disparaitre ce commit !  
### 