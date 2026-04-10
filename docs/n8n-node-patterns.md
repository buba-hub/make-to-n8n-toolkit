# Patterns de configuration des nodes n8n

Reference extraite des workflows existants dans l'instance n8n.
**TOUJOURS** consulter ce fichier avant de creer un workflow via MCP.

## Regles generales

- Les IDs de nodes doivent etre des strings descriptifs (ex: `"webhook-1"`, `"get-record"`, `"if-resolue"`)
- Les positions suivent une grille de 250px environ entre chaque node
- Toujours ajouter `"options": {}` quand le node le supporte
- Utiliser `__rl: true` pour les champs Airtable base/table
- Credential keys doivent correspondre exactement au type de credential

## Nodes — Configuration de reference

### Webhook (declencheur)
```json
{
  "parameters": {
    "httpMethod": "POST",
    "path": "mon-webhook-path",
    "options": {}
  },
  "id": "webhook-1",
  "name": "Webhook",
  "type": "n8n-nodes-base.webhook",
  "typeVersion": 2,
  "position": [0, 0]
}
```

### Manual Trigger
```json
{
  "parameters": {},
  "id": "manual-trigger",
  "name": "When clicking 'Execute workflow'",
  "type": "n8n-nodes-base.manualTrigger",
  "typeVersion": 1,
  "position": [0, -200]
}
```

### Airtable — Get Record
```json
{
  "parameters": {
    "base": {
      "__rl": true,
      "value": "appXXXXXXXXXXXXX",
      "mode": "id"
    },
    "table": {
      "__rl": true,
      "value": "tblXXXXXXXXXXXXX",
      "mode": "id"
    },
    "id": "={{ $json.body.id_airtable }}",
    "options": {}
  },
  "id": "get-record",
  "name": "Get Record",
  "type": "n8n-nodes-base.airtable",
  "typeVersion": 2.1,
  "position": [250, 0],
  "credentials": {
    "airtableTokenApi": {
      "id": "YOUR_AIRTABLE_CREDENTIAL_ID",
      "name": "Airtable Personal Access Token account"
    }
  }
}
```

### Airtable — Search Records
```json
{
  "parameters": {
    "operation": "search",
    "base": {
      "__rl": true,
      "value": "appXXXXXXXXXXXXX",
      "mode": "id"
    },
    "table": {
      "__rl": true,
      "value": "tblXXXXXXXXXXXXX",
      "mode": "id"
    },
    "options": {}
  },
  "id": "search-records",
  "name": "Search Records",
  "type": "n8n-nodes-base.airtable",
  "typeVersion": 2.1,
  "position": [250, 0],
  "credentials": {
    "airtableTokenApi": {
      "id": "YOUR_AIRTABLE_CREDENTIAL_ID",
      "name": "Airtable Personal Access Token account"
    }
  }
}
```

### Airtable — Create Record
```json
{
  "parameters": {
    "operation": "create",
    "base": {
      "__rl": true,
      "value": "appXXXXXXXXXXXXX",
      "mode": "id"
    },
    "table": {
      "__rl": true,
      "value": "tblXXXXXXXXXXXXX",
      "mode": "id"
    },
    "columns": {
      "mappingMode": "defineBelow",
      "value": {
        "field1": "value1",
        "field2": "={{ $json.someField }}"
      }
    },
    "options": {}
  },
  "id": "create-record",
  "name": "Create Record",
  "type": "n8n-nodes-base.airtable",
  "typeVersion": 2.1,
  "position": [250, 0],
  "credentials": {
    "airtableTokenApi": {
      "id": "YOUR_AIRTABLE_CREDENTIAL_ID",
      "name": "Airtable Personal Access Token account"
    }
  }
}
```

### Airtable — Update Record
```json
{
  "parameters": {
    "operation": "update",
    "base": {
      "__rl": true,
      "value": "appXXXXXXXXXXXXX",
      "mode": "id"
    },
    "table": {
      "__rl": true,
      "value": "tblXXXXXXXXXXXXX",
      "mode": "id"
    },
    "id": "={{ $json.id }}",
    "columns": {
      "mappingMode": "defineBelow",
      "value": {
        "field1": "new value"
      }
    },
    "options": {}
  },
  "id": "update-record",
  "name": "Update Record",
  "type": "n8n-nodes-base.airtable",
  "typeVersion": 2.1,
  "position": [250, 0],
  "credentials": {
    "airtableTokenApi": {
      "id": "YOUR_AIRTABLE_CREDENTIAL_ID",
      "name": "Airtable Personal Access Token account"
    }
  }
}
```

### FTP/SFTP — Upload
**ATTENTION** : Il n'y a PAS de node `n8n-nodes-base.sftp`. Utiliser `n8n-nodes-base.ftp` avec `protocol: "sftp"`.
```json
{
  "parameters": {
    "protocol": "sftp",
    "operation": "upload",
    "path": "/chemin/distant/fichier.csv",
    "options": {}
  },
  "id": "sftp-upload",
  "name": "SFTP Upload",
  "type": "n8n-nodes-base.ftp",
  "typeVersion": 1,
  "position": [250, 0],
  "credentials": {
    "sftp": {
      "id": "YOUR_SFTP_CREDENTIAL_ID",
      "name": "SFTP account"
    }
  },
  "retryOnFail": true,
  "maxTries": 3,
  "waitBetweenTries": 5000
}
```

