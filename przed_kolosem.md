# ⏱️ Przed Kolosem: Esencja Teorii Jakarta EE

> Skondensowana „ściągawka” teorii na kolokwium.  
> Dokładnie **3 kluczowe zagadnienia na każdy wykład**, każde opisane w **maksymalnie 2–3 zdaniach** – idealne do powtórki na 15 minut przed wejściem na salę.

---

## 🏛️ Wykład 1: Platforma Jakarta EE

### 1. Zakres `provided` w Maven (`pom.xml`)
Zakres `provided` oznacza, że biblioteka (np. Jakarta API) jest potrzebna wyłącznie na etapie kompilacji i testów, natomiast w środowisku uruchomieniowym dostarcza ją sam serwer aplikacji. Dzięki temu wynikowe archiwum (`.war` lub `.ear`) jest znacznie lżejsze i nie dochodzi do konfliktów wersji z bibliotekami kontenera.

### 2. Architektura wielowarstwowa i kontenery
Platforma dzieli aplikację na warstwy (klienta, sieciową, biznesową i danych), którymi zarządzają wyspecjalizowane kontenery (Web Container oraz EJB Container). Kontenery automatyzują zarządzanie cyklem życia komponentów, transakcjami, bezpieczeństwem i wielowątkowością, zdejmując te niskopoziomowe obowiązki z programisty.

### 3. Profile platformy i formaty archiwów (`.war` vs `.ear`)
Profile definiują podzbiory specyfikacji dopasowane do wielkości systemu – Core Profile dla lekkich mikrousług, Web Profile dla typowych aplikacji internetowych oraz Full Platform dla kompletnych systemów korporacyjnych. Aplikacje pakowane są w pojedyncze archiwa `.war` (standard dla Web Profile) lub złożone archiwa `.ear`, które mogą grupować wiele modułów webowych i biznesowych w ramach Full Platform.

---

## 🌐 Wykład 2: Jakarta Servlet

### 1. Cykl życia servletu
Cykl życia obejmuje załadowanie klasy, powołanie instancji, jednorazową inicjalizację metodą `init()`, wielokrotną obsługę przychodzących żądań metodą `service()` oraz zniszczenie przez `destroy()`. Za cały ten proces odpowiada kontener serwletów, który w typowym scenariuszu utrzymuje tylko jedną instancję danego serwletu na całą aplikację.

### 2. Współbieżność i bezpieczeństwo wątkowe
Ponieważ jedna instancja serwletu obsługuje współbieżnie wielu klientów w osobnych wątkach, servlet z definicji nie jest bezpieczny wątkowo. Stan serwletu (pola klasy) musi być ściśle chroniony lub niemutowalny, a wszelkie dane powiązane z przetwarzanym żądaniem muszą być przechowywane wyłącznie w zmiennych lokalnych metod.

### 3. Łańcuch filtrów (`FilterChain`) i nasłuchiwacze (`Listeners`)
Filtry tworzą łańcuch przetwarzania, który może badać, modyfikować lub blokować żądania i odpowiedzi przed dotarciem do serwletu lub po wygenerowaniu odpowiedzi. Nasłuchiwacze reagują asynchronicznie na kluczowe zdarzenia cyklu życia kontenera, takie jak start/stop aplikacji (`ServletContextListener`) czy utworzenie sesji użytkownika (`HttpSessionListener`).

---

## 💉 Wykład 3: Jakarta Contexts and Dependency Injection (CDI)

### 1. IoC (Inversion of Control) a DI (Dependency Injection)
Inversion of Control to ogólna zasada architektoniczna („nie dzwoń do nas, my zadzwonimy do ciebie”), w której sterowanie przepływem programu przekazuje się frameworkowi. Dependency Injection to konkretna realizacja IoC, polegająca na automatycznym dostarczaniu gotowych obiektów zależnych przez kontener z zewnątrz, zamiast ich bezpośredniego tworzenia operatorem `new`.

### 2. Wstrzykiwanie mniejszego zasięgu do większego i Client Proxy
Wstrzyknięcie komponentu o krótszym cyklu życia (np. `@RequestScoped`) do komponentu o dłuższym cyklu życia (np. `@ApplicationScoped`) jest możliwe wyłącznie dzięki wstrzyknięciu dynamicznego pośrednika (`Client Proxy`). Proxy przechwytuje każde wywołanie metody i dynamicznie odpytuje menedżer kontekstów CDI, przekierowując wywołanie do właściwej instancji powiązanej z wątkiem bieżącego żądania (`ThreadLocal`).

