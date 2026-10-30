# Kompendium Teoretyczne – Narzędzia i Aplikacje Jakarta EE

Zbiór opracowań teoretycznych przygotowanych pod kątem kolokwium/egzaminu teoretycznego, bez kodu implementacyjnego. Opracowania skupiają się na definicjach, wzorcach projektowych, cyklach życia, modelach architektonicznych, porównaniach pojęć i mechanizmach platformy.

## Spis Wykładów:

1. [01. Platforma Jakarta EE](01_platforma_jakarta_ee.md)
   - Wymagania i specyfika aplikacji enterprise
   - Definicja i ewolucja Jakarta EE (J2EE -> Java EE -> Jakarta EE)
   - Poziomy architektury (Client, Web, Business, EIS Tier)
   - Kontenery platformy i poprawne relacje między nimi
   - Profile platformy (Full, Web, Core Profile)
   - Struktura archiwów (WAR, EJB-JAR, EAR) oraz zależność `provided`

2. [02. Jakarta Servlet](02_jakarta_servlet.md)
   - Pojęcie i rola servletu (HttpServlet)
   - Cykl życia servletu (init -> service/doGet/doPost -> destroy)
   - Wielowątkowość a bezpieczeństwo wątkowe (jedna instancja servletu dla wielu wątków)
   - Poziomy współdzielenia obiektów i zasięgi (Request, Session, Application / ServletContext)
   - Nasłuchiwacze zdarzeń (Listeners) i kolejność w web.xml
   - Filtry (Filter / HttpFilter, FilterChain) i ich zastosowania
   - Niemodyfikowalność parametrów HTTP vs modyfikowalność atrybutów
   - Rola servletów we współczesnych systemach

3. [03. Jakarta Contexts and Dependency Injection (CDI)](03_cdi_i_wstrzykiwanie_zaleznosci.md)
   - Wzorzec Dependency Injection (DI) – pojęcie, zalety, konsekwencje
   - Trzy formy wstrzykiwania (konstruktor, setter, pole) – wady i zalety
   - DI a Inversion of Control (IoC) – kluczowa różnica pojęciowa
   - Standardy JSR 330 a Jakarta CDI (Weld)
   - Rodzaje archiwów CDI (explicit vs implicit, bean-discovery-mode: all vs annotated)
   - Warunki kwalifikacji klasy do zarządzania przez CDI
   - Zasięgi CDI (@Dependent, @RequestScoped, @ConversationScoped, @SessionScoped, @ApplicationScoped, @Singleton, @ViewScoped, @TransactionScoped)
   - Integracja CDI z Expression Language (@Named)

4. [04. Jakarta Persistence (JPA)](04_jakarta_persistence_jpa.md)
   - Pojęcie ORM i specyfikacja JPA (ACID, niezależność od bazy, implementacje: Hibernate, EclipseLink)
   - Klasy encyjne (POJO, wymogi formalne, klucz główny)
   - Strategie generowania kluczy głównych (IDENTITY, SEQUENCE, TABLE, AUTO)
   - Relacje między encjami, strona właścicielska relacji a atrybut `mappedBy`, kaskadowość
   - Jednostka trwałości (Persistence Unit, JTA vs RESOURCE_LOCAL)
   - Kontekst trwałości (Persistence Context, dirty checking, cache 1 poziomu)
   - 4 Stany encji i cykl życia (New/Transient, Managed, Detached, Removed)
   - Kluczowe operacje EntityManager (persist, merge, remove, refresh, find, flush)
   - Język JPQL – różnice względem SQL, zapytania nazwane (@NamedQuery)
   - Bazy danych: standalone DBMS vs embedded (H2)

5. [05. Jakarta Faces (JSF)](05_jakarta_faces_jsf.md)
   - Pojęcie komponentowego frameworka UI po stronie serwera (server-side UI)
   - Rola FacesServlet (Front Controller) i struktura widoków (Facelets, przestrzenie nazw)
   - Expression Language: różnica między `${...}` (immediate, rvalue) a `#{...}` (deferred, rvalue/lvalue, two-way binding)
   - Backing Beans i dlaczego zasięg `@ViewScoped` jest kluczowy dla JSF
   - Cykl życia żądania w JSF – 6 FAZ (Restore View, Apply Request Values, Process Validations, Update Model Values, Invoke Application, Render Response)
   - Obsługa błędów walidacji (skok z Fazy 3 do Fazy 6)
   - Problem podwójnego zatwierdzenia formularza i idiom POST-REDIRECT-GET (PRG)
   - Szablony Facelets, konwertery, walidatory, faces-config.xml

