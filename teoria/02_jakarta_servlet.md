# 02. Jakarta Servlet – Teoria na Kolokwium

## 1. Czym jest Servlet?
- **Definicja:** Podstawowy komponent warstwy webowej w środowisku Jakarta EE. Jest to klasa języka Java zarządzana przez kontener webowy, która rozszerza możliwości serwera o dynamiczną obsługę żądań sieciowych (typowo żądań protokołu HTTP).
- **Klasa bazowa:** Najczęściej dziedziczy po klasie `HttpServlet`, która dostarcza szablonową implementację metod obsługi protokołu HTTP.
- **Rola:** Odbiera żądania HTTP, przetwarza parametry i nagłówki wejściowe, komunikuje się z logiką biznesową i generuje odpowiedź dowolnego typu MIME:
  - Tekstową (HTML, JSON, XML, zwykły tekst).
  - Binarną (pliki do pobrania, generowane dokumenty PDF, dynamiczne obrazy).
- **Deklaracja i konfiguracja:** Może być konfigurowany deklaratywnie za pomocą adnotacji `@WebServlet` lub w deskryptorze wdrożenia `web.xml`.
  - **Zasada pierwszeństwa:** Jeśli konfiguracja występuje równolegle w adnotacji oraz w pliku `web.xml`, **konfiguracja w deskryptorze `web.xml` zawsze ma bezwzględne pierwszeństwo** i nadpisuje ustawienia adnotacji.

---

## 2. Cykl życia Servletu (Bardzo częste pytanie egzaminacyjne!)
Cykl życia servletu jest w pełni kontrolowany przez **kontener webowy** i składa się z trzech głównych etapów:

```
[ Brak instancji w pamięci ]
             |
             | 1. Załadowanie klasy servletu
             | 2. Utworzenie instancji (konstruktor)
             | 3. Wywołanie metody init()  (inicjalizacja jednorazowa)
             v
      [ Instancja Gotowa ] <-------+
             |                     |  Obsługa wielu żądań (współbieżnie)
             | Wywołanie service() |  (delegacja do doGet, doPost, etc.)
             +---------------------+
             |
             | Zatrzymanie aplikacji / usunięcie servletu
             v
    [ Wywołanie destroy() ] (zwolnienie zasobów)
             |
             v
      [ Usunięcie obiektu ]
```

### Szczegółowy przebieg etapów:
1. **Inicjalizacja (etap jednorazowy):**
   - Gdy do serwera przychodzi pierwsze żądanie skierowane do danego servletu (lub podczas startu serwera, jeśli skonfigurowano opcję wcześniejszego ładowania *load-on-startup*):
     a) Kontener ładuje klasę servletu do pamięci za pomocą dedykowanego classloadera.
     b) Kontener tworzy pojedynczą instancję klasy servletu (wywołując konstruktor bezargumentowy).
     c) Kontener przekazuje konfigurację i wywołuje metodę `init(ServletConfig config)`. Jest to faza, w której servlet może otworzyć połączenia, odczytać parametry inicjalizacyjne i przygotować struktury danych.
2. **Obsługa żądań (wielokrotna, współbieżna):**
   - Dla każdego napływającego żądania HTTP kontener przydziela odrębny wątek roboczy z puli wątków i wywołuje metodę `service(HttpServletRequest, HttpServletResponse)`.
   - Domyślna implementacja metody `service()` bada czasownik metody HTTP (np. GET, POST, PUT, DELETE) i deleguje wykonanie do odpowiedniej metody chronionej: `doGet()`, `doPost()`, `doPut()`, `doDelete()` itd.
3. **Zniszczenie (etap jednorazowy):**
   - W momencie zatrzymywania aplikacji, przeładowywania kontekstu (*hot-deploy*) lub zamykania serwera kontener wycofuje servlet ze służby.
   - Kontener wywołuje metodę `destroy()`. Pozwala to na bezpieczne zamknięcie otwartych plików, połączeń sieciowych, wątków i zwolnienie zasobów systemowych.

### Kluczowa uwaga dotycząca wielowątkowości (Thread Safety):
- W standardowych warunkach kontener tworzy **dokładnie jedną instancję danego servletu** dla całej aplikacji (wzorzec Singleton).
- Wszystkie przychodzące żądania są obsługiwane przez **wiele wątków jednocześnie operujących na tej samej instancji**.
- **Konsekwencja:** Pola klasy servletu są współdzielone pomiędzy wszystkimi użytkownikami i wątkami. Przechowywanie stanu użytkownika lub danych transakcyjnych w polach klasy servletu prowadzi do wyścigów (*race conditions*) i błędów bezpieczeństwa. Pola w servlecie powinny być niemutowalne (*read-only*) lub wstrzykiwane jako serwisy bezstanowe.

