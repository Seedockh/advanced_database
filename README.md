# BASES DE DONNÉES AVANCÉES
## AUTHOR : Pierre Hérissé

# TP1

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





## IV - Créer des vues (Views)

Toujours sur le modèle WorkBench, créez des Vues :

- Vue film_list : elle doit présenter toutes les informations d’un film ainsi que sa
catégorie ainsi que la liste concaténée des acteurs

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

Procédure film_in_stock : qui permet de déterminer si des exemplaires d’un film
donné sont disponibles dans un magasin donné :

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


Une fois synchronisé le modèle, tentez d’ajouter un enregistrement factice.

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


Adaptez le fichier CSV et importez-le en ligne de commandes dans votre base Velib.
- Comment avez-vous formaté le fichier ?

> Chaque ligne du fichier CSV doit être de ce format :
```csv
4	128989986	Place Georges Guillaumin	29	POINT(48.881949 2.352339)
```

> Pour ce faire, les colonnes latitude et longitude étant dans un mauvais format, il a fallu effectuer des opérations sur la colonne des coordonnées, sous la forme `latitude,longitude`, ce qui donne :
```csv
=CONCAT("POINT(";LEFT(F2;FIND(",";F2)-1);" ";RIGHT(F2;LEN(F2)-FIND(",";F2));")")
```

