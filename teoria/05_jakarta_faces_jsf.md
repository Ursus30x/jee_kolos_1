# 05. Jakarta Faces (JSF) – Teoria na Kolokwium

## 1. Czym jest Jakarta Faces (dawniej JavaServer Faces – JSF)?
- **Definicja:** Komponentowy framework platformy Jakarta EE służący do budowy graficznego interfejsu użytkownika (UI) dla aplikacji internetowych, działający po stronie serwera (**server-side component-based UI**).
- **Zasada działania:**
  - W przeciwieństwie do frameworków zorientowanych na żądania (ang. *request-driven / action-oriented*, takich jak Spring MVC czy Struts), JSF opiera się na **drzewie komponentów interfejsu (Component Tree)** utrzymywanym w pamięci serwera.
  - Komponenty interfejsu (pola tekstowe, przyciski, tabele danych) są reprezentowane na serwerze jako obiekty Javy, które tłumaczą się (*renderują*) do postaci standardowego kodu HTML/CSS/JavaScript wysyłanego do przeglądarki.
  - Zdarzenia wygenerowane przez użytkownika w przeglądarce (kliknięcie przycisku, zmiana wartości w polu) są przesyłane do serwera i odwzorowywane na wywołania metod w komponentach Javy (Backing Beans).
- **Rola servletu `FacesServlet`:**
  - Centralny punkt wejściowy całej architektury JSF (wzorzec *Front Controller*).
  - Przechwytuje wszystkie żądania skierowane do stron JSF (typowo z rozszerzeniem `*.xhtml` lub prefiksem `/faces/*`), inicjuje cykl życia żądania JSF i zarządza generowaniem odpowiedzi.
- **Przestrzenie nazw w plikach widoku Facelets (`.xhtml`):**
  - Komponenty standardowe HTML (`jakarta.faces.html`): np. przyciski, formularze, tabele.
  - Mechanizmy rdzenne (`jakarta.faces.core`): walidatory, konwertery, parametry, obsługa zdarzeń.
  - Szablony Facelets (`jakarta.faces.facelets`): mechanizm kompozycji i szablonowania stron.

---

## 2. Expression Language (EL) w JSF – Porównanie `${...}` a `#{...}`
*Jest to fundamentalne pytanie teoretyczne dotyczące integracji widoku z modelem.*

| Własność | Wyrażenie `${wyrazenie}` | Wyrażenie `#{wyrazenie}` |
| :--- | :--- | :--- |
| **Typ ewaluacji** | **Natychmiastowa (Immediate evaluation)** | **Odroczona (Deferred evaluation)** |
| **Kiedy jest obliczane?** | Obliczane **dokładnie raz**, na samym początku przetwarzania strony / w fazie generowania widoku. | Obliczane w **dowolnej fazie cyklu życia JSF**, w momencie gdy wartość jest rzeczywiście potrzebna kontenerowi. |
| **Kierunek dostępu do danych** | **Tylko do odczytu (rvalue)** – potrafi jedynie pobrać wartość ze zmiennej i wstawić ją w treść strony. | **Do odczytu i do zapisu (rvalue i lvalue)** – potrafi odczytać wartość z beana do wyświetlenia, ale potrafi również **zapisać nową wartość wprowadzoną przez użytkownika** z powrotem do beana. |
| **Główne zastosowanie** | Historycznie w starszych stronach JSP, dynamiczny tekst statyczny. | **Standard w Jakarta Faces (JSF)**; podstawa dwukierunkowego wiązania danych (*two-way data binding*) oraz wywoływania metod akcji beana (np. `action="#{bean.zapisz}"`). |

---

