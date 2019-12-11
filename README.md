# DA-RNN for timeseries prediction

# Quel probleme cherche-t-on a résoudre ? 
On cherche à déterminer les futures valeurs d'un cours boursier. Plus précisement, on souhaite predire les valeurs d'une serie temporelle à partir des occurences precedentes de celles ci mais aussi de plusieurs séries exogènes, c'est à dire extérieures au probleme principal bien qu'ayant une influence sur la serie etudiée. Cette manière de modeliser une situation se nomme le model NARX (Nonlinear Autoregressive Exogenous Model). L'objectif est de deceler un comportement non linéaire de la serie. 

# Originilatité/specificité de la Methode
## Etat de l'art 
Beaucoups de methodes ont été mise en oeuvre pour répondre au problème : méthodes à noyau, ensemblistes, gaussiennes. Le probleme est que la majorité de ces modélisation se basent sur des formes non-lineaires prédéfinies qui peuvent ne pas saisir de manière approprié les relations non linéaires du problème.

Les RNNs ont été une solution à ce probleme, car plus flexibles pour capturer des relations non linéaires. Mais encore une fois ils ont leurs limites comme le montre Joshua Bengio avec le probleme de la disparition du gradient lorsque la serie a predire est trop longue (le resultat c'est qu'il n'apprend plus). Les RNNs ont ainsi du mal à prendre l'information lorsque les séries temporelles dépendent de de nombreux états cachés. 

Pour palier a ce probleme, les scientifiques ont developpé des réseaux neuronaux à mémoires comme les LSTMs et GRUs. Ces architectures permetent de capturer des dépendances temporelles plus étendues que les RNNs standards. De nombreux défis techniques comme, entre autres, la traduction ou la reconnaissance vocale ont ainsi pu être relevées. Dans le cas de la traduction automatique on utilise un reseau de type encodeur/decodeur dans lequel l'encodeur analyse la sequence d'entrée et le decodeur retourne la traduction associée. Encore une fois, la précision de la méthode est vite deteriorée lorsque la longueur de la serie devient trop importante. Malheureusement cette situation est très courante dans le contexte d'analyse de series financières...

## Methode presentée
Ici on se base sur une structure de reseaux d'encodeur/décodeur et de reseaux d'attention en 2 temps pour sélectioner des parties d'etats cachés au fur et a mesure du temps. 

Dans un premier temps on developpe un mécanisme d'attention pour extraire adaptativement les series les plus pertinentes a chaque instant.
Dans un second temps un mechanisme d'attention est utilisé pour selectionner les états cachés pertinents des encodeurs.
Ces deux réseaux d'attention sont implémentés avec des LSTMS. 

L'avantage de cette méthode c'est que le reseau d'attention en 2 temps peut à la fois selectionner les données importantes du problème mais aussi capturer les dependances temporelles d'une serie de manière appropriée. 

Elle est plus performante que l'état de l'art, robuste aux données bruitées et facilement interprétable. 

L'encodeur est un RNN qui transforme les sequences d'entrées en liste de features, la fonction d'activation est un LSTM pour capturer les dependances a long terme. Cet encodeur sélectionne les séries pertinentes. 

Les poids d'attentions de chaques états cachés de l'encodeur sont calculés a partir des etats cachés precedents du décodeur. Ces poids representent l'importance qu'a le i-ème état caché de l'encodeur pour la prédiction. Comme chaque etat caché d'encodeur *hi* correspond à une composante temporelle en entrée, le mecanisme d'attention calcule le vecteur de contexte *ct* comme la somme ponderée de l'ensemble des états cachés de l'encodeur par les poids du reseau d'attention. 
On note que le vecteur de contexte *ct* change à chaque time-step. Une fois qu'on a l'ensemble de ces vecteurs de contexte on peut les combiner avec les series cibles données. Cette combinaison de serie permet de mettre a jour le décodeur à l'instant t. Encore une fois la fonction d'activation non linéaire est un LSTM.

# Inconvenients de la méthode

Sur des jeux de données divers la methode a tendance a sur-apprendre, c'est a dire qu'elle predit une serie très proche de la serie d'origine avec un leger decalage temporel. Nous avons focalisé notre attention sur le reglages des parametres permettant d'eviter ce probleme. Ainsi, nous cherchons a reduire le nombre de parametres du modèle tel qu'il est defini dans le papier... 
Elle est aussi très dependante des parametres qu'on lui donne :  

# Data CAC40

Pour tester ce modèle, nous utilisons les données historiques du cours du CAC40 à l'instant t-1 ainsi que les données de la valeur boursière de 11 entreprises du CAC40 (séries éxogènes).
Ces times series s'étalent sur 5 années, du 22/11/2014 au 28/11/2019. 
Pour chacune des 11 entreprises, nous avons au total 1500 time series (correspondantes à 1500 jours d'ouverture de la bourse sur 5 ans)
Notre base de données est donc composée de (11 * 1500 time series) + (1 * 1500 time series) 



![alt text](https://github.com/lulmer/DA-RNN_for_timeseries/blob/master/Capture%20d%E2%80%99e%CC%81cran%202019-11-28%20a%CC%80%2011.50.29.png)
