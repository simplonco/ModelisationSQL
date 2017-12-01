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

## Afficher toutes les cards du user 1

```sql
select c.id as 'cardId', c.name as 'cardName' from users_cards_lists as ucl
JOIN cards as c ON ucl.card_id = c.id
where ucl.user_id = 1
```

## Afficher tous les users qui ont des cards en lists 3

solution 1

```sql
select DISTINCT users.id, CONCAT(users.firstname, ' ',users.lastname) as userName FROM users_cards_lists
JOIN users ON user_id = users.id
WHERE list_id = 3
```

## Pour plus de détail, ajouter, pour chaque utilisateur, le nom des cards qu'ils ont en list 3

```sql
select DISTINCT users.id, 
CONCAT(users.firstname, ' ',users.lastname) as userName,
GROUP_CONCAT( 
CONCAT('{"id":"', cards.id, '","name":"',cards.name,'"}')
 ) as cards 
FROM users_cards_lists
JOIN users ON user_id = users.id
JOIN cards ON card_id = cards.id
WHERE list_id = 3
GROUP BY user_id
```

```sql
select DISTINCT users.id, 
CONCAT(users.firstname, ' ',users.lastname) as userName,
JSON_ARRAYAGG(JSON_OBJECT('id',cards.id, 'name', cards.name)) as cards 
FROM users_cards_lists
JOIN users ON user_id = users.id
JOIN cards ON card_id = cards.id
WHERE list_id = 3
GROUP BY user_id
```

## Afficher les cards avec les lists associés

```sql
SELECT c.id, c.name, cat.name
FROM cards as c
JOIN users_cards_lists as ucl ON ucl.card_id = c.id
JOIN lists as cat ON cat.id = ucl.list_id
ORDER BY c.id
```

## Afficher les listes avec leurs cards associées et avec pour chaque cards, la liste des utilisateurs associés

```sql
SELECT cat.name, JSON_ARRAYAGG( JSON_OBJECT('card', rucl.cname, 'users', rucl.users )) as cards
FROM (
  SELECT c.id as cid, c.name as cname, ucl.list_id as lid, JSON_ARRAYAGG( u.firstname ) as users
  FROM users_cards_lists as ucl
  JOIN users as u ON u.id = ucl.user_id
  JOIN cards as c ON c.id = ucl.card_id
  GROUP BY ucl.list_id, ucl.card_id
) as rucl
JOIN lists as cat ON cat.id = rucl.cid
GROUP BY cat.id
```


