## 3. Backing Beans i Zasięg `@ViewScoped`
- **Backing Bean:** Zarządzany komponent CDI (`@Named`), którego zadaniem jest dostarczanie danych do komponentów widoku oraz obsługa akcji użytkownika (logika prezentacji).
- **Dlaczego zasięg `@ViewScoped` jest kluczowy dla JSF?**
  - W tradycyjnych aplikacjach webowych występował konflikt zasięgów:
    - `@RequestScoped` był zbyt krótki: każda interakcja użytkownika (np. sortowanie tabeli przez AJAX, wieloetapowa walidacja) powodowała zniszczenie beana i utratę stanu widoku.
    - `@SessionScoped` był zbyt długi: obiekt żył w pamięci serwera przez całą sesję, marnując pamięć RAM serwera i powodując konflikty, gdy użytkownik otworzył tę samą stronę w dwóch kartach przeglądarki.
  - **Zasięg `@ViewScoped` (zasięg widoku):**
    - Bean powstaje w momencie otwarcia danej strony internetowej.
    - **Pozostaje aktywny w pamięci serwera tak długo, jak użytkownik pozostaje na tej samej stronie** i wykonuje operacje na niej (żądania postback, zapytania asynchroniczne AJAX).
    - Jest automatycznie niszczony w chwili, gdy użytkownik nawiguje na inną stronę (inny widok).

---

## 4. Cykl Życia Żądania w JSF (6 Faz – Bardzo ważne na kolokwium!)
Każde żądanie skierowane do `FacesServlet` przechodzi przez ściśle zdefiniowaną sekwencję **sześciu faz**:

```
[ Żądanie HTTP ]
       |
       v
+-----------------------------------------------------------+
| 1. Restore View (Odtworzenie Widoku)                     |
+-----------------------------------------------------------+
       |
       v
+-----------------------------------------------------------+
| 2. Apply Request Values (Pobranie Wartości Żądania)       |
+-----------------------------------------------------------+
       |
       v
+-----------------------------------------------------------+
| 3. Process Validations (Przetwarzanie Walidacji)          |
+-----------------------------------------------------------+
       |                                          |
       | Sukces                                   | BŁĄD WALIDACJI / KONWERSJI
       v                                          | (Skok bezpośredni!)
+-------------------------------------------------+         |
| 4. Update Model Values (Aktualizacja Modelu)    |         |
+-------------------------------------------------+         |
       |                                                    |
       v                                                    |
+-------------------------------------------------+         |
| 5. Invoke Application (Wywołanie Aplikacji)     |         |
+-------------------------------------------------+         |
       |                                                    |
       v                                                    |
+-----------------------------------------------------------+
| 6. Render Response (Generowanie Odpowiedzi) <-------------+
+-----------------------------------------------------------+
       |
       v
[ Odpowiedź HTML do przeglądarki ]
```

### Szczegółowy opis każdej fazy:
1. **Faza 1: Restore View (Odtworzenie Widoku):**
   - Kontener odszukuje i rekonstruuje drzewo komponentów strony, która została zażądana.
   - Jeśli to pierwsze wejście na stronę (tzw. żądanie *non-postback* / initial request), tworzone jest nowe, puste drzewo komponentów i następuje natychmiastowy skok do fazy 6.
   - Jeśli to kolejne żądanie z tej samej strony (żądanie *postback*, np. kliknięcie przycisku formularza), drzewo jest przywracane ze stanu zapisanego po stronie serwera lub klienta.
2. **Faza 2: Apply Request Values (Pobranie i Przypisanie Wartości):**
   - Każdy komponent w drzewie widoku odczytuje swoje parametry przesłane w ciele żądania HTTP i zapisuje je w sobie jako lokalną, surową wartość tekstową (*submitted value*).
3. **Faza 3: Process Validations (Przetwarzanie Walidacji i Konwersji):**
   - Surowe wartości tekstowe są konwertowane przez zarejestrowane **konwertery** na odpowiednie typy języka Java (np. String na Integer, Date).
   - Następnie uruchamiane są **walidatory** (np. sprawdzanie zakresu liczb, poprawności adresu email, długości tekstu).
   - **Kluczowy mechanizm błędu:** Jeśli jakakolwiek konwersja lub walidacja nie powiedzie się, do kontekstu dodawany jest komunikat błędu (`FacesMessage`), a kontener **natychmiast przerywa przetwarzanie i przeskakuje bezpośrednio do Fazy 6 (Render Response)**, pomijając aktualizację modelu i logikę biznesową!
4. **Faza 4: Update Model Values (Aktualizacja Modelu Danych):**
   - Wykonuje się tylko wtedy, gdy faza walidacji zakończyła się pełnym sukcesem.
   - Przekonwertowane i poprawne wartości z komponentów widoku są wpisywane bezpośrednio do właściwości powiązanych Backing Beans (za pośrednictwem wyrażeń `#{...}`).
