# Petit script pour ajouter la branche git dans le prompt
Après avoir déposé les fichiers git-prompt.sh et git-completion.bash dans le répertoire ~ on peut ensuite les activer dans le `.bashrc`

```bash
# enable git-prompt options
. ~/.git-prompt.sh
# enable git completion
source ~/.git-completion.bash
```

Puis on peut modifier le prompt pour afficher la branche active quand on est dans un repo git, et même colorer le nom si on veut indiquer une branche spéciale (ici main ou master, mais c'est facile à modifier !)

```bash
# Modifié pour avoir le nom de la branche GIT en cours.  Dépend de l'existence de git-sh-prompt (ligne 4)

if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]$(if [ "$(__git_ps1 %s)" = main ] || [ "$(__git_ps1 %s)" = master ]; then echo "\[\033[01;31m\] ($(__git_ps1 %s))"; elif [ -n "$(__git_ps1 %s)" ]; then echo "\[\033[01;33m\] ($(__git_ps1 %s))";fi)\[\033[00m\] \$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w$(__git_ps1 " (%s)")\$ '
fi
unset color_prompt force_color_prompt
```

> Si vous avez des problèmes d'affichage du prompt, vérifiez si vous avez pas une version de git-prompt déjà installée dans votre système sur /usr/lib/git-core/git-sh-prompt
> Auquel cas, remplacez la ligne `. ~/.git-prompt.sh` par `. /usr/lib/git-core/git-sh-prompt`