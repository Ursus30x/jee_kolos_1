# 07. Jakarta Enterprise Beans (EJB) – Teoria na Kolokwium

## 1. Czym są Enterprise Beans (dawniej EJB)?
- **Definicja:** Komponenty poziomu biznesowego zarządzane w całości przez **kontener EJB** na serwerze aplikacji Jakarta EE.
- **Rola kontenera EJB:**
  - Kontener otacza komponenty EJB powłoką zarządczą (interceptor proxy) i automatycznie dostarcza programiście zaawansowane usługi infrastrukturalne enterprise bez konieczności ich ręcznego kodowania:
    1. **Deklaratywne zarządzanie transakcjami (CMT – Container-Managed Transactions):** kontener automatycznie otwiera, zatwierdza lub wycofuje transakcje bazodanowe na granicach wywołań metod beana.
    2. **Deklaratywne bezpieczeństwo:** kontener weryfikuje uprawnienia i role użytkownika przed dopuszczeniem do metody.
    3. **Równoważenie obciążenia (Load Balancing) i Klastrowanie:** kontener potrafi rozdzielać wywołania na wiele węzłów klastra serwerów.
    4. **Zarządzanie pulą instancji (Pooling):** optymalizacja zużycia pamięci i zasobów serwera.
    5. **Zdalne wywołania procedur (Remoting):** obsługa klientów działających w oddzielnych maszynach wirtualnych (oddzielne procesy, inne serwery, aplikacje okienkowe).

---

## 2. Rodzaje komponentów EJB
Specyfikacja dzieli komponenty EJB na dwie główne kategorie:

```
                          +------------------------+
                          | Enterprise Beans (EJB) |
                          +------------------------+
                                      |
         +----------------------------+----------------------------+
         |                                                         |
         v                                                         v
+-------------------------------+                         +-----------------------------+
|    Beany Sesyjne (Session)    |                         | Beany Komunikatowe (MDB)    |
| (synchroniczna obsługa logiki)|                         | (asynchroniczne, JMS)       |
+-------------------------------+                         +-----------------------------+
         |
         +--------------------+--------------------+
         |                    |                    |
         v                    v                    v
  +--------------+     +--------------+     +--------------+
  |  Stateless   |     |   Stateful   |     |  Singleton   |
  | (bezstanowe) |     |  (stanowe)   |     | (jedynaki)   |
  +--------------+     +--------------+     +--------------+
```

### 1. Beany Sesyjne (Session Beans):
Wykonują operacje biznesowe na żądanie konkretnego klienta. Dzielą się na:
- **Beany Bezstanowe (`@Stateless`):**
  - **Nie zachowują stanu konwersacyjnego z klientem** pomiędzy kolejnymi wywołaniami metod. Wszelkie dane potrzebne do wykonania metody muszą być przekazane w jej argumentach lub pobrane z bazy.
  - Serwer utrzymuje pulę (*pool*) identycznych instancji beana. Każde żądanie od dowolnego klienta może zostać obsłużone przez dowolną wolną instancję pobraną z puli.
  - **Zaleta:** Najwyższa wydajność i doskonała skalowalność horyzontalna.
- **Beany Stanowe (`@Stateful`):**
  - **Utrzymują dedykowany stan konwersacyjny z jednym konkretnym klientem** w polach klasy pomiędzy kolejnymi wywołaniami metod w ramach jednej sesji biznesowej (np. zawartość wieloetapowego koszyka zakupowego, kreator wniosku kredytowego).
  - Instancja beana jest zarezerwowana wyłącznie dla danego klienta.
  - **Koszt:** Większe zużycie zasobów serwera oraz konieczność zarządzania pasywacją/aktywacją.
- **Beany Singleton (`@Singleton`):**
  - W obrębie całej aplikacji istnieje **dokładnie jedna instancja beana**, współdzielona jednocześnie przez wszystkich klientów i wszystkie wątki.
  - Umożliwia inicjalizację danych przy starcie aplikacji (`@Startup`) oraz definiowanie zależności kolejności startu (`@DependsOn`).
  - Domyślnie zoptymalizowany pod kątem bezpiecznego współbieżnego dostępu wielowątkowego.