---

## 3. Parametry inicjalizacyjne
- Servlety mogą posiadać parametry początkowe zdefiniowane w adnotacji (`@WebInitParam`) lub deskryptorze `web.xml` (`<init-param>`).
- Są one przekazywane w obiekcie `ServletConfig` do metody `init()` i pozwalają na parametryzację działania komponentu bez konieczności retranslacji kodu.

---

## 4. Wgrywanie plików (Multipart)
- Tradycyjne formularze sieciowe przesyłają dane zakodowane jako `application/x-www-form-urlencoded` (pary klucz=wartość).
- Przesyłanie plików binarnych wymaga zastosowania typu MIME `multipart/form-data`, który dzieli ciało żądania HTTP na odrębne części (*parts*), z osobnymi nagłówkami i zawartością binarną.
- Kontener webowy wymaga oznaczenia servletu znacznikiem konfiguracji wieloczęściowej (`@MultipartConfig`), co instruuje kontener, by automatycznie sparsował strumień wejściowy na obiekty reprezentujące części formularza (`Part`).

---

## 5. Współdzielenie obiektów i poziomy zasięgów (Scopes)
Aplikacja webowa oparta o servlety udostępnia trzy podstawowe przestrzenie (konteksty) do przechowywania i współdzielenia atrybutów w pamięci:

| Poziom zasięgu | Obiekt kontenera | Czas życia | Dostępność / Zastosowanie |
| :--- | :--- | :--- | :--- |
| **Poziom żądania (Request Scope)** | `HttpServletRequest` | Od momentu wpłynięcia żądania HTTP do serwera do momentu odesłania odpowiedzi. | Dane specyficzne dla jednego zapytania użytkownika; przekazywanie danych pomiędzy filtrami, servletami a widokami (*forward*). |
| **Poziom sesji (Session Scope)** | `HttpSession` | Od momentu utworzenia sesji użytkownika do jej wygaśnięcia (*timeout*) lub jawnego unieważnienia (*invalidate*). | Dane specyficzne dla konkretnego użytkownika/klienta w ramach wielu kolejnych żądań (np. koszyk sklepowy, tożsamość zalogowanego). Identyfikowana typowo ciasteczkiem `JSESSIONID`. |
| **Poziom aplikacji (Application Scope)** | `ServletContext` | Od momentu uruchomienia/wdrożenia aplikacji do momentu jej zamknięcia na serwerze. | Dane globalne, współdzielone pomiędzy wszystkimi użytkownikami, wszystkimi sesjami i wszystkimi komponentami w całej aplikacji webowej (np. liczniki globalne, parametry konfiguracyjne aplikacji). |

---

## 6. Nasłuchiwacze zdarzeń w kontenerze (Listeners)
Listeners to komponenty implementujące interfejsy z rodziny `EventListener`, które są powiadamiane przez kontener o istotnych zdarzeniach cyklu życia lub modyfikacjach danych w poszczególnych kontekstach.

### Klasyfikacja listenerów:
1. **Kontekst aplikacji (`ServletContext`):**
   - `ServletContextListener`: reaguje na utworzenie aplikacji (inicjalizacja globalna przed obsłużeniem pierwszego żądania) oraz na jej niszczenie (sprzątanie zasobów).
   - `ServletContextAttributeListener`: reaguje na dodanie, usunięcie lub zastąpienie atrybutu w zasięgu aplikacji.
2. **Kontekst sesji (`HttpSession`):**
   - `HttpSessionListener`: reaguje na rozpoczęcie nowej sesji oraz jej wygaśnięcie/zniszczenie.
   - `HttpSessionActivationListener`: reaguje na procesy pasywacji (zrzucenia sesji z RAM na dysk lub do innego węzła klastra) oraz aktywacji w środowiskach klastrowych.
   - `HttpSessionAttributeListener`: reaguje na zmiany atrybutów w sesji użytkownika.
3. **Kontekst żądania (`ServletRequest`):**
   - `ServletRequestListener`: reaguje na rozpoczęcie przetwarzania każdego nowego żądania HTTP oraz na zakończenie jego obsługi.
   - `ServletRequestAttributeListener`: reaguje na modyfikację atrybutów w ramach pojedynczego żądania.