### 3. Wstrzykiwanie przez interfejs
Wstrzykiwanie interfejsu zamiast konkretnej klasy implementacyjnej realizuje zasadę odwrócenia zależności (DIP) i drastycznie redukuje sprzężenie (*loose coupling*). Umożliwia to elastyczną podmianę implementacji w kontenerze (np. zależnie od środowiska lub kwalifikatorów) oraz bezproblemowe podstawianie atrap (mocków) w testach jednostkowych.

---

## 🗄️ Wykład 4: Jakarta Persistence (JPA)

### 1. Cztery stany cyklu życia encji
Encja może znajdować się w stanie Nowym/Tymczasowym (obiekt w pamięci bez klucza w bazie), Zarządzanym (reprezentowany w pamięci L1 i powiązany z sesją), Odłączonym (posiada ID bazodanowe, ale sesja została zamknięta) lub Usuniętym (oznaczony do usunięcia z bazy). Tylko obiekty w stanie Zarządzanym podlegają automatycznemu wykrywaniu modyfikacji pól (*dirty checking*) i synchronizacji z bazą podczas zatwierdzania transakcji.

### 2. Poziomy pamięci podręcznej (Cache L1 vs L2)
Pamięć podręczna pierwszego poziomu (L1 / Persistence Context) jest obowiązkowa, bezpłatna wydajnościowo i ściśle powiązana z czasem życia pojedynczego obiektu `EntityManager` oraz jednego wątku. Pamięć podręczna drugiego poziomu (L2) jest opcjonalna, współdzielona globalnie przez całą aplikację na poziomie `EntityManagerFactory` i buforuje stan encji pomiędzy różnymi sesjami użytkowników.

### 3. Strona zarządzająca relacją (`mappedBy`)
W relacjach dwukierunkowych strona, która fizycznie przechowuje kolumnę z kluczem obcym w bazie danych, jest stroną zarządzającą (*owning side*). Druga strona jest stroną odwrotną (*inverse side*) i musi zawierać atrybut `mappedBy`, który informuje dostawcę JPA, że modyfikacje tej kolekcji w kodzie Javy nie będą skutkować aktualizacją kluczy obcych w bazie.

---

## 🎭 Wykład 5: Jakarta Faces (JSF)

### 1. Sześć faz cyklu życia żądania Faces
Cykl przetwarzania żądania Faces składa się kolejno z faz: Odtworzenie widoku, Pobranie wartości żądania, Przetwarzanie walidacji, Aktualizacja wartości modelu, Wywołanie logiki aplikacji oraz Renderowanie odpowiedzi. W przypadku wykrycia błędów konwersji lub walidacji normalny przepływ jest przerywany i kontener natychmiast przeskakuje do ostatniej fazy, renderując formularz z komunikatami błędów.

### 2. Problem podwójnego zatwierdzenia formularza i wzorzec PRG
Wysłanie formularza żądaniem POST pozostawia przeglądarkę na odpowiedzi POST, przez co zwykłe odświeżenie strony (F5) powoduje ponowne wysłanie tych samych danych i zduplikowanie transakcji biznesowej. Standardowym rozwiązaniem jest wzorzec Post-Redirect-Get (PRG), w którym serwer po przetworzeniu danych odpowiada statusem przekierowania (HTTP 302/303), zmuszając przeglądarkę do pobrania strony wynikowej bezpieczną metodą GET.

### 3. Zasięg `@ViewScoped`
Komponent w zasięgu `@ViewScoped` istnieje dokładnie tak długo, jak długo użytkownik przebywa na tej samej stronie i wchodzi z nią w interakcję przez wywołania AJAX lub postbacki. Jest bezpieczniejszy pamięciowo niż `@SessionScoped`, a jednocześnie rozwiązuje problem `@RequestScoped`, w którym stan formularza byłby bezpowrotnie niszczony po każdym pojedynczym kliknięciu.

---

## 🔌 Wykład 6: Jakarta RESTful Web Services

### 1. Model dojrzałości Richardsona i Poziom 0 (The Swamp of POX)
Model dzieli usługi sieciowe na 4 poziomy: Poziom 0 (jedno URI i metoda POST, czyli RPC ukryte w HTTP – tzw. "The Swamp of POX"), Poziom 1 (zasoby z osobnymi adresami URI), Poziom 2 (właściwe czasowniki HTTP i kody statusu) oraz Poziom 3 (hipermedia HATEOAS). Poziom 0 jest krytykowany za traktowanie HTTP wyłącznie jako ślepego tunelu transmisyjnego z całkowitym zignorowaniem standardów architektury sieci Web.

