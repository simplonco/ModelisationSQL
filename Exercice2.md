# Exercice 2

## Brief

Après une première itération, nous apercevons que le premier brief ne satisfait pas exactement les besoins de nos utilisateurs.

Alors nous avons remis l'ouvrage sur le métier et nous avons pondu de nouvelles user stories aditionnelles et correctives.

Nous avons défini 2 rôles distinct :

* *F = formateur*
* *A = apprenant*

# User stories

## Gestion des roles, des autorisations et des invitations

Les actions définies dans les **user stories de 1 à 6** (creation / supression / modification des listes et des cartes) ne seront réalisable que par le formateur.

L'action definie dans la **user story 7** sera remplacer par un système d'invitation.

L'actions définie dans la **user story 8** (supression d'un utilisateur) sera aussi réservé au formateur.

### *La gestion des roles, des autorisations et des invitations n'est pas attendu pour le sprint à venir*

## Gestion des déplacements de carte personnalisé par apprenant

Afin de pouvoir gérer plus finement l'avancement de la promo et de permettre une auto-évaluation par les apprenants, nous souhaiterions qu'un apprenant puisse déplacer ses cartes individuellement.

Néanmoins nous aimerions toujours avoir une vue condensée de l'état de l'évolution de la promo. Ainsi, dans la vue générale du board, une carte pourra être dupliquée sur les listes pour lesquels on trouve des apprenants associés à cette carte.

Ce qui nous ammène à modifier les user stories 10, 11, 12 et à ajouter une story 14

10. en tant qu'Apprenant, je veux déplacer **mes** cartes d'une liste à une autre afin de montrer l'évolution de **mon** travail.

11. en tant qu'U, je veux **m'**ajouter à une carte afin de lui attribuer l'exercice ou la compétence

12. en tant qu'U, je veux **me** supprimer d'une carte afin de corriger une erreur d'attribution

13. (aucune modification)

14. en tant qu'U, je veux voir mon board personnel afin d'avoir un vision condensée de l'évolution de mon travail

# Nouveau use case

![](UseCase2.svg)

# Modelisation des données

De par l'ajout d'une gestion personnelle des déplacements de carte par les apprenants, nous devons modifier le modèle de données.

**Pouvez-vous nous fournir un nouveau diagramme UML représentant le nouveau modèle de données ?**

A vous de jouer !!!

# exercices sql SELECT

## Afficher toutes les cards du user avec l'id 1

```sql
SELECT card_id, cards.name FROM users_cards 
JOIN cards ON users_cards.card_id = cards.id
WHERE user_id=1
```

## Afficher tous les users qui ont des cards en lists avec l'id 3

```sql
SELECT DISTINCT user_id, CONCAT(users.firstname, ' ', users.lastname) as name FROM users_cards 
JOIN users ON users_cards.user_id = users.id
WHERE list_id=3
```

## Pour plus de détail, ajouter, pour chaque utilisateur, le nom des cards qu'ils ont en list 3

```sql
SELECT DISTINCT user_id, CONCAT(users.firstname, ' ', users.lastname) as name FROM users_cards 
JOIN users ON users_cards.user_id = users.id
WHERE list_id=3
```

## Afficher toutes les lists avec pour chacune le nombre de card associés

```sql
SELECT user_id, 
CONCAT(users.firstname, ' ', users.lastname) as name,
JSON_AGG(cards.name) as cards_name
FROM users_cards 
JOIN users ON users_cards.user_id = users.id
JOIN cards ON users_cards.card_id = cards.id
WHERE list_id=3
GROUP BY user_id, users.firstname, users.lastname
```

## Afficher toutes les lists avec pour chacune le nombre de card associés

```sql
SELECT list_id, COUNT(list_id)
FROM users_cards 
GROUP BY list_id
```

## Afficher toutes les lists avec pour chacune le nombre de user associés

```sql
SELECT list_id, COUNT(list_id)
FROM users_cards 
GROUP BY list_id
```

## Afficher toutes les lists avec pour chacune les noms des cards associés

```sql
SELECT list_id, COUNT(DISTINCT user_id) as user_count
FROM users_cards 
GROUP BY list_id
```

## Afficher tous les noms de users qui ont au moins une carte dans la liste avec l'id 3



