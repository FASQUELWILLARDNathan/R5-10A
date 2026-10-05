1 - C'est l'outil qui a envoyé les documents qui verifie si il n'y a pas d'identifiant il en genere un automatiquement et l'envoie en complément

2 - Si c'est dans la meme table on aurait juste à faire un where et si c'est dans une autre comme une relation many to many on ferait un join.

3 - On ajoute mongosh car c'est le nom de commande dans le terminal qui permet de faire des commandes et mongoimport qui permet simplement d'importer les données. Le compte pixelhub se situe dans la base admin.

4 - C'est une bonne ET une mauvaise nouvelle dans le sens où chaque jeux n'ont pas besoin des mêmes spécifications certains auront besoin de choses en plus, mais si l'on veut par exemple chercher dans plusieurs jeux des informations, et bien si ils n'ont pas les mêmes noms il pourrait y avoir des problemes du style une personne met "nom" et un autre "name" et bien ca poserait un probleme.

5 - Il cherche un document nommé Jeux mais il n'existe pas, il n'y a pas d'erreurs dans la commande en lui même juste dans le nom du document donc il ne retourne pas d'erreurs.

6 - ```MongoInvalidArgumentError: Update document requires atomic operators```
Il refuse car on n'a pas mis un mot clé devant note donc il ne sait pas ce qu'il doit faire. Si on voulait tout remplacer on aurait mis replaceOne.

7 - ALTER TABLE jeux ADD COLUMN 'nbVotes: int';
Les autres lignes auraient donc tous une colonne vide.

8 - C'est une interface donc on l'implémente pas dans Mongo.

9 - ```Element 'joueursParEquipe' does not match any field or property of class PixelHub.Api.Models.Game```
Il essaye de recuperer le champ joueursParEquipe du document Game mais il n'existe pas.

10 - Il l'affiche pas ducoup car il n'est pas dans la classe Game

11 - 

```text
+-------------------------+------------------------------+------------------------------+
| Approche                | Avantage                     | Inconvenient                 |
+-------------------------+------------------------------+------------------------------+
| BsonIgnoreExtraElements | Simple et robuste.           | Les champs inconnus sont     |
|                         |                              | perdus.                      |
+-------------------------+------------------------------+------------------------------+
| BsonExtraElements       | Conserve les champs          | Les champs ne sont pas       |
|                         | variables.                   | fortement types.             |
+-------------------------+------------------------------+------------------------------+
| Hierarchie de classes   | Donnees fortement typees et  | Plus rigide et complexe.     |
| FpsGame : Game          | explicites.                  | Il faut connaitre les types  |
|                         |                              | a l'avance.                  |
+-------------------------+------------------------------+------------------------------+
```

12 - 
```SQL
SELECT genre, AVG(note) AS note_moyenne, COUNT(*) AS nombre
FROM jeux
GROUP BY genre
ORDER BY note_moyenne DESC;
```
Cette requete peut etre fais en PostgreSQL sans soucis, le vrai apport de MongoDB est que ce soit flexible et que on peut apporter des informations a des documents que d'autres n'auraient pas besoin.