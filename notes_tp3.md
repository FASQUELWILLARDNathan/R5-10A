Q1 - On utilise merge car create pour recréer une nouvelle fois une importation des données alors qu'elles sont déjà présentes.
Les trois contraintes du début permettent que si un nom de joueur, de jeu ou de guilde existe déjà, de ne pas pouvoir l'importer à nouveau. Ce qui équivaut à un merge finallement.

Q2 - La requête sans flèche retourne 2 résultats : Nova et Pixel. La requête avec
une flèche retourne 1 résultat : Pixel.

Dans le script d'import, les relations sont créées dans une seule direction avec
`MERGE (a)-[:AMI_DE]->(b)`. Pour Krayz, on obtient donc `Nova -> Krayz` et
`Krayz -> Pixel`. La relation avec Nova existe, mais dans le sens inverse de la
requête qui part de Krayz. La requête sans flèche est adaptée à une amitié
réciproque, tandis que Neo4j ne signale aucune erreur car une relation orientée
est parfaitement valide ; Neo4j ne sait pas que `AMI_DE` devrait être
symétrique.

Q3 - `count(DISTINCT ami)` compte le nombre d'amis différents qui possèdent
chaque jeu. Un même ami ne compte donc qu'une seule fois pour un jeu.
`count(*)` compte les chemins trouvés par la requête. Un même ami peut être
atteint par plusieurs chemins lorsque la profondeur passe de `1..2` à `1..4`,
et il peut alors compter plusieurs fois.

Avec `*1..2`, les deux versions donnent les mêmes nombres dans ce graphe :
les 7 jeux recommandés apparaissent chacun sur une ligne. Avec `*1..4`, les
7 lignes sont toujours présentes, mais les nombres changent avec `count(*)`
(par exemple Counter-Strike 2 et Vampire Survivors passent à 4), car les
chemins supplémentaires créent des doublons. En SQL, sans CTE récursive, il
aurait fallu ajouter 2 niveaux de parcours, donc 2 CTE supplémentaires pour
passer de 2 à 4 degrés. Avec une CTE récursive, une seule CTE aurait suffi
pour gérer une profondeur variable.

Q4 - Stardew Valley n'apparaît jamais car Nova possède déjà ce jeu :
`['Nova','Stardew Valley']` apparaît dans la section Possessions du script
d'import. La condition `NOT (moi)-[:POSSEDE]->(j)` exclut donc Stardew Valley
des recommandations. Pour la requete avec aucunes fleches, il y a 2 résultats qui sont Nova et Pixel.
Pour la requete avec une fleche il y a uniquement Pixel.
Dans le script d'import on suit une seule direction
```
MERGE (a)-[:AMI_DE]->(b)
```
La requete sans fleche est correcte pour les amis car normalement c'est réciproque, si Nova est ami avec Krayz, Krayz est aussi amis avec Nova. La fleche elle permet seulement de voir le sens technique de la base dans laquelle la donnée est enregisté.
Aucune erreur est retourné car Neo4j accepte les relations orientiés c'est à dire que AMI_DE n'est pas automatiquement symétrique, c'est au développeur de choisir.

Q5 - Neo4j utilise deux ports car il propose deux services différents :

- le port `17474` correspond à l'interface web Neo4j Browser, accessible avec
  un navigateur en HTTP ;
- le port `17687` correspond au protocole Bolt, utilisé par les applications
  et les drivers pour communiquer avec Neo4j.

Ce sont deux ports côté machine qui redirigent vers les ports internes
standard de Neo4j (`7474` pour HTTP et `7687` pour Bolt). L'application utilise
donc `bolt://localhost:17687`, tandis que l'utilisateur ouvre le Browser sur
`http://localhost:17474`. Ce n'est pas deux serveurs Neo4j, mais deux façons
de communiquer avec le même serveur.

Q6 - L'utilité du pseudo paramétré est d'empecher les injections de données utilisateur. La profondeur doit être interpolée pour des raisons de rapidité d'éxécution et encore sécurité.

Q7 - IDriver est créer en singleton car il est thread-safe couteux et gere un pool de connexions. Il est donc partagé par toute l'application.
La session neo4j elle est créée pour chaque méthode, car elle correspond à une opération temporaire. Elle est libéréé avec await using meme en cas d'erreur.
Cette regle s'applique a MongoClient et ConnectionMultiplexer car on partage les clients thread safe et longue durée au lieu d'en créer a chaque requete.

Q8 - On met les guillemets doubles car elles permettent d'utiliser le nom comme on l'écris car sans celles ci si on en met pas il retourne tout en minuscule et donc Players != players

Q9 - On garantit que le joueur ne peut pas payer le meme jeu, et la commande retourne une erreur car on essaye de mettre une valeur negatif a coins ce qui est impossible car il y a une contrainte absolue de positivité

Q10 -   ModifiedCount: 11
        ModifiedCount: 4
        ModifiedCount: 2

en SQL il aurait fallu utiliser update avec set, il aurait fallu contrairement a mongodb fournir toutes les colonnes alors que mongodb permet de modifier que certains champs et si on lancait la troisieme commande en premier alors il y aurait 2 documents réécrits entierement.

Q11 - Oui, la valeur en cache contient les prix, car Redis enregistre toute la liste des jeux sérialisée en JSON :
```
JsonSerializer.Serialize(jeux)
```
Le cache est conservé max 60 secondes.
Pendant ce temps l'api conserve un ancien catalogue, apres 60 secondse la clé expire et l'api recharge les données depuis MongoDB.
Pour forcer la mise a jour on supprime la clé
```
docker exec pixelhub-redis redis-cli DEL cache:catalogue:tous
```
Si la clé était déjà présente elle aurait retourner 1, ici 0.

