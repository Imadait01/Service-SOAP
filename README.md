# Service_SOAP

Projet Maven Java (Apache CXF) exposant un service SOAP simple et un endpoint sécurisé (UsernameToken WSS4J).

## Sommaire
1. Présentation
2. Arborescence / architecture
3. Prérequis
4. Compilation & exécution
5. Endpoints disponibles
6. Client d'exemple
7. Sécurité (WS‑Security)
8. Dépendances importantes / remarque `pom.xml`
9. Débogage & dépannage rapide
10. Fichiers importants

---

## 1. Présentation
Ce projet illustre :
- Publication d'un service JAX‑WS avec Apache CXF.
- Consommation via l'API JAX‑WS standard.
- Protection SOAP par WS‑Security (UsernameToken) via WSS4J.

## 2. Arborescence / architecture
Structure principale du projet (extrait) :

```
Service_SOAP/
├─ pom.xml
├─ README.md
└─ src/
   ├─ main/
   │  ├─ java/
   │  │  └─ com/
   │  │     └─ acme/
   │  │        └─ cxf/
   │  │           ├─ Server.java             # Endpoint non sécurisé (publie ?wsdl)
   │  │           ├─ SecureServer.java       # Endpoint sécurisé (WSS UsernameToken)
   │  │           ├─ api/
   │  │           │  └─ HelloService.java    # Interface JAX‑WS (contrat)
   │  │           ├─ impl/
   │  │           │  └─ HelloServiceImpl.java# Implémentation du service
   │  │           ├─ client/
   │  │           │  └─ ClientDemo.java      # Exemple de consommation JAX‑WS
   │  │           ├─ model/
   │  │           │  └─ Person.java          # POJO exposé par le service
   │  │           └─ security/
   │  │              └─ UTPasswordCallback.java # Callback WSS4J
   │  └─ resources/
   └─ test/
      └─ java/
```

Archi logique :
- Serveur JAX‑WS (Apache CXF) exposant des endpoints HTTP.
- Intercepteurs WSS4J pour validation UsernameToken (IN) et éventuellement signature/chiffrement (OUT).
- Les POJOs (ex: `Person`) sont traduits en types XSD par JAXB.

## 3. Prérequis
- JDK 17
- Maven 3.6+
- Port 8080 libre
- (Optionnel) IDE : IntelliJ IDEA, Eclipse

## 4. Compilation & exécution
1. Compiler le projet :

```bash
mvn clean package
```

2. Exécuter l'un des serveurs (depuis l'IDE ou en ligne de commande via `java -cp target/classes;...`):
- `com.acme.cxf.Server` → service non sécurisé
- `com.acme.cxf.SecureServer` → service sécurisé (WSS UsernameToken)

3. Vérifier le WSDL dans un navigateur ou SoapUI :

- Non sécurisé : `http://localhost:8080/services/hello?wsdl`
- Sécurisé : `http://localhost:8080/services/hello-secure?wsdl`

> Remarque : lancer `SecureServer` expose l'endpoint sécurisé mais nécessite que le client fournisse un UsernameToken valide (voir section sécurité).

## 5. Endpoints disponibles
- WSDL non sécurisé : `http://localhost:8080/services/hello?wsdl`
- WSDL sécurisé : `http://localhost:8080/services/hello-secure?wsdl`

L'interface JAX‑WS (`com.acme.cxf.api.HelloService`) définit les opérations exposées ; CXF génère le WSDL à partir des annotations (`@WebService`, `@WebMethod`, etc.).

## 6. Client d'exemple
Fichier : `src/main/java/com/acme/cxf/client/ClientDemo.java`

Exemple d'utilisation :
- Créez un `Service` à partir de l'URL WSDL et du `QName` du service, puis appelez `getPort(HelloService.class)`.

Extrait :
```java
URL wsdl = new URL("http://localhost:8080/services/hello?wsdl");
QName qname = new QName("http://api.cxf.acme.com/", "HelloService");
Service svc = Service.create(wsdl, qname);
HelloService port = svc.getPort(HelloService.class);
```

## 7. Sécurité (WS‑Security)
- Dépendances : WSS4J (`wss4j-ws-security-common`) et `cxf-rt-ws-security`.
- Callback de mot de passe : `com.acme.cxf.security.UTPasswordCallback`.
- Identifiants de démonstration :
  - utilisateur : `student`
  - mot de passe : `secret123`

Test avec SoapUI :
1. Importer `http://localhost:8080/services/hello-secure?wsdl`.
2. Configurer WS‑Security → UsernameToken (Password Type = Text) avec `student` / `secret123`.
3. Envoyer la requête : sans token le serveur renverra une Fault.

Bonnes pratiques : en production, préférez `PasswordDigest` + HTTPS, et ajoutez signature & chiffrement pour protéger l'intégrité et la confidentialité.

## 8. Dépendances importantes / remarque `pom.xml`
- Apache CXF: version définie dans la propriété `${cxf.version}`.
- WSS4J: `wss4j-ws-security-common` (ex. 3.0.0).
- JAXB (Jakarta) : `jakarta.xml.bind-api` + `jaxb-runtime` (pour Java 11+).

Remarque : le fichier `pom.xml` du projet contient actuellement une duplication de la dépendance `cxf-rt-ws-security` (deux entrées identiques) — cela fonctionne mais est redondant. Vous pouvez supprimer l'une des deux occurrences pour clarifier le POM.

## 9. Débogage & dépannage rapide
- `Address already in use` → vérifier si un autre processus utilise le port 8080 (`netstat -ano | findstr 8080` sous Windows).
- Erreurs JAXB → vérifier les versions `jakarta.xml.bind-api` et `jaxb-runtime`.
- Afficher l'arbre des dépendances :

```bash
mvn dependency:tree
```

- Activer les logs CXF / WSS4J (SLF4J / Logback) pour diagnostiquer les échanges SOAP et la validation WS‑Security.

## 10. Fichiers importants
- `src/main/java/com/acme/cxf/Server.java`
- `src/main/java/com/acme/cxf/SecureServer.java`
- `src/main/java/com/acme/cxf/api/HelloService.java`
- `src/main/java/com/acme/cxf/impl/HelloServiceImpl.java`
- `src/main/java/com/acme/cxf/client/ClientDemo.java`
- `src/main/java/com/acme/cxf/security/UTPasswordCallback.java`
- `pom.xml`

---

Annexe — Exécution rapide

1. Compiler :

```bash
mvn clean package
```

2. Lancer `com.acme.cxf.SecureServer` (ou `Server`).
3. Tester les WSDL et les appels via SoapUI ou un navigateur.

<img width="1892" height="1042" alt="Screenshot 2025-11-11 145025" src="https://github.com/user-attachments/assets/988b4f8b-8941-4b22-81ab-4c367c868245" />
<img width="1897" height="1028" alt="Screenshot 2025-11-11 145051" src="https://github.com/user-attachments/assets/ba715f71-0b61-4d1e-8188-6aa336faf3ce" />
<img width="1831" height="972" alt="Screenshot 2025-11-11 163448" src="https://github.com/user-attachments/assets/12a52bcb-6623-40cc-9e59-6e8f91acff37" />



