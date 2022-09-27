---
tags: Tuto, Domotique
---

# Manuel d'appairage du module communicant Linky

Dans ce manuel, vous trouverez un guide pour pouvoir connecter votre module en toute simplicité au système Home Assistant
Ce manuel se base sur une utilisation du système domotique home assistant.

## Etape 1 : Automatisation pour l’appairage

 :::info 
 Si vous avez déjà créé cette automatisation, vous pouvez sauter cette étape.
 :::
 
 
 Dans home assistant, créer une automatisation en code yaml, et écrire : 
![](https://arno59.ynh.fr//hedgedoc/uploads/c890b30383216ee80ef478200.png)

```yaml
alias: Appairage MQTT
description: Permet envoi trame de validation pour les ESP maison
trigger:
  - platform: mqtt
    topic: +/bind
condition: []
action:
  - delay:
      hours: 0
      minutes: 0
      seconds: 5
      milliseconds: 0
  - service: mqtt.publish
    data:
      payload: '2'
      topic: '{{trigger.topic.split("/")[0]}}/action/param/id'
  - service: persistent_notification.create
    data:
      message: 'Nom/adresse IP : {{trigger.payload}}'
      title: Nouveau module détecté
mode: single
```

## Etape 2 : Connecter votre module à votre Linky
Connecter votre module au Linky via un cordon avec deux câbles. Connecter ensuite à une prise USB. Le module va alors créer un réseau wifi intitulé ESPap.

## Etape 3 : Se connecter au wifi ESPap
Sur windows, en bas à droite cliquer sur l’icône Wifi, et se connecter au wifi ESPap, avec le mot de passe : 123456789

## Etape 4 : Renseigner la configuration de votre réseau
Une fois connecté sur ESPap, dans le navigatuer, entrer l'addresse : **192.168.4.1/config_local_network**
![](https://arno59.ynh.fr//hedgedoc/uploads/c890b30383216ee80ef478201.png)

Entrer les identifiants, wifi, broker MQTT, et IP du broker.
![](https://arno59.ynh.fr//hedgedoc/uploads/c890b30383216ee80ef478202.png)

SSID network est le nom de votre wifi  à la maison.
Password est le mot de passe (en général présent sous la box)

le nom du broker MQTT se figure au niveau de votre configuration MQTT (par exemple dans home assistant, dans l’onglet configuration (username sous Home Assistant)
Idem pour le mot de passe.

L’IP est celle du broker. Si installé sous Home assistant, il s’agit de l’ip de votre raspberry. Vous pouvez la trouver facilement dans l’onglet système -> réseau, au niveau du superviseur (attention, ici sans le /24) :
![](https://arno59.ynh.fr//hedgedoc/uploads/00fdd2fb-f372-445a-816a-5b8432a3403f.png)


Une fois que tout est renseigné, appuyer sur ‘SEND’, et normalement, vous recevrez un message comme quoi les infos ont bien été rentrées.

Attendre un peu, et vous devrez trouver votre module sur le réseau.

:::warning
Si jamais vous ne retrouvez pas votre module sur le réseau, une erreur est certainement survenue. Il faut alors redémarrer le module et recommencer depuis l'étape 2. 
:::

Aller ensuite sur l'interface Home Assistant. Une notification va apparaître avec le nom de votre nouveau module et son adresse IP. 

## Etape 5 : Configurer Home assistant

Afin de pouvoir visualiser les index de votre compteur, il faut écrire dans le fichier de configuration HA. Dans le fichier configuration.yaml, ajouter les lignes suivantes : 
:::info 
Le module est capable de transférer les 5 premiers index du linky. Vous pourrez alors supprimer les index inutilisés de votre fichier de config par la suite
:::

```yaml
mqtt:
  sensor:
  #compteur_EDF
    - name: "index1"
      unit_of_measurement: "Wh"
      state_class: "total_increasing"
      device_class: "energy"
      state_topic: "compteur_EDF/inHP"
    - name: "index2"
      unit_of_measurement: "Wh"
      state_class: "total_increasing"
      device_class: "energy"
      state_topic: "compteur_EDF/inHP2"
    - name: "index3"
      unit_of_measurement: "Wh"
      state_class: "total_increasing"
      device_class: "energy"
      state_topic: "compteur_EDF/inHP3"
    - name: "index4"
      unit_of_measurement: "Wh"
      state_class: "total_increasing"
      device_class: "energy"
      state_topic: "compteur_EDF/inHP4"
    - name: "index5"
      unit_of_measurement: "Wh"
      state_class: "total_increasing"
      device_class: "energy"
      state_topic: "compteur_EDF/inHP5"
    - name: "index_inj"
      unit_of_measurement: "Wh"
      state_class: "total_increasing"
      device_class: "energy"
      state_topic: "compteur_EDF/inINJ" 
    - name: "Puissance instantanée"
      unit_of_measurement: "W"
      state_topic: "compteur_EDF/Pins"  
    - name: "Etat compteur EDF"
      state_topic: "compteur_EDF/state"
      expire_after : 7200     
```

Redémarrer HA pour que vos modifications soient prises en compte. 

Vous pourrez ensuite utiliser les index pour afficher vos courbes de consommation dans l'onglet énergie de HA. 