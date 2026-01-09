# Qualité de Developpement - Git Avancé - Examen
## Partie 1
### A - Qu'est ce que git ?
réponse 4.

Pour les quelques personnes qui ont répondu 1. je précise qu'une interface en ligne de commande (CLI en anglais) n'est pas un langage.
Un langage de script ça serait par exemple celui qu'inteprète `bash` (ou tout autre shell que vous utilisez).
Et effectivement, on peut écrire des scripts pour bash *qui vont faire appel* à git, au travers de cette interface en ligne de commande.

Mais git, et c'est ce qui m'importait que vous compreniez pour ce cours, c'est avant tout un logiciel de gestion de versions de fichiers. Et je ne l'ai pas écrit dans la question, mais il est également essentiel que vous compreniez l'épithète "décentralisé" qui est généralement associé. Comme je vous l'ai montré dans le dernier TD : vous n'avez pas besoin d'un serveur central pour utiliser git. Vous n'avez même pas besoin d'une *version serveur*.

### B - Pour vérifier si vous avez bien sauvegardé vos modifications vous tapez :
réponse 3 : `git status`

La commande `status` vous dit où vous en êtes.
Si vous venez de modifier des fichiers et que vous ne les avez pas encore mis dans l'index, pour les sauvegarder ensuite via un `commit`, cela apparaitra dans `git status`.  
Attention, vous pouvez mettre un fichier dans l'index avec `git add` et continuer à le modifier. La version envoyé dans le commit sera `celle que vous avez mise dans l'index`, pas celle en cours dans votre répertoire de travail.
Et cela, `git status` vous le montrera aussi : vous aurez votre fichier à la fois dans la partie "Changes to be commited" (index) et à la fois dans le répertoire de travail "Changes not staged for commit".

### C - Comment indiquer à git quels fichiers vous voulez mettre dans votre prochaine commit ? 
réponse 3 : `git add .`

Pour préparer un commit on doit d'abord *ajouter* les fichiers modifiés à l'index. Donc `add`. 
Certains ont précisé que ce n'est évidemment pas la seule méthode et c'est très bien d'en être conscient, bravo.

### D - Vous venez de faire un commit A avec le message "C'est fini", puis vous faites un `git commit -m "C'est fait" --amend`

réponse 3

C'était la question pour voir si vous aviez bien écouté toutes les fois où je me suis repris pour bien préciser que "dire qu'on modifie le message d'un commit est un abus de langage. Git ne modifie pas les commit une fois créés".  
C'est une notion vraiment fondamentale à la compréhension de beaucoup de choses dans git.
Et donc non, le commit A n'a pas changé de message.  
Le commit A a disparu *du log* et à sa place est apparu un nouveau commit avec exactement les mêmes modifications aux fichiers, mais avec un hash et un message de commit différents.

Si ce n'est pas acquis, je vous invite à refaire l'expérience en ligne de commande et regarder les hash de vos commits.

### E - Je suis sur la branche master et je veux fusionner la branche feat.

réponse 4 : `git merge feat`

Y'a pas grand chose à dire : vous vous souvenez que `merge` est la commande pour fusionner une branche, ou vous ne savez pas travailler avec des branches :/

### F - Je veux effacer la branche feat ...

réponse 4 : `git branche -D feat`

Certains ont bien précisé que l'option `-D` allait forcer git à effacer la branche, là où `-d` nous aurait indiqué si l'opération n'était pas une bonne idée. Dans les deux cas le d représente le verbe *delete*.

### G - Je viens d'effacer un fichier par erreur. Il était dans mondernier ocmmit et je n'ai rien touché d'autre dans mon dépôt.

réponse 1 - `git reset --hard HEAD`

Les autres méthodes ne vont pas marcher, je vous assure que j'ai testé moi-même en ligne de commande avant de les écrire ;)
Le --hard est ici primordial pour bien écrire/effacer les fichiers. Et HEAD vous indique que vous revenez au dernier commit fait, à ne pas confondre avec le commit d'avant `HEAD^`.
J'ai bien précisé qu'on avait rien fait d'autre pour que ceux qui étaient conscients que `reset` est une opération radicale soient rassurés quant au fait qu'on allait perdre aucun travail.

Ce n'était pas la seule façon de faire, bien sûr. Mais la plus simple dans la situation que j'avais donnée.

### H - Je viens de finir un rebase et après avoir tapé git status je vois que je me suis trompé.

réponse 1 : `git reset --hard ORIG_HEAD`

Encore une fois, je précise le contexte pour qu'il n'y ait pas d'ambiguité. On a fait un rebase, on a fait un git status, donc le pointeur `ORIG_HEAD` pointe toujours sur le commit juste avant notre rebase. C'est la seule réponse appropriée.
`git rebase --abort` n'est utilisable que *pendant* le rebase. Et vous savez que vous êtes en cours de rebase grâce à votre prompte qui vous l'indique. Là on "vient de finir un rebase" donc c'est trop tard pour un `--abort`.

### I - Je crée un fichier .gitignore contenant 
```
.*
~.gitignore
```
quel fichier va rester visible dans mon `git status` ?

réponse 1 `.gitignore`

La dernière ligne `!.gitignore` indique explicitement à git de *ne pas ignorer* ce fichier. Donc par définition il sera visible. Donc vous n'avez même pas besoin de regarder les autres réponses.  
Et oui j'ai testé toutes les réponses avant de les écrire ;)

### J - `git rebase feat fix --onto master` sur l'arbre suivant : 
```
<- A <- C   --- master
   \
    B <- D  --- feat
     \
      E     --- fix
```

réponse A 

```
<- A <- C     --- master
   \    \
    \    E'    --- fix
     \
      B <- D  --- feat
