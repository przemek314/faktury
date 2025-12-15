# Porównanie Make.com vs n8n - Automatyzacja Faktur

## Przegląd konwersji

Ten dokument opisuje różnice między oryginalnym workflow Make.com a nową wersją n8n.

## Mapping modułów

| Make.com Module | n8n Node | Uwagi |
|----------------|----------|-------|
| `email:TriggerNewEmail` | `emailReadImap` | n8n używa pollingu zamiast instant triggera |
| `util:FunctionSleep` | `wait` | Podobna funkcjonalność |
| `anthropic-claude:createAMessage` | `httpRequest` + custom body | n8n nie ma dedykowanego node Anthropic, używamy HTTP Request |
| `json:ParseJSON` | `code` node | n8n używa JavaScript do parsowania |
| `airtable:ActionSearchRecords` | `airtable` (search) | Natywna integracja |
| `airtable:ActionCreateRecord` | `airtable` (append) | Natywna integracja |
| `airtable:ActionUpdateRecords` | `airtable` (update) | *Nie zaimplementowane w workflow n8n* |
| `builtin:BasicRouter` | `if` node | n8n używa IF node do routingu |
| `builtin:Resume` | *brak* | n8n używa connections zamiast Resume |
| `google-drive:recognizeAFileFolderPath` | `googleDrive` (search) | Natywna integracja |
| `google-drive:createAFolder` | `googleDrive` (create) | Natywna integracja |
| `google-drive:uploadAFile` | `googleDrive` (upload) | Natywna integracja |
| `google-drive:shareAFileFolder` | `googleDrive` (share) | Natywna integracja |
| `slack:CreateMessage` | `slack` | Natywna integracja |
| `openai-gpt-3:CreateCompletion` | `httpRequest` | Używamy HTTP Request do OpenAI API |

## Kluczowe różnice

### 1. Triggery

**Make.com:**
- Instant trigger (webhook-based)
- Natychmiast reaguje na nowe emaile

**n8n:**
- Polling trigger (sprawdza co X minut)
- Domyślnie: co 1 minutę
- Można zmienić w ustawieniach node

### 2. Error Handling

**Make.com:**
- `onerror` routes
- Automatyczny retry z delay
- Resume po naprawieniu błędu

**n8n:**
- Error connections
- Manual retry configuration
- Merge nodes do łączenia ścieżek success/error

### 3. Expressions

**Make.com:**
```
{{1.attachments[].fileName}}
{{71.invoice_number}}
```

**n8n:**
```
={{ $json.attachments[0].fileName }}
={{ $('Parse Claude Response').item.json.invoice_number }}
```

### 4. Anthropic Claude Integration

**Make.com:**
- Dedykowany moduł `anthropic-claude:createAMessage`
- Obsługa PDF przez `pdfFileData` i `pdfFileName`
- Automatyczna obsługa API versioning

**n8n:**
- HTTP Request node
- Manual PDF encoding do base64
- Manual headers (`x-api-key`, `anthropic-version`)
- Body JSON:
  ```json
  {
    "model": "claude-3-7-sonnet-20250219",
    "max_tokens": 8192,
    "messages": [{
      "role": "user",
      "content": [
        {
          "type": "document",
          "source": {
            "type": "base64",
            "media_type": "application/pdf",
            "data": "={{ $json.attachments[0].data.toString('base64') }}"
          }
        },
        {
          "type": "text",
          "text": "[prompt]"
        }
      ]
    }]
  }
  ```

### 5. Routing Logic

**Make.com:**
- `builtin:BasicRouter` z multiple routes
- Każda route ma własny `filter`

**n8n:**
- Multiple IF nodes
- Każdy IF ma True/False paths
- Można łączyć IF nodes do stworzenia złożonej logiki

### 6. Data Flow

**Make.com:**
- Bundle-based (każdy moduł przetwarza wszystkie bundles)
- Automatyczny iterator dla arrays

**n8n:**
- Item-based (każdy item jest przetwarzany oddzielnie)
- Trzeba użyć `SplitInBatches` lub loop dla arrays

