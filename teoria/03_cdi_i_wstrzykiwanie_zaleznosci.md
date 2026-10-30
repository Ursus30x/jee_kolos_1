# 03. Jakarta CDI i Wstrzykiwanie Zależności – Teoria na Kolokwium

## 1. Wzorzec Wstrzykiwania Zależności (Dependency Injection – DI)
- **Definicja:** Wzorzec projektowy polegający na tym, że obiekt nie odpowiada za samodzielne tworzenie, wyszukiwanie ani pozyskiwanie instancji innych obiektów (zależności), których potrzebuje do pracy. Zamiast tego zależności są dostarczane ("wstrzykiwane") do obiektu z zewnątrz przez zewnętrzny komponent zarządzający (kontener DI).
- **Problemy podejścia tradycyjnego (bez DI):**
  - Klasa samodzielnie tworzy zależności operatorem `new` lub korzysta ze statycznych fabryk (`Factory.getInstance()`) czy lokalizatorów usług (`JNDI lookup`).
  - Powstaje silne sprzężenie (*tight coupling*) pomiędzy klasą a konkretną implementacją zależności.
  - Niemożliwa lub wysoce utrudniona staje się izolacja komponentu w testach jednostkowych (brak możliwości podłożenia mocka).
  - Klasa jest zmuszona do zarządzania konfiguracją i cyklem życia swoich zależności.
- **Konsekwencje i zalety stosowania DI:**
  1. **Separacja odpowiedzialności:** Klasa skupia się wyłącznie na realizacji własnej logiki biznesowej, a odpowiedzialność za cykl życia, tworzenie i konfigurację zależności spada w całości na kontener.
  2. **Luźne powiązania (*Loose Coupling*):** Klasa zależy od abstrakcji (interfejsu), a nie od konkretnej implementacji.
  3. **Wymienność implementacji:** Bardzo łatwa podmiana dostawcy usługi (np. zmiana mechanizmu wysyłania powiadomień SMS na Email bez modyfikacji klasy biznesowej).
  4. **Wysoka testowalność:** W testach jednostkowych można łatwo przekazać obiekty zaślepiające (*mocks / stubs*).

---

## 2. Trzy formy wstrzykiwania zależności – Porównanie
Zależności mogą zostać przekazane do komponentu na trzy sposoby:

| Forma wstrzykiwania | Charakterystyka i zalety | Wady i ograniczenia |
| :--- | :--- | :--- |
| **Przez parametry konstruktora (Constructor Injection)** | - Umożliwia tworzenie obiektów **niemutowalnych** (*immutable*) – pola mogą być oznaczone jako `final`.<br>- Obiekt od momentu utworzenia jest w **stanie spójnym** (nie ma możliwości istnienia obiektu częściowo zainicjalizowanego).<br>- Nie wymaga kontenera ani refleksji do testów jednostkowych (wystarczy zwykłe wywołanie konstruktora). | - Wymaga zdefiniowania dedykowanego konstruktora.<br>- Brak konstruktora domyślnego może rodzić wyzwania przy generowaniu dynamicznych proxy (choć standard CDI radzi sobie z tym poprzez odpowiednie reguły). |
| **Przez metody ustawiające (Setter Injection)** | - Pozwala na opcjonalną podmianę zależności w trakcie życia obiektu.<br>- Czytelny podział na tworzenie obiektu (konstruktor domyślny) i jego konfigurację. | - Obiekt może przez pewien czas znajdować się w stanie niespójnym (utworzony, ale jeszcze nieskonfigurowany).<br>- Narzuca obecność publicznych setterów, które łamią enkapsulację i mogą być omyłkowo wywołane w złym momencie. |
| **Bezpośrednio do pól klasy (Field Injection)** | - Zwięzłość zapisu – brak konieczności pisania powtarzalnego kodu (boilerplate) w postaci konstruktorów i setterów. | - Wymaga użycia mechanizmu **refleksji** (naruszenie prywatności pól przez `setAccessible(true)`).<br>- Niemożliwe jest oznaczenie pól słowem `final`.<br>- Utrudnione testowanie jednostkowe bez uruchamiania środowiska kontenera lub refleksyjnych bibliotek testowych. |

---

## 3. DI a Inversion of Control (IoC) – Kluczowe Rozróżnienie!
*To jedno z najważniejszych i najczęściej mylonych pojęć w inżynierii oprogramowania.*

