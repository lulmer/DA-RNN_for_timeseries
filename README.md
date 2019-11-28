# DA-RNN for timeseries


# Quel probleme cherche-t-on a résoudre ? 
On cherche a determiner les valeurs futures d'un cours de bourse. Plus precisement, on souhaite predire les valeurs d'une serie temporelle a partir des occurences precedentes de celles ci mais aussi de plusieurs plusieures series exogènes c'est à dire exterieures au probleme principal bien qu'ayant une influence sur la serie etudiée. Cette manière de modeliser une situation se nomme le model NARX (Nonlinear Autoregressive Exogenous Model). L'objectif est de deceler un comportement non linéaire de la serie. 

# l’originilatité/specificité de la Methode
## Etat de l'art 
Beaucoups de methodes ont étées mise en oeuvre pour répondre au problème : methodes a noyau, ensemblistes, gaussiennes. Le probleme est que la majorité de ces modelisation se basent sur des formes non-lineaires predefinies qui peuvent ne pas saisir de manière approprié les relations non linéaires du problème.

Les RNNs ont été une solution à ce probleme, car plus flexibles pour capturer des relations non linéaires. Mais encore une fois ils ont leurs limites comme le montre Joshua Bengio avec le probleme de l'annulation du gradient lorsque le reseau est trop profond (cad qu'il n'apprend plus). Les RNNs ont ainsi du mal à prendre l'information lorsque les series temporelles dependent de plus d'un état passé. 

Pour palier a ce probleme, les scientifiques on developpé des reseau neuronaux a memoires comme les LSTMs et GRUs. Ces architectures permetent de capturer des dependances temporelles plus etendues que les RNNs standards. De nombreux defis techniques comme, entre autres, la traduction ou la reconnaissance vocale ont ainsi pu etre relevées. Dans le cas de la traduction automatique on utilise un reseau de type encodeur/decodeur dans lequel l'encodeur analyse la sequence d'entrée et le decodeur retourne la traduction associée. Encore une fois la precision methode est vite deteriorée lorsque la longueur de la serie devient trop importante. Malheureusement cette situation est très courante dans le contexte d'analyse de series financières...

## Methode presentée
Ici on se base sur une structure de reseaux d'encodeur/décodeur et de reseaux d'attention en 2 temps pour selectioner des parties d'etats cachés au fur et a mesure du temps. 
Dans un premier temps on developpe un mecanisme d'attention pour extraire adaptativement les series les plus pertinentes a chaque instant.
Dans un second temps un mechanisme d'attention est utilisé pour selectionner les états cachés pertinents des encodeurs.
Ces deux reseaux d'attention sont implémentés avec des LSTMS. 

L'avantage de cette méthode c'est que le reseau d'attention en 2 temps peut a la fois selectionner les données importantes du probleme mais aussi capturer les dependances temporelles d'une serie de manière appropriée. 

Elle est plus performante que l'état de l'art, robuste aux données bruitées et facilement interpretable. 

L'encodeur est un RNN qui transforme les sequences d'entrées en liste de features, la fonction d'activation est un LSTM pour capturer les dependances a long terme. Cet encodeur selectionne les series pertinentes. 

Le decodeur avec attention temporelle
Les poids d'attentions de chaques états cachés de l'encodeur sont calculés a partir des etats cachés precedents du decodeur. Ces poids representent l'importance qu'a le i-eme etat caché de l'encodeur pour la prediction. Comme chaque etat caché d'encodeur *hi* correpond a une composante temporelle en entrée, le mecanisme d'attention calcule le vecteur de contexte *ct* comme la somme ponderée de l'ensemble des états cachés de l'encodeur. 
On note que le vecteur de contexte *ct* change à chaque time-step.  Une fois qu'on a l'ensemble de ces vecteurs de contexte on peut les combiner avec les series cibles données. Cette combinaison de serie permet de mettre a jour le décodeur à l'instant t. Encore une fois la fonction d'activation non linéaire est un LSTM.

# Inconvenients de la méthode

Sur des jeux de données divers la methode a tendance a sur-apprendre, c'est a dire qu'elle predit une serie très proche de la serie d'origine avec un leger decalage temporel. Nous avons focalisé notre attention sur le reglages des parametres permettant d'eviter ce probleme. Ainsi, nous cherchons a reduire le nombre de parametres du modèle tel qu'il est defini dans le papier... 

