# 01. Platforma Jakarta EE – Teoria na Kolokwium

## 1. Charakterystyka i wymagania aplikacji klasy Enterprise
Aplikacje enterprise (korporacyjne) to złożone systemy informatyczne wspierające kluczowe procesy biznesowe w przedsiębiorstwach i instytucjach. Charakteryzują się:
- **Wysokim poziomem złożoności:** składają się z dziesiątek lub setek współpracujących modułów, realizują tysiące reguł biznesowych o skomplikowanych powiązaniach.
- **Krytycznością biznesową (Mission-Critical):** awaria lub przestój przynoszą bezpośrednie straty finansowe i wizerunkowe; wymagają najwyższej niezawodności (*reliability*), ciągłości działania (*high availability*) oraz rygorystycznego bezpieczeństwa.
- **Wymaganiami wydajnościowymi i skalowalnością:** obsługa ogromnych wolumenów danych (dane wielkiej skali) oraz jednoczesna obsługa tysięcy lub milionów użytkowników i procesów.
- **Specyfiką wytwarzania i utrzymania:** rozwijane przez wieloosobowe, rozproszone zespoły programistów; cykl życia systemu wynosi od kilku do kilkunastu lat; kluczowe jest modularne projektowanie, definiowanie ścisłych kontraktów interfejsowych oraz przemyślany długoterminowy dług technologiczny.
- **Integracją heterogeniczną:** konieczność jednoczesnej obsługi wielu zróżnicowanych klientów (aplikacje webowe, mobilne, desktopowe, usługi B2B) oraz integracji z zewnętrznymi systemami dziedzinowymi i systemami spuścizny (*legacy systems*), których modernizacja jest nieopłacalna.
- **Specyficznymi wymaganiami technologicznymi:**
  - Asynchroniczna wymiana komunikatów.
  - Gwarantowane dostarczanie wiadomości (kolejkowanie).
  - Rozproszone transakcje dwufazowe (2PC).
  - Przetwarzanie wsadowe wielkich zbiorów danych (*batch processing*).
  - Realizacja zadań cyklicznych (*scheduler*).
  - Wielopoziomowa testowalność: testy jednostkowe, integracyjne, akceptacyjne, interfejsu użytkownika.
- **Modelem programistycznym:** silne typowanie, architektoniczny podział na warstwy logiczne, separacja odpowiedzialności, luźne powiązania (*loose coupling*) i programowanie oparte o kontrakty (interfejsy).

---

## 2. Czym jest Jakarta EE? Definicja i ewolucja
**Jakarta EE (dawniej Java EE / J2EE)** to ustandaryzowany zbiór specyfikacji rozszerzających bibliotekę bazową Java SE o komponenty i usługi niezbędne do budowania aplikacji klasy enterprise.

### Historia i wersje:
1. **J2EE (Java 2 Platform, Enterprise Edition)** – rozwijana przez firmę Sun Microsystems:
   - J2EE 1.2 (1999)
   - J2EE 1.3 (2001)
   - J2EE 1.4 (2003)
2. **Java EE (Java Platform, Enterprise Edition)** – przejęcie i rozwój przez Oracle:
   - Java EE 5 (2006) – rewolucja w prostocie: wprowadzenie adnotacji i POJO, uproszczenie EJB 3.0.
   - Java EE 6 (2009) – wprowadzenie profilu Web Profile oraz specyfikacji CDI.
   - Java EE 7 (2013) – wsparcie dla HTML5, WebSocket, Batch Processing.
   - Java EE 8 (2017) – JSON-B, Security API, reaktywność.
3. **Jakarta EE** – przekazanie projektu fundacji Eclipse Foundation:
   - **Jakarta EE 8 (2019):** przeniesienie do otwartego zarządu Eclipse, pełna zgodność z Java EE 8 pod starą przestrzenią nazw pakietów (`javax.*`).
   - **Jakarta EE 9 (2020):** kluczowa zmiana przestrzeni nazw pakietów – przejście z `javax.*` na `jakarta.*` (tzw. "Big Bang").
   - **Jakarta EE 10 (2022):** wprowadzenie Core Profile pod mikrousługi, modernizacja CDI (CDI Lite).
   - **Jakarta EE 11 (2025):** nowa specyfikacja Jakarta Data, odświeżenie specyfikacji i wymagań językowych.
   - **Jakarta EE 12:** wersja w ciągłym rozwoju.

---

## 3. Poziomy architektury platformy Jakarta EE (Tiers)
Aplikacje Jakarta EE implementują architekturę wielowarstwową / wielopoziomową (*n-tier*):