6. [06. Jakarta RESTful Web Services (JAX-RS)](06_jakarta_restful_web_services.md)
   - Usługi sieciowe i styl architektoniczny REST (założenia: bezstanowość, buforowalność)
   - Model dojrzałości usług REST według Richardsona (Poziom 0 do 3: Swamp of POX, Zasoby, Czasowniki HTTP, HATEOAS)
   - Semantyka metod HTTP, pojęcie metody bezpiecznej oraz metody idempotentnej
   - Porównanie metod: GET, POST, PUT, PATCH, DELETE
   - Projektowanie URI: hierarchia zasobów, parametry ścieżki vs parametry zapytania
   - Kody statusu HTTP (200, 201 + Location, 202, 204, 400, 401, 403, 404, 500)
   - Same Origin Policy (SOP) a Cross-Origin Resource Sharing (CORS, preflight OPTIONS)

7. [07. Jakarta Enterprise Beans (EJB)](07_jakarta_enterprise_beans_ejb.md)
   - Pojęcie komponentów EJB i rola kontenera (automatyczne transakcje CMT, bezpieczeństwo, pooling, klastry)
   - Rodzaje beanów EJB: sesyjne (Stateless, Stateful, Singleton) oraz Message-Driven Beans (MDB, JMS)
   - Interfejsy biznesowe: lokalny (@Local), zdalny (@Remote - wymóg Serializable, brak lazy fetch), no-interface (@LocalBean)
   - Cykl życia beanów bezstanowych/singletonów (@PostConstruct, @PreDestroy)
   - Cykl życia beana stanowego: proces pasywacji (@PrePassivate) i aktywacji (@PostActivate), zakończenie (@Remove)
   - Usługa katalogowa JNDI i przestrzenie nazw (java:global, java:app, java:module)
   - Współbieżność w EJB Singleton: ConcurrencyManagement (CONTAINER vs BEAN), blokady LockType (READ vs WRITE)

8. [08. Uwierzytelnianie i Autoryzacja](08_uwierzytelnianie_i_autoryzacja.md)
   - Uwierzytelnianie (AuthN - "kim jesteś?") vs Autoryzacja (AuthZ - "co wolno ci zrobić?")
   - Model kontroli dostępu oparty o role (RBAC) – uprawnienia przypisane do ról, role do użytkowników
   - Uwierzytelnianie w HTTP: Basic Auth, Form Auth, tokeny, OpenID Connect
   - Utrzymywanie sesji: pliki cookie (JSESSIONID), URL Rewriting, tokeny
   - Przebieg logowania formularzowego w Jakarta EE (przechwycenie wejścia na chroniony zasób -> formularz -> przekierowanie)
   - Protokół OAuth2: definicja, cel, 4 role, przegląd przepływów (Authorization Code z PKCE, Client Credentials, etc.)
   - Tokeny JWT: struktura Header.Payload.Signature w Base64Url, standardowe roszczenia (iss, sub, aud, exp, nbf, iat, jti), podpisywanie symetryczne vs asymetryczne
   - Bezpieczeństwo deklaratywne (web.xml, @RolesAllowed, @ServletSecurity, @RunAs) vs programistyczne (SecurityContext, isCallerInRole)
   - Jakarta Security: Identity Stores (@DatabaseIdentityStoreDefinition, LDAP), haszowanie haseł Pbkdf2PasswordHash

---

## Opracowania Egzaminacyjne:
- ⏱️ **[Przed Kolosem – Esencja Teorii (3 zdania na temat)](../przed_kolosem.md)**
- 📑 **[Zestaw 18 Pytań i Odpowiedzi Egzaminacyjnych](../pytania/zestaw_pytan_i_odpowiedzi.md)**
- 🎯 **[Zagadnienia Wskazane przez Wykładowcę](../pytania/zagadnienia_od_wykladowcy.md)** (CDI, OAuth Flows, Persistence, Cache JPA L1/L2, PRG, Wszystkie Cykle Życia)
- 📖 **[Słownik Pojęć i Skrótów](../slownik_skrotow.md)** (ponad 80 skrótów z rozwinięciem i rolą)
