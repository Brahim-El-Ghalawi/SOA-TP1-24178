# TP1 — SOAP et WSDL — Réponses

## Étape 1

1. `@WebService` sur `EmpruntService` en fait un service web. `@WebFault` sur `LivreIndisponibleException` en fait une erreur déclarée dans le contrat (elle apparaît comme `fault` dans le WSDL).
2. JAXB instancie les objets par réflexion lors de la (dé)sérialisation XML : il crée l'objet avec le constructeur sans argument, puis remplit les champs. Sans constructeur vide, l'instanciation échoue.

## Étape 2

Fichiers : `emprunt-codefirst/xml/wsdl-codefirst.xml`, `schema-codefirst.xsd`, `00-etape2-requete.xml`, `00-etape2-reponse.xml`.

1. **URL et opérations.**
   - URL : `http://localhost:8080/bib/EmpruntService` (`<soap:address location=...>`, service `EmpruntService`, port `EmpruntPort`).
   - `emprunterLivre` : entrée `tns:emprunterLivre` (idAdherent, isbn), sortie `tns:emprunterLivreResponse` (emprunt : numero, dateRetour), fault `tns:LivreIndisponibleException`.
   - `rendreLivre` : entrée `tns:rendreLivre` (numeroEmprunt), sortie `tns:rendreLivreResponse` (penalite : `xs:decimal`).

2. **Style / use / version SOAP.** Le style est `document` et le use `literal`, ce qui correspond au mode document/literal wrapped : un seul part `parameters` par message, et l'élément porte le nom de l'opération. La version est SOAP 1.1. La preuve se trouve dans le binding : le préfixe `soap` y vaut `http://schemas.xmlsoap.org/wsdl/soap/`, l'espace de noms de SOAP 1.1. SOAP 1.2 utiliserait `http://schemas.xmlsoap.org/wsdl/soap12/`.
   ```xml
   <soap:binding transport="http://schemas.xmlsoap.org/soap/http" style="document"/>
   <soap:body use="literal"/>
   ```
   La réponse le confirme : son enveloppe utilise `http://schemas.xmlsoap.org/soap/envelope/` (SOAP 1.1) avec `Content-type: text/xml`.

3. **Types XSD.** `isbn` et `dateRetour` sont tous les deux en `xs:string`. La règle de format de l'ISBN n'apparaît **pas** dans le XSD : il n'y a ni `xs:pattern` ni `xs:restriction`. Le XSD ne contient que (ligne 22) :
   ```xml
   <xs:element name="isbn" type="xs:string" minOccurs="0"/>
   ```
   Et pour dateRetour (ligne 33) :
   ```xml
   <xs:element name="dateRetour" type="xs:string" minOccurs="0"/>
   ```
   En code-first, le contrat ne publie que le type Java (`String`). Les règles métier restent dans le code. De plus, tous les champs sont optionnels (`minOccurs="0"`).

4. **Préfixes.** `idAdherent` et `isbn` n'ont **pas** de préfixe, alors que `bib:emprunterLivre` en a un. La balise `<xs:schema>` (ligne 3) ne déclare pas d'`elementFormDefault`, donc la valeur par défaut `unqualified` s'applique. Seuls les éléments globaux comme `emprunterLivre` appartiennent au `targetNamespace` `http://supnum.mr/bib`. Les éléments locaux déclarés dans un `complexType` (`idAdherent`, `isbn`) sont sans espace de noms. La réponse suit la même règle : `ns2:emprunterLivreResponse` est préfixé, `<emprunt><numero><dateRetour>` ne le sont pas.

5. **Couches dans la requête brute (`curl -v`).**
   - **HTTP** : c'est le transport. On le voit dans la ligne de requête `POST /bib/EmpruntService HTTP/1.1` et dans les en-têtes `Host`, `Content-Type: text/xml; charset=utf-8`, `SOAPAction: ""`, `Content-Length: 348`. La réponse est `HTTP/1.1 200 OK`. HTTP achemine le message vers l'URL du service.
   - **XML** : c'est le format du corps HTTP (prologue `<?xml version="1.0" encoding="UTF-8"?>`, espaces de noms). Il rend le message structuré et vérifiable (`xmllint`).
   - **SOAP** : c'est la structure du message. On le voit dans `soapenv:Envelope` (espace de noms SOAP 1.1), `soapenv:Header` (vide, réservé aux méta-données comme la sécurité) et `soapenv:Body`, qui contient la charge utile `bib:emprunterLivre`. SOAP fixe le protocole d'échange (enveloppe, en-têtes, faults).
   - **WSDL** : c'est le contrat lisible par machine. Il donne l'adresse (`service`), les opérations et leurs messages (`portType`), le format sur le fil (`binding` : SOAP 1.1, document/literal, `soapAction=""`) et les types (XSD importé). Il permet de générer clients et requêtes, par exemple avec SoapUI ou wsimport.

### Résumé

- Le serveur répond : le WSDL et le XSD (`?xsd=1`, un seul schéma importé) ont été récupérés, et `emprunterLivre` renvoie `HTTP/1.1 200 OK` avec l'emprunt `E-2026-0812` et la date de retour `2026-12-09`.
- Le binding est SOAP 1.1 en document/literal wrapped, avec `soapAction=""` et une fault déclarée `LivreIndisponibleException` sur `emprunterLivre` uniquement.
- Écarts à signaler : `dateRetour` est en `xs:string` et non en `xs:date`, la règle de format de l'ISBN est absente du XSD, et tous les éléments sont `minOccurs="0"`. Le contrat est donc moins strict que les règles métier.
- `elementFormDefault` est absent, donc `unqualified` : les éléments enfants ne sont pas préfixés, contrairement à l'élément racine du Body.

### Vérification dans SoapUI

- Requête envoyée depuis SoapUI : ...
- Réponse reçue : ...
- Constat : identique à curl (oui/non)