```
+-------------------------------------------------------------+
| 1. Poziom klienta (Client Tier)                             |
|    - Przeglądarka internetowa (HTML/JS)                     |
|    - Aplikacje klienckie desktop/okienkowe                  |
+-------------------------------------------------------------+
                              | (HTTP / RMI-IIOP)
                              v
+-------------------------------------------------------------+
| Serwer Aplikacji (Application Server)                       |
|  +-------------------------------------------------------+  |
|  | 2. Poziom webowy (Web Tier)                           |  |
|  |    - Servlety (przetwarzanie żądań HTTP)              |  |
|  |    - Strony widoków (JSF/Jakarta Faces, JSP)          |  |
|  |    - Punkty końcowe REST (JAX-RS)                     |  |
|  +-------------------------------------------------------+  |
|                             |                               |
|  +-------------------------------------------------------+  |
|  | 3. Poziom biznesowy (Business Tier)                   |  |
|  |    - Komponenty logiki biznesowej (EJB Session Beans)  |  |
|  |    - Komponenty sterowane komunikatami (MDB)          |  |
|  |    - Encje trwałości (JPA Entities)                   |  |
|  +-------------------------------------------------------+  |
+-------------------------------------------------------------+
                              | (JDBC / JCA)
                              v
+-------------------------------------------------------------+
| 4. Poziom danych (EIS Tier - Enterprise Information System) |
|    - Relacyjne bazy danych (RDBMS)                          |
|    - Systemy transakcyjne, systemy klasy ERP / CRM          |
+-------------------------------------------------------------+
```

### Charakterystyka poziomów:
- **Poziom klienta (Client Tier):** środowisko uruchamiane na maszynie użytkownika. Może to być cienki klient (przeglądarka renderująca HTML/JS) lub gruby klient (dedykowana aplikacja okienkowa komunikująca się ze środowiskiem serwera).
- **Poziom webowy (Web Tier):** działa po stronie serwera aplikacji. Odpowiada za bezpośrednią interakcję z klientem sieciowym przez protokół HTTP, routing żądań, autoryzację wejścia, parsowanie danych oraz generowanie dynamicznych odpowiedzi widoku (strony HTML, JSON).
- **Poziom biznesowy (Business Tier):** serce logiki dziedzinowej systemu. Zawiera implementacje reguł biznesowych, kalkulacji, zarządza transakcjami wielokrokowymi i koordynacją procesów.
- **Poziom danych (EIS Tier):** zewnętrzne źródła danych i systemy centralne przedsiębiorstwa.

---

## 4. Pojęcie kontenera i relacje między kontenerami
**Kontener** w Jakarta EE to zintegrowana część środowiska serwera aplikacji, która zarządza cyklem życia wdrażanych komponentów i udostępnia im transparentnie usługi niskopoziomowe (wstrzykiwanie zależności, transakcyjność, bezpieczeństwo, pooling, wielowątkowość, routing).

### Rodzaje kontenerów:
1. **Kontener Webowy (Web Container / Servlet Container):**
   - Zarządza cyklem życia servletów, filtrów, listenerów oraz komponentów widoków (Jakarta Faces / JSF, JSP).
   - Odpowiada za mapowanie żądań sieciowych HTTP na odpowiednie komponenty.
2. **Kontener EJB (EJB Container):**
   - Zarządza cyklem życia komponentów biznesowych EJB (sesyjnych bezstanowych, stanowych, singletonów oraz message-driven).
   - Zapewnia deklaratywne zarządzanie transakcjami (CMT), dostęp współbieżny oraz wywołania zdalne.
3. **Kontener Aplikacji Klienckiej (Application Client Container):**
   - Działa na maszynie klienta, wspiera wykonywanie aplikacji okienkowych, umożliwiając wstrzykiwanie zasobów serwerowych i lookup JNDI.
4. **Kontener apletów (Applet Container - archiwalny/wycofany):**
   - Działał w kontekście przeglądarki klienta.

### Poprawne relacje między kontenerami:
- Ścisła zasada separacji odpowiedzialności: **wyłącznie komponenty poziomu biznesowego (EJB / warstwa serwisowa z JPA) powinny bezpośrednio komunikować się z bazą danych (EIS)**.
- Komponenty warstwy webowej (Servlety, kontrolery JSF, endpointy REST) nie powinny samodzielnie manipulować bazą danych – powinny delegować operacje do komponentów biznesowych.
- Klienci zewnętrzni mogą komunikować się z warstwą webową (HTTP/REST) lub bezpośrednio z EJB poprzez interfejsy zdalne (Remote/RMI).

---

## 5. Profile platformy Jakarta EE
Profil to standaryzowany podzbiór specyfikacji Jakarta EE, ukierunkowany na konkretną klasę zastosowań architektonicznych:

