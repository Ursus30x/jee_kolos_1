# 04. Jakarta Persistence (JPA) – Teoria na Kolokwium

## 1. Czym jest Jakarta Persistence (dawniej JPA)?
- **Definicja:** Standardowa specyfikacja platformy Jakarta EE służąca do **odwzorowania obiektowo-relacyjnego (ORM – Object-Relational Mapping)**.
- **Rola:** Stanowi most pomiędzy obiektowym modelem domenowym w języku Java (klasy, obiekty, relacje dziedziczenia i asocjacji) a relacyjnym modelem w bazie danych (tabele, kolumny, klucze główne i obce).
- **Zalety w systemach enterprise:**
  - Eliminuje konieczność ręcznego pisania powtarzalnego kodu dostępu do danych (Data Access Object – DAO) oraz niskopoziomowych operacji JDBC.
  - Zapewnia automatyczne zarządzanie transakcjami o własnościach **ACID** (*Atomicity, Consistency, Isolation, Durability*).
  - Zapewnia **niezależność od konkretnego dostawcy bazy danych** – dostawca JPA (np. Hibernate) dynamicznie generuje dialekt SQL dopasowany do bazy podłączonej sterownikiem JDBC (PostgreSQL, Oracle, MySQL, DB2 itp.).
- **Implementacje JPA (JPA to jedynie interfejs/standard, a nie kod wykonawczy):**
  - **EclipseLink** – oficjalna implementacja referencyjna specyfikacji Jakarta Persistence (domyślnie używana w serwerze GlassFish).
  - **Hibernate ORM** – najpopularniejsza otwarta implementacja firmy Red Hat (domyślnie używana w serwerze WildFly).
  - **Apache OpenJPA** – implementacja fundacji Apache.

---

## 2. Pojęcie klasy encyjnej (Entity) i jej wymogi formalne
Encja to zwykła klasa języka Java (klasa typu POJO – *Plain Old Java Object*), która reprezentuje rekord/wiersz w tabeli relacyjnej bazy danych.

### Wymogi formalne stawiane klasom encyjnym przez specyfikację:
1. Musi być oznaczona adnotacją `@Entity`.
2. Musi posiadać publiczny lub chroniony (*protected*) **konstruktor bezargumentowy** (niezbędny dla dostawcy JPA do tworzenia instancji przez refleksję podczas odczytu danych z bazy).
3. Klasa nie może być oznaczona jako `final` (dostawcy ORM generują dynamiczne proxy rozszerzające klasę encji do obsługi *lazy loading*).
4. Żadne pole instancji podlegające mapowaniu do bazy nie może być publiczne (dostęp do pól realizowany jest przez gettery/settery lub bezpośrednio przez prywatne pola – enkapsulacja).
5. Musi posiadać zdefiniowany jednoznaczny **klucz główny** (pole oznaczone `@Id`). Klucze złożone muszą być reprezentowane przez osobną klasę posiadającą poprawnie zaimplementowane metody `equals()` i `hashCode()`.

---

## 3. Strategie generowania wartości klucza głównego (`@GeneratedValue`)
Wartość identyfikatora encji może być wprowadzana manualnie przez programistę lub generowana automatycznie przez jedną z czterech strategii:

| Strategia (`GenerationType`) | Mechanizm działania | Własności i charakterystyka |
| :--- | :--- | :--- |
| **`IDENTITY`** | Klucz jest generowany automatycznie przez mechanizm samej kolumny bazy danych (np. typ kolumny `AUTO_INCREMENT` w MySQL lub `SERIAL`/`IDENTITY` w PostgreSQL). | Wartość klucza jest znana dopiero **po fizycznym wykonaniu instrukcji `INSERT`** w bazie danych, co wyłącza niektóre optymalizacje buforowania wsadowego (*batch insert*). |
| **`SEQUENCE`** | Wartości są pobierane z dedykowanego obiektu bazodanowego – **sekwencji** (wspieranej m.in. przez PostgreSQL i Oracle). | Dostawca JPA może pobrać z góry pulę identyfikatorów z bazy, dzięki czemu klucz jest znany w pamięci **przed fizycznym wykonaniem instrukcji `INSERT`**, co umożliwia wydajne operacje wsadowe. |
| **`TABLE`** | Wartości kolejnych identyfikatorów są przechowywane i inkrementowane w specjalnie utworzonej, odrębnej tabeli w bazie danych. | Strategia w 100% przenośna pomiędzy dowolnymi relacyjnymi bazami danych, ale charakteryzuje się najniższą wydajnością (narzut blokowania tabeli liczników przy pobieraniu każdego klucza). |
| **`AUTO`** | Wybór strategii jest w całości delegowany do dostawcy JPA w oparciu o wykryty dialekt bazy danych. | W zależności od bazy danych zostanie automatycznie wybrana sekwencja lub tabela/kolumna identity. |

---

## 4. Związki między encjami i pojęcie strony właścicielskiej
W relacyjnych bazach danych jedynym mechanizmem łączącym tabele są **klucze obce**, które z natury są asocjacjami jednokierunkowymi (tabela podrzędna wskazuje na nadrzędną). W obiektowym świecie Javy naturalne są relacje dwukierunkowe (obiekt A odwołuje się do B, a obiekt B do A).

### Typy liczności asocjacji:
- `@OneToOne` – jeden do jednego (np. Użytkownik <-> Profil).
- `@OneToMany` / `@ManyToOne` – jeden do wielu / wiele do jednego (np. Dział <-> Pracownicy).
- `@ManyToMany` – wiele do wielu (np. Książka <-> Autor; na poziomie bazy modelowane tabelą łączącą).

### Strona właścicielska (*Owning Side*) a atrybut `mappedBy`:
- W związku dwukierunkowym jedna ze stron **musi być wyznaczona jako strona właścicielska relacji**.
- **Strona właścicielska:** to ta klasa encyjna, która **fizycznie odpowiada za aktualizację kolumny klucza obcego** (lub tabeli łączącej) w bazie danych. Jest to strona bez atrybutu `mappedBy` (w relacji `@OneToMany` / `@ManyToOne` stroną właścicielską jest **zawsze strona `@ManyToOne`**, bo to w tabeli wielu znajduje się fizyczna kolumna klucza obcego).
- **Strona odwrócona / pasywna (*Inverse Side*):** oznaczana atrybutem `mappedBy = "nazwaPolaWStronieWlascicielskiej"`. Zmiany dokonane na liście w stronie pasywnej nie powodują wygenerowania zapytań SQL, chyba że relacja została zaktualizowana również po stronie właściciela!

### Kaskadowość operacji (`cascade`):
- Mechanizm pozwalający na automatyczne kaskadowe propagowanie operacji wykonanych na encji nadrzędnej na encje powiązane (np. usunięcie dokumentu automatycznie usuwa wszystkie jego pozycje: `CascadeType.REMOVE`, `CascadeType.PERSIST`, `CascadeType.ALL`).

---

## 5. Jednostka Trwałości (Persistence Unit) i `persistence.xml`
- **Definicja:** Jednostka trwałości grupuje logicznie powiązany zestaw klas encyjnych, które są odwzorowywane na tę samą bazę danych.
- Standardowo definiowana w pliku `META-INF/persistence.xml`.
- **Dwa typy zarządzania transakcjami (`transaction-type`):**
  1. **`JTA` (Java Transaction API):** używany w aplikacjach wdrażanych na serwerach aplikacji Jakarta EE. Transakcjami zarządza kontener serwera. Umożliwia transakcje rozproszone pomiędzy wieloma bazami danych i systemami kolejkowymi. Połączenie pobierane jest ze źródła danych JNDI (`DataSource`).
  2. **`RESOURCE_LOCAL`:** transakcjami zarządza bezpośrednio aplikacja poprzez interfejs `EntityTransaction`. Typowo stosowany w standalone Java SE lub prostych aplikacjach bez kontenera EE. Dane dostępowe (URL, sterownik JDBC, login, hasło) definiowane są bezpośrednio w pliku konfiguracyjnym.

