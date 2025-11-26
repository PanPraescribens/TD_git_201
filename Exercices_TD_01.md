# Exercices du TD n°1

## Exercice 01
### 1. Créez un répertoire de travail et allez dedans  
```bash
mkdir td01ex01  
cd td01ex01
````
### 2. Créez votre repo  
````bash
git init
````  
### 3. Commencez à créer des fichiers  
````bash
touch main.txt                          #crée un fichier vide  
echo 'MonMotDePasse!' > secrets.txt     #crée un fichier avec un peu de texte
````
### 4. Ajoutez tout à l'index git  
````bash
git add .
````
### 5. Vous avez été un peu vite ; enlevez le fichier secrets.txt avant de faire un commit !  
````bash
git restore --staged secrets.txt
````

NB : vous auriez pu retirer le fichier de l'index autrement;  
testez si vous avez le temps ! 

### 5<sup>bis</sup>. Rajoutez le fichier et enlevez le de l'index à nouveau

````bash
git add secrets.txt
git rm --cached secrets.txt  
# ou encore 
git add secrets.txt
git reset HEAD secrets.txt
```` 
### 6. Faites un commit 
````bash
git commit -m "Ajoute main.txt"
````

## Exercice 02
### 1. Dupliquons le repo créé dans l'exercice 1
````bash
cd ..
cp -R td01ex01 td01ex02
````
NB : on a tout copié, y compris .git donc c'est un dépôt git près à utiliser! Et comme il n'est pas lié à un dépôt distant, pas de problème. 

### 2. Éditons main.txt puis faisons un commit
````bash
echo 'Hello world!' >> main.txt #pour ajouter un peu de texte dans un fichier existant sans l'écraser
git add .
git commit -m "Ajoute hello dans main"
````
Oh non ! On a encore ajouté secrets.txt !  
### 3. Enlevons secrets.txt de notre dépôt
````bash
git rm secrets.txt
git commit --amend
````
Le `commit --amend` a donc écrasé notre commit précédent. Mais maintenant on a perdu secrets.txt entièrement !  

### 4. Revenons en arrière
````bash
git reset --hard HEAD~1 #revient au commit parent "Ajoute main.txt"
````
Problème : on a certes fait disparaitre notre erreur, mais on a également fait disparaitre nos modifications dans main.txt ! 
````bash
cat main.txt
````
On voit que le fichier est vide.  
Comment le récupérer ?  
On sait qu'il existait avec les modifications dans le dernier commit maintenant disparu.  
### 5. Récupérons le fichier main.txt modifié
````bash
git reflog
    #va nous montrer le commit "Ajoute hello dans main"
    #notez bien la référence au commit. Remarquez également la référence relative qui est du style HEAD@{n}
    #Encore une notation alternative qui vous permet de naviguer dans le reflog!
git checkout hash-du-commit-Ajoute-hello -- main.txt
    #ATTENTION aux espaces autour du -- 
````
Normalement vous avez à ce stade récupéré la version main.txt avec "hello world" dedans.  
Vérifiez que le fichier est bien indexé avec `git status` sinon ajoutez-le.  
Vous n'avez plus secrets.txt dans le répertoire.  

### 6. Ajoutez un fichier .gitignore
````bash
#normalement vous êtes dans votre répertoire /td01ex02
echo '/secrets/**' >> .gitignore
mkdir secrets
echo 'MonSecret' >> /secrets/secrets.txt
````
Normalement à ce stade vous pouvez créer ce que vous voulez dans le sous-répertoire /secrets et si vous faites un `git status` ça n'apparaitra pas.
### 7. Commit final
````bash
git add .       #ne devrait ajouter que .gitignore
git commit -m "Ajoute .gitignore"
git tag ex02    #crée un tag, qui vous permet de faire référence à un commit spécifique plus facilement.
````
