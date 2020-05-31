# computer-go
A policy and value estimation network in Go with residual CNN.

Collaborators: 

- AZORIN Raphaël
- COHEN Marc

------------

French version:

# Démarche


Nous avons parcouru les articles suggérés (Cazenave, 2018, Cazenave et al., 2020, Schrittwieser et al., 2019) et effectué des recherches sur l’état de l’art du computer Go. Nous avons notamment étudié l’architecture du réseau de neurones d’AlphaGo (cf. Fig. 1). 

Ensuite nous avons développé différents modèles basés sur des architectures issues de ces recherches, que nous avons comparés et progressivement soumis dans un tournoi de computer Go. Les entraînements de ces modèles ont été effectués sur les données du programme de Facebook ELF OpenGo et ces entraînements ont été ajustés tout au long du tournoi.

# Modèles développés
Les modèles à soumettre pour le tournoi ont été entraînés sur les 109 000 000 exemples de Facebook ELF OpenGo, en utilisant des dynamic batches de 100 000 exemples.


## Cheetah

### Description du modèle 

Pour mettre notre premier modèle en place, nous nous sommes inspirés de l’architecture générale d’Alpha Go tout en respectant la contrainte de la borne de 1 million de paramètres. Cette limite nous à contraint à peu différencier les deux têtes et à se concentrer sur des couches communes.
 
Les cinq couches convolutionnelles sont composées de 32 filtres de taille 3x3. Pour chacune, la fonction d’activation est relu. Ce modèle comporte deux couches résiduelles. 

Nous avons, respectivement, utilisé la categorical-crossentrepy et la MSE pour la politique et la valeur, intervenant chacune avec un poids respectif de 1 et 0.4 dans la loss globale. Nous avons accordé plus d’importance à la politique.

Comme optimiser, nous avons utilisé un SGD avec un learning rate de 0.01. 

Nous avons fait près de 1000 epochs en changeant de dynamic batchs lors de chacune, et en utilisant 5% des 100 000 données du batch comme ensemble de validation. 


### Résultats du modèle 

Nous pouvons observer sur ces figures l’évolution des fonctions de pertes respectives de la politique et de la valeur au cours de 1000 epochs. La politique ayant un poids plus fort dans la fonction de perte globale, elle décroît vite au cours des 400 premières epochs. Ensuite, son évolution est ralentie jusqu’à la 800ème epoch où elle stagne. L’effet contraire est observé pour la loss de la valeur, qui décroît réellement à partir de la 800ème epoch, lorsque le seul moyen de faire baisser la fonction de perte globale est par la fonction de perte de la valeur. Nous aurions pu essayer d’optimiser ce modèle, en modifiant les poids dans la fonction de perte, ou encore en continuant de l'entraîner avec un learning rate plus faible. Cependant, nous avons choisi de modifier l’architecture et de nous concentrer sur un autre modèle. Son accuracy était d’environ 0.34 sur la politique et 0.63 sur la loss.

## Dumbo
### Description du modèle
Nous avons construit ce réseau de neurone simplifié en nous inspirant de l’architecture d’AlphaGo, notamment pour les deux têtes : policy et value. 
Les couches convolutionnelles du tronc commun sont constituées de 194 filtres de 3x3. Ceux des têtes policy et value sont respectivement constitués de 2 filtres 1x1 et 1 filtre 1x1. Enfin, la couche cachée dense de la tête value contient 128 unités. Nous avons utilisé l’activation softmax pour la policy et la tanh pour la value (et relu pour toutes les autres).

Nous avons utilisé une categorical cross-entropy pour la policy et une MSE pour la value, chacune avec un poids de 1 dans la loss totale. Le training a été effectué en dynamic batches avec Golois, en effectuant un fit sur chaque batch (donc chaque “epoch” en ce sens). 

Enfin, nous avons utilisé différents optimiseurs : 
Stochastic Gradient Descent (Nesterov Momentum à 0.9 et decay de 0.000001) avec un learning rate de 0.01 de la première à la 500ème epoch.


 Stochastic Gradient Descent (Nesterov Momentum à 0.9 et decay de 0.000001) avec un learning rate de 0.005 de la 500ème à la 800ème epoch.


 Stochastic Gradient Descent (Nesterov Momentum à 0.9 et decay de 0.000001) avec un learning rate de 0.0025 de la 800ème à la 1200ème epoch.

Cette diminution progressive du learning rate par palier nous a permis de “relancer” l’apprentissage du modèle lorsque la loss stagnait.

### Résultats du modèle 

Nous avons finalement obtenu ainsi un modèle aux alentours de 37.2% d’accuracy pour la policy et 65.9% d’accuracy pour la value. Nous avons donc décidé de vous soumettre ce modèle Dumbo pour évaluation (cf. annexes AZORIN_COHEN_Dumbo.h5 et AZORIN_COHEN_Dumbo.ipynb). C’est sur ce modèle que nous nous sommes concentrés pour effectuer des améliorations, tournoi après tournoi.
Conclusions
Ce projet nous a permis de mettre en pratique les différents paradigmes abordés en cours. Il nous a également montré la difficulté de la tâche d’optimisation d’un modèle. Le nombre de paramètres modifiables est énorme, entre la structure du modèle, les hyperparamètres ou encore la manière d’effectuer l’apprentissage (nombre d’epochs et utilisation du dynamic batch). Il faut donc adopter une méthodologie bien précise pour s’y retrouver et d’autant plus lorsque l’on travaille en groupe.  

A posteriori et avec un peu de recul sur notre travail, nous estimons que nous n’avons pas suffisamment optimisé la valeur, qui est pourtant cruciale lors du déroulement d’un match de computer Go. Aussi, nous nous proposons de conserver l’architecture de notre modèle Dumbo, mais en se concentrant davantage sur la loss de la value (avec un poids plus fort que celui de la policy dans la fonction de perte globale).

Enfin, nous aurions également pu mettre en place une procédure de test plus efficace. En effet, nous avons utilisé les dynamic batchs pour entraîner nos modèles et ceux ci étaient issus d’un tirage aléatoire dans l’ensemble de données que nous avions. Il aurait été judicieux d’extraire un sous ensemble pour réaliser un même test sur chacun de nos modèles, avec des données qui n’auraient pas été utilisées lors de leurs entraînements respectifs.
Annexes
Code et modèle

# Sources

Medium. 2020. Alphago Zero Explained In One Diagram. [online] Available at: <https://medium.com/applied-data-science/alphago-zero-explained-in-one-diagram-365f5abf67e0> [Accessed 15 March 2020].

"Residual Networks for Computer Go", Tristan Cazenave. IEEE Transactions on Games, Vol. 10 (1), pp 107-110, March 2018.
"Mastering the game of Go without human knowledge", David Silver et al. 2017.
"Spatial Average Pooling for Computer Go", Tristan Cazenave. CGW at IJCAI 2018.
"Mastering Atari, Go, Chess and Shogi by Planning with a Learned Model", Julian Schrittwieser et al. 2019
"Polygames: Improved Zero Learning", Tristan Cazenave et al. 2020