```

Beaucoup de monde a répondu B ce qui indique que vous avez tous bien utilisé l'indice du `--onto master`, mais malheureusement il fallait aussi bien se souvenir de l'ordre des paramètres pour savoir *ce qu'on va déplacer* sur master !  

Mais vous auriez pu regarder B et réaliser qu'on ne pouvait pas arriver à cet arbre en une seule opération. Même sans analyser sa structure vous auriez pu voir les B' et E' qui vous indiquaient un premier rebase, et D" qui indiquait un deuxième.  
Pour être clair, l'arbre B peut s'obtenir depuis la situation initiale en faisant :  
`git rebase fix feat`
qui vous donne 
```
<- A <- C         --- master
   \
    B <- E        --- fix
          \
           D'     --- feat
```
puis `git rebase feat master` qui vous donne bien 
```
<- A <- C         --- master
         \
          B' <- E'        --- fix
                 \
                  D''     --- feat
```
Pourquoi on voudrait faire tout cela est une autre discussion ;)

Une autre approche était de comprendre de par les noms des branches qu'on avait fait un fix (correctif rapide) sur une branche (feat) par erreur et qu'on voulait le déplacer directement sur master, ce que représentait bien la situation A.

## Partie 2

Pour ces questions, qui se suivaient, j'ai essayé de répondre aux problèmes spécifiques de chacun sur leur copie, et je voulais donc ici simplement indiquer les notions essentielles que je recherchais dans vos réponses, et dont la présence ou l'absence m'a guidé pour vous noter.  
De manière générale, je n'ai pas *enlevé* de points si vous disiez des bêtises après avoir dit des choses justes. Parfois vous aviez des réponses un peu floues mais pas loin du but, et votre texte m'a permis de décider si vous aviez compris un peu ou si vous étiez à côté de la plaque, mais mon but n'était pas de punir les erreurs.

### A - les deux approches pour fusionner des branches
Je voulais que vous m'indiquiez les deux grandes approches, à savoir **fusion directe** qui va créer un commit de fusion, et fusion **après rebase** qui va donc être une fusion "fast forward".  
Je ne vous notais pas sur une liste d'avantages et d'inconvénients car ceux-ci vont dépendre du contexte dans lequel vous utilisez ces approches, mais au moins savoir que ces approches existent était pour moi le coeur du problème.  

### B - conflit entre branches
Je voulais que vous identifiez qu'il y avait un **conflit**, qu'il allait falloir **résoudre** ce conflit, en **parlant** avec des collègues (si possible) puis en appliquant la décision prise **en éditant les fichiers**.  

Il y a eu plus ou moins de détails techniques, mais la notion qui a manqué le plus souvent c'était le fait de **parler**.  
Et je suis désolé pour ceux qui ont fait preuve de précision technique, mais la communication reste une compétence absolument indispensable quand on travaille en équipe donc je ne pouvais pas vous mettre tous les points sans cette notion que j'ai explicitement rabâchée (parce que je sais combien elle est souvent ignorée par les élèves autant que les profs d'info).  

### C - identifier un rebase
Il fallait juste me dire qu'on avait fait un rebase. Préciser que c'était un rebase de la branche dev sur la branche trunk était une bonne idée.  
Je voulais aussi la *bonne* commande `git rebase trunk dev` car c'est un piège tellement commun et qui cause tant de maux, que je voulais vraiment que vous ayiez acquis cette notion.  
J'aurais accepté `git rebase trunk` si vous aviez précisé que vous étiez sur la branche `dev`.  

### D - expliquer les différences entre la situation initiale et la situation dans la question C.  

Alors toutes mes excuses d'avoir utilisé une lettre "C" à la fois pour un commit et pour une question. Normalement ça n'aurait pas dû vous mettre en erreur, mais bon.  
Et non, il ne s'agissait pas d'un jeu des sept erreurs.  
Je voulais simplement que vous me décriviez avec des mots pourquoi on avait soudain des commits à un endroit différent et avec des noms différents.  
Je voulais la notion de **recréer les commits B et D à partir de C**.  
La résolution de conflits éventuels était un plus qui compensasit une éventuelle faiblesse de votre réponse ailleurs.  
Je voulais que vous me disiez clairement que **non**, le rebase n'était pas suffisant pour *"intégrer vos changements dans trunk"*.  
Il fallait encore **fusionner dev sur trunk**. Là aussi, préciser que c'était du *fast-forward* était optimal mais pas nécessaire pour avoir les points.  
Idem pour le `push`, mais bravo aux pointilleux, ça vous servira dans votre carrière.

### E - expliquer la fusion finale
Alors là encore pardon pour le nom de commit qui aura pu en induire certains en erreur, mais j'espèrais que vous aviez bien écouté lors du dernier TD et que vous aviez tou.te.s reconnu une **fusion avec écrasement**.  
C'était vraiment tout ce que j'attendais.  
Préciser que le `merge --squash` ne faisait *pas* le commit final était optionnel (car git vous le rappelera explicitement), mais bravo pour ceux qui se sont souvenus.  

Certains on émis des hypothèses de rebase mais non.  
Pour ma défense, et comme j'ai essayé de l'expliquer lors du TD, lorsqu'on fait un squash, on fait péter généralement tous les commits superflus, et *le plus logique* c'est de ne garder que le dernier commit, ici D' et de fusionner seulement celui-là vers la branche cible, d'où le nom de D" choisi. Ce n'est pas *le même commit* puisque le hash est différent, mais c'est le même contenu, donc D".  
J'ai hésité à l'appeler "B'D'" mais ça me semblait un peu trop facile.

N'hésitez pas à poser vos question sur mon email de l'IUT - philippe.jofresa@iut-rodez.fr
Je vais peut-être pas le consulter souvent, mais je m'efforcerai de répondre si je vois vos emails.