## Afficher les listes avec leurs cards associées et avec pour chaque cards, la liste des utilisateurs associés

### on commence par faire la sous-requeque (sub-select)

* Donne-moi toutes les cards avec pour chaque cards, la liste des utilisateurs associés

```sql
SELECT card_id, list_id, 
JSON_AGG( users.firstname ) as user_name
FROM users_cards
JOIN users ON users.id = user_id
GROUP BY card_id, list_id
```

### Après on fait la requete parent

* Donne-moi toutes les lists avec pour chaque lists, la liste des cards associés

```sql

```

### Requete final

```sql
SELECT lists.id, JSON_AGG(

JSON_BUILD_OBJECT('name',
r.card_name,
'users', r.user_name
))

FROM (
 
SELECT card_id, cards.name as card_name, list_id, 
JSON_AGG( users.firstname ) as user_name
FROM users_cards
JOIN users ON users.id = user_id
JOIN cards ON cards.id = card_id
GROUP BY card_id, card_name, list_id

) as r
JOIN lists ON lists.id = r.list_id
GROUP BY lists.id
```

```sql
SELECT lists.id, JSON_AGG(

JSON_BUILD_OBJECT('name',
r.card_name,
'users', r.user_name
))

FROM lists
JOIN (
 
SELECT card_id, cards.name as card_name, list_id, 
JSON_AGG( users.firstname ) as user_name
FROM users_cards
JOIN users ON users.id = user_id
JOIN cards ON cards.id = card_id
GROUP BY card_id, card_name, list_id

) as r ON lists.id = r.list_id
GROUP BY lists.id
```

```sql
CREATE TEMPORARY TABLE resu as SELECT card_id, cards.name as card_name, list_id, 
JSON_AGG( users.firstname ) as user_name
FROM users_cards
JOIN users ON users.id = user_id
JOIN cards ON cards.id = card_id
GROUP BY card_id, card_name, list_id;

SELECT lists.id, 
JSON_AGG(
 JSON_BUILD_OBJECT('name',
  r.card_name,
  'users', r.user_name
 )
)
FROM resu as r
JOIN lists ON lists.id = r.list_id
GROUP BY lists.id
```

```sql
CREATE TEMPORARY TABLE cards_distrib as 
SELECT 
cards.name as card_name, 
JSON_AGG( users.firstname ) as user_list,
users_cards.list_id as lid
FROM users_cards
JOIN users ON users.id = user_id
JOIN cards ON cards.id = card_id
GROUP BY card_id, card_name, list_id;

SELECT lists.id, 
lists.name,
JSON_AGG(
 JSON_BUILD_OBJECT('name',
  cards_distrib.card_name,
  'users', cards_distrib.user_list
 )
) as cards
FROM cards_distrib
JOIN lists ON lists.id = cards_distrib.lid
GROUP BY lists.id
```

## Donne-moi toutes les cards du user qui a l'id 1

```sql
SELECT card_id, cards.name FROM users_cards
JOIN cards ON users_cards.card_id = cards.id
WHERE user_id  = 1
```

## Donne tous les users avec leur cards associées

  ```sql
  SELECT user_id, users.firstname, JSON_AGG(cards.name) FROM users_cards
JOIN users ON users_cards.user_id = users.id
JOIN cards ON users_cards.card_id = cards.id
GROUP BY user_id, users.firstname
  ```
  
 ## Donne-moi tous les utilisateurs qui ont au moins UNE CARD SUR LA liste avec l'id 2. 
 
 ```sql
SELECT DISTINCT CONCAT(users.lastname, ' ', users.firstname) as user_name from users_cards
JOIN users ON users_cards.user_id = users.id
WHERE list_id = 2
```
  
  ## A partir de l'exercice précédent, ajouter une colonne avec les cards associés.

```sql
SELECT CONCAT(users.lastname, ' ', users.firstname) as user_name,
... as cards
FROM users_cards
JOIN users ON users_cards.user_id = users.id
JOIN ...
WHERE list_id = 2
GROUP BY ...
```

## Afficher les users avec leurs cards associées et avec pour chaque cards, la liste des listes associés

## Afficher les users avec leurs lists associées et avec pour chaque list, la liste des cards associés

## Afficher les cards avec leurs lists associées et avec pour chaque list, la liste des users associés















