# Utilisation ds données satellites pour détecter l'habitat spontané en Guyane et en Mayotte

[![Onyxia](https://img.shields.io/static/v1?logo=visualstudiocode&label=&message=Open%20in%20VS%20Code&labelColor=2c2c32&color=007acc&logoColor=007acc)](https://datalab.sspcloud.fr/launcher/ide/vscode-python?autoLaunch=false&onyxia.friendlyName=%C2%ABslums-detection%C2%BB&init.personalInit=%C2%ABhttps%3A%2F%2Fraw.githubusercontent.com%2FInseeFrLab%2Fdetection-bidonvilles%2Fmain%2Fsetup.sh%C2%BB&fauxpilot.enabled=true&service.image.custom.enabled=true&service.image.pullPolicy=%C2%ABAlways%C2%BB&service.image.custom.version=%C2%ABinseefrlab%2Fdetection-bidonvilles%3Alatest%C2%BB)
[![Build](https://img.shields.io/github/actions/workflow/status/InseeFrLab/detection-bidonvilles/build-image.yaml?label=Build
)](https://hub.docker.com/repository/docker/inseefrlab/detection-bidonvilles)


## Objectif :
à partir des images fournies par Philippe + des données Ril constituer des jeux de données exploitables.

## Différents types d'exemple d'apprentissage :

1) Couples - images masques
2) Couples - images / bounding box
3) couples (petits patchs + label associé (slum/logement/autre))


## Les fonctions :
Les images d'entrées font du 2000 x 2000 en pixel il faudra donc les couper à minima en 500 x 500 pour pouvoir nourir un réseau de neurones.

Il nous faut donc :

- une fonction *decoup_raster* de découpage de raster : qui prend en entrée un raster de RGBI x 2000 x 2000 le coefficient de division (4 pour diviser par 4 l'image) et qui nous retourne 4 sous-raster.

- Pour créer les masques il nous faut une fonction *creer_masque* qui a partir  dd'un raster et donc de son extentd'une extent de raster (raster.bounds) retourne le raster à une couche dans lequel 1 correspond à un logement RIL  et 0 sinon (en gros le masque)

- Pour créer le jeu de bounding box il nous faut la fonction *creer_bb* qui au raster et son extent et au ROL associe la liste des boundings box des logements

## Pipeline Générale :

En combinant les fonctions decoup_raster + creer_masque (ou creer_bb) on peut avoir assez rapidement le couple image masque (+ info géométrique à côté).

En sortie on voudrait avoir une liste de dico ={ image : IMAGE, masque = MASK, Infogeo = INFOGEO}
avec :

- IMAGE l'image de taille nxn n <2000
- MASK le raster 0 et 1 de taille nxn
- INFOGEO l'information géographique associée

Le jeu 3) avec les patchs se constitue plutôt en créant un buffer de taille caré et de petite taille autour des points RIL et en récupérant l'image associée de petite taille également à ce patch correspond donc la classe "logement". Echantillonner d'autres points pour avoir des exemples de la classe hors logement

## Nettoyage des jeux obtenus 👍

- Certaines images ne comporteront pas de logement ou dans certains cas le RIL sera de mauvaise qualité.

LE RIL n'apporte de l'information fraiche que pour des sous ensemble de lgements chaque année (certains ilots des GC et certaines petites communes).
Il est nécessaire d'être en mesure d'isoler facilement ces endroits (définis par des polygones) et d'éliminer les endroits à cheval ?

- Réfléchir à la détection des couple images masque de pauvrequalité
  - i) Pas suffisamment de logements dans le masque voire 0 (utilisation de l'infrarouge éventuellement sur l'image ?)
  - ii) masque contenant moins de XX% de l'ilot considéré ou de la petice comune considérée ?
  - iii) Quelques controles visuels peut-être


- Quid des méthodes on supervisées
