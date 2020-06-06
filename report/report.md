# Open-source code optimization project : Imcat

#### Auteur : Pierrick Muller

## Choix du projet : Imcat

Imcat est un projet permettant de visualiser une image de n'importe quelle taille dans un terminal. J'ai choisi ce projet car l'idée de pouvoir visualiser des images dans un terminal m'intriguait et que je souhaitais travailler sur un projet concernant du traitement d'image.

Afin d'installer le programme, il suffit d'utiliser git afin de cloner le projet sur sa machine local :
- Projet de base : https://github.com/stolk/imcat
- Projet optimisé : https://github.com/mullerPierrickHeig/imcat-improved

Suite à cela, il suffit d'utiliser la commande `make` afin de créer l'éxécutable.

Le programme s'utilise de la manière suivante :
```
Usage: ./imcat image [image2 .. imageN]
```
Le programme est capable d'afficher plusieurs images à la suite.

## Stratégie de benchmark et résultats de base
En observant le code, j'ai tout de suite penser à la possibilité de paraleliser le traitement interne à la fonction `process_image`. Cela vient probablement du fait que j'étais en train d'étudier ce sujet au moment ou je me suis intéréssé a ce projet. Dans cette idée , j'ai penser m'intérésser au temps d'éxécution du programme. Pour obtenir des résultats probant, j'ai récupérer l'image "edge_stars.png" provenant d'un ancien laboratoire afin d'effectuer les tests car elle à l'avantage d'être lourde , ce qui implique un temps de traitement plus long et potentiellement par la suite une meilleure mesure de l'optimisation effectuée. Un autre facteur à prendre en compte vient de la taille de la fenêtre. En effet, plus la fenetre du terminal est grande plus le programme prend de temps car l'affichage est plus conséquent. Fort de ces informations j'ai décider de prendre 10 première mesures avec la fenêtre du terminal en plein écran et en utilisant la commande suivante permettant l'éxécution du programme 10 fois d'affilée :
```
for i in {1..10}; do time ./imcat images/edge_stars.png ; done
```
et voici les résultats obtenus lors de ces executions :
```
./imcat images/edge_stars.png  1.29s user 0.13s system 91% cpu 1.561 total
./imcat images/edge_stars.png  1.28s user 0.11s system 88% cpu 1.557 total
./imcat images/edge_stars.png  1.30s user 0.14s system 88% cpu 1.627 total
./imcat images/edge_stars.png  1.31s user 0.15s system 88% cpu 1.647 total
./imcat images/edge_stars.png  1.35s user 0.16s system 83% cpu 1.810 total
./imcat images/edge_stars.png  1.26s user 0.16s system 75% cpu 1.882 total
./imcat images/edge_stars.png  1.39s user 0.15s system 88% cpu 1.729 total
./imcat images/edge_stars.png  1.25s user 0.15s system 75% cpu 1.861 total
./imcat images/edge_stars.png  1.23s user 0.10s system 79% cpu 1.670 total
./imcat images/edge_stars.png  1.24s user 0.13s system 72% cpu 1.892 total
```

Ces résultats de base nous permettait d'établir un temps moyen d'execution du programme étant de *1.7236* secondes.

## Analyse du benchmark et idée d'optimisations
Comme dis plus haut, l'optimisation principale qui m'as sauté aux yeux et que j'ai voulu tester directement, c'est la paralelisation du contenu de la fonction `process_image`. En effet, durant mes recherches sur le code, j'ai pu m'intérésser à cette fonction qui contient des boucles traitant les pixels de l'images. Ainsi, je me suis dis que c'étais un bon endroit ou commencer. J'ai hésiter entre l'utilisation d'instructions SIMD et l'utilisation d'openMP, mais j'ai finalement choisit openMP car rien ne m'empéchait d'utiliser le pragma omp for pour la boucle interne a la fonction.

## Implémentation du code
L'ajout en code en lui même est assez simple. J'ai ajouté les lignes suivantes :
```
#include <omp.h>

.
.
.

	unsigned char out[ outh ][ outw ][ 4 ];
	#pragma omp parallel
	{
		#pragma omp for collapse(2)
		for ( int y=0; y<outh; ++y )
			for ( int x=0; x<outw; ++x )
			{

.
.
.

```
Cette solution est possible tel quelle car Le contenu de la boucle en dessous du pragma permetetait déja sans modification de la structure du code d'utiliser le pragma. En effet, toutes les problématiques qui peuvent ammenner à la modification d'une structure de boucle afin de pouvoir utiliser le pragma (dependance entre les itérations principalement) ne sont pas présentes dans cette boucle.

## Résultats du benchmark du code optimisé
```
./imcat images/edge_stars.png  2.05s user 0.12s system 132% cpu 1.630 total
./imcat images/edge_stars.png  1.81s user 0.19s system 128% cpu 1.565 total
./imcat images/edge_stars.png  2.07s user 0.13s system 152% cpu 1.440 total
./imcat images/edge_stars.png  2.05s user 0.15s system 148% cpu 1.476 total
./imcat images/edge_stars.png  2.05s user 0.15s system 171% cpu 1.283 total
./imcat images/edge_stars.png  1.90s user 0.18s system 167% cpu 1.247 total
./imcat images/edge_stars.png  2.07s user 0.14s system 137% cpu 1.605 total
./imcat images/edge_stars.png  2.11s user 0.14s system 155% cpu 1.448 total
./imcat images/edge_stars.png  1.93s user 0.18s system 134% cpu 1.572 total
./imcat images/edge_stars.png  2.02s user 0.22s system 138% cpu 1.613 total
```
résultat final : *1.4879* secondes.

On peut voir une amélioration mesurable entre le projet de base et le projet optimisé.

## Conclusion