### Czym jest Inversion of Control (Odwrócenie Sterowania)?
- **IoC to ogólny wzorzec architektoniczny** dotyczący **przepływu sterowania** w aplikacji.
- W programowaniu tradycyjnym to kod napisany przez dewelopera kontroluje główną pętlę programu, decyduje kiedy wywołać bibliotekę i zarządza całym procesem wykonania.
- W Odwróceniu Sterowania następuje zmiana kierunku kontroli: to **kontener / framework kontroluje główny przepływ programu**. Kontener zarządza wątkami, nasłuchuje na zdarzenia zewnętrzne (np. nadejście pakietu sieciowego HTTP) i w odpowiednim momencie sam wywołuje kod użytkownika.
- Obrazuje to zasada hollywoodzka (*Hollywood Principle*): *"Nie dzwoń do nas, to my zadzwonimy do ciebie"*.

### Jaka jest relacja pomiędzy DI a IoC?
- **IoC i DI to nie są synonimy!**
- **IoC** to szeroka koncepcja architektoniczna (obejmująca m.in. pętle zdarzeń, frameworki webowe, kontenery servletów, wzorzec Template Method).
- **DI (Wstrzykiwanie Zależności)** to **konkretny wzorzec projektowy**, który stanowi jedną z realizacji idei IoC, ukierunkowaną ściśle na **rozwiązywanie zależności między obiektami**.
- **Współpraca w kontenerach:** Kontener IoC zarządza cyklem życia komponentów. Po utworzeniu instancji komponentu, kontener **wykorzystuje wzorzec DI**, aby dostarczyć do niego wszystkie wymagane referencje, a dopiero gdy obiekt jest kompletny i spójny, kontener przekazuje do niego sterowanie w odpowiedzi na zdarzenie.

---

## 4. Standardy: JSR 330 a Jakarta CDI
1. **JSR 330 (Dependency Injection for Java):**
   - Podstawowa, uniwersalna specyfikacja definiująca fundamentalne interfejsy i adnotacje dla języka Java (m.in. adnotację `@Inject` wskazującą punkt wstrzyknięcia, `@Qualifier`, `@Scope`, `@Singleton`).
   - Została zaadoptowana przez większość ekosystemów: Jakarta EE, Spring Framework, Google Guice, Micronaut.
2. **Jakarta Contexts and Dependency Injection (CDI):**
   - Kompleksowa specyfikacja platformy Jakarta EE (zgodna z JSR 330 i rozszerzająca ją).
   - Wprowadza pojęcie **kontekstowości (Contexts)** – wiązania cyklu życia obiektów z kontekstami protokołu sieciowego i aplikacji (żądanie, sesja, aplikacja, widok).
   - Dodaje zaawansowane mechanizmy: dekoratory (`@Decorator`), interceptory AOP (`@Interceptor`), zdarzenia i obserwatorów (`@Observes`), fabryki producentów (`@Produces`), dynamiczne proxy i typobezpieczne kwalifikatory.
   - Referencyjna implementacja: biblioteka **Weld**.

---

## 5. Konfiguracja projektu CDI – Rodzaje archiwów
Specyfikacja CDI dzieli archiwa aplikacji na dwa typy:
1. **Archiwum jawne (*Explicit Bean Archive*):**
   - Zawiera plik konfiguracyjny `beans.xml` w katalogu `WEB-INF/` (dla modułów WAR) lub `META-INF/` (dla modułów JAR).
   - Umożliwia zdefiniowanie atrybutu `bean-discovery-mode`:
     - `all`: kontener zarządza **wszystkimi** kwalifikującymi się klasami Javy w projekcie, nawet jeśli nie posiadają one żadnej adnotacji zasięgu (domyślnie przyjmują wtedy zasięg `@Dependent`).
     - `annotated`: kontener zarządza **wyłącznie** klasami, które posiadają jawną adnotację definiującą zasięg (tzw. *bean defining annotations*).
2. **Archiwum niejawne (*Implicit Bean Archive*):**
   - Archiwum nie zawiera pliku `beans.xml`.
   - W tym trybie kontener automatycznie zachowuje się tak, jakby tryb wynosił `annotated` – zarządza tylko klasami z jawną adnotacją zasięgu.

---

## 6. Warunki zarządzania klasą przez kontener CDI
Kontener CDI zarządza klasą (traktuje ją jako managed bean), jeżeli spełnia ona następujące wymagania formalne:
- Nie jest klasą wewnętrzną (niestatyczną klasą zagnieżdżoną).
- Jest klasą konkretną (nie jest klasą abstrakcyjną ani interfejsem) – wyjątkiem są klasy oznaczone jako `@Decorator`.
- Nie implementuje interfejsu rozszerzeń kontenera (`Extension`).
- Posiada konstruktor bezargumentowy LUB konstruktor oznaczony adnotacją `@Inject`, którego wszystkie parametry kontener potrafi wstrzyknąć.
- W trybie `annotated` (lub w archiwum implicit) – posiada adnotację definiującą zasięg.

