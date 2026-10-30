# 06. Jakarta RESTful Web Services (JAX-RS) – Teoria na Kolokwium

## 1. Usługi sieciowe i styl architektoniczny REST
- **Usługi sieciowe (Web Services):** Oprogramowanie umożliwiające komunikację i wymianę danych pomiędzy heterogenicznymi aplikacjami (napisanymi w różnych językach, działającymi na różnych platformach) za pośrednictwem sieci i standardowych protokołów internetowych (przede wszystkim HTTP) przy użyciu uniwersalnych formatów danych (JSON, XML).
- **Czym jest REST (Representational State Transfer)?**
  - REST to **styl architektoniczny** projektowania rozproszonych systemów hipermedialnych (nie jest protokołem ani biblioteką, lecz zbiorem reguł i ograniczeń architektonicznych).
  - **Kluczowe założenia stylu REST:**
    1. **Bezstanowość (Statelessness):** Serwer nie przechowuje stanu sesji klienta pomiędzy kolejnymi żądaniami. Każde żądanie przychodzące od klienta musi zawierać kompletny zestaw informacji niezbędnych do jego poprawnego przetworzenia (w tym dane uwierzytelniające). Pozwala to na pełną odporność na restarty serwera oraz łatwe horyzontalne skalowanie i load balancing.
    2. **Architektura Klient-Serwer:** Ścisła separacja interfejsu użytkownika od logiki przechowywania danych.
    3. **Buforowalność (Cacheability):** Odpowiedzi serwera muszą jawnie definiować, czy mogą być buforowane przez klienta lub serwery pośredniczące (proxy, CDN), co drastycznie redukuje opóźnienia i obciążenie sieci.
    4. **Jednolity interfejs (Uniform Interface):** Standaryzowany sposób manipulacji zasobami za pomocą uniwersalnych metod protokołu HTTP.
    5. **System warstwowy (Layered System):** Klient nie wie (i nie musi wiedzieć), czy łączy się bezpośrednio z serwerem końcowym, czy z pośrednikiem (bramą API, firewallem, warstwą cache).

---

## 2. Model Dojrzałości Usług REST (Richardson Maturity Model)
*Fundamentalne pytanie sprawdzające zrozumienie idei "prawdziwego" RESTful API (od zera do pełnej dojrzałości).*

```
+-----------------------------------------------------------+
| Poziom 3: Sterowanie Hipermediami (HATEOAS)               |
+-----------------------------------------------------------+
                              ^
                              |
+-----------------------------------------------------------+
| Poziom 2: Czasowniki HTTP (HTTP Verbs & Status Codes)     |
+-----------------------------------------------------------+
                              ^
                              |
+-----------------------------------------------------------+
| Poziom 1: Zasoby (Resources & URIs)                       |
+-----------------------------------------------------------+
                              ^
                              |
+-----------------------------------------------------------+
| Poziom 0: Bagno POX (The Swamp of POX / HTTP jako tunel)  |
+-----------------------------------------------------------+
```

### Szczegółowy opis poziomów:
1. **Poziom 0: The Swamp of POX (Bagno Zwykłego XML/JSON – HTTP jako tunel):**
   - Protokół HTTP jest traktowany wyłącznie jako przezroczysty tunel transportowy (mechanizm RPC – Remote Procedure Call).
   - Aplikacja posiada zazwyczaj **jeden wspólny punkt końcowy (pojedynczy adres URL)**, a wszystkie operacje są wysyłane za pomocą metody **POST**.
   - Nazwa wywoływanej operacji i parametry są ukryte w ciele wiadomości (np. SOAP, XML-RPC).
2. **Poziom 1: Zasoby (Resources):**
   - Wprowadzenie pojęcia **adresowalnych zasobów**. Zamiast jednego wspólnego adresu, każde pojęcie biznesowe otrzymuje własny, unikalny identyfikator URI (np. `/api/ksiazki`, `/api/ksiazki/15`).
   - Nadal jednak operacje bywają wykonywane jedną metodą (np. wyłącznie POST lub wyłącznie GET).