## Funkcje zaimplementowane w n8n

✅ Email IMAP trigger
✅ PDF attachment detection
✅ Claude AI invoice scanning
✅ JSON parsing z error handling
✅ OpenAI backup dla błędnego JSON
✅ Airtable duplicate detection
✅ Google Drive folder management (auto-create)
✅ Google Drive file upload
✅ Google Drive file sharing
✅ Airtable record creation
✅ Slack notifications (new invoice + duplicate)
✅ Retry logic dla Claude API
✅ Filtrowanie nie-faktur (NOT_INVOICE)

## Funkcje NIE zaimplementowane w n8n

❌ **OpenAI dodatkowe przetwarzanie** (node ID 240 w Make.com)
  - W oryginalnym Make.com był node OpenAI po utworzeniu rekordu Airtable
  - Nie było jasne co to node robi (brak danych w eksporcie)
  - Można dodać jeśli potrzebne

❌ **Airtable Update Records** (node ID 243 w Make.com)
  - W oryginalnym Make.com był update po OpenAI
  - Nie było jasne co aktualizuje
  - Można dodać jeśli potrzebne

## Zalety n8n

1. **Open Source** - można hostować self-hosted
2. **Niższe koszty** - brak limitów operacji (przy self-hosted)
3. **Pełna kontrola** - dostęp do kodu i danych
4. **Extensible** - można pisać własne node'y
5. **Community** - aktywna społeczność

## Wady n8n

1. **Polling zamiast webhooks** - opóźnienie do 1 minuty
2. **Brak niektórych natywnych integracji** - trzeba używać HTTP Request
3. **Bardziej techniczne** - wymaga więcej konfiguracji
4. **Maintenance** - przy self-hosted trzeba samemu aktualizować

## Koszty

### Make.com
- Plan Pro: $29/mo (10,000 operations)
- Plan Teams: $99/mo (40,000 operations)
- Każda operacja to execution jednego modułu
- Ten workflow ~15-20 operacji na fakturę

### n8n
- Cloud: $20/mo (startowy plan)
- Self-hosted: Darmowy (koszty serwera: $5-20/mo)
- Unlimited executions przy self-hosted

## Wydajność

| Aspekt | Make.com | n8n |
|--------|----------|-----|
| Trigger delay | Instant | 1 min (polling) |
| Execution speed | Bardzo szybki | Szybki |
| Concurrent executions | Wysokie | Zależy od planu |
| Reliability | 99.9% uptime | Zależy od hostingu |

## Rekomendacje

### Zostań z Make.com jeśli:
- Potrzebujesz instant triggers
- Nie chcesz zarządzać infrastrukturą
- Wolisz prostotę nad kontrolą
- Masz budżet na SaaS

### Przejdź na n8n jeśli:
- Chcesz obniżyć koszty
- Potrzebujesz pełnej kontroli nad danymi
- Masz zasoby techniczne do self-hostingu
- 1 minuta opóźnienia jest akceptowalna
- Chcesz możliwość customizacji

## Migracja krok po kroku

1. ✅ Eksportuj workflow z Make.com
2. ✅ Przeanalizuj wszystkie moduły i ich funkcje
3. ✅ Stwórz odpowiedniki w n8n
4. ✅ Skonfiguruj credentials
5. ⚠️ **TESTUJ równolegle** przez 1-2 tygodnie
6. ⚠️ Monitoruj błędy i edge cases
7. ⚠️ Gdy pewny - wyłącz Make.com workflow
8. ⚠️ Anuluj subskrypcję Make.com

## Conclusion

Workflow został successfully skonwertowany z Make.com do n8n. Wszystkie kluczowe funkcje są zaimplementowane. Główna różnica to polling trigger zamiast instant, co daje opóźnienie do 1 minuty.

Dla większości use case'ów automatyzacji faktur, 1 minuta opóźnienia jest w pełni akceptowalna i oszczędność kosztów (szczególnie przy self-hosted n8n) może być znacząca.