---

## 6. Kontekst Trwałości (Persistence Context) i Tożsamość Bazodanowa
1. **Kontekst Trwałości (*Persistence Context*):**
   - Zbiór wszystkich instancji encji, które w danej chwili są **zarządzane (*managed*)** przez obiekt `EntityManager`.
   - Działa jako **pamięć podręczna pierwszego poziomu (1st Level Cache)** w ramach bieżącej transakcji lub sesji.
   - Realizuje mechanizm automatycznego wykrywania modyfikacji (**dirty checking**) – przy zatwierdzaniu transakcji dostawca JPA porównuje stan obiektów w kontekście ze stanem początkowym i automatycznie generuje odpowiednie instrukcje SQL `UPDATE`.
2. **Tożsamość Bazodanowa (*Persistent Identity*):**
   - Tożsamość encji w kontekście trwałości wyznaczana jest przez jej **klucz główny (`@Id`) w połączeniu z typem klasy encji**.
   - Gwarancja powtarzalności: W obrębie jednego kontekstu trwałości odpytanie bazy o encję o tym samym kluczu głównym zawsze zwróci **dokładnie tę samą instancję w pamięci** (`a == b`).

---

## 7. Stany encji i cykl życia obiektu (Kluczowe pytanie z teorii JPA!)
Obiekt encyjny w JPA może znajdować się w jednym z **czterech stanów**:

```
                       +-------------------+
                       |    New/Transient  |
                       +-------------------+
                                 |
                                 | persist()
                                 v
   find(), query,      +-------------------+       remove()
 +-------------------> |      Managed      | --------------------+
 |                     +-------------------+                     |
 |                               |                               |
 |                      close(), |                               v
 |                      detach() | commit()            +-------------------+
 |                               v                     |      Removed      |
 |                     +-------------------+           +-------------------+
 |                     |     Detached      |                     |
 |                     +-------------------+                     | commit()
 |                               |                               v
 +-------------------------------+                     [ Usunięcie z bazy ]
              merge()
```

### Charakterystyka 4 stanów:
1. **New (Transient – nowy / ulotny):**
   - Obiekt został właśnie utworzony w Javie operatorem `new`.
   - Nie jest powiązany z kontekstem trwałości (`EntityManager`), nie ma jeszcze przypisanego identyfikatora bazodanowego (Persistence Identity) i nie istnieje w bazie danych.
2. **Managed (Zarządzany):**
   - Obiekt posiada tożsamość bazodanową (klucz główny) i jest zarejestrowany w aktywnym kontekście trwałości.
   - Każda zmiana stanu pól obiektu w Javie jest śledzona przez kontener i zostanie automatycznie zapisana do bazy danych w chwili commita transakcji.
   - Obiekt wchodzi w ten stan po wywołaniu `em.persist()`, po pobraniu z bazy przez `em.find()` lub zapytanie JPQL, albo po scaleniu przez `em.merge()`.
3. **Detached (Odłączony):**
   - Obiekt posiada tożsamość bazodanową (reprezentuje istniejący wiersz w bazie danych), ale **nie jest już powiązany z żadnym aktywnym kontekstem trwałości** (np. po zamknięciu `EntityManager`, wyczyszczeniu kontekstu `em.clear()` lub po zakończeniu transakcji).
   - Zmiany wprowadzane na obiekcie w stanie detached **nie są śledzone** i nie trafią do bazy danych. Aby przywrócić obiekt do stanu managed, należy wywołać operację `em.merge()`.
4. **Removed (Usunięty):**
   - Obiekt posiada tożsamość bazodanową i jest powiązany z kontekstem, ale został oznaczony do skasowania z bazy danych za pomocą metody `em.remove()`.
   - Fizyczne usunięcie rekordu z bazy następuje w chwili zatwierdzenia transakcji (*commit*) lub wywołania `em.flush()`.