3. **Poziom 2: Czasowniki HTTP (HTTP Verbs):**
   - Poprawne i świadome wykorzystanie standardowych **metod protokołu HTTP** zgodnie z ich semantyką:
     - `GET` – pobieranie danych.
     - `POST` – dodawanie nowego zasobu do kolekcji.
     - `PUT` – aktualizacja całego zasobu / utworzenie zasobu pod zadanym identyfikatorem.
     - `DELETE` – usuwanie zasobu.
   - Prawidłowe stosowanie **kodów odpowiedzi HTTP** (np. `200 OK`, `201 Created`, `400 Bad Request`, `404 Not Found`).
4. **Poziom 3: Sterowanie Hipermediami – HATEOAS (Hypermedia As The Engine Of Application State):**
   - Najwyższy poziom dojrzałości REST.
   - Serwer w zwracanej odpowiedzi (np. w dokumencie JSON) przesyła nie tylko surowe dane biznesowe, ale również **zestaw hiperłączy (linków nawigacyjnych)** wskazujących klientowi, jakie akcje może podjąć w danym momencie i jakie powiązane zasoby są dla niego dostępne.
   - Klient nie musi znać na sztywno struktury adresów URL całego API – nawiguje po systemie dynamicznie na podstawie odnośników dostarczonych w odpowiedziach serwera.

---

## 3. Semantyka metod HTTP, Bezpieczeństwo i Idempotentność
*To jedno z najczęstszych pytań teoretycznych na egzaminach z systemów rozproszonych i JEE.*

### Pojęcia kluczowe:
- **Metoda bezpieczna (Safe method):** Metoda, której wywołanie **nie powoduje żadnych skutków ubocznych na serwerze** (nie modyfikuje stanu bazy danych ani zasobów; służy wyłącznie do odczytu).
- **Metoda idempotentna (Idempotent method):** Metoda, której **wielokrotne wywołanie z identycznymi parametrami wywołuje dokładnie taki sam skutek uboczny na serwerze, jak pojedyncze wywołanie**. Nawet jeśli sieć powtórzy pakiet 5 razy, stan serwera pozostanie niezmieniony w stosunku do pierwszego wywołania.

### Zestawienie metod:

| Metoda HTTP | Bezpieczna? | Idempotentna? | Semantyczne przeznaczenie w REST |
| :--- | :---: | :---: | :--- |
| **`GET`** | **TAK** | **TAK** | Pobranie reprezentacji istniejącego zasobu lub listy zasobów. Nigdy nie modyfikuje stanu serwera. |
| **`POST`** | NIE | **NIE** | Dodanie nowego elementu do kolekcji zasobów (serwer sam generuje nowy identyfikator) lub wywołanie niespecyficznej operacji biznesowej. Każde kolejne wywołanie tworzy kolejny, nowy rekord! |
| **`PUT`** | NIE | **TAK** | Całkowite zastąpienie istniejącego zasobu nową reprezentacją LUB utworzenie zasobu pod wskazanym przez klienta adresem URI. Wielokrotne wysłanie tego samego obiektu pod ten sam adres daje ten sam stan końcowy w bazie. |
| **`PATCH`** | NIE | *Zazwyczaj NIE* | Częściowa aktualizacja zasobu (modyfikacja tylko wybranych pól). |
| **`DELETE`** | NIE | **TAK** | Usunięcie wskazanego zasobu. Pierwsze wywołanie usuwa rekord, kolejne wywołania nie zmieniają już stanu bazy (zasób pozostaje usunięty). |

---

## 4. Zasady projektowania identyfikatorów URI i typy parametrów
- **Struktura adresów:** Identyfikatory URI powinny operować na **rzeczownikach w liczbie mnogiej**, odzwierciedlając hierarchię powiązań między zasobami (np. `/api/orders/42/items/3`), unikając czasowników w ścieżkach (np. unikać błędnego `/api/deleteOrder`).
- **Typy parametrów w zapytaniach:**
  1. **Parametry ścieżki (Path Parameters):** Integralna część adresu URL służąca do **jednoznacznej identyfikacji konkretnego zasobu** w hierarchii (np. `/api/books/{id}`).
  2. **Parametry zapytania (Query Parameters):** Doklejane na końcu adresu po znaku `?`, służące do **doprecyzowania operacji na kolekcji zasobów**:
     - Filtrowanie: `/api/books?author=Lem`
     - Sortowanie: `/api/books?sort=title`
     - Stronicowanie (Paginacja): `/api/books?offset=20&limit=10`
  3. **Parametry formularza (Form Parameters):** Przesyłane w ciele żądania zakodowanego jako `application/x-www-form-urlencoded`.
  4. **Ciało żądania (Request Body):** Strukturalna reprezentacja obiektu (JSON/XML) przesyłana w żądaniach tworzenia lub aktualizacji zasobów.