---

## 7. Zasięgi (Scopes) w CDI – Cykl życia i widoczność obiektów
Zasięg określa:
- Kiedy instancja beana jest tworzona.
- Kiedy instancja beana jest niszczona.
- W jakich granicach kontekstowych ta sama instancja jest współdzielona pomiędzy różnymi wywołaniami.

### Przegląd zasięgów:

| Adnotacja zasięgu | Czas życia i zachowanie instancji | Dostępność / Kontekst |
| :--- | :--- | :--- |
| **`@Dependent`** *(Domyślny zasięg CDI)* | - **Pseudo-zasięg** (brak współdzielenia).<br>- Przy **każdym wstrzyknięciu tworzona jest zupełnie nowa instancja** beana.<br>- Cykl życia instancji jest ściśle związany z cyklem życia obiektu, do którego została wstrzyknięta (niszczenie następuje wraz z obiektem nadrzędnym). | Każde miejsce wstrzyknięcia otrzymuje osobną instancję. |
| **`@RequestScoped`** | - Instancja tworzona jest przy nadejściu pojedynczego żądania HTTP.<br>- Żyje przez czas przetwarzania tego żądania i jest niszczona w chwili odesłania odpowiedzi do klienta. | Wszystkie komponenty obsługujące dane żądanie HTTP odwołują się do tej samej instancji. |
| **`@ConversationScoped`** | - Reprezentuje kontekst konwersacji wieloetapowej (np. wielostronicowy formularz zamówienia, kreator zakupu).<br>- Dwa stany: **chwilowy** (*transient* – zachowuje się jak `@RequestScoped`) oraz **długotrwały** (*long-running* – przechodzi w ten stan po wywołaniu `conversation.begin()`).<br>- Bean żyje przez kolejne żądania HTTP aż do momentu jawnego wywołania `conversation.end()`. | Dostępny w obrębie kolejnych powiązanych żądań identyfikowanych parametrem konwersacji (CID). |
| **`@SessionScoped`** | - Instancja tworzona podczas pierwszego odwołania w danej sesji HTTP użytkownika.<br>- Żyje tak długo, jak aktywna jest sesja użytkownika (niszczona przy wylogowaniu lub po upływie czasu bezczynności). | Współdzielona pomiędzy wszystkimi żądaniami pochodzącymi od tego samego użytkownika w danej sesji. |
| **`@ApplicationScoped`** | - Dokładnie jedna instancja tworzona na cały czas działania aplikacji (od wdrożenia do usunięcia aplikacji z serwera).<br>- Stan jest współdzielony globalnie pomiędzy wszystkimi użytkownikami, sesjami i wątkami. | Cała aplikacja. |
| **`@Singleton`** *(JSR 330)* | - Podobnie jak `@ApplicationScoped` tworzy jedną instancję dla aplikacji.<br>- **Różnica teoretyczna:** W specyfikacji CDI (Weld) dla `@ApplicationScoped` kontener wstrzykuje **proxy klienta** (umożliwia to poprawną serializację, lazy loading i odświeżanie kontekstów), natomiast dla `@Singleton` wstrzykiwana jest **bezpośrednia referencja** do obiektu, co rodzi problemy przy serializacji sesji. Dlatego dokumentacja zaleca stosowanie `@ApplicationScoped`. | Cała aplikacja. |
| **`@ViewScoped`** *(Jakarta Faces)* | - Pochodzi ze specyfikacji Jakarta Faces (JSF).<br>- Instancja żyje tak długo, jak użytkownik pozostaje na tej samej stronie internetowej (widoku) i wykonuje akcje postback/AJAX.<br>- Niszczona w momencie nawigacji na inną stronę. | Bieżący widok strony JSF. |
| **`@TransactionScoped`** *(JTA)* | - Pochodzi ze specyfikacji Java Transaction API (JTA).<br>- Instancja żyje dokładnie w granicach aktywnej transakcji bazodanowej; niszczona po zatwierdzeniu (*commit*) lub wycofaniu (*rollback*) transakcji. | Bieżąca transakcja. |

---

## 8. Integracja CDI z Expression Language (`@Named`)
- Domyślnie beany zarządzane przez CDI są anonimowe dla warstwy prezentacji (widoków).
- Zastosowanie adnotacji `@Named` nadaje beanowi nazwę symboliczną (domyślnie nazwa klasy z małej litery).
- Umożliwia to bezpośrednie odwoływanie się do właściwości i metod beana w szablonach widoku (JSF/Facelets) za pomocą języka wyrażeń Expression Language (np. `#{bookView.books}`).
