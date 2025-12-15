# Automatyzacja Faktur - n8n

## Opis

Ten workflow automatyzuje proces obsługi faktur w Twojej organizacji:
- Monitoruje skrzynkę email pod kątem nowych faktur PDF
- Wykorzystuje AI (Claude + OpenAI) do ekstrakcji danych z faktur
- Sprawdza duplikaty w bazie Airtable
- Zapisuje faktury na Google Drive (z automatycznym tworzeniem folderów miesięcznych)
- Archiwizuje dane w Airtable
- Wysyła powiadomienia na Slack

## Wymagania

### 1. Konta i API Keys

Musisz posiadać następujące konta i klucze API:

- **Email IMAP** - dostęp do skrzynki `faktury@dotykowewlaczniki.pl`
- **Anthropic Claude API** - klucz API do Claude 3.7 Sonnet
- **OpenAI API** - klucz API do GPT-4 (backup dla naprawy JSON)
- **Airtable** - API key oraz IDs:
  - Base ID: `appbLQjOUm58ckRwu`
  - Table ID: `tbl2jnB2TinJgSh6b`
- **Google Drive** - OAuth2 credentials
- **Slack** - API token lub OAuth2

### 2. n8n

Zainstalowany i uruchomiony n8n (self-hosted lub cloud)

## Instalacja

### Krok 1: Import workflow do n8n

1. Zaloguj się do swojej instancji n8n
2. Kliknij "Import from File" lub "Import from URL"
3. Wybierz plik `workflow-n8n.json`
4. Workflow zostanie zaimportowany

### Krok 2: Konfiguracja credentials

Musisz skonfigurować następujące credentials w n8n:

#### 2.1. Email IMAP
```
Typ: IMAP
Nazwa: faktury@dotykowewlaczniki.pl
Host: [twój serwer IMAP]
Port: 993
User: faktury@dotykowewlaczniki.pl
Password: [hasło]
SSL: Tak
```

#### 2.2. Anthropic Claude API
```
Typ: HTTP Header Auth
Nazwa: Anthropic API Key
Header: x-api-key
Value: [twój klucz API Anthropic]
```

#### 2.3. OpenAI API
```
Typ: HTTP Header Auth
Nazwa: OpenAI API Key
Header: Authorization
Value: Bearer [twój klucz API OpenAI]
```

#### 2.4. Airtable
```
Typ: Airtable API
Nazwa: Airtable API
API Key: [twój klucz API Airtable]
```

#### 2.5. Google Drive
```
Typ: Google Drive OAuth2
Nazwa: Google Drive
[Postępuj zgodnie z instrukcjami n8n do konfiguracji OAuth2]
```

#### 2.6. Slack
```
Typ: Slack API lub Slack OAuth2
Nazwa: Slack
Token: [twój token Slack]
Kanał: faktury
```

### Krok 3: Dostosowanie workflow

#### Zmień nazwę kanału Slack (jeśli potrzeba)
W node'ach:
- "Slack - Nowa faktura"
- "Slack - Duplikat"

Zmień `channelId.value` na nazwę twojego kanału.

#### Dostosuj strukturę Airtable (jeśli potrzeba)
W node "Airtable - Utwórz rekord" sprawdź czy nazwy kolumn pasują do twojej bazy:
- Numer FV
- Dostawca
- Data wystawienia
- Data sprzedaży
- NIP
- Netto
- Brutto
- VAT
- Link do pliku

### Krok 4: Testowanie

1. **Test emaila**: Wyślij testową fakturę PDF na adres email
2. **Monitoruj wykonanie**: W n8n kliknij "Executions" aby zobaczyć przebieg workflow
3. **Sprawdź rezultaty**:
   - Czy dane zostały wyekstrahowane poprawnie?
   - Czy plik pojawił się na Google Drive?
   - Czy rekord został utworzony w Airtable?
   - Czy otrzymałeś powiadomienie na Slack?