### Podsumowanie w tabeli:
| Stan encji | Powiązana z Persistence Context? | Posiada tożsamość (Persistent Identity)? |
| :--- | :---: | :---: |
| **New** | NIE | NIE |
| **Managed** | **TAK** | **TAK** |
| **Detached** | NIE | **TAK** |
| **Removed** | **TAK** | **TAK** (do momentu usunięcia) |

---

## 8. Kluczowe operacje obiektu `EntityManager`
- `persist(entity)`: włącza nowy obiekt do kontekstu (przejście: New -> Managed).
- `find(Class, id)`: wyszukuje encję w kontekście lub bazie danych na podstawie jej klucza głównego (zwraca obiekt w stanie Managed).
- `remove(entity)`: oznacza zarządzany obiekt do usunięcia (przejście: Managed -> Removed).
- `merge(entity)`: kopiuje stan z odłączonego obiektu do obiektu zarządzanego i zwraca referencję do instancji Managed (przejście: Detached -> Managed).
- `refresh(entity)`: nadpisuje stan obiektu zarządzanego aktualnymi danymi pobranymi bezpośrednio z bazy danych.
- `flush()`: wymusza natychmiastowe zsynchronizowanie zmian oczekujących w kontekście trwałości z bazą danych (wygenerowanie SQL INSERT/UPDATE/DELETE przed commitem).
- `clear()`: odłącza wszystkie obiekty z bieżącego kontekstu (wszystkie encje przechodzą w stan Detached).

---

## 9. Język JPQL (Java Persistence Query Language)
- **Definicja:** Obiektowo zorientowany język zapytań o składni wzorowanej na języku SQL.
- **Fundamentalna różnica między JPQL a SQL:**
  - **SQL** operuje na fizycznych strukturach relacyjnych: tabelach, wierszach i kolumnach bazy danych.
  - **JPQL** operuje na abstrakcyjnym modelu obiektowym aplikacji: **na klasach encyjnych, ich właściwościach i obiektowych powiązaniach**.
- **Cechy JPQL:**
  - Zapytania JPQL są niezależne od używanego silnika bazy danych – dostawca JPA w locie tłumaczy zapytanie JPQL na natywny dialekt SQL danej bazy.
  - Automatyczne trawersowanie asocjacji (automatyczne złączenia tabel `JOIN` na podstawie relacji zdefiniowanych w encjach).
  - Wsparcie dla stronicowania (ang. *pagination*) za pomocą metod `setFirstResult(offset)` oraz `setMaxResults(limit)`.
- **Zapytania nazwane (`@NamedQuery`):**
  - Statyczne zapytania zdefiniowane deklaratywnie w metadanych samej klasy encyjnej.
  - Są parsowane i kompilowane do postaci zapytań bazodanowych **podczas startu aplikacji (inicjalizacji jednostki trwałości)**, co pozwala na wczesne wykrycie błędów składniowych oraz zapewnia wyższą wydajność wykonania.

---

## 10. Baza danych: Standalone DBMS a Baza Osadzona (Embedded)
- **Standalone DBMS (np. PostgreSQL, Oracle, MySQL):**
  - Niezależny serwer bazy danych działający jako oddzielna usługa/proces systemowy.
  - Niezbędny w środowiskach produkcyjnych ze względu na bezpieczeństwo, klastrowanie, trwałość i niezależną skalowalność.
- **Embedded Database (Baza osadzona, np. H2, HSQLDB, Java DB/Derby):**
  - Silnik bazy danych działa wewnątrz tej samej maszyny wirtualnej (JVM) co sama aplikacja.
  - Może przechowywać dane wyłącznie w pamięci operacyjnej RAM (tryb in-memory).
  - **Zalety:** natychmiastowy start, brak konieczności zewnętrznej instalacji i konfiguracji. Zawartość bazy jest automatycznie czyszczona przy zamknięciu aplikacji, co czyni ją idealnym narzędziem do szybkich testów integracyjnych i prototypowania.
