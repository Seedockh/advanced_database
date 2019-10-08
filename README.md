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

![https://image.noelshack.com/fichiers/2019/41/2/1570536384-screenshot-from-2019-10-08-12-13-27.png](https://image.noelshack.com/fichiers/2019/41/2/1570536384-screenshot-from-2019-10-08-12-13-27.png)


Synchroniser votre schéma et tester que les triggers fonctionnent. Merci de me
mettre des copies d’écran


## IV - Créer des vues (Views)

Toujours sur le modèle WorkBench, créez des Vues :

- Vue film_list : elle doit présenter toutes les informations d’un film ainsi que sa
catégorie ainsi que la liste concaténée des acteurs

![https://i.ibb.co/HF5qSB2/Screenshot-from-2019-10-07-15-28-54.png](https://i.ibb.co/HF5qSB2/Screenshot-from-2019-10-07-15-28-54.png)

```SQL
CREATE VIEW show_film_details
	(FID,
	Titre,
	Description,
	Catégorie,
	Durée,
	Prix,
	Note,
	Acteurs) 
AS SELECT 
	film.film_id,
	title,
	description,
	category.name,
	replacement_cost,
	length,
	rating,
	GROUP_CONCAT(' ', actor.first_name, ' ', actor.last_name) AS actors
FROM film	
LEFT JOIN film_category ON film.film_id = film_category.film_id
LEFT JOIN category ON film_category.category_id = category.category_id
LEFT JOIN film_actor ON film.film_id = film_actor.film_id
LEFT JOIN actor ON film_actor.actor_id = actor.actor_id
GROUP BY film.film_id, category.name;
```

![https://image.noelshack.com/fichiers/2019/41/2/1570538691-screenshot-from-2019-10-08-14-44-36.png](https://image.noelshack.com/fichiers/2019/41/2/1570538691-screenshot-from-2019-10-08-14-44-36.png)


- Vue sales_by_store : créer une vue qui présente pour chaque magasin :
	- Ses informations Ville + Pays
	- Les informations du manager Prénom + nom
	- Le total des ventes de ce magasin
	
![https://image.noelshack.com/fichiers/2019/41/2/1570540196-view2.png](https://image.noelshack.com/fichiers/2019/41/2/1570540196-view2.png)	
	
```SQL
CREATE VIEW show_store_details
	(Store, Manager, TotalSales) 
AS SELECT 
	CONCAT(city.city, ', ', country.country),
	CONCAT(staff.first_name, ' ', staff.last_name),
	SUM(payment.amount)
FROM store	
LEFT JOIN address ON store.address_id = address.address_id
LEFT JOIN city ON address.city_id = city.city_id
LEFT JOIN country ON city.country_id = country.country_id
LEFT JOIN staff ON store.store_id = staff.store_id
LEFT JOIN payment ON staff.staff_id = payment.staff_id
GROUP BY 
	store.store_id, 
	staff.first_name, 
	staff.last_name;
```

![https://image.noelshack.com/fichiers/2019/41/2/1570540095-screenshot-from-2019-10-08-15-07-20.png](https://image.noelshack.com/fichiers/2019/41/2/1570540095-screenshot-from-2019-10-08-15-07-20.png)


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


```SQL
DELIMITER $$

CREATE PROCEDURE film_in_stock (
	IN p_film_id INT,
	IN p_store_id INT,
	OUT p_film_count INT
)
BEGIN
	SELECT inventory_id
	FROM inventory 
	WHERE 
		film_id = p_film_id 
		AND store_id = p_store_id;
	
	SELECT COUNT(inventory_id) INTO p_film_count
	FROM inventory 
	WHERE 
		film_id = p_film_id 
		AND store_id = p_store_id;
END; $$

DELIMITER ;
```


![https://image.noelshack.com/fichiers/2019/41/2/1570541797-screenshot-from-2019-10-08-15-36-16.png](https://image.noelshack.com/fichiers/2019/41/2/1570541797-screenshot-from-2019-10-08-15-36-16.png)


## VI - Données géographiques et MySQL

MySQL propose un ensemble de types de données pour stocker des informations
géographiques.

Le but est d’appréhender son type de base : le point.
Créez le modèle Workbench d’une BDD Vélib avec une tables ‘stations’. Chaque
station aura un id (bref un id), un numéro de station (un entier) obligatoire, un nom
(une chaine de caractères) obligatoire, une capacité d’accueil (un petit entier) et des
coordonnées (notre fameux point).

```SQL
CREATE TABLE stations
(
	station_id INT PRIMARY KEY NOT NULL,
	station_number INT NOT NULL,
	name VARCHAR(255) NOT NULL,
	capacity SMALLINT,
	coordinates POINT
);
```

![https://image.noelshack.com/fichiers/2019/41/2/1570542620-screenshot-from-2019-10-08-15-50-06.png](https://image.noelshack.com/fichiers/2019/41/2/1570542620-screenshot-from-2019-10-08-15-50-06.png)


Une fois synchronisé le modèle, tentez d’ajouter un enregistrement factice. En vous
aidant de la documentation
https://dev.mysql.com/doc/refman/8.0/en/spatial-types.html

Comment faut-il saisir les coordonnées ?

```SQL
INSERT INTO stations 
	(station_number, name, capacity, coordinates)
	VALUES (
		10, 
		'First Station', 
		10, 
		ST_GeomFromText('POINT(15 15)')
	);
```

![https://image.noelshack.com/fichiers/2019/41/2/1570543312-screenshot-from-2019-10-08-16-01-32.png](https://image.noelshack.com/fichiers/2019/41/2/1570543312-screenshot-from-2019-10-08-16-01-32.png)

Voir la doc : https://dev.mysql.com/doc/refman/8.0/en/gis-wkt-functions.html#function_st-geomfromtext

Une fois que c’est OK, rendez-vous sur le site Open Data Paris (https://opendata.paris.fr/ ), trouvez le jeu de données des Velib et téléchargez-le au format CSV.

Adaptez le fichier CSV et importez-le en ligne de commandes dans votre base Velib.
- Comment avez-vous formaté le fichier ?

Chaque ligne du fichier CSV doit être de ce format :
```csv
4	128989986	Place Georges Guillaumin	29	POINT(48881949 2352339)
```

- Quelle commande avez-vous saisi pour importer ?

```SQL
ALTER TABLE sakila.stations MODIFY COLUMN station_number BIGINT NOT NULL;

LOAD DATA INFILE '/var/lib/mysql-files/velib.csv'
REPLACE
INTO TABLE stations
COLUMNS TERMINATED BY ';' OPTIONALLY ENCLOSED BY '"'
IGNORE 1 ROWS
(station_id, station_number, name, capacity, @coordinates)
SET coordinates = ST_GeomFromText(@coordinates);
```

- Trouvez les 5 stations de Velib les plus proches de l’Arc de Triomphe. Quelle requête avez-vous saisi ?

```SQL
DELIMITER $$

CREATE PROCEDURE distance_from_adtriomphe (
	IN p_point_to_reach INT,
	OUT p_distance DECIMAL
)
BEGIN
	SET @arcDeTriomphe = ST_GeomFromText('POINT(48.8738 2.295)');
	SET @pointToReach = ST_GeomFromText(p_point_to_reach);

	SELECT ST_Distance_Sphere(@arcDeTriomphe, @pointToReach) 
	INTO p_distance;
END; $$

DELIMITER ;
```
