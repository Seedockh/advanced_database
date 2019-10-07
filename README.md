# BASES DE DONNÉES AVANCÉES

## AUTHOR : Pierre Hérissé
## ~~~~~~~~~~~~~~~~~~~~~~~~


## I - Installer MySQL 8 :
	-> OK

## ~~~

## II - Installer la structure de base
	- `mysql -u pierre -p sakila < ./sakila-db/sakila-schema.sql`
	- `mysql -u pierre -p sakila < ./sakila-db/sakila-data.sql`


## ~~~

## III - Créer des déclencheurs (Triggers)


- Sur la table film, ajouter 3 triggers :
➔ après l'insertion d'un film : ajouter un enregistrement dans la table film_text
➔ après la mise à jour d'un film : si le nouveau titre est différent de l'ancien OU
si la nouvelle description est différente de l'ancienne alors mettre à jour
l'enregistrement correspondant dans la table film_text
➔ après la suppression d'un film : supprimer l'enregistrement dans la table
film_text

Synchroniser votre schéma et tester que les triggers fonctionnent. Merci de me
mettre des copies d’écran


## ~~~

## IV - Créer des vues (Views)

Toujours sur le modèle WorkBench, créez des Vues :
Vue film_list : elle doit présenter toutes les informations d’un film ainsi que sa
catégorie ainsi que la liste concaténée des acteurs
Vue
•
•
•
sales_by_store : créer une vue qui présente pour chaque magasin :
Ses informations Ville + Pays
Les informations du manager Prénom + nom
Le total des ventes de ce magasin


## ~~~

## V - Créer des procédures stockées (Routines)

Toujours sur le modèle WorkBench, créons une Procédure Stockée
Procédure film_in_stock : qui permet de déterminer si des exemplaires d’un film
donné sont disponibles dans un magasin donné.
Paramètres :
L’ID du film à vérifier
L’ID du magasin pour lequel on souhaite vérifier
p_film_count : Un paramètre OUT de sortie qui retourne le nombre d’exemplaires
du film en stock dans le magasin
• p_film_id :
• p_store_id :
•
Exemple :



## ~~~

## VI - Données géographiques et MySQL

MySQL propose un ensemble de types de données pour stocker des informations
géographiques.
Le but est d’appréhender son type de base : le point.
Créez le modèle Workbench d’une BDD Vélib avec une tables ‘stations’. Chaque
station aura un id (bref un id), un numéro de station (un entier) obligatoire, un nom
(une chaine de caractères) obligatoire, une capacité d’accueil (un petit entier) et des
coordonnées (notre fameux point).
Une fois synchronisé le modèle, tentez d’ajouter un enregistrement factice. En vous
aidant de la documentation
https://dev.mysql.com/doc/refman/8.0/en/spatial-types.html
Comment faut-il saisir les coordonnées ?

Voir
la
doc :
https://dev.mysql.com/doc/refman/8.0/en/gis-wkt-
functions.html#function_st-geomfromtext

Une fois que c’est OK, rendez-vous sur le site Open Data Paris (
https://opendata.paris.fr/ ), trouvez le jeu de données des Velib et téléchargez-le au
format CSV.
Adaptez le fichier CSV et importez-le en ligne de commandes dans votre base Velib.
Comment avez-vous formaté le fichier ?
Quelle commande avez-vous saisi pour importer ?
Trouvez les 5 stations de Velib les plus proches de l’Arc de Triomphe.
Quelle requête avez-vous saisi ?