5. **Faza 5: Invoke Application (Wywołanie Logiki Aplikacji):**
   - Uruchamiane są docelowe metody akcji zdefiniowane przez programistę w Backing Beans (np. metoda zapisu encji do bazy powiązana z przyciskiem `<h:commandButton action="#{bean.zapisz}">`).
   - Metoda akcji może zwrócić wynik nawigacji (nazwę widoku docelowego).
6. **Faza 6: Render Response (Generowanie Odpowiedzi):**
   - Drzewo komponentów jest trawersowane i tłumaczone na kod HTML/XML.
   - Stan drzewa komponentów jest zapisywany na potrzeby kolejnego żądania postback.
   - Odpowiedź HTML jest odsyłana strumieniem sieciowym do przeglądarki użytkownika.

---

## 5. Problem wielokrotnego zatwierdzania i Idiom POST-REDIRECT-GET (PRG)
### Na czym polega problem wielokrotnego przesłania (Duplicate Form Submission)?
- Gdy użytkownik wysyła formularz (żądanie HTTP POST), serwer przetwarza dane (np. wykonuje transakcję przelewu pieniędzy w banku) i w tradycyjnym modelu zwraca stronę wynikową z kodem statusu `200 OK`.
- Przeglądarka internetowa zapamiętuje, że ostatnią operacją było żądanie POST.
- Jeśli użytkownik wciśnie teraz klawisz **F5 (Odśwież)**, przeglądarka spróbuje **ponownie wysłać identyczne żądanie POST** z tymi samymi danymi formularza, co spowoduje ponowne, niepożądane wykonanie operacji biznesowej (np. podwójny przelew).

### Rozwiązanie: Idiom POST-REDIRECT-GET (PRG)
- Zamiast zwracać kod `200 OK` bezpośrednio po wykonaniu operacji POST zmieniającej stan serwera, serwer zwraca odpowiedź **przekierowującą (kod HTTP 302 / 303 Redirect)** z nagłówkiem `Location: adres_strony_wynikowej`.
- Przeglądarka po odebraniu kodu 302 automatycznie wykonuje bezpieczne żądanie **HTTP GET** pod wskazany adres.
- W rezultacie w historii przeglądarki jako ostatnie żądanie widnieje GET. Ewentualne odświeżenie strony (F5) powtarza jedynie bezpieczne pobranie widoku (GET), nie niosąc ryzyka powtórzenia transakcji.
- **W JSF:** Idiom ten wymusza się poprzez dodanie parametru nawigacyjnego `faces-redirect=true` do adresu docelowego widoku.

---

## 6. Szablony stron (Facelets Templating)
Mechanizm Facelets umożliwia tworzenie spójnego wyglądu aplikacji poprzez separację szablonu głównego od treści poszczególnych podstron:
- **Szablon nadrzędny:** Definiuje ogólny układ graficzny (nagłówek, menu, stopka) i wyznacza punkty zmienne za pomocą znacznika `<ui:insert name="nazwaSekcji"/>`.
- **Podstrony klienckie:** Korzystają z szablonu za pomocą `<ui:composition template="sciezkaDoSzablonu">` i wstrzykują specyficzną zawartość w wyznaczone punkty zmienne przy użyciu znacznika `<ui:define name="nazwaSekcji">`.

---

## 7. Rola Konwerterów, Walidatorów i Deskryptora `faces-config.xml`
- **Konwertery (Converters):** Odpowiadają za translację pomiędzy formatem tekstowym przesyłanym protokołem HTTP a obiektami Javy (i odwrotnie: formatowanie obiektu Javy do tekstu w polu formularza).
- **Walidatory (Validators):** Zapewniają integralność danych po stronie prezentacji przed dopuszczeniem ich do warstwy biznesowej.
- **Deskryptor `faces-config.xml`:** Opcjonalny plik konfiguracyjny frameworka JSF. Służy do globalnej rejestracji reguł nawigacji pomiędzy widokami, własnych fabryk komponentów, niestandardowych procedur obsługi widoku (*ViewHandler*) oraz definicji internacjonalizacji (i18n). We współczesnych wersjach większość jego ról można zastąpić adnotacjami w kodzie.