### FTP/SFTP — List
```json
{
  "parameters": {
    "protocol": "sftp",
    "operation": "list",
    "path": "/chemin/distant/",
    "options": {}
  },
  "id": "sftp-list",
  "name": "SFTP List",
  "type": "n8n-nodes-base.ftp",
  "typeVersion": 1,
  "position": [250, 0],
  "credentials": {
    "sftp": {
      "id": "YOUR_SFTP_CREDENTIAL_ID",
      "name": "SFTP account"
    }
  }
}
```

### HTTP Request
```json
{
  "parameters": {
    "method": "GET",
    "url": "https://api.example.com/endpoint",
    "authentication": "genericCredentialType",
    "genericAuthType": "httpBasicAuth",
    "options": {}
  },
  "id": "http-request",
  "name": "HTTP Request",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "position": [250, 0],
  "credentials": {
    "httpBasicAuth": {
      "id": "YOUR_CREDENTIAL_ID",
      "name": "Mon credential"
    }
  }
}
```

### HTTP Request — avec Headers custom (sans credential)
```json
{
  "parameters": {
    "method": "GET",
    "url": "https://api.example.com/endpoint",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        { "name": "Header-Name", "value": "header-value" }
      ]
    },
    "options": {}
  },
  "id": "http-request",
  "name": "HTTP Request",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "position": [250, 0]
}
```

### If (condition)
```json
{
  "parameters": {
    "conditions": {
      "options": {
        "caseSensitive": true,
        "leftValue": "",
        "typeValidation": "strict",
        "version": 2
      },
      "conditions": [
        {
          "id": "cond-1",
          "leftValue": "={{ $json.someField }}",
          "rightValue": "expected",
          "operator": {
            "type": "string",
            "operation": "equals"
          }
        }
      ],
      "combinator": "and"
    },
    "options": {}
  },
  "id": "if-check",
  "name": "If Check",
  "type": "n8n-nodes-base.if",
  "typeVersion": 2.2,
  "position": [250, 0]
}
```

### Switch
```json
{
  "parameters": {
    "rules": {
      "values": [
        {
          "conditions": {
            "conditions": [
              {
                "leftValue": "={{ $json.field }}",
                "rightValue": "value1",
                "operator": {
                  "type": "string",
                  "operation": "equals"
                }
              }
            ]
          },
          "renameOutput": true,
          "outputKey": "Route 1"
        }
      ]
    },
    "options": {
      "fallbackOutput": "extra"
    }
  },
  "id": "switch-1",
  "name": "Switch",
  "type": "n8n-nodes-base.switch",
  "typeVersion": 3.2,
  "position": [250, 0]
}
```

### Code (JavaScript)
```json
{
  "parameters": {
    "jsCode": "// Code JavaScript ici\nconst items = $input.all();\n// ... traitement ...\nreturn items;"
  },
  "id": "code-1",
  "name": "Code",
  "type": "n8n-nodes-base.code",
  "typeVersion": 2,
  "position": [250, 0]
}
```

### Slack — Send Message
```json
{
  "parameters": {
    "select": "channel",
    "channelId": {
      "__rl": true,
      "value": "YOUR_CHANNEL_ID",
      "mode": "id"
    },
    "text": "Mon message Slack",
    "otherOptions": {}
  },
  "id": "slack-1",
  "name": "Slack",
  "type": "n8n-nodes-base.slack",
  "typeVersion": 2.2,
  "position": [250, 0],
  "credentials": {
    "slackApi": {
      "id": "YOUR_SLACK_CREDENTIAL_ID",
      "name": "Slack account"
    }
  }
}
```

### Set (Edit Fields)
```json
{
  "parameters": {
    "mode": "manual",
    "duplicateItem": false,
    "assignments": {
      "assignments": [
        {
          "id": "field-1",
          "name": "myField",
          "value": "={{ $json.sourceField }}",
          "type": "string"
        }
      ]
    },
    "options": {}
  },
  "id": "set-1",
  "name": "Set Fields",
  "type": "n8n-nodes-base.set",
  "typeVersion": 3.4,
  "position": [250, 0]
}
```

### Merge
```json
{
  "parameters": {
    "mode": "combine",
    "combineBy": "combineAll",
    "options": {}
  },
  "id": "merge-1",
  "name": "Merge",
  "type": "n8n-nodes-base.merge",
  "typeVersion": 3.1,
  "position": [250, 0]
}
```

## Credentials disponibles (IDs)

> **IMPORTANT** : Remplacer les placeholders `YOUR_*_CREDENTIAL_ID` par les vrais IDs de votre instance n8n.
> Lister vos credentials via : `GET /api/v1/credentials` ou via MCP n8n (`n8n_manage_credentials`).

| Service | ID | Nom | Type | Credentials key |
|---------|-----|------|------|----------------|
| Airtable | `a_completer` | Airtable Personal Access Token account | airtableTokenApi | `airtableTokenApi` |
| SFTP | `a_completer` | SFTP account | sftp | `sftp` |
| Slack | `a_completer` | Slack account | slackApi | `slackApi` |
| Freshdesk | `a_completer` | Freshdesk | httpBasicAuth | `httpBasicAuth` |
| Gmail | `a_completer` | Gmail account | gmailOAuth2 | `gmailOAuth2` |
| Notion | `a_completer` | Notion account | notionApi | `notionApi` |
| Google Sheets | `a_completer` | Google Sheets account | googleSheetsOAuth2Api | `googleSheetsOAuth2Api` |
| Google Calendar | `a_completer` | Google Calendar | googleCalendarOAuth2Api | `googleCalendarOAuth2Api` |

## Workflow settings recommandes
```json
{
  "executionOrder": "v1",
  "saveDataErrorExecution": "all",
  "saveDataSuccessExecution": "all",
  "saveManualExecutions": true
}
```