1. **Full Profile (Pełna Platforma):**
   - Zawiera pełen zestaw wszystkich specyfikacji Jakarta EE (m.in. EJB Full, JMS, Jakarta Mail, Batch Processing, Connectors/JCA, transakcje rozproszone).
   - Obsługuje archiwa typu EAR oraz WAR.
   - Dedykowany wielkim systemom monolitycznym i rozproszonym systemom korporacyjnym.
2. **Web Profile (od Java EE 6):**
   - Odchudzony profil zoptymalizowany dla standardowych aplikacji WWW i portali internetowych.
   - Zawiera specyfikacje warstwy webowej (Servlet, Faces, Pages/JSP, JSTL) oraz kluczowe elementy warstwy biznesowej (CDI, JPA, Bean Validation, transakcje, EJB Lite).
   - Wyklucza "ciężkie" technologie enterprise: zdalne wywołania EJB (remoting), JMS, przetwarzanie wsadowe (Batch), Connectors.
   - Wdrażany wyłącznie jako archiwa WAR.
3. **Core Profile (od Jakarta EE 10):**
   - Minimalistyczny profil zaprojektowany pod kątem lekkich mikrousług (*microservices*) oraz środowisk chmurowych (cloud-native, konteneryzacja).
   - Zawiera tylko niezbędne minimum: Jakarta RESTful Web Services (JAX-RS), CDI Lite, JSON-B, JSON-P, Annotations, Interceptors.
   - Pozwala na szybki start, niskie zużycie pamięci i kompilację do obrazów natywnych (np. GraalVM).

---

## 6. Serwery kompatybilne
- **Serwery pełne (Full / Web Profile):** Eclipse GlassFish (referencyjny), Payara Server, Red Hat WildFly (dawniej JBoss), IBM Open Liberty.
- **Kontenery webowe (Servlet Containers - tylko wybrane specyfikacje webowe, bez pełnego profilu EE):** Apache Tomcat, Eclipse Jetty.

---

## 7. Struktura archiwów aplikacji Enterprise
Aplikacje enterprise są pakowane w ustandaryzowane archiwa oparte na formacie ZIP:

### 1. Archiwum WAR (Web Archive – `.war`):
- Reprezentuje samodzielny moduł aplikacji webowej.
- Struktura:
  - Zasoby publiczne w katalogu głównym (strony HTML, XHTML, arkusze CSS, skrypty JS, grafiki).
  - Katalog `WEB-INF/`:
    - `web.xml` – deskryptor wdrożenia aplikacji webowej.
    - `beans.xml` – konfiguracja kontekstu CDI.
    - `classes/` – skompilowane klasy Javy poziomu webowego (pliki `.class`).
    - `lib/` – zależne biblioteki zewnętrzne (pliki `.jar`).

### 2. Archiwum EJB-JAR (`.jar`):
- Zawiera samodzielny moduł komponentów biznesowych EJB.
- Struktura:
  - Skompilowane klasy komponentów biznesowych, interfejsów i encji.
  - Katalog `META-INF/`:
    - `ejb-jar.xml` – standardowy deskryptor komponentów EJB (opcjonalny).
    - `persistence.xml` – konfiguracja jednostki trwałości JPA.
    - Specyficzne deskryptory serwerowe (np. `glassfish-ejb-jar.xml`, `jboss-ejb3.xml`).

### 3. Archiwum EAR (Enterprise Archive – `.ear`):
- Złożone archiwum najwyższego rzędu, łączące wiele modułów w jedną aplikację enterprise.
- Może zawierać:
  - Jeden lub wiele modułów webowych (`.war`).
  - Jeden lub wiele modułów biznesowych (`.jar` / EJB-JAR).
  - Moduły adapterów zasobów (`.rar` / Resource Adapter).
  - Moduły klienta aplikacji (`.jar`).
  - Katalog `META-INF/application.xml` – główny deskryptor spinający wszystkie moduły w całość.
- Wdrażane wyłącznie na serwerach obsługujących profil Full Profile.

---

## 8. Pojęcie zależności o zasięgu "provided" (Maven)
W projektach aplikacji Jakarta EE biblioteki platformy (interfejsy i API) są deklarowane ze statusem / zasięgiem `provided`, ponieważ:
- **Ich implementacja fizycznie znajduje się w samym serwerze aplikacji (jest dostarczana przez środowisko wykonawcze kontenera).**
- Włączenie tych bibliotek do tworzonego archiwum (`.war` lub `.ear`) skutkowałoby konfliktami wersji classloaderów (*ClassCastException*, *LinkageError*) oraz niepotrzebnym powiększeniem rozmiaru paczki.
