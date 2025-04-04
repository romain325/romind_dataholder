# API Interactive  

## Introduction  

Le design d’API repose sur plusieurs philosophies. L’objectif n’est pas de déterminer laquelle est la meilleure, mais plutôt de proposer des solutions adaptées à des problématiques spécifiques.  
L’une de ces approches est celle des **APIs interactives**.  

Par **API interactive**, on entend la capacité de naviguer à travers l’API et d’y effectuer des actions sans nécessairement passer par une interface graphique.  
Dans cette vision, le **front** est simplement une version **"human-friendly"** de l’API. Cela signifie que la logique métier est directement intégrée à l’API.  

En **Java**, adopter cette approche serait une mauvaise pratique, car cela lierait l’ensemble du comportement aux données, entraînant un couplage fort et l’absence d’inversion de dépendance.  
Cependant, une API est un **contrat**, un ensemble de règles permettant à un utilisateur d’exécuter un nombre fini d’actions. Il est donc possible d’y associer une logique métier pour lui donner davantage de **sémantique**.  

## Problématique  

Selon le type d’API que l’on souhaite écrire, les besoins peuvent varier.  
Parfois, nous avons simplement besoin de **rendre des données**, auquel cas un **CRUD** classique est la solution la plus adaptée.  

Cependant, dans d’autres cas, nous avons besoin d’exécuter des **actions concrètes**.  
La méthode la plus courante consiste alors à proposer une API **CRUD**, puis à relier ces opérations via un **frontend**.  

Mais pourquoi ne pas effectuer cette liaison directement dans le **back-end** ?  
Ainsi, le **front-end** n’aurait plus à interpréter la logique métier et deviendrait simplement une **représentation visuelle** du back.  

## Une solution parmi tant d'autres  

Cette philosophie, bien que controversée, s'inscrit dans une tendance actuelle du développement logiciel.  
Elle est en partie une réponse à la complexification croissante des stacks front-end.  

Des acteurs comme **Carson Gross** ont largement contribué à cette réflexion, notamment à travers l’ouvrage **Hypermedia Systems**.  

## Avantages  

- Une **distinction plus claire** entre le **back-end** et le **front-end** : chaque expert se concentre sur son domaine, assurant ainsi une meilleure spécialisation.  
- Un **design plus simple et plus compréhensible** pour les acteurs externes et les parties prenantes.  

## Inconvénients  

- Un design **totalement différent** des pratiques habituelles.  
- Toute modification doit être effectuée sur le **back-end**, nécessitant un **redéploiement** potentiellement plus lourd que l'actualisation d’un front.  
- Moins adapté aux **frameworks modernes**, qui sont conçus pour assurer la liaison entre les **CRUDs**. Ces frameworks perdent alors une grande partie de leur intérêt et peuvent être perçus comme inutiles (bloat).  

## Cas concret  

Pour illustrer cette approche, prenons un **cas d’usage** et voyons comment une API plus interactive peut alléger la charge du **front-end** en transférant la complexité vers le **back-end**.  

Nous avons déjà utilisé l’exemple d’une **transaction** pour expliquer ce principe, mais explorons maintenant un **cas métier plus spécifique**.  

> Use case
> Je suis un utilisateur sur un site marchand et je veux pouvoir faire mon panier, le valider, gerer le paiement puis la livraison et enfin cloturer

On va donc commencer par une ressources basket avec un get tel que:

> GET /basket/123
```json
{
    "@id": "123",
    "items": [
        {
            "@id": "1",
            "name": "Pomme",
            "quantity": 1
        },
        {
            "@id": "2",
            "name": "Poire",
            "quantity": 1
        },
        {
            "@id": "3",
            "name": "Banane",
            "quantity": 6
        },
    ],
    "@actions": {
        "reserve": "http://mon-site/api/v1/reservation",
        "@cancel": "http://mon-site/api/v1/basket/123"
    }
}
```

On va donc pouvoir faire une reservation 

> POST /reservation -- basket=123
```json
{
    "@id": "1",
    "basketId": "123",
    "expirationDate": "17654837649873",
    "canBeValidated": false,
    "@actions": {
        "delivery": "http://mon-site/api/v1/reservation/1/delivery",
        "payment": "http://mon-site/api/v1/reservation/1/payment",
        "@cancel": "http://mon-site/api/v1/reservation/1"
    }
}
```

Maintenant qu'on a a reservé notre panier et son contenu on a plus qu'a remplir les informations jusqua cque notre panier soit valide
Je vais donc passer a travers les actions pour préparer ma livraison

> POST /delivery
```json
{
    "reservationId": "1",
    "address": "132 Avenue Roger Salengro, 69100 Villeurbanne",
    "contact": {
        "tel": "+33666666666",
        "mail": "monmail@gmail.com"
    }
}
```

Si on decide de get la reservation pour checker l'avancement on obtient:

> GET /reservation/1
```json
{
    "@id": "1",
    "basketId": "123",
    "expirationDate": "17654837649873",
    "canBeValidated": false,
    "@actions": {
        "payment": "http://mon-site/api/v1/reservation/1/payment",
        "@cancel": "http://mon-site/api/v1/reservation/1"
    }
}
```

On peut a present fill up le payement avec nos informations:

> POST /payment
```json
{
    "reservationId": "1",
    "method": "VISA",
    "code": "1234 5678 9123 4567",
    "expiration": "10/27",
    "key": "123"
}
```

On retourne sur notre reservation pour checker l'avancement on obtient:

> GET /reservation/1
```json
{
    "@id": "1",
    "basketId": "123",
    "expirationDate": "17654837649873",
    "canBeValidated": true,
    "@actions": {
        "validate": "http://mon-site/api/v1/reservation/1",
        "@cancel": "http://mon-site/api/v1/reservation/1"
    }
}
```

On peut a present valider

> PUT /reservation/1
```json
{
    "@id": "1"
}
```

et on obtient en réponse:

```json
{
    "@id": "1",
    "status": "done",
    "orderId": "14928",
    "order": "http://mon-site/api/v1/order/14928",
    "shippingStatus": "http://site-livreur/order/14928/status",
    "ticket": "http://mon-site/api/v1/order/14928/ticket"
}
```

On a donc un flow complet ou l'entierté de la dynamique est géré par le back avec une vision très métier d'un flow
On peut par la suite utiliser des outil tel que arazzo pour modéliser ce flow

Et pour finir notre front n'a quasi pas de logique a avoir, simplement afficher les résultats de ces appels et faire avancer le tout
Evidemment ces fluxs ne sont pas parfait mais permette de donner une vision technique métier et une compréhension de ce qu'il se passe à l'utilisateur sans avoir besoin d'un front end pour expliquer cqui se passe
On remets la connaissance métier au moteur de fond
