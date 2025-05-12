
# ✅ RAG Workshop – Lista kontrolna etapów wdrożenia

Podsumowanie warsztatu - elementy kluczowe w RAG

---

## 🧾 Etap 1: Przygotowanie danych
- [ ] Zidentyfikowano źródła danych (PDF, OneDrive, SharePoint, SQL, blob storage)
- [ ] Dane zostały pobrane lub podłączone do środowiska (UI lub kodem)
- [ ] Zastosowano czyszczenie danych (np. stopki, nagłówki)
- [ ] Usunięto duplikaty i przestarzałe wersje (np. MinHash, heurystyki)

👉 Notatnik: `1. DataPipeline.ipynb`

---

## ✂️ Etap 2: Chunkowanie i wzbogacenie
- [ ] Dane zostały podzielone na fragmenty logiczne (np. akapity, tematy, nagłówki)
- [ ] Zastosowano odpowiednią strategię chunkowania (np. Recursive, MarkdownHeader)
- [ ] Dodano metadane: tytuł, typ, dział, słowa kluczowe
- [ ] Opcjonalnie: zastosowano wzbogacenie LLM (np. streszczenie, tagowanie)

👉 Notatnik: `2. Chunkowanie.ipynb`

---

## 🧠 Etap 3: Embedding
- [ ] Wybrano model embeddingów (np. ada-002, GTE, bge)
- [ ] Zweryfikowano limit tokenów i rozmiar chunków
- [ ] Wygenerowano embeddingi i zapisano je z metadanymi
- [ ] Embeddingi zostały zindeksowane (np. do FAISS, Weaviate, Fabric AI Index)

👉 Notatnik: `2. Chunkowanie.ipynb`, `3. Core Databota.ipynb`

---

## 🔍 Etap 4: Retrieval
- [ ] Zaimplementowano wyszukiwanie wektorowe lub hybrydowe
- [ ] Przetestowano różne top_k, filtry, dopasowania
- [ ] Upewniono się, że kontekst trafia do modelu LLM poprawnie (prompt chaining)

👉 Notatnik: `3. Core Databota.ipynb`

---

## 💬 Etap 5: Generacja odpowiedzi
- [ ] Model LLM otrzymuje pytanie i kontekst
- [ ] Generuje spójną i opartą na kontekście odpowiedź
- [ ] Odpowiedź jest logowana, zwracana użytkownikowi i analizowana

👉 Notatnik: `3. Core Databota.ipynb`

---

## 🧪 Etap 6: Ewaluacja i eksperymenty
- [ ] Przeprowadzono porównanie: model bazowy vs model fine-tuned
- [ ] Zastosowano metryki: groundedness, relevancy, completeness
- [ ] Użyto MLflow do logowania runów
- [ ] Zinterpretowano wyniki i podjęto decyzję o utrwaleniu modelu

👉 Notatnik: `4. Finetuning modelu.ipynb`

---

## 🔐 Etap 7: Uprawnienia i bezpieczeństwo
- [ ] Wdrożono RBAC i RLS na poziomie danych (Unity Catalog)
- [ ] Dane dostępne są zgodnie z rolą pytającego
- [ ] Logi przechowywane w MLflow, Log Analytics lub Delta Table
- [ ] Sekrety trzymane w Databricks Secrets

👉 Notatnik: `06_Zarzadzanie_i_Uprawnienia.md`

---

## 🌐 Etap 8: Frontend i integracja
- [ ] DataBot udostępniony jako API / aplikacja webowa
- [ ] Autoryzacja tokenem JWT (Azure AD / MSAL)
- [ ] Użytkownik może zadawać pytania przez interfejs
- [ ] Błądzenie i brak dostępu są poprawnie obsługiwane

👉 Notatnik: `5. Frontend.ipynb`

---


