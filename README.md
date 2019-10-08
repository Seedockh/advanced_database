# BASES DE DONNÉES AVANCÉES
## AUTHOR : Pierre Hérissé

## I - Installer MySQL 8 :

	-> OK

## II - Installer la structure de base

```console
mysql -u pierre -p sakila < ./sakila-db/sakila-schema.sql
```

```console
mysql -u pierre -p sakila < ./sakila-db/sakila-data.sql
```

## III - Créer des déclencheurs (Triggers)

- Sur la table film, ajouter 3 triggers :

➔ après l'insertion d'un film : ajouter un enregistrement dans la table film_text

```SQL
CREATE TRIGGER add_film_text_on_new_film 
AFTER INSERT ON film 
FOR EACH ROW
	INSERT INTO film_text (film_id, title, description)
	VALUES (NEW.film_id, NEW.title, NEW.description);
	
INSERT INTO film (film_id, title, description, release_year, language_id)
VALUES (NULL, 'FIRST FILM', 'Une description', '2019', 5);
```

![https://i.ibb.co/jD6BLx0/Screenshot-from-2019-10-08-08-32-26.png](https://i.ibb.co/jD6BLx0/Screenshot-from-2019-10-08-08-32-26.png)


➔ après la mise à jour d'un film : si le nouveau titre est différent de l'ancien OU
si la nouvelle description est différente de l'ancienne alors mettre à jour
l'enregistrement correspondant dans la table film_text

```SQL
CREATE TRIGGER update_film_text_on_update_film
   AFTER UPDATE ON film FOR EACH ROW
   BEGIN
      IF (NEW.title!=OLD.title) OR (NEW.description!=OLD.description)
      THEN
         UPDATE film_text
         SET title=NEW.title,
             description=NEW.description,
             film_id=NEW.film_id
	 WHERE film_id=OLD.film_id;
      END IF;
  END;
  /* This solution didn't work well because of the multiple delimiters */
  
  /* Here is a working alternative solution : */
  DELIMITER $$
  CREATE TRIGGER update_film_text
  AFTER UPDATE
  ON film FOR EACH ROW
    BEGIN
    IF OLD.title<>NEW.title THEN
    UPDATE film_text SET title=NEW.title WHERE film_id=NEW.film_id;
    END IF;
		
    IF OLD.description<>NEW.description THEN
      UPDATE film_text SET description=NEW.description WHERE film_id=NEW.film_id;
    END IF;
    END$$
  DELIMITER
 
 
  UPDATE film SET title = 'UPDATED FIRST FILM' WHERE film_id = 5002;
```

![https://i.ibb.co/7bpwLVT/Screenshot-from-2019-10-08-10-48-55.png](https://i.ibb.co/7bpwLVT/Screenshot-from-2019-10-08-10-48-55.png)


➔ après la suppression d'un film : supprimer l'enregistrement dans la table
film_text

```SQL
DELIMITER $$
CREATE TRIGGER delete_film_text_on_delete_film 
AFTER DELETE ON film
FOR EACH ROW
BEGIN
	DELETE FROM film_text
	WHERE (film_id = OLD.film_id);
END; $$
DELETE FROM film WHERE (film_id=5002);
```

!(https://i.ibb.co/HTTbntV/Screenshot-from-2019-10-08-12-13-27.png)[https://i.ibb.co/HTTbntV/Screenshot-from-2019-10-08-12-13-27.png]

Synchroniser votre schéma et tester que les triggers fonctionnent. Merci de me
mettre des copies d’écran


## IV - Créer des vues (Views)

Toujours sur le modèle WorkBench, créez des Vues :

- Vue film_list : elle doit présenter toutes les informations d’un film ainsi que sa
catégorie ainsi que la liste concaténée des acteurs

![https://i.ibb.co/HF5qSB2/Screenshot-from-2019-10-07-15-28-54.png](https://i.ibb.co/HF5qSB2/Screenshot-from-2019-10-07-15-28-54.png)

- Vue sales_by_store : créer une vue qui présente pour chaque magasin :
	- Ses informations Ville + Pays
	- Les informations du manager Prénom + nom
	- Le total des ventes de ce magasin

![https://i.ibb.co/z2Z6XbZ/Screenshot-from-2019-10-07-15-29-05.png](https://i.ibb.co/z2Z6XbZ/Screenshot-from-2019-10-07-15-29-05.png)


## V - Créer des procédures stockées (Routines)

Toujours sur le modèle WorkBench, créons une Procédure Stockée
Procédure film_in_stock : qui permet de déterminer si des exemplaires d’un film
donné sont disponibles dans un magasin donné.
Paramètres :
- p_film_id : L’ID du film à vérifier
- p_store_id : L’ID du magasin pour lequel on souhaite vérifier
- p_film_count : Un paramètre OUT de sortie qui retourne le nombre d’exemplaires 
du film en stock dans le magasin

Exemple :

![https://i.ibb.co/MN0TN7w/Screenshot-from-2019-10-07-15-29-12.png](https://i.ibb.co/MN0TN7w/Screenshot-from-2019-10-07-15-29-12.png)


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

Voir la doc : https://dev.mysql.com/doc/refman/8.0/en/gis-wkt-functions.html#function_st-geomfromtext

Une fois que c’est OK, rendez-vous sur le site Open Data Paris (https://opendata.paris.fr/ ), trouvez le jeu de données des Velib et téléchargez-le au format CSV.

Adaptez le fichier CSV et importez-le en ligne de commandes dans votre base Velib.
- Comment avez-vous formaté le fichier ?
- Quelle commande avez-vous saisi pour importer ?
- Trouvez les 5 stations de Velib les plus proches de l’Arc de Triomphe. Quelle requête avez-vous saisi ?
