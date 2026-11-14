# Web Deployer

Kit autonome à copier dans la racine d'un projet Python.

## Installation

Copiez le dossier `web_deploy/` et le fichier `deploy.py` à la racine du projet.

Lancez :

```bash
python deploy.py
```

Une interface graphique permet de sélectionner le fichier Python qui démarre votre serveur.

## Installation automatique via git

**Windows (CMD) :**

```cmd
git clone https://github.com/LukeCOULON/web_deploy_kit.git "%TEMP%\web_deploy_kit" && xcopy "%TEMP%\web_deploy_kit\*" "." /E /I /Y && rmdir /S /Q "%TEMP%\web_deploy_kit" && python deploy.py
```

**Linux (Terminal) :**

```bash
git clone https://github.com/LukeCOULON/web_deploy_kit.git /tmp/web_deploy_kit && \
cp -rn /tmp/web_deploy_kit/. . && \
rm -rf /tmp/web_deploy_kit && \
python3 deploy.py
```

**Précision :** les fichiers du kit sont copiés directement dans le dossier courant, sans créer de sous-dossier. Aucun fichier ne doit déjà porter le même nom : le dossier doit donc être vide.

## Desinstallation

**Windows (CMD) :**

```cmd
rmdir /S /Q web_deploy 2>nul & del /Q deploy.py web_deploy_config.json 2>nul & rmdir /S /Q .venv 2>nul
```

**Linux (Terminal) :**

```bash
rm -rf web_deploy deploy.py web_deploy_config.json .venv
```

## Exemple

Si votre projet contient :

```text
mon_projet/
├── app/
│   └── server.py
├── web_deploy/
└── deploy.py
```

sélectionnez :

```text
app/server.py
```

Le serveur doit écouter sur l'adresse et le port configurés.

Le kit fournit ensuite un proxy HTTP qui écoute sur toutes les interfaces réseau :

```text
0.0.0.0:8080 -> 127.0.0.1:8000
```

Cela permet au routeur de rediriger le port public vers le PC. L'application peut rester liée à `127.0.0.1`.


## Environnement virtuel et dépendances

Lors du déploiement, l'option **Créer/utiliser un venv et installer automatiquement les dépendances** est activée par défaut.

Le kit :

1. crée `.venv/` à la racine du projet s'il n'existe pas ;
2. met à jour `pip` dans ce venv ;
3. installe les dépendances depuis `requirements.txt` s'il existe ;
4. sinon, si `pyproject.toml` ou `setup.py` est présent, installe le projet avec `pip install .` ;
5. lance ensuite le serveur avec le Python du venv, et non avec le Python système.

Pour désactiver ce comportement, décochez l'option dans l'interface.

## Important

Le kit ne configure pas automatiquement tous les routeurs/box Internet. Pour exposer le site sur Internet, configurez sur votre routeur une redirection NAT/port forwarding vers la machine qui exécute le kit.

Exemple :

```text
Port WAN 8080 -> IP locale du PC:8080
```

Une IP publique dynamique peut changer. Pour un vrai service public, un nom de domaine, HTTPS et une solution adaptée de DNS dynamique sont recommandés.

## Limites actuelles

- Le serveur Python est lancé comme un processus enfant.
- Le proxy est un proxy HTTP minimal intégré à Python.
- Le support HTTPS n'est pas inclus.
- La configuration automatique du routeur n'est pas universelle et dépend du modèle.
- Le port choisi doit être disponible et autorisé par le pare-feu.


## Tutoriel intégré

Au démarrage, Web Deployer affiche maintenant un guide avant le menu de configuration.

Le guide indique :

1. où chercher la fonction NAT / Port Forwarding du routeur ;
2. l'adresse IP locale détectée pour la machine ;
3. le protocole TCP à sélectionner ;
4. le port interne du serveur ;
5. le port externe à ouvrir ;
6. pourquoi il est recommandé de réserver l'adresse IP locale dans le DHCP.

Exemple de règle :

```text
IP interne    : 192.168.1.42
Protocole     : TCP
Port externe : 8080
Port interne : 8000
```

Une fois le guide lu, le bouton `J'AI COMPRIS — CONTINUER` ouvre l'interface principale.


## Adresse publique

L'interface affiche en haut l'adresse publique détectée et un lien cliquable de la forme :

```text
http://IP_PUBLIQUE:PORT_PUBLIC
```

Le port du lien est synchronisé en temps réel avec le champ **Port d'accès**. L'IP publique est rafraîchie périodiquement. Pour que le lien fonctionne depuis Internet, le routeur doit rediriger le port public vers le PC, et le pare-feu Windows/Linux doit autoriser ce port. Certains fournisseurs utilisent un CGNAT : dans ce cas, une simple redirection de port ne suffit pas.

## Diagnostics automatiques

Le journal traduit maintenant plusieurs erreurs courantes en messages lisibles avec une solution courte :

- **PORT DÉJÀ UTILISÉ** — fermer le programme qui utilise le port ou choisir un autre port.
- **PERMISSION REFUSÉE** — utiliser un port supérieur à 1024 et vérifier les droits.
- **SERVEUR INJOIGNABLE** — vérifier que l'application écoute sur le port serveur configuré.
- **FICHIER INTROUVABLE** — vérifier le chemin du projet ou du fichier concerné.
- **DÉPENDANCE PYTHON MANQUANTE** — installer la dépendance dans le venv ou l'ajouter à `requirements.txt`.
- **DÉPENDANCE PYTHON INCOMPATIBLE** — vérifier la version de Python et celle du paquet.
- **DÉLAI D'ATTENTE DÉPASSÉ** — vérifier le démarrage de l'application ou augmenter le délai.
- **ADRESSE RÉSEAU INVALIDE** — vérifier l'adresse d'écoute utilisée par le serveur ou le proxy.
- **CONFIGURATION INVALIDE** — vérifier ou recréer `web_deploy_config.json`.

## Nettoyage automatique des ports

Avant chaque déploiement, Web Deployer vérifie les ports sélectionnés. Si un processus écoute déjà sur le port serveur ou sur le port public du proxy, il est arrêté afin de libérer le port avant le lancement. Le journal indique le PID concerné et confirme la libération. Un arrêt forcé est utilisé uniquement si l'arrêt normal ne libère pas le port après quelques secondes.

## Base locale des ports

Web Deployer inclut une petite base SQLite locale : `web_deploy/data/ports.db`.

Elle contient notamment :

- des ports connus comme sensibles ou refusés par certains navigateurs (`browser_unsafe`) ;
- des ports souvent réservés à des services réseau et susceptibles d'être filtrés par certaines box/routeurs (`router_reserved`) ;
- quelques ports courants adaptés à un service Web (`recommended`).

La base sert à prévenir les erreurs avant le déploiement. Elle ne prétend pas connaître les règles particulières de chaque routeur ou fournisseur d'accès : le filtrage des ports publics dépend de l'équipement et du réseau.

Le port public est contrôlé avant le lancement. Un port classé `browser_unsafe` bloque le déploiement avec une proposition de ports alternatifs. L'interface permet aussi de vérifier manuellement le port public.
