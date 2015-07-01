---
layout: post
title: API Rest - Le bien, le mal et le reste
date: 2014-12-23 22:33:29.000000000 -05:00
---
Ce document reprend ce que l'on peut trouver de mieux au niveau des bonnes pratiques et des retours d'expérience de différentes grandes compagnies. Certains points sont sujets à polémiques, je les préciserai quand nécessaire.

## Gestion des versions de l'API

Le premier point fondamental concerne les changements que l'on ferait à une API en exploitation ; En modifiant par exemple une réponse serveur, on pourrait briser la rétro-compatibilité. Si Paypal changeait sa manière de recevoir des paiements, ce sont des milliers de commerçants qui ne pourraient plus faire de vente...

On peut définir deux catégories de changement sur une API : Les changements structurels et les changements fonctionnels. 

### Changements structurels

Ce sont les changements qui affectent le fonctionnement de l'API. Cela peut affecter la manière dont l'API reçoit et renvoie des messages. Cela inclue les changements sur la structure d'URL, la pagination, les erreurs, l'encodage, etc.

Ce sont les changements les plus difficiles à faire et, dans l'idéal, il faudrait avoir pensé à un système de versionnage et incrémenter la version courante. Il faut aussi, en amont à sa création, bien penser à la structure de l'API.

### Changements fonctionnels

Ce sont les changements qui affectent ce que fait l'API. Cela inclue l'ajout de méthodes ou d'attributs ou encore la manière dont fonctionne un paramètre ou une fonction existante.

Heureusement ce sont des changements qui peuvent être ajouté de manière incrémentale et le plus souvent sans briser la rétro-compatibilité. Si cela devait être fait, l'idéal serait de créer une nouvelle méthode ou d'incrémenter la version de l'API pour ne pas affecter les utilisateurs.

Encore une fois, un système de version doit être pensé en amont.

### Version de l'API, comment demander une version spécifique

Pour versionner une API, plusieurs méthodes et celles-ci peuvent se combiner.

La première méthode est de préfixer les URLs avec une version: http://api.invoice.com/MyAPI/VERSION/endpoint

La deuxième méthode est de gérer la version dans le header HTTP ACCEPT des appels à l'API:

~~~
Accept: application/[app].[version]+[format]
~~~

__[app]__ est le nom de votre application, exemple: vnd.github
__[version]__ est le numéro de version
__[format]__ est le format de retour, exemple json.
Lorsque les deux sont combinés, l'URL à prédominance sur le header.

Exemple:
~~~
GET /invoices
Accept: application/invoice.api.v1.0+json
~~~

Est identique à :
~~~
GET /v1.0/invoices
Accept: application/json
~~~

Mais aussi à :
~~~
GET /v1.0/invoices
Accept: application/invoice.api.v1.5+json
~~~

### Format de la version

_C'est un point de controverse et il n'existe pas de standard sur comment versionner une API_

Il faut savoir qu'il n'y a pas de standard sur le nommage d'une version, et on trouve plusieurs manière de faire :

* Mettre la date du changement (YYYY-mm-dd)
* Mettre v+numéro de version majeur (v1)
* Mettre v+numéro de version majeur et mineur (v1.0)
* Mettre le numéro de version majeur ou majeur et mineur (1 ou 1.0)

Par expérience, je préfère utiliser un numéro de version au lieu d'une date, et je préfère avec une version majeur et mineur pour permettre de faire des changements sans devoir changer la version majeure de l'API, qui le plus souvent effraie et freine à l'intégration de la nouvelle version.

# Faire une requête à l'API

## Verbe à utiliser

<table>
	<tr>
		<th>HTTP Verb</th>
        <th>POST</th>
        <th>GET</th>
        <th>PUT</th>
        <th>PATCH</th>
        <th>DELETE</th>
    </tr><tr>
    	<td>CRUD operation</td>
        <td>Create</td>
        <td>Read</td>
        <td>Update</td>
        <td>Update</td>
        <td>Delete</td>
    </tr><tr>
    	<td>/entités</td>
        <td>Créé une nouvelle entité</td>
        <td>Retourne la liste des entités</td>
        <td>Update en batch</td>
        <td>Erreur</td>
        <td>Supprime toutes les entités</td>
    </tr><tr>
    	<td>/entités/id</td>
        <td>Erreur</td>
        <td>Retourne une entité s'il existe</td>
        <td>Remplace l'entité par la nouvelle, si l'entité n'existe pas, il la créé si possible</td>
        <td>Met à jour partiellement l'entité</td>
        <td>Supprime l'entité</td>
    </tr>
</table>
					
_L'utilisation du PUT et du PATCH est très controversé, mais si l'on se fit à la RFC du protocol HTTP 1.0, le PUT remplace une ressource par la nouvelle définition, et si celle-ci n'existe pas, il tente de la créer._

_Il est souvent admis que si la ressource n'existe pas, il est préférable de renvoyer une erreur et de laisser la création uniquement au POST._

_Il convient souvent de ne pas autoriser le DELETE sur l'ensemble des entités, mais cela relève plutôt de la logique applicative que d'une norme._

## Formats d'URL

En REST, les URLs réfèrent à des entités et non à des actions. Les actions sont définies par le VERBE HTTP utilisé (voir point précédent).

Si on a besoin de filtrer une collection, on utilise des paramètres dans l'url (exemple: /invoices?date=2014-12-01). On pourrait aussi filter les attributs d'une entité.

Si une entité est reliée à une autre, elle devra plutôt retourner les références (URL) vers les objets au lieu de les inclure dans la réponse (model hypermedia, ou chaque ressource a une URL unique propre - la base du protocol HTTP) :