### 2. Bezpieczeństwo i idempotencja metod HTTP (POST vs PUT)
Metody bezpieczne (GET, HEAD) gwarantują brak jakichkolwiek modyfikacji stanu na serwerze, natomiast metody idempotentne (GET, PUT, DELETE) dają identyczny stan końcowy zasobu niezależnie od liczby powtórzonych wywołań. POST jest nieidempotentny i służy do tworzenia nowego zasobu podrzędnego (gdzie serwer sam nadaje ID), podczas gdy PUT jest idempotentny i służy do pełnego nadpisania zasobu pod z góry znanym adresem URI.

### 3. HATEOAS (Hipermedia jako motor stanu aplikacji)
HATEOAS to 3. poziom modelu Richardsona, w którym serwer zwraca w odpowiedzi nie tylko surowe dane, ale także dynamiczny zestaw hiperłączy prowadzących do dozwolonych kolejnych akcji. Dzięki temu klient nie musi znać struktury adresów URL na sztywno, lecz dynamicznie odkrywa dozwolone przejścia stanu aplikacji w sposób analogiczny do użytkownika klikającego linki na stronie WWW.

---

## ⚙️ Wykład 7: Jakarta Enterprise Beans (EJB)

### 1. Typy beanów sesyjnych: Stateless vs Stateful vs Singleton
Bean bezstanowy (Stateless) jest pobierany ze współdzielonej puli kontenera i nie przechowuje stanu klienta między wywołaniami, co daje najwyższą skalowalność. Bean stanowy (Stateful) jest dedykowany jednemu klientowi na czas trwania konwersacji, natomiast Singleton istnieje w dokładnie jednej instancji w całej aplikacji i jest współdzielony przez wszystkie wątki.

### 2. Pasywacja i aktywacja w Stateful Session Beans
Aby ograniczyć zużycie pamięci operacyjnej, kontener może przenieść nieaktywny bean stanowy na dysk w procesie pasywacji (`@PrePassivate`), o ile wszystkie jego pola są serializowalne. Gdy klient ponawia interakcję, kontener przywraca stan obiektu z dysku do pamięci operacyjnej w procesie aktywacji (`@PostActivate`).

### 3. Zarządzanie współbieżnością w Singleton EJB
Współbieżnością Singletona może sterować sam kontener (CMT) za pomocą blokad odczytu i zapisu na poziomie metod. Blokada `LockType.READ` zezwala na jednoczesne, równoległe wykonywanie metody przez wiele wątków, natomiast domyślna blokada `LockType.WRITE` blokuje cały komponent na wyłączność jednego wątku na czas modyfikacji danych.

---

## 🛡️ Wykład 8: Uwierzytelnianie i Autoryzacja

### 1. Uwierzytelnianie (AuthN) a Autoryzacja (AuthZ) i model RBAC
Uwierzytelnianie polega na zweryfikowaniu tożsamości podmiotu (odpowiedź na pytanie „Kim jesteś?”), natomiast autoryzacja sprawdza, czy uwierzytelniony podmiot posiada uprawnienia do zasobu (odpowiedź na pytanie „Co ci wolno?”). Model RBAC (Role-Based Access Control) opiera kontrolę dostępu na przypisaniu użytkownikom abstrakcyjnych ról biznesowych, a rolom konkretnych uprawnień do operacji w systemie.

### 2. Przepływy protokołu OAuth 2.0 / 2.1
Przepływ Authorization Code z rozszerzeniem PKCE jest obecnie jedynym rekomendowanym standardem dla aplikacji klienckich (SPA i mobilnych), zapobiegającym kradzieży kodu autoryzacyjnego w niezaufanym środowisku. Przepływ Client Credentials służy wyłącznie do bezpośredniej integracji serwer-serwer (M2M) bez udziału użytkownika, natomiast podatny na ataki przepływ Implicit został w standardzie OAuth 2.1 całkowicie usunięty.

### 3. Struktura i bezstanowa weryfikacja tokena JWT
Token JWT składa się z trzech części zakodowanych w Base64Url i rozdzielonych kropkami: nagłówka (typ i algorytm), ładunku (zestaw oświadczeń/danych *claims*) oraz podpisu kryptograficznego. Serwer weryfikuje integralność tokena czysto matematycznie za pomocą klucza publicznego lub wspólnego sekretu, dzięki czemu uwierzytelnianie jest całkowicie bezstanowe i nie obciąża bazy danych sesjami.
