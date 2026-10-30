# Zestaw Pytań i Odpowiedzi Egzaminacyjnych – Jakarta EE

Niniejszy dokument zawiera szczegółowe opracowania odpowiedzi na 18 pytań z materiału teoretycznego wykładów platformy Jakarta EE. Opracowanie przygotowano z naciskiem na teorię, koncepcje architektoniczne, cykle życia i porównania, bez kodu implementacyjnego.

---

### Spis pytań:
1. [Po co dawać `provided` w `pom.xml`?](#pytanie-1-po-co-dawać-provided-w-pomxml)
2. [Jaki jest cykl życia servletu?](#pytanie-2-jaki-jest-cykl-życia-servletu)
3. [Czego lepiej używać – adnotacji czy konfiguracji w XMLu?](#pytanie-3-czego-lepiej-używać--adnotacji-czy-konfiguracji-w-xmlu)
4. [Czym jest IoC (Inversion of Control), a czym jest Dependency Injection (DI)?](#pytanie-4-czym-jest-ioc-inversion-of-control-a-czym-jest-dependency-injection-di)
5. [Do czego powinny służyć wszystkie rodzaje zapytań HTTP (w szczególności różnica między PUT a POST)?](#pytanie-5-do-czego-powinny-służyć-wszystkie-rodzaje-zapytań-http-w-szczególności-różnica-między-put-a-post)
6. [Dlaczego działa wstrzykiwanie mniejszego zasięgu do większego? W jaki sposób mniejszy zasięg „wchodzi” do większego i działa?](#pytanie-6-dlaczego-działa-wstrzykiwanie-mniejszego-zasięgu-do-większego-w-jaki-sposób-mniejszy-zasięg-wchodzi-do-większego-i-działa)
7. [Czemu wstrzykiwanie przez interfejs jest lepsze?](#pytanie-7-czemu-wstrzykiwanie-przez-interfejs-jest-lepsze)
8. [Działanie mechanizmu PROXY](#pytanie-8-działanie-mechanizmu-proxy)
9. [Po co jest „lazy getter”?](#pytanie-9-po-co-jest-lazy-getter)
10. [Model dojrzałości usług REST (rozszerzony) + przykłady poziomów + dlaczego SWAMP?](#pytanie-10-model-dojrzałości-usług-rest-rozszerzony--przykłady-poziomów--dlaczego-swamp)
11. [Problem wielokrotnego zatwierdzania formularza oraz sposób jego poprawnego rozwiązania](#pytanie-11-problem-wielokrotnego-zatwierdzania-formularza-oraz-sposób-jego-poprawnego-rozwiązania)
12. [Porównanie cykli życia i zastosowań sesyjnego beana EJB oraz sesyjnego singletonu EJB](#pytanie-12-porównanie-cykli-życia-i-zastosowań-sesyjnego-beana-ejb-oraz-sesyjnego-singletonu-ejb)
13. [Porównanie zasięgów w specyfikacji CDI oraz praktyczne przykłady ich zawartości](#pytanie-13-porównanie-zasięgów-w-specyfikacji-cdi-oraz-praktyczne-przykłady-ich-zawartości)
14. [Czy w CDI możliwe jest wstrzykiwanie beana o mniejszym zasięgu do beana o większym zasięgu? – Uzasadnienie](#pytanie-14-czy-w-cdi-możliwe-jest-wstrzykiwanie-beana-o-mniejszym-zasięgu-do-beana-o-większym-zasięgu--uzasadnienie)
15. [Opis poziomów modelu dojrzałości REST z przykładami](#pytanie-15-opis-poziomów-modelu-dojrzałości-rest-z-przykładami)
16. [Porównanie wykorzystania JPA w aplikacjach Jakarta EE (Java EE) oraz Java SE](#pytanie-16-porównanie-wykorzystania-jpa-w-aplikacjach-jakarta-ee-java-ee-oraz-java-se)
17. [Porównanie cyklu życia bezstanowego beana EJB i singletonu EJB](#pytanie-17-porównanie-cyklu-życia-bezstanowego-beana-ejb-i-singletonu-ejb)
18. [Problem wielokrotnego zatwierdzania formularza (Post-Redirect-Get)](#pytanie-18-problem-wielokrotnego-zatwierdzania-formularza-post-redirect-get)

---

### Pytanie 1: Po co dawać `provided` w `pom.xml`?

**Odpowiedź:**
Zasięg (scope) `provided` w pliku `pom.xml` narzędzia Maven informuje kompilator, że dana zależność (np. API platformy `jakarta.jakartaee-api`) jest niezbędna **wyłącznie na etapie kompilacji kodu**, natomiast **na etapie uruchomienia aplikacji (runtime) jej implementacja zostanie dostarczona przez środowisko wykonawcze (serwer aplikacji)**, takie jak GlassFish, Payara czy WildFly.

**Dlaczego to jest konieczne?**
1. **Unikanie konfliktów klas (Classloader Hell):** Gdyby biblioteki platformy zostały spakowane do archiwum `.war` (np. ze statusem domyślnym `compile`), w pamięci serwera znalazłyby się dwie różne kopie tych samych klas (jedna wbudowana w serwer, druga wewnątrz archiwum aplikacji). Prowadzi to do trudnych do zdiagnozowania błędów `ClassCastException`, `LinkageError` lub błędów inicjalizacji kontenera.
2. **Optymalizacja rozmiaru archiwum:** Paczka wdrożeniowa `.war` lub `.ear` nie zawiera setek megabajtów zbędnych bibliotek serwerowych, lecz tylko specyficzny kod aplikacji i biblioteki pomocnicze.
3. **Przenośność i separacja:** Aplikacja deklaruje jedynie kontrakt (interfejsy API), a o konkretną implementację (np. Hibernate vs EclipseLink, Weld vs OpenWebBeans) dba serwer, na którym aplikacja zostanie wdrożona.

---

### Pytanie 2: Jaki jest cykl życia servletu?

**Odpowiedź:**
Cykl życia servletu jest w całości kontrolowany przez kontener webowy (np. Tomcat, Jetty) i składa się z trzech kolejnych faz:

```
[ Żądanie klienta ]
        |
        v (jeśli brak instancji)
1. Załadowanie klasy, utworzenie instancji, wywołanie init()   [Jednorazowo]
        |
        +-----------------------+
        |                       |
        v                       |
2. Wywołanie service() <--------+  (wielokrotnie, współbieżnie dla każdego żądania)
   (delegacja do doGet/doPost)
        |
        v (zatrzymanie aplikacji / serwera)
3. Wywołanie destroy()                                         [Jednorazowo]
```

1. **Inicjalizacja (jednorazowa):**
   - Gdy wpłynie pierwsze żądanie do servletu (lub przy starcie serwera, jeśli skonfigurowano wcześniejsze ładowanie):
     - Kontener ładuje klasę servletu do pamięci.
     - Tworzy instancję servletu (konstruktor bezargumentowy).
     - Wywołuje metodę `init(ServletConfig)`. Służy ona do jednorazowego przygotowania zasobów (odczyt parametrów konfiguracyjnych, nawiązanie połączeń).
2. **Obsługa żądań (wielokrotna i współbieżna):**
   - Dla każdego napływającego żądania HTTP kontener przydziela odrębny wątek roboczy i wywołuje metodę `service(request, response)`.
   - Domyślna implementacja metody `service()` sprawdza typ czasownika HTTP i deleguje zadanie do odpowiedniej metody chronionej: `doGet()`, `doPost()`, `doPut()`, `doDelete()` itd.
3. **Zniszczenie (jednorazowe):**
   - Podczas wyłączania aplikacji lub przeładowania kontenera wywoływana jest metoda `destroy()`.
   - Pozwala to servletowi na zwolnienie zajmowanych zasobów systemowych, zamknięcie otwartych plików i zakończenie wątków.

**Kluczowa uwaga o wielowątkowości:** W kontenerze istnieje standardowo **tylko jedna instancja danego servletu**, z której korzysta **wiele wątków jednocześnie**. Z tego względu pola instancji servletu są współdzielone i nie wolno w nich przechowywać stanu specyficznego dla pojedynczego użytkownika (zagrożenie *race condition*).

---

### Pytanie 3: Czego lepiej używać – adnotacji czy konfiguracji w XMLu?

**Odpowiedź:**
Nie ma jednej uniwersalnej odpowiedzi – oba mechanizmy wzajemnie się uzupełniają, a wybór zależy od wymagań architektonicznych projektu:

| Cecha | Adnotacje w kodzie | Konfiguracja w XML (`web.xml`, `beans.xml`) |
| :--- | :--- | :--- |
| **Lokalizacja** | Bezpośrednio w kodzie źródłowym (*co-location*). | W odrębnych plikach konfiguracyjnych. |
| **Zalety** | - Zwięzłość, brak narzutu powtarzalnego kodu (boilerplate).<br>- Zasada *Convention over Configuration*.<br>- Kontrola typów na etapie kompilacji (trudniej o literówkę w nazwie klasy). | - **Możliwość zmiany bez rekompilacji kodu** (np. zmiana mapowania URL na produkcji).<br>- Możliwość **wymuszenia ścisłej kolejności wykonania** (np. łańcuch filtrów, listenery).<br>- Centralizacja konfiguracji. |
| **Wady** | - Brak możliwości zmiany bez rekompilacji i ponownego wdrożenia paczki.<br>- **Brak możliwości określenia kolejności komponentów** (np. filtrów).<br>- Może wydłużać start serwera (skanowanie klas). | - Rozwlekłość, podatność na błędy przy refaktoryzacji nazw klas w kodzie.<br>- Rozproszenie wiedzy poza kodem biznesowym. |
| **Pierwszeństwo** | Ustępują konfiguracji XML. | **Mają bezwzględne pierwszeństwo** – konfiguracja w XML zawsze nadpisuje adnotacje. |

**Podsumowanie inżynierskie:**
- **Adnotacje** są preferowane do codziennej pracy programistycznej i definiowania stałych reguł biznesowych (np. mapowanie encji JPA, wstrzykiwanie zależności CDI, deklaracja endpointów REST).
- **Deskryptory XML** stosuje się tam, gdzie adnotacje są niewystarczające: do ustalania kolejności filtrów/listenerów, do nadpisywania zachowania bibliotek zewnętrznych bez ingerencji w ich kod oraz przy konfiguracji zależnej od środowiska uruchomieniowego.

---

### Pytanie 4: Czym jest IoC (Inversion of Control), a czym jest Dependency Injection (DI)?

**Odpowiedź:**
Pojęcia te opisują powiązane, ale fundamentalnie różne poziomy abstrakcji:

```
+-----------------------------------------------------------+
| Inversion of Control (IoC) - Wzorzec Architektoniczny     |
| (Kierunek sterowania aplikacją: kontener zarządza kodem)  |
|                                                           |
|    +-------------------------------------------------+    |
|    | Dependency Injection (DI) - Wzorzec Projektowy  |    |
|    | (Sposób dostarczania zależności do obiektów)    |    |
|    +-------------------------------------------------+    |
+-----------------------------------------------------------+
```

1. **Inversion of Control (IoC – Odwrócenie Sterowania):**
   - Jest to **ogólny wzorzec architektoniczny** dotyczący **przepływu sterowania** w programie.
   - W modelu tradycyjnym to kod napisany przez programistę kontroluje wykonanie (tworzy pętle, decyduje kiedy wywołać funkcję z biblioteki).
   - W modelu IoC kontrolę przejmuje **zewnętrzne środowisko (kontener/framework)**. Kontener steruje cyklem życia, odbiera zdarzenia i w odpowiednim momencie sam wywołuje kod użytkownika. Zasada hollywoodzka: *"Nie dzwoń do nas, to my zadzwonimy do ciebie"*. Przykładem IoC jest cykl życia servletu kontrolowany przez serwer Tomcat.
2. **Dependency Injection (DI – Wstrzykiwanie Zależności):**
   - Jest to **konkretny wzorzec projektowy**, stanowiący szczególną formę realizacji zasady IoC, dotyczącą **zarządzania powiązaniami między obiektami**.
   - Obiekt nie odpowiada za tworzenie swoich zależności (brak `new`, brak fabryk) – zależności są do niego dostarczane z zewnątrz przez kontener.
3. **Relacja między nimi:**
   - IoC to szeroka reguła architektoniczna, a DI to specyficzny mechanizm.
   - Kontener IoC zarządza cyklem życia komponentów, a w procesie ich tworzenia **wykorzystuje wzorzec DI**, aby wstrzyknąć do nich wymagane referencje przed przekazaniem do nich sterowania.

---

### Pytanie 5: Do czego powinny służyć wszystkie rodzaje zapytań HTTP (w szczególności różnica między PUT a POST)?

**Odpowiedź:**

#### Zestawienie metod protokołu HTTP:
- **`GET`:** Pobranie reprezentacji istniejącego zasobu lub kolekcji zasobów. Jest metodą **bezpieczną** (nie modyfikuje stanu serwera) i **idempotentną**.
- **`DELETE`:** Usunięcie wskazanego zasobu o określonym URI. Metoda **idempotentna** (ponowienie usuwania nie zmienia faktu, że zasób nie istnieje).
- **`PATCH`:** Częściowa modyfikacja zasobu (aktualizacja tylko wskazanych pól).
- **`HEAD` / `OPTIONS`:** Pobranie samych nagłówków metadanych (HEAD) lub zapytanie o możliwości serwera i politykę CORS (OPTIONS).

#### Kluczowa różnica: `PUT` a `POST`

| Kryterium | `POST` | `PUT` |
| :--- | :--- | :--- |
| **Idempotentność** | **NIE jest idempotentny.** Każde kolejne wywołanie żądania tworzy kolejny nowy zasób lub powtarza operację biznesową. | **JEST idempotentny.** Wielokrotne wykonanie identycznego żądania z tym samym ciałem wywołuje dokładnie taki sam skutek na serwerze jak pojedyncze wywołanie. |
| **Adresowanie URI** | Żądanie kierowane jest na **adres kolekcji nadrzędnej** (np. `/api/books`). Klient nie zna jeszcze identyfikatora. | Żądanie kierowane jest na **dokładny adres konkretnego zasobu** (np. `/api/books/123`). |
| **Rola identyfikatora** | **Serwer decyduje** o wygenerowaniu identyfikatora i odsyła go w nagłówku `Location` (status `201 Created`). | **Klient wskazuje** dokładne docelowe miejsce zasobu. |
| **Semantyka działania** | *"Dodaj ten element do kolekcji i nadaj mu ID"*. | *"Pod tym adresem ma się znaleźć dokładnie taki zasób"* (zastąpienie istniejącego zasobu lub wstawienie nowego, jeśli nie istniał). |

---

### Pytanie 6: Dlaczego działa wstrzykiwanie mniejszego zasięgu do większego? W jaki sposób mniejszy zasięg „wchodzi” do większego i działa?

*(Tożsame z pytaniem 14)*

**Odpowiedź:**

#### 1. Na czym polega pozorny paradoks (dlaczego to wydaje się niemożliwe)?
Wyobraźmy sobie komponent o zasięgu aplikacyjnym (`@ApplicationScoped` – Singleton), np. `GeneratorRaportow`. Jest on tworzony **dokładnie raz** w chwili startu serwera (np. o 8:00 rano) i istnieje nieprzerwanie w pamięci przez wiele dni.

Do tego komponentu chcemy wstrzyknąć obiekt o węższym zasięgu (`@RequestScoped`), np. `KontekstUzytkownika` (przechowujący tożsamość i uprawnienia osoby wysyłającej pojedyncze żądanie HTTP).
- Pojedyncze żądanie HTTP trwa zaledwie **kilkanaście milisekund**, po czym jego obiekty są niszczone.
- W ciągu sekundy z aplikacji mogą korzystać setki różnych użytkowników (np. Jan, Anna, Piotr).

**Gdyby wstrzykiwanie działało w sposób tradycyjny (bezpośrednia referencja w pamięci):**
- Podczas pierwszego żądania (od Jana) do pola w `GeneratorRaportow` zostałby na sztywno przypisany obiekt Jana.
- Po zakończeniu żądania Jana obiekt ten uległby zniszczeniu (błędna referencja).
- Co gorsza, gdyby za chwilę z serwisu skorzystała Anna, serwis odczytałby dane Jana! Doszłoby do wycieku danych pomiędzy użytkownikami oraz błędów wielowątkowości.

#### 2. Rozwiązanie kontenera CDI: Mechanizm Client Proxy (Inteligentnego Pełnomocnika)
Kontener CDI rozwiązuje ten problem w genialny sposób: **nigdy nie wstrzykuje do pola rzeczywistego obiektu docelowego!**

Zamiast tego wstrzykiwany jest tzw. **Client Proxy (pełnomocnik klienta)**:
- Proxy to lekki, pusty obiekt wygenerowany dynamicznie przez kontener w kodzie bajtowym. Z zewnątrz wygląda identycznie jak wstrzykiwany bean (dziedziczy po nim i posiada te same metody), ale **sam w sobie nie posiada żadnego stanu biznesowego**.
- Ten jeden obiekt proxy zostaje na stałe osadzony w polu beana aplikacyjnego.

#### 3. Przebieg działania krok po kroku (Scenariusz z dwoma użytkownikami):
1. **Jan wysyła żądanie HTTP:** Serwer przydziela do jego obsługi **Wątek #1**. Kontener CDI tworzy w pamięci świeży obiekt `KontekstUzytkownika` Jana i rejestruje go w zmiennej lokalnej tego wątku (`ThreadLocal`).
2. Kod w Wątku #1 wywołuje metodę w singletonie aplikacyjnym: `GeneratorRaportow.generuj()`.
3. Singleton wywołuje metodę na wstrzykniętej zależności: `proxy.pobierzDaneUzytkownika()`.
4. **Proxy przechwytuje to wywołanie:**
   - Proxy pyta wewnętrzny Menedżer Kontekstów CDI: *"W jakim wątku procesora się znajduję?"* $\rightarrow$ *"W Wątku #1"*.
   - Menedżer sprawdza: *"Jaka instancja `@RequestScoped` jest aktualnie aktywna w Wątku #1?"* $\rightarrow$ *"Instancja Jana"*.
   - Proxy dynamicznie pobiera prawdziwy obiekt Jana i deleguje do niego wywołanie metody.
5. **Równolegle Anna wysyła żądanie HTTP:** Serwer obsługuje ją w **Wątku #2**. Anna wywołuje ten sam singleton i dokładnie to samo pole proxy. Proxy bada Wątek #2, odnajduje obiekt Anny i deleguje wywołanie do Anny!
6. Po zakończeniu obsługi żądania Jana, jego instancja `@RequestScoped` jest niszczona. Obiekt **Proxy w singletonie żyje nadal** i czeka na kolejne żądania innych klientów.

#### 4. Praktyczna analogia z życia:
Singleton aplikacyjny to **recepcja w biurowcu**, a bean `@RequestScoped` to **kurier**, który przychodzi na 3 minuty i odchodzi.
Zamiast "przywiązywać" jednego kuriera Jana do biurka w recepcji na stałe (co uniemożliwiłoby obsługę przez innych kurierów), w recepcji instaluje się **telefon wewnętrzny (Proxy)**. Gdy recepcjonista podnosi słuchawkę i prosi o paczkę, centrala telefoniczna (Menedżer Kontekstów) automatycznie łączy go z kurierem, który w danej minucie dyżuruje przy wejściu.

---

### Pytanie 7: Czemu wstrzykiwanie przez interfejs jest lepsze?

**Odpowiedź:**

#### 1. Zasada Architektoniczna: Dependency Inversion Principle (DIP)
Wstrzykiwanie zależności zadeklarowanych jako **interfejs** (np. `Powiadamiacz`), a nie konkretna klasa implementacyjna (np. `EmailPowiadamiacz`), realizuje zasadę odwrócenia zależności z kanonu SOLID:
> *Moduły wysokiego poziomu nie powinny zależeć od modułów niskiego poziomu. Obie warstwy powinny zależeć od abstrakcji (interfejsów).*

#### 2. Szczegółowe uzasadnienie zalet:
1. **Luźne sprzężenie (*Loose Coupling*):**
   - Klasa wołająca zna wyłącznie publiczny kontrakt (nazwy metod, parametry, zwracane typy).
   - Nie wie i nie musi wiedzieć, jak usługa jest realizowana pod maską, jakich bibliotek używa ani czy łączy się z chmurą, bazą danych czy dyskiem.
2. **Polimorfizm i bezinwazyjna wymiana implementacji:**
   - **Przykład scenariusza:** Mamy serwis zamówień `SklepService`, który wstrzykuje interfejs `KalkulatorPodatkowy`.
   - Na rynku polskim wstrzykujemy implementację `PolskiKalkulatorPodatkowy`.
   - Gdy wdrażamy aplikację w Niemczech, tworzymy klasę `NiemieckiKalkulatorPodatkowy`.
   - Klasa `SklepService` **nie zmienia się ani o jedną linijkę kodu!** Podmiana implementacji następuje wyłącznie na poziomie konfiguracji kontenera (profil wdrożeniowy lub kwalifikator CDI), bez konieczności modyfikowania, rekompilowania czy ponownego testowania serwisu zamówień.
3. **Łatwość testowania jednostkowego (*Mocking*):**
   - W testach jednostkowych chcemy sprawdzić wyłącznie logikę biznesową danej klasy, bez uruchamiania prawdziwych zewnętrznych baz danych czy wysyłania prawdziwych wiadomości SMS do klientów.
   - Jeśli klasa zależy od interfejsu, w teście możemy w ułamku sekundy podstawić atrapę (obiekt typu *mock/stub*), która symuluje działanie interfejsu w pamięci RAM. Gdyby klasa zależała na sztywno od konkretnej klasy implementacyjnej, podmiana taka byłaby niemożliwa lub wymagała skomplikowanej manipulacji kodem bajtowym.
4. **Zdalne wywołania w systemach rozproszonych (EJB Remoting):**
   - W architekturze zintegrowanej ze zdalnymi komponentami EJB (`@Remote`), interfejs jest jedynym elementem, który kompilujemy i udostępniamy klientowi zewnętrznemu (np. aplikacji okienkowej).
   - Klient posiada tylko plik JAR z czystym interfejsem, a cały ciężki kod implementacji wykonuje się na odległym serwerze korporacyjnym.

---

### Pytanie 8: Działanie mechanizmu PROXY

**Odpowiedź:**

#### 1. Czym jest Proxy (Pełnomocnik)?
W architekturze kontenerów enterprise (Jakarta CDI, EJB, Spring) **Proxy to obiekt-pośrednik**, wygenerowany automatycznie w locie w pamięci maszyny wirtualnej, który "udaje" prawdziwy komponent biznesowy i przechwytuje każde wywołanie dowolnej metody skierowane do tego komponentu.

```
+----------------+          +------------------------------------------------------+
| Klient         |          | Obiekt PROXY (Dynamicznie generowany)                |
| (kod wołający) | -------> | 1. Przechwycenie wywołania metody (Interception)     |
+----------------+          | 2. Wykonanie logiki PRZED (transakcja, security, itp)|
                            | 3. Delegacja wywołania --------------------------+   |
                            | 4. Wykonanie logiki PO (commit transakcji, audyt)|   |
                            +--------------------------------------------------|---+
                                                                               |
                                                                               v
                                                            +-------------------------------+
                                                            | Prawdziwa Instancja Docelowa  |
                                                            | (wykonanie czystego kodu Java)|
                                                            +-------------------------------+
```

#### 2. Jakie zadania realizuje Proxy pod maską?
Dzięki umieszczeniu Proxy pomiędzy klientem a obiektem docelowym, kontener może wstrzyknąć do aplikacji zaawansowane mechanizmy enterprise w sposób całkowicie przezroczysty dla programisty:

1. **Zarządzanie Zasięgami i Kontekstami (Kluczowe powiązanie z Pytaniem 6):**
   - Gdy klient wywołuje metodę, proxy sprawdza w menedżerze kontekstów identyfikator bieżącego wątku i dynamicznie przekazuje wywołanie do właściwej instancji (np. instancji przypisanej do bieżącego żądania HTTP lub bieżącej sesji).
2. **Automatyczne Zarządzanie Transakcjami (CMT – Container-Managed Transactions):**
   - Zanim metoda biznesowa się rozpocznie, proxy automatycznie kontaktuje się z menedżerem transakcji serwera i otwiera nową transakcję bazodanową (`BEGIN TRANSACTION`).
   - Następnie wywoływana jest właściwa metoda beana.
   - Jeśli metoda zakończy się sukcesem, proxy zatwierdza transakcję (`COMMIT`). Jeśli rzuci wyjątek systemowy, proxy wycofuje zmiany (`ROLLBACK`). Programista nie musi pisać ani jednej linii kodu transakcyjnego!
3. **Deklaratywne Bezpieczeństwo i Kontrola Uprawnień (`@RolesAllowed`):**
   - Proxy sprawdza tożsamość zalogowanego użytkownika i jego role. Jeśli użytkownik nie ma wymaganej roli (np. `ADMIN`), proxy natychmiast rzuca wyjątek `SecurityException`, a kod wewnątrz właściwego komponentu w ogóle nie zostaje dotknięty.
4. **Leniwe Ładowanie (Lazy Loading):**
   - Prawdziwy, "ciężki" obiekt docelowy wcale nie musi istnieć w chwili wstrzyknięcia. Proxy może opóźnić jego utworzenie aż do momentu, gdy klient po raz pierwszy faktycznie wywoła na nim jakąś metodę.
5. **Asynchroniczność (`@Asynchronous` w EJB):**
   - Proxy przyjmuje wywołanie metody, natychmiast zwraca sterowanie do klienta, a samo wywołanie właściwego obiektu deleguje do osobnego wątku roboczego z puli serwera.

---

### Pytanie 9: Po co jest „lazy getter”?

**Odpowiedź:**
Pojęcie "lazy getter" (leniwy getter / leniwa inicjalizacja w metodzie dostępowej) odnosi się do techniki, w której kosztowne pozyskanie danych następuje dopiero w momencie pierwszego wywołania metody odczytującej (`getter`), a uzyskany wynik jest zapamiętywany w polu instancji na potrzeby kolejnych odczytów.

**Dlaczego jest potrzebny w architekturze Java EE / Jakarta Faces (JSF)?**
1. **Ochrona przed wielokrotnym odpytywaniem bazy danych w JSF:**
   - W cyklu życia żądania JSF silnik renderujący widok oraz ewaluator Expression Language (`#{...}`) potrafią wywołać ten sam getter **wielokrotnie w ramach obsługi pojedynczego żądania** (np. raz przy sprawdzaniu renderowania komponentu, drugi raz przy iteracji pętli tabeli, trzeci raz przy liczeniu rozmiaru).
   - Gdyby getter za każdym razem odpytywał bazę danych przez serwis, wygenerowałoby to lawinę redundantnych zapytań SQL i dramatyczny spadek wydajności. Leniwy getter przy pierwszym wywołaniu sprawdza `if (dane == null) { pobierzZSerwisu(); } return dane;`.
2. **Optymalizacja zużycia zasobów (Leniwe ładowanie):**
   - Jeśli strona posiada sekcje warunkowe (np. widoczne tylko po kliknięciu zakładki), dane nie są pobierane z bazy podczas konstruowania beana, lecz dopiero wtedy, gdy dana sekcja rzeczywiście zażąda ich wyświetlenia.

---

### Pytanie 10: Model dojrzałości usług REST (rozszerzony) + przykłady poziomów + dlaczego SWAMP?

*(Tożsame z pytaniem 15)*

**Odpowiedź:**
Model dojrzałości opracowany przez Leonarda Richardsona (*Richardson Maturity Model*) definiuje 4 etapy dochodzenia od tradycyjnych usług zdalnych (RPC) do pełnego, kanonicznego stylu RESTful API:

#### Dlaczego poziom zerowy nazwano "SWAMP" (Bagno)?
Określenie **The Swamp of POX** (*Bagno Zwykłego Starego XML-a / POX - Plain Old XML*) odnosi się do sytuacji, w której architektura ignoruje wszystkie unikalne mechanizmy sieci Web i protokołu HTTP. Cała komunikacja zostaje "wrzucona do jednego bezkształtnego dołu / bagna":
- Istnieje tylko **jeden wspólny adres URL** dla wszystkich operacji w systemie.
- Wykorzystywana jest **wyłącznie jedna metoda HTTP (zawsze POST)**.
- Protokół HTTP jest zdegradowany do roli zwykłego kabla transmisyjnego (tunelu), a kody odpowiedzi to niemal zawsze `200 OK`, nawet jeśli wewnątrz ciała XML znajduje się opis błędu biznesowego (np. SOAP).

#### Cztery poziomy dojrzałości:
1. **Poziom 0: The Swamp of POX (HTTP jako tunel RPC):**
   - Jeden adres URI, jedna metoda (POST). Intencja operacji zaszyta w ciele żądania.
   - *Przykład:* `POST /services/api` z ciałem `<command>getStudentDetails</command><id>123</id>`.
2. **Poziom 1: Zasoby (Resources):**
   - Przejście od pojedynczego punktu końcowego do **indywidualnych adresów URI** reprezentujących zasoby biznesowe (rzeczowniki w hierarchii).
   - *Przykład:* `/api/students`, `/api/students/123`, `/api/students/123/courses`.
   - *Wada tego poziomu:* Mimo wydzielenia zasobów, operacje nadal są wykonywane niewłaściwymi metodami (np. wywołania usuwania przez GET: `GET /api/students/123/delete`).
3. **Poziom 2: Czasowniki HTTP i Kody Statusu (HTTP Verbs & Status Codes):**
   - Świadome i rygorystyczne użycie standardowych czasowników HTTP zgodnie z ich semantyką: `GET` (odczyt), `POST` (tworzenie), `PUT` (zastąpienie/aktualizacja całości), `DELETE` (usunięcie), `PATCH` (edycja częściowa).
   - Stosowanie standardowych kodów odpowiedzi: `201 Created` z nagłówkiem `Location`, `204 No Content`, `400 Bad Request`, `404 Not Found`.
   - *Przykład:* `DELETE /api/students/123` $\rightarrow$ Odpowiedź: `204 No Content`.
4. **Poziom 3: Sterowanie Hipermediami – HATEOAS (Hypermedia As The Engine Of Application State):**
   - Najwyższy poziom dojrzałości REST.
   - Serwer zwraca w odpowiedzi dane zasobu wraz z **aktywnymi hiperłączami (linkami nawigacyjnymi)**, które informują klienta, jakie kolejne kroki biznesowe może w tym momencie wykonać.
   - *Przykład:* Pobranie stanu konta bankowego zwraca kwotę oraz linki relacyjne: link do wykonania przelewu (`rel="transfer"`), link do historii operacji (`rel="history"`), ale jeśli konto jest zablokowane, serwer dynamicznie usuwa link przelewu i odsyła jedynie link do procedury odblokowania konta (`rel="unlock"`).

---

### Pytanie 11: Problem wielokrotnego zatwierdzania formularza oraz sposób jego poprawnego rozwiązania

*(Tożsame z pytaniem 18)*

**Odpowiedź:**

#### 1. Na czym polega problem (z przykładem z życia)?
Wyobraźmy sobie formularz płatności w sklepie internetowym:
1. Użytkownik klika przycisk *"Kupuję i płacę 250 zł"*.
2. Przeglądarka wysyła żądanie **HTTP POST** z danymi zamówienia.
3. Serwer obciąża kartę płatniczą kwotą 250 zł, zapisuje zamówienie w bazie i – w naiwnym podejściu – od razu zwraca kod **`200 OK`** wraz z kodem HTML strony: *"Dziękujemy za zakup"*.
4. Użytkownik widzi podsumowanie na ekranie. Jednak z powodu chwilowego zawieszenia łącza lub odruchu klika klawisz **F5 (Odśwież stronę)** na klawiaturze.
5. **Konsekwencja:** Przeglądarka pamięta, że aktualny widok był wynikiem żądania POST. Pyta użytkownika: *"Czy chcesz ponownie przesłać formularz?"*. Jeśli użytkownik kliknie *"Tak"*, przeglądarka **ponownie wysyła to samo żądanie POST**. Serwer otrzymuje drugie żądanie i **po raz drugi pobiera 250 zł z karty oraz tworzy drugie, zdublowane zamówienie w bazie!**

#### 2. Poprawne rozwiązanie: Idiom POST-REDIRECT-GET (PRG)
Wzorzec architektoniczny PRG rozwiązuje ten problem poprzez rozbicie operacji na trzy ściśle rozdzielone kroki:

```
Klient (Przeglądarka)                                Serwer Aplikacji
         |                                                   |
         | -------- 1. POST /oplac (kwota 250 zł) ---------> | Pobranie 250 zł,
         |                                                   | utworzenie zamówienia w bazie
         | <------- 2. HTTP 302/303 Redirect --------------- | 
         |             Nagłówek Location: /sukces?id=99      | (Brak ciała HTML!)
         |                                                   |
         | -------- 3. GET /sukces?id=99 ------------------> | Bezpieczne odczytanie
         |                                                   | danych z bazy
         | <------- 4. HTTP 200 OK (widok podsumowania) ---- |
```

1. **Krok 1 (POST):** Klient wysyła dane formularza żądaniem POST, które modyfikuje stan po stronie serwera (zapis do bazy, obciążenie konta).
2. **Krok 2 (REDIRECT):** Serwer po pomyślnym wykonaniu logiki **pod żadnym pozorem nie zwraca kodu 200 z treścią HTML**. Zamiast tego odsyła odpowiedź przekierowującą (nagłówek statusu **HTTP 302 Found** lub **HTTP 303 See Other**) z nagłówkiem `Location: /sukces?id=99`.
3. **Krok 3 (GET):** Przeglądarka po odebraniu kodu przekierowania 302/303 natychmiast i automatycznie wykonuje żądanie **HTTP GET** pod wskazany adres `/sukces?id=99`.
4. **Krok 4 (Prezentacja):** Serwer w odpowiedzi na GET zwraca stronę podsumowania z kodem **`200 OK`**.

#### 3. Dlaczego to całkowicie rozwiązuje problem?
- W historii i buforze przeglądarki internetowej **ostatnią wykonaną akcją jest żądanie GET**, a nie POST.
- Gdy użytkownik naciśnie teraz klawisz **F5 (Odśwież)**, przeglądarka powtórzy wyłącznie ostatnie żądanie – czyli **bezpieczne, idempotentne żądanie `GET /sukces?id=99`**.
- Użytkownik po prostu ponownie odczyta stronę z podziękowaniem, a serwer nie wykona żadnej operacji finansowej ani bazodanowej.
- **W Jakarta Faces (JSF):** Domyślnie JSF wykonuje postbacki na ten sam adres URL. Aby aktywować idiom PRG, do zwracanego łańcucha nawigacji dodaje się parametr `?faces-redirect=true`.

---

### Pytanie 12: Porównanie cykli życia i zastosowań sesyjnego beana EJB oraz sesyjnego singletonu EJB

**Odpowiedź:**

#### Porównanie w tabeli:

| Aspekt | Bezstanowy Bean EJB (`@Stateless`) | Sesyjny Singleton EJB (`@Singleton`) |
| :--- | :--- | :--- |
| **Liczba instancji** | **Wiele instancji zgrupowanych w puli (*pool*)**. Liczba rośnie i maleje pod kontrolą serwera w zależności od obciążenia. | **Dokładnie jedna instancja** w skali całej aplikacji dla wszystkich klientów i wątków. |
| **Cykl życia** | Utworzenie instancji $\rightarrow$ wstrzyknięcie DI $\rightarrow$ `@PostConstruct` $\rightarrow$ stan gotowości (*Ready*) w puli $\rightarrow$ wycofanie z puli $\rightarrow$ `@PreDestroy` $\rightarrow$ usunięcie. | Utworzenie $\rightarrow$ wstrzyknięcie DI $\rightarrow$ `@PostConstruct` $\rightarrow$ obsługa wszystkich żądań $\rightarrow$ `@PreDestroy` przy zamykaniu aplikacji. Może startować przy starcie serwera (`@Startup`). |
| **Współbieżność** | **Brak problemów współbieżności.** W danej chwili dana instancja z puli obsługuje dokładnie jednego klienta. Nie wymaga blokad ani synchronizacji. | **Wielowątkowy dostęp do tej samej instancji.** Wymaga zarządzania współbieżnością: domyślnie przez kontener (`@ConcurrencyManagement(CONTAINER)`) z blokadami `@Lock(READ/WRITE)`. |
| **Stan konwersacyjny** | Nie przechowuje stanu klienta pomiędzy wywołaniami metod. | Przechowuje stan globalny, współdzielony pomiędzy wszystkimi użytkownikami. |
| **Praktyczne zastosowanie** | - Standardowa logika biznesowa aplikacji.<br>- Operacje bazodanowe CRUD, serwisy transakcyjne.<br>- Przetwarzanie płatności, kalkulacje, generowanie faktur. | - Współdzielony bufor pamięci podręcznej (Cache aplikacji).<br>- Zarządca konfiguracji systemu wczytywanej z dysku/bazy.<br>- Liczniki globalne, harmonogramy zadań.<br>- Klasy inicjalizujące dane startowe bazy (`@Startup`). |

---

### Pytanie 13: Porównanie zasięgów w specyfikacji CDI oraz praktyczne przykłady ich zawartości

**Odpowiedź:**
Zasięg w specyfikacji Contexts and Dependency Injection (CDI) określa cykl życia instancji oraz granice kontekstu, w którym instancja jest współdzielona:

1. **`@Dependent` (Zasięg zależny – pseudozasięg, domyślny w CDI):**
   - Instancja powstaje na nowo przy każdym wstrzyknięciu i nie jest współdzielona. Jej czas życia kończy się wraz ze zniszczeniem obiektu, do którego została wstrzyknięta.
   - *Co powinno się w nim znaleźć:* Małe klasy pomocnicze, utility classes, konwertery, obiekty walidujące, kalkulatory matematyczne.
2. **`@RequestScoped` (Zasięg żądania HTTP):**
   - Instancja powstaje przy nadejściu żądania HTTP i jest niszczona w chwili odesłania odpowiedzi do klienta.
   - *Co powinno się w nim znaleźć:* Dane przesłane w formularzu bieżącego zapytania, kontekst bieżącego żądania sieciowego, token uwierzytelniający żądania, dane filtrów wyszukiwania.
3. **`@ConversationScoped` (Zasięg konwersacji):**
   - Może rozciągać się na wiele kolejnych żądań HTTP. Sterowany programistycznie za pomocą metod `conversation.begin()` oraz `conversation.end()`.
   - *Co powinno się w nim znaleźć:* Wieloetapowe procesy biznesowe (kreatory typu *wizard*), np. 4-krokowy proces rezerwacji biletu lotniczego, formularz wniosku kredytowego.
4. **`@SessionScoped` (Zasięg sesji HTTP):**
   - Instancja żyje tak długo, jak aktywna jest sesja danego użytkownika w przeglądarce (niszczona po wylogowaniu lub po upływie czasu bezczynności).
   - *Co powinno się w nim znaleźć:* Koszyk zakupowy w sklepie internetowym, preferencje językowe i motyw graficzny użytkownika, dane tożsamości zalogowanego klienta.
5. **`@ApplicationScoped` (Zasięg aplikacji):**
   - Dokładnie jedna instancja tworzona na cały czas działania aplikacji webowej (od wdrożenia do wyłączenia).
   - *Co powinno się w nim znaleźć:* Słowniki referencyjne (np. kody pocztowe, kraje, stawki podatkowe), globalna konfiguracja systemu, klienci zewnętrznych usług sieciowych.
6. **`@ViewScoped` (Zasięg widoku – ze specyfikacji Faces):**
   - Instancja żyje tak długo, jak użytkownik pozostaje na danej stronie widoku JSF i wykonuje akcje postback/AJAX.
   - *Co powinno się w nim znaleźć:* Stan tabeli danych na widoku (stronicowanie, sortowanie w locie), edytowany w danym oknie obiekt encji.

---

### Pytanie 14: Czy w CDI możliwe jest wstrzykiwanie beana o mniejszym zasięgu do beana o większym zasięgu? – Uzasadnienie

*(Odpowiedź tożsama merytorycznie z pytaniem 6)*

**Odpowiedź:**
**TAK, w specyfikacji CDI jest to w pełni możliwe i stanowi jedną z jej kluczowych cech architektonicznych.**

**Uzasadnienie:**
- Wstrzyknięcie beana o węższym cyklu życia (np. `@RequestScoped`) do beana o szerszym cyklu życia (np. `@ApplicationScoped`) nie powoduje błędu ani wycieku stanu dzięki mechanizmowi **Client Proxy (pełnomocnika klienta)**.
- Kontener CDI nie przekazuje bezpośredniego wskaźnika do obiektu w pamięci, lecz wstrzykuje dynamicznie wygenerowane proxy.
- Proxy przy każdym wywołaniu metody dynamicznie odnajduje instancję właściwą dla bieżącego wątku i aktywnego kontekstu żądania.
- *Uwaga:* W przypadku wstrzyknięcia beana o pseudozasięgu `@Dependent` (który nie używa proxy) do beana `@ApplicationScoped`, instancja dependent zostanie utworzona raz i będzie żyć tak długo, jak bean aplikacyjny.

---

### Pytanie 15: Opis poziomów modelu dojrzałości REST z przykładami

*(Szczegółowy opis zawarty w Pytaniu 10 – zwięzłe podsumowanie zestawcze)*

**Odpowiedź:**
1. **Level 0 (The Swamp of POX):** Pojedynczy punkt końcowy, HTTP używany jako tunel RPC, wyłącznie metoda POST.
   - *Przykład:* `POST /api` (w ciele XML komenda wywołania procedury).
2. **Level 1 (Resources):** Wprowadzenie wielu adresów URI odpowiadających różnym zasobom domenowym.
   - *Przykład:* `/api/books`, `/api/books/42`.
3. **Level 2 (HTTP Verbs & Status Codes):** Wykorzystanie czasowników HTTP zgodnie z ich semantyką (GET, POST, PUT, DELETE) oraz standardowych kodów odpowiedzi (200, 201 + Location, 204, 400, 404).
   - *Przykład:* `GET /api/books/42` $\rightarrow$ `200 OK`; `DELETE /api/books/42` $\rightarrow$ `204 No Content`.
4. **Level 3 (HATEOAS):** Sterowanie stanem aplikacji przez hipermedia. Odpowiedzi zawierają dynamiczne linki do kolejnych dozwolonych operacji.
   - *Przykład:* Obiekt zamówienia w formacie JSON zawiera linki `_links: { "pay": { "href": "/orders/1/payment" } }`.

---

### Pytanie 16: Porównanie wykorzystania JPA w aplikacjach Jakarta EE (Java EE) oraz Java SE

**Odpowiedź:**

| Obszar porównania | Aplikacje Java SE (Standalone) | Aplikacje Jakarta EE (Serwer Aplikacji) |
| :--- | :--- | :--- |
| **Pozyskanie i zarządzanie `EntityManager`** | **Ręczne zarządzanie przez programistę:**<br>- Tworzenie fabryki: `Persistence.createEntityManagerFactory(...)`.<br>- Manualne otwieranie i zamykanie: `em = emf.createEntityManager();` oraz `em.close()`.<br>- Obiekt `EntityManager` **nie jest bezpieczny wątkowo** – aplikacja musi pilnować izolacji wątków. | **Automatyczne zarządzanie przez kontener:**<br>- Deklaratywne wstrzykiwanie za pomocą adnotacji `@PersistenceContext`.<br>- Serwer wstrzykuje bezpieczne wątkowo proxy powiązane z kontekstem transakcyjnym bieżącego wątku.<br>- Kontener sam dba o zamykanie sesji. |
| **Typ zarządzania transakcjami (`transaction-type`)** | **`RESOURCE_LOCAL`:**<br>- Transakcjami steruje programista w kodzie: `em.getTransaction().begin()`, `commit()`, `rollback()`. | **`JTA` (Java Transaction API):**<br>- Domyślnie transakcje kontrolowane przez kontener (CMT) na granicach metod EJB lub beanów CDI z `@Transactional`.<br>- Obsługa transakcji rozproszonych (wielobazowych, 2PC). |
| **Konfiguracja bazy danych i źródeł danych** | Dane dostępowe (sterownik JDBC, adres URL bazy, login, hasło) wpisane są **bezpośrednio w pliku `persistence.xml`**. | Konfiguracja połączenia i pula połączeń definiowana jest w serwerze jako zasób **`DataSource`**. W aplikacji w `persistence.xml` wskazuje się jedynie logiczną nazwę JNDI (`<jta-data-source>`). |
| **Przenośność wdrożeniowa** | Niska – zmiana bazy danych wymaga ingerencji w pliki konfiguracyjne spakowane w archiwum. | Bardzo wysoka – ta sama paczka `.war` działa w środowisku dev i prod, bo łączy się z zasobem JNDI serwera. |

---

### Pytanie 17: Porównanie cyklu życia bezstanowego beana EJB i singletonu EJB

**Odpowiedź:**

#### 1. Faza powstawania i inicjalizacji:
- **`@Stateless`:**
  - Kontener tworzy **pulę wielu instancji**.
  - Nowa instancja jest tworzona w miarę napływu żądań klientów (lub alokowana z góry).
  - Dla każdej nowo tworzonej instancji kontener wstrzykuje zależności (DI) i wywołuje metodę **`@PostConstruct`**. Metoda ta wykona się w trakcie działania aplikacji **wielokrotnie** (tyle razy, ile instancji beana powstanie w puli).
- **`@Singleton`:**
  - Kontener tworzy **dokładnie jedną instancję** dla całej aplikacji.
  - Może powstać leniwie (przy pierwszym odwołaniu) lub eager w trakcie startu aplikacji (adnotacja `@Startup`).
  - Metoda **`@PostConstruct`** wykonuje się **dokładnie jeden raz** w całym cyklu życia aplikacji.

#### 2. Faza obsługi żądań i współbieżność:
- **`@Stateless`:**
  - Instancje spoczywają w puli. Gdy przychodzi żądanie, kontener pobiera jedną wolną instancję i przydziela ją do obsługi danego wywołania.
  - Instancja w danej chwili wykonuje kod tylko dla jednego klienta – **brak współdzielenia instancji między wątkami**, brak potrzeby blokad.
- **`@Singleton`:**
  - Ta sama, jedyna instancja obsługuje jednocześnie wszystkie wątki i wszystkich klientów.
  - Wymaga mechanizmu blokad współbieżności kontenera (`@ConcurrencyManagement` z blokadami `@Lock(READ)` lub wyłącznymi `@Lock(WRITE)`).

#### 3. Faza niszczenia:
- **`@Stateless`:** Metoda **`@PreDestroy`** wywoływana jest wielokrotnie – za każdym razem, gdy kontener decyduje o zmniejszeniu rozmiaru puli instancji.
- **`@Singleton`:** Metoda **`@PreDestroy`** wywoływana jest **dokładnie jeden raz**, w momencie zatrzymywania lub usuwania całej aplikacji z serwera.

---

### Pytanie 18: Problem wielokrotnego zatwierdzania formularza (Post-Redirect-Get)

*(Szczegółowy opis zawarty w Pytaniu 11 – zwięzłe podsumowanie)*

**Odpowiedź:**
- **Istota problemu:** Wysłanie formularza metodą HTTP POST zmieniającego stan serwera, po którym serwer zwraca kod `200 OK`, powoduje, że odświeżenie strony w przeglądarce (F5) skutkuje ponownym wysłaniem żądania POST i powtórzeniem operacji biznesowej (np. ponowna płatność).
- **Rozwiązanie: Idiom POST-REDIRECT-GET (PRG):**
  1. Klient wysyła formularz żądaniem **POST**.
  2. Serwer wykonuje logikę i zamiast treści HTML zwraca nagłówek przekierowania **HTTP 302/303 Redirect** z adresem docelowym (`Location`).
  3. Przeglądarka natychmiast wykonuje żądanie **GET** pod wskazany adres i pobiera stronę wynikową.
  4. Odświeżenie strony (F5) powtarza jedynie bezpieczne żądanie GET, eliminując ryzyko zdublowania danych.
  5. W Jakarta Faces (JSF) realizowane parametrem `faces-redirect=true`.