### 2. Beany Sterowane Komunikatami (`Message-Driven Beans` – MDB):
- Komponenty wykonujące zadania **asynchronicznie**.
- Nie posiadają interfejsów biznesowych i **nie mogą być wywoływane bezpośrednio przez klienta**.
- Działają jako konsumenci / nasłuchiwacze wiadomości z systemów kolejkowych (JMS – Java Message Service). Kontener uruchamia MDB w momencie nadejścia nowego komunikatu do monitorowanej kolejki lub tematu.

---

## 3. Interfejsy biznesowe EJB (Widoki klienta)
Sesyjny bean EJB może udostępniać swoją funkcjonalność klientom za pośrednictwem trzech różnych widoków:

| Rodzaj interfejsu | Położenie klienta i środowisko | Mechanizm wywołania | Własności i konsekwencje |
| :--- | :--- | :--- | :--- |
| **Interfejs Lokalny (`@Local`)** | Klient działa w **tej samej maszynie wirtualnej (JVM)** co serwer aplikacji (np. moduł webowy WAR w tej samej aplikacji). | Bezpośrednie wywołanie metody w pamięci (mechanizm referencji – *pass-by-reference*). | **Brak narzutu sieciowego.** Bardzo wysoka wydajność. Encje JPA mogą korzystać z mechanizmu opóźnionego ładowania (*lazy loading*). |
| **Interfejs Zdalny (`@Remote`)** | Klient może działać w **innej maszynie wirtualnej JVM** lub na fizycznie odrębnym komputerze w sieci (np. zewnętrzna aplikacja okienkowa, inny serwer aplikacji). | Zdalne wywołanie procedur przez sieć (RMI/IIOP – *Remote Method Invocation*). | - Wszystkie parametry i zwracane obiekty **muszą implementować interfejs `Serializable`** (są kopiowane przez sieć – *pass-by-value*).<br>- **Brak możliwości lazy loading:** encje przekazywane przez interfejs zdalny muszą mieć wszystkie potrzebne pola pobrane natychmiast (*eager fetch*), bo po drugiej stronie łącza kontekst trwałości JPA nie istnieje.<br>- Definicje interfejsów i encji muszą być wydzielone do osobnego archiwum JAR współdzielonego między serwerem a klientem. |
| **Widok bez interfejsu (*No-interface view* / `@LocalBean`)** | Klient w tej samej JVM. | Podobnie jak lokalny, ale klient odwołuje się bezpośrednio do typu klasy samego beana. | Upraszcza wytwarzanie oprogramowania w prostych architekturach – brak konieczności tworzenia sztucznych interfejsów jednorazowego użytku. |

---

## 4. Cykl życia komponentów EJB
Cykl życia jest w całości sterowany przez kontener serwera.

### A. Cykl życia Beana Bezstanowego (`@Stateless`) i Singletona (`@Singleton`):
```
[ Nie istnieje ] ---> Utworzenie instancji + DI ---> @PostConstruct ---> [ Gotowy do użycia (Ready) ]
                                                                                   |
[ Nie istnieje ] <---------------------------------- @PreDestroy <-----------------+
```
1. **Inicjalizacja:** Kontener tworzy instancję beana, wstrzykuje wszystkie wymagane zależności (DI), a następnie wywołuje metodę oznaczoną adnotacją `@PostConstruct`. Bean trafia do puli i jest gotowy do obsługi żądań (`Ready`).
2. **Niszczenie:** Przy wycofywaniu instancji z puli (np. zmniejszenie obciążenia serwera lub shutdown) kontener wywołuje metodę `@PreDestroy`, po czym obiekt podlega czyszczeniu pamięci (Garbage Collector).

### B. Cykl życia Beana Stanowego (`@Stateful`) – Mechanizm Pasywacji:
Ze względu na to, że bean stanowy przechowuje stan użytkownika w pamięci, a liczba klientów może rosnąć, kontener EJB posiada mechanizm zarządzania pamięcią RAM:

```
[ Nie istnieje ] ---> Utworzenie + DI ---> @PostConstruct ---> [ Gotowy (Ready) ]
                                                                   |       ^
                                                   @PrePassivate   |       |   @PostActivate
                                                   (Zrzut na dysk) |       |   (Odczyt z dysku)
                                                                   v       |
                                                               [ Pasywny (Passive) ]
                                                                   |
[ Nie istnieje ] <--- @PreDestroy / @Remove <----------------------+ (lub timeout)
```
- **Pasywacja (*Passivation*):** Gdy serwerowi brakuje pamięci RAM lub klient przez dłuższy czas nie wykonuje operacji, kontener wywołuje na beanie metodę `@PrePassivate` i **serializuje stan beana (zrzuca go z pamięci RAM na dysk twardy)**, zwalniając pamięć operacyjną.
- **Aktywacja (*Activation*):** Gdy klient ponownie wywoła metodę tego beana, kontener odszukuje jego stan na dysku, wczytuje go z powrotem do pamięci RAM i wywołuje metodę `@PostActivate`.
- **Zakończenie życia:** Następuje, gdy klient wywoła metodę biznesową oznaczoną adnotacją `@Remove`, wygaśnie limit czasu bezczynności sesji (*session timeout*) lub nastąpi zatrzymanie aplikacji (`@PreDestroy`).

---

## 5. Usługa Katalogowa JNDI (Java Naming and Directory Interface)
- **Definicja:** Ustandaryzowana usługa katalogowa wbudowana w środowisko Jakarta EE, umożliwiająca komponentom rejestrowanie i lokalizowanie zasobów serwerowych oraz innych komponentów za pomocą unikalnych nazw hierarchicznych (ścieżek URL).
- Stosowana wtedy, gdy nie można użyć automatycznego wstrzykiwania zależności (`@Inject` / `@EJB`), np. w tradycyjnych klasach pomocniczych lub w odległych klientach zewnętrznych (`InitialContext.doLookup(...)`).

### Ustandaryzowane przestrzenie nazw JNDI:
1. `java:global[/nazwa_aplikacji]/nazwa_modulu/nazwa_beana[!interfejs]`
   - Wyszukiwanie globalne – pozwala zlokalizować bean EJB z dowolnego modułu w obrębie całego serwera lub przez klienta zewnętrznego z innej maszyny wirtualnej.
2. `java:app/nazwa_modulu/nazwa_beana[!interfejs]`
   - Wyszukiwanie na poziomie aplikacji – widoczne dla wszystkich modułów (WAR, EJB-JAR) wchodzących w skład tego samego archiwum EAR.
3. `java:module/nazwa_beana[!interfejs]`
   - Wyszukiwanie na poziomie modułu – widoczne wyłącznie wewnątrz tego samego modułu (np. wewnątrz danego pliku WAR lub EJB-JAR).

---

## 6. Zarządzanie współbieżnością w EJB Singleton
Ponieważ bean `@Singleton` posiada tylko jedną instancję dla całej aplikacji, serwer musi koordynować równoległy dostęp wielu wątków.

### Dwa tryby zarządzania synchronizacją (`@ConcurrencyManagement`):
1. **Zarządzana przez programistę (Bean-Managed Concurrency – `BMC`):**
   - Programista sam odpowiada za thread-safety, posługując się niskopoziomowymi mechanizmami Javy (słowo kluczowe `synchronized`, zmienne atomowe, obiekty `java.util.concurrent.locks`).
2. **Zarządzana przez kontener (Container-Managed Concurrency – `CMC` – Domyślna):**
   - Kontener automatycznie nakłada blokady na metody beana w oparciu o adnotację `@Lock`.
   - **Dwa rodzaje blokad (`LockType`):**
     - **`@Lock(LockType.READ)` (Blokada współdzielona odczytu):** Pozwala na równoczesne, równoległe wykonywanie danej metody przez **wielu klientów naraz**, o ile żaden wątek nie modyfikuje stanu beana w metodzie z blokadą WRITE. Stosowana dla bezpiecznych metod pobierających dane.
     - **`@Lock(LockType.WRITE)` (Blokada wyłączna zapisu):** Zapewnia wyłączność – w danej chwili metodę może wykonywać **dokładnie jeden wątek**. Wszystkie inne wątki (zarówno czytające, jak i piszące) są blokowane i czekają w kolejce.
     - **Ważna uwaga:** W trybie kontenerowym, **jeśli programista nie poda adnotacji `@Lock`, domyślną blokadą jest `@Lock(LockType.WRITE)`** (kontener z ostrożności domyślnie blokuje współbieżny dostęp do każdej metody).