---

## 5. Kody statusu odpowiedzi HTTP w architekturze REST
- **Kody sukcesu (2xx):**
  - `200 OK`: Standardowa odpowiedź sukcesu przy pobraniu (GET) lub udanej aktualizacji (PUT).
  - `201 Created`: Nowy zasób został pomyślnie utworzony (POST/PUT); odpowiedź **musi zawierać nagłówek `Location`** wskazujący bezpośredni adres URI nowo powstałego zasobu.
  - `202 Accepted`: Żądanie zostało przyjęte do asynchronicznego przetworzenia, ale operacja jeszcze się nie zakończyła.
  - `204 No Content`: Operacja zakończona sukcesem, ale serwer celowo nie zwraca żadnego ciała odpowiedzi (często przy DELETE).
- **Kody błędów klienta (4xx):**
  - `400 Bad Request`: Błąd składniowy zapytania lub błąd walidacji danych wejściowych.
  - `401 Unauthorized`: Brak uwierzytelnienia (brak poświadczeń tożsamości lub są one niepoprawne).
  - `403 Forbidden`: Użytkownik jest poprawnie uwierzytelniony, ale **nie posiada wystarczających uprawnień (ról)** do wykonania żądanej operacji na zasobie.
  - `404 Not Found`: Zasób pod wskazanym adresem URI nie istnieje.
  - `409 Conflict`: Konflikt stanu zasobu (np. próba utworzenia rekordu o unikalnym identyfikatorze, który już istnieje).
- **Kody błędów serwera (5xx):**
  - `500 Internal Server Error`: Nieobsłużony wyjątek lub awaria po stronie serwera aplikacji.

---

## 6. Bezpieczeństwo w przeglądarkach: SOP a CORS
*Kluczowe zagadnienie teoretyczne dotyczące architektury nowoczesnych aplikacji webowych (Single Page Apps / Frontend vs API).*

### Czym jest Same Origin Policy (SOP)?
- **SOP (Polityka Jednego Pochodzenia):** Fundamentalny mechanizm bezpieczeństwa wbudowany w każdą przeglądarkę internetową.
- Definicja **Pochodzenia (Origin):** Pochodzenie wyznacza trójka parametrów: `(protokół, host/domena, port)`. Jeśli chociaż jeden element się różni, przeglądarka traktuje to jako obce pochodzenie (*cross-origin*).
- **Działanie:** Domyślnie skrypt JavaScript uruchomiony na stronie pochodzącej z domeny `https://mojadomena.pl` **nie może odczytać odpowiedzi na zapytanie AJAX wysłane do obcej domeny** (np. `https://api.innybank.com`).
- **Cel:** Ochrona użytkownika przed atakami typu Cross-Site Scripting (XSS) i kradzieżą danych sesyjnych z innych otwartych serwisów.

### Czym jest Cross-Origin Resource Sharing (CORS)?
- **CORS to mechanizm pozwalający serwerowi na kontrolowane poluzowanie restrykcji SOP.**
- Umożliwia serwerowi backendowemu poinstruowanie przeglądarki klienta, z jakich zewnętrznych domen dopuszcza on wywołania skryptowe.
- **Zasada działania nagłówków CORS:**
  - Serwer przesyła w odpowiedzi nagłówki kontrolne, m.in.:
    - `Access-Control-Allow-Origin`: lista dozwolonych domen (lub `*` dla publicznych API).
    - `Access-Control-Allow-Methods`: dozwolone czasowniki HTTP (`GET, POST, PUT, DELETE, OPTIONS`).
    - `Access-Control-Allow-Headers`: akceptowane niestandardowe nagłówki żądania (np. `Authorization, Content-Type`).
    - `Access-Control-Allow-Credentials`: informacja, czy przeglądarka może dołączyć ciasteczka i poświadczenia w zapytaniu cross-origin.
  - W przypadku zapytań złożonych (np. metoda PUT/DELETE lub nagłówki niestandardowe) przeglądarka przed właściwym żądaniem automatycznie wysyła zapytanie wstępne (**preflight request**) metodą `OPTIONS`, aby sprawdzić, czy serwer wyraża zgodę na operację.
