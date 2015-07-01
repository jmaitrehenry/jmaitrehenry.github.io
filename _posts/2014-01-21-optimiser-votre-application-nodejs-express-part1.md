---
layout: post
title: Optimiser votre application NodeJS + Express
date: 2014-01-21 22:29:46.000000000 -05:00
---
# Optimiser le système (linux / CentOS)

La première des choses est d'avoir un système optimal sur le quel rouler notre application. Si le système est mal configuré, on pourra optimiser autant que l'on veut l'application, on n'aura jamais des performances optimal, et c'est pourquoi, cela va être mon premier axe d'optimisation.

## Optimiser les paramètre du Kernel

Le kernel, par défaut, possède des limites assez basse au niveau du réseau, on va tenter des les augmenter un petit peu.

### Augmenter le backlog de socket pour le kernel

Par défaut, le paramètre SOMAXCONN est à 128 sous CentOS ainsi que dans la plus part des distribution linux. Se paramètre détermine le nombre maximum de _listen()_ qu'un socket peut accepter simultanément. Cette limite est trop basse pour les serveurs actuels et peut dégrader les performances de notre application si on a plus de connection simultannée que cette limite.

Pour ceux qui disent que cette limite est dangereuse car elle peut utiliser trop de mémoire, pour un serveur 64b, chaque listen consomme 160b en mémoire, avec une valeur à 8k, cela représente seulement 1.25M.

Pour la bonne valeur à mettre ici, cela va dépendre, généralement, la bonne limite est entre 128 et 8192.

### Permettre au système de réutiliser les sockets plus utilisé (optionel)

Un des problèmes que j'ai rencontré est le nombre énorme de conenction en _TIME_WAIT_. J'ai pas de solution magique pour cela, sous très fort load, j'en ai encore beaucoup, par contre, ce qui peut aider le système à réutiliser ses sockets en fin de vie, les paramètres suivants peuvent être changé :

    tcp_tw_reuse = 1
    tcp_fin_timeout = 15 (in second)

Par défaut _tcp_fin_timeout_ est à 60s, c'est à dire que le kernel va attendre jusqu'à 60s avant de fermer la connection si c'est nous qui fermons la connection et non le client.

## Augmenter la limite de _file descriptor_

C'est l'un des points les plus importants. Sur CentOS, la limite est de 1024 _file descriptor_ par application, par défaut. C'est en gros le nombre de fichier que l'application peut d'avoir ouvert en même temps, mais comme, sous linux, tout est fichier, cela inclue le nombre de connection. C'est l'un des principaux problèmes de performance. Pour le modifier, il y a deux moyen, soit on le défini au niveau de l'utilisateur, soit de la session en cours.

De mon bord, l'application nodejs possède soit son propre utilisateur, soit il utilise _nobody_, et j'augmente la limite au niveau du système.

### Augmenter la limite pour la session en cours

	ulimit -n 9999
    node app.js
C'est aussi simple que cela.

### Augmenter la limite pour un utilisateur

	vim /etc/security/limits.d/99-nobody
    #user  soft/hard/all option    value
	nobody       -       nofile    9999

### Vérifier la limite pour le processus en cours

Si vous voulez vérifier la limite pour le processus en cours, vous devez avoir son PID :

	#  ps ax | grep node
    PID  [...]  COMMAND
    5699 [...] node app.js

Ensuite, on va voir les limites du PID :

	cat /proc/__PID__/limits
    Limit                     Soft Limit           Hard Limit           Units
    [...]
    Max open files            9999                 9999                 files

Et voilà, on a bien la limite de définie avant.
