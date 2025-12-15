{
    "subflows": [
        {
            "flow": [
                {
                    "id": 1,
                    "module": "email:TriggerNewEmail",
                    "version": 7,
                    "parameters": {
                        "to": "",
                        "from": "",
                        "text": "",
                        "folder": "INBOX",
                        "account": 8852424,
                        "subject": "",
                        "criteria": "UNSEEN",
                        "markSeen": true,
                        "maxResults": 50
                    },
                    "mapper": {},
                    "metadata": {
                        "designer": {
                            "x": -506,
                            "y": 571
                        },
                        "restore": {
                            "parameters": {
                                "folder": {
                                    "path": [
                                        "SKRZYNKA ODBIORCZA"
                                    ]
                                },
                                "account": {
                                    "data": {
                                        "scoped": "true",
                                        "connection": "imap"
                                    },
                                    "label": "faktury@dotykowewlaczniki.pl (IMAP faktury@dotykowewlaczniki.pl)"
                                },
                                "criteria": {
                                    "label": "Only Unread emails"
                                }
                            }
                        },
                        "parameters": [
                            {
                                "name": "account",
                                "type": "account:imap,google-restricted,microsoft-smtp-imap",
                                "label": "Connection",
                                "required": true
                            },
                            {
                                "name": "criteria",
                                "type": "select",
                                "label": "Criteria",
                                "required": true,
                                "validate": {
                                    "enum": [
                                        "ALL",
                                        "SEEN",
                                        "UNSEEN"
                                    ]
                                }
                            },
                            {
                                "name": "from",
                                "type": "email",
                                "label": "Sender email address"
                            },
                            {
                                "name": "to",
                                "type": "email",
                                "label": "Recipient email address"
                            },
                            {
                                "name": "subject",
                                "type": "text",
                                "label": "Subject"
                            },
                            {
                                "name": "text",
                                "type": "text",
                                "label": "Phrase"
                            },
                            {
                                "name": "markSeen",
                                "type": "boolean",
                                "label": "Mark message(s) as read when fetched"
                            },
                            {
                                "name": "maxResults",
                                "type": "number",
                                "label": "Maximum number of results"
                            },
                            {
                                "type": "hidden"
                            },
                            {
                                "name": "folder",
                                "type": "folder",
                                "label": "Folder",
                                "required": true
                            }
                        ]
                    }
                },
                {
                    "id": 84,
                    "module": "util:FunctionSleep",
                    "version": 1,
                    "parameters": {},
                    "filter": {
                        "name": "PDF",
                        "conditions": [
                            [
                                {
                                    "a": "{{1.attachments[].contentType}}",
                                    "b": "application",
                                    "o": "text:contain:ci"
                                }
                            ]
                        ]
                    },
                    "mapper": {
                        "duration": "5"
                    },
                    "metadata": {
                        "designer": {
                            "x": -65,
                            "y": 573,
                            "name": "Opóźnienie"
                        },
                        "restore": {},
                        "expect": [
                            {
                                "name": "duration",
                                "type": "uinteger",
                                "label": "Delay",
                                "required": true,
                                "validate": {
                                    "max": 300,
                                    "min": 1
                                }
                            }
                        ]
                    }
                },
                {
                    "id": 70,
                    "module": "anthropic-claude:createAMessage",
                    "version": 1,
                    "parameters": {
                        "__IMTCONN__": 8853167
                    },
                    "mapper": {
                        "model": "claude-3-7-sonnet-20250219",
                        "messages": [
                            {
                                "role": "user",
                                "content": [
                                    {
                                        "type": "document",
                                        "pdfFileData": "{{1.attachments[].data}}",
                                        "pdfFileName": "{{1.attachments[].fileName}}"
                                    },
                                    {
                                        "text": "Instrukcja ekstrakcji danych z faktur\r\nNa podstawie dostarczonego tekstu faktury, znajdź i wyodrębnij następujące informacje. Zwróć TYLKO obiekt JSON, bez żadnych dodatkowych komentarzy, oznaczeń formatowania czy znaczników kodu.\r\n\r\nNajpierw sprawdź czy to jest faktura szukając wskaźników:\r\n\r\nA. Bezpośrednie wskazania faktury:\r\n- Słowa kluczowe: \"faktura\", \"FV\", \"VAT\", \"rachunek\", \"należność\", \"dokument księgowy\", \"invoice\", \"invoiced\", \"billing\"\r\n- Numery faktur (np. FV/2024/123, VAT/2024/01)\r\n- Frazy \"w załączeniu faktura\", \"przesyłam fakturę\", \"załączam FV\"\r\n\r\nB. Pośrednie wskazania faktury:\r\n- Kwoty z określeniem \"netto\" lub \"brutto\"\r\n- Kwoty poprzedzone znakiem waluty ($, €, £)\r\n- Terminy płatności (np. \"termin płatności\", \"płatne do\")\r\n- Numery kont bankowych\r\n- Dane do przelewu\r\n- \"Do zapłaty\"\r\n- \"Proszę o zapłatę\"\r\n- \"Proforma\"\r\n- \"Nota księgowa\"\r\n- \"Rozliczenie\"\r\n- Frazy potwierdzające wystawienie dokumentu księgowego:\r\n * \"invoiced amount\"\r\n * \"payment received\"\r\n * \"payment confirmation\"\r\n- Daty transakcji/płatności w kontekście kwot\r\n- Informacje o metodzie płatności (np. \"payment method\", \"card ending with\")\n\nJeśli to nie jest faktura (brak powyższych wskaźników), zwróć obiekt JSON z wartością \"NOT_INVOICE\" w polu invoice_number i null w pozostałych polach.\r\n\r\nJeśli to jest faktura, wyodrębnij następujące informacje:\r\n\r\nNumer faktury (invoice_number)\r\nSzukaj: \"FV\", \"Faktura VAT\", \"Invoice no.\", \"Invoice number\"\r\nFormaty: FV/2024/01/123, INV-2024-001, 2024/ABC/123\r\n\r\nDane dostawcy (supplier)\r\nSzukaj: \"Sprzedawca\", \"Wystawca\", \"Supplier\", \"Seller\"\r\nTylko nazwa firmy, bez adresu\r\n\r\nData wystawienia (issue_date)\r\nSzukaj: \"Data wystawienia\", \"Issue date\", \"Invoice date\"\r\nFormat wyjściowy: YYYY-MM-DD\r\n\r\nData sprzedaży (sale date)\r\nSzukaj: \"Data sprzedaży\"\r\nFormat wyjściowy: YYYY-MM-DD\r\n\r\nNIP sprzedawcy (seller's ID)\r\nSzukaj w danych adresowych sprzedawcy \"NIP\"\r\nFormat wyjściowy: dziesięcio liczbowy kod + może być poprzedzony przedrostkiem Kraju np. PL lub z myślnikami\r\n\r\n\r\nKwota netto (net_amount)\r\nSzukaj: \"Wartość netto\", \"Net amount\", \"Subtotal\"\r\nLiczba z kropką, 2 miejsca po przecinku\r\nBez waluty\r\n\r\nKwota brutto (gross_amount)\r\nSzukaj: \"Wartość brutto\", \"Gross amount\", \"Total\", \"Do zapłaty\", \"Total due\"\r\nLiczba z kropką, 2 miejsca po przecinku\r\nBez waluty\n\n\nKwota VAT (vat amount)\r\nSzukaj: \"Wartość VAT\",\"Kwota VAT\" \"vat amount\", \"Total \",\r\nLiczba z kropką, 2 miejsca po przecinku\r\nBez waluty\r\n\r\nZasady przetwarzania:\r\n- Dla nieznalezionych danych użyj null\r\n- Nie zgaduj brakujących danych\r\n- Przy wielu wartościach wybierz najbardziej prawdopodobną\r\n- Ignoruj przykładowe i przekreślone dane\r\n\r\nFormat odpowiedzi:\r\nZWRÓĆ TYLKO PONIŻSZY OBIEKT JSON, BEZ ŻADNYCH DODATKOWYCH ZNACZNIKÓW CZY KOMENTARZY:\r\n{\r\n\"invoice_number\": \"string or NOT_INVOICE or null\",\r\n\"supplier\": \"string or null\",\r\n\"issue_date\": \"YYYY-MM-DD or null\",\r\n\"sale date\": \"YYYY-MM-DD or null\",\r\n\"seller's ID\": number or null\r\n\"net_amount\": number or null,\r\n\"gross_amount\": number or null\r\n\"vat amount\":number or null\r\n}\nNiektóre faktury będą w innej walucie niż PLN. Np. \n*CZK, EUR,USD, HUF\n\nJeśli jest taka faktura to zastosuj przelicznik na PLN (polski złoty) z ostatniego dnia kursu.\r\n​",
                                        "type": "text"
                                    }
                                ],
                                "inputType": "multiple"
                            }
                        ],
                        "metadata": {},
                        "thinking": {
                            "type": "disabled"
                        },
                        "container": {},
                        "max_tokens": "8192",
                        "temperature": "1",
                        "outputFormat": {
                            "type": "text"
                        }
                    },
                    "metadata": {
                        "designer": {
                            "x": 426,
                            "y": 620,
                            "name": "📇 Skanowanie danych z FV"
                        },
                        "restore": {
                            "expect": {
                                "model": {
                                    "mode": "chose",
                                    "label": "Claude Sonnet 3.7"
                                },
                                "messages": {
                                    "mode": "chose",
                                    "items": [
                                        {
                                            "role": {
                                                "mode": "chose",
                                                "label": "User"
                                            },
                                            "content": {
                                                "mode": "chose",
                                                "items": [
                                                    {
                                                        "type": {
                                                            "mode": "chose",
                                                            "label": "PDF document"
                                                        }
                                                    },
                                                    {
                                                        "type": {
                                                            "mode": "chose",
                                                            "label": "Text"
                                                        }
                                                    }
                                                ]
                                            },
                                            "inputType": {
                                                "mode": "chose",
                                                "label": "Multiple content blocks"
                                            }
                                        }
                                    ]
                                },
                                "thinking": {
                                    "nested": {
                                        "type": {
                                            "mode": "chose",
                                            "label": "Disabled"
                                        }
                                    }
                                },
                                "container": {
                                    "nested": {
                                        "skills": {
                                            "mode": "chose"
                                        }
                                    }
                                },
                                "outputFormat": {
                                    "nested": {
                                        "type": {
                                            "mode": "chose",
                                            "label": "Text"
                                        }
                                    }
                                },
                                "stop_sequences": {
                                    "mode": "chose"
                                },
                                "additionalParameters": {
                                    "mode": "chose"
                                }
                            },
                            "parameters": {
                                "__IMTCONN__": {
                                    "data": {
                                        "scoped": "true",
                                        "connection": "anthropic-claude"
                                    },
                                    "label": "My Anthropic Claude connection"
                                }
                            }
                        },
                        "parameters": [
                            {
                                "name": "__IMTCONN__",
                                "type": "account:anthropic-claude",
                                "label": "Connection",
                                "required": true
                            }
                        ],
                        "expect": [
                            {
                                "name": "model",
                                "type": "select",
                                "label": "Model",
                                "required": true
                            },
                            {
                                "name": "max_tokens",
                                "type": "uinteger",
                                "label": "Max Tokens",
                                "required": true
                            },
                            {
                                "name": "messages",
                                "spec": {
                                    "name": "value",
                                    "spec": [
                                        {
                                            "name": "role",
                                            "type": "select",
                                            "label": "Role",
                                            "options": [
                                                {
                                                    "label": "User",
                                                    "value": "user",
                                                    "nested": [
                                                        {
                                                            "name": "inputType",
                                                            "type": "select",
                                                            "label": "Input Type",
                                                            "default": "multiple",
                                                            "options": [
                                                                {
                                                                    "label": "Single string",
                                                                    "value": "single",
                                                                    "nested": [
                                                                        {
                                                                            "help": "If the request ends with an assistant turn, then the response content will continue directly from that last turn.",
                                                                            "name": "content",
                                                                            "type": "text",
                                                                            "label": "Content",
                                                                            "required": true
                                                                        }
                                                                    ]
                                                                },
                                                                {
                                                                    "label": "Multiple content blocks",
                                                                    "value": "multiple",
                                                                    "nested": [
                                                                        {
                                                                            "name": "content",
                                                                            "spec": {
                                                                                "spec": [
                                                                                    {
                                                                                        "help": "For models that support image content blocks (vision), see the [Anthropic model comparison table](https://docs.anthropic.com/en/docs/about-claude/models#model-comparison-table).",
                                                                                        "name": "type",
                                                                                        "type": "select",
                                                                                        "label": "Type",
                                                                                        "options": [
                                                                                            {
                                                                                                "label": "Text",
                                                                                                "value": "text",
                                                                                                "nested": [
                                                                                                    {
                                                                                                        "help": "Content of the message on behalf of the selected **Role** .",
                                                                                                        "name": "text",
                                                                                                        "type": "text",
                                                                                                        "label": "Text",
                                                                                                        "required": true
                                                                                                    }
                                                                                                ]
                                                                                            },
                                                                                            {
                                                                                                "label": "Image",
                                                                                                "value": "image",
                                                                                                "nested": [
                                                                                                    {
                                                                                                        "name": "source",
                                                                                                        "spec": [
                                                                                                            {
                                                                                                                "help": "File name, including the extension, for example, `image.jpg`.",
                                                                                                                "name": "filename",
                                                                                                                "type": "filename",
                                                                                                                "label": "Filename",
                                                                                                                "required": true,
                                                                                                                "semantic": "file:name",
                                                                                                                "extension": [
                                                                                                                    "jpeg",
                                                                                                                    "jpg",
                                                                                                                    "png",
                                                                                                                    "gif",
                                                                                                                    "webp"
                                                                                                                ]
                                                                                                            },
                                                                                                            {
                                                                                                                "help": "Binary data of the image. For example, you can retrieve the binary data of your image using the **HTTP: Get a file** module.",
                                                                                                                "name": "data",
                                                                                                                "type": "buffer",
                                                                                                                "label": "Data",
                                                                                                                "required": true,
                                                                                                                "semantic": "file:data"
                                                                                                            }
                                                                                                        ],
                                                                                                        "type": "collection",
                                                                                                        "label": "Source",
                                                                                                        "required": true
                                                                                                    }
                                                                                                ]
                                                                                            },
                                                                                            {
                                                                                                "label": "PDF document",
                                                                                                "value": "document",
                                                                                                "nested": [
                                                                                                    "rpc://anthropic-claude/1/notePdfDocument",
                                                                                                    {
                                                                                                        "name": "pdfFileName",
                                                                                                        "type": "filename",
                                                                                                        "label": "Filename",
                                                                                                        "required": true,
                                                                                                        "semantic": "file:name",
                                                                                                        "extension": [
                                                                                                            "pdf"
                                                                                                        ]
                                                                                                    },
                                                                                                    {
                                                                                                        "name": "pdfFileData",
                                                                                                        "type": "buffer",
                                                                                                        "label": "Data",
                                                                                                        "required": true,
                                                                                                        "semantic": "file:data"
                                                                                                    }
                                                                                                ]
                                                                                            },
                                                                                            {
                                                                                                "label": "Tool result",
                                                                                                "value": "tool_result",
                                                                                                "nested": [
                                                                                                    {
                                                                                                        "name": "tool_use_id",
                                                                                                        "type": "text",
                                                                                                        "label": "Tool Use ID",
                                                                                                        "required": true
                                                                                                    },
                                                                                                    {
                                                                                                        "name": "inputType",
                                                                                                        "type": "select",
                                                                                                        "label": "Input Type",
                                                                                                        "default": "single",
                                                                                                        "options": [
                                                                                                            {
                                                                                                                "label": "Single string",
                                                                                                                "value": "single",
                                                                                                                "nested": [
                                                                                                                    {
                                                                                                                        "help": "If the request ends with an assistant turn, then the response content will continue directly from that last turn.",
                                                                                                                        "name": "content",
                                                                                                                        "type": "text",
                                                                                                                        "label": "Content",
                                                                                                                        "required": true
                                                                                                                    }
                                                                                                                ]
                                                                                                            },
                                                                                                            {
                                                                                                                "label": "Multiple content blocks",
                                                                                                                "value": "multiple",
                                                                                                                "nested": [
                                                                                                                    {
                                                                                                                        "name": "content",
                                                                                                                        "spec": {
                                                                                                                            "spec": [
                                                                                                                                {
                                                                                                                                    "help": "For models that support image content blocks (vision), see the [Anthropic model comparison table](https://docs.anthropic.com/en/docs/about-claude/models#model-comparison-table).",
                                                                                                                                    "name": "type",
                                                                                                                                    "type": "select",
                                                                                                                                    "label": "Type",
                                                                                                                                    "options": [
                                                                                                                                        {
                                                                                                                                            "label": "Text",
                                                                                                                                            "value": "text",
                                                                                                                                            "nested": [
                                                                                                                                                {
                                                                                                                                                    "help": "Content of the message on behalf of the selected **Role** .",
                                                                                                                                                    "name": "text",
                                                                                                                                                    "type": "text",
                                                                                                                                                    "label": "Text",
                                                                                                                                                    "required": true
                                                                                                                                                }
                                                                                                                                            ]
                                                                                                                                        },
                                                                                                                                        {
                                                                                                                                            "label": "Image",
                                                                                                                                            "value": "image",
                                                                                                                                            "nested": [
                                                                                                                                                {
                                                                                                                                                    "name": "source",
                                                                                                                                                    "spec": [
                                                                                                                                                        {
                                                                                                                                                            "help": "File name, including the extension, for example, `image.jpg`.",
                                                                                                                                                            "name": "filename",
                                                                                                                                                            "type": "filename",
                                                                                                                                                            "label": "Filename",
                                                                                                                                                            "required": true,
                                                                                                                                                            "semantic": "file:name",
                                                                                                                                                            "extension": [
                                                                                                                                                                "jpeg",
                                                                                                                                                                "jpg",
                                                                                                                                                                "png",
                                                                                                                                                                "gif",
                                                                                                                                                                "webp"
                                                                                                                                                            ]
                                                                                                                                                        },
                                                                                                                                                        {
                                                                                                                                                            "help": "Binary data of the image. For example, you can retrieve the binary data of your image using the **HTTP: Get a file** module.",
                                                                                                                                                            "name": "data",
                                                                                                                                                            "type": "buffer",
                                                                                                                                                            "label": "Data",
                                                                                                                                                            "required": true,
                                                                                                                                                            "semantic": "file:data"
                                                                                                                                                        }
                                                                                                                                                    ],
                                                                                                                                                    "type": "collection",
                                                                                                                                                    "label": "Source",
                                                                                                                                                    "required": true
                                                                                                                                                }
                                                                                                                                            ]
                                                                                                                                        },
                                                                                                                                        {
                                                                                                                                            "label": "PDF document",
                                                                                                                                            "value": "document",
                                                                                                                                            "nested": [
                                                                                                                                                "rpc://anthropic-claude/1/notePdfDocument",
                                                                                                                                                {
                                                                                                                                                    "name": "pdfFileName",
                                                                                                                                                    "type": "filename",
                                                                                                                                                    "label": "Filename",
                                                                                                                                                    "required": true,
                                                                                                                                                    "semantic": "file:name",
                                                                                                                                                    "extension": [
                                                                                                                                                        "pdf"
                                                                                                                                                    ]
                                                                                                                                                },
                                                                                                                                                {
                                                                                                                                                    "name": "pdfFileData",
                                                                                                                                                    "type": "buffer",
                                                                                                                                                    "label": "Data",
                                                                                                                                                    "required": true,
                                                                                                                                                    "semantic": "file:data"
                                                                                                                                                }
                                                                                                                                            ]
                                                                                                                                        }
                                                                                                                                    ],
                                                                                                                                    "required": true
                                                                                                                                }
                                                                                                                            ],
                                                                                                                            "type": "collection",
                                                                                                                            "label": "Content"
                                                                                                                        },
                                                                                                                        "type": "array",
                                                                                                                        "label": "Content",
                                                                                                                        "labels": {
                                                                                                                            "add": "Add content"
                                                                                                                        },
                                                                                                                        "required": true
                                                                                                                    }
                                                                                                                ]
                                                                                                            }
                                                                                                        ],
                                                                                                        "mappable": false,
                                                                                                        "required": true
                                                                                                    }
                                                                                                ]
                                                                                            }
                                                                                        ],
                                                                                        "required": true
                                                                                    }
                                                                                ],
                                                                                "type": "collection",
                                                                                "label": "Content"
                                                                            },
                                                                            "type": "array",
                                                                            "label": "Content",
                                                                            "labels": {
                                                                                "add": "Add content"
                                                                            },
                                                                            "required": true
                                                                        }
                                                                    ]
                                                                }
                                                            ],
                                                            "mappable": false,
                                                            "required": true
                                                        }
                                                    ]
                                                },
                                                {
                                                    "label": "Assistant",
                                                    "value": "assistant",
                                                    "nested": [
                                                        {
                                                            "name": "inputType",
                                                            "type": "select",
                                                            "label": "Input Type",
                                                            "default": "single",
                                                            "options": [
                                                                {
                                                                    "label": "Single string",
                                                                    "value": "single",
                                                                    "nested": [
                                                                        {
                                                                            "help": "If the request ends with an assistant turn, then the response content will continue directly from that last turn.",
                                                                            "name": "content",
                                                                            "type": "text",
                                                                            "label": "Content",
                                                                            "required": true
                                                                        }
                                                                    ]
                                                                },
                                                                {
                                                                    "label": "Multiple content blocks",
                                                                    "value": "multiple",
                                                                    "nested": [
                                                                        {
                                                                            "name": "content",
                                                                            "spec": {
                                                                                "spec": [
                                                                                    {
                                                                                        "help": "For models that support image content blocks (vision), see the [Anthropic model comparison table](https://docs.anthropic.com/en/docs/about-claude/models#model-comparison-table).",
                                                                                        "name": "type",
                                                                                        "type": "select",
                                                                                        "label": "Type",
                                                                                        "options": [
                                                                                            {
                                                                                                "label": "Text",
                                                                                                "value": "text",
                                                                                                "nested": [
                                                                                                    {
                                                                                                        "help": "Content of the message on behalf of the selected **Role** .",
                                                                                                        "name": "text",
                                                                                                        "type": "text",
                                                                                                        "label": "Text",
                                                                                                        "required": true
                                                                                                    }
                                                                                                ]
                                                                                            },
                                                                                            {
                                                                                                "label": "Tool use",
                                                                                                "value": "tool_use",
                                                                                                "nested": [
                                                                                                    {
                                                                                                        "name": "id",
                                                                                                        "type": "text",
                                                                                                        "label": "Tool ID",
                                                                                                        "required": true
                                                                                                    },
                                                                                                    {
                                                                                                        "name": "name",
                                                                                                        "type": "text",
                                                                                                        "label": "Name",
                                                                                                        "required": true
                                                                                                    },
                                                                                                    {
                                                                                                        "help": "To map a value from a previous module, follow the syntax `{{1.id}}`.",
                                                                                                        "name": "input",
                                                                                                        "type": "editor",
                                                                                                        "label": "Tool input",
                                                                                                        "editor": "monaco",
                                                                                                        "default": "{}",
                                                                                                        "language": "json",
                                                                                                        "required": true
                                                                                                    }
                                                                                                ]
                                                                                            }
                                                                                        ],
                                                                                        "required": true
                                                                                    }
                                                                                ],
                                                                                "type": "collection",
                                                                                "label": "Content"
                                                                            },
                                                                            "type": "array",
                                                                            "label": "Content",
                                                                            "labels": {
                                                                                "add": "Add content"
                                                                            },
                                                                            "required": true
                                                                        }
                                                                    ]
                                                                }
                                                            ],
                                                            "mappable": false,
                                                            "required": true
                                                        }
                                                    ]
                                                }
                                            ],
                                            "required": true
                                        }
                                    ],
                                    "type": "collection",
                                    "label": "Message"
                                },
                                "type": "array",
                                "label": "Messages",
                                "required": true
                            },
                            {
                                "name": "thinking",
                                "spec": [
                                    {
                                        "name": "type",
                                        "type": "select",
                                        "label": "Type",
                                        "validate": {
                                            "enum": [
                                                "enabled",
                                                "disabled"
                                            ]
                                        }
                                    }
                                ],
                                "type": "collection",
                                "label": "Extended Thinking"
                            },
                            {
                                "name": "container",
                                "spec": [
                                    {
                                        "name": "id",
                                        "type": "text",
                                        "label": "Container ID"
                                    },
                                    {
                                        "name": "skills",
                                        "spec": {
                                            "name": "value",
                                            "spec": [
                                                {
                                                    "name": "type",
                                                    "type": "select",
                                                    "label": "Type",
                                                    "options": {
                                                        "store": [
                                                            {
                                                                "label": "Custom",
                                                                "value": "custom"
                                                            },
                                                            {
                                                                "label": "Anthropic",
                                                                "value": "anthropic"
                                                            }
                                                        ]
                                                    },
                                                    "required": true
                                                }
                                            ],
                                            "type": "collection",
                                            "label": "Skill"
                                        },
                                        "type": "array",
                                        "label": "Skills",
                                        "validate": {
                                            "maxItems": 8
                                        }
                                    }
                                ],
                                "type": "collection",
                                "label": "Container"
                            },
                            {
                                "name": "system",
                                "type": "text",
                                "label": "System Prompt"
                            },
                            {
                                "name": "metadata",
                                "spec": [
                                    {
                                        "name": "user_id",
                                        "type": "text",
                                        "label": "User ID"
                                    }
                                ],
                                "type": "collection",
                                "label": "Metadata"
                            },
                            {
                                "name": "stop_sequences",
                                "spec": {
                                    "name": "value",
                                    "type": "text",
                                    "label": "Stop Sequence"
                                },
                                "type": "array",
                                "label": "Stop Sequences"
                            },
                            {
                                "name": "temperature",
                                "type": "number",
                                "label": "Temperature",
                                "validate": {
                                    "max": 1,
                                    "min": 0
                                }
                            },
                            {
                                "name": "top_p",
                                "type": "number",
                                "label": "Top P",
                                "validate": {
                                    "max": 1
                                }
                            },
                            {
                                "name": "top_k",
                                "type": "uinteger",
                                "label": "Top K"
                            },
                            {
                                "name": "outputFormat",
                                "spec": [
                                    {
                                        "name": "type",
                                        "type": "select",
                                        "label": "Type",
                                        "validate": {
                                            "enum": [
                                                "text",
                                                "json_schema"
                                            ]
                                        }
                                    }
                                ],
                                "type": "collection",
                                "label": "Output Format"
                            },
                            {
                                "name": "additionalParameters",
                                "spec": {
                                    "name": "value",
                                    "spec": [
                                        {
                                            "name": "key",
                                            "type": "text",
                                            "label": "Parameter Name",
                                            "required": true
                                        },
                                        {
                                            "name": "type",
                                            "type": "select",
                                            "label": "Input Type",
                                            "options": [
                                                {
                                                    "label": "Text",
                                                    "value": "text",
                                                    "nested": [
                                                        {
                                                            "name": "value",
                                                            "type": "text",
                                                            "label": "Parameter Value"
                                                        }
                                                    ],
                                                    "default": true
                                                },
                                                {
                                                    "label": "Number",
                                                    "value": "number",
                                                    "nested": [
                                                        {
                                                            "name": "value",
                                                            "type": "number",
                                                            "label": "Parameter Value"
                                                        }
                                                    ]
                                                },
                                                {
                                                    "label": "Boolean",
                                                    "value": "boolean",
                                                    "nested": [
                                                        {
                                                            "name": "value",
                                                            "type": "boolean",
                                                            "label": "Parameter Value"
                                                        }
                                                    ]
                                                },
                                                {
                                                    "label": "Date",
                                                    "value": "date",
                                                    "nested": [
                                                        {
                                                            "name": "value",
                                                            "type": "date",
                                                            "label": "Parameter Value"
                                                        }
                                                    ]
                                                },
                                                {
                                                    "label": "Tools",
                                                    "value": "tools",
                                                    "nested": [
                                                        {
                                                            "name": "value",
                                                            "spec": {
                                                                "type": "text",
                                                                "label": "Parameter Value"
                                                            },
                                                            "type": "array",
                                                            "label": "Parameter Values",
                                                            "labels": {
                                                                "add": "Add parameter value"
                                                            }
                                                        }
                                                    ]
                                                },
                                                {
                                                    "label": "Any",
                                                    "value": "any",
                                                    "nested": [
                                                        {
                                                            "name": "value",
                                                            "type": "any",
                                                            "label": "Parameter Value"
                                                        }
                                                    ]
                                                }
                                            ]
                                        }
                                    ],
                                    "type": "collection",
                                    "label": "Input Parameter"
                                },
                                "type": "array",
                                "label": "Other Input Parameters"
                            }
                        ],
                        "interface": [
                            {
                                "name": "files",
                                "spec": [
                                    {
                                        "name": "fileName",
                                        "type": "filename",
                                        "label": "File Name",
                                        "semantic": "file:name"
                                    },
                                    {
                                        "name": "fileData",
                                        "type": "buffer",
                                        "label": "File Data",
                                        "semantic": "file:data"
                                    },
                                    {
                                        "name": "fileId",
                                        "type": "text",
                                        "label": "File ID"
                                    }
                                ],
                                "type": "array",
                                "label": "Files"
                            },
                            {
                                "name": "textResponse",
                                "type": "text",
                                "label": "Text Response"
                            },
                            {
                                "name": "id",
                                "type": "text",
                                "label": "Unique ID"
                            },
                            {
                                "name": "type",
                                "type": "text",
                                "label": "Type"
                            },
                            {
                                "name": "role",
                                "type": "text",
                                "label": "Role"
                            },
                            {
                                "name": "content",
                                "spec": {
                                    "spec": [
                                        {
                                            "name": "type",
                                            "type": "text",
                                            "label": "Type"
                                        },
                                        {
                                            "name": "thinking",
                                            "type": "text",
                                            "label": "Thinking"
                                        },
                                        {
                                            "name": "signature",
                                            "type": "text",
                                            "label": "Signature"
                                        },
                                        {
                                            "name": "text",
                                            "type": "text",
                                            "label": "Text"
                                        },
                                        {
                                            "name": "id",
                                            "type": "text",
                                            "label": "ID"
                                        },
                                        {
                                            "name": "name",
                                            "type": "text",
                                            "label": "Name"
                                        },
                                        {
                                            "name": "input",
                                            "spec": [],
                                            "type": "collection",
                                            "label": "Input"
                                        }
                                    ],
                                    "type": "collection"
                                },
                                "type": "array",
                                "label": "Content"
                            },
                            {
                                "name": "model",
                                "type": "text",
                                "label": "Model"
                            },
                            {
                                "name": "container",
                                "spec": [
                                    {
                                        "name": "id",
                                        "type": "text",
                                        "label": "ID"
                                    },
                                    {
                                        "name": "expires_at",
                                        "type": "date",
                                        "label": "Expires at"
                                    },
                                    {
                                        "name": "skills",
                                        "spec": {
                                            "spec": [
                                                {
                                                    "name": "type",
                                                    "type": "text",
                                                    "label": "Type"
                                                },
                                                {
                                                    "name": "skill_id",
                                                    "type": "text",
                                                    "label": "Skill ID"
                                                },
                                                {
                                                    "name": "version",
                                                    "type": "text",
                                                    "label": "Version"
                                                }
                                            ],
                                            "type": "collection"
                                        },
                                        "type": "array",
                                        "label": "Skills"
                                    }
                                ],
                                "type": "collection",
                                "label": "Container"
                            },
                            {
                                "name": "stop_reason",
                                "type": "text",
                                "label": "Stop Reason"
                            },
                            {
                                "name": "stop_sequence",
                                "type": "text",
                                "label": "Stop Sequence"
                            },
                            {
                                "name": "usage",
                                "spec": [
                                    {
                                        "name": "input_tokens",
                                        "type": "uinteger",
                                        "label": "Input Tokens"
                                    },
                                    {
                                        "name": "cache_creation_input_tokens",
                                        "type": "number",
                                        "label": "Cache Creation Input Tokens"
                                    },
                                    {
                                        "name": "cache_read_input_tokens",
                                        "type": "number",
                                        "label": "Cache Read Input Tokens"
                                    },
                                    {
                                        "name": "output_tokens",
                                        "type": "uinteger",
                                        "label": "Output Tokens"
                                    },
                                    {
                                        "name": "cache_creation",
                                        "spec": [
                                            {
                                                "name": "ephemeral_5m_input_tokens",
                                                "type": "number",
                                                "label": "Ephemeral 5m Input Tokens"
                                            },
                                            {
                                                "name": "ephemeral_1h_input_tokens",
                                                "type": "number",
                                                "label": "Ephemeral 1h Input Tokens"
                                            }
                                        ],
                                        "type": "collection",
                                        "label": "Cache Creation"
                                    },
                                    {
                                        "name": "service_tier",
                                        "type": "text",
                                        "label": "Service Tier"
                                    },
                                    {
                                        "name": "server_tool_use",
                                        "spec": [
                                            {
                                                "name": "web_search_requests",
                                                "type": "number",
                                                "label": "Web Search Requests"
                                            }
                                        ],
                                        "type": "collection",
                                        "label": "Server Tool  Use"
                                    }
                                ],
                                "type": "collection",
                                "label": "Usage"
                            },
                            {
                                "name": "tools",
                                "spec": [],
                                "type": "collection",
                                "label": "Tools"
                            }
                        ]
                    },
                    "onerror": [
                        {
                            "id": 109,
                            "module": "util:FunctionSleep",
                            "version": 1,
                            "parameters": {},
                            "mapper": {
                                "duration": "70"
                            },
                            "metadata": {
                                "designer": {
                                    "x": 673,
                                    "y": 954
                                },
                                "restore": {},
                                "expect": [
                                    {
                                        "name": "duration",
                                        "type": "uinteger",
                                        "label": "Delay",
                                        "required": true,
                                        "validate": {
                                            "max": 300,
                                            "min": 1
                                        }
                                    }
                                ]
                            }
                        },
                        {
                            "id": 115,
                            "module": "anthropic-claude:createAMessage",
                            "version": 1,
                            "parameters": {
                                "__IMTCONN__": 8853167
                            },
                            "mapper": {
                                "model": "claude-3-7-sonnet-20250219",
                                "messages": [
                                    {
                                        "role": "user",
                                        "content": [
                                            {
                                                "type": "document",
                                                "pdfFileData": "{{1.attachments[].data}}",
                                                "pdfFileName": "{{1.attachments[].fileName}}"
                                            },
                                            {
                                                "text": "Instrukcja ekstrakcji danych z faktur\r\nNa podstawie dostarczonego tekstu faktury, znajdź i wyodrębnij następujące informacje. Zwróć TYLKO obiekt JSON, bez żadnych dodatkowych komentarzy, oznaczeń formatowania czy znaczników kodu.\r\n\r\nNajpierw sprawdź czy to jest faktura szukając wskaźników:\r\n\r\nA. Bezpośrednie wskazania faktury:\r\n- Słowa kluczowe: \"faktura\", \"FV\", \"VAT\", \"rachunek\", \"należność\", \"dokument księgowy\", \"invoice\", \"invoiced\", \"billing\"\r\n- Numery faktur (np. FV/2024/123, VAT/2024/01)\r\n- Frazy \"w załączeniu faktura\", \"przesyłam fakturę\", \"załączam FV\"\r\n\r\nB. Pośrednie wskazania faktury:\r\n- Kwoty z określeniem \"netto\" lub \"brutto\"\r\n- Kwoty poprzedzone znakiem waluty ($, €, £)\r\n- Terminy płatności (np. \"termin płatności\", \"płatne do\")\r\n- Numery kont bankowych\r\n- Dane do przelewu\r\n- \"Do zapłaty\"\r\n- \"Proszę o zapłatę\"\r\n- \"Proforma\"\r\n- \"Nota księgowa\"\r\n- \"Rozliczenie\"\r\n- Frazy potwierdzające wystawienie dokumentu księgowego:\r\n * \"invoiced amount\"\r\n * \"payment received\"\r\n * \"payment confirmation\"\r\n- Daty transakcji/płatności w kontekście kwot\r\n- Informacje o metodzie płatności (np. \"payment method\", \"card ending with\")\r\n\r\nJeśli to nie jest faktura (brak powyższych wskaźników), zwróć obiekt JSON z wartością \"NOT_INVOICE\" w polu invoice_number i null w pozostałych polach.\r\n\r\nJeśli to jest faktura, wyodrębnij następujące informacje:\r\n\r\nNumer faktury (invoice_number)\r\nSzukaj: \"FV\", \"Faktura VAT\", \"Invoice no.\", \"Invoice number\"\r\nFormaty: FV/2024/01/123, INV-2024-001, 2024/ABC/123\r\n\r\nDane dostawcy (supplier)\r\nSzukaj: \"Sprzedawca\", \"Wystawca\", \"Supplier\", \"Seller\"\r\nTylko nazwa firmy, bez adresu\r\n\r\nData wystawienia (issue_date)\r\nSzukaj: \"Data wystawienia\", \"Issue date\", \"Invoice date\"\r\nFormat wyjściowy: YYYY-MM-DD\r\n\r\nData sprzedaży (sale date)\r\nSzukaj: \"Data sprzedaży\"\r\nFormat wyjściowy: YYYY-MM-DD\r\n\r\nNIP sprzedawcy (seller's ID)\r\nSzukaj w danych adresowych sprzedawcy \"NIP\"\r\nFormat wyjściowy: dziesięcio liczbowy kod, może być poprzedzony przedrostkiem Kraju np. PL lub z myślnikami\r\n\r\n\r\nKwota netto (net_amount)\r\nSzukaj: \"Wartość netto\", \"Net amount\", \"Subtotal\"\r\nLiczba z kropką, 2 miejsca po przecinku\r\nBez waluty\r\n\r\nKwota brutto (gross_amount)\r\nSzukaj: \"Wartość brutto\", \"Gross amount\", \"Total\", \"Do zapłaty\", \"Total due\"\r\nLiczba z kropką, 2 miejsca po przecinku\r\nBez waluty\r\n\r\nZasady przetwarzania:\r\n- Dla nieznalezionych danych użyj null\r\n- Nie zgaduj brakujących danych\r\n- Przy wielu wartościach wybierz najbardziej prawdopodobną\r\n- Ignoruj przykładowe i przekreślone dane\r\n\r\nFormat odpowiedzi:\r\nZWRÓĆ TYLKO PONIŻSZY OBIEKT JSON, BEZ ŻADNYCH DODATKOWYCH ZNACZNIKÓW CZY KOMENTARZY:\r\n{\r\n\"invoice_number\": \"string or NOT_INVOICE or null\",\r\n\"supplier\": \"string or null\",\r\n\"issue_date\": \"YYYY-MM-DD or null\",\r\n\"sale date\": \"YYYY-MM-DD or null\",\r\n\"seller's ID\": number or null\r\n\"net_amount\": number or null,\r\n\"gross_amount\": number or null\r\n\"vat amount\":number or null\r\n}",
                                                "type": "text"
                                            }
                                        ],
                                        "inputType": "multiple"
                                    }
                                ],
                                "metadata": {},
                                "thinking": {
                                    "type": "disabled"
                                },
                                "container": {},
                                "max_tokens": "8192",
                                "temperature": "1",
                                "outputFormat": {
                                    "type": "text"
                                }
                            },
                            "metadata": {
                                "designer": {
                                    "x": 991,
                                    "y": 1077,
                                    "name": "📇 Skanowanie danych z FV"
                                },
                                "restore": {
                                    "expect": {
                                        "model": {
                                            "mode": "chose",
                                            "label": "Claude Sonnet 3.7"
                                        },
                                        "messages": {
                                            "mode": "chose",
                                            "items": [
                                                {
                                                    "role": {
                                                        "mode": "chose",
                                                        "label": "User"
                                                    },
                                                    "content": {
                                                        "mode": "chose",
                                                        "items": [
                                                            {
                                                                "type": {
                                                                    "mode": "chose",
                                                                    "label": "PDF document"
                                                                }
                                                            },
                                                            {
                                                                "type": {
                                                                    "mode": "chose",
                                                                    "label": "Text"
                                                                }
                                                            }
                                                        ]
                                                    },
                                                    "inputType": {
                                                        "mode": "chose",
                                                        "label": "Multiple content blocks"
                                                    }
                                                }
                                            ]
                                        },
                                        "thinking": {
                                            "nested": {
                                                "type": {
                                                    "mode": "chose",
                                                    "label": "Disabled"
                                                }
                                            }
                                        },
                                        "container": {
                                            "nested": {
                                                "skills": {
                                                    "mode": "chose"
                                                }
                                            }
                                        },
                                        "outputFormat": {
                                            "nested": {
                                                "type": {
                                                    "mode": "chose",
                                                    "label": "Text"
                                                }
                                            }
                                        },
                                        "stop_sequences": {
                                            "mode": "chose"
                                        },
                                        "additionalParameters": {
                                            "mode": "chose"
                                        }
                                    },
                                    "parameters": {
                                        "__IMTCONN__": {
                                            "data": {
                                                "scoped": "true",
                                                "connection": "anthropic-claude"
                                            },
                                            "label": "My Anthropic Claude connection"
                                        }
                                    }
                                },
                                "parameters": [
                                    {
                                        "name": "__IMTCONN__",
                                        "type": "account:anthropic-claude",
                                        "label": "Connection",
                                        "required": true
                                    }
                                ],
                                "expect": [
                                    {
                                        "name": "model",
                                        "type": "select",
                                        "label": "Model",
                                        "required": true
                                    },
                                    {
                                        "name": "max_tokens",
                                        "type": "uinteger",
                                        "label": "Max Tokens",
                                        "required": true
                                    },
                                    {
                                        "name": "messages",
                                        "spec": {
                                            "name": "value",
                                            "spec": [
                                                {
                                                    "name": "role",
                                                    "type": "select",
                                                    "label": "Role",
                                                    "options": [
                                                        {
                                                            "label": "User",
                                                            "value": "user",
                                                            "nested": [
                                                                {
                                                                    "name": "inputType",
                                                                    "type": "select",
                                                                    "label": "Input Type",
                                                                    "default": "multiple",
                                                                    "options": [
                                                                        {
                                                                            "label": "Single string",
                                                                            "value": "single",
                                                                            "nested": [
                                                                                {
                                                                                    "help": "If the request ends with an assistant turn, then the response content will continue directly from that last turn.",
                                                                                    "name": "content",
                                                                                    "type": "text",
                                                                                    "label": "Content",
                                                                                    "required": true
                                                                                }
                                                                            ]
                                                                        },
                                                                        {
                                                                            "label": "Multiple content blocks",
                                                                            "value": "multiple",
                                                                            "nested": [
                                                                                {
                                                                                    "name": "content",
                                                                                    "spec": {
                                                                                        "spec": [
                                                                                            {
                                                                                                "help": "For models that support image content blocks (vision), see the [Anthropic model comparison table](https://docs.anthropic.com/en/docs/about-claude/models#model-comparison-table).",
                                                                                                "name": "type",
                                                                                                "type": "select",
                                                                                                "label": "Type",
                                                                                                "options": [
                                                                                                    {
                                                                                                        "label": "Text",
                                                                                                        "value": "text",
                                                                                                        "nested": [
                                                                                                            {
                                                                                                                "help": "Content of the message on behalf of the selected **Role** .",
                                                                                                                "name": "text",
                                                                                                                "type": "text",
                                                                                                                "label": "Text",
                                                                                                                "required": true
                                                                                                            }
                                                                                                        ]
                                                                                                    },
                                                                                                    {
                                                                                                        "label": "Image",
                                                                                                        "value": "image",
                                                                                                        "nested": [
                                                                                                            {
                                                                                                                "name": "source",
                                                                                                                "spec": [
                                                                                                                    {
                                                                                                                        "help": "File name, including the extension, for example, `image.jpg`.",
                                                                                                                        "name": "filename",
                                                                                                                        "type": "filename",
                                                                                                                        "label": "Filename",
                                                                                                                        "required": true,
                                                                                                                        "semantic": "file:name",
                                                                                                                        "extension": [
                                                                                                                            "jpeg",
                                                                                                                            "jpg",
                                                                                                                            "png",
                                                                                                                            "gif",
                                                                                                                            "webp"
                                                                                                                        ]
                                                                                                                    },
                                                                                                                    {
                                                                                                                        "help": "Binary data of the image. For example, you can retrieve the binary data of your image using the **HTTP: Get a file** module.",
                                                                                                                        "name": "data",
                                                                                                                        "type": "buffer",
                                                                                                                        "label": "Data",
                                                                                                                        "required": true,
                                                                                                                        "semantic": "file:data"
                                                                                                                    }
                                                                                                                ],
                                                                                                                "type": "collection",
                                                                                                                "label": "Source",
                                                                                                                "required": true
                                                                                                            }
                                                                                                        ]
                                                                                                    },
                                                                                                    {
                                                                                                        "label": "PDF document",
                                                                                                        "value": "document",
                                                                                                        "nested": [
                                                                                                            "rpc://anthropic-claude/1/notePdfDocument",
                                                                                                            {
                                                                                                                "name": "pdfFileName",
                                                                                                                "type": "filename",
                                                                                                                "label": "Filename",
                                                                                                                "required": true,
                                                                                                                "semantic": "file:name",
                                                                                                                "extension": [
                                                                                                                    "pdf"
                                                                                                                ]
                                                                                                            },
                                                                                                            {
                                                                                                                "name": "pdfFileData",
                                                                                                                "type": "buffer",
                                                                                                                "label": "Data",
                                                                                                                "required": true,
                                                                                                                "semantic": "file:data"
                                                                                                            }
                                                                                                        ]
                                                                                                    },
                                                                                                    {
                                                                                                        "label": "Tool result",
                                                                                                        "value": "tool_result",
                                                                                                        "nested": [
                                                                                                            {
                                                                                                                "name": "tool_use_id",
                                                                                                                "type": "text",
                                                                                                                "label": "Tool Use ID",
                                                                                                                "required": true
                                                                                                            },
                                                                                                            {
                                                                                                                "name": "inputType",
                                                                                                                "type": "select",
                                                                                                                "label": "Input Type",
                                                                                                                "default": "single",
                                                                                                                "options": [
                                                                                                                    {
                                                                                                                        "label": "Single string",
                                                                                                                        "value": "single",
                                                                                                                        "nested": [
                                                                                                                            {
                                                                                                                                "help": "If the request ends with an assistant turn, then the response content will continue directly from that last turn.",
                                                                                                                                "name": "content",
                                                                                                                                "type": "text",
                                                                                                                                "label": "Content",
                                                                                                                                "required": true
                                                                                                                            }
                                                                                                                        ]
                                                                                                                    },
                                                                                                                    {
                                                                                                                        "label": "Multiple content blocks",
                                                                                                                        "value": "multiple",
                                                                                                                        "nested": [
                                                                                                                            {
                                                                                                                                "name": "content",
                                                                                                                                "spec": {
                                                                                                                                    "spec": [
                                                                                                                                        {
                                                                                                                                            "help": "For models that support image content blocks (vision), see the [Anthropic model comparison table](https://docs.anthropic.com/en/docs/about-claude/models#model-comparison-table).",
                                                                                                                                            "name": "type",
                                                                                                                                            "type": "select",
                                                                                                                                            "label": "Type",
                                                                                                                                            "options": [
                                                                                                                                                {
                                                                                                                                                    "label": "Text",
                                                                                                                                                    "value": "text",
                                                                                                                                                    "nested": [
                                                                                                                                                        {
                                                                                                                                                            "help": "Content of the message on behalf of the selected **Role** .",
                                                                                                                                                            "name": "text",
                                                                                                                                                            "type": "text",
                                                                                                                                                            "label": "Text",
                                                                                                                                                            "required": true
                                                                                                                                                        }
                                                                                                                                                    ]
                                                                                                                                                },
                                                                                                                                                {
                                                                                                                                                    "label": "Image",
                                                                                                                                                    "value": "image",
                                                                                                                                                    "nested": [
                                                                                                                                                        {
                                                                                                                                                            "name": "source",
                                                                                                                                                            "spec": [
                                                                                                                                                                {
                                                                                                                                                                    "help": "File name, including the extension, for example, `image.jpg`.",
                                                                                                                                                                    "name": "filename",
                                                                                                                                                                    "type": "filename",
                                                                                                                                                                    "label": "Filename",
                                                                                                                                                                    "required": true,
                                                                                                                                                                    "semantic": "file:name",
                                                                                                                                                                    "extension": [
                                                                                                                                                                        "jpeg",
                                                                                                                                                                        "jpg",
                                                                                                                                                                        "png",
                                                                                                                                                                        "gif",
                                                                                                                                                                        "webp"
                                                                                                                                                                    ]
                                                                                                                                                                },
                                                                                                                                                                {
                                                                                                                                                                    "help": "Binary data of the image. For example, you can retrieve the binary data of your image using the **HTTP: Get a file** module.",
                                                                                                                                                                    "name": "data",
                                                                                                                                                                    "type": "buffer",
                                                                                                                                                                    "label": "Data",
                                                                                                                                                                    "required": true,
                                                                                                                                                                    "semantic": "file:data"
                                                                                                                                                                }
                                                                                                                                                            ],
                                                                                                                                                            "type": "collection",
                                                                                                                                                            "label": "Source",
                                                                                                                                                            "required": true
                                                                                                                                                        }
                                                                                                                                                    ]
                                                                                                                                                },
                                                                                                                                                {
                                                                                                                                                    "label": "PDF document",
                                                                                                                                                    "value": "document",
                                                                                                                                                    "nested": [
                                                                                                                                                        "rpc://anthropic-claude/1/notePdfDocument",
                                                                                                                                                        {
                                                                                                                                                            "name": "pdfFileName",
                                                                                                                                                            "type": "filename",
                                                                                                                                                            "label": "Filename",
                                                                                                                                                            "required": true,
                                                                                                                                                            "semantic": "file:name",
                                                                                                                                                            "extension": [
                                                                                                                                                                "pdf"
                                                                                                                                                            ]
                                                                                                                                                        },
                                                                                                                                                        {
                                                                                                                                                            "name": "pdfFileData",
                                                                                                                                                            "type": "buffer",
                                                                                                                                                            "label": "Data",
                                                                                                                                                            "required": true,
                                                                                                                                                            "semantic": "file:data"
                                                                                                                                                        }
                                                                                                                                                    ]
                                                                                                                                                }
                                                                                                                                            ],
                                                                                                                                            "required": true
                                                                                                                                        }
                                                                                                                                    ],
                                                                                                                                    "type": "collection",
                                                                                                                                    "label": "Content"
                                                                                                                                },
                                                                                                                                "type": "array",
                                                                                                                                "label": "Content",
                                                                                                                                "labels": {
                                                                                                                                    "add": "Add content"
                                                                                                                                },
                                                                                                                                "required": true
                                                                                                                            }
                                                                                                                        ]
                                                                                                                    }
                                                                                                                ],
                                                                                                                "mappable": false,
                                                                                                                "required": true
                                                                                                            }
                                                                                                        ]
                                                                                                    }
                                                                                                ],
                                                                                                "required": true
                                                                                            }
                                                                                        ],
                                                                                        "type": "collection",
                                                                                        "label": "Content"
                                                                                    },
                                                                                    "type": "array",
                                                                                    "label": "Content",
                                                                                    "labels": {
                                                                                        "add": "Add content"
                                                                                    },
                                                                                    "required": true
                                                                                }
                                                                            ]
                                                                        }
                                                                    ],
                                                                    "mappable": false,
                                                                    "required": true
                                                                }
                                                            ]
                                                        },
                                                        {
                                                            "label": "Assistant",
                                                            "value": "assistant",
                                                            "nested": [
                                                                {
                                                                    "name": "inputType",
                                                                    "type": "select",
                                                                    "label": "Input Type",
                                                                    "default": "single",
                                                                    "options": [
                                                                        {
                                                                            "label": "Single string",
                                                                            "value": "single",
                                                                            "nested": [
                                                                                {
                                                                                    "help": "If the request ends with an assistant turn, then the response content will continue directly from that last turn.",
                                                                                    "name": "content",
                                                                                    "type": "text",
                                                                                    "label": "Content",
                                                                                    "required": true
                                                                                }
                                                                            ]
                                                                        },
                                                                        {
                                                                            "label": "Multiple content blocks",
                                                                            "value": "multiple",
                                                                            "nested": [
                                                                                {
                                                                                    "name": "content",
                                                                                    "spec": {
                                                                                        "spec": [
                                                                                            {
                                                                                                "help": "For models that support image content blocks (vision), see the [Anthropic model comparison table](https://docs.anthropic.com/en/docs/about-claude/models#model-comparison-table).",
                                                                                                "name": "type",
                                                                                                "type": "select",
                                                                                                "label": "Type",
                                                                                                "options": [
                                                                                                    {
                                                                                                        "label": "Text",
                                                                                                        "value": "text",
                                                                                                        "nested": [
                                                                                                            {
                                                                                                                "help": "Content of the message on behalf of the selected **Role** .",
                                                                                                                "name": "text",
                                                                                                                "type": "text",
                                                                                                                "label": "Text",
                                                                                                                "required": true
                                                                                                            }
                                                                                                        ]
                                                                                                    },
                                                                                                    {
                                                                                                        "label": "Tool use",
                                                                                                        "value": "tool_use",
                                                                                                        "nested": [
                                                                                                            {
                                                                                                                "name": "id",
                                                                                                                "type": "text",
                                                                                                                "label": "Tool ID",
                                                                                                                "required": true
                                                                                                            },
                                                                                                            {
                                                                                                                "name": "name",
                                                                                                                "type": "text",
                                                                                                                "label": "Name",
                                                                                                                "required": true
                                                                                                            },
                                                                                                            {
                                                                                                                "help": "To map a value from a previous module, follow the syntax `{{1.id}}`.",
                                                                                                                "name": "input",
                                                                                                                "type": "editor",
                                                                                                                "label": "Tool input",
                                                                                                                "editor": "monaco",
                                                                                                                "default": "{}",
                                                                                                                "language": "json",
                                                                                                                "required": true
                                                                                                            }
                                                                                                        ]
                                                                                                    }
                                                                                                ],
                                                                                                "required": true
                                                                                            }
                                                                                        ],
                                                                                        "type": "collection",
                                                                                        "label": "Content"
                                                                                    },
                                                                                    "type": "array",
                                                                                    "label": "Content",
                                                                                    "labels": {
                                                                                        "add": "Add content"
                                                                                    },
                                                                                    "required": true
                                                                                }
                                                                            ]
                                                                        }
                                                                    ],
                                                                    "mappable": false,
                                                                    "required": true
                                                                }
                                                            ]
                                                        }
                                                    ],
                                                    "required": true
                                                }
                                            ],
                                            "type": "collection",
                                            "label": "Message"
                                        },
                                        "type": "array",
                                        "label": "Messages",
                                        "required": true
                                    },
                                    {
                                        "name": "thinking",
                                        "spec": [
                                            {
                                                "name": "type",
                                                "type": "select",
                                                "label": "Type",
                                                "validate": {
                                                    "enum": [
                                                        "enabled",
                                                        "disabled"
                                                    ]
                                                }
                                            }
                                        ],
                                        "type": "collection",
                                        "label": "Extended Thinking"
                                    },
                                    {
                                        "name": "container",
                                        "spec": [
                                            {
                                                "name": "id",
                                                "type": "text",
                                                "label": "Container ID"
                                            },
                                            {
                                                "name": "skills",
                                                "spec": {
                                                    "name": "value",
                                                    "spec": [
                                                        {
                                                            "name": "type",
                                                            "type": "select",
                                                            "label": "Type",
                                                            "options": {
                                                                "store": [
                                                                    {
                                                                        "label": "Custom",
                                                                        "value": "custom"
                                                                    },
                                                                    {
                                                                        "label": "Anthropic",
                                                                        "value": "anthropic"
                                                                    }
                                                                ]
                                                            },
                                                            "required": true
                                                        }
                                                    ],
                                                    "type": "collection",
                                                    "label": "Skill"
                                                },
                                                "type": "array",
                                                "label": "Skills",
                                                "validate": {
                                                    "maxItems": 8
                                                }
                                            }
                                        ],
                                        "type": "collection",
                                        "label": "Container"
                                    },
                                    {
                                        "name": "system",
                                        "type": "text",
                                        "label": "System Prompt"
                                    },
                                    {
                                        "name": "metadata",
                                        "spec": [
                                            {
                                                "name": "user_id",
                                                "type": "text",
                                                "label": "User ID"
                                            }
                                        ],
                                        "type": "collection",
                                        "label": "Metadata"
                                    },
                                    {
                                        "name": "stop_sequences",
                                        "spec": {
                                            "name": "value",
                                            "type": "text",
                                            "label": "Stop Sequence"
                                        },
                                        "type": "array",
                                        "label": "Stop Sequences"
                                    },
                                    {
                                        "name": "temperature",
                                        "type": "number",
                                        "label": "Temperature",
                                        "validate": {
                                            "max": 1,
                                            "min": 0
                                        }
                                    },
                                    {
                                        "name": "top_p",
                                        "type": "number",
                                        "label": "Top P",
                                        "validate": {
                                            "max": 1
                                        }
                                    },
                                    {
                                        "name": "top_k",
                                        "type": "uinteger",
                                        "label": "Top K"
                                    },
                                    {
                                        "name": "outputFormat",
                                        "spec": [
                                            {
                                                "name": "type",
                                                "type": "select",
                                                "label": "Type",
                                                "validate": {
                                                    "enum": [
                                                        "text",
                                                        "json_schema"
                                                    ]
                                                }
                                            }
                                        ],
                                        "type": "collection",
                                        "label": "Output Format"
                                    },
                                    {
                                        "name": "additionalParameters",
                                        "spec": {
                                            "name": "value",
                                            "spec": [
                                                {
                                                    "name": "key",
                                                    "type": "text",
                                                    "label": "Parameter Name",
                                                    "required": true
                                                },
                                                {
                                                    "name": "type",
                                                    "type": "select",
                                                    "label": "Input Type",
                                                    "options": [
                                                        {
                                                            "label": "Text",
                                                            "value": "text",
                                                            "nested": [
                                                                {
                                                                    "name": "value",
                                                                    "type": "text",
                                                                    "label": "Parameter Value"
                                                                }
                                                            ],
                                                            "default": true
                                                        },
                                                        {
                                                            "label": "Number",
                                                            "value": "number",
                                                            "nested": [
                                                                {
                                                                    "name": "value",
                                                                    "type": "number",
                                                                    "label": "Parameter Value"
                                                                }
                                                            ]
                                                        },
                                                        {
                                                            "label": "Boolean",
                                                            "value": "boolean",
                                                            "nested": [
                                                                {
                                                                    "name": "value",
                                                                    "type": "boolean",
                                                                    "label": "Parameter Value"
                                                                }
                                                            ]
                                                        },
                                                        {
                                                            "label": "Date",
                                                            "value": "date",
                                                            "nested": [
                                                                {
                                                                    "name": "value",
                                                                    "type": "date",
                                                                    "label": "Parameter Value"
                                                                }
                                                            ]
                                                        },
                                                        {
                                                            "label": "Tools",
                                                            "value": "tools",
                                                            "nested": [
                                                                {
                                                                    "name": "value",
                                                                    "spec": {
                                                                        "type": "text",
                                                                        "label": "Parameter Value"
                                                                    },
                                                                    "type": "array",
                                                                    "label": "Parameter Values",
                                                                    "labels": {
                                                                        "add": "Add parameter value"
                                                                    }
                                                                }
                                                            ]
                                                        },
                                                        {
                                                            "label": "Any",
                                                            "value": "any",
                                                            "nested": [
                                                                {
                                                                    "name": "value",
                                                                    "type": "any",
                                                                    "label": "Parameter Value"
                                                                }
                                                            ]
                                                        }
                                                    ]
                                                }
                                            ],
                                            "type": "collection",
                                            "label": "Input Parameter"
                                        },
                                        "type": "array",
                                        "label": "Other Input Parameters"
                                    }
                                ],
                                "interface": [
                                    {
                                        "name": "files",
                                        "spec": [
                                            {
                                                "name": "fileName",
                                                "type": "filename",
                                                "label": "File Name",
                                                "semantic": "file:name"
                                            },
                                            {
                                                "name": "fileData",
                                                "type": "buffer",
                                                "label": "File Data",
                                                "semantic": "file:data"
                                            },
                                            {
                                                "name": "fileId",
                                                "type": "text",
                                                "label": "File ID"
                                            }
                                        ],
                                        "type": "array",
                                        "label": "Files"
                                    },
                                    {
                                        "name": "textResponse",
                                        "type": "text",
                                        "label": "Text Response"
                                    },
                                    {
                                        "name": "id",
                                        "type": "text",
                                        "label": "Unique ID"
                                    },
                                    {
                                        "name": "type",
                                        "type": "text",
                                        "label": "Type"
                                    },
                                    {
                                        "name": "role",
                                        "type": "text",
                                        "label": "Role"
                                    },
                                    {
                                        "name": "content",
                                        "spec": {
                                            "spec": [
                                                {
                                                    "name": "type",
                                                    "type": "text",
                                                    "label": "Type"
                                                },
                                                {
                                                    "name": "thinking",
                                                    "type": "text",
                                                    "label": "Thinking"
                                                },
                                                {
                                                    "name": "signature",
                                                    "type": "text",
                                                    "label": "Signature"
                                                },
                                                {
                                                    "name": "text",
                                                    "type": "text",
                                                    "label": "Text"
                                                },
                                                {
                                                    "name": "id",
                                                    "type": "text",
                                                    "label": "ID"
                                                },
                                                {
                                                    "name": "name",
                                                    "type": "text",
                                                    "label": "Name"
                                                },
                                                {
                                                    "name": "input",
                                                    "spec": [],
                                                    "type": "collection",
                                                    "label": "Input"
                                                }
                                            ],
                                            "type": "collection"
                                        },
                                        "type": "array",
                                        "label": "Content"
                                    },
                                    {
                                        "name": "model",
                                        "type": "text",
                                        "label": "Model"
                                    },
                                    {
                                        "name": "container",
                                        "spec": [
                                            {
                                                "name": "id",
                                                "type": "text",
                                                "label": "ID"
                                            },
                                            {
                                                "name": "expires_at",
                                                "type": "date",
                                                "label": "Expires at"
                                            },
                                            {
                                                "name": "skills",
                                                "spec": {
                                                    "spec": [
                                                        {
                                                            "name": "type",
                                                            "type": "text",
                                                            "label": "Type"
                                                        },
                                                        {
                                                            "name": "skill_id",
                                                            "type": "text",
                                                            "label": "Skill ID"
                                                        },
                                                        {
                                                            "name": "version",
                                                            "type": "text",
                                                            "label": "Version"
                                                        }
                                                    ],
                                                    "type": "collection"
                                                },
                                                "type": "array",
                                                "label": "Skills"
                                            }
                                        ],
                                        "type": "collection",
                                        "label": "Container"
                                    },
                                    {
                                        "name": "stop_reason",
                                        "type": "text",
                                        "label": "Stop Reason"
                                    },
                                    {
                                        "name": "stop_sequence",
                                        "type": "text",
                                        "label": "Stop Sequence"
                                    },
                                    {
                                        "name": "usage",
                                        "spec": [
                                            {
                                                "name": "input_tokens",
                                                "type": "uinteger",
                                                "label": "Input Tokens"
                                            },
                                            {
                                                "name": "cache_creation_input_tokens",
                                                "type": "number",
                                                "label": "Cache Creation Input Tokens"
                                            },
                                            {
                                                "name": "cache_read_input_tokens",
                                                "type": "number",
                                                "label": "Cache Read Input Tokens"
                                            },
                                            {
                                                "name": "output_tokens",
                                                "type": "uinteger",
                                                "label": "Output Tokens"
                                            },
                                            {
                                                "name": "cache_creation",
                                                "spec": [
                                                    {
                                                        "name": "ephemeral_5m_input_tokens",
                                                        "type": "number",
                                                        "label": "Ephemeral 5m Input Tokens"
                                                    },
                                                    {
                                                        "name": "ephemeral_1h_input_tokens",
                                                        "type": "number",
                                                        "label": "Ephemeral 1h Input Tokens"
                                                    }
                                                ],
                                                "type": "collection",
                                                "label": "Cache Creation"
                                            },
                                            {
                                                "name": "service_tier",
                                                "type": "text",
                                                "label": "Service Tier"
                                            },
                                            {
                                                "name": "server_tool_use",
                                                "spec": [
                                                    {
                                                        "name": "web_search_requests",
                                                        "type": "number",
                                                        "label": "Web Search Requests"
                                                    }
                                                ],
                                                "type": "collection",
                                                "label": "Server Tool  Use"
                                            }
                                        ],
                                        "type": "collection",
                                        "label": "Usage"
                                    },
                                    {
                                        "name": "tools",
                                        "spec": [],
                                        "type": "collection",
                                        "label": "Tools"
                                    }
                                ]
                            }
                        },
                        {
                            "id": 112,
                            "module": "builtin:Resume",
                            "version": 1,
                            "parameters": {},
                            "mapper": {
                                "id": "",
                                "role": "",
                                "type": "",
                                "model": "",
                                "tools": {},
                                "usage": {
                                    "input_tokens": "",
                                    "output_tokens": ""
                                },
                                "content": [],
                                "stop_reason": "",
                                "textResponse": "{{115.textResponse}}",
                                "stop_sequence": ""
                            },
                            "metadata": {
                                "designer": {
                                    "x": 1543,
                                    "y": 1157
                                },
                                "restore": {},
                                "expect": [
                                    {
                                        "name": "textResponse",
                                        "type": "text",
                                        "label": "Text Response"
                                    },
                                    {
                                        "name": "id",
                                        "type": "text",
                                        "label": "Unique ID"
                                    },
                                    {
                                        "name": "type",
                                        "type": "text",
                                        "label": "Type"
                                    },
                                    {
                                        "name": "role",
                                        "type": "text",
                                        "label": "Role"
                                    },
                                    {
                                        "name": "content",
                                        "spec": {
                                            "name": "value",
                                            "spec": [
                                                {
                                                    "name": "type",
                                                    "type": "text",
                                                    "label": "Type"
                                                },
                                                {
                                                    "name": "text",
                                                    "type": "text",
                                                    "label": "Text"
                                                },
                                                {
                                                    "name": "id",
                                                    "type": "text",
                                                    "label": "ID"
                                                },
                                                {
                                                    "name": "name",
                                                    "type": "text",
                                                    "label": "Name"
                                                },
                                                {
                                                    "name": "input",
                                                    "spec": [],
                                                    "type": "collection",
                                                    "label": "Input"
                                                }
                                            ],
                                            "type": "collection"
                                        },
                                        "type": "array",
                                        "label": "Content"
                                    },
                                    {
                                        "name": "model",
                                        "type": "text",
                                        "label": "Model"
                                    },
                                    {
                                        "name": "stop_reason",
                                        "type": "text",
                                        "label": "Stop Reason"
                                    },
                                    {
                                        "name": "stop_sequence",
                                        "type": "text",
                                        "label": "Stop Sequence"
                                    },
                                    {
                                        "name": "usage",
                                        "spec": [
                                            {
                                                "name": "input_tokens",
                                                "type": "uinteger",
                                                "label": "Input Tokens"
                                            },
                                            {
                                                "name": "output_tokens",
                                                "type": "uinteger",
                                                "label": "Output Tokens"
                                            }
                                        ],
                                        "type": "collection",
                                        "label": "Usage"
                                    },
                                    {
                                        "name": "tools",
                                        "spec": [],
                                        "type": "collection",
                                        "label": "Tools"
                                    }
                                ]
                            }
                        }
                    ]
                },
                {
                    "id": 71,
                    "module": "json:ParseJSON",
                    "version": 1,
                    "parameters": {
                        "type": ""
                    },
                    "filter": {
                        "name": "#2 Warunek sprawdzający FV",
                        "conditions": [
                            [
                                {
                                    "a": "{{70.textResponse}}",
                                    "b": "NOT_INVOICE",
                                    "o": "text:notcontain:ci"
                                }
                            ]
                        ]
                    },
                    "mapper": {
                        "json": "{{70.textResponse}}"
                    },
                    "metadata": {
                        "designer": {
                            "x": 1038,
                            "y": 381
                        },
                        "restore": {
                            "parameters": {
                                "type": {
                                    "label": "Choose a data structure"
                                }
                            }
                        },
                        "parameters": [
                            {
                                "name": "type",
                                "type": "udt",
                                "label": "Data structure"
                            }
                        ],
                        "expect": [
                            {
                                "name": "json",
                                "type": "text",
                                "label": "JSON string",
                                "required": true
                            }
                        ]
                    },
                    "onerror": [
                        {
                            "id": 105,
                            "module": "openai-gpt-3:CreateCompletion",
                            "version": 1,
                            "parameters": {
                                "__IMTCONN__": 4017109
                            },
                            "mapper": {
                                "model": "gpt-4o",
                                "top_p": "1",
                                "select": "chat",
                                "messages": [
                                    {
                                        "role": "system",
                                        "content": "# Instrukcja czyszczenia i naprawy JSON\n\nPobierz zawartość ze zmiennej <output>{output}</output> i wykonaj następujące operacje:\n\n1. Usuń wszystkie znaki i tekst przed pierwszym znakiem { oraz po ostatnim znaku }\n2. Usuń wszystkie znaczniki kodu (np. ```json, ```)\n3. Napraw strukturę JSON:\n   - Dodaj brakujące przecinki między elementami\n   - Dodaj brakujące cudzysłowy przy kluczach\n   - Usuń nadmiarowe przecinki\n   - Zamień pojedyncze cudzysłowy na podwójne\n   - Popraw format wartości: null zamiast \"null\", liczby bez cudzysłowów\n   - Usuń komentarze (wszystko po // lub między /* */)\n\nZwróć TYLKO poprawny JSON bez żadnych dodatkowych znaków, komentarzy czy formatowania.\n\nPrzykłady:\n\n1. Przykład - Brakujące przecinki i niepoprawne cudzysłowy:\nWejście:\n```json\n{\n  name: \"John\"\n  age: 25\n  city: 'New York'\n  active: true,\n}\n```\n\nWyjście:\n{\n  \"name\": \"John\",\n  \"age\": 25,\n  \"city\": \"New York\",\n  \"active\": true\n}\n\n2. Przykład - Tekst przed i po JSON:\nWejście:\nOto wynik analizy:\n{items: [\n    {id: 1, value: \"test\"},\n    {id: 2, value: 'example'}\n]}\nKoniec wyniku.\n\nWyjście:\n{\n  \"items\": [\n    {\n      \"id\": 1,\n      \"value\": \"test\"\n    },\n    {\n      \"id\": 2,\n      \"value\": \"example\"\n    }\n  ]\n}\n\n3. Przykład - Niepoprawne wartości null i liczby:\nWejście:\n{\n  \"price\": \"123.45\",\n  \"quantity\": \"null\",\n  \"inStock\": \"false\",\n  \"description\": null,\n  /* To jest komentarz */\n  \"tags\": [\"tag1\" \"tag2\"]  // lista tagów\n}\n\nWyjście:\n{\n  \"price\": 123.45,\n  \"quantity\": null,\n  \"inStock\": false,\n  \"description\": null,\n  \"tags\": [\"tag1\", \"tag2\"]\n}\n\n4. Przykład - Zagnieżdżona struktura z błędami:\nWejście:\n```\n{\n  user: {\n    personalInfo: {\n      firstName: 'Jan',\n      lastName: \"Kowalski\"\n    }\n    contact: {\n      email: \"jan@example.com\"\n      phone: '123-456-789',\n    },\n  },\n  orders: [\n    {id: '1', status: \"completed\"},\n    {id: '2', status: \"pending\"},\n  ]\n}\n```\n\nWyjście:\n{\n  \"user\": {\n    \"personalInfo\": {\n      \"firstName\": \"Jan\",\n      \"lastName\": \"Kowalski\"\n    },\n    \"contact\": {\n      \"email\": \"jan@example.com\",\n      \"phone\": \"123-456-789\"\n    }\n  },\n  \"orders\": [\n    {\n      \"id\": \"1\",\n      \"status\": \"completed\"\n    },\n    {\n      \"id\": \"2\",\n      \"status\": \"pending\"\n    }\n  ]\n}\n\n5. Przykład - Mieszane problemy:\nWejście:\nResult:\n```json\n{\n  productId: \"ABC-123\",\n  price: \"99.99\",\n  features: ['feature1' 'feature2', 'feature3',], //lista funkcji\n  metadata: {\n    created: \"2024-01-15\"\n    updated: null,\n    isValid: \"true\"\n  },\n}"
                                    },
                                    {
                                        "role": "user",
                                        "content": "**Dane do poprawy:**\n<output>{{70.textResponse}}</output>",
                                        "imageDetail": "auto"
                                    }
                                ],
                                "max_tokens": "2048",
                                "temperature": "0.3",
                                "n_completions": "1",
                                "response_format": "text"
                            },
                            "metadata": {
                                "designer": {
                                    "x": 1208,
                                    "y": 726,
                                    "name": "🛠️ Naprawa JSON"
                                },
                                "restore": {
                                    "expect": {
                                        "stop": {
                                            "mode": "chose"
                                        },
                                        "model": {
                                            "mode": "chose",
                                            "label": "gpt-4o (system)"
                                        },
                                        "select": {
                                            "label": "Create a Chat Completion (GPT and o1 models)"
                                        },
                                        "messages": {
                                            "mode": "chose",
                                            "items": [
                                                {
                                                    "role": {
                                                        "mode": "chose",
                                                        "label": "Developer / System"
                                                    }
                                                },
                                                {
                                                    "role": {
                                                        "mode": "chose",
                                                        "label": "User"
                                                    },
                                                    "imageDetail": {
                                                        "mode": "chose",
                                                        "label": "Auto"
                                                    },
                                                    "imageInputType": {
                                                        "mode": "chose",
                                                        "label": "Empty",
                                                        "collapsed": true
                                                    }
                                                }
                                            ]
                                        },
                                        "logit_bias": {
                                            "mode": "chose"
                                        },
                                        "response_format": {
                                            "mode": "chose",
                                            "label": "Text"
                                        },
                                        "additionalParameters": {
                                            "mode": "chose"
                                        }
                                    },
                                    "parameters": {
                                        "__IMTCONN__": {
                                            "data": {
                                                "scoped": "true",
                                                "connection": "openai-gpt-3"
                                            },
                                            "label": "(OpenAI) aktywny"
                                        }
                                    }
                                },
                                "parameters": [
                                    {
                                        "name": "__IMTCONN__",
                                        "type": "account:openai-gpt-3",
                                        "label": "Connection",
                                        "required": true
                                    }
                                ],
                                "expect": [
                                    {
                                        "name": "select",
                                        "type": "select",
                                        "label": "Select Method",
                                        "required": true,
                                        "validate": {
                                            "enum": [
                                                "chat",
                                                "prompt"
                                            ]
                                        }
                                    },
                                    {
                                        "name": "temperature",
                                        "type": "number",
                                        "label": "Temperature",
                                        "validate": {
                                            "max": 2,
                                            "min": 0
                                        }
                                    },
                                    {
                                        "name": "top_p",
                                        "type": "number",
                                        "label": "Top P",
                                        "validate": {
                                            "max": 1,
                                            "min": 0
                                        }
                                    },
                                    {
                                        "name": "n_completions",
                                        "type": "number",
                                        "label": "Number"
                                    },
                                    {
                                        "name": "frequency_penalty",
                                        "type": "number",
                                        "label": "Frequency Penalty",
                                        "validate": {
                                            "max": 2,
                                            "min": -2
                                        }
                                    },
                                    {
                                        "name": "presence_penalty",
                                        "type": "number",
                                        "label": "Presence Penalty",
                                        "validate": {
                                            "max": 2,
                                            "min": -2
                                        }
                                    },
                                    {
                                        "name": "logit_bias",
                                        "spec": {
                                            "name": "value",
                                            "spec": [
                                                {
                                                    "name": "token",
                                                    "type": "text",
                                                    "label": "Token ID",
                                                    "required": true
                                                },
                                                {
                                                    "name": "probability",
                                                    "type": "number",
                                                    "label": "Probability",
                                                    "required": true,
                                                    "validate": {
                                                        "max": 100,
                                                        "min": -100
                                                    }
                                                }
                                            ],
                                            "type": "collection",
                                            "label": "Token Probability"
                                        },
                                        "type": "array",
                                        "label": "Token Probability"
                                    },
                                    {
                                        "name": "seed",
                                        "type": "integer",
                                        "label": "Seed"
                                    },
                                    {
                                        "name": "stop",
                                        "spec": {
                                            "name": "value",
                                            "type": "text",
                                            "label": "Stop Sequence"
                                        },
                                        "type": "array",
                                        "label": "Stop Sequences",
                                        "validate": {
                                            "maxItems": 4
                                        }
                                    },
                                    {
                                        "name": "additionalParameters",
                                        "spec": {
                                            "name": "value",
                                            "spec": [
                                                {
                                                    "name": "key",
                                                    "type": "text",
                                                    "label": "Parameter Name",
                                                    "required": true
                                                },
                                                {
                                                    "name": "type",
                                                    "type": "select",
                                                    "label": "Input Type",
                                                    "options": [
                                                        {
                                                            "label": "Text",
                                                            "value": "text",
                                                            "nested": [
                                                                {
                                                                    "name": "value",
                                                                    "type": "text",
                                                                    "label": "Parameter Value"
                                                                }
                                                            ],
                                                            "default": true
                                                        },
                                                        {
                                                            "label": "Number",
                                                            "value": "number",
                                                            "nested": [
                                                                {
                                                                    "name": "value",
                                                                    "type": "number",
                                                                    "label": "Parameter Value"
                                                                }
                                                            ]
                                                        },
                                                        {
                                                            "label": "Boolean",
                                                            "value": "boolean",
                                                            "nested": [
                                                                {
                                                                    "name": "value",
                                                                    "type": "boolean",
                                                                    "label": "Parameter Value"
                                                                }
                                                            ]
                                                        },
                                                        {
                                                            "label": "Date",
                                                            "value": "date",
                                                            "nested": [
                                                                {
                                                                    "name": "value",
                                                                    "type": "date",
                                                                    "label": "Parameter Value"
                                                                }
                                                            ]
                                                        },
                                                        {
                                                            "label": "Any",
                                                            "value": "any",
                                                            "nested": [
                                                                {
                                                                    "name": "value",
                                                                    "type": "any",
                                                                    "label": "Parameter Value"
                                                                }
                                                            ]
                                                        }
                                                    ]
                                                }
                                            ],
                                            "type": "collection",
                                            "label": "Input Parameter"
                                        },
                                        "type": "array",
                                        "label": "Other Input Parameters"
                                    },
                                    {
                                        "name": "model",
                                        "type": "select",
                                        "label": "Model",
                                        "required": true
                                    },
                                    {
                                        "name": "max_tokens",
                                        "type": "uinteger",
                                        "label": "Max Completion Tokens"
                                    },
                                    {
                                        "name": "messages",
                                        "spec": {
                                            "name": "value",
                                            "spec": [
                                                {
                                                    "name": "role",
                                                    "type": "select",
                                                    "label": "Role",
                                                    "options": {
                                                        "store": [
                                                            {
                                                                "label": "User",
                                                                "value": "user",
                                                                "nested": [
                                                                    {
                                                                        "help": "Text content of the message on behalf of the selected __Role__.",
                                                                        "name": "content",
                                                                        "type": "text",
                                                                        "label": "Text Content"
                                                                    },
                                                                    {
                                                                        "name": "imageInputType",
                                                                        "type": "select",
                                                                        "label": "Image Input Type",
                                                                        "options": [
                                                                            {
                                                                                "label": "URL",
                                                                                "value": "url",
                                                                                "nested": [
                                                                                    {
                                                                                        "help": "Make sure to use a publicly accessible URL.\nYou can test if your image is publicly accessible by opening the link in an incognito tab.",
                                                                                        "name": "imageUrl",
                                                                                        "type": "url",
                                                                                        "label": "Image URL"
                                                                                    }
                                                                                ]
                                                                            },
                                                                            {
                                                                                "label": "Image File",
                                                                                "value": "file",
                                                                                "nested": [
                                                                                    {
                                                                                        "name": "imageFile",
                                                                                        "spec": [
                                                                                            {
                                                                                                "help": "Accepted extensions: `.jpg`, `.jpeg`, `.png`, `.webp` and `.gif`.",
                                                                                                "name": "imageFilename",
                                                                                                "type": "filename",
                                                                                                "label": "Image Filename",
                                                                                                "semantic": "file:name",
                                                                                                "extension": [
                                                                                                    "jpg",
                                                                                                    "jpeg",
                                                                                                    "png",
                                                                                                    "webp",
                                                                                                    "gif"
                                                                                                ]
                                                                                            },
                                                                                            {
                                                                                                "name": "imageData",
                                                                                                "type": "buffer",
                                                                                                "label": "Image Data",
                                                                                                "semantic": "file:data"
                                                                                            }
                                                                                        ],
                                                                                        "type": "collection",
                                                                                        "label": "Image"
                                                                                    }
                                                                                ]
                                                                            }
                                                                        ],
                                                                        "mappable": false
                                                                    },
                                                                    {
                                                                        "help": "Recommended value: `Auto`",
                                                                        "name": "imageDetail",
                                                                        "type": "select",
                                                                        "label": "Image Detail",
                                                                        "options": [
                                                                            {
                                                                                "label": "Auto",
                                                                                "value": "auto",
                                                                                "default": true
                                                                            },
                                                                            {
                                                                                "label": "High",
                                                                                "value": "high"
                                                                            },
                                                                            {
                                                                                "label": "Low",
                                                                                "value": "low"
                                                                            }
                                                                        ]
                                                                    }
                                                                ]
                                                            },
                                                            {
                                                                "label": "Assistant",
                                                                "value": "assistant",
                                                                "nested": [
                                                                    {
                                                                        "help": "Text content of the message on behalf of the selected __Role__.",
                                                                        "name": "content",
                                                                        "type": "text",
                                                                        "label": "Text Content"
                                                                    }
                                                                ]
                                                            },
                                                            {
                                                                "label": "Developer / System",
                                                                "value": "system",
                                                                "nested": [
                                                                    {
                                                                        "help": "Text content of the message on behalf of the selected __Role__.",
                                                                        "name": "content",
                                                                        "type": "text",
                                                                        "label": "Text Content"
                                                                    }
                                                                ]
                                                            }
                                                        ]
                                                    },
                                                    "required": true
                                                }
                                            ],
                                            "type": "collection",
                                            "label": "Message"
                                        },
                                        "type": "array",
                                        "label": "Messages",
                                        "required": true
                                    },
                                    {
                                        "name": "response_format",
                                        "type": "select",
                                        "label": "Response Format",
                                        "validate": {
                                            "enum": [
                                                "text",
                                                "json_object"
                                            ]
                                        }
                                    },
                                    {
                                        "name": "prediction",
                                        "type": "text",
                                        "label": "Predicted Outputs"
                                    }
                                ],
                                "interface": [
                                    {
                                        "name": "result",
                                        "type": "any",
                                        "label": "Result"
                                    },
                                    {
                                        "name": "id",
                                        "type": "text",
                                        "label": "ID"
                                    },
                                    {
                                        "name": "object",
                                        "type": "text",
                                        "label": "Object"
                                    },
                                    {
                                        "name": "created",
                                        "type": "date",
                                        "label": "Created"
                                    },
                                    {
                                        "name": "model",
                                        "type": "text",
                                        "label": "Model"
                                    },
                                    {
                                        "name": "choices",
                                        "spec": [
                                            {
                                                "name": "text",
                                                "type": "text",
                                                "label": "Text"
                                            },
                                            {
                                                "name": "index",
                                                "type": "number",
                                                "label": "Index"
                                            },
                                            {
                                                "name": "logprobs",
                                                "type": "text",
                                                "label": "Log Probs"
                                            },
                                            {
                                                "name": "finish_reason",
                                                "type": "text",
                                                "label": "Finish Reason"
                                            },
                                            {
                                                "name": "message",
                                                "spec": [
                                                    {
                                                        "name": "role",
                                                        "type": "text",
                                                        "label": "Role"
                                                    },
                                                    {
                                                        "name": "content",
                                                        "type": "text",
                                                        "label": "Content"
                                                    },
                                                    {
                                                        "name": "refusal",
                                                        "type": "text",
                                                        "label": "Refusal"
                                                    },
                                                    {
                                                        "name": "annotations",
                                                        "spec": [
                                                            {
                                                                "name": "type",
                                                                "type": "text",
                                                                "label": "Type"
                                                            },
                                                            {
                                                                "name": "url_citation",
                                                                "spec": [
                                                                    {
                                                                        "name": "end_index",
                                                                        "type": "number",
                                                                        "label": "End Index"
                                                                    },
                                                                    {
                                                                        "name": "start_index",
                                                                        "type": "number",
                                                                        "label": "Start Index"
                                                                    },
                                                                    {
                                                                        "name": "title",
                                                                        "type": "text",
                                                                        "label": "Title"
                                                                    },
                                                                    {
                                                                        "name": "url",
                                                                        "type": "text",
                                                                        "label": "URL"
                                                                    }
                                                                ],
                                                                "type": "collection",
                                                                "label": "URL Citation"
                                                            }
                                                        ],
                                                        "type": "array",
                                                        "label": "Annotations"
                                                    }
                                                ],
                                                "type": "collection",
                                                "label": "Message"
                                            }
                                        ],
                                        "type": "array",
                                        "label": "Choices"
                                    },
                                    {
                                        "name": "usage",
                                        "spec": [
                                            {
                                                "name": "prompt_tokens",
                                                "type": "number",
                                                "label": "Prompt Tokens"
                                            },
                                            {
                                                "name": "completion_tokens",
                                                "type": "text",
                                                "label": "Completion Tokens"
                                            },
                                            {
                                                "name": "total_tokens",
                                                "type": "number",
                                                "label": "Total Tokens"
                                            },
                                            {
                                                "name": "prompt_tokens_details",
                                                "spec": [
                                                    {
                                                        "name": "cached_tokens",
                                                        "type": "uinteger",
                                                        "label": "Cached Tokens"
                                                    },
                                                    {
                                                        "name": "text_tokens",
                                                        "type": "uinteger",
                                                        "label": "Text Tokens"
                                                    },
                                                    {
                                                        "name": "image_tokens",
                                                        "type": "uinteger",
                                                        "label": "Image Tokens"
                                                    },
                                                    {
                                                        "name": "audio_tokens",
                                                        "type": "uinteger",
                                                        "label": "Audio Tokens"
                                                    }
                                                ],
                                                "type": "collection",
                                                "label": "Prompt Tokens Details"
                                            },
                                            {
                                                "name": "completion_tokens_details",
                                                "spec": [
                                                    {
                                                        "name": "reasoning_tokens",
                                                        "type": "uinteger",
                                                        "label": "Reasoning Tokens"
                                                    },
                                                    {
                                                        "name": "text_tokens",
                                                        "type": "uinteger",
                                                        "label": "Text Tokens"
                                                    },
                                                    {
                                                        "name": "audio_tokens",
                                                        "type": "uinteger",
                                                        "label": "Audio Tokens"
                                                    },
                                                    {
                                                        "name": "accepted_prediction_tokens",
                                                        "type": "uinteger",
                                                        "label": "Accepted Prediction Tokens"
                                                    },
                                                    {
                                                        "name": "rejected_prediction_tokens",
                                                        "type": "uinteger",
                                                        "label": "Rejected Prediction Tokens"
                                                    }
                                                ],
                                                "type": "collection",
                                                "label": "Completion Tokens Details"
                                            }
                                        ],
                                        "type": "collection",
                                        "label": "Usage"
                                    },
                                    {
                                        "name": "service_tier",
                                        "type": "text",
                                        "label": "Service Tier"
                                    },
                                    {
                                        "name": "system_fingerprint",
                                        "type": "text",
                                        "label": "System Fingerprint"
                                    }
                                ],
                                "advanced": true
                            }
                        },
                        {
                            "id": 106,
                            "module": "json:ParseJSON",
                            "version": 1,
                            "parameters": {
                                "type": ""
                            },
                            "mapper": {
                                "json": "{{105.result}}"
                            },
                            "metadata": {
                                "designer": {
                                    "x": 1516,
                                    "y": 748
                                },
                                "restore": {
                                    "parameters": {
                                        "type": {
                                            "label": "Choose a data structure"
                                        }
                                    }
                                },
                                "parameters": [
                                    {
                                        "name": "type",
                                        "type": "udt",
                                        "label": "Data structure"
                                    }
                                ],
                                "expect": [
                                    {
                                        "name": "json",
                                        "type": "text",
                                        "label": "JSON string",
                                        "required": true
                                    }
                                ]
                            }
                        },
                        {
                            "id": 108,
                            "module": "builtin:Resume",
                            "version": 1,
                            "parameters": {},
                            "mapper": {
                                "supplier": "{{71.supplier}}",
                                "sale date": "{{71.`sale date`}}",
                                "issue_date": "{{71.issue_date}}",
                                "net_amount": "{{71.net_amount}}",
                                "vat amount": "{{71.`vat amount`}}",
                                "seller's ID": "{{71.`seller's ID`}}",
                                "gross_amount": "{{71.gross_amount}}",
                                "invoice_number": "{{71.invoice_number}}"
                            },
                            "metadata": {
                                "designer": {
                                    "x": 1807,
                                    "y": 846
                                },
                                "restore": {},
                                "expect": [
                                    {
                                        "name": "supplier",
                                        "type": "text"
                                    },
                                    {
                                        "name": "sale date",
                                        "type": "text"
                                    },
                                    {
                                        "name": "issue_date",
                                        "type": "text"
                                    },
                                    {
                                        "name": "net_amount",
                                        "type": "number"
                                    },
                                    {
                                        "name": "vat amount",
                                        "type": "number"
                                    },
                                    {
                                        "name": "seller's ID",
                                        "type": "number"
                                    },
                                    {
                                        "name": "gross_amount",
                                        "type": "number"
                                    },
                                    {
                                        "name": "invoice_number",
                                        "type": "text"
                                    }
                                ]
                            }
                        }
                    ]
                },
                {
                    "id": 162,
                    "module": "airtable:ActionSearchRecords",
                    "version": 3,
                    "parameters": {
                        "__IMTCONN__": 8854084
                    },
                    "filter": {
                        "name": "",
                        "conditions": [
                            [
                                {
                                    "a": "{{`71`}}",
                                    "b": "```json",
                                    "o": "array:notcontain"
                                }
                            ]
                        ]
                    },
                    "mapper": {
                        "base": "appbLQjOUm58ckRwu",
                        "table": "tbl2jnB2TinJgSh6b",
                        "formula": "AND({Numer FV} = '{{71.invoice_number}}', {Dostawca} = '{{71.supplier}}')",
                        "maxRecords": "100",
                        "useColumnId": false
                    },
                    "metadata": {
                        "designer": {
                            "x": 1435,
                            "y": 341,
                            "name": "Sprawdź duplikaty"
                        },
                        "restore": {
                            "expect": {
                                "base": {
                                    "mode": "chose",
                                    "label": "Automatyzacja faktur - Gotowy Scenariusz"
                                },
                                "sort": {
                                    "mode": "chose"
                                },
                                "view": {
                                    "mode": "chose"
                                },
                                "table": {
                                    "mode": "chose",
                                    "label": "Zestawienie"
                                },
                                "fields": {
                                    "mode": "chose"
                                },
                                "useColumnId": {
                                    "mode": "chose"
                                }
                            },
                            "parameters": {
                                "__IMTCONN__": {
                                    "data": {
                                        "scoped": "true",
                                        "connection": "airtable3"
                                    },
                                    "label": "My Airtable OAuth connection (User ID: usrcV7XR8wZp1djp3)"
                                }
                            }
                        },
                        "parameters": [
                            {
                                "name": "__IMTCONN__",
                                "type": "account:airtable3,airtable2",
                                "label": "Connection",
                                "required": true
                            }
                        ],
                        "expect": [
                            {
                                "name": "base",
                                "type": "select",
                                "label": "Base",
                                "required": true
                            },
                            {
                                "name": "useColumnId",
                                "type": "boolean",
                                "label": "Use Column ID",
                                "required": true
                            },
                            {
                                "name": "table",
                                "type": "select",
                                "label": "Table",
                                "required": true
                            },
                            {
                                "name": "formula",
                                "type": "text",
                                "label": "Formula"
                            },
                            {
                                "name": "maxRecords",
                                "type": "integer",
                                "label": "Limit"
                            },
                            {
                                "name": "sort",
                                "spec": [
                                    {
                                        "name": "field",
                                        "type": "select",
                                        "label": "Field",
                                        "dynamic": true,
                                        "options": []
                                    },
                                    {
                                        "name": "direction",
                                        "type": "select",
                                        "label": "Direction",
                                        "options": [
                                            {
                                                "label": "Descending",
                                                "value": "desc"
                                            },
                                            {
                                                "label": "Ascending",
                                                "value": "asc"
                                            }
                                        ]
                                    }
                                ],
                                "type": "array",
                                "label": "Sort"
                            },
                            {
                                "name": "view",
                                "type": "select",
                                "label": "View"
                            },
                            {
                                "name": "fields",
                                "type": "select",
                                "label": "Output Fields",
                                "multiple": true
                            }
                        ],
                        "interface": [
                            {
                                "name": "__IMTLENGTH__",
                                "type": "uinteger",
                                "label": "Total number of bundles"
                            },
                            {
                                "name": "__IMTINDEX__",
                                "type": "uinteger",
                                "label": "Bundle order position"
                            },
                            {
                                "name": "id",
                                "type": "text",
                                "label": "ID"
                            },
                            {
                                "name": "createdTime",
                                "type": "date",
                                "label": "Created Time"
                            },
                            {
                                "name": "Data pobrania",
                                "time": false,
                                "type": "date",
                                "label": "Data pobrania"
                            },
                            {
                                "name": "Dostawca",
                                "type": "text",
                                "label": "Dostawca"
                            },
                            {
                                "name": "Numer NIP",
                                "type": "text",
                                "label": "Numer NIP"
                            },
                            {
                                "name": "Numer FV",
                                "type": "text",
                                "label": "Numer FV"
                            },
                            {
                                "name": "Data wystawienia FV",
                                "time": false,
                                "type": "date",
                                "label": "Data wystawienia FV"
                            },
                            {
                                "name": "Data sprzedaży FV",
                                "time": false,
                                "type": "date",
                                "label": "Data sprzedaży FV"
                            },
                            {
                                "name": "Kwota netto",
                                "type": "number",
                                "label": "Kwota netto"
                            },
                            {
                                "name": "Kwota VAT",
                                "type": "number",
                                "label": "Kwota VAT"
                            },
                            {
                                "name": "Kwota brutto ",
                                "type": "number",
                                "label": "Kwota brutto "
                            },
                            {
                                "name": "KATEGORIA",
                                "type": "text",
                                "label": "KATEGORIA"
                            },
                            {
                                "name": "Krótki Opis",
                                "type": "text",
                                "label": "Krótki Opis"
                            },
                            {
                                "name": "Faktura",
                                "spec": [
                                    {
                                        "name": "id",
                                        "type": "text",
                                        "label": "ID"
                                    },
                                    {
                                        "name": "width",
                                        "type": "uinteger",
                                        "label": "Width"
                                    },
                                    {
                                        "name": "height",
                                        "type": "uinteger",
                                        "label": "Height"
                                    },
                                    {
                                        "name": "url",
                                        "type": "url",
                                        "label": "URL"
                                    },
                                    {
                                        "name": "filename",
                                        "type": "filename",
                                        "label": "File name"
                                    },
                                    {
                                        "name": "size",
                                        "type": "uinteger",
                                        "label": "Size"
                                    },
                                    {
                                        "name": "type",
                                        "type": "text",
                                        "label": "MIME type"
                                    },
                                    {
                                        "name": "thumbnails",
                                        "spec": [
                                            {
                                                "name": "small",
                                                "spec": [
                                                    {
                                                        "name": "url",
                                                        "type": "url",
                                                        "label": "URL"
                                                    },
                                                    {
                                                        "name": "width",
                                                        "type": "uinteger",
                                                        "label": "Width"
                                                    },
                                                    {
                                                        "name": "height",
                                                        "type": "uinteger",
                                                        "label": "Height"
                                                    }
                                                ],
                                                "type": "collection",
                                                "label": "Small"
                                            },
                                            {
                                                "name": "large",
                                                "spec": [
                                                    {
                                                        "name": "url",
                                                        "type": "url",
                                                        "label": "URL"
                                                    },
                                                    {
                                                        "name": "width",
                                                        "type": "uinteger",
                                                        "label": "Width"
                                                    },
                                                    {
                                                        "name": "height",
                                                        "type": "uinteger",
                                                        "label": "Height"
                                                    }
                                                ],
                                                "type": "collection",
                                                "label": "Large"
                                            },
                                            {
                                                "name": "full",
                                                "spec": [
                                                    {
                                                        "name": "url",
                                                        "type": "url",
                                                        "label": "URL"
                                                    },
                                                    {
                                                        "name": "width",
                                                        "type": "uinteger",
                                                        "label": "Width"
                                                    },
                                                    {
                                                        "name": "height",
                                                        "type": "uinteger",
                                                        "label": "Height"
                                                    }
                                                ],
                                                "type": "collection",
                                                "label": "Full"
                                            }
                                        ],
                                        "type": "collection",
                                        "label": "Thumbnails"
                                    }
                                ],
                                "type": "array",
                                "label": "Faktura"
                            },
                            {
                                "name": "Lokalizacja - GDrive",
                                "type": "text",
                                "label": "Lokalizacja - GDrive"
                            },
                            {
                                "name": "Data_importu",
                                "time": false,
                                "type": "date",
                                "label": "Data_importu"
                            },
                            {
                                "name": "ID_platnosci_Fakturownia",
                                "type": "text",
                                "label": "ID_platnosci_Fakturownia"
                            },
                            {
                                "name": "Miesiąc",
                                "type": "text",
                                "label": "Miesiąc"
                            },
                            {
                                "name": "ID 2",
                                "type": "number",
                                "label": "ID 2"
                            },
                            {
                                "name": "Pobrano go sheets",
                                "type": "boolean",
                                "label": "Pobrano go sheets"
                            }
                        ]
                    }
                },
                {
                    "id": 163,
                    "module": "builtin:BasicRouter",
                    "version": 1,
                    "mapper": null,
                    "metadata": {
                        "designer": {
                            "x": 1771,
                            "y": 249
                        }
                    },
                    "routes": [
                        {
                            "flow": [
                                {
                                    "id": 149,
                                    "module": "google-drive:recognizeAFileFolderPath",
                                    "version": 4,
                                    "parameters": {
                                        "__IMTCONN__": 8853889
                                    },
                                    "filter": {
                                        "name": "Brak duplikatu",
                                        "conditions": [
                                            [
                                                {
                                                    "a": "{{162.__IMTLENGTH__}}",
                                                    "b": "0",
                                                    "o": "text:equal"
                                                }
                                            ]
                                        ]
                                    },
                                    "mapper": {
                                        "path": "Projekty/Faktury/2025/{{substring(71.issue_date; 5; 7)}}"
                                    },
                                    "metadata": {
                                        "designer": {
                                            "x": 2071,
                                            "y": -51,
                                            "name": "Folder na ten miesiąc istnieje?"
                                        },
                                        "restore": {
                                            "parameters": {
                                                "__IMTCONN__": {
                                                    "data": {
                                                        "scoped": "true",
                                                        "connection": "google-restricted"
                                                    },
                                                    "label": "My Google Restricted connection (przemo.dud@gmail.com)"
                                                }
                                            }
                                        },
                                        "parameters": [
                                            {
                                                "name": "__IMTCONN__",
                                                "type": "account:google-restricted",
                                                "label": "Connection",
                                                "required": true
                                            }
                                        ],
                                        "expect": [
                                            {
                                                "type": "hidden"
                                            },
                                            {
                                                "name": "path",
                                                "type": "text",
                                                "label": "Folder Path",
                                                "required": true
                                            }
                                        ]
                                    },
                                    "onerror": [
                                        {
                                            "id": 150,
                                            "module": "google-drive:createAFolder",
                                            "version": 4,
                                            "parameters": {
                                                "__IMTCONN__": 8853889
                                            },
                                            "mapper": {
                                                "name": "{{substring(71.issue_date; 5; 7)}}",
                                                "shared": false,
                                                "folderId": "/1L0YIH0lNKUYbInjq0-tNXEHAWxrdmwFN/1oVJ1_nUgEoGji6bjXg7L8Yv_nh9q0ZZu/1ZzrR9xzUmltnlFxr8O_znr3PKMyK4SS_",
                                                "destination": "drive"
                                            },
                                            "metadata": {
                                                "designer": {
                                                    "x": 2100,
                                                    "y": 300,
                                                    "name": "Utwórz folder"
                                                },
                                                "restore": {
                                                    "expect": {
                                                        "folderId": {
                                                            "mode": "chose",
                                                            "path": [
                                                                "Projekty",
                                                                "Faktury",
                                                                "2025"
                                                            ]
                                                        },
                                                        "destination": {
                                                            "label": "My Drive"
                                                        }
                                                    },
                                                    "parameters": {
                                                        "__IMTCONN__": {
                                                            "data": {
                                                                "scoped": "true",
                                                                "connection": "google-restricted"
                                                            },
                                                            "label": "My Google Restricted connection (przemo.dud@gmail.com)"
                                                        }
                                                    }
                                                },
                                                "parameters": [
                                                    {
                                                        "name": "__IMTCONN__",
                                                        "type": "account:google-restricted",
                                                        "label": "Connection",
                                                        "required": true
                                                    }
                                                ],
                                                "expect": [
                                                    {
                                                        "type": "hidden"
                                                    },
                                                    {
                                                        "name": "destination",
                                                        "type": "select",
                                                        "label": "New Drive Location",
                                                        "required": true,
                                                        "validate": {
                                                            "enum": [
                                                                "drive",
                                                                "share",
                                                                "team"
                                                            ]
                                                        }
                                                    },
                                                    {
                                                        "name": "folderId",
                                                        "type": "folder",
                                                        "label": "New Folder Location",
                                                        "required": true
                                                    },
                                                    {
                                                        "name": "name",
                                                        "type": "text",
                                                        "label": "New Folder's Name"
                                                    },
                                                    {
                                                        "name": "shared",
                                                        "type": "boolean",
                                                        "label": "Share Folder",
                                                        "required": true
                                                    }
                                                ]
                                            }
                                        },
                                        {
                                            "id": 151,
                                            "module": "builtin:Resume",
                                            "version": 1,
                                            "parameters": {},
                                            "mapper": {
                                                "id": "{{150.id}}",
                                                "__IMTINDEX__": "",
                                                "__IMTLENGTH__": ""
                                            },
                                            "metadata": {
                                                "designer": {
                                                    "x": 2400,
                                                    "y": 300
                                                },
                                                "restore": {},
                                                "expect": [
                                                    {
                                                        "name": "__IMTLENGTH__",
                                                        "type": "uinteger",
                                                        "label": "Total number of bundles"
                                                    },
                                                    {
                                                        "name": "__IMTINDEX__",
                                                        "type": "uinteger",
                                                        "label": "Bundle order position"
                                                    },
                                                    {
                                                        "name": "id",
                                                        "type": "text",
                                                        "label": "Folder ID"
                                                    }
                                                ]
                                            }
                                        }
                                    ]
                                },
                                {
                                    "id": 152,
                                    "module": "google-drive:uploadAFile",
                                    "version": 4,
                                    "parameters": {
                                        "__IMTCONN__": 8853889
                                    },
                                    "mapper": {
                                        "data": "{{1.attachments[].data}}",
                                        "select": "value",
                                        "convert": false,
                                        "filename": "{{1.attachments[].fileName}}",
                                        "folderId": "{{149.id}}",
                                        "destination": "drive"
                                    },
                                    "metadata": {
                                        "designer": {
                                            "x": 2226,
                                            "y": -247,
                                            "name": "Zapisz FV na dysku"
                                        },
                                        "restore": {
                                            "expect": {
                                                "select": {
                                                    "label": "Select from the list"
                                                },
                                                "folderId": {
                                                    "mode": "edit",
                                                    "path": []
                                                },
                                                "destination": {
                                                    "label": "My Drive"
                                                }
                                            },
                                            "parameters": {
                                                "__IMTCONN__": {
                                                    "data": {
                                                        "scoped": "true",
                                                        "connection": "google-restricted"
                                                    },
                                                    "label": "My Google Restricted connection (przemo.dud@gmail.com)"
                                                }
                                            }
                                        },
                                        "parameters": [
                                            {
                                                "name": "__IMTCONN__",
                                                "type": "account:google-restricted",
                                                "label": "Connection",
                                                "required": true
                                            }
                                        ],
                                        "expect": [
                                            {
                                                "type": "hidden"
                                            },
                                            {
                                                "name": "select",
                                                "type": "select",
                                                "label": "Enter a Folder ID",
                                                "required": true,
                                                "validate": {
                                                    "enum": [
                                                        "map",
                                                        "value"
                                                    ]
                                                }
                                            },
                                            {
                                                "name": "title",
                                                "type": "text",
                                                "label": "New File Name"
                                            },
                                            {
                                                "name": "filename",
                                                "type": "text",
                                                "label": "File Name",
                                                "required": true
                                            },
                                            {
                                                "name": "data",
                                                "type": "buffer",
                                                "label": "Data",
                                                "required": true
                                            },
                                            {
                                                "name": "convert",
                                                "type": "boolean",
                                                "label": "Convert a File",
                                                "required": true
                                            },
                                            {
                                                "name": "destination",
                                                "type": "select",
                                                "label": "New Drive Location",
                                                "required": true,
                                                "validate": {
                                                    "enum": [
                                                        "drive",
                                                        "share",
                                                        "team"
                                                    ]
                                                }
                                            },
                                            {
                                                "name": "folderId",
                                                "type": "folder",
                                                "label": "New Folder Location",
                                                "required": true
                                            }
                                        ]
                                    }
                                },
                                {
                                    "id": 158,
                                    "module": "google-drive:shareAFileFolder",
                                    "version": 4,
                                    "parameters": {
                                        "__IMTCONN__": 8853889
                                    },
                                    "mapper": {
                                        "file": "{{152.id}}",
                                        "role": "reader",
                                        "type": "anyone",
                                        "select": "file",
                                        "destination": "drive",
                                        "allowFileDiscovery": false
                                    },
                                    "metadata": {
                                        "designer": {
                                            "x": 2526,
                                            "y": -57
                                        },
                                        "restore": {
                                            "expect": {
                                                "file": {
                                                    "mode": "edit",
                                                    "path": []
                                                },
                                                "role": {
                                                    "mode": "chose",
                                                    "label": "Reader"
                                                },
                                                "type": {
                                                    "mode": "chose",
                                                    "label": "Anyone"
                                                },
                                                "select": {
                                                    "label": "File"
                                                },
                                                "destination": {
                                                    "label": "My Drive"
                                                },
                                                "allowFileDiscovery": {
                                                    "mode": "chose"
                                                }
                                            },
                                            "parameters": {
                                                "__IMTCONN__": {
                                                    "data": {
                                                        "scoped": "true",
                                                        "connection": "google-restricted"
                                                    },
                                                    "label": "My Google Restricted connection (przemo.dud@gmail.com)"
                                                }
                                            }
                                        },
                                        "parameters": [
                                            {
                                                "name": "__IMTCONN__",
                                                "type": "account:google-restricted",
                                                "label": "Connection",
                                                "required": true
                                            }
                                        ],
                                        "expect": [
                                            {
                                                "type": "hidden"
                                            },
                                            {
                                                "name": "destination",
                                                "type": "select",
                                                "label": "Choose a Drive",
                                                "required": true,
                                                "validate": {
                                                    "enum": [
                                                        "drive",
                                                        "share",
                                                        "team"
                                                    ]
                                                }
                                            },
                                            {
                                                "name": "select",
                                                "type": "select",
                                                "label": "Select",
                                                "required": true,
                                                "validate": {
                                                    "enum": [
                                                        "file",
                                                        "folder"
                                                    ]
                                                }
                                            },
                                            {
                                                "name": "role",
                                                "type": "select",
                                                "label": "Role",
                                                "required": true,
                                                "validate": {
                                                    "enum": [
                                                        "owner",
                                                        "writer",
                                                        "commenter",
                                                        "reader"
                                                    ]
                                                }
                                            },
                                            {
                                                "name": "type",
                                                "type": "select",
                                                "label": "Type",
                                                "required": true,
                                                "validate": {
                                                    "enum": [
                                                        "user",
                                                        "group",
                                                        "domain",
                                                        "anyone"
                                                    ]
                                                }
                                            },
                                            {
                                                "mode": "edit",
                                                "name": "file",
                                                "type": "file",
                                                "label": "File ID",
                                                "required": true
                                            },
                                            {
                                                "name": "allowFileDiscovery",
                                                "type": "boolean",
                                                "label": "Allow File Discovery",
                                                "required": true
                                            }
                                        ],
                                        "interface": [
                                            {
                                                "name": "fileId",
                                                "type": "text",
                                                "label": "File ID"
                                            },
                                            {
                                                "name": "kind",
                                                "type": "text",
                                                "label": "Kind"
                                            },
                                            {
                                                "name": "id",
                                                "type": "text",
                                                "label": "Permission ID"
                                            },
                                            {
                                                "name": "type",
                                                "type": "text",
                                                "label": "Type"
                                            },
                                            {
                                                "name": "shareLink",
                                                "type": "url",
                                                "label": "Share Link"
                                            },
                                            {
                                                "name": "emailAddress",
                                                "type": "text",
                                                "label": "Email Address"
                                            },
                                            {
                                                "name": "domain",
                                                "type": "text",
                                                "label": "Domain"
                                            },
                                            {
                                                "name": "role",
                                                "type": "text",
                                                "label": "Role"
                                            },
                                            {
                                                "name": "allowFileDiscovery",
                                                "type": "boolean",
                                                "label": "Allow File Discovery"
                                            },
                                            {
                                                "name": "displayName",
                                                "type": "text",
                                                "label": "Display Name"
                                            },
                                            {
                                                "name": "photoLink",
                                                "type": "text",
                                                "label": "Photo Link"
                                            },
                                            {
                                                "name": "expirationTime",
                                                "type": "date",
                                                "label": "Expiration Time"
                                            },
                                            {
                                                "name": "teamDrivePermissionDetails",
                                                "spec": {
                                                    "spec": [
                                                        {
                                                            "name": "teamDrivePermissionType",
                                                            "type": "text",
                                                            "label": "Team Drive Permission Type"
                                                        },
                                                        {
                                                            "name": "role",
                                                            "type": "text",
                                                            "label": "Role"
                                                        },
                                                        {
                                                            "name": "inheritedFrom",
                                                            "type": "text",
                                                            "label": "Inherited From"
                                                        },
                                                        {
                                                            "name": "inherited",
                                                            "type": "boolean",
                                                            "label": "Inherited"
                                                        }
                                                    ],
                                                    "type": "collection"
                                                },
                                                "type": "array",
                                                "label": "Team Drive Permission Details"
                                            },
                                            {
                                                "name": "permissionDetails",
                                                "spec": {
                                                    "spec": [
                                                        {
                                                            "name": "permissionType",
                                                            "type": "text",
                                                            "label": "Permission Type"
                                                        },
                                                        {
                                                            "name": "role",
                                                            "type": "text",
                                                            "label": "Role"
                                                        },
                                                        {
                                                            "name": "inheritedFrom",
                                                            "type": "text",
                                                            "label": "Inherited From"
                                                        },
                                                        {
                                                            "name": "inherited",
                                                            "type": "boolean",
                                                            "label": "Inherited"
                                                        }
                                                    ],
                                                    "type": "collection"
                                                },
                                                "type": "array",
                                                "label": "Permission Details"
                                            },
                                            {
                                                "name": "deleted",
                                                "type": "boolean",
                                                "label": "Deleted"
                                            },
                                            {
                                                "name": "pendingOwner",
                                                "type": "boolean",
                                                "label": "Pending Owner"
                                            },
                                            {
                                                "name": "webContentLink",
                                                "type": "url",
                                                "label": "Web Content Link"
                                            }
                                        ]
                                    }
                                },
                                {
                                    "id": 155,
                                    "module": "airtable:ActionCreateRecord",
                                    "version": 3,
                                    "parameters": {
                                        "__IMTCONN__": 8854084
                                    },
                                    "mapper": {
                                        "base": "appbLQjOUm58ckRwu",
                                        "table": "tbl2jnB2TinJgSh6b",
                                        "record": {
                                            "fld6bgt6cHkiVUfFA": "{{71.`seller's ID`}}",
                                            "fld6vG3DGoUpzPcA4": "{{71.`vat amount`}}",
                                            "fldK8tEFinHZ9abHT": "{{now}}",
                                            "fldS3rOhvpGs37yNF": "{{158.shareLink}}",
                                            "fldbApkvRSBl9Qw4y": "{{71.invoice_number}}",
                                            "fldn2IJFDXqBUzHaL": "{{71.net_amount}}",
                                            "fldnCRKsb5H1wjBjG": "{{71.supplier}}",
                                            "fldoh6Gji56J0vmAf": [
                                                {
                                                    "url": "{{158.webContentLink}}"
                                                }
                                            ],
                                            "fldw6nVdJUaRz9po7": "{{71.`sale date`}}",
                                            "fldy4mjVMVUImNtZm": "{{71.issue_date}}",
                                            "fldyl1RnGNnH4jIvN": "{{71.gross_amount}}"
                                        },
                                        "typecast": false,
                                        "useColumnId": false
                                    },
                                    "metadata": {
                                        "designer": {
                                            "x": 2889,
                                            "y": 28,
                                            "name": "💾 Zapisanie danych w zestawieniu"
                                        },
                                        "restore": {
                                            "expect": {
                                                "base": {
                                                    "label": "Automatyzacja faktur - Gotowy Scenariusz"
                                                },
                                                "table": {
                                                    "label": "Zestawienie",
                                                    "nested": [
                                                        {
                                                            "name": "record",
                                                            "spec": [
                                                                {
                                                                    "name": "fldK8tEFinHZ9abHT",
                                                                    "time": false,
                                                                    "type": "date",
                                                                    "label": "Data pobrania"
                                                                },
                                                                {
                                                                    "name": "fldnCRKsb5H1wjBjG",
                                                                    "type": "text",
                                                                    "label": "Dostawca"
                                                                },
                                                                {
                                                                    "name": "fld6bgt6cHkiVUfFA",
                                                                    "type": "text",
                                                                    "label": "Numer NIP"
                                                                },
                                                                {
                                                                    "name": "fldbApkvRSBl9Qw4y",
                                                                    "type": "text",
                                                                    "label": "Numer FV"
                                                                },
                                                                {
                                                                    "name": "fldy4mjVMVUImNtZm",
                                                                    "time": false,
                                                                    "type": "date",
                                                                    "label": "Data wystawienia FV"
                                                                },
                                                                {
                                                                    "name": "fldw6nVdJUaRz9po7",
                                                                    "time": false,
                                                                    "type": "date",
                                                                    "label": "Data sprzedaży FV"
                                                                },
                                                                {
                                                                    "name": "fldn2IJFDXqBUzHaL",
                                                                    "type": "number",
                                                                    "label": "Kwota netto"
                                                                },
                                                                {
                                                                    "name": "fld6vG3DGoUpzPcA4",
                                                                    "type": "number",
                                                                    "label": "Kwota VAT"
                                                                },
                                                                {
                                                                    "name": "fldyl1RnGNnH4jIvN",
                                                                    "type": "number",
                                                                    "label": "Kwota brutto "
                                                                },
                                                                {
                                                                    "mode": "edit",
                                                                    "name": "flduAgQl018DzJAn8",
                                                                    "type": "select",
                                                                    "label": "KATEGORIA",
                                                                    "dynamic": true,
                                                                    "options": [
                                                                        {
                                                                            "label": "Allegro",
                                                                            "value": "Allegro"
                                                                        },
                                                                        {
                                                                            "label": "Księgowość",
                                                                            "value": "Księgowość"
                                                                        },
                                                                        {
                                                                            "label": "Marketing",
                                                                            "value": "Marketing"
                                                                        },
                                                                        {
                                                                            "label": "Odpady Komunalne",
                                                                            "value": "Odpady Komunalne"
                                                                        },
                                                                        {
                                                                            "label": "Odprawa Celna",
                                                                            "value": "Odprawa Celna"
                                                                        },
                                                                        {
                                                                            "label": "Ogrzewanie",
                                                                            "value": "Ogrzewanie"
                                                                        },
                                                                        {
                                                                            "label": "Oprogramowanie Sprzedażowe",
                                                                            "value": "Oprogramowanie Sprzedażowe"
                                                                        },
                                                                        {
                                                                            "label": "Podatek",
                                                                            "value": "Podatek"
                                                                        },
                                                                        {
                                                                            "label": "Pakowanie",
                                                                            "value": "Pakowanie"
                                                                        },
                                                                        {
                                                                            "label": "Pozycjonowanie",
                                                                            "value": "Pozycjonowanie"
                                                                        },
                                                                        {
                                                                            "label": "Prąd",
                                                                            "value": "Prąd"
                                                                        },
                                                                        {
                                                                            "label": "Prestashop",
                                                                            "value": "Prestashop"
                                                                        },
                                                                        {
                                                                            "label": "Promowanie Sklepu",
                                                                            "value": "Promowanie Sklepu"
                                                                        },
                                                                        {
                                                                            "label": "Prowizje Płatności",
                                                                            "value": "Prowizje Płatności"
                                                                        },
                                                                        {
                                                                            "label": "Przesyłki",
                                                                            "value": "Przesyłki"
                                                                        },
                                                                        {
                                                                            "label": "Samochód",
                                                                            "value": "Samochód"
                                                                        },
                                                                        {
                                                                            "label": "Szkolenia",
                                                                            "value": "Szkolenia"
                                                                        },
                                                                        {
                                                                            "label": "Telefon i internet",
                                                                            "value": "Telefon i internet"
                                                                        },
                                                                        {
                                                                            "label": "Transport Chiny",
                                                                            "value": "Transport Chiny"
                                                                        },
                                                                        {
                                                                            "label": "Ubezpieczenia",
                                                                            "value": "Ubezpieczenia"
                                                                        },
                                                                        {
                                                                            "label": "VAT",
                                                                            "value": "VAT"
                                                                        },
                                                                        {
                                                                            "label": "Własne",
                                                                            "value": "Własne"
                                                                        },
                                                                        {
                                                                            "label": "Wynajem Sklep",
                                                                            "value": "Wynajem Sklep"
                                                                        },
                                                                        {
                                                                            "label": "Wyposażenie Magazyn",
                                                                            "value": "Wyposażenie Magazyn"
                                                                        },
                                                                        {
                                                                            "label": "Zakup Towaru",
                                                                            "value": "Zakup Towaru"
                                                                        },
                                                                        {
                                                                            "label": "ZUS",
                                                                            "value": "ZUS"
                                                                        },
                                                                        {
                                                                            "label": "BDO",
                                                                            "value": "BDO"
                                                                        },
                                                                        {
                                                                            "label": "serwer",
                                                                            "value": "serwer"
                                                                        },
                                                                        {
                                                                            "label": "Google ADS",
                                                                            "value": "Google ADS"
                                                                        }
                                                                    ],
                                                                    "validate": false
                                                                },
                                                                {
                                                                    "name": "fld0vlZQoRMps1ORS",
                                                                    "type": "text",
                                                                    "label": "Krótki Opis"
                                                                },
                                                                {
                                                                    "name": "fldoh6Gji56J0vmAf",
                                                                    "spec": [
                                                                        {
                                                                            "help": "Airtable will download the file at the given url and keep its own copy of it.",
                                                                            "name": "url",
                                                                            "type": "text",
                                                                            "label": "File URL"
                                                                        },
                                                                        {
                                                                            "help": "If empty, Airtable will generate the name automatically.",
                                                                            "name": "filename",
                                                                            "type": "filename",
                                                                            "label": "File name"
                                                                        }
                                                                    ],
                                                                    "type": "array",
                                                                    "label": "Faktura",
                                                                    "editable": true
                                                                },
                                                                {
                                                                    "name": "fldS3rOhvpGs37yNF",
                                                                    "type": "text",
                                                                    "label": "Lokalizacja - GDrive"
                                                                },
                                                                {
                                                                    "name": "fldYSsK0K0uRcAMBD",
                                                                    "time": false,
                                                                    "type": "date",
                                                                    "label": "Data_importu"
                                                                },
                                                                {
                                                                    "name": "fldwS5ZZpq5f98gHc",
                                                                    "type": "text",
                                                                    "label": "ID_platnosci_Fakturownia"
                                                                }
                                                            ],
                                                            "type": "collection",
                                                            "label": "Record"
                                                        }
                                                    ]
                                                },
                                                "record": {
                                                    "nested": {
                                                        "fldoh6Gji56J0vmAf": {
                                                            "mode": "chose",
                                                            "items": [
                                                                null
                                                            ]
                                                        },
                                                        "flduAgQl018DzJAn8": {
                                                            "mode": "edit"
                                                        }
                                                    }
                                                },
                                                "typecast": {
                                                    "mode": "chose"
                                                },
                                                "useColumnId": {
                                                    "mode": "chose"
                                                }
                                            },
                                            "parameters": {
                                                "__IMTCONN__": {
                                                    "data": {
                                                        "scoped": "true",
                                                        "connection": "airtable3"
                                                    },
                                                    "label": "My Airtable OAuth connection (User ID: usrcV7XR8wZp1djp3)"
                                                }
                                            }
                                        },
                                        "parameters": [
                                            {
                                                "name": "__IMTCONN__",
                                                "type": "account:airtable3,airtable2",
                                                "label": "Connection",
                                                "required": true
                                            }
                                        ],
                                        "expect": [
                                            {
                                                "name": "base",
                                                "type": "select",
                                                "label": "Base",
                                                "required": true
                                            },
                                            {
                                                "name": "typecast",
                                                "type": "boolean",
                                                "label": "Smart links",
                                                "required": true
                                            },
                                            {
                                                "name": "useColumnId",
                                                "type": "boolean",
                                                "label": "Use Column ID",
                                                "required": true
                                            },
                                            {
                                                "name": "table",
                                                "type": "select",
                                                "label": "Table",
                                                "required": true
                                            },
                                            {
                                                "name": "record",
                                                "spec": [
                                                    {
                                                        "name": "fldK8tEFinHZ9abHT",
                                                        "time": false,
                                                        "type": "date",
                                                        "label": "Data pobrania"
                                                    },
                                                    {
                                                        "name": "fldnCRKsb5H1wjBjG",
                                                        "type": "text",
                                                        "label": "Dostawca"
                                                    },
                                                    {
                                                        "name": "fld6bgt6cHkiVUfFA",
                                                        "type": "text",
                                                        "label": "Numer NIP"
                                                    },
                                                    {
                                                        "name": "fldbApkvRSBl9Qw4y",
                                                        "type": "text",
                                                        "label": "Numer FV"
                                                    },
                                                    {
                                                        "name": "fldy4mjVMVUImNtZm",
                                                        "time": false,
                                                        "type": "date",
                                                        "label": "Data wystawienia FV"
                                                    },
                                                    {
                                                        "name": "fldw6nVdJUaRz9po7",
                                                        "time": false,
                                                        "type": "date",
                                                        "label": "Data sprzedaży FV"
                                                    },
                                                    {
                                                        "name": "fldn2IJFDXqBUzHaL",
                                                        "type": "number",
                                                        "label": "Kwota netto"
                                                    },
                                                    {
                                                        "name": "fld6vG3DGoUpzPcA4",
                                                        "type": "number",
                                                        "label": "Kwota VAT"
                                                    },
                                                    {
                                                        "name": "fldyl1RnGNnH4jIvN",
                                                        "type": "number",
                                                        "label": "Kwota brutto "
                                                    },
                                                    {
                                                        "mode": "edit",
                                                        "name": "flduAgQl018DzJAn8",
                                                        "type": "select",
                                                        "label": "KATEGORIA"
                                                    },
                                                    {
                                                        "name": "fld0vlZQoRMps1ORS",
                                                        "type": "text",
                                                        "label": "Krótki Opis"
                                                    },
                                                    {
                                                        "name": "fldoh6Gji56J0vmAf",
                                                        "spec": [
                                                            {
                                                                "name": "url",
                                                                "type": "text",
                                                                "label": "File URL"
                                                            },
                                                            {
                                                                "name": "filename",
                                                                "type": "filename",
                                                                "label": "File name"
                                                            }
                                                        ],
                                                        "type": "array",
                                                        "label": "Faktura"
                                                    },
                                                    {
                                                        "name": "fldS3rOhvpGs37yNF",
                                                        "type": "text",
                                                        "label": "Lokalizacja - GDrive"
                                                    },
                                                    {
                                                        "name": "fldYSsK0K0uRcAMBD",
                                                        "time": false,
                                                        "type": "date",
                                                        "label": "Data_importu"
                                                    },
                                                    {
                                                        "name": "fldwS5ZZpq5f98gHc",
                                                        "type": "text",
                                                        "label": "ID_platnosci_Fakturownia"
                                                    }
                                                ],
                                                "type": "collection",
                                                "label": "Record"
                                            }
                                        ],
                                        "interface": [
                                            {
                                                "name": "id",
                                                "type": "text",
                                                "label": "ID"
                                            },
                                            {
                                                "name": "createdTime",
                                                "type": "date",
                                                "label": "Created Time"
                                            },
                                            {
                                                "name": "Data pobrania",
                                                "time": false,
                                                "type": "date",
                                                "label": "Data pobrania"
                                            },
                                            {
                                                "name": "Dostawca",
                                                "type": "text",
                                                "label": "Dostawca"
                                            },
                                            {
                                                "name": "Numer NIP",
                                                "type": "text",
                                                "label": "Numer NIP"
                                            },
                                            {
                                                "name": "Numer FV",
                                                "type": "text",
                                                "label": "Numer FV"
                                            },
                                            {
                                                "name": "Data wystawienia FV",
                                                "time": false,
                                                "type": "date",
                                                "label": "Data wystawienia FV"
                                            },
                                            {
                                                "name": "Data sprzedaży FV",
                                                "time": false,
                                                "type": "date",
                                                "label": "Data sprzedaży FV"
                                            },
                                            {
                                                "name": "Kwota netto",
                                                "type": "number",
                                                "label": "Kwota netto"
                                            },
                                            {
                                                "name": "Kwota VAT",
                                                "type": "number",
                                                "label": "Kwota VAT"
                                            },
                                            {
                                                "name": "Kwota brutto ",
                                                "type": "number",
                                                "label": "Kwota brutto "
                                            },
                                            {
                                                "name": "KATEGORIA",
                                                "type": "text",
                                                "label": "KATEGORIA"
                                            },
                                            {
                                                "name": "Krótki Opis",
                                                "type": "text",
                                                "label": "Krótki Opis"
                                            },
                                            {
                                                "name": "Faktura",
                                                "spec": [
                                                    {
                                                        "name": "id",
                                                        "type": "text",
                                                        "label": "ID"
                                                    },
                                                    {
                                                        "name": "width",
                                                        "type": "uinteger",
                                                        "label": "Width"
                                                    },
                                                    {
                                                        "name": "height",
                                                        "type": "uinteger",
                                                        "label": "Height"
                                                    },
                                                    {
                                                        "name": "url",
                                                        "type": "url",
                                                        "label": "URL"
                                                    },
                                                    {
                                                        "name": "filename",
                                                        "type": "filename",
                                                        "label": "File name"
                                                    },
                                                    {
                                                        "name": "size",
                                                        "type": "uinteger",
                                                        "label": "Size"
                                                    },
                                                    {
                                                        "name": "type",
                                                        "type": "text",
                                                        "label": "MIME type"
                                                    },
                                                    {
                                                        "name": "thumbnails",
                                                        "spec": [
                                                            {
                                                                "name": "small",
                                                                "spec": [
                                                                    {
                                                                        "name": "url",
                                                                        "type": "url",
                                                                        "label": "URL"
                                                                    },
                                                                    {
                                                                        "name": "width",
                                                                        "type": "uinteger",
                                                                        "label": "Width"
                                                                    },
                                                                    {
                                                                        "name": "height",
                                                                        "type": "uinteger",
                                                                        "label": "Height"
                                                                    }
                                                                ],
                                                                "type": "collection",
                                                                "label": "Small"
                                                            },
                                                            {
                                                                "name": "large",
                                                                "spec": [
                                                                    {
                                                                        "name": "url",
                                                                        "type": "url",
                                                                        "label": "URL"
                                                                    },
                                                                    {
                                                                        "name": "width",
                                                                        "type": "uinteger",
                                                                        "label": "Width"
                                                                    },
                                                                    {
                                                                        "name": "height",
                                                                        "type": "uinteger",
                                                                        "label": "Height"
                                                                    }
                                                                ],
                                                                "type": "collection",
                                                                "label": "Large"
                                                            },
                                                            {
                                                                "name": "full",
                                                                "spec": [
                                                                    {
                                                                        "name": "url",
                                                                        "type": "url",
                                                                        "label": "URL"
                                                                    },
                                                                    {
                                                                        "name": "width",
                                                                        "type": "uinteger",
                                                                        "label": "Width"
                                                                    },
                                                                    {
                                                                        "name": "height",
                                                                        "type": "uinteger",
                                                                        "label": "Height"
                                                                    }
                                                                ],
                                                                "type": "collection",
                                                                "label": "Full"
                                                            }
                                                        ],
                                                        "type": "collection",
                                                        "label": "Thumbnails"
                                                    }
                                                ],
                                                "type": "array",
                                                "label": "Faktura"
                                            },
                                            {
                                                "name": "Lokalizacja - GDrive",
                                                "type": "text",
                                                "label": "Lokalizacja - GDrive"
                                            },
                                            {
                                                "name": "Data_importu",
                                                "time": false,
                                                "type": "date",
                                                "label": "Data_importu"
                                            },
                                            {
                                                "name": "ID_platnosci_Fakturownia",
                                                "type": "text",
                                                "label": "ID_platnosci_Fakturownia"
                                            }
                                        ]
                                    }
                                },
                                {
                                    "id": 154,
                                    "module": "slack:CreateMessage",
                                    "version": 4,
                                    "parameters": {
                                        "__IMTCONN__": 8853058
                                    },
                                    "mapper": {
                                        "parse": false,
                                        "blocks": "{\n  \"blocks\": [\n    {\n      \"type\": \"section\",\n      \"text\": {\n        \"type\": \"mrkdwn\",\n        \"text\": \"📄 *Nowa faktura do sprawdzenia!*\"\n      }\n    },\n    {\n      \"type\": \"section\",\n      \"fields\": [\n        {\n          \"type\": \"mrkdwn\",\n          \"text\": \"*Dostawca:*\\n{{155.Dostawca}}\"\n        },\n        {\n          \"type\": \"mrkdwn\", \n          \"text\": \"*Kwota netto:*\\n{{155.`Kwota netto`}} PLN\"\n        }\n      ]\n    },\n    {\n      \"type\": \"section\",\n      \"text\": {\n        \"type\": \"mrkdwn\",\n        \"text\": \"*<{{158.shareLink}}|Zobacz dokument na Dysku Google>*\"\n      }\n    }\n  ]\n}",
                                        "mrkdwn": true,
                                        "channel": "C08V6TJ9K2T",
                                        "channelType": "public",
                                        "channelWType": "list"
                                    },
                                    "metadata": {
                                        "designer": {
                                            "x": 3341,
                                            "y": -6,
                                            "name": "Powiadomienie"
                                        },
                                        "restore": {
                                            "expect": {
                                                "parse": {
                                                    "mode": "chose"
                                                },
                                                "mrkdwn": {
                                                    "mode": "chose"
                                                },
                                                "channel": {
                                                    "mode": "chose",
                                                    "label": "faktury"
                                                },
                                                "link_names": {
                                                    "mode": "chose"
                                                },
                                                "channelType": {
                                                    "label": "Public channel"
                                                },
                                                "channelWType": {
                                                    "label": "Select from the list"
                                                },
                                                "unfurl_links": {
                                                    "mode": "chose"
                                                },
                                                "unfurl_media": {
                                                    "mode": "chose"
                                                },
                                                "reply_broadcast": {
                                                    "mode": "chose"
                                                }
                                            },
                                            "parameters": {
                                                "__IMTCONN__": {
                                                    "data": {
                                                        "scoped": "true",
                                                        "connection": "slack2"
                                                    },
                                                    "label": "dotykowewlaczniki@gmail.com (dcsmarthomeworkspace.slack.com - dotykowewlaczniki)"
                                                }
                                            }
                                        },
                                        "parameters": [
                                            {
                                                "name": "__IMTCONN__",
                                                "type": "account:slack2,slack3",
                                                "label": "Connection",
                                                "required": true
                                            }
                                        ],
                                        "expect": [
                                            {
                                                "name": "channelWType",
                                                "type": "select",
                                                "label": "Enter a channel ID or name",
                                                "required": true,
                                                "validate": {
                                                    "enum": [
                                                        "manualy",
                                                        "list"
                                                    ]
                                                }
                                            },
                                            {
                                                "name": "text",
                                                "type": "text",
                                                "label": "Text"
                                            },
                                            {
                                                "name": "blocks",
                                                "type": "text",
                                                "label": "Blocks"
                                            },
                                            {
                                                "name": "thread_ts",
                                                "type": "text",
                                                "label": "Thread message ID (time stamp)"
                                            },
                                            {
                                                "name": "reply_broadcast",
                                                "type": "boolean",
                                                "label": "Reply broadcast"
                                            },
                                            {
                                                "name": "link_names",
                                                "type": "boolean",
                                                "label": "Link names"
                                            },
                                            {
                                                "name": "parse",
                                                "type": "boolean",
                                                "label": "Parse message text"
                                            },
                                            {
                                                "name": "mrkdwn",
                                                "type": "boolean",
                                                "label": "Use markdown"
                                            },
                                            {
                                                "name": "unfurl_links",
                                                "type": "boolean",
                                                "label": "Unfurl primarily text-based content"
                                            },
                                            {
                                                "name": "unfurl_media",
                                                "type": "boolean",
                                                "label": "Unfurl media content"
                                            },
                                            {
                                                "name": "icon_emoji",
                                                "type": "text",
                                                "label": "Icon emoji"
                                            },
                                            {
                                                "name": "icon_url",
                                                "type": "url",
                                                "label": "Icon url"
                                            },
                                            {
                                                "name": "username",
                                                "type": "text",
                                                "label": "User name"
                                            },
                                            {
                                                "name": "channelType",
                                                "type": "select",
                                                "label": "Channel type",
                                                "required": true,
                                                "validate": {
                                                    "enum": [
                                                        "public",
                                                        "private",
                                                        "im",
                                                        "mpim"
                                                    ]
                                                }
                                            },
                                            {
                                                "name": "channel",
                                                "type": "select",
                                                "label": "Public channel",
                                                "required": true
                                            }
                                        ]
                                    }
                                },
                                {
                                    "id": 240,
                                    "module": "openai-gpt-3:CreateCompletion",
                                    "version": 1,
                                    "parameters": {
                                        "__IMTCONN__": 4016539
                                    },
                                    "mapper": {
                                        "model": "o4-mini",
                                        "top_p": "1",
                                        "select": "chat",
                                        "messages": [
                                            {
                                                "role": "system",
                                                "content": "Zadanie  \r\nOtrzymujesz listę nazw sprzedawców (po jednym wierszu).  \r\nDla każdego sprzedawcy:\r\n\r\n1. Wyszukaj odpowiadającą **kategorię** w słowniku poniżej.  \r\n2. Zwróć **wyłącznie nazwę kategorii**, dokładnie w takiej postaci (zachowaj wielkość liter i polskie znaki).  \r\n3. Jeśli sprzedawcy nie ma w słowniku, zwróć dokładnie: uzupelnic\r\n\r\nSłownik  \r\nAllegro sp. z o.o. – Allegro  \r\nAVT SPV Sp. z o.o. – Zakup Towaru  \r\nBaseLinker Sp. z o.o. – Oprogramowanie Sprzedażowe  \r\nBiuro Rachunkowe Ekonomus – Księgowość  \r\nBL LOGISTICS Sp. z o.o. – Przesyłki  \r\nDesign Book Spółka z o.o. – Marketing  \r\ndhosting.pl Sp. z o.o. – serwer  \r\nDPD Polska Sp. z o.o. – Przesyłki  \r\nECO POWER ENERGY – Zakup Towaru  \r\nEnerga Obrót SA – Prąd  \r\nGoogle Ireland Limited – Google ADS  \r\nIMZ Company Izabela Stanisz – Zakup Towaru  \r\nIndual Sp. z o.o. – Zakup Towaru  \r\nINPOST – Przesyłki  \r\nMOTO TMK S.C. – Zakup Towaru  \r\nPAWEŁ MIELCZAREK VSPRINT\" – Szkolenia  \r\nPayPro Spółka Akcyjna – Prowizje Płatności  \r\nTEKTOR – Pakowanie  \r\nSEMHUB – Pozycjonowanie  \r\nSENUTO Sp. z o.o. – Pozycjonowanie  \r\nStronyInternetowe.dev Amadeusz Weroński – Google ADS  \r\nTRU LABS – Zakup Towaru  \r\nVolkswagen Financial Services Polska Sp. z o.o.  – Samochód  \r\nZakład Usług Miejskich Sp. z o.o. – Odpady Komunalne\nP.H.U. KILMIR - Wynajem Sklep\n\n\r\nPrzykład  \r\n**Wejście:** Volkswagen  \r\n**Wyjście:** Samochód\r\n​"
                                            },
                                            {
                                                "role": "user",
                                                "content": "Gotowe dane pobranych faktur - {{71.supplier}}",
                                                "imageDetail": "auto"
                                            }
                                        ],
                                        "max_tokens": "2048",
                                        "temperature": "1",
                                        "n_completions": "1",
                                        "response_format": "text"
                                    },
                                    "metadata": {
                                        "designer": {
                                            "x": 3696,
                                            "y": -2
                                        },
                                        "restore": {
                                            "expect": {
                                                "stop": {
                                                    "mode": "chose"
                                                },
                                                "model": {
                                                    "mode": "chose",
                                                    "label": "o4-mini (system)"
                                                },
                                                "select": {
                                                    "label": "Create a Chat Completion (GPT and o1 models)"
                                                },
                                                "messages": {
                                                    "mode": "chose",
                                                    "items": [
                                                        {
                                                            "role": {
                                                                "mode": "chose",
                                                                "label": "Developer / System"
                                                            }
                                                        },
                                                        {
                                                            "role": {
                                                                "mode": "chose",
                                                                "label": "User"
                                                            },
                                                            "imageDetail": {
                                                                "mode": "chose",
                                                                "label": "Auto"
                                                            },
                                                            "imageInputType": {
                                                                "mode": "chose",
                                                                "label": "Empty"
                                                            }
                                                        }
                                                    ]
                                                },
                                                "logit_bias": {
                                                    "mode": "chose"
                                                },
                                                "tool_choice": {
                                                    "mode": "chose",
                                                    "label": "Empty"
                                                },
                                                "response_format": {
                                                    "mode": "chose",
                                                    "label": "Text"
                                                },
                                                "reasoning_effort": {
                                                    "mode": "chose",
                                                    "label": "Empty"
                                                },
                                                "additionalParameters": {
                                                    "mode": "chose"
                                                }
                                            },
                                            "parameters": {
                                                "__IMTCONN__": {
                                                    "data": {
                                                        "scoped": "true",
                                                        "connection": "openai-gpt-3"
                                                    },
                                                    "label": "My OpenAI connection"
                                                }
                                            }
                                        },
                                        "parameters": [
                                            {
                                                "name": "__IMTCONN__",
                                                "type": "account:openai-gpt-3",
                                                "label": "Connection",
                                                "required": true
                                            }
                                        ],
                                        "expect": [
                                            {
                                                "name": "select",
                                                "type": "select",
                                                "label": "Select Method",
                                                "required": true,
                                                "validate": {
                                                    "enum": [
                                                        "chat",
                                                        "prompt"
                                                    ]
                                                }
                                            },
                                            {
                                                "name": "temperature",
                                                "type": "number",
                                                "label": "Temperature",
                                                "validate": {
                                                    "max": 2,
                                                    "min": 0
                                                }
                                            },
                                            {
                                                "name": "top_p",
                                                "type": "number",
                                                "label": "Top P",
                                                "validate": {
                                                    "max": 1,
                                                    "min": 0
                                                }
                                            },
                                            {
                                                "name": "n_completions",
                                                "type": "number",
                                                "label": "Number"
                                            },
                                            {
                                                "name": "frequency_penalty",
                                                "type": "number",
                                                "label": "Frequency Penalty",
                                                "validate": {
                                                    "max": 2,
                                                    "min": -2
                                                }
                                            },
                                            {
                                                "name": "presence_penalty",
                                                "type": "number",
                                                "label": "Presence Penalty",
                                                "validate": {
                                                    "max": 2,
                                                    "min": -2
                                                }
                                            },
                                            {
                                                "name": "logit_bias",
                                                "spec": {
                                                    "name": "value",
                                                    "spec": [
                                                        {
                                                            "name": "token",
                                                            "type": "text",
                                                            "label": "Token ID",
                                                            "required": true
                                                        },
                                                        {
                                                            "name": "probability",
                                                            "type": "number",
                                                            "label": "Probability",
                                                            "required": true,
                                                            "validate": {
                                                                "max": 100,
                                                                "min": -100
                                                            }
                                                        }
                                                    ],
                                                    "type": "collection",
                                                    "label": "Token Probability"
                                                },
                                                "type": "array",
                                                "label": "Token Probability"
                                            },
                                            {
                                                "name": "seed",
                                                "type": "integer",
                                                "label": "Seed"
                                            },
                                            {
                                                "name": "tool_choice",
                                                "type": "select",
                                                "label": "Tool Choice",
                                                "validate": {
                                                    "enum": [
                                                        "none",
                                                        "auto",
                                                        "required"
                                                    ]
                                                }
                                            },
                                            {
                                                "name": "stop",
                                                "spec": {
                                                    "name": "value",
                                                    "type": "text",
                                                    "label": "Stop Sequence"
                                                },
                                                "type": "array",
                                                "label": "Stop Sequences",
                                                "validate": {
                                                    "maxItems": 4
                                                }
                                            },
                                            {
                                                "name": "additionalParameters",
                                                "spec": {
                                                    "name": "value",
                                                    "spec": [
                                                        {
                                                            "name": "key",
                                                            "type": "text",
                                                            "label": "Parameter Name",
                                                            "required": true
                                                        },
                                                        {
                                                            "name": "type",
                                                            "type": "select",
                                                            "label": "Input Type",
                                                            "options": [
                                                                {
                                                                    "label": "Text",
                                                                    "value": "text",
                                                                    "nested": [
                                                                        {
                                                                            "name": "value",
                                                                            "type": "text",
                                                                            "label": "Parameter Value"
                                                                        }
                                                                    ],
                                                                    "default": true
                                                                },
                                                                {
                                                                    "label": "Number",
                                                                    "value": "number",
                                                                    "nested": [
                                                                        {
                                                                            "name": "value",
                                                                            "type": "number",
                                                                            "label": "Parameter Value"
                                                                        }
                                                                    ]
                                                                },
                                                                {
                                                                    "label": "Boolean",
                                                                    "value": "boolean",
                                                                    "nested": [
                                                                        {
                                                                            "name": "value",
                                                                            "type": "boolean",
                                                                            "label": "Parameter Value"
                                                                        }
                                                                    ]
                                                                },
                                                                {
                                                                    "label": "Date",
                                                                    "value": "date",
                                                                    "nested": [
                                                                        {
                                                                            "name": "value",
                                                                            "type": "date",
                                                                            "label": "Parameter Value"
                                                                        }
                                                                    ]
                                                                },
                                                                {
                                                                    "label": "Any",
                                                                    "value": "any",
                                                                    "nested": [
                                                                        {
                                                                            "name": "value",
                                                                            "type": "any",
                                                                            "label": "Parameter Value"
                                                                        }
                                                                    ]
                                                                }
                                                            ]
                                                        }
                                                    ],
                                                    "type": "collection",
                                                    "label": "Input Parameter"
                                                },
                                                "type": "array",
                                                "label": "Other Input Parameters"
                                            },
                                            {
                                                "name": "model",
                                                "type": "select",
                                                "label": "Model",
                                                "required": true
                                            },
                                            {
                                                "name": "max_tokens",
                                                "type": "uinteger",
                                                "label": "Max Completion Tokens"
                                            },
                                            {
                                                "name": "messages",
                                                "spec": {
                                                    "name": "value",
                                                    "spec": [
                                                        {
                                                            "name": "role",
                                                            "type": "select",
                                                            "label": "Role",
                                                            "options": {
                                                                "store": [
                                                                    {
                                                                        "label": "User",
                                                                        "value": "user",
                                                                        "nested": [
                                                                            {
                                                                                "help": "Text content of the message on behalf of the selected __Role__.",
                                                                                "name": "content",
                                                                                "type": "text",
                                                                                "label": "Text Content"
                                                                            },
                                                                            {
                                                                                "name": "imageInputType",
                                                                                "type": "select",
                                                                                "label": "Image Input Type",
                                                                                "options": [
                                                                                    {
                                                                                        "label": "URL",
                                                                                        "value": "url",
                                                                                        "nested": [
                                                                                            {
                                                                                                "help": "Make sure to use a publicly accessible URL.\nYou can test if your image is publicly accessible by opening the link in an incognito tab.",
                                                                                                "name": "imageUrl",
                                                                                                "type": "url",
                                                                                                "label": "Image URL"
                                                                                            }
                                                                                        ]
                                                                                    },
                                                                                    {
                                                                                        "label": "Image File",
                                                                                        "value": "file",
                                                                                        "nested": [
                                                                                            {
                                                                                                "name": "imageFile",
                                                                                                "spec": [
                                                                                                    {
                                                                                                        "help": "Accepted extensions: `.jpg`, `.jpeg`, `.png`, `.webp` and `.gif`.",
                                                                                                        "name": "imageFilename",
                                                                                                        "type": "filename",
                                                                                                        "label": "Image Filename",
                                                                                                        "semantic": "file:name",
                                                                                                        "extension": [
                                                                                                            "jpg",
                                                                                                            "jpeg",
                                                                                                            "png",
                                                                                                            "webp",
                                                                                                            "gif"
                                                                                                        ]
                                                                                                    },
                                                                                                    {
                                                                                                        "name": "imageData",
                                                                                                        "type": "buffer",
                                                                                                        "label": "Image Data",
                                                                                                        "semantic": "file:data"
                                                                                                    }
                                                                                                ],
                                                                                                "type": "collection",
                                                                                                "label": "Image"
                                                                                            }
                                                                                        ]
                                                                                    }
                                                                                ],
                                                                                "mappable": false
                                                                            },
                                                                            {
                                                                                "help": "Recommended value: `Auto`",
                                                                                "name": "imageDetail",
                                                                                "type": "select",
                                                                                "label": "Image Detail",
                                                                                "options": [
                                                                                    {
                                                                                        "label": "Auto",
                                                                                        "value": "auto",
                                                                                        "default": true
                                                                                    },
                                                                                    {
                                                                                        "label": "High",
                                                                                        "value": "high"
                                                                                    },
                                                                                    {
                                                                                        "label": "Low",
                                                                                        "value": "low"
                                                                                    }
                                                                                ]
                                                                            }
                                                                        ]
                                                                    },
                                                                    {
                                                                        "label": "Assistant",
                                                                        "value": "assistant",
                                                                        "nested": [
                                                                            {
                                                                                "help": "Text content of the message on behalf of the selected __Role__.",
                                                                                "name": "content",
                                                                                "type": "text",
                                                                                "label": "Text Content"
                                                                            },
                                                                            {
                                                                                "mode": "edit",
                                                                                "name": "tool_calls",
                                                                                "spec": {
                                                                                    "spec": [
                                                                                        {
                                                                                            "name": "type",
                                                                                            "type": "hidden",
                                                                                            "default": "function"
                                                                                        },
                                                                                        {
                                                                                            "help": "Map this directly from the output of a previous **Create a Completion** module. Look for `Choices[]: Message.Tool Calls[]: ID`.",
                                                                                            "name": "id",
                                                                                            "type": "text",
                                                                                            "label": "Tool call ID"
                                                                                        },
                                                                                        {
                                                                                            "name": "function",
                                                                                            "spec": [
                                                                                                {
                                                                                                    "help": "The name of the function previously called.",
                                                                                                    "name": "name",
                                                                                                    "type": "text",
                                                                                                    "label": "Name",
                                                                                                    "required": true
                                                                                                },
                                                                                                {
                                                                                                    "help": "The arguments previously output by the AI.",
                                                                                                    "name": "arguments",
                                                                                                    "type": "text",
                                                                                                    "label": "Arguments",
                                                                                                    "required": true
                                                                                                }
                                                                                            ],
                                                                                            "type": "collection",
                                                                                            "label": "Function"
                                                                                        }
                                                                                    ],
                                                                                    "type": "collection",
                                                                                    "label": "Tool Call"
                                                                                },
                                                                                "type": "array",
                                                                                "label": "Tool Calls",
                                                                                "labels": {
                                                                                    "add": "Add tool call"
                                                                                },
                                                                                "mappable": {
                                                                                    "help": "You can map the entire `Choices[]: Message.Tool Calls` array from a previous Create a Completion module here."
                                                                                }
                                                                            }
                                                                        ]
                                                                    },
                                                                    {
                                                                        "label": "Developer / System",
                                                                        "value": "system",
                                                                        "nested": [
                                                                            {
                                                                                "help": "Text content of the message on behalf of the selected __Role__.",
                                                                                "name": "content",
                                                                                "type": "text",
                                                                                "label": "Text Content"
                                                                            }
                                                                        ]
                                                                    },
                                                                    {
                                                                        "label": "Tool",
                                                                        "value": "tool",
                                                                        "nested": [
                                                                            {
                                                                                "help": "The return of the function. This role should only be used when you have processed a previous function call and want to send the output of the function execution back to the AI.",
                                                                                "name": "content",
                                                                                "type": "text",
                                                                                "label": "Text Content",
                                                                                "required": true
                                                                            },
                                                                            {
                                                                                "help": "Map this directly from the output of a previous **Create a Completion** module. Look for `Choices[]: Message.Tool Calls[]: ID`.",
                                                                                "name": "tool_call_id",
                                                                                "type": "text",
                                                                                "label": "Tool Call ID.",
                                                                                "required": true
                                                                            }
                                                                        ]
                                                                    }
                                                                ]
                                                            },
                                                            "required": true
                                                        }
                                                    ],
                                                    "type": "collection",
                                                    "label": "Message"
                                                },
                                                "type": "array",
                                                "label": "Messages",
                                                "required": true
                                            },
                                            {
                                                "name": "response_format",
                                                "type": "select",
                                                "label": "Response Format",
                                                "validate": {
                                                    "enum": [
                                                        "text",
                                                        "json_object"
                                                    ]
                                                }
                                            },
                                            {
                                                "name": "reasoning_effort",
                                                "type": "select",
                                                "label": "Reasoning Effort",
                                                "validate": {
                                                    "enum": [
                                                        "minimal",
                                                        "low",
                                                        "medium",
                                                        "high"
                                                    ]
                                                }
                                            }
                                        ],
                                        "interface": [
                                            {
                                                "name": "result",
                                                "type": "any",
                                                "label": "Result"
                                            },
                                            {
                                                "name": "id",
                                                "type": "text",
                                                "label": "ID"
                                            },
                                            {
                                                "name": "object",
                                                "type": "text",
                                                "label": "Object"
                                            },
                                            {
                                                "name": "created",
                                                "type": "date",
                                                "label": "Created"
                                            },
                                            {
                                                "name": "model",
                                                "type": "text",
                                                "label": "Model"
                                            },
                                            {
                                                "name": "choices",
                                                "spec": [
                                                    {
                                                        "name": "text",
                                                        "type": "text",
                                                        "label": "Text"
                                                    },
                                                    {
                                                        "name": "index",
                                                        "type": "number",
                                                        "label": "Index"
                                                    },
                                                    {
                                                        "name": "logprobs",
                                                        "type": "text",
                                                        "label": "Log Probs"
                                                    },
                                                    {
                                                        "name": "finish_reason",
                                                        "type": "text",
                                                        "label": "Finish Reason"
                                                    },
                                                    {
                                                        "name": "message",
                                                        "spec": [
                                                            {
                                                                "name": "role",
                                                                "type": "text",
                                                                "label": "Role"
                                                            },
                                                            {
                                                                "name": "content",
                                                                "type": "text",
                                                                "label": "Content"
                                                            },
                                                            {
                                                                "name": "tool_calls",
                                                                "spec": [
                                                                    {
                                                                        "name": "id",
                                                                        "type": "text",
                                                                        "label": "ID"
                                                                    },
                                                                    {
                                                                        "name": "type",
                                                                        "type": "text",
                                                                        "label": "Type"
                                                                    },
                                                                    {
                                                                        "name": "function",
                                                                        "spec": [
                                                                            {
                                                                                "name": "name",
                                                                                "type": "text",
                                                                                "label": "Name"
                                                                            },
                                                                            {
                                                                                "name": "arguments",
                                                                                "type": "text",
                                                                                "label": "Arguments"
                                                                            }
                                                                        ],
                                                                        "type": "collection",
                                                                        "label": "Function"
                                                                    }
                                                                ],
                                                                "type": "array",
                                                                "label": "Tool Calls"
                                                            },
                                                            {
                                                                "name": "refusal",
                                                                "type": "text",
                                                                "label": "Refusal"
                                                            },
                                                            {
                                                                "name": "annotations",
                                                                "spec": [
                                                                    {
                                                                        "name": "type",
                                                                        "type": "text",
                                                                        "label": "Type"
                                                                    },
                                                                    {
                                                                        "name": "url_citation",
                                                                        "spec": [
                                                                            {
                                                                                "name": "end_index",
                                                                                "type": "number",
                                                                                "label": "End Index"
                                                                            },
                                                                            {
                                                                                "name": "start_index",
                                                                                "type": "number",
                                                                                "label": "Start Index"
                                                                            },
                                                                            {
                                                                                "name": "title",
                                                                                "type": "text",
                                                                                "label": "Title"
                                                                            },
                                                                            {
                                                                                "name": "url",
                                                                                "type": "text",
                                                                                "label": "URL"
                                                                            }
                                                                        ],
                                                                        "type": "collection",
                                                                        "label": "URL Citation"
                                                                    }
                                                                ],
                                                                "type": "array",
                                                                "label": "Annotations"
                                                            }
                                                        ],
                                                        "type": "collection",
                                                        "label": "Message"
                                                    }
                                                ],
                                                "type": "array",
                                                "label": "Choices"
                                            },
                                            {
                                                "name": "usage",
                                                "spec": [
                                                    {
                                                        "name": "prompt_tokens",
                                                        "type": "number",
                                                        "label": "Prompt Tokens"
                                                    },
                                                    {
                                                        "name": "completion_tokens",
                                                        "type": "text",
                                                        "label": "Completion Tokens"
                                                    },
                                                    {
                                                        "name": "total_tokens",
                                                        "type": "number",
                                                        "label": "Total Tokens"
                                                    },
                                                    {
                                                        "name": "prompt_tokens_details",
                                                        "spec": [
                                                            {
                                                                "name": "cached_tokens",
                                                                "type": "uinteger",
                                                                "label": "Cached Tokens"
                                                            },
                                                            {
                                                                "name": "text_tokens",
                                                                "type": "uinteger",
                                                                "label": "Text Tokens"
                                                            },
                                                            {
                                                                "name": "image_tokens",
                                                                "type": "uinteger",
                                                                "label": "Image Tokens"
                                                            },
                                                            {
                                                                "name": "audio_tokens",
                                                                "type": "uinteger",
                                                                "label": "Audio Tokens"
                                                            }
                                                        ],
                                                        "type": "collection",
                                                        "label": "Prompt Tokens Details"
                                                    },
                                                    {
                                                        "name": "completion_tokens_details",
                                                        "spec": [
                                                            {
                                                                "name": "reasoning_tokens",
                                                                "type": "uinteger",
                                                                "label": "Reasoning Tokens"
                                                            },
                                                            {
                                                                "name": "text_tokens",
                                                                "type": "uinteger",
                                                                "label": "Text Tokens"
                                                            },
                                                            {
                                                                "name": "audio_tokens",
                                                                "type": "uinteger",
                                                                "label": "Audio Tokens"
                                                            },
                                                            {
                                                                "name": "accepted_prediction_tokens",
                                                                "type": "uinteger",
                                                                "label": "Accepted Prediction Tokens"
                                                            },
                                                            {
                                                                "name": "rejected_prediction_tokens",
                                                                "type": "uinteger",
                                                                "label": "Rejected Prediction Tokens"
                                                            }
                                                        ],
                                                        "type": "collection",
                                                        "label": "Completion Tokens Details"
                                                    }
                                                ],
                                                "type": "collection",
                                                "label": "Usage"
                                            },
                                            {
                                                "name": "service_tier",
                                                "type": "text",
                                                "label": "Service Tier"
                                            },
                                            {
                                                "name": "system_fingerprint",
                                                "type": "text",
                                                "label": "System Fingerprint"
                                            }
                                        ]
                                    }
                                },
                                {
                                    "id": 243,
                                    "module": "airtable:ActionUpdateRecords",
                                    "version": 3,
                                    "parameters": {
                                        "__IMTCONN__": 8854084
                                    },
                                    "mapper": {
                                        "id": "{{155.id}}",
                                        "base": "appbLQjOUm58ckRwu",
                                        "table": "tbl2jnB2TinJgSh6b",
                                        "record": {
                                            "flduAgQl018DzJAn8": "{{240.result}}"
                                        },
                                        "typecast": false,
                                        "useColumnId": false
                                    },
                                    "metadata": {
                                        "designer": {
                                            "x": 4086,
                                            "y": 126
                                        },
                                        "restore": {
                                            "expect": {
                                                "base": {
                                                    "label": "Automatyzacja faktur - Gotowy Scenariusz"
                                                },
                                                "table": {
                                                    "label": "Zestawienie"
                                                },
                                                "record": {
                                                    "nested": {
                                                        "fldoh6Gji56J0vmAf": {
                                                            "mode": "chose"
                                                        },
                                                        "flduAgQl018DzJAn8": {
                                                            "mode": "edit"
                                                        }
                                                    }
                                                },
                                                "typecast": {
                                                    "mode": "chose"
                                                },
                                                "useColumnId": {
                                                    "mode": "chose"
                                                }
                                            },
                                            "parameters": {
                                                "__IMTCONN__": {
                                                    "data": {
                                                        "scoped": "true",
                                                        "connection": "airtable3"
                                                    },
                                                    "label": "My Airtable OAuth connection (User ID: usrcV7XR8wZp1djp3)"
                                                }
                                            }
                                        },
                                        "parameters": [
                                            {
                                                "name": "__IMTCONN__",
                                                "type": "account:airtable3,airtable2",
                                                "label": "Connection",
                                                "required": true
                                            }
                                        ],
                                        "expect": [
                                            {
                                                "name": "base",
                                                "type": "select",
                                                "label": "Base",
                                                "required": true
                                            },
                                            {
                                                "name": "typecast",
                                                "type": "boolean",
                                                "label": "Smart links",
                                                "required": true
                                            },
                                            {
                                                "name": "useColumnId",
                                                "type": "boolean",
                                                "label": "Use Column ID",
                                                "required": true
                                            },
                                            {
                                                "name": "table",
                                                "type": "select",
                                                "label": "Table",
                                                "required": true
                                            },
                                            {
                                                "name": "id",
                                                "type": "text",
                                                "label": "Record ID",
                                                "required": true
                                            },
                                            {
                                                "name": "record",
                                                "spec": [
                                                    {
                                                        "name": "fldK8tEFinHZ9abHT",
                                                        "time": false,
                                                        "type": "date",
                                                        "label": "Data pobrania"
                                                    },
                                                    {
                                                        "name": "fldnCRKsb5H1wjBjG",
                                                        "type": "text",
                                                        "label": "Dostawca"
                                                    },
                                                    {
                                                        "name": "fld6bgt6cHkiVUfFA",
                                                        "type": "text",
                                                        "label": "Numer NIP"
                                                    },
                                                    {
                                                        "name": "fldbApkvRSBl9Qw4y",
                                                        "type": "text",
                                                        "label": "Numer FV"
                                                    },
                                                    {
                                                        "name": "fldy4mjVMVUImNtZm",
                                                        "time": false,
                                                        "type": "date",
                                                        "label": "Data wystawienia FV"
                                                    },
                                                    {
                                                        "name": "fldw6nVdJUaRz9po7",
                                                        "time": false,
                                                        "type": "date",
                                                        "label": "Data sprzedaży FV"
                                                    },
                                                    {
                                                        "name": "fldn2IJFDXqBUzHaL",
                                                        "type": "number",
                                                        "label": "Kwota netto"
                                                    },
                                                    {
                                                        "name": "fld6vG3DGoUpzPcA4",
                                                        "type": "number",
                                                        "label": "Kwota VAT"
                                                    },
                                                    {
                                                        "name": "fldyl1RnGNnH4jIvN",
                                                        "type": "number",
                                                        "label": "Kwota brutto "
                                                    },
                                                    {
                                                        "mode": "edit",
                                                        "name": "flduAgQl018DzJAn8",
                                                        "type": "select",
                                                        "label": "KATEGORIA"
                                                    },
                                                    {
                                                        "name": "fld0vlZQoRMps1ORS",
                                                        "type": "text",
                                                        "label": "Krótki Opis"
                                                    },
                                                    {
                                                        "name": "fldoh6Gji56J0vmAf",
                                                        "spec": [
                                                            {
                                                                "name": "url",
                                                                "type": "text",
                                                                "label": "File URL"
                                                            },
                                                            {
                                                                "name": "filename",
                                                                "type": "filename",
                                                                "label": "File name"
                                                            }
                                                        ],
                                                        "type": "array",
                                                        "label": "Faktura"
                                                    },
                                                    {
                                                        "name": "fldS3rOhvpGs37yNF",
                                                        "type": "text",
                                                        "label": "Lokalizacja - GDrive"
                                                    },
                                                    {
                                                        "name": "fldYSsK0K0uRcAMBD",
                                                        "time": false,
                                                        "type": "date",
                                                        "label": "Data_importu"
                                                    },
                                                    {
                                                        "name": "fldwS5ZZpq5f98gHc",
                                                        "type": "text",
                                                        "label": "ID_platnosci_Fakturownia"
                                                    }
                                                ],
                                                "type": "collection",
                                                "label": "Record"
                                            }
                                        ],
                                        "interface": [
                                            {
                                                "name": "id",
                                                "type": "text",
                                                "label": "ID"
                                            },
                                            {
                                                "name": "createdTime",
                                                "type": "date",
                                                "label": "Created Time"
                                            },
                                            {
                                                "name": "Data pobrania",
                                                "time": false,
                                                "type": "date",
                                                "label": "Data pobrania"
                                            },
                                            {
                                                "name": "Dostawca",
                                                "type": "text",
                                                "label": "Dostawca"
                                            },
                                            {
                                                "name": "Numer NIP",
                                                "type": "text",
                                                "label": "Numer NIP"
                                            },
                                            {
                                                "name": "Numer FV",
                                                "type": "text",
                                                "label": "Numer FV"
                                            },
                                            {
                                                "name": "Data wystawienia FV",
                                                "time": false,
                                                "type": "date",
                                                "label": "Data wystawienia FV"
                                            },
                                            {
                                                "name": "Data sprzedaży FV",
                                                "time": false,
                                                "type": "date",
                                                "label": "Data sprzedaży FV"
                                            },
                                            {
                                                "name": "Kwota netto",
                                                "type": "number",
                                                "label": "Kwota netto"
                                            },
                                            {
                                                "name": "Kwota VAT",
                                                "type": "number",
                                                "label": "Kwota VAT"
                                            },
                                            {
                                                "name": "Kwota brutto ",
                                                "type": "number",
                                                "label": "Kwota brutto "
                                            },
                                            {
                                                "name": "KATEGORIA",
                                                "type": "text",
                                                "label": "KATEGORIA"
                                            },
                                            {
                                                "name": "Krótki Opis",
                                                "type": "text",
                                                "label": "Krótki Opis"
                                            },
                                            {
                                                "name": "Faktura",
                                                "spec": [
                                                    {
                                                        "name": "id",
                                                        "type": "text",
                                                        "label": "ID"
                                                    },
                                                    {
                                                        "name": "width",
                                                        "type": "uinteger",
                                                        "label": "Width"
                                                    },
                                                    {
                                                        "name": "height",
                                                        "type": "uinteger",
                                                        "label": "Height"
                                                    },
                                                    {
                                                        "name": "url",
                                                        "type": "url",
                                                        "label": "URL"
                                                    },
                                                    {
                                                        "name": "filename",
                                                        "type": "filename",
                                                        "label": "File name"
                                                    },
                                                    {
                                                        "name": "size",
                                                        "type": "uinteger",
                                                        "label": "Size"
                                                    },
                                                    {
                                                        "name": "type",
                                                        "type": "text",
                                                        "label": "MIME type"
                                                    },
                                                    {
                                                        "name": "thumbnails",
                                                        "spec": [
                                                            {
                                                                "name": "small",
                                                                "spec": [
                                                                    {
                                                                        "name": "url",
                                                                        "type": "url",
                                                                        "label": "URL"
                                                                    },
                                                                    {
                                                                        "name": "width",
                                                                        "type": "uinteger",
                                                                        "label": "Width"
                                                                    },
                                                                    {
                                                                        "name": "height",
                                                                        "type": "uinteger",
                                                                        "label": "Height"
                                                                    }
                                                                ],
                                                                "type": "collection",
                                                                "label": "Small"
                                                            },
                                                            {
                                                                "name": "large",
                                                                "spec": [
                                                                    {
                                                                        "name": "url",
                                                                        "type": "url",
                                                                        "label": "URL"
                                                                    },
                                                                    {
                                                                        "name": "width",
                                                                        "type": "uinteger",
                                                                        "label": "Width"
                                                                    },
                                                                    {
                                                                        "name": "height",
                                                                        "type": "uinteger",
                                                                        "label": "Height"
                                                                    }
                                                                ],
                                                                "type": "collection",
                                                                "label": "Large"
                                                            },
                                                            {
                                                                "name": "full",
                                                                "spec": [
                                                                    {
                                                                        "name": "url",
                                                                        "type": "url",
                                                                        "label": "URL"
                                                                    },
                                                                    {
                                                                        "name": "width",
                                                                        "type": "uinteger",
                                                                        "label": "Width"
                                                                    },
                                                                    {
                                                                        "name": "height",
                                                                        "type": "uinteger",
                                                                        "label": "Height"
                                                                    }
                                                                ],
                                                                "type": "collection",
                                                                "label": "Full"
                                                            }
                                                        ],
                                                        "type": "collection",
                                                        "label": "Thumbnails"
                                                    }
                                                ],
                                                "type": "array",
                                                "label": "Faktura"
                                            },
                                            {
                                                "name": "Lokalizacja - GDrive",
                                                "type": "text",
                                                "label": "Lokalizacja - GDrive"
                                            },
                                            {
                                                "name": "Data_importu",
                                                "time": false,
                                                "type": "date",
                                                "label": "Data_importu"
                                            },
                                            {
                                                "name": "ID_platnosci_Fakturownia",
                                                "type": "text",
                                                "label": "ID_platnosci_Fakturownia"
                                            }
                                        ]
                                    }
                                }
                            ]
                        },
                        {
                            "flow": [
                                {
                                    "id": 164,
                                    "module": "slack:CreateMessage",
                                    "version": 4,
                                    "parameters": {
                                        "__IMTCONN__": 8853058
                                    },
                                    "filter": {
                                        "name": "Duplikat znaleziony",
                                        "conditions": [
                                            [
                                                {
                                                    "a": "{{162.__IMTLENGTH__}}",
                                                    "b": "0",
                                                    "o": "number:greater"
                                                }
                                            ],
                                            [
                                                {
                                                    "a": "{{162.__IMTLENGTH__}}",
                                                    "b": "0",
                                                    "o": "text:notequal"
                                                }
                                            ]
                                        ]
                                    },
                                    "mapper": {
                                        "parse": false,
                                        "blocks": "{\n  \"blocks\": [\n    {\n      \"type\": \"section\",\n      \"text\": {\n        \"type\": \"mrkdwn\",\n        \"text\": \"⚠️ *DUPLIKAT FAKTURY WYKRYTY!*\"\n      }\n    },\n    {\n      \"type\": \"section\",\n      \"fields\": [\n        {\n          \"type\": \"mrkdwn\",\n          \"text\": \"*Numer FV:*\\n{{71.invoice_number}}\"\n        },\n        {\n          \"type\": \"mrkdwn\",\n          \"text\": \"*Dostawca:*\\n{{71.supplier}}\"\n        }\n      ]\n    },\n    {\n      \"type\": \"section\",\n      \"fields\": [\n        {\n          \"type\": \"mrkdwn\",\n          \"text\": \"*Kwota brutto:*\\n{{71.gross_amount}} PLN\"\n        },\n        {\n          \"type\": \"mrkdwn\",\n          \"text\": \"*Data wystawienia:*\\n{{71.issue_date}}\"\n        }\n      ]\n    },\n    {\n      \"type\": \"section\",\n      \"text\": {\n        \"type\": \"mrkdwn\",\n        \"text\": \"📧 *Email:* {{1.subject}}\\n❌ Faktura nie została dodana do systemu.\"\n      }\n    }\n  ]\n}",
                                        "mrkdwn": true,
                                        "channel": "C08V6TJ9K2T",
                                        "channelType": "public",
                                        "channelWType": "list"
                                    },
                                    "metadata": {
                                        "designer": {
                                            "x": 2017,
                                            "y": 620
                                        },
                                        "restore": {
                                            "expect": {
                                                "parse": {
                                                    "mode": "chose"
                                                },
                                                "mrkdwn": {
                                                    "mode": "chose"
                                                },
                                                "channel": {
                                                    "mode": "chose",
                                                    "label": "faktury"
                                                },
                                                "link_names": {
                                                    "mode": "chose"
                                                },
                                                "channelType": {
                                                    "label": "Public channel"
                                                },
                                                "channelWType": {
                                                    "label": "Select from the list"
                                                },
                                                "unfurl_links": {
                                                    "mode": "chose"
                                                },
                                                "unfurl_media": {
                                                    "mode": "chose"
                                                },
                                                "reply_broadcast": {
                                                    "mode": "chose"
                                                }
                                            },
                                            "parameters": {
                                                "__IMTCONN__": {
                                                    "data": {
                                                        "scoped": "true",
                                                        "connection": "slack2"
                                                    },
                                                    "label": "dotykowewlaczniki@gmail.com (dcsmarthomeworkspace.slack.com - dotykowewlaczniki)"
                                                }
                                            }
                                        },
                                        "parameters": [
                                            {
                                                "name": "__IMTCONN__",
                                                "type": "account:slack2,slack3",
                                                "label": "Connection",
                                                "required": true
                                            }
                                        ],
                                        "expect": [
                                            {
                                                "name": "channelWType",
                                                "type": "select",
                                                "label": "Enter a channel ID or name",
                                                "required": true,
                                                "validate": {
                                                    "enum": [
                                                        "manualy",
                                                        "list"
                                                    ]
                                                }
                                            },
                                            {
                                                "name": "text",
                                                "type": "text",
                                                "label": "Text"
                                            },
                                            {
                                                "name": "blocks",
                                                "type": "text",
                                                "label": "Blocks"
                                            },
                                            {
                                                "name": "thread_ts",
                                                "type": "text",
                                                "label": "Thread message ID (time stamp)"
                                            },
                                            {
                                                "name": "reply_broadcast",
                                                "type": "boolean",
                                                "label": "Reply broadcast"
                                            },
                                            {
                                                "name": "link_names",
                                                "type": "boolean",
                                                "label": "Link names"
                                            },
                                            {
                                                "name": "parse",
                                                "type": "boolean",
                                                "label": "Parse message text"
                                            },
                                            {
                                                "name": "mrkdwn",
                                                "type": "boolean",
                                                "label": "Use markdown"
                                            },
                                            {
                                                "name": "unfurl_links",
                                                "type": "boolean",
                                                "label": "Unfurl primarily text-based content"
                                            },
                                            {
                                                "name": "unfurl_media",
                                                "type": "boolean",
                                                "label": "Unfurl media content"
                                            },
                                            {
                                                "name": "icon_emoji",
                                                "type": "text",
                                                "label": "Icon emoji"
                                            },
                                            {
                                                "name": "icon_url",
                                                "type": "url",
                                                "label": "Icon url"
                                            },
                                            {
                                                "name": "username",
                                                "type": "text",
                                                "label": "User name"
                                            },
                                            {
                                                "name": "channelType",
                                                "type": "select",
                                                "label": "Channel type",
                                                "required": true,
                                                "validate": {
                                                    "enum": [
                                                        "public",
                                                        "private",
                                                        "im",
                                                        "mpim"
                                                    ]
                                                }
                                            },
                                            {
                                                "name": "channel",
                                                "type": "select",
                                                "label": "Public channel",
                                                "required": true
                                            }
                                        ],
                                        "advanced": true
                                    }
                                }
                            ]
                        }
                    ]
                }
            ]
        }
    ],
    "metadata": {
        "version": 1
    }
}