![https://image.noelshack.com/fichiers/2019/41/3/1570572780-screenshot-from-2019-10-09-00-12-26.png](https://image.noelshack.com/fichiers/2019/41/3/1570572780-screenshot-from-2019-10-09-00-12-26.png)

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

![https://image.noelshack.com/fichiers/2019/41/3/1570572637-screenshot-from-2019-10-09-00-09-35.png](https://image.noelshack.com/fichiers/2019/41/3/1570572637-screenshot-from-2019-10-09-00-09-35.png)

- Trouvez les 5 stations de Velib les plus proches de l’Arc de Triomphe. Quelle requête avez-vous saisi ?

```SQL
SELECT name AS Station, capacity AS Capacite, ST_Distance_Sphere(
      ST_GeomFromText('POINT(48.8738 2.295)'), 
      coordinates
   ) AS distance_in_meter
FROM stations
ORDER BY distance_in_meter
LIMIT 5;
```

![https://image.noelshack.com/fichiers/2019/41/3/1570572659-screenshot-from-2019-10-09-00-09-51.png](https://image.noelshack.com/fichiers/2019/41/3/1570572659-screenshot-from-2019-10-09-00-09-51.png)






# TP2

## VII - Transactions 

### Sur la base sakila :
Désactiver l’autocommit.

```sql
SET autocommit = 0;
```

Insérer 2 films en une requête.

```sql
INSERT INTO film (title, description, language_id) VALUES
	('Film commit 1', 'Description 1', 5),
	('Film commit 2', 'Description 2', 5);
```

Ouvrez un autre client mysql et vérifiez si les films ont été ajoutés. Oui ou non ? Et 
dans la table film_text (où vous avez fait un trigger) ?

> Non, les films n'ont pas été ajoutés, ni dans `film`, ni dans `film_text`.

![https://image.noelshack.com/fichiers/2019/41/3/1570601748-screenshot-from-2019-10-09-08-15-28.png](https://image.noelshack.com/fichiers/2019/41/3/1570601748-screenshot-from-2019-10-09-08-15-28.png)


Faites un commit.

```sql
COMMIT;
```

Vérifiez si les films ont été ajoutés. Oui ou non ? Et dans la table film_text ?

![https://image.noelshack.com/fichiers/2019/41/3/1570601937-screenshot-from-2019-10-09-08-18-42.png](https://image.noelshack.com/fichiers/2019/41/3/1570601937-screenshot-from-2019-10-09-08-18-42.png)

![https://image.noelshack.com/fichiers/2019/41/3/1570601972-screenshot-from-2019-10-09-08-19-19.png](https://image.noelshack.com/fichiers/2019/41/3/1570601972-screenshot-from-2019-10-09-08-19-19.png)

Insérer un nouveau film.

```sql
INSERT INTO film (title, description, language_id) VALUES
	('Film commit 3', 'Description 3', 5);
```

Fermez votre client mysql.
Relancez un client. L’enregistrement a-t-il été commité ? Pensez à re-désactiver
l’autocommit !

> Non, l'enregistrement n'a pas été commité.

![https://image.noelshack.com/fichiers/2019/41/3/1570602115-screenshot-from-2019-10-09-08-21-18.png](https://image.noelshack.com/fichiers/2019/41/3/1570602115-screenshot-from-2019-10-09-08-21-18.png)

```sql
SET autocommit=0;
```

Insérez un film. Faites un ROLLBACK. Insérez un film. Faites un COMMIT. Combien de 
films ont été ajoutés ?

```sql
USE sakila;

INSERT INTO film (title, description, language_id) VALUES
	('Film commit 4', 'Description 4', 5);
ROLLBACK;

INSERT INTO film (title, description, language_id) VALUES
	('Film commit 5', 'Description 5', 5);
COMMIT;
```

> Seul le 2eme film a été ajouté.

![https://image.noelshack.com/fichiers/2019/41/3/1570602685-screenshot-from-2019-10-09-08-30-40.png](https://image.noelshack.com/fichiers/2019/41/3/1570602685-screenshot-from-2019-10-09-08-30-40.png)


Réactiver l’autocommit.
Commencez un bloc d’instructions par START TRANSACTION, insérez 2 films et faites 
un ROLLBACK. Les films ont-ils été enregistrés ?

```sql
SET autocommit=1;

START TRANSACTION;
INSERT INTO film (title, description, language_id) VALUES
	('Film commit 6', 'Description 6', 5);
INSERT INTO film (title, description, language_id) VALUES
	('Film commit 7', 'Description 7', 5);	
	
ROLLBACK;
```

> Non, les films n'ont pas étés enregistrés.

![https://image.noelshack.com/fichiers/2019/41/3/1570602925-screenshot-from-2019-10-09-08-34-49.png](https://image.noelshack.com/fichiers/2019/41/3/1570602925-screenshot-from-2019-10-09-08-34-49.png)

Commencez un bloc d’instructions par START TRANSACTION, insérez 2 films et faites 
un COMMIT. Les films ont-ils été enregistrés ?

```sql
START TRANSACTION;
INSERT INTO film (title, description, language_id) VALUES
	('Film commit 6', 'Description 6', 5);
INSERT INTO film (title, description, language_id) VALUES
	('Film commit 7', 'Description 7', 5);	
	
COMMIT;
```

> Oui, les films ont bien été enregistrés.

![https://image.noelshack.com/fichiers/2019/41/3/1570603017-screenshot-from-2019-10-09-08-36-39.png](https://image.noelshack.com/fichiers/2019/41/3/1570603017-screenshot-from-2019-10-09-08-36-39.png)


Commencez un bloc d’instructions par START TRANSACTION, insérez 1 film et créez 
un jalon. Insérez un nouveau film et faites un ROLLBACK au précédent jalon.
Commitez. Certains films ont-ils été enregistrés ?

```sql
START TRANSACTION;
INSERT INTO film (title, description, language_id) VALUES
	('Film commit 8', 'Description 8', 5);

SAVEPOINT FilmEightInsert;

INSERT INTO film (title, description, language_id) VALUES
	('Film commit 9', 'Description 9', 5);

ROLLBACK TO FilmEightInsert;

COMMIT;
```

> Oui, seul le film 8 a été enregistré.

![https://image.noelshack.com/fichiers/2019/41/3/1570603210-screenshot-from-2019-10-09-08-39-38.png](https://image.noelshack.com/fichiers/2019/41/3/1570603210-screenshot-from-2019-10-09-08-39-38.png)





## VIII - Cluster MySQL

Par groupe de 3 machines, montez un cluster MySQL.

L’objectif cible est d’obtenir la configuration suivante :

![https://image.noelshack.com/fichiers/2019/41/3/1570573603-cluster.png](https://image.noelshack.com/fichiers/2019/41/3/1570573603-cluster.png)

Il nous faut :
- 1 machine qui va servir de 
	- Serveur Mysql (mysqld et mysql)
	- Cluster manager (ndb_mgmd)
- 2 machines qui vont servir de
	- Nœuds de données (ndbd)

Vous pouvez trouver des informations / tutoriels sur :
- https://dev.mysql.com/doc/refman/8.0/en/mysql-innodb-cluster.userguide.html
- https://www.digitalocean.com/community/tutorials/how-to-create-a-multi.node-mysql-cluster-on-ubuntu-18-04


# TP3 & TP4

Go to this PDF :
[https://github.com/Seedockh/advanced_database/blob/master/tp3/Pierre%20H%C3%A9riss%C3%A9%20-%20TP3.pdf](https://github.com/Seedockh/advanced_database/blob/master/tp3/Pierre%20H%C3%A9riss%C3%A9%20-%20TP3.pdf)
