# LAB 15 : Analyse Dynamique Android — Inspection TLS/HTTPS et Gestion du SSL Pinning

 
**Cours : Sécurité des applications mobiles**  
**Environnement : Windows PowerShell, Android Emulator, ADB, Frida, Burp Suite**

---

## 1. Introduction

Ce lab porte sur l’analyse dynamique d’une application Android dans un environnement de test contrôlé.  
L’objectif est d’observer le comportement d’une application Android pendant son exécution, de configurer un proxy d’interception, puis d’utiliser Frida pour analyser les mécanismes liés au trafic HTTPS, au SSL/TLS et au SSL Pinning.

L’application utilisée dans ce lab est :

**owasp.mstg.uncrackable3**

Ce travail est réalisé uniquement dans un cadre pédagogique et légal.

---

## 2. Avertissement éthique

Les techniques utilisées dans ce lab doivent être appliquées uniquement dans un environnement autorisé :

- applications personnelles ;
- laboratoires de cybersécurité ;
- plateformes de formation ;
- environnements de test contrôlés ;
- audits autorisés.

Il est interdit d’utiliser ces techniques contre des applications réelles sans autorisation.

---

## 3. Objectifs du lab

Les objectifs principaux de ce lab sont :

- préparer un environnement Android pour l’analyse dynamique ;
- utiliser ADB pour communiquer avec l’émulateur Android ;
- utiliser Frida pour lister et instrumenter les applications ;
- configurer Burp Suite comme proxy d’interception ;
- configurer le proxy réseau dans Android ;
- analyser les fonctions SSL/TLS avec Frida ;
- tester le comportement de l’application face au SSL Pinning ;
- interpréter les résultats obtenus pendant l’exécution.

---

## 4. Outils utilisés

Les outils utilisés sont :

- Android Emulator ;
- ADB ;
- Frida ;
- Frida-tools ;
- Burp Suite ;
- Windows PowerShell ;
- Application Android de test : owasp.mstg.uncrackable3.

---

## 5. Préparation de l’environnement avec ADB et Frida

La première étape consiste à vérifier que l’émulateur Android est bien lancé et accessible depuis la machine Windows.

Ensuite, les ports utilisés par Frida sont redirigés avec ADB :

    adb forward tcp:27042 tcp:27042
    adb forward tcp:27043 tcp:27043

Après la redirection des ports, la commande suivante permet de lister les applications visibles par Frida :

    frida-ps -Uai

Cette commande affiche les applications installées sur l’émulateur Android ainsi que leurs identifiants de package.


<img width="1449" height="1085" alt="pic1" src="https://github.com/user-attachments/assets/b431ba13-56e0-42a8-b9b8-cee48d034899" />



---

## 6. Configuration du proxy dans Burp Suite

Burp Suite est utilisé comme proxy afin d’intercepter et analyser le trafic HTTP/HTTPS de l’émulateur Android.

Dans Burp Suite, il faut accéder aux paramètres du proxy listener et vérifier que le port utilisé est :

    8080

Le listener doit être configuré pour accepter les connexions provenant de l’émulateur Android.

Dans ce lab, le proxy listener est configuré sur toutes les interfaces afin de permettre la communication entre Android et Burp Suite.

<img width="532" height="353" alt="pic2" src="https://github.com/user-attachments/assets/83c5c866-1a77-4c15-905b-fa07dcc0b6fd" />


---

## 7. Configuration du proxy sur Android

Après la configuration de Burp Suite, le proxy doit être configuré dans les paramètres Wi-Fi de l’émulateur Android.

La configuration utilisée est :

    Proxy : Manual
    Proxy hostname : 192.168.1.x
    Proxy port : 8080

Cette étape permet de rediriger le trafic réseau de l’émulateur Android vers Burp Suite.

Le but est de pouvoir observer les requêtes générées par l’application pendant son exécution.


<img width="363" height="733" alt="pic3" src="https://github.com/user-attachments/assets/345c5fc2-75d6-4c0b-89b1-805b484ae186" />




---

## 8. Lancement de l’application avec Frida

Une fois l’environnement prêt, l’application cible est lancée avec Frida.

La commande utilisée est :

    frida -U -f owasp.mstg.uncrackable3 -l bypass_root_basic.js

Cette commande permet de lancer l’application Android tout en chargeant un script Frida.

Le script utilisé permet d’observer certains mécanismes de protection de l’application, notamment les vérifications liées au root.

L’exécution montre que l’application est lancée dans l’émulateur Android et que certains contrôles sont interceptés.

<img width="1539" height="1022" alt="pic4" src="https://github.com/user-attachments/assets/6c28fa89-8eb3-4885-a55e-cce618ef50c7" />

