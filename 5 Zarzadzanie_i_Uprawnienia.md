# Zarządzanie dostępem i uprawnieniami w DataBocie (RLS, RBAC, audyt)

## 🎯 Cel sekcji
W tej części warsztatu skupimy się na tym, jak kontrolować dostęp do danych i wyników generowanych przez DataBota. Nauczysz się:
- tworzyć role użytkowników w Databricks,
- ograniczać dane widoczne dla różnych grup użytkowników (RLS),
- analizować logi i zdarzenia (audyt),
- stosować dobre praktyki bezpieczeństwa modeli AI,
- konfigurować integracje z Azure AD i kontrolą dostępu opartą o tokeny.

## 🧾 Kontekst
DataBot działa jako „przedłużenie” dostępu do danych w organizacji. Musimy zadbać, aby jego odpowiedzi:
- były zgodne z uprawnieniami pytającego,
- nie ujawniały danych poufnych przypadkowym użytkownikom,
- dawały się śledzić (co, kto, kiedy pytał i otrzymał),
- respektowały ograniczenia regulacyjne i wewnętrzne polityki danych (np. RODO, ISO 27001).

## 🧩 Rola, użytkownik, kontekst
Użytkownicy mogą mieć różne role – np. *Użytkownik*, *Analityk*, *Administrator*. Odpowiedzi DataBota mogą być filtrowane na podstawie:
- tożsamości użytkownika (np. adres email, Azure AD Object ID),
- roli przypisanej w systemie RBAC,
- przynależności do zespołu lub działu (np. na podstawie claimów z tokena JWT).

### Przykład scenariusza:
```
Użytkownik = Magda (rola: Analityk HR)
Pytanie: „Ilu pracowników opuściło firmę w Q1?”
→ Odpowiedź zawiera tylko dane HR, a nie np. finansowe lub strategiczne.
```

## 🔐 RBAC – Role-Based Access Control w Databricks
RBAC służy do nadawania uprawnień do zasobów w oparciu o role przypisane użytkownikom lub grupom. W Databricks oraz Azure, uprawnienia ustala się na poziomie:
- Workspace (czy można edytować/uruchamiać notebooki),
- Klasterów (np. dostęp do GPU),
- Zasobów danych: Delta table, Unity Catalog, DBFS,
- Sekretów (`Secrets`),
- Modeli ML (MLflow Registry lub Azure AI Foundry).

### Przykład – nadanie uprawnienia SELECT:
```sql
GRANT SELECT ON TABLE faktura_dane TO `rola_analityk_hr`;
```

### Konfiguracja ról w Unity Catalog:
1. Przejdź do **Data > Catalog > Permissions**.
2. Wybierz bazę danych, tabelę lub katalog.
3. Kliknij **Permissions** → dodaj rolę lub użytkownika.

## 🔎 Row-Level Security (RLS)
RLS pozwala ograniczyć dostęp do poszczególnych wierszy w tabeli w zależności od tożsamości użytkownika. Działa w oparciu o **Dynamic Views** w Unity Catalog.

### Przykład – widok z dynamicznym filtrem:
```sql
CREATE OR REPLACE VIEW rls_faktury AS
SELECT * FROM faktury
WHERE dział = current_user_dział();
```
Użytkownik musi mieć przypisane `current_user_dział()` jako funkcję zwracającą dozwoloną wartość z profilu/logiki tokena (np. „HR”, „Finanse”).

### Alternatywa w kodzie DataBota:
W funkcji generującej kontekst do odpowiedzi można uwzględnić dodatkowy filtr:
```python
filtered_df = df.filter(df.dzial == user_claims["dzial"])
```

## 📦 Sekrety i autoryzacja (Databricks Secrets + Azure Entra ID)
Wszystkie dane wrażliwe (np. klucze API, tokeny, hasła) powinny być przechowywane w Secret Scopes.

### Utworzenie Secret Scope:
```bash
databricks secrets create-scope --scope databoty
```
Dodanie sekretu:
```bash
databricks secrets put --scope databoty --key openai_token
```
Odczyt w kodzie:
```python
token = dbutils.secrets.get(scope="databoty", key="openai_token")
```

## 🔐 Autoryzacja przez token JWT (np. w FastAPI frontend)
Jeśli DataBot jest zabezpieczony przez Azure AD:
- Użytkownik loguje się, otrzymuje token JWT,
- Token jest przekazywany do backendu,
- Backend dekoduje token i pozyskuje `sub`, `email`, `roles`.

W FastAPI:
```python
from fastapi.security import OAuth2PasswordBearer
from jose import jwt
user = jwt.decode(token, key, algorithms=["RS256"])
```

## 📊 Audyt i logi
Każde zapytanie do DataBota powinno być rejestrowane:
- użytkownik (email, ID),
- treść pytania,
- czas zapytania,
- wynik i źródła danych (konkretne pliki/tabele).

### Logowanie do MLflow:
```python
import mlflow
mlflow.set_experiment("DataBot_Logi")
with mlflow.start_run():
    mlflow.log_param("user", user_email)
    mlflow.log_param("question", query)
    mlflow.log_text(generated_answer, "answer.txt")
```

Alternatywnie:
- Wysyłaj logi do Azure Log Analytics,
- Przechowuj logi w Delta Table jako historia konwersacji.

## ✅ Ćwiczenie praktyczne
1. W Databricks:
   - Stwórz `Secret Scope` i dodaj przykładowy token dostępu.
   - Utwórz katalog i tabelę `raporty_finansowe`.
   - Utwórz `VIEW rls_raporty` ograniczający dostęp do działów.
   - Nadaj `GRANT SELECT` tylko wybranym rolom.
2. Zmodyfikuj kod DataBota tak, by uwzględniał ograniczenia w `WHERE` lub dynamicznie filtrował kontekst.
3. Przetestuj:
   - Osoba z działu HR i osoba z działu sprzedaży zadają to samo pytanie,
   - Sprawdź, że odpowiedzi różnią się zgodnie z widocznością danych.

## 🧠 Dobre praktyki bezpieczeństwa
- Wprowadź klasyfikację danych (np. Publiczne, Poufne, Krytyczne).
- Oddziel tożsamości i role dla użytkowników, aplikacji i usług.
- Stosuj rotację sekretów i ogranicz czas ich ważności.
- Ogranicz liczbę użytkowników mogących edytować modele lub pipeline’y.
- Stosuj tagowanie źródeł danych w Unity Catalog dla celów audytu i zarządzania.

---

📌 W kolejnych krokach możesz połączyć logowanie z Azure Monitor lub Log Analytics, by mieć pełną obserwowalność działania bota i ścieżek danych.