## Struktura Workflow

### Główne kroki:

1. **Email Trigger** - monitoruje INBOX co minutę
2. **Delay 5s** - opóźnienie dla stabilności
3. **Filtr PDF** - przepuszcza tylko emaile z załącznikami PDF
4. **Claude AI** - ekstrakcja danych z faktury (z retry 3x)
5. **Parse JSON** - przetworzenie odpowiedzi Claude
   - **Error Handler**: OpenAI GPT naprawia błędny JSON
6. **Check Invoice** - sprawdza czy to faktuar czy spam
7. **Airtable Search** - szuka duplikatów
8. **Router** - podział na 2 ścieżki:

#### Ścieżka 1: Nowa faktura
9. **Google Drive Check** - sprawdza czy folder miesięczny istnieje
   - **Error Handler**: Tworzy folder jeśli nie istnieje
10. **Google Drive Upload** - zapisuje plik PDF
11. **Google Drive Share** - udostępnia plik
12. **Airtable Create** - tworzy rekord z danymi
13. **Slack Notification** - powiadamia o nowej fakturze

#### Ścieżka 2: Duplikat
14. **Slack Warning** - powiadamia o duplikacie

## Format danych wyekstrahowanych przez AI

```json
{
  "invoice_number": "FV/2024/01/123",
  "supplier": "Nazwa Dostawcy Sp. z o.o.",
  "issue_date": "2024-01-15",
  "sale_date": "2024-01-15",
  "seller's ID": "1234567890",
  "net_amount": 1000.00,
  "gross_amount": 1230.00,
  "vat_amount": 230.00
}
```

## Obsługa błędów

Workflow zawiera następujące mechanizmy obsługi błędów:

1. **Claude AI retry**: 3 próby z 70s odstępem
2. **JSON Parse error**: Automatyczna naprawa przez OpenAI GPT-4
3. **Folder creation**: Automatyczne tworzenie folderu jeśli nie istnieje
4. **Duplicate detection**: Powiadomienie zamiast błędu

## Troubleshooting

### Problem: Claude nie może sparsować faktury

**Rozwiązanie**: Sprawdź czy:
- Plik PDF nie jest chroniony hasłem
- Plik PDF zawiera tekst (nie jest skanem bez OCR)
- Format faktury jest czytelny

### Problem: Duplikaty nie są wykrywane

**Rozwiązanie**: Sprawdź:
- Czy formula w Airtable Search jest poprawna
- Czy nazwy pól w Airtable są zgodne z workflow
- Czy numer faktury został poprawnie wyekstrahowany

### Problem: Powiadomienia Slack nie działają

**Rozwiązanie**: Sprawdź:
- Czy token Slack jest ważny
- Czy bot ma uprawnienia do pisania na kanale
- Czy nazwa kanału jest poprawna

## Optymalizacja

### Redukcja kosztów AI:
- Claude API: ~$0.01-0.03 za fakturę
- OpenAI (tylko w razie błędu): ~$0.02 za naprawę

### Zwiększenie wydajności:
- Zmień polling z 1 minuty na dłuższy interwał jeśli nie potrzebujesz real-time
- Rozważ użycie webhooków zamiast pollingu (jeśli dostępne)

## Migracja z Make.com

Jeśli migrowałeś z Make.com, pamiętaj że:

1. **Credentials**: Musisz ponownie skonfigurować wszystkie połączenia
2. **Variables**: n8n używa `{{ }}` zamiast Make.com `{{}}`
3. **Error Handling**: n8n ma inny mechanizm obsługi błędów
4. **Scheduling**: n8n używa polling zamiast instant triggers Make.com

## Wsparcie

W razie problemów:
1. Sprawdź logi wykonań w n8n
2. Sprawdź czy wszystkie credentials są poprawnie skonfigurowane
3. Przetestuj każdy node osobno używając "Execute Node"

## Licencja

Ten workflow jest dostępny na licencji MIT.