---

## 9. Analyse des fonctions SSL/TLS avec Frida Trace

Pour analyser les fonctions liées au SSL/TLS, la commande frida-trace est utilisée.

La commande exécutée est :

    frida-trace -U -i SSL_* -i X509_* owasp.mstg.uncrackable3

Cette commande permet de tracer les fonctions SSL et X509 utilisées par l’application.

Frida génère automatiquement des handlers pour les fonctions détectées.  
Ces handlers permettent ensuite d’observer les appels liés aux certificats, aux connexions sécurisées et aux mécanismes de vérification SSL.

Cette étape est importante pour comprendre comment l’application gère les connexions HTTPS.

<img width="832" height="362" alt="pic5" src="https://github.com/user-attachments/assets/96e5155b-8fd8-4a7b-9c96-bc301c584a68" />


---

## 10. Test du SSL Pinning

Dans cette étape, un script Frida est utilisé pour tester le comportement de l’application face au SSL Pinning.

La commande utilisée est :

    frida -U -f owasp.mstg.uncrackable3 -l sslpin_bypass_universal.js -l sslpin_bypass_native.js

Pendant l’exécution, Frida affiche plusieurs messages indiquant que certaines fonctions SSL sont interceptées.

Cependant, l’application provoque ensuite un crash avec le message :

    Process crashed: Trace/BPT trap

Ce résultat montre que l’application peut réagir de manière sensible à l’instrumentation dynamique.

Le crash peut être causé par plusieurs éléments :

- incompatibilité du script Frida avec l’application ;
- mauvaise fonction native ciblée ;
- protection anti-debug ;
- détection d’instrumentation ;
- incompatibilité avec l’architecture de l’émulateur ;
- erreur dans le hook utilisé.

Même si l’application a crashé, ce résultat reste utile car il montre que l’analyse dynamique permet d’identifier les réactions de sécurité d’une application.

<img width="1364" height="1153" alt="pic6" src="https://github.com/user-attachments/assets/9e1fb62a-c9af-4509-b8b9-78c8ccabea85" />


## 11. Résultats obtenus

Durant ce lab, les étapes suivantes ont été réalisées avec succès :

- redirection des ports Frida avec ADB ;
- détection des applications Android avec Frida ;
- configuration du proxy Burp Suite ;
- configuration du proxy manuel dans Android ;
- lancement de l’application cible avec Frida ;
- exécution d’un script de test root ;
- analyse des fonctions SSL/TLS avec frida-trace ;
- test d’un script lié au SSL Pinning ;
- observation d’un crash de l’application pendant l’instrumentation.

---

## 12. Analyse des résultats

L’analyse montre que l’application contient plusieurs mécanismes de sécurité.

L’utilisation de Frida permet d’observer dynamiquement le comportement de l’application pendant son exécution.  
La commande frida-ps permet de confirmer que l’application est visible et accessible depuis l’environnement d’analyse.

La configuration du proxy avec Burp Suite permet de préparer l’interception du trafic réseau.  
Cependant, les applications Android modernes peuvent utiliser le SSL Pinning pour empêcher l’interception classique du trafic HTTPS.

Le test avec Frida montre que certaines fonctions SSL peuvent être ciblées, mais l’application peut également réagir par un crash lorsqu’une instrumentation native est détectée ou mal supportée.

Ce comportement confirme l’importance d’analyser progressivement chaque couche de sécurité.

---

## 13. Difficultés rencontrées

Pendant ce lab, la principale difficulté rencontrée est le crash de l’application lors du test du SSL Pinning.

Ce crash peut être lié à :

- une incompatibilité du script utilisé ;
- une mauvaise adaptation du script à l’application cible ;
- une protection intégrée dans l’application ;
- une architecture différente entre le script et l’émulateur ;
- une erreur dans l’appel d’une fonction native.

Cette difficulté montre que l’analyse dynamique demande des tests progressifs et une bonne compréhension de l’environnement Android.

---

## 14. Conclusion

Ce lab m’a permis de comprendre les étapes principales de l’analyse dynamique Android.

J’ai appris à préparer un environnement avec ADB et Frida, à configurer Burp Suite comme proxy, à configurer le proxy Android, à lancer une application sous Frida et à analyser les fonctions liées au SSL/TLS.

Le test du SSL Pinning a montré que certaines protections peuvent provoquer un crash de l’application lorsqu’elles détectent une instrumentation ou lorsqu’un hook n’est pas compatible.

Ce lab constitue une bonne introduction à l’analyse dynamique des applications Android et à la compréhension des mécanismes de sécurité liés au trafic HTTPS.

---


Lab réalisé dans le cadre du cours :

**Sécurité des applications mobiles**