~~~
GET /invoices/1234
{
    [...]
    "amountDue": 123.25,
    "href": "https://api.invoice.com/1.0/invoices/1234",
    "accounts": {
        "href": "https://api.invoice.com/1.0/accounts/crakmedia",
    }
}
~~~

Au niveau du formatage du href, on peut mettre cela dans une balise meta qui va regrouper le href mais aussi un mediaType.
 
# Réponse d'une API

## Code de retour

<table>
	<tr>
        <th>HTTP Code</th>
        <th>Nom</th>
        <th>Explication</th>
    </tr>
    <tr>
    	<td>200</td>
        <td>OK</td>
        <td>Réponse lors d'une réussite sur un GET, PUT, PATCH</td>
    </tr>
  	<tr>
        <td>201</td>
        <td>Created</td>
        <td>Lors d'une réussite d'un POST ou d'un PUT qui créé une ressource</td>
    </tr>
    <tr>
        <td>400</td>
        <td>Bad Request</td>
        <td>Lorsque la requête est mal formé:
le JSON fournit est mal formatté
le JSON est valide mais la structure du document ne correspond pas à la structure attendu</td>
    </tr>
    <tr>
        <td>401</td>
        <td>Unauthorized</td>
        <td>Lorsqu'une requête arrive non authentifié</td>
    </tr>
	<tr>
        <td>402</td>
        <td>Request Failed</td>
        <td>Pour pas mal toutes les erreurs client que l'on en sait pas classifié</td>
    </tr>
	<tr>
        <td>403</td>
        <td>Forbidden</td>
        <td>Lorsqu'une requête authentifié ne possède pas les droits pour accéder à la ressource demandé</td>
    </tr>
	<tr>
        <td>404</td>
        <td>Not Found</td>
        <td>L'entité ou la collection n'existe pas</td>
    </tr>
	<tr>
        <td>405</td>
        <td>Method Not Allowed</td>
        <td>Lorsque une requête HTTP est demandé mais le verbe utilisé n'est pas autorisé pour l'utilisateur authentifié</td>
    </tr>
	<tr>
        <td>422</td>
        <td>Unprocessable Entity</td>
        <td>Lorsque la validation du model fail pour certaines valeurs (ex: nom trop long)
Lorsque la ressource est lié à une autre dans un état invalide</td>
    </tr>
	<tr>
        <td>429</td>
        <td>Too Many Request</td>
        <td>Lorsqu'un requête est rejeté à cause d'un rate limiting</td>
    </tr>
  
	<tr>
        <td>5xx</td>
        <td>Erreurs serveurs</td>
        <td>Dès lors que c'est une erreur en dehors du scope de l'utilisateur (problème de connections à une base de donnée, etc)</td>
    </tr>
</table>

## Type de réponse

### Erreurs

Les erreurs doivent toujours être retourné avec un code HTTP correspondant au type d'erreur. Une erreur ne doit jamais être retourné avec un code 2xx.

Dans le message d'une erreur, on doit retrouvé le message et l'erreur ou les erreurs de la page. Une erreur peut très bien en encapsuler d'autre.

~~~
HTTP/1.1 422 Unprocessable Entity
 
{
  "code": 422,
  "message": "Invalid value '-1' for amountDue. Value must be within the range: [1, 1000]",
  "errors": [
   {
    "reason": "invalidParameter",
    "message": "Invalid value '-1' for amountDue. Value must be within the range: [1, 1000]",
    "resource": "invoice",
    "field": "amountDue"
   }
  ]
}
~~~
 
### Contenue

La réponse peut être en plusieurs format, pour un format précis, le header Accept doit être utilisé. On peut aussi utiliser l'extention à la fin de l'URL qui aura précédence sur le header, par contre l'utilisation du header est préférable.

Le contenue doit être paginé et avoir un nombre d'élément par défaut. Le nombre d'item retourné doit être paramétrable par contre. Dans la réponse, il faut mettre les liens vers les autres pages.

La réponse doit préférablement être en JSON pour sa simplicité, flexibilité et son support dans tous les principaux langages de programmation.

Les attributs de la réponse doivent être en camel case, JS dans JSON = JavaScript et les underscore ne sont pas conventionnel en JS.

Les dates/time/timestamp devrait suivre la norme ISO 8601 (exemple: "2014-12-22T16:01:31.343Z") et utiliser l'UTC.

~~~
GET /invoices?page=2
Status: 200 OK
Vary: Accept, Authorization, Cookie
X-RateLimit-Limit: 5000
X-RateLimit-Remaining: 4996
X-RateLimit-Reset: 1372700873
 
{
    // meta
    "apiVersion": "1.0",
    "method": "VERB.entities[.ID]",
    "params": {
        // params
    },
    "links": [
        {"href":"https://api.invoice.com/entities?page=3", "rel": "next"},
        {"href":"https://api.invoice.com/entities?page=1", "rel": "prev"}
    ],
    // data
    "data": {
        "items": []
    }
}
~~~

# Source
[GoCardless - HTTP API design](https://github.com/gocardless/http-api-design/blob/master/README.md)
[Google Analytics API v3 - Errors](https://developers.google.com/analytics/devguides/reporting/core/v3/coreErrors?hl=FR)
[Google analytics API v3 - ](https://developers.google.com/analytics/devguides/reporting/core/v3/reference?hl=FR)
[mailjet API](https://www.mailjet.com/docs/api/message/campaigns)
[Atlassian REST API](https://developer.atlassian.com/docs/atlassian-platform-common-components/rest-api-development/atlassian-rest-api-design-guidelines-version-1)
[Github API](https://developer.github.com/v3/)
[HTTP/1.0 RFC](http://www.w3.org/Protocols/rfc1945/rfc1945.txt)