### Kolejność wykonywania zdarzeń:
- Deklaracja nasłuchiwaczy za pomocą adnotacji `@WebListener` **nie pozwala na zdefiniowanie kolejności ich uruchamiania**.
- Jeśli wymagana jest ściśle określona sekwencja (np. inicjalizacja modułu B wymaga wcześniejszego załadowania modułu A), jedynym standaryzowanym rozwiązaniem jest **zadeklarowanie listenerów w pliku `web.xml`**, gdzie kolejność definicji znaczników `<listener>` wyznacza kolejność ich wywoływania przez kontener.

---

## 7. Filtry (Filters) w architekturze Servlet
**Filtr** to komponent warstwy webowej (implementujący interfejs `Filter` lub rozszerzający klasę bazową `HttpFilter`), który potrafi przechwycić żądanie przed dotarciem do zasobu docelowego (servletu, strony) oraz odpowiedź powracającą z zasobu docelowego przed wysłaniem jej do klienta.

### Architektura łańcucha filtrów (Chain of Responsibility):
```
Klient  --->  [ Filtr 1 (przed) ]  --->  [ Filtr 2 (przed) ]  --->  [ Docelowy Servlet ]
                                                                             |
Klient  <---  [ Filtr 1 (po)    ]  <---  [ Filtr 2 (po)    ]  <--------------+
```

### Mechanizm działania:
- Kontener wywołuje metodę `doFilter(request, response, chain)`.
- Kod wykonany przed wywołaniem metody `chain.doFilter()` realizuje przetwarzanie wstępne żądania.
- Wywołanie `chain.doFilter()` przekazuje sterowanie do kolejnego filtra w łańcuchu lub – jeśli to ostatni filtr – do właściwego servletu.
- Kod umieszczony po instrukcji `chain.doFilter()` wykonuje się w drodze powrotnej i pozwala na manipulację wygenerowaną odpowiedzią HTTP.
- Filtr może również zdecydować o **przerwaniu przetwarzania** (brak wywołania `chain.doFilter()`), np. odsyłając bezpośrednio błąd 401 Unauthorized lub wykonując przekierowanie.

### Typowe zastosowania filtrów:
- Uwierzytelnianie i autoryzacja (sprawdzanie uprawnień do chronionych ścieżek URL).
- Zmiana kodowania znaków (np. wymuszenie UTF-8).
- Logowanie, audytowanie i monitorowanie czasu przetwarzania zapytań.
- Kompresja danych wyjściowych (np. kompresja GZIP).
- Szyfrowanie i deszyfrowanie.
- Zapewnienie nagłówków bezpieczeństwa (CORS, CSP).

### Ważne ograniczenie parametrów:
- Parametry żądania HTTP (`request.getParameter(...)`) są w modelu servletów **niemodyfikowalne (tylko do odczytu)**.
- Jeśli filtr musi przekazać dodatkowe informacje wyliczone w locie do dalszych warstw, **nie może zmienić parametrów**, lecz musi zapisać te wartości jako **atrybuty żądania** (`request.setAttribute(...)`).

### Kolejność filtrów:
- Podobnie jak w przypadku listenerów, adnotacja `@WebFilter` nie definiuje deterministycznej kolejności nakładania filtrów.
- Deterministyczną kolejność łańcucha filtrów ustala się wyłącznie poprzez sekwencję wpisów `<filter-mapping>` w pliku `web.xml`.

---

## 8. Miejsce servletów we współczesnych systemach
- W nowoczesnym programowaniu enterprise deweloper rzadko tworzy bezpośrednie servlety od zera do pisania logiki aplikacji.
- Servlety stanowią jednak **fundamentalny fundament niskopoziomowy**, na którym oparte są wszystkie wysokopoziomowe frameworki webowe Javy:
  - W **Jakarta Faces (JSF)** całe sterowanie spoczywa na jednym centralnym servlecie `FacesServlet` (wzorzec Front Controller).
  - W **JAX-RS (REST)** żądania odbiera dedykowany servlet mapujący endpointy.
  - We frameworkach zewnętrznych (np. **Spring MVC**) centralnym punktem wejścia jest `DispatcherServlet`.
- Bezpośrednia implementacja servletów i filtrów pozostaje nieodzowna w sytuacjach nietypowych: optymalizacja przesyłania dużych strumieni binarnych, customowe protokoły czy zaawansowane filtry infrastrukturalne.
