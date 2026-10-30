# Kluczowe Zagadnienia Wskazane przez Wykładowcę – Opracowanie Teoretyczne

Niniejsze opracowanie obejmuje szczegółowe omówienie sześciu wytypowanych przez wykładowcę tematów egzaminacyjnych z naciskiem na teorię, architekturę, porównania oraz cykle życia (bez kodu implementacyjnego).

---

## SPIS TREŚCI:
1. [1. CDI (Contexts and Dependency Injection)](#1-cdi-contexts-and-dependency-injection)
2. [2. Przepływy OAuth 2.0 – Rozróżnianie i Porównywanie](#2-przepływy-oauth-20--rozróżnianie-i-porównywanie)
3. [3. Persistence (Jakarta Persistence / JPA)](#3-persistence-jakarta-persistence--jpa)
4. [4. Obsługa Pamięci Podręcznej (Cache) w JPA (L1 vs L2)](#4-obsługa-pamięci-podręcznej-cache-w-jpa-l1-vs-l2)
5. [5. Wzorzec POST-REDIRECT-GET (PRG)](#5-wzorzec-post-redirect-get-prg)
6. [6. Zestawienie Wszystkich Cykli Życia w Jakarta EE](#6-zestawienie-wszystkich-cykli-życia-w-jakarta-ee)

---

## 1. CDI (Contexts and Dependency Injection)

### 1.1. Czym jest CDI i jaka jest jego rola w architekturze?
**Jakarta CDI** to fundamentalny szkielet integrujący wszystkie warstwy platformy Jakarta EE. Stanowi zaawansowane rozszerzenie bazowego standardu wstrzykiwania zależności (JSR 330) o koncepcję **zarządzania cyklem życia obiektów w ramach kontekstów sieciowych i aplikacyjnych**.

### 1.2. IoC (Odwrócenie Sterowania) a DI (Wstrzykiwanie Zależności)
- **Inversion of Control (IoC):** Ogólna zasada architektoniczna mówiąca, że to **kontener frameworka steruje przepływem programu** (cyklem życia komponentów, wątkami, reakcją na zdarzenia) i sam wywołuje kod użytkownika (Hollywood Principle: *"Nie dzwoń do nas, to my zadzwonimy do ciebie"*).
- **Dependency Injection (DI):** Wzorzec projektowy będący techniczną realizacją zasady IoC, w którym obiekt nie tworzy samodzielnie swoich zależności operatorem `new` ani ich nie wyszukuje, lecz są mu one dostarczane z zewnątrz przez kontener.

### 1.3. Trzy formy wstrzykiwania zależności:
1. **Przez konstruktor (*Constructor Injection*):**
   - Pozwala na tworzenie obiektów **niemutowalnych** (pola `final`).
   - Gwarantuje, że obiekt natychmiast po utworzeniu jest kompletny i spójny.
   - Bardzo łatwy w testach jednostkowych (brak potrzeby refleksji).
2. **Przez metody ustawiające (*Setter Injection*):**
   - Umożliwia opcjonalną podmianę zależności w trakcie życia obiektu.
   - Wymaga konstruktora domyślnego i niesie ryzyko istnienia obiektu częściowo zainicjalizowanego.
3. **Bezpośrednio do pól (*Field Injection*):**
   - Najbardziej zwięzły zapis.
   - Wymaga użycia mechanizmu refleksji (omijanie hermetyzacji pól prywatnych), uniemożliwia stosowanie pól `final`, utrudnia testy jednostkowe poza kontenerem.

### 1.4. Zasięgi (Scopes) w CDI – Porównanie
Zasięg definiuje, kiedy obiekt powstaje, kiedy jest niszczony i dla kogo jest widoczny:

| Adnotacja Zasięgu | Czas trwania i widoczność | Praktyczne zastosowanie |
| :--- | :--- | :--- |
| **`@Dependent`** *(Domyślny)* | **Brak współdzielenia.** Każde wstrzyknięcie tworzy nową instancję. Żyje tyle, ile obiekt nadrzędny. | Klasy pomocnicze, kalkulatory, formatery danych. |
| **`@RequestScoped`** | Żyje przez czas trwania **pojedynczego żądania HTTP** (niszczony po odesłaniu odpowiedzi). | Dane formularza wejściowego, token zapytania, filtry wyszukiwania. |
| **`@ConversationScoped`** | Obejmuje **ciąg powiązanych żądań HTTP**. Sterowany metodami `begin()` i `end()`. | Wieloetapowe kreatory (*wizardy*), rezerwacja biletów. |
| **`@SessionScoped`** | Żyje tak długo, jak aktywna jest **sesja danego użytkownika** w przeglądarce. | Koszyk zakupowy, preferencje językowe, dane zalogowanego klienta. |
| **`@ApplicationScoped`** | **Jedna instancja** współdzielona w całej aplikacji dla wszystkich użytkowników i sesji. | Globalny bufor podręczny (cache), parametry konfiguracyjne, słowniki. |
| **`@ViewScoped`** *(Faces)* | Żyje tak długo, jak użytkownik pozostaje na **tym samym widoku/stronie JSF**. | Stan tabeli (sortowanie w locie przez AJAX), stan edytowanego rekordu. |

### 1.5. Dlaczego działa wstrzykiwanie mniejszego zasięgu do większego? (Mechanizm Client Proxy)
- **Problem:** Jak wstrzyknąć krótkożyjący bean `@RequestScoped` (żyjący 20 ms) do długożyjącego `@ApplicationScoped` (żyjącego tygodniami), aby nie doszło do powiązania singletonu z danymi pierwszego użytkownika?
- **Rozwiązanie (Client Proxy):** Kontener CDI nie wstrzykuje bezpośredniego obiektu, lecz generuje w locie **inteligentnego pełnomocnika (Client Proxy)**.
- Gdy metoda na proxy jest wywoływana, proxy sprawdza identyfikator wątku (`ThreadLocal`), odnajduje u Menedżera Kontekstów instancję właściwą dla bieżącego żądania HTTP i przekazuje do niej wywołanie. Dzięki temu ten sam singleton równolegle obsługuje Jana i Annę, kierując wywołania do ich odrębnych obiektów żądania.

### 1.6. Rodzaje archiwów CDI:
- **Jawne (*Explicit*):** Zawiera plik `beans.xml`. W trybie `bean-discovery-mode="all"` kontener zarządza wszystkimi klasami; w trybie `"annotated"` tylko klasami z jawnymi adnotacjami zasięgu.
- **Niejawne (*Implicit*):** Brak pliku `beans.xml`. Kontener działa w domyślnym trybie `annotated`.

---

## 2. Przepływy OAuth 2.0 – Rozróżnianie i Porównywanie

### 2.1. Czym jest OAuth 2.0?
OAuth 2.0 to otwarty standard **autoryzacji delegowanej**. Pozwala użytkownikowi (Resource Owner) na udzielenie aplikacji trzeciej (Client) ograniczonego dostępu do swoich zasobów zgromadzonych na serwerze API (Resource Server) za pośrednictwem Serwera Autoryzacji (Authorization Server), **bez ujawniania aplikacji swojego hasła**.

### 2.2. Podział klientów w OAuth 2.0:
- **Klient poufny (*Confidential Client*):** Aplikacja posiadająca bezpieczny backend serwerowy (np. tradycyjna aplikacja Java EE/Spring). Potrafi utrzymać w tajemnicy własne hasło/sekret (`client_secret`).
- **Klient publiczny (*Public Client*):** Aplikacja działająca na urządzeniu użytkownika (aplikacja mobilna na smartfonie, aplikacja SPA w JavaScript w przeglądarce). Kod można zdekompilować lub podejrzeć w dev-toolsach, przez co **nie jest w stanie bezpiecznie ukryć sekretu klienta**.

---

### 2.3. Przegląd i Porównanie Wszystkich Przepływów (Grant Types):

```
                        +-------------------------------------------------------------+
                        | Jaki typ aplikacji żąda dostępu do zasobów?                 |
                        +-------------------------------------------------------------+
                                                       |
            +------------------------------------------+------------------------------------------+
            |                                          |                                          |
            v                                          v                                          v
+------------------------+                 +------------------------+                 +------------------------+
| Komunikacja M2M        |                 | Użytkownik + Bezpieczny|                 | Użytkownik + Klient    |
| (Serwer - Serwer)      |                 | Backend (Aplikacja WWW)|                 | Publiczny (SPA/Mobile) |
+------------------------+                 +------------------------+                 +------------------------+
            |                                          |                                          |
            v                                          v                                          v
 [ Client Credentials ]                    [ Authorization Code ]                     [ Authorization Code     ]
                                                                                      [ z rozszerzeniem PKCE   ]
```

#### 1. Authorization Code Flow (Przepływ z kodem autoryzacyjnym)
- **Dla kogo:** Klasyczne aplikacje internetowe z bezpiecznym backendem (klienci poufni).
- **Przebieg:**
  1. Przeglądarka przekierowuje użytkownika na stronę logowania Serwera Autoryzacji.
  2. Użytkownik loguje się i zatwierdza uprawnienia (zgodę).
  3. Serwer Autoryzacji odsyła przeglądarkę do aplikacji klienta z jednorazowym **kodem autoryzacyjnym** (*authorization code*).
  4. Backend klienta łączy się bezpośrednio (poza przeglądarką) z Serwerem Autoryzacji i wymienia kod autoryzacyjny na **Access Token**, uwierzytelniając się swoim poufnym `client_secret`.
- **Bezpieczeństwo:** Bardzo wysokie – token dostępu nigdy nie trafia do przeglądarki użytkownika, pozostając wyłącznie w bezpiecznym backendzie.

#### 2. Authorization Code Flow z PKCE (Proof Key for Code Exchange)
- **Dla kogo:** Aplikacje jednostronicowe (SPA – Angular, React, Vue) oraz aplikacje mobilne (iOS, Android).
- **Dlaczego powstał:** Ponieważ klienci publiczni nie mogą posiadać `client_secret` (zostałby natychmiast wykradziony). Zwykły Authorization Code bez sekretu byłby podatny na przechwycenie kodu autoryzacyjnego przez złośliwą aplikację w systemie operacyjnym.
- **Jak działa PKCE:** Aplikacja generuje dynamicznie w locie losowy ciąg znaków (`code_verifier`) oraz jego skrót kryptograficzny (`code_challenge`). Skrót wysyłany jest przy żądaniu kodu, a oryginalny ciąg przy wymianie na token. Serwer autoryzacji weryfikuje matematycznie, czy ten sam podmiot zainicjował i zakończył proces.
- **Status:** **Obecny złoty standard** dla aplikacji frontendowych i mobilnych.

#### 3. Client Credentials Flow (Przepływ poświadczeń klienta)
- **Dla kogo:** Komunikacja **maszyna-maszyna (M2M)**, mikroserwisy, demony systemowe, zadania wsadowe (Batch).
- **Cechy:** **Całkowity brak udziału użytkownika końcowego**. Aplikacja uwierzytelnia się wyłącznie własnym identyfikatorem `client_id` i sekretem `client_secret`.
- **Token:** Wydany token reprezentuje uprawnienia samej aplikacji jako instytucji, a nie konkretnej osoby fizycznej.

#### 4. Resource Owner Password Credentials Flow (Bezpośrednie hasło)
- **Dla kogo:** Użytkownik wpisuje swój login i hasło bezpośrednio w formularzu aplikacji klienta, a ta wysyła je do serwera autoryzacji.
- **Ocena:** **Przestarzały i wysoce niezalecany (antywzorzec)**. Łamie główny postulat bezpieczeństwa OAuth (użytkownik nie powinien ujawniać swojego hasła aplikacjom trzecim). Dozwolony wyłącznie w wyizolowanych systemach wewnętrznych o najwyższym poziomie zaufania.

#### 5. Implicit Flow (Przepływ uproszczony)
- **Dla kogo:** Dawniej projektowany dla aplikacji przeglądarkowych JavaScript. Token był zwracany bezpośrednio w hashu adresu URL (`#access_token=...`) bez etapu wymiany kodu.
- **Ocena:** **Wycofany i zakazany w specyfikacji OAuth 2.1**. Niezwykle podatny na wyciek tokena z historii przeglądarki i logów serwera proxy. Został w całości zastąpiony przez Authorization Code z PKCE.

#### 6. Device Authorization Flow (Przepływ dla urządzeń)
- **Dla kogo:** Urządzenia bez wygodnej klawiatury lub bez przeglądarki internetowej (Smart TV, konsole do gier, urządzenia IoT).
- **Przebieg:** Urządzenie wyświetla na ekranie krótki kod użytkownika i adres strony www. Użytkownik otwiera stronę na smartfonie lub komputerze, loguje się i wpisuje kod. Urządzenie w tym czasie odpytuje serwer w pętli (*polling*) i po zatwierdzeniu przez użytkownika na telefonie, odbiera Access Token.

---

### 2.4. Zestawienie Porównawcze Przepływów OAuth 2.0:

| Przepływ (Flow) | Typ Klienta | Czy obecny jest użytkownik? | Gdzie trafia Access Token? | Poziom Bezpieczeństwa / Status |
| :--- | :--- | :---: | :--- | :--- |
| **Authorization Code** | Poufny (Backend) | TAK | Wyłącznie do bezpiecznego backendu. | **Bardzo wysoki** (Standard dla aplikacji WWW). |
| **Auth Code + PKCE** | Publiczny (SPA, Mobile) | TAK | Do aplikacji klienckiej / pamięci przeglądarki. | **Wysoki** (Obowiązkowy standard dla SPA/Mobile). |
| **Client Credentials** | Poufny (M2M) | **NIE** | Do serwera wywołującego. | **Bardzo wysoki** (Tylko dla integracji backend-backend). |
| **Resource Owner Password**| Zaufany | TAK | Do aplikacji klienta. | **Niski / Przestarzały** (Niezalecany). |
| **Implicit Flow** | Publiczny (SPA) | TAK | Bezpośrednio do adresu URL przeglądarki. | **Krytycznie niski / Zdeprecjonowany** (Zakazany). |
| **Device Flow** | Urządzenie (Smart TV) | TAK (na innym sprzęcie) | Do urządzenia końcowego. | **Wysoki** (Zaprojektowany pod specyficzne urządzenia). |

---

## 3. Persistence (Jakarta Persistence / JPA)

### 3.1. Czym jest JPA?
Specyfikacja odwzorowania obiektowo-relacyjnego (ORM) w języku Java. Zapewnia automatyczne mapowanie klas na tabele, translację zapytań obiektowych (JPQL) na dialekty SQL oraz gwarantuje realizację transakcji o własnościach **ACID**.

### 3.2. Jednostka Trwałości (*Persistence Unit*) a Kontekst Trwałości (*Persistence Context*)
*To kluczowe rozróżnienie pojęciowe na egzaminie:*
- **Jednostka Trwałości (Persistence Unit):** Byt statyczny/konfiguracyjny zdefiniowany w pliku `persistence.xml`. Grupuje klasy encyjne, definiuje źródło danych (DataSource) oraz typ zarządzania transakcjami (`JTA` na serwerze aplikacji vs `RESOURCE_LOCAL` w Java SE).
- **Kontekst Trwałości (Persistence Context):** Byt dynamiczny/pamięciowy zarządzany przez obiekt `EntityManager`. Jest to zbiór instancji encji, które w danej chwili są **zarządzane (Managed)**. Odpowiada za śledzenie zmian (*dirty checking*) i gwarantuje tożsamość obiektów w transakcji.

### 3.3. Relacje i Pojęcie Strony Właścicielskiej (*Owning Side*)
- W bazie danych relacje łączą tabele za pomocą kluczy obcych (zawsze jednostronnie).
- W JPA asocjacje mogą być dwukierunkowe.
- **Strona właścicielska:** Encja, która **zarządza fizyczną kolumną klucza obcego** w bazie danych (np. strona `@ManyToOne`). Zmiana relacji w pamięci Java musi nastąpić na stronie właścicielskiej, aby zmiana została odzwierciedlona w bazie!
- **Strona odwrócona (pasywna):** Zawiera atrybut `mappedBy = "nazwaPolaWStronieWlascicielskiej"`. Służy jedynie do wygodnego odczytu powiązanych obiektów.

### 3.4. Różnice między JPA w Jakarta EE a Java SE:
- **W Java SE:** Programista manualnie tworzy fabrykę `EntityManagerFactory`, ręcznie pobiera i zamyka `EntityManager` (który nie jest bezpieczny wątkowo) oraz ręcznie steruje transakcjami `RESOURCE_LOCAL` (`begin()`, `commit()`).
- **W Jakarta EE:** Kontener sam zarządza sesjami. Wstrzykuje bezpieczne wątkowo proxy za pomocą `@PersistenceContext`, a transakcjami `JTA` zarządza kontener (CMT) deklaratywnie.

---

## 4. Obsługa Pamięci Podręcznej (Cache) w JPA (L1 vs L2)
*Zagadnienie szczegółowe, o które pyta wykładowca:*

W architekturze JPA wyróżnia się dwa poziomy pamięci podręcznej oraz dodatkowy bufor zapytań:

```
[ Aplikacja / Wątek 1 ]                 [ Aplikacja / Wątek 2 ]
          |                                       |
          v                                       v
+-----------------------+               +-----------------------+
| Cache 1. Poziomu (L1) |               | Cache 1. Poziomu (L1) |
| (Persistence Context) |               | (Persistence Context) |
| - Izolacja wątku      |               | - Izolacja wątku      |
+-----------------------+               +-----------------------+
          |                                       |
          +-------------------+-------------------+
                              |
                              v
              +-------------------------------+
              |     Cache 2. Poziomu (L2)     |
              | (EntityManagerFactory / Serwer) |
              | - Współdzielony w aplikacji   |
              +-------------------------------+
                              |
                              v
              +-------------------------------+
              |   Relacyjna Baza Danych (SQL) |
              +-------------------------------+
```

### 4.1. Cache Pierwszego Poziomu (Level 1 Cache – L1)
- **Zasięg i czas życia:** Ściśle powiązany z pojedynczym obiektem `EntityManager` i trwającym **kontekstem trwałości** (Persistence Context). Typowo żyje w granicach jednej transakcji bazodanowej lub jednego żądania HTTP.
- **Włączanie / Wyłączanie:** **Jest wbudowany na stałe i NIE MOŻNA go wyłączyć.** Jest integralną częścią specyfikacji JPA.
- **Zasada działania:**
  1. Gdy aplikacja wywołuje `em.find(Klient.class, 10)`, `EntityManager` sprawdza najpierw bufor L1.
  2. Jeśli encja o identyfikatorze 10 znajduje się w L1, zwracana jest natychmiast z pamięci podręcznej (brak zapytania SQL do bazy).
  3. Jeśli jej nie ma, wykonywany jest SQL `SELECT`, a pobrana encja jest rejestrowana w buforze L1.
  4. Każde kolejne odpytanie o ID 10 w tej samej sesji zwróci **dokładnie tę samą instancję z pamięci** (`instancja1 == instancja2`).
- **Główny cel L1:**
  - Zapewnienie spójności tożsamości obiektów w ramach transakcji (*Repeatable Read* w pamięci).
  - Obsługa mechanizmu **Dirty Checking** (automatyczne wykrywanie zmian pól obiektów managed).
  - Unikanie powielania zapytań SQL w obrębie tej samej operacji biznesowej.

### 4.2. Cache Drugiego Poziomu (Level 2 Cache – L2)
- **Zasięg i czas życia:** Powiązany z obiektem `EntityManagerFactory`. Jest to **globalny bufor współdzielony pomiędzy wszystkimi instancjami `EntityManager`**, wszystkimi wątkami, transakcjami i sesjami w całej aplikacji.
- **Włączanie / Wyłączanie:** **Jest domyślnie wyłączony i w pełni opcjonalny.** Wymaga jawnej konfiguracji w `persistence.xml` oraz dostarczenia wyspecjalizowanego dostawcy bufora (np. Infinispan, Ehcache, Hazelcast).
- **Zasada działania:**
  1. Wątek #1 pobiera klienta o ID 10 z bazy. Po zakończeniu transakcji obiekt trafia do bufora L2.
  2. Wątek #2 (inny użytkownik, zupełnie nowa transakcja) wywołuje `em.find(Klient.class, 10)`.
  3. `EntityManager` wątku #2 nie znajduje encji w swoim lokalnym L1, więc odpytuje wspólny cache L2.
  4. Obiekt zostaje odnaleziony w L2 i przekazany do wątku #2 **bez konieczności wysyłania zapytania SQL do bazy danych**.
- **Wyzwania i wady L2:**
  - **Spójność danych (*Cache Invalidation*):** Jeśli zewnętrzny proces zmodyfikuje bazę danych z pominięciem JPA, cache L2 będzie serwował nieświeże dane (*stale data*).
  - W środowisku klastrowym (wiele serwerów aplikacji) wymaga skomplikowanego cache'u rozproszonego (*distributed cache*) synchronizowanego przez sieć.
  - Najlepiej sprawdza się dla danych rzadko zmienianych, a często odczytywanych (słowniki, cenniki, konfiguracje).

### 4.3. Cache Zapytań (Query Cache)
- Stanowi uzupełnienie L2. Nie buforuje całych encji, lecz **zestawy identyfikatorów będące wynikiem konkretnego zapytania JPQL** wraz z parametrami (np. *"ID książek autora o nazwisku Lem"*).
- Przy ponownym wykonaniu identycznego zapytania JPA pobiera z Query Cache listę ID, a same encje pobiera z bufora L2.

---

## 5. Wzorzec POST-REDIRECT-GET (PRG)

### 5.1. Istota problemu wielokrotnego przesłania (Duplicate Form Submission):
- Gdy formularz przesyłany jest metodą **HTTP POST** (np. złożenie zamówienia, przelew), serwer wykonuje modyfikację stanu (np. obciążenie konta bankowego kwotą 100 zł).
- W tradycyjnym, błędnym modelu serwer natychmiast odsyła stronę HTML z kodem `200 OK`.
- Przeglądarka internetowa zapamiętuje, że ostatnią operacją było żądanie POST.
- **Skutek awarii:** Jeśli użytkownik naciśnie klawisz **F5 (Odśwież)**, kliknie przycisk "Wstecz/Dalej" lub dwukrotnie kliknie przycisk wysłania, przeglądarka wyświetli monit o ponowne przesłanie formularza i **wyśle powtórnie identyczne żądanie POST**.
- Serwer wykona operację po raz drugi: nastąpi podwójne pobranie opłaty i zdublowanie zamówienia w bazie.

### 5.2. Rozwiązanie krok po kroku (Idiom PRG):
PRG narzuca ścisłą separację operacji modyfikującej stan od operacji wyświetlającej wynik:

```
Przeglądarka (Użytkownik)                                  Serwer Aplikacji
            |                                                     |
            | -------- 1. POST /zloz-zamowienie (dane) ---------> | Zapis w bazie danych,
            |                                                     | realizacja transakcji.
            | <------- 2. HTTP 302/303 Redirect ----------------- | 
            |             Nagłówek Location: /sukces?id=123       | (Brak treści HTML!)
            |                                                     |
            | -------- 3. GET /sukces?id=123 -------------------> | Bezpieczny odczyt
            |                                                     | strony podsumowania.
            | <------- 4. HTTP 200 OK (widok podsumowania) ------ |
```

1. **POST:** Klient wysyła dane formularza. Serwer wykonuje logikę biznesową i zapisuje zmiany w bazie.
2. **REDIRECT:** Serwer **nie generuje widoku HTML**, lecz odsyła nagłówek przekierowania **HTTP 302 Found** lub **HTTP 303 See Other** ze wskazaniem adresu docelowego: `Location: /sukces?id=123`.
3. **GET:** Przeglądarka natychmiast automatycznie wysyła bezpieczne, idempotentne zapytanie **HTTP GET** pod adres `/sukces?id=123`.
4. **Prezentacja (200 OK):** Serwer w odpowiedzi na GET odsyła stronę z potwierdzeniem.
5. **Dlaczego to działa:** Ostatnim wpisem w historii przeglądarki jest **GET**. Naciśnięcie klawisza F5 (Odśwież) powoduje jedynie ponowne, nieszkodliwe odczytanie strony podsumowania – nie grozi ponownym wykonaniem transakcji biznesowej.
- **W Jakarta Faces (JSF):** Standardowo JSF używa żądań postback na tę samą stronę. Aby włączyć PRG, do zwracanego wyniku nawigacji dodaje się parametr `faces-redirect=true`.

---

## 6. Zestawienie Wszystkich Cykli Życia w Jakarta EE
*Pytanie przekrojowe – wykładowca bardzo często wymaga porównania cykli życia różnych komponentów platformy:*

### 6.1. Cykl życia Servletu:
Cykl kontrolowany w całości przez kontener webowy:
1. **Załadowanie i Inicjalizacja (jednorazowa):** Kontener ładuje klasę servletu, tworzy instancję i wywołuje metodę `init(ServletConfig)`.
2. **Obsługa żądań (wielokrotna, współbieżna):** Dla każdego zapytania HTTP kontener przydziela wątek i wywołuje metodę `service()`, która deleguje wykonanie do `doGet()`, `doPost()`, `doPut()`, `doDelete()` itd.
3. **Zniszczenie (jednorazowe):** Przy zamykaniu aplikacji kontener wywołuje metodę `destroy()`.
- *Ważne:* Jedna instancja servletu obsługuje równolegle wiele wątków!

---

### 6.2. Cykl życia Encji JPA (4 Stany):
Obiekt encji przechodzi pomiędzy czterema stanami pod kontrolą `EntityManager`:

```
                 +-------------------+
                 |    New/Transient  | (utworzony przez new, brak ID, poza kontekstem)
                 +-------------------+
                           |
                           | persist()
                           v
+----------------> +-------------------+ ---- remove() ----> +-------------------+
| find(), query,   |      Managed      |                     |      Removed      |
| merge()          | (w kontekście L1, |                     | (do usunięcia     |
|                  |  dirty checking)  |                     |  przy commicie)   |
|                  +-------------------+                     +-------------------+
|                            |                                         |
|                   detach(),|                                         | commit()
|                   clear(), | commit()                                v
|                   close()  v                                [ Usunięcie z bazy ]
+----------------- +-------------------+
                   |     Detached      | (posiada ID, ale poza aktywnym kontekstem)
                   +-------------------+
```

---

### 6.3. Cykle życia komponentów EJB (Session Beans):

#### A. Beany Bezstanowe (`@Stateless`) i Singletony (`@Singleton`):
1. **Utworzenie instancji i wstrzyknięcie zależności (DI).**
2. **`@PostConstruct`:** Jednorazowa inicjalizacja po wstrzyknięciu referencji.
3. **Stan gotowości (*Ready*):** Obsługa wywołań klientów (dla Stateless instancja w puli; dla Singleton jedna instancja dla całej aplikacji).
4. **`@PreDestroy`:** Wywoływana przed usunięciem instancji przez kontener.
- *Różnica:* W `@Stateless` metody `@PostConstruct` i `@PreDestroy` wykonują się wielokrotnie dla każdej instancji tworzonej i usuwanej z puli. W `@Singleton` wykonują się dokładnie raz w całym życiu aplikacji.

#### B. Beany Stanowe (`@Stateful`) – Proces Pasywacji:
Bean stanowy jest przypisany do jednego klienta i przechowuje stan sesji:
1. Utworzenie + DI + `@PostConstruct` $\rightarrow$ Stan gotowości (*Ready*).
2. **Pasywacja (`@PrePassivate`):** Gdy serwerowi brakuje pamięci RAM, kontener zrzuca stan beana na dysk twardy (serializacja). Bean przechodzi w stan **Pasywny (*Passive*)**.
3. **Aktywacja (`@PostActivate`):** Gdy klient ponownie wywoła metodę, kontener wczytuje stan beana z dysku z powrotem do pamięci RAM.
4. **Zakończenie życia:** Wywołanie metody biznesowej z adnotacją `@Remove`, timeout sesji lub zatrzymanie serwera (`@PreDestroy`).

---

### 6.4. Cykl życia Żądania w Jakarta Faces (JSF – 6 Faz):
Każde żądanie skierowane do `FacesServlet` przechodzi przez sekwencję 6 faz:

```
[ Żądanie HTTP ]
       |
       v
1. Restore View ---------> Odtworzenie lub utworzenie drzewa komponentów widoku.
       |
       v
2. Apply Request Values -> Pobranie surowych wartości z żądania do komponentów UI.
       |
       v
3. Process Validations --> Konwersja typów i walidacja wartości.
       |                   * JEŚLI BŁĄD WALIDACJI: Natychmiastowy skok do Fazy 6!
       v
4. Update Model Values --> Przepisanie poprawnych wartości do pól Backing Beans.
       |
       v
5. Invoke Application ---> Wywołanie metod akcji biznesowych (zapis, nawigacja).
       |
       v
6. Render Response ------> Wygenerowanie kodu HTML i odesłanie odpowiedzi do klienta.
```

---

### 6.5. Cykl życia komponentów CDI:
1. Kontener sprawdza kwalifikację klasy i rozwiązuje powiązania zależności.
2. Tworzona jest instancja (konstruktor) i wstrzykiwane są zależności (`@Inject`).
3. Wywoływana jest metoda oznaczona adnotacją **`@PostConstruct`**.
4. Obiekt realizuje swoje zadania w ramach przypisanego kontekstu (np. żądania, sesji, aplikacji).
5. Gdy zasięg kontekstu ulega zamknięciu (np. koniec żądania HTTP, wylogowanie sesji, zamknięcie aplikacji), kontener wywołuje metodę **`@PreDestroy`** i niszczy obiekt.