Q12 -
file:matchmaking : stocke la file d'attente des joueurs pour le matchmaking
compteur:parties : stocke le compteur de parties
joueur:7 : stocke les infos des joueurs en hash (pseudo niveau xp)
classement:global : stocke le classemnt des joueurs avec leurs scores dans un sorted set

Q13 - 
Luna existe dans Neo4j, mais pas dans PostgreSQL.

Si Luna essaie d’acheter un jeu, l’achat est enregistré dans PostgreSQL. Or PostgreSQL ne connaît pas Luna dans sa table "Players". La contrainte de clé étrangère vers "Players" empêchera donc l’achat : la requête échouera, car aucun joueur correspondant à Luna n’existe dans PostgreSQL.

Cela montre que les bases ont des données indépendantes : le fait que Luna existe dans Neo4j ne la crée pas automatiquement dans PostgreSQL.

Q14 - 
```sql
UPDATE "Players"
SET "Coins" = "Coins" - @prix
WHERE "Id" = @joueurId
  AND "Coins" >= @prix;
```
ExecuteUpdateAsync réalise directement un UPDATE SQL avec une condition sur le solde. Le test Coins >= prix et la soustraction sont effectués dans une seule opération atomique. Cela évite qu’un achat concurrent lise un ancien solde et dépense deux fois la même somme.

Q15 - 
AnyAsync est seulement un contrôle anticipé pour le cas courant ; il ne protège pas contre deux requêtes concurrentes, car elles peuvent toutes les deux voir que l’achat n’existe pas encore. L’index unique de PostgreSQL constitue la vraie garantie. Si deux insertions arrivent simultanément, PostgreSQL n’en accepte qu’une et l’autre provoque une violation d’unicité. Grâce à la transaction, le débit associé à l’insertion refusée est également annulé.
Neo4j sert à consulter les relations et les recommandations, tandis que PostgreSQL reste la source fiable pour valider et enregistrer un achat.

Q16 - L’achat passe car PostgreSQL ne contient pas encore cet achat. PostgreSQL a raison pour l’état officiel de la boutique, car il est la base de référence des achats et du solde. Neo4j contient une projection importée manuellement qui indique que Krayz possède déjà Valorant, mais cette donnée n’est pas suffisante pour empêcher un achat. Cela montre que les données importées manuellement dans un graphe peuvent être incohérentes ou obsolètes par rapport à la base transactionnelle.

Q17 - Les deux achats passent et le solde final est de 0 pièce. Grâce à ExecuteUpdateAsync, chaque débit vérifie et modifie le solde directement dans PostgreSQL. Les deux opérations ne peuvent donc pas utiliser deux fois le même ancien solde.

Avec la version « lire, tester en C#, écrire », les deux requêtes auraient pu lire simultanément 800 pièces, considérer toutes les deux que l’achat était possible, puis écrire des résultats incorrects. On aurait alors pu obtenir un solde incohérent, par exemple 300 au lieu de 0, ou autoriser une dépense supérieure au solde réel.

Q18 - Une fiche joueur envoie 6 requêtes : 2 à PostgreSQL, 1 à Redis, 2 à Neo4j et 1 à MongoDB. MongoDB ne reçoit qu’une seule requête grâce à $in, même si le joueur possède 200 jeux. Avec une boucle de GetByTitleAsync, il faudrait envoyer 200 requêtes.

Q19 - Luna existe dans Neo4j, mais pas dans PostgreSQL. Le service considère PostgreSQL comme la source de vérité pour l’identité du joueur, le solde et les achats. Il s’arrête donc avant d’interroger Redis, Neo4j et MongoDB, puis renvoie 404. C’est cohérent si PostgreSQL est la base de référence des joueurs. La présence de Luna dans Neo4j ne suffit pas, car Neo4j contient seulement une projection sociale importée séparément. Cela montre que les données peuvent être incohérentes entre les moteurs.

Q20 - L’API répond 500, car elle essaie ensuite d’enregistrer la possession dans Neo4j, qui est arrêté. Cependant, PostgreSQL a déjà validé la transaction : Ombre a été débité et l’achat existe dans "Purchases". Ombre, qui ne voit que la réponse HTTP, croit donc que l’achat a échoué, alors qu’il a en réalité été enregistré dans PostgreSQL.

L’attente d’environ trente secondes vient des tentatives automatiques du driver Neo4j pour se reconnecter avant d’abandonner.

Q21 - Le deuxième appel retourne 409, car l’achat existe déjà dans PostgreSQL. Ombre n’est donc pas débité une seconde fois. Le test AnyAsync détecte normalement l’achat déjà présent, et l’index unique constitue une protection supplémentaire contre les requêtes concurrentes.

Q22 - La fiche n’est pas totalement cohérente. PostgreSQL est correct : l’achat est enregistré et le solde a été débité. En revanche, Neo4j est faux ou incomplet, car la relation POSSEDE pour Rocket League n’a pas été créée pendant la panne. Les recommandations peuvent donc encore proposer ce jeu à Ombre. La vente est également absente de Redis, car l’enregistrement Redis n’a pas été exécuté après l’échec de Neo4j. Cela montre qu’une transaction PostgreSQL ne peut pas annuler automatiquement les modifications faites dans Neo4j ou Redis.