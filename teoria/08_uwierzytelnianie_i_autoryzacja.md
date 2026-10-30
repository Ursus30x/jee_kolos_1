# 08. Uwierzytelnianie i Autoryzacja – Teoria na Kolokwium

## 1. Uwierzytelnianie a Autoryzacja – Fundamentalne Rozróżnienie
*To absolutna podstawa pytań egzaminacyjnych z cyberbezpieczeństwa i systemów enterprise.*

| Pojęcie | Definicja | Główne pytanie | Sposób weryfikacji / Przykłady |
| :--- | :--- | :--- | :--- |
| **Uwierzytelnianie (Authentication – AuthN)** | Proces **potwierdzania tożsamości** podmiotu (użytkownika, procesu lub systemu) próbującego uzyskać dostęp do aplikacji. | **"Kim jesteś?"** | Sprawdzenie czegoś, co podmiot: <br>- **wie** (hasło, PIN), <br>- **posiada** (klucz sprzętowy, token SMS, karta inteligentna), <br>- **czym jest** (biometria: odcisk palca, skan twarzy). |
| **Autoryzacja (Authorization – AuthZ)** | Proces **weryfikacji uprawnień** podmiotu o ustalonej wcześniej tożsamości do wykonania konkretnej operacji lub uzyskania dostępu do chronionego zasobu. | **"Co wolno ci zrobić?"** | Sprawdzenie przypisanych ról użytkownika, list kontroli dostępu (ACL), polityk uprawnień czy reguł biznesowych. |

---

## 2. Model Kontroli Dostępu Oparty o Role (RBAC – Role-Based Access Control)
Standardowy model bezpieczeństwa w systemach Jakarta EE opiera się na koncepcji **RBAC**:
- **Zasada działania:** Uprawnienia do wykonywania operacji nie są przypisywane bezpośrednio do poszczególnych użytkowników. Zamiast tego:
  1. Uprawnienia do operacji i zasobów są przypisywane do **ról** (np. `ADMIN`, `MANAGER`, `USER`). Rola odzwierciedla funkcję pełnioną w organizacji.
  2. Użytkownikom przypisuje się jedną lub więcej ról.
- **Relacja wiele-do-wielu:** Jeden użytkownik może posiadać wiele ról jednocześnie; jedna rola może być nadana wielu użytkownikom.
- **Hierarchiczne modele korporacyjne:** W wielkich systemach stosuje się struktury wielopoziomowe: `Użytkownicy` -> `Kohorty / Działy` -> `Grupy uprawnień` -> `Role techniczne`.

---

## 3. Metody Uwierzytelniania Webowego i Utrzymywanie Sesji
### Podstawowe mechanizmy uwierzytelniania w protokole HTTP:
1. **HTTP Basic Authentication:**
   - Przeglądarka przesyła login i hasło połączone dwukropkiem i zakodowane w formacie Base64 w nagłówku każdego żądania: `Authorization: Basic [base64(login:haslo)]`.
   - **Wada:** Base64 to zaciemnienie, a nie szyfrowanie – bezwzględnie wymaga transmisji przez bezpieczny kanał TLS/HTTPS. Brak możliwości wygodnego wylogowania przez aplikację.
2. **Form-Based Authentication (Logowanie formularzowe):**
   - Aplikacja wyświetla formularz logowania HTML. Po pomyślnym zalogowaniu serwer tworzy sesję użytkownika i odsyła identyfikator sesji.
3. **Uwierzytelnianie oparte o tokeny (Token-based):**
   - Po podaniu poświadczeń serwer wydaje cyfrowo podpisany token (np. JWT). Klient dołącza token do kolejnych żądań w nagłówku `Authorization: Bearer <token>`. Rozwiązanie bezstanowe (*stateless*).
4. **Federacja tożsamości (OpenID Connect / SAML):**
   - Oddelegowanie procesu logowania do zewnętrznego dostawcy tożsamości (IdP, np. Google, Microsoft, Keycloak).

