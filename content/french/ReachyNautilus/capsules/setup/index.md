---
title: "Connexion au robot"
menu:
  main:
    name: "Connexion au robot Reachy"
    weight: 2
    parent: "capsules"
---

| Classe de capsule  | &emsp; durée recommandée |
|:-------------------|:------------------|
| Setup  &emsp;  🛠️  |&emsp; 10 min      |


## 📗 Ressources

Plus d'informations sur le robot et sa mise en route avec ces liens :  
- [Doc Pollen Robotics](https://pollen-robotics.github.io/reachy-2019-docs/docs/getting-started/)  (en anglais)
- [Prise en main Reachy](https://github.com/ta18/Reachy_Nautilus/blob/main/Prise%20en%20main.md)

  
### **Infos robot 🤖** : 
Nom du robot: **Nemo**  
Pour trouver l'adresse IP de reachy tu peux taper la commande suivante sur un terminal : `ifconfig`
![ip](img/ip.png)


## 1. Mise en route 

1. Branche l'alimentation fournie sur la prise ronde au dos du robot.
2. Vérifier que l'HDMI de l'écran et le HUB USB sont bien brancher au dos du robot 
3. Appuie sur le bouton poussoir a gauche pour mettre sous tension le NUC et sur le bouton ON/OFF à droite pour mettre sous tension les moteurs.
4. Allumer l'écran du Reachy

![Dos du robot](img/back2021.png)

## 2. Connexion au robot

Le robot Reachy est livré avec une carte NUC (mini ordinateur) qui permet de contrôler les moteurs et les périphériques qui l'équipent.   
Tu va travailler directement sur le robot.     
Lance Jupyter Notebook avec les commandes :    
```bash
cd ~/Reachy_Nautilus
jupyter notebook 
```

Et voilà tu es prêt à commancer avec le robot Reachy ! 🎉


