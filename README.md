# DA-RNN for timeseries prediction
Dans le cadre de notre projet de Deep Learning, nous avons décidé d'étudier la méthode "Dual-Attention Recurrent Neural Network". 
**A Dual-Stage Attention-Based Recurrent Neural Network for Time Series Prediction**
*Yao Qin1∗, Dongjin Song2, Haifeng Chen2, Wei Cheng2, Guofei Jiang2, Garrison W. Cottrell1 1University of California, San Diego; 2NEC Laboratories America, Inc.*

Nous avons, durant ce projet, analysé la méthode puis testé sur d'autres bases de données.

# Quel probleme cherche-t-on a résoudre ? 
On cherche a déterminer l'instant t+1 d'une serie temporelle . Plus précisement, on souhaite predire cette valeur à partir des occurences precedentes de celles ci mais aussi de plusieurs séries exogènes, c'est à dire extérieures au probleme principal bien qu'ayant une influence sur la serie etudiée. Cette manière de modeliser une situation se nomme le model NARX (Nonlinear Autoregressive Exogenous Model). L'objectif est de deceler un comportement non linéaire de la serie. 

# Originilatité/specificité de la Methode
## Etat de l'art 
Beaucoups de methodes ont été mise en oeuvre pour répondre au problème : méthodes à noyau, ensemblistes, gaussiennes. Le probleme est que la majorité de ces modélisation se basent sur des formes non-lineaires prédéfinies qui peuvent ne pas saisir de manière approprié les relations non linéaires du problème.

Les RNNs ont été une solution à ce probleme, car plus flexibles pour capturer des relations non linéaires. Mais encore une fois ils ont leurs limites comme le montre Joshua Bengio avec le probleme de l'annulation du gradient lorsque le réseau est trop profond (c.a.d qu'il n'apprend plus). Les RNNs ont ainsi du mal à prendre l'information lorsque les séries temporelles dépendent de plus d'un état passé. 

Pour palier a ce probleme, les scientifiques ont developpé des réseaux neuronaux à mémoires comme les LSTMs et GRUs. Ces architectures permetent de capturer des dépendances temporelles plus étendues que les RNNs standards. De nombreux défis techniques comme, entre autres, la traduction ou la reconnaissance vocale ont ainsi pu être relevées. Dans le cas de la traduction automatique on utilise un reseau de type encodeur/decodeur dans lequel l'encodeur analyse la sequence d'entrée et le decodeur retourne la traduction associée. Encore une fois la précision méthode est vite deteriorée lorsque la longueur de la serie devient trop importante. Malheureusement cette situation est très courante dans le contexte d'analyse de series financières...

## Methode presentée
Ici on se base sur une structure de reseaux d'encodeur/décodeur et de reseaux d'attention en 2 temps pour sélectioner des parties d'etats cachés au fur et a mesure du temps. 
Dans un premier temps on developpe un mécanisme d'attentioén pour extraire adaptativement les series les plus pertinentes a chaque instant.
Dans un second temps un mechanisme d'attention est utilisé pour selectionner les états cachés pertinents des encodeurs.
Ces deux réseaux d'attention sont implémentés avec des LSTMS. 

L'avantage de cette méthode c'est que le reseau d'attention en 2 temps peut à la fois selectionner les données importantes du problème mais aussi capturer les dependances temporelles d'une serie de manière appropriée. 

Elle est plus performante que l'état de l'art, robuste aux données bruitées et facilement interprétable. 

L'encodeur est un RNN qui transforme les sequences d'entrées en liste de features, la fonction d'activation est un LSTM pour capturer les dependances a long terme. Cet encodeur sélectionne les séries pertinentes. 

Le décodeur avec attention temporelle
Les poids d'attentions de chaques états cachés de l'encodeur sont calculés a partir des etats cachés precedents du décodeur. Ces poids representent l'importance qu'a le i-ème état caché de l'encodeur pour la prédiction. Comme chaque etat caché d'encodeur *hi* correspond a une composante temporelle en entrée, le mecanisme d'attention calcule le vecteur de contexte *ct* comme la somme ponderée de l'ensemble des états cachés de l'encodeur. 
On note que le vecteur de contexte *ct* change à chaque time-step. Une fois qu'on a l'ensemble de ces vecteurs de contexte on peut les combiner avec les series cibles données. Cette combinaison de serie permet de mettre a jour le décodeur à l'instant t. Encore une fois la fonction d'activation non linéaire est un LSTM.

# Inconvenients de la méthode

La méthode développée à pour inconvénient de prédire y(t+1) à partir de y(t) ce qui n’est pas toujours pertinent, en effet l’intérêt principale du machine learning est de faire des prédictions sur le long terme, si l’on prends un modèle qui prédit simplement y(t-1) pour y(t) on obtient en générale un petit RMSE. 
Ce sont les séries exogènes qui ici empêchent de faire des prédictions sur le long terme, on ne les prédit pas à un instant t. 
De plus le modèle est dépendant des séries exogènes, malgré qu’il soit résistant au bruit, si il n’y a pas de corrélation avec le signal à prédire, cela va dégrader les prédictions. 

# Data 
Pour tester ce modèle, nous avons utiliser différents DataSets (séries éxogènes). Nous avons testé des données plus ou moins volumineuses et avec plus ou moins de données exogènes. 
Notre modèle a notamment été testé sur des données financières, mais également sur des données métérologiques.

![alt text](https://github.com/lulmer/DA-RNN_for_timeseries/blob/master/illustrations/Capture%20d%E2%80%99e%CC%81cran%202019-12-11%20a%CC%80%2011.40.04.png)

Le schéma ci-dessus présente comment se réprésentent nos données dans notre modèle. 
En prenant l'exemple des données du CAC40, on va apprendre notre modèle sur différents time series correspondant au cours boursier de plusieurs entreprises du CAC40 et du cours du CAC40 à l'instant t-1 également. A partir de ces données d'apprentissage, on va chercher à prédire le futur cours du CAC40. On utilise donc des données extérieures (les cours d'autres entreprises) au probleme principal mais ayant une influence sur la serie etudiée.  

## CAC40
11 time series
1277 observations (ou points de data) 
Ces times series s'étalent sur 1 année, du 22 Novembre 2018 au 28 Novembre 2019. 
Cela correspond à 1 observation/jour/timeserie

## NASDAQ
81 time series
34448 observations (ou points de data) 
Ces times series s'étalent du 26 Juillet 2016 au 22 Décembre 2016. 
Cela correspond à 390 observations/jour/timeserie

## Cryptomonnaie
2 time series 
846 observations (ou points de data) 

## Weather 
17 time series 
2764 observations (ou points de data) 


# Expérimentations

## CAC40
![alt text](https://github.com/lulmer/DA-RNN_for_timeseries/blob/master/illustrations/exp_CAC40.png)

## NASDAQ
![alt text](https://github.com/lulmer/DA-RNN_for_timeseries/blob/master/illustrations/exp_NASDAQ.png)

![alt text](https://github.com/lulmer/DA-RNN_for_timeseries/blob/master/illustrations/exp_NASDAQ_noise.png)

## Cryptomonnaie
![alt text](https://github.com/lulmer/DA-RNN_for_timeseries/blob/master/illustrations/exp_crypto.png)

## Weather 
![alt text](https://github.com/lulmer/DA-RNN_for_timeseries/blob/master/illustrations/exp_weather.png)
