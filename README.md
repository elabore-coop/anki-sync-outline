# Anki Sync Outline : synchroniser un deck avec un document outline

Importer un document depuis Outline pour l’ajouter dans un Deck Anki

> Permet une synchronisation (quotidienne) d’un packet en nourissant le paquet à plusieurs depuis outline (outils prise de note collaboratif).
> L’idée est de pouvoir contribuer à plusieurs à la création de deck, par de l’écriture simple, l’écriture des questions permet aussi bien d’apprendre que de faire les quizzs de cartes.

Nécessite d’avoir Anki installé en local sur son ordi, et outline (mais le processus pourrait-être adapté un n’importe quel autre outil, format texte).

## Ressources :

** [Anki Flashcards](https://apps.ankiweb.net/) ** : Logiciel open-source réputé pour l’apprentissage (avec beaucoup de plugins + grosse communauté), peut se synchroniser entre différents appareil (ordi, mobile)

** [AnkiConnect](https://ankiweb.net/shared/info/2055492159) ** : Greffons qui permet d’envoyer des commandes à anki en HTTP (donc de communiquer avec anki par script)

** [Outline](https://www.getoutline.com/) ** : Logiciel de gestion de notes en ligne, possède [une belle API](https://www.getoutline.com/developers)


## Explication :

Besoin de conserver chacun son compte, pour enregistrer la progression.
Donc il n’est pas vraiment possible d’avoir un Deck commun en ligne.
De plus ce sont des informations plutôt privées, nous ne souhaitons pas
construire des decks public sur Ankiweb.

Mon souhait est que tout le monde puisse participer à la création du deck. Et conserver sa progession individuelle.

## Action du script :

- Téléchargement du document au format .txt
- Comparaison avec l’ancien document
- SI CHANGEMENT :
    - Lancement de Anki
    - Ajout du document à Anki
    - ACTION REQUISE : validation de l’ajout des cartes manuellement.

## Utilisation :

### Prérequis

1. Installer anki : https://apps.ankiweb.net/
2. Installer le [greffon AnkiConnect](https://ankiweb.net/shared/info/2055492159) : https://docs.ankiweb.net/addons.html
3. Cloner le repos
4. Remplir le `.env` basé sur `.env.sample` (Pour récupérer la clé d’API outline )

Exécuter le script pour tester !

### Mise en place d’une synchronisation quotidienne

Je propose pour la synchro quotidienne de créer un service :

Crée le fichier suivant dans `/etc/systemd/system/anki_sync.service` :
```
[Unit]
Description=Synchroniser Anki avec Outline

[Service]
Type=simple
ExecStart=/chemin/vers/ton/script/anki_sync.sh
```
Crée le fichier suivant dans `/etc/systemd/system/anki_sync.timer` :
```
[Unit]
Description=Planification quotidienne pour Anki Sync

[Timer]
OnCalendar=*-*-* 08:00:00
Persistent=true

[Install]
WantedBy=timers.target
```
Explication :

    OnCalendar=-- 06:00:00* : Exécute la tâche tous les jours à 6h00 du matin.
        * (année, mois, jour) signifie "tous les jours".
        06:00:00 représente l'heure à laquelle la tâche sera lancée (6h00 du matin).
    Persistent=true : Si l'ordinateur était éteint à l'heure prévue, la tâche sera exécutée au prochain démarrage.

Après avoir créé les fichiers, exécute les commandes suivantes pour activer et démarrer le timer :
```
sudo systemctl daemon-reload
sudo systemctl enable anki_sync.timer
sudo systemctl start anki_sync.timer
```

Pour vérifier si ton timer est correctement configuré et actif :

`sudo systemctl status anki_sync.timer`

## Ajouter des questions

La documentation Anki est plutôt claire concernant l’[ajout de question au format texte](https://docs.ankiweb.net/importing/text-files.html)

En résumé : il y a trois colonnes séparées par des `;` :
- Question
- Réponse
- Tags

> le texte est du html :

Exemple d’une question simple  (= une carte): `Que produisent les abeilles ?; du miel; nature, abeilles`

Ou une multi-ligne :
```
C’est quoi un logiciel open-source ? ;  un code conçu pour être accessible
au public : n'importe qui peut
voir,
modifier et distribuer le code à sa convenance ; code
```


## SyncServer :

Anki propose une version gratuite **AnkiWeb**
Toutefois pour les amoureux de l’autohébergement il existe aussi une version docker **Anki Sync Server**
Qui malheureusement ne possède pas d’interface graphique.