# Étape 2 : ce que SoapUI doit montrer

Référence : requête et réponse réelles obtenues avec `curl -i -v` (`00-etape2-requete.xml`, `00-etape2-reponse.xml`).

## 1. Création du projet

- File > New SOAP Project > Initial WSDL :
  `http://localhost:8080/bib/EmpruntService?wsdl`
- Binding attendu dans l'arbre : `EmpruntPortBinding`, avec deux opérations : `emprunterLivre` et `rendreLivre`.
- Ouvrir `emprunterLivre` > `Request 1`. SoapUI génère des `?` et des commentaires `<!--Optional:-->`, parce que les éléments sont en `minOccurs="0"`. Remplacer les `?` :
  - `idAdherent` = `A-117`
  - `isbn` = `978-1-234-5678-9`

## 2. Onglet Raw : requête

Ce que curl a envoyé :

```
POST /bib/EmpruntService HTTP/1.1
Host: localhost:8080
Content-Type: text/xml; charset=utf-8
SOAPAction: ""
Content-Length: 348

<?xml version="1.0" encoding="UTF-8"?>
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:bib="http://supnum.mr/bib">
  <soapenv:Header/>
  <soapenv:Body>
    <bib:emprunterLivre>
      <idAdherent>A-117</idAdherent>
      <isbn>978-1-234-5678-9</isbn>
    </bib:emprunterLivre>
  </soapenv:Body>
</soapenv:Envelope>
```

Il faut retrouver à l'identique dans SoapUI :
- la méthode `POST` vers `/bib/EmpruntService` ;
- l'en-tête `Content-Type: text/xml` (signe de SOAP 1.1) ;
- l'en-tête `SOAPAction: ""` ;
- l'enveloppe `http://schemas.xmlsoap.org/soap/envelope/` ;
- `emprunterLivre` préfixé dans `http://supnum.mr/bib`, avec `idAdherent` et `isbn` **sans** préfixe.

Différences normales, qui viennent du client et non du service (je ne les ai pas observées moi-même, à confirmer) :
- la ligne de requête peut afficher l'URL complète (`POST http://localhost:8080/bib/EmpruntService HTTP/1.1`) ;
- en-têtes `User-Agent: Apache-HttpClient/...`, `Accept-Encoding: gzip,deflate`, `Connection: Keep-Alive` ;
- `charset=UTF-8` en majuscules ;
- un préfixe d'espace de noms qui peut porter un autre nom, et un `Content-Length` différent selon les espaces et les commentaires.

## 3. Onglet Raw : réponse (code 200)

Ce que curl a reçu :

```
HTTP/1.1 200 OK
Transfer-encoding: chunked
Content-type: text/xml; charset=utf-8

<?xml version='1.0' encoding='UTF-8'?><S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/"><S:Body><ns2:emprunterLivreResponse xmlns:ns2="http://supnum.mr/bib"><emprunt><numero>E-2026-0812</numero><dateRetour>2026-12-09</dateRetour></emprunt></ns2:emprunterLivreResponse></S:Body></S:Envelope>
```

La valeur de `numero` et la date dépendent de l'état du serveur et du jour.

## 4. Attention : l'état est en mémoire

Si on renvoie le même ISBN sans redémarrer le serveur, la réponse n'est plus 200. Voici ce que curl a obtenu en rejouant la requête :

```
HTTP/1.1 500 Internal Server Error
... <S:Fault><faultcode>S:Server</faultcode><faultstring>Livre indisponible</faultstring>
<detail><ns2:LivreIndisponible xmlns:ns2="http://supnum.mr/bib"><isbn>978-1-234-5678-9</isbn><retourPrevu>2026-12-09</retourPrevu></ns2:LivreIndisponible></detail></S:Fault> ...
```

Le serveur a été redémarré après ce test : le premier envoi depuis SoapUI doit donner 200.
