\# TP2 — Orchestrer une application multi-conteneurs avec Docker Compose



\*\*Module :\*\* DevOps 2 (ECUE733)



\## Introduction



L'objectif de ce TP est de conteneuriser et d'orchestrer une application web composée de quatre services : Next.js pour le frontend, Strapi pour le backend et l'API REST, PostgreSQL pour le stockage des données et pgAdmin pour l'administration de la base de données.



Docker Compose permet de définir les services, les réseaux, les volumes et les dépendances dans un seul fichier de configuration.



\## Réponses aux questions



\### Q1. Quelles variables d'environnement Strapi attend-il pour PostgreSQL ? Pourquoi `DATABASE\_HOST=postgres` et non `localhost` ?



Strapi utilise notamment les variables `DATABASE\_CLIENT`, `DATABASE\_HOST`, `DATABASE\_PORT`, `DATABASE\_NAME`, `DATABASE\_USERNAME` et `DATABASE\_PASSWORD` pour se connecter à PostgreSQL.



La valeur `DATABASE\_HOST=postgres` correspond au nom du service PostgreSQL dans Docker Compose. Dans un conteneur, `localhost` désigne le conteneur lui-même et non le conteneur PostgreSQL.



\### Q2. Quel est le rôle de `.env.example` ? Pourquoi ne pas committer `.env` ?



Le fichier `.env.example` fournit un modèle des variables nécessaires au fonctionnement de l'application. Il peut être versionné dans Git pour aider à configurer le projet.



Le fichier `.env` contient les valeurs réellement utilisées, notamment des mots de passe et des clés secrètes. Il ne doit pas être commité pour éviter d'exposer ces informations.



\### Q3. Quelle est la différence entre `docker compose down` et `docker compose down -v` ?



`docker compose down` arrête et supprime les conteneurs ainsi que les réseaux créés par Compose, mais conserve les volumes nommés.



`docker compose down -v` supprime également les volumes nommés du projet. Les données persistantes stockées dans ces volumes peuvent donc être perdues.



\### Q4. Pourquoi utilise-t-on `$$` dans le healthcheck PostgreSQL ?



Dans le fichier Compose, `$$` empêche Compose d'interpréter immédiatement la variable. Le signe `$` est transmis au shell du conteneur, qui peut alors lire les variables `POSTGRES\_USER` et `POSTGRES\_DB`.



\### Q5. Dans `servers.json`, pourquoi `Host=postgres` ? Que signifie `:ro` et `${PGADMIN\_PORT:-5050}` ?



Le nom `postgres` permet à pgAdmin de joindre le service PostgreSQL sur le réseau Docker.



Le suffixe `:ro` signifie que le fichier monté est accessible en lecture seule depuis le conteneur.



L'expression `${PGADMIN\_PORT:-5050}` utilise la valeur de `PGADMIN\_PORT` si elle est définie ; sinon, elle utilise `5050` par défaut.



\### Q6. Pourquoi copier `package.json` et `package-lock.json` avant le reste du code dans le Dockerfile Strapi ?



Cette organisation permet à Docker de réutiliser le cache de l'installation des dépendances lorsque les fichiers de dépendances n'ont pas changé. Cela évite de relancer `npm ci` à chaque modification du code source.



\### Q7. Pourquoi utiliser `USER node` et `--chown=node:node` ?



`USER node` exécute l'application avec un utilisateur non privilégié plutôt qu'avec `root`, ce qui réduit les risques de sécurité.



`--chown=node:node` attribue les fichiers copiés à l'utilisateur et au groupe `node`, afin que l'application puisse y accéder avec les permissions appropriées.



\### Q8. Pourquoi utiliser `depends\_on` avec `condition: service\_healthy` pour Strapi ?



Cette configuration fait attendre Strapi jusqu'à ce que le healthcheck de PostgreSQL indique que la base est prête. Cela réduit le risque que Strapi tente de se connecter avant que PostgreSQL puisse accepter des connexions.



\### Q9. Combien d'articles sont présents dans la base de données ?



Trois articles ont été créés et publiés dans Strapi. L'API `/api/articles` renvoie trois articles. Le nombre de lignes de la table PostgreSQL doit être vérifié dans pgAdmin, car les lignes de la table ne correspondent pas nécessairement une à une aux articles publiés dans Strapi.



\### Q10. Pourquoi `STRAPI\_URL=http://localhost:1337` ne fonctionnerait-il pas dans le conteneur frontend ?



Dans le conteneur frontend, `localhost` désigne le conteneur frontend lui-même. Pour joindre le backend, il faut utiliser le nom du service Docker Compose : `http://strapi:1337`.



\### Q11. Combien d'étapes contient le Dockerfile frontend ? Que contient l'image finale ? Que se passe-t-il sans `output: "standalone"` ?



Le Dockerfile frontend contient trois étapes : `deps` pour installer les dépendances, `builder` pour construire l'application et `runner` pour exécuter l'application en production.



L'image finale contient les fichiers nécessaires à l'exécution de Next.js en mode standalone, notamment le serveur généré, les dépendances nécessaires et les fichiers statiques copiés dans l'image.



Sans `output: "standalone"`, Next.js ne génère pas le dossier `.next/standalone` attendu par le Dockerfile. La commande de copie correspondante échoue donc pendant la construction. Cette option doit être présente pour utiliser cette méthode de construction.



Taille de l'image finale : \*\*à compléter avec le résultat de `docker images tp2/frontend`.\*\*



\### Q12. Pourquoi Strapi est-il connecté aux deux réseaux ? Pourquoi ne pas publier le port de PostgreSQL ?



Strapi doit communiquer avec le frontend pour fournir l'API et avec PostgreSQL pour lire et enregistrer les données. Il est donc connecté aux réseaux `frontend` et `backend`.



Le port PostgreSQL n'est pas publié sur l'hôte, ce qui évite de rendre directement la base accessible via un port de la machine hôte. Les services autorisés peuvent communiquer avec elle sur le réseau Docker `backend`.



\### Q13. Citez deux autres mesures pour renforcer la sécurité de la stack en production.



Deux mesures possibles sont :

1\. Utiliser des secrets gérés par un mécanisme adapté plutôt que de stocker les mots de passe en clair dans un fichier `.env`.

2\. Restreindre les accès aux interfaces d'administration et n'exposer que les ports strictement nécessaires, avec des règles réseau et une authentification adaptées.



\### Q14. Que fait `restart: unless-stopped` ? Pourquoi Strapi ne redémarre-t-il pas après `docker compose stop strapi` ?



La politique `unless-stopped` permet à Docker de redémarrer automatiquement un conteneur arrêté de manière inattendue et de le relancer après le redémarrage du démon Docker, sauf si le conteneur a été explicitement arrêté.



Après `docker compose stop strapi`, le service a été arrêté volontairement. Docker respecte cet arrêt et ne relance pas automatiquement le conteneur tant qu'il reste dans cet état d'arrêt explicite.



\## Conclusion



Ce TP m'a permis de construire et d'orchestrer une application web multi-conteneurs avec Docker Compose. J'ai configuré PostgreSQL et pgAdmin, construit les images de Strapi et de Next.js, configuré les variables d'environnement, les healthchecks, les volumes et les réseaux Docker. Les tests ont confirmé que le frontend pouvait récupérer les articles depuis Strapi et que le réseau isolait le frontend de l'accès direct à PostgreSQL.



Les tests de persistance, de sauvegarde et de reconstruction ciblée complètent la validation du cycle de vie de l'application.