### Utrzymywanie sesji w kolejnych żądaniach HTTP:
- Ponieważ protokół HTTP jest bezstanowy, powiązanie kolejnych żądań z uwierzytelnioną sesją wymaga mechanizmu identyfikacji:
  - **Pliki cookie (Ciasteczka – standard współczesny):** Serwer odsyła nagłówek `Set-Cookie: JSESSIONID=xyz`. Przeglądarka automatycznie odsyła to ciasteczko przy każdym kolejnym żądaniu.
  - **Przepisywanie adresów URL (*URL Rewriting* – technologia rezerwowa):** Doklejanie identyfikatora sesji do każdego adresu URL (np. `index.xhtml;jsessionid=xyz`), gdy klient ma wyłączoną obsługę ciasteczek.
  - **Token w nagłówku:** Przesyłanie tokena w nagłówku HTTP.

### Specyfika logowania formularzowego w Jakarta EE (Przepływ zdarzeń):
- W standardzie kontenera webowego użytkownik **nigdy nie powinien wchodzić bezpośrednio na stronę logowania**:
  1. Użytkownik próbuje otworzyć chroniony zasób (np. `/admin/dashboard.xhtml`).
  2. Kontener wykrywa brak aktywnej sesji i **przechwytuje żądanie**, po czym automatycznie wyświetla formularz logowania (standardowy cel akcji `j_security_check`).
  3. Po poprawnym sprawdzeniu hasła i ról kontener samoczynnie przekierowuje użytkownika pod pierwotnie żądany adres URL.

---

## 4. Protokół OAuth2 (Open Authorization 2.0)
- **Definicja:** Ustandaryzowany protokół **autoryzacji delegowanej**.
- **Główny cel:** Umożliwienie aplikacji trzeciej (Client) uzyskania ograniczonego dostępu do chronionych zasobów użytkownika (Resource Server) w jego imieniu, **bez konieczności ujawniania hasła użytkownika tej aplikacji**.

### Cztery role w protokole OAuth2:
1. **Właściciel Zasobu (Resource Owner):** Użytkownik końcowy posiadający konto i dane.
2. **Klient (Client):** Aplikacja (webowa, mobilna, desktopowa), która żąda dostępu do zasobów w imieniu użytkownika.
3. **Serwer Zasobów (Resource Server):** Serwer udostępniający chronione API/dane (weryfikuje token dostępu przed zwróceniem danych).
4. **Serwer Autoryzacji (Authorization Server):** Serwer odpowiedzialny za uwierzytelnienie użytkownika, odebranie zgody (*consent*) i wystawienie tokenów dostępu (*Access Token* oraz *Refresh Token*). Przykłady: Keycloak, Auth0, Okta.

### Przegląd przepływów OAuth2 (Grant Types):
- **Authorization Code Flow (Przepływ z kodem autoryzacyjnym):**
  - Dedykowany dla aplikacji z bezpiecznym backendem (serwerowych).
  - Użytkownik loguje się na serwerze autoryzacji w przeglądarce, serwer zwraca do backendu klienta jednorazowy kod autoryzacji (*authorization code*). Backend wymienia ten kod bezpośrednio z serwerem autoryzacji na token dostępu, uwierzytelniając się własnym sekretem (*client_secret*).
- **Authorization Code Flow z rozszerzeniem PKCE (Proof Key for Code Exchange):**
  - Obowiązkowy standard dla **klientów publicznych**, którzy nie są w stanie bezpiecznie ukryć sekretu klienta (aplikacje jednostronicowe SPA w JavaScript, aplikacje mobilne na smartfony).
  - Dynamicznie generuje parę kryptograficzną (`code_verifier` i `code_challenge`), co uniemożliwia przechwycenie i nieuprawnione użycie kodu autoryzacyjnego.
