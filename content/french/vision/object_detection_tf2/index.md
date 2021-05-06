---
title: "Détection d'objets avec tensorflow2"
menu:
  main:
    name: "Détection d'objets avec tf2"
    weight: 3
    parent: "vision"
---

Dans cette section nous proposons d'utiliser l'API __Tensorflow Object Detection__ (_a.k.a_ TOD) qui propose :
* une collection de réseaux déjà entraînés spécialement conçus pour pour la détection d'objets dans des images (__Object Detection__),
* le mécanisme de _transfert learning_ pour continuer l'entraînement des réseaux pré-entraînés avec nos propres images labellisées, 
pour obtenir la détection des objets qui nous intéressent.

Contrairement à la stratégie de __Classification__ présentée dans dans la section [Classification tf2](https://learn.e.ros4.pro/fr/vision/classification_tf2/), 
la __Détection d'objets__ permet de trouver directement les boîtes englobantes des objets "face de cube avec un 1" et "face de cube avec un 2".

Cette approche évite la phase de traitement d'image classique pour extraire puis classifier les images des faces des cubes. 

Le traitement d'image basé sur une approche traditionnelle de manipulation des pixels de l'image (seuillage, extraction de contour, segmentation...) reste assez fragile : en particulier il est  sensible à la luminosité, à la présence ou non d'un fond noir... Un avantage attendu de l'approcje Object Detection est de fournir directement les boîtes englobantes des faces des cubes sans passer par une étapde de traitement d'image.

## Prérequis

* Bonne compréhension de Python et numpy
* Une première expérience des réseaux de neurones est souhaitable.

L'entraînement des réseaux de neurones avec le module `tensorflow` se fera de préférence dans un environnement virtuel Python (EVP) qui permet de travailler dans un environnement Python dédié.
Vous pouvez vous réferrer à la [FAQ Python : environnement virtuel](https://learn.e.ros4.pro/fr/faq/python_venv/) 

## 1. Documentation

Documentation générale sur numpy :
* [Numpy cheatsheet](https://s3.amazonaws.com/assets.datacamp.com/blog_assets/Numpy_Python_Cheat_Sheet.pdf)
* [NumPy quickstart](https://numpy.org/devdocs/user/quickstart.html)

Documentation sur l'_API TOD_ pour `tensorflow2` :
* Le tutoriel officiel complet : [TensorFlow 2 Object Detection API tutorial](https://tensorflow-object-detection-api-tutorial.readthedocs.io/en/latest/index.html)
* Le dépôt git : [models/research/object_detection](https://github.com/tensorflow/models/tree/master/research/object_detection)

Ce tutoriel peut être consulté pour aller chercher des détails qui ne sont pas développés dans l'activité proposée, mais il est préferrable de suivre 
les indications du paragraphe suivant pour installer une version récente de tensorflow2. 

## 2. Installation de l'API TOD

### 2.1 Clonage du dépôt `tensorflow/models`

Créér un dossier spécifique pour le travail avec l'API TOD, par exemple tod_tf2 :

	(tf2) jlc@pikatchou~$ cd <quelque_part>   # choisis le répertoire où créer tod_tf2, par exemple : ~/catkin_ws/
	(tf2) jlc@pikatchou~$ mkdir tod_tf2

Va dans le dossier `tod_tf2`  et clone la branche master du dépôt github tensorflow/models :

	(tf2) jlc@pikatchou~$ cd tod_tf2
	(tf2) jlc@pikatchou~$ git clone https://github.com/tensorflow/models.git

Une fois téléchargé, tu obtiens un dossier racine `models` qui contient quatre dossiers : l’API TOD est dans le dossier `models/research/object_detection` :
	
	(tf2) jlc@pikatchou~$ cd .. && tree -d -L 2 tod_tf2
	tod_tf2
	└── models
	    ├── community
	    ├── official
	    ├── orbit
	    └── research

	

Complète ton installation avec quelques paquets Python utiles pour le travail avec l'API TOD :

	(tf2) jlc@pikatchou $ conda install cython contextlib2 pillow lxml
	(tf2) jlc@pikatchou $ pip install labelimg

Mets à jour la variable d’environnement `PYTHONPATH` en ajoutant à la fin du fichier ~/.bashrc les deux lignes :

	export TOD_TF2="<chemin absolu du dossier tod_tf2>"
	export PYTHONPATH=$TOD_TF2/models:$TOD_TF2/models/research:$PYTHONPATH

remplace `"<chemin absolu du dossier tod_tf2>"` par le chemin absolu du dossier `tod_tf2` sur ta machine.

Lance un nouveau terminal pour activer le nouvel environnement shell ; tout ce qui suit sera fait dans ce nouveau terminal.

### 2.2 Installer les outils `protobuf`

L’API native TOD utilise des fichiers `*.proto` pour la configuration des modèles et le stockage des paramètres d’entraînement. 
Ces fichiers doivent être traduits en fichiers `*.py` afin que l’API Python puisse fonctionner correctement.  

Du dois installer en premier la commande `protoc` :

	(tf2) jlc@pikatchou $ sudo apt install protobuf-compiler

Tu peux ensuite te positionner dans le dossier `tod_tf2/models/research` et taper :

	# From tod_tf2/models/research/
	(tf2) jlc@pikatchou $ protoc object_detection/protos/*.proto  --python_out=.

Cette commande travaille de façon muette.

### 2.3 Installer l'API COCO

COCO est une banque de données destinée à alimenter les algorithmes de détection d’objets, de segmentation… voir [cocodataset.org](https://cocodataset.org) pour les tutoriels, publications… 

💻 Pour installer l’API Python de COCO, clone le site cocoapi dans le dossie `/tmp`, tape la commande `make` dans le dossier `cocoapi/PythonAPI`, puis recopie le dossier `pycococtools` dans `models/research/` :

	(tf2) jlc@pikatchou~$ cd /tmp
	(tf2) jlc@pikatchou~$ git clone  https://github.com/cocodataset/cocoapi.git
	(tf2) jlc@pikatchou~$ cd cocoapi/PythonAPI/
	(tf2) jlc@pikatchou~$ make
	(tf2) jlc@pikatchou~$ cp -r pycocotools/ <chemin absolu du dossier tod_tf2>/models/research/

### 2.4 Finalisation 

💻 Pour finir l'installation, place-toi dans le dossier  `models/research/` et tape les commandes :

	# From tod_tf2/models/research/
	(tf2) jlc@pikatchou $ cp object_detection/packages/tf2/setup.py .
	(tf2) jlc@pikatchou $ python setup.py build
	(tf2) jlc@pikatchou $ pip install .

### 2.5 Tester l'installation de l'API TOD

💻 Pour tester ton installation de l’API TOD, place-toi dans le dossier `models/research/` et tape la commande :

	# From within tod_tf2/models/research/
	(tf2) jlc@pikatchou~$ python object_detection/builders/model_builder_tf2_test.py

Le programme déroule toute une série de tests et doit se terminer par un OK :

	...
	[       OK ] ModelBuilderTF2Test.test_invalid_second_stage_batch_size
	[ RUN      ] ModelBuilderTF2Test.test_session
	[  SKIPPED ] ModelBuilderTF2Test.test_session
	[ RUN      ] ModelBuilderTF2Test.test_unknown_faster_rcnn_feature_extractor
	INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_unknown_faster_rcnn_feature_extractor): 0.0s
	I0505 18:19:38.639148 140634691176256 test_util.py:2075] time(__main__.ModelBuilderTF2Test.test_unknown_faster_rcnn_feature_extractor): 0.0s
	[       OK ] ModelBuilderTF2Test.test_unknown_faster_rcnn_feature_extractor
	[ RUN      ] ModelBuilderTF2Test.test_unknown_meta_architecture
	INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_unknown_meta_architecture): 0.0s
	I0505 18:19:38.640017 140634691176256 test_util.py:2075] time(__main__.ModelBuilderTF2Test.test_unknown_meta_architecture): 0.0s
	[       OK ] ModelBuilderTF2Test.test_unknown_meta_architecture
	[ RUN      ] ModelBuilderTF2Test.test_unknown_ssd_feature_extractor
	INFO:tensorflow:time(__main__.ModelBuilderTF2Test.test_unknown_ssd_feature_extractor): 0.0s
	I0505 18:19:38.641987 140634691176256 test_util.py:2075] time(__main__.ModelBuilderTF2Test.test_unknown_ssd_feature_extractor): 0.0s
	[       OK ] ModelBuilderTF2Test.test_unknown_ssd_feature_extractor
	----------------------------------------------------------------------
	Ran 21 tests in 53.105s

	OK (skipped=1)

💻 Pour finir, tu peux vérifier l’installation en utilisant un cahier IPython de l'API TOD  : dans le dossier `models/research/object_detection` lance la commande jupyter notebook et charge le cahier `colab/object_detection_tutorial.ipynb` : 

* ⚠️ Avant d'exécuter les cellules du notebook, il faut corriger une erreur dans le fichier `.../miniconda3/envs/tf2/lib/python3.8/site-packages/object_detection/utils/ops.py`, ligne 825 :
remplacer `tf.uint8` par `tf.uint8.as_numpy_dtype`
* ⚠️ Attention à ne pas excécuter les cellules du notebook comportant les commandes `!pip install ...` ou qui contiennent la _magic chain_ `%%bash` :	

![notebook_test_TOD.png](img/notebook_test_TOD.png)

Exécute les cellules une à une, tu ne dois pas avoir d’erreur :

* La partie "__Detection__" (qui dure quelques secondes ou quelques minutes suivant ton CPU…) utilise le réseau pré-entraîné `ssd_mobilenet_v1_coco_2017_11_17` pour détecter des objets dans les   images de test :	

![notebook_test_TOD_image1et2.png](img/notebook_test_TOD_image1et2.png)

* La partie "__Instance Segmentation__" utilise le réseau pré-entraîné `mask_rcnn_inception_resnet_v2_atrous_coco_2018_01_28` pour détecter les objets et leurs masques, par exemple :

![notebook_test_TOD_image-mask1.png](img/notebook_test_TOD_image-mask1.png)

La suite du travail se décompose ainsi :
* Créer l'arborescence de travail
* Télécharger le réseau pré-entraîné
* Créer la banque d'images labellisées pour l'entraînement supervisé du réseau choisi
* Entraîner le réseau avec la banque d'images labellisées.
* Évaluer les inférences du réseau avec les images de test
* Intégrer de l'exploitation du réseau dans l'environnement ROS.

## 3. Créer l'arborescence de travail

L'arborescence générique de travail proposée pour cette activité est la suivante :

	tod_tf2
	├── workspace
	│   ├── images
	│   │   └──<project>
	│   │       ├── test
	│   │       │   └── *.jpg, *.png ... *.xml
	│   │       ├── train
	│   │       │   └── *.jpg, *.png ... *.xml
	│   │       └── *.csv
	│   ├── pre_trained
	│   │	└── <pre_trained-network>
	│   └── training
	│       ├──<project>
	│       │   └── <pre_trained-network>
	│       ├── train.record
	│       ├── test.record
	│       └── label_map.txt
	└── models
	    └── research
	        └── object_detection
	
	

* Le dossier `workspace/images/` contient un dossier pour chaque projet, avec dedans :
	* les dossiers `test` et `train` qui contiennent chacun :
		* les images (\*.png, \*.jpg) à analyser,
		* les fichiers d'annotation (\*.xml) faits avec le logiciel `labelImg` : ils donnent, pour chacun des objets d'une image, les coordonnées de la boîte englobante et le label de l'objet.
	* les fichiers d'annotation \*.cvs (convertis au format CSV), qui seront à leur tour convertis au format _tensorflow record_.
* Le dossier `workspace/pre_trained/` contient un sous-dossier pour chacun des réseaux pré-entrainés utilisé.
* le dossier `workspace/training/` contient  également un dossier pour chaque projet, avec dedans :
	* un dossier pour chacun des réseaux pré-entrainés utilisé : c'est dans ce dossier que seront stockés les fichiers des poids du réseau entraîné,
	* les fichiers `train.reccord`  et `test.reccord` : contiennent les données labelisées d'entraînement et de test converties du format CSV au format _tensorflow record_,
	* le fichier `label_map.txt` : liste les labels correpondants aux objets à détecter.
	
Pour la détection des faces des cubes dans les images faites par le robot Poppy Ergo Jr, le dossier `<project>` sera nommé `faces_cubes`, ce qui donne l'arborescence de travail :

	tod_tf2
	├── workspace
	│   ├── images
	│   │   └── faces_cubes
	│   │       ├── test
	│   │       │   └── *.jpg, *.png ... *.xml
	│   │       ├── train
	│   │       │   └── *.jpg, *.png ... *.xml
	│   │       └── *.csv
	│   ├── pre_trained
	│   │	└── <pre_trained-network>
	│   └── training
	│       ├── faces_cubes
	│       │   └── <pre_trained-network>
	│       ├── train.record
	│       ├── test.record
	│       └── label_map.txt
	└── models
	    └── research
	        └── object_detection
	
	


Quelques commandes shell suffisent pour créer les premiers niveaux de cette arborescence :

	# From within tod_tf2
	(tf2) jlc@pikatchou $ mkdir -p workspace/images/faces_cubes/test
	(tf2) jlc@pikatchou $ mkdir -p workspace/images/faces_cubes/train
	(tf2) jlc@pikatchou $ mkdir -p workspace/pre_trained
	(tf2) jlc@pikatchou $ mkdir -p workspace/training/faces_cubes

On vérifie :

	# From within tod_tf2
	(tf2) jlc@pikatchou $ tree -d workspace
	workspace/
	├── images
	│   └── faces_cubes
	│       ├── test
	│       └── train
	├── pre_trained
	└── training
	    └── faces_cubes

	 
## 4. Télécharger le réseau pré-entraîné

Deux grandes familles de réseaux dédiées à la détection d’objets dans des images sont proposés sur Le dépôt git [TensorFlow 2 Detection Model Zoo](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/tf2_detection_zoo.md) :

* Les réseaux __R-CNN__ (_Region-based Convolutional Neural Network_) : basés sur le concept de __recherche ciblée__ (_selective search_). Au lieu d’appliquer une seule fenêtre à toutes les positions possibles de l’image, l’algorithme de recherche ciblée génère 2000 propositions de régions d’intérêts où il est le plus probable de trouver des objets à détecter. Cet algorithme se base sur des éléments tels que la texture, l’intensité et la couleur des objets qu’il a appris à détecter pour proposer des régions d’intérêt. Une fois les 2000 régions choisies, la dernière partie du réseau produit la probabilité que l’objet dans la région appartienne à chaque classe.
Il existe également des versions __Fast R-CNN__ et __Faster R-CNN__ qui permettent de rendre l’entraînement plus rapide;

* Les réseaux __SSD__ (_Single Shot Detector_) : font partie des détecteurs considérant la détection d’objets comme un problème de régression les plus connus. L'algorithme __SSD__ utilise d’abord un réseau de neurones convolutif pour produire une carte des points clés dans l’image puis, comme __Faster R-CNN__, utilise des cadres de différentes tailles pour traiter les échelles et les ratios d’aspect.

La différence entre Faster R-CNN et SSD est qu’avec R-CNN on réalise une classification sur chacune des 2000 fenêtres générées par l’algorithme de recherche ciblée, alors qu’avec SSD on cherche à prédire la classe ET la fenêtre de l’objet, en même temps. SSD apprend les décalages à appliquer sur les cadres utilisées pour encadrer au mieux l’objet plutôt que d’apprendre les fenêtres en elle-mêmes (ce que fait Faster R-CNN). Cela rend SSD plus rapide que Faster R-CNN, mais également moins précis.

Pour le travail de reconnaissance des faces des cubes dans les images fournies par la caméra du robot Ergo Jr tu peux télécharger le réseau `SSD MobileNet V1 FPN 640x640` sur le site [TensorFlow 2 Detection Model Zoo](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/tf2_detection_zoo.md).

Une fois téléchargé, il faut extraire l'archive TGZ au bon endroit de l'arborescence de travail :

	# From within tod_tf2
	(tf2) jlc@pikatchou $ tar xvzf ~/Téléchargements/ssd_mobilenet_v1_fpn_640x640_coco17_tpu-8.tar.gz -C workspace/pre_trained

puis créer le dossier `ssd_mobilenet_v1_fpn_640x640_coco17_tpu-8` dans le dossier `workspace/training/faces_cubes` :
	
	(tf2) jlc@pikatchou $ mkdir workspace/training/faces_cubes/ssd_mobilenet_v1_fpn_640x640_coco17_tpu-8

On vérifie :

	(tf2) jlc@pikatchou $ tree -d workspace
	workspace
	├── images
	│   └── faces_cubes
	│       ├── test
	│       └── train
	├── pre_trained
	│   └── ssd_mobilenet_v1_fpn_640x640_coco17_tpu-8
	│       ├── checkpoint
	│       └── saved_model
	│           └── variables
	└── training
	    └── faces_cubes
		└── ssd_mobilenet_v1_fpn_640x640_coco17_tpu-8


## 4. Construction de la banque d'images labellisées 


## 5. Entraînement du réseau avec la banque d'images


## 6. Évaluation des inférences du réseau avec les images de test


## 7. Intégration

Il est maintenant temps d'intégrer les deux parties du pipeline pour l'utilisation finale. Ouvrez le fichier `main.py` à la racine du projet.

Pour que les deux parties du pipeline s'adaptent correctement, vous avez complété la fonction `preprocess_sprites` pour mettre les vignettes renvoyées par la partie détection dans un format compatible avec celui des images MNIST.

Exécuter maintenant le programme `main.py` : donner le chemin d'un dossier qui contient les fichiers du réseau entraîné et vous devriez commencer à obtenir la reconnaissance des chiffres '1' et '2' dans les images fournies.

Il faudra certainement refaire plusieurs fois l'entraînement du réseau en jouant sur plusieurs paramètres avant d'obtenir un réseau entraîné qui fonctionne correctement :

* la valeur de la graine `SEED` peut conduire à un état initial des poids du réseau qui donne un entraînement meilleur ou pas...

* augmenter/diminuer `BATCH_SIZE` peut modifier les temps de calcul et la qualité du réseau entraîné...

* augmenter/diminuer le paramètre `patience` du callback `EarlyStopping`...

* enfin, tous les paramètres qui définissent les couches de convolution et de __spooling__ du réseau convolutif sont autant de possibilités d'améliorer ou pas les performances du réseau entraîné....

À vous de jouer pour obtenir un réseau entraîné classifiant le mieux possible les chiffres '1' et '2' dans les images fournies par la caméra du robot...

Pour confirmer la qualité de votre réseau entraîné vous pouvez enregistrer vos propres fichiers PNG avec les images faites avec la caméra du robot en utilisant le service ROS `/get_image`. Aidez-vous des idications du paragraphe __2.4. Récupérer les images de la caméra en Python__ dans la section [Manipulation/Poppy Ergo Jr](https://learn.ros4.pro/fr/manipulation/ergo-jr/) : vous pouvez ajouter une instruction `cv2.imwrite(<file_name>, image)` pour écrire vos propres fichiers PNG dans le répertoire `data/ergo_cubes/perso` et modifier en conséquence la variable `img_dir` du fichier `main.py`.

Lancer le programme et observer les performances de votre réseau opérant sur vos propres images.
