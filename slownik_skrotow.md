# Słowniczek Skrótów i Pojęć – Jakarta EE

Kompletny skorowidz wszystkich akronimów, skrótów i pojęć występujących w materiale wykładowym platformy Jakarta EE. Opracowany z myślą o szybkim powtórzeniu przed kolokwium i egzaminem teoretycznym.

---

## SPIS KATEGORII:
1. [Platforma, Architektura i Wzorce Ogólne](#1-platforma-architektura-i-wzorce-ogólne)
2. [Warstwa Webowa i Servlety](#2-warstwa-webowa-i-servlety)
3. [CDI i Wstrzykiwanie Zależności](#3-cdi-i-wstrzykiwanie-zależności)
4. [Trwałość Danych i JPA](#4-trwałość-danych-i-jpa)
5. [Jakarta Faces (JSF)](#5-jakarta-faces-jsf)
6. [Usługi Sieciowe i REST (JAX-RS)](#6-usługi-sieciowe-i-rest-jax-rs)
7. [Komponenty Biznesowe EJB](#7-komponenty-biznesowe-ejb)
8. [Bezpieczeństwo, Uwierzytelnianie i Autoryzacja](#8-bezpieczeństwo-uwierzytelnianie-i-autoryzacja)
9. [Indeks Alfabetyczny Skrótów](#9-indeks-alfabetyczny-skrótów)

---

## 1. Platforma, Architektura i Wzorce Ogólne

- **J2EE (Java 2 Platform, Enterprise Edition)**
  - Pierwotna nazwa platformy korporacyjnej Javy (wersje 1.2–1.4) rozwijanej przez firmę Sun Microsystems.
- **Java EE (Java Platform, Enterprise Edition)**
  - Druga generacja platformy (wersje 5–8) rozwijana pod skrzydłami firmy Oracle. Wprowadziła adnotacje, POJO i uproszczony model komponentowy.
- **Jakarta EE (Jakarta Enterprise Edition)**
  - Aktualna nazwa platformy (od wersji 8 wzwyż) po przekazaniu praw i rozwoju do fundacji Eclipse Foundation. Od wersji 9 charakteryzuje się zmianą przestrzeni pakietów z `javax.*` na `jakarta.*`.
- **Java SE (Java Platform, Standard Edition)**
  - Bazowa platforma języka Java dostarczająca podstawowe biblioteki uruchomieniowe (kolekcje, wejście/wyjście, wielowątkowość), którą Jakarta EE rozszerza o technologie korporacyjne.
- **POJO (Plain Old Java Object)**
  - Zwykły, prosty obiekt Javy, niezawierający dziedziczenia po specyficznych klasach frameworka ani implementacji ciężkich interfejsów systemowych.
- **DAO (Data Access Object)**
  - Wzorzec projektowy izolujący warstwę logiki biznesowej od operacji na bazie danych.
- **DTO (Data Transfer Object)**
  - Prosty obiekt służący wyłącznie do przesyłania danych między procesami lub warstwami systemu, pozbawiony logiki biznesowej.
- **EIS (Enterprise Information System)**
  - Poziom systemów informatycznych przedsiębiorstwa (systemy bazodanowe, ERP, CRM, systemy transakcyjne mainframe), z którymi integruje się aplikacja enterprise.
- **ERP (Enterprise Resource Planning)**
  - Zaawansowane systemy planowania zasobów przedsiębiorstwa integrujące procesy księgowe, magazynowe, kadrowe i produkcyjne.
- **EAR (Enterprise Archive – `.ear`)**
  - Archiwum ZIP najwyższego rzędu w Jakarta EE, agregujące moduły webowe (WAR), moduły biznesowe (EJB-JAR) i adaptery zasobów (RAR) w jedną całość wdrażaną na serwerze.
- **WAR (Web Archive – `.war`)**
  - Ustandaryzowane archiwum aplikacji internetowej zawierające strony WWW, servlety, filtry oraz katalog `WEB-INF`.
- **JAR (Java Archive – `.jar`)**
  - Podstawowe archiwum klas skompilowanych i zasobów Javy.
- **RAR (Resource Adapter Archive – `.rar`)**
  - Archiwum adaptera zasobów zgodne ze specyfikacją JCA, służące do łączenia serwera aplikacji z zewnętrznymi systemami EIS.
- **JVM (Java Virtual Machine)**
  - Maszyna wirtualna Javy – środowisko uruchomieniowe wykonujące kod bajtowy Javy.
- **AOP (Aspect-Oriented Programming)**
  - Programowanie zorientowane aspektowo – paradygmat pozwalający na separację zagadnień przekrojowych (np. transakcje, logowanie, bezpieczeństwo) poprzez interceptory.
- **SOLID**
  - Pięć fundamentalnych zasad obiektowego projektowania oprogramowania (Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion).
- **DIP (Dependency Inversion Principle)**
  - Zasada odwrócenia zależności: moduły wysokiego poziomu nie powinny zależeć od modułów niskiego poziomu – oba powinny zależeć od abstrakcji (interfejsów).

---

## 2. Warstwa Webowa i Servlety

- **HTTP (Hypertext Transfer Protocol)**
  - Bezstanowy protokół warstwy aplikacji stanowiący fundament komunikacji w sieci WWW.
- **HTTPS (Hypertext Transfer Protocol Secure)**
  - Bezpieczna wersja protokołu HTTP szyfrowana za pomocą TLS/SSL.
- **URI (Uniform Resource Identifier)**
  - Ustandaryzowany identyfikator jednoznacznie wskazujący zasób w sieci.
- **URL (Uniform Resource Locator)**
  - Szczególna postać identyfikatora URI, określająca dodatkowo lokalizację zasobu i mechanizm dostępu (np. `https://...`).
- **MIME (Multipurpose Internet Mail Extensions)**
  - Standard formatowania i oznaczania typów przesyłanych danych w sieci (np. `text/html`, `application/json`, `multipart/form-data`).
- **SSL / TLS (Secure Sockets Layer / Transport Layer Security)**
  - Protokoły kryptograficzne zapewniające poufność i integralność przesyłanych danych w sieci komputerowej.
- **GZIP**
  - Format i algorytm bezstratnej kompresji strumieni danych, często stosowany w filtrach webowych do redukcji rozmiaru odpowiedzi HTTP.

---

## 3. CDI i Wstrzykiwanie Zależności

- **CDI (Contexts and Dependency Injection)**
  - Specyfikacja platformy Jakarta EE integrująca wstrzykiwanie zależności (DI) z obsługą cyklu życia komponentów w kontekstach webowych (żądanie, sesja, aplikacja).
- **IoC (Inversion of Control)**
  - Odwrócenie sterowania – wzorzec architektoniczny, w którym to kontener frameworka zarządza przepływem programu i cyklem życia obiektów, wywołując kod użytkownika.
- **DI (Dependency Injection)**
  - Wstrzykiwanie zależności – wzorzec projektowy będący realizacją IoC, polegający na dostarczaniu referencji do obiektów z zewnątrz zamiast ich ręcznego tworzenia.
- **JSR (Java Specification Request)**
  - Formalny dokument propozycji nowej specyfikacji lub standardu w ekosystemie Javy (np. JSR 330 definiujący standard `@Inject`).
- **EL (Expression Language)**
  - Język wyrażeń umożliwiający prosty i elegancki dostęp do właściwości beanów Javy z poziomu stron widoku (`${...}` lub `#{...}`).
- **CID (Conversation Identifier)**
  - Unikalny identyfikator długotrwałej konwersacji w zasięgu `@ConversationScoped`, doklejany do żądań HTTP w celu powiązania ich z danym stanem.

---

## 4. Trwałość Danych i JPA

- **JPA (Jakarta Persistence / dawniej Java Persistence API)**
  - Standardowa specyfikacja platformy do odwzorowania obiektowo-relacyjnego (ORM) i zarządzania relacyjnymi bazami danych.
- **ORM (Object-Relational Mapping)**
  - Technika programistyczna automatyzująca mapowanie obiektów w pamięci języka programowania na tabele i rekordy w relacyjnej bazie danych.
- **JPQL (Java Persistence Query Language)**
  - Zorientowany obiektowo język zapytań w JPA, operujący na nazwach klas encyjnych i ich polach (w odróżnieniu od SQL operującego na tabelach i kolumnach).
- **SQL (Structured Query Language)**
  - Strukturalny język zapytań służący do zarządzania danymi w relacyjnych bazach danych.
- **JDBC (Java Database Connectivity)**
  - Niskopoziomowy standard komunikacji i wykonywania instrukcji SQL z poziomu języka Java za pomocą dedykowanych sterowników bazodanowych.
- **JNDI (Java Naming and Directory Interface)**
  - Ustandaryzowana usługa katalogowa platformy, pozwalająca komponentom na wyszukiwanie zasobów (np. puli połączeń bazy danych, beanów EJB) po nazwach hierarchicznych.
- **JTA (Java Transaction API / Jakarta Transactions)**
  - Specyfikacja platformy odpowiedzialna za koordynację transakcji rozproszonych i transakcji zarządzanych przez kontener (CMT).
- **ACID (Atomicity, Consistency, Isolation, Durability)**
  - Cztery pożądane własności transakcji bazodanowej: Niepodzielność, Spójność, Izolacja oraz Trwałość.
- **2PC (Two-Phase Commit)**
  - Protokół dwufazowego zatwierdzania transakcji rozproszonych obejmujących wiele niezależnych baz danych lub węzłów sieci.
- **CMT (Container-Managed Transactions)**
  - Transakcje zarządzane automatycznie przez kontener serwera (np. w EJB lub przy `@Transactional`).
- **BMT (Bean-Managed Transactions)**
  - Transakcje zarządzane manualnie przez programistę w kodzie beana (poprzez obiekt `UserTransaction`).
- **DBMS (Database Management System)**
  - System zarządzania bazą danych (np. PostgreSQL, Oracle).
- **RDBMS (Relational Database Management System)**
  - Relacyjny system zarządzania bazą danych.
- **L1 Cache (Level 1 Cache)**
  - Pamięć podręczna pierwszego poziomu w JPA – nierozerwalnie związana z kontekstem trwałości (`EntityManager`); aktywna zawsze, nie można jej wyłączyć.
- **L2 Cache (Level 2 Cache)**
  - Pamięć podręczna drugiego poziomu w JPA – globalny, opcjonalny bufor współdzielony w skali całej fabryki `EntityManagerFactory` dla wszystkich sesji i wątków.

---

## 5. Jakarta Faces (JSF)

- **JSF (Jakarta Faces / dawniej JavaServer Faces)**
  - Komponentowy framework platformy Jakarta EE służący do budowy graficznego interfejsu użytkownika działający po stronie serwera (*server-side UI*).
- **JSP (JavaServer Pages)**
  - Starsza technologia generowania dynamicznych stron WWW oparta o znaczniki i skrawki kodu Javy (zastąpiona przez Facelets).
- **JSTL (JavaServer Pages Standard Tag Library)**
  - Zbiór standardowych bibliotek znaczników dla stron JSP/Facelets (pętle, warunki, formatowanie).
- **HTML (Hypertext Markup Language)**
  - Standardowy język znaczników używany do tworzenia dokumentów wyświetlanych w przeglądarce internetowej.
- **XHTML (Extensible Hypertext Markup Language)**
  - Dokument HTML sformatowany zgodnie z rygorystycznymi regułami XML; standard zapisu widoków Facelets w JSF.
- **XML (Extensible Markup Language)**
  - Uniwersalny, rozszerzalny język znaczników przeznaczony do reprezentowania danych i konfiguracji w ustrukturyzowany sposób.
- **CSS (Cascading Style Sheets)**
  - Kaskadowe arkusze stylów definiujące warstwę wizualną i prezentację dokumentu HTML.
- **AJAX (Asynchronous JavaScript and XML)**
  - Technika asynchronicznej wymiany danych z serwerem w tle bez konieczności przeładowywania całej strony internetowej w przeglądarce.
- **PRG (Post-Redirect-Get)**
  - Wzorzec architektoniczny eliminujący problem podwójnego zatwierdzenia formularza przy odświeżeniu strony poprzez przekierowanie z żądania POST do bezpiecznego żądania GET.
- **UI (User Interface)**
  - Interfejs użytkownika – warstwa wizualna i interaktywna aplikacji.

---

## 6. Usługi Sieciowe i REST (JAX-RS)

- **REST (Representational State Transfer)**
  - Styl architektoniczny komunikacji w systemach rozproszonych oparty o protokół HTTP, bezstanowość oraz jednoznacznie adresowalne zasoby.
- **JAX-RS (Jakarta RESTful Web Services / dawniej Java API for RESTful Web Services)**
  - Standard platformy Jakarta EE do tworzenia usług sieciowych zgodnych ze stylem REST za pomocą adnotacji.
- **JSON (JavaScript Object Notation)**
  - Lekki, tekstowy format wymiany danych oparty na parach klucz-wartość, dominujący we współczesnych API REST.
- **JSON-P (Jakarta JSON Processing)**
  - Niskopoziomowe API platformy do strumieniowego i modelowego przetwarzania dokumentów JSON.
- **JSON-B (Jakarta JSON Binding)**
  - Wysokopoziomowy standard platformy do automatycznej serializacji obiektów Javy na JSON i deserializacji JSON na obiekty.
- **CRUD (Create, Read, Update, Delete)**
  - Cztery elementarne operacje na danych, odwzorowywane w REST na metody: POST (Create), GET (Read), PUT/PATCH (Update) i DELETE (Delete).
- **POX (Plain Old XML)**
  - Zwykły, tradycyjny format XML stosowany w starszych usługach sieciowych (jako kontrast dla semantycznego REST).
- **HATEOAS (Hypermedia As The Engine Of Application State)**
  - Najwyższy poziom dojrzałości REST (Poziom 3 w modelu Richardsona); odsyłanie przez serwer hiperłączy sterujących kolejnymi możliwymi akcjami klienta.
- **WADL (Web Application Description Language)**
  - Format oparty na XML służący do formalnego opisu usług sieciowych REST (odpowiednik WSDL dla SOAP).
- **OpenAPI**
  - Powszechny, nowoczesny standard definiowania i dokumentowania interfejsów REST API w formatach JSON/YAML (dawniej znany jako Swagger).
- **RPC (Remote Procedure Call)**
  - Model komunikacji rozproszonej polegający na zdalnym wywoływaniu procedur na innym komputerze tak, jakby były wywołaniami lokalnymi.
- **SOP (Same Origin Policy)**
  - Fundamentalna reguła bezpieczeństwa w przeglądarkach internetowych, blokująca skryptom JS dostęp do zasobów pochodzących z innego źródła (inny protokół, host lub port).
- **CORS (Cross-Origin Resource Sharing)**
  - Mechanizm nagłówków HTTP pozwalający serwerowi na kontrolowane zezwolenie przeglądarce na wykonywanie zapytań z obcych domen (obchodzenie restrykcji SOP).

---

## 7. Komponenty Biznesowe EJB

- **EJB (Jakarta Enterprise Beans / dawniej Enterprise JavaBeans)**
  - Architektura komponentów biznesowych zarządzanych przez kontener serwera aplikacji, zapewniająca deklaratywne transakcje, bezpieczeństwo i pooling.
- **MDB (Message-Driven Bean)**
  - Asynchroniczny komponent EJB nasłuchujący i reagujący na komunikaty nadsyłane do systemów kolejkowych JMS.
- **JMS (Jakarta Messaging / dawniej Java Message Service)**
  - Standard platformy do asynchronicznej wymiany komunikatów pomiędzy aplikacjami za pomocą kolejek (*point-to-point*) oraz tematów (*publish-subscribe*).
- **JCA (Jakarta Connectors / Java EE Connector Architecture)**
  - Architektura ustandaryzowanych łączników (adapterów) pozwalająca serwerom aplikacji na komunikację z heterogenicznymi systemami korporacyjnymi (EIS).
- **CMC (Container-Managed Concurrency)**
  - Domyślny tryb zarządzania współbieżnością w singletonach EJB, w którym to kontener nakłada blokady `@Lock(READ)` lub `@Lock(WRITE)`.
- **BMC (Bean-Managed Concurrency)**
  - Tryb zarządzania współbieżnością w EJB, w którym to programista manualnie synchronizuje dostęp do pól (np. słowem `synchronized`).
- **RMI (Remote Method Invocation)**
  - Mechanizm wirtualnej maszyny Javy umożliwiający obiektowi wywołanie metody na obiekcie działającym w innej maszynie JVM.
- **IIOP (Internet Inter-ORB Protocol)**
  - Protokół transmisji sieciowej protokołu CORBA, historycznie używany w zdalnych wywołaniach EJB.

---

## 8. Bezpieczeństwo, Uwierzytelnianie i Autoryzacja

- **AuthN (Authentication – Uwierzytelnianie)**
  - Proces weryfikacji tożsamości podmiotu (*"Kim jesteś?"*).
- **AuthZ (Authorization – Autoryzacja)**
  - Proces weryfikacji uprawnień uwierzytelnionego podmiotu do wykonania danej operacji (*"Co wolno ci zrobić?"*).
- **RBAC (Role-Based Access Control)**
  - Model kontroli dostępu oparty o role, gdzie uprawnienia przypisuje się do ról, a role do użytkowników.
- **ACL (Access Control List)**
  - Lista kontroli dostępu definiująca uprawnienia konkretnych użytkowników lub ról bezpośrednio do danego zasobu/pliku.
- **OAuth / OAuth2 (Open Authorization 2.0)**
  - Ustandaryzowany protokół autoryzacji delegowanej, pozwalający aplikacjom na bezpieczny dostęp do zasobów użytkownika bez znajomości jego hasła.
- **PKCE (Proof Key for Code Exchange)**
  - Rozszerzenie kryptograficzne dla przepływu Authorization Code w OAuth 2.0, chroniące klientów publicznych (SPA, aplikacje mobilne) przed przechwyceniem kodu autoryzacyjnego.
- **JWT (JSON Web Token)**
  - Otwarty standard (RFC 7519) reprezentacji poświadczeń w postaci zwartego, samowystarczalnego i cyfrowo podpisanego obiektu JSON.
- **JWS (JSON Web Signature)**
  - Standard cyfrowego podpisywania tokenów JWT gwarantujący ich integralność i autentyczność.
- **JWE (JSON Web Encryption)**
  - Standard szyfrowania zawartości tokenów JWT zapewniający ich poufność (ukrycie danych ładunku).
- **IdP (Identity Provider)**
  - Zewnętrzny dostawca tożsamości (np. Keycloak, Auth0, Google, Okta) odpowiedzialny za uwierzytelnianie użytkowników w architekturze federacyjnej.
- **SSO (Single Sign-On)**
  - Mechanizm pojedynczego logowania – użytkownik loguje się raz, uzyskując dostęp do wielu niezależnych aplikacji w organizacji.
- **LDAP (Lightweight Directory Access Protocol)**
  - Protokół sieciowy służący do odpytywania i modyfikacji hierarchicznych usług katalogowych (np. Active Directory), powszechnie stosowany do przechowywania kont użytkowników i ról.
- **PBKDF2 (Password-Based Key Derivation Function 2)**
  - Standard kryptograficzny bezpiecznego haszowania haseł z użyciem soli kryptograficznej i wielotysięcznych iteracji, chroniący przed atakami słownikowymi.
- **HMAC (Hash-based Message Authentication Code)**
  - Mechanizm uwierzytelniania wiadomości z użyciem funkcji skrótu i wspólnego klucza symetrycznego (np. HS256 w JWT).
- **SHA (Secure Hash Algorithm)**
  - Rodzina kryptograficznych funkcji skrótu (np. SHA-256) przekształcających dane w ciąg o stałej długości.
- **RSA (Rivest–Shamir–Adleman)**
  - Asymetryczny algorytm kryptograficzny oparty o parę kluczy: prywatny (do podpisywania) i publiczny (do weryfikacji).
- **ECDSA (Elliptic Curve Digital Signature Algorithm)**
  - Asymetryczny algorytm podpisu cyfrowego oparty na krzywych eliptycznych, oferujący wysokie bezpieczeństwo przy mniejszej długości klucza niż RSA.
- **XSS (Cross-Site Scripting)**
  - Atak polegający na wstrzyknięciu złośliwego kodu JavaScript do strony wyświetlanej innemu użytkownikowi.
- **CSRF (Cross-Site Request Forgery)**
  - Atak polegający na wymuszeniu na przeglądarce zalogowanego użytkownika wykonania niechcianej akcji na podatnym serwerze.

---

## 9. Indeks Alfabetyczny Skrótów

| Skrót | Pełna Nazwa Angielska | Kategoria |
| :--- | :--- | :--- |
| **2PC** | Two-Phase Commit | Transakcje / Bazy danych |
| **ACID** | Atomicity, Consistency, Isolation, Durability | Bazy danych / Transakcje |
| **ACL** | Access Control List | Bezpieczeństwo |
| **AJAX** | Asynchronous JavaScript and XML | Web / Interfejs |
| **AOP** | Aspect-Oriented Programming | Architektura |
| **AuthN** | Authentication | Bezpieczeństwo |
| **AuthZ** | Authorization | Bezpieczeństwo |
| **BMC** | Bean-Managed Concurrency | EJB |
| **BMT** | Bean-Managed Transactions | Transakcje / EJB |
| **CDI** | Contexts and Dependency Injection | Komponenty / Wstrzykiwanie |
| **CID** | Conversation Identifier | CDI |
| **CMC** | Container-Managed Concurrency | EJB |
| **CMT** | Container-Managed Transactions | EJB / Transakcje |
| **CORS** | Cross-Origin Resource Sharing | Bezpieczeństwo sieciowe |
| **CRUD** | Create, Read, Update, Delete | Bazy danych / REST |
| **CSS** | Cascading Style Sheets | Web / Prezentacja |
| **DAO** | Data Access Object | Wzorce projektowe |
| **DI** | Dependency Injection | Wzorce projektowe / CDI |
| **DIP** | Dependency Inversion Principle | Zasady SOLID |
| **DTO** | Data Transfer Object | Wzorce projektowe |
| **EAR** | Enterprise Archive | Pakiety wdrożeniowe |
| **ECDSA** | Elliptic Curve Digital Signature Algorithm | Kryptografia |
| **EIS** | Enterprise Information System | Architektura Enterprise |
| **EJB** | Jakarta Enterprise Beans (Enterprise JavaBeans) | Komponenty biznesowe |
| **EL** | Expression Language | Web / Prezentacja |
| **ERP** | Enterprise Resource Planning | Systemy korporacyjne |
| **HATEOAS**| Hypermedia As The Engine Of Application State | Styl REST |
| **HMAC** | Hash-based Message Authentication Code | Kryptografia |
| **HTML** | Hypertext Markup Language | Web / Prezentacja |
| **HTTP** | Hypertext Transfer Protocol | Protokoły sieciowe |
| **HTTPS** | Hypertext Transfer Protocol Secure | Protokoły sieciowe |
| **IdP** | Identity Provider | Bezpieczeństwo / Tożsamość |
| **IIOP** | Internet Inter-ORB Protocol | Protokoły zdalne |
| **IoC** | Inversion of Control | Wzorce architektoniczne |
| **J2EE** | Java 2 Platform, Enterprise Edition | Platforma |
| **JAR** | Java Archive | Pakiety wdrożeniowe |
| **Java EE** | Java Platform, Enterprise Edition | Platforma |
| **Java SE** | Java Platform, Standard Edition | Platforma bazowa |
| **Jakarta EE**| Jakarta Enterprise Edition | Platforma |
| **JAX-RS** | Jakarta RESTful Web Services | Usługi sieciowe |
| **JCA** | Jakarta Connectors | Integracja Enterprise |
| **JDBC** | Java Database Connectivity | Bazy danych |
| **JMS** | Jakarta Messaging (Java Message Service) | Kolejki komunikatów |
| **JNDI** | Java Naming and Directory Interface | Usługa katalogowa |
| **JPA** | Jakarta Persistence (Java Persistence API) | Trwałość / ORM |
| **JPQL** | Java Persistence Query Language | Języki zapytań |
| **JSF** | Jakarta Faces (JavaServer Faces) | Frameworki UI |
| **JSON** | JavaScript Object Notation | Formaty danych |
| **JSON-B** | Jakarta JSON Binding | Serializacja JSON |
| **JSON-P** | Jakarta JSON Processing | Przetwarzanie JSON |
| **JSP** | JavaServer Pages | Widoki Web |
| **JSR** | Java Specification Request | Standardy Javy |
| **JSTL** | JSP Standard Tag Library | Biblioteki znaczników |
| **JVM** | Java Virtual Machine | Środowisko uruchomieniowe |
| **JWT** | JSON Web Token | Tokeny bezpieczeństwa |
| **L1 Cache**| Level 1 Cache (Persistence Context) | Pamięć podręczna JPA |
| **L2 Cache**| Level 2 Cache (EntityManagerFactory) | Pamięć podręczna JPA |
| **LDAP** | Lightweight Directory Access Protocol | Usługi katalogowe |
| **MDB** | Message-Driven Bean | Komponenty asynchroniczne |
| **MIME** | Multipurpose Internet Mail Extensions | Typy mediów |
| **OAuth2** | Open Authorization 2.0 | Protokoły autoryzacji |
| **OpenAPI** | OpenAPI Specification | Dokumentacja REST |
| **ORM** | Object-Relational Mapping | Wzorce bazodanowe |
| **PBKDF2** | Password-Based Key Derivation Function 2 | Haszowanie haseł |
| **PKCE** | Proof Key for Code Exchange | Bezpieczeństwo OAuth |
| **POJO** | Plain Old Java Object | Obiekty domenowe |
| **POX** | Plain Old XML | Formaty danych |
| **PRG** | Post-Redirect-Get | Wzorce webowe |
| **RAR** | Resource Adapter Archive | Pakiety wdrożeniowe |
| **RBAC** | Role-Based Access Control | Kontrola dostępu |
| **REST** | Representational State Transfer | Style architektoniczne |
| **RMI** | Remote Method Invocation | Wywołania zdalne |
| **RPC** | Remote Procedure Call | Modele komunikacji |
| **RSA** | Rivest–Shamir–Adleman | Kryptografia |
| **SOLID** | S-O-L-I-D Principles | Projektowanie obiektowe |
| **SOP** | Same Origin Policy | Bezpieczeństwo przeglądarek |
| **SQL** | Structured Query Language | Bazy danych |
| **SSL** | Secure Sockets Layer | Bezpieczeństwo sieciowe |
| **SSO** | Single Sign-On | Uwierzytelnianie |
| **TLS** | Transport Layer Security | Bezpieczeństwo sieciowe |
| **UI** | User Interface | Interfejs użytkownika |
| **URI** | Uniform Resource Identifier | Adresowanie sieciowe |
| **URL** | Uniform Resource Locator | Adresowanie sieciowe |
| **WADL** | Web Application Description Language | Opis usług REST |
| **WAR** | Web Archive | Pakiety wdrożeniowe |
| **XHTML** | Extensible Hypertext Markup Language | Języki znaczników |
| **XML** | Extensible Markup Language | Formaty danych |
| **XSS** | Cross-Site Scripting | Podatności bezpieczeństwa |