- **Client Credentials Flow (Przepływ poświadczeń klienta):**
  - Używany w komunikacji **maszyna-maszyna / serwer-serwer (M2M)** bez udziału bezpośredniego użytkownika końcowego (np. usługa synchronizacji w tle). Klient uwierzytelnia się wyłącznie własnymi poświadczeniami (`client_id` i `client_secret`).
- **Resource Owner Password Credentials Flow (z poświadczeniami użytkownika):**
  - Użytkownik podaje login i hasło bezpośrednio do aplikacji klienta. **Przestarzały i wysoce niezalecany**, łamie ideę ochrony poświadczeń.
- **Implicit Flow (Przepływ uproszczony):**
  - Dawniej stosowany w przeglądarkach (token zwracany wprost w hashu adresu URL). **Obecnie wycofywany i zakazany** ze względów bezpieczeństwa na rzecz kodu z PKCE.
- **Device Authorization Flow:**
  - Zaprojektowany dla urządzeń o ograniczonym interfejsie wejściowym (Smart TV, konsole gier) – urządzenie wyświetla krótki kod i adres www, a użytkownik loguje się na smartfonie lub komputerze.

---

## 5. Tokeny JWT (JSON Web Token)
- **Definicja:** Otwarty standard (RFC 7519) reprezentacji poświadczeń w postaci kompaktowego, bezpiecznego obiektu JSON przesyłanego jako pojedynczy ciąg znaków.
- **Cecha kluczowa: Samowystarczalność (*Self-contained*):** Token zawiera w sobie wszystkie niezbędne informacje o tożsamości użytkownika i jego uprawnieniach. Serwer zasobów może zweryfikować ważność i uprawnienia tokena lokalnie (poprzez weryfikację podpisu kryptograficznego), **bez konieczności odpytywania bazy danych czy serwera autoryzacji przy każdym zapytaniu**.

### Struktura tokena JWT:
Składa się z trzech części oddzielonych kropkami: `Nagłówek.Ładunek.Podpis` (`Header.Payload.Signature`), zakodowanych w standardzie **Base64Url**:
1. **Nagłówek (Header):** Określa typ tokena (`JWT`) oraz użyty algorytm kryptograficzny podpisu (np. `HS256`, `RS256`).
2. **Ładunek (Payload / Claims – Roszczenia):** Zawiera właściwe dane o użytkowniku, uprawnieniach oraz metadane tokena.
   - **Standardowe roszczenia zarejestrowane (Registered Claims):**
     - `iss` (*Issuer*): identyfikator podmiotu, który wystawił token.
     - `sub` (*Subject*): podmiot tokena (identyfikator/login użytkownika).
     - `aud` (*Audience*): zamierzony odbiorca tokena (dla jakiego serwisu/API jest przeznaczony).
     - `exp` (*Expiration Time*): data i godzina wygaśnięcia tokena (kluczowe dla bezpieczeństwa).
     - `nbf` (*Not Before*): czas, przed upływem którego token nie może być honorowany.
     - `iat` (*Issued At*): data i czas wystawienia tokena.
     - `jti` (*JWT ID*): unikalny identyfikator tokena (służy do zapobiegania atakom powtórzeniowym).
3. **Podpis cyfrowy (Signature):** Gwarantuje integralność i autentyczność tokena. Powstaje przez zaszyfrowanie skrótu nagłówka i ładunku kluczem tajnym lub prywatnym.
   - **Podpis symetryczny (np. HS256 – HMAC SHA-256):** Serwer wystawiający i serwer weryfikujący korzystają z tego samego, wspólnego sekretu.
   - **Podpis asymetryczny (np. RS256 – RSA, ES256 – ECDSA):** Serwer autoryzacji podpisuje token swoim **kluczem prywatnym**, natomiast serwery zasobów weryfikują podpis za pomocą ogólnodostępnego **klucza publicznego**.

---

## 6. Bezpieczeństwo w architekturze Java EE / Jakarta EE
W platformie Jakarta EE bezpieczeństwo realizowane jest na dwóch poziomach:

### 1. Bezpieczeństwo Deklaratywne (Declarative Security):
- Reguły bezpieczeństwa, powiązania ról i restrykcje dostępu do adresów URL oraz metod biznesowych są **zdefiniowane poza kodem aplikacji**:
  - Za pomocą wpisów w deskryptorze `web.xml` (sekcje `<security-constraint>`, `<web-resource-collection>`, `<auth-constraint>`).
  - Za pomocą ustandaryzowanych adnotacji na poziomie klas i metod:
    - `@ServletSecurity`, `@HttpConstraint`, `@HttpMethodConstraint` – ochrona servletów.
    - `@RolesAllowed("ADMIN")` – restrykcja wykonania metody EJB lub endpointu JAX-RS tylko dla użytkowników z określoną rolą.
    - `@PermitAll` – metoda publicznie dostępna dla każdego.
    - `@DenyAll` – całkowity zakaz wywoływania metody.
    - `@RunAs("ROLA")` – wymuszenie wykonania danej operacji z uprawnieniami określonej roli systemowej, niezależnie od roli zalogowanego użytkownika.
- **Zaleta:** Logika biznesowa jest całkowicie wolna od kodu sprawdzającego uprawnienia; zmiana reguł dostępu nie wymaga modyfikacji kodu wewnętrznego komponentów.

### 2. Bezpieczeństwo Programistyczne (Programmatic Security):
- Stosowane w sytuacjach, gdy reguły dostępu zależą od dynamicznych warunków biznesowych (np. "użytkownik może edytować zamówienie tylko wtedy, gdy sam jest jego autorem lub gdy jest administratorem").
- Realizowane poprzez bezpośrednie odpytanie kontekstu bezpieczeństwa z poziomu kodu Javy:
  - We współczesnym Jakarta Security: wstrzykiwany obiekt `SecurityContext`:
    - `securityContext.isCallerInRole("NAZWA_ROLI")` – sprawdza, czy aktualny wywołujący posiada wskazaną rolę.
    - `securityContext.getCallerPrincipal()` – zwraca obiekt reprezentujący tożsamość zalogowanego użytkownika (np. pobranie nazwy przez `getName()`).
  - W starszych rozwiązaniach dedykowane obiekty poszczególnych warstw: `HttpServletRequest.isUserInRole(...)`, `EJBContext.isCallerInRole(...)`.

---

## 7. Specyfikacja Jakarta Security
Zapewnia nowoczesny, ujednolicony standard bezpieczeństwa niezależny od specyfiki konkretnego serwera aplikacji.

### Główne elementy specyfikacji:
1. **Dostawcy tożsamości (Identity Stores):** Komponenty odpowiedzialne za weryfikację poświadczeń i pobranie ról użytkownika z zewnętrznych magazynów:
   - `@InMemoryIdentityStoreDefinition` – tożsamości zapisane w pamięci serwera (do testów).
   - `@DatabaseIdentityStoreDefinition` – integracja z tabelami użytkowników i ról w relacyjnej bazie danych (poprzez zapytania `callerQuery` i `groupsQuery`).
   - `@LdapIdentityStoreDefinition` – integracja z korporacyjnym katalogiem LDAP / Active Directory.
2. **Bezpieczne przechowywanie haseł (`Pbkdf2PasswordHash`):**
   - Hasła użytkowników w bazie danych nigdy nie mogą być przechowywane tekstem jawnym (*plaintext*).
   - Specyfikacja dostarcza mechanizm bezpiecznego haszowania haseł z użyciem soli kryptograficznej (*salt*) oraz tysięcy iteracji funkcji skrótu (standard PBKDF2), co skutecznie zabezpiecza przed atakami słownikowymi i tęczowymi tablicami (*rainbow tables*).
