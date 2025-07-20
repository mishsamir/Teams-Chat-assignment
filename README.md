# Teams-Chat-assignment



# Microsoft Teams Content Moderation System

## 📋 Overview

Automated Microsoft Teams chat monitoring using **Azure Logic Apps**. Detects inappropriate content in real-time and sends email alerts to administrators.

**Tech Stack**: Azure Logic Apps • Microsoft Teams Connector • Office 365 Outlook

---

## 🏗️ Implementation

### Trigger
- **Microsoft Teams**: "When a new message is added to a chat or channel"
- **Source**: Group Chat monitoring

### Workflow
1. **Get Message Details** - Retrieves full message content via Teams API
2. **Content Check** - Evaluates for banned words (e.g., "sh*t")
3. **Send Alert** - Email notification if violation detected
4. **End** - Silent processing for clean messages

---

## 📊 Logic App Flow

<img width="1907" height="959" alt="Screenshot 2025-07-20 at 5 07 31 PM" src="https://github.com/user-attachments/assets/69516263-3c30-428f-8977-ca8e79ec8350" />

---

## 🧪 Testing Results

**✅ Clean Message**: `"Hello, good morning"`
- Result: No alert sent ✓

**🚨 Violation**: `"this is sh*t!"`  
- Result: Email alert received within seconds ✓

---

## 📧 Email Alert Sample

<img width="1907" height="959" alt="Screenshot 2025-07-20 at 5 08 00 PM" src="https://github.com/user-attachments/assets/6cbe7064-65a9-4856-9a29-e473fdf360ba" />

---

## 📈 Run History

<img width="1912" height="1080" alt="Screenshot 2025-07-20 at 5 11 43 PM" src="https://github.com/user-attachments/assets/194a86c7-e3ed-457f-ba02-c95b29037023" />

---

## 🚧 Key Challenge & Solution

**Problem**: Teams trigger doesn't provide message content directly

**Solution**: Added "Get message details" action to retrieve full message using message ID from trigger

---

## 🚀 Features

- Real-time message monitoring
- Instant email notifications
- Automated content filtering
- Administrator alerts with full context


### Logic app designer Code

```json
{
    "definition": {
        "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
        "contentVersion": "1.0.0.0",
        "triggers": {
            "When_a_new_message_is_added_to_a_chat_or_channel": {
                "type": "ApiConnectionWebhook",
                "inputs": {
                    "host": {
                        "connection": {
                            "name": "@parameters('$connections')['teams']['connectionId']"
                        }
                    },
                    "body": {
                        "notificationUrl": "@listCallbackUrl()",
                        "chats": [
                            "19:8298d7f29d244001af954d64c9275dd1@thread.v2"
                        ]
                    },
                    "path": "/beta/subscriptions/newmessagetrigger/threadType/@{encodeURIComponent('groupchat')}"
                }
            }
        },
        "actions": {
            "For_each": {
                "type": "Foreach",
                "foreach": "@triggerBody()?['value']",
                "actions": {
                    "Get_message_details": {
                        "type": "ApiConnection",
                        "inputs": {
                            "host": {
                                "connection": {
                                    "name": "@parameters('$connections')['teams']['connectionId']"
                                }
                            },
                            "method": "post",
                            "body": {
                                "recipient": "19:8298d7f29d244001af954d64c9275dd1@thread.v2"
                            },
                            "path": "/beta/teams/messages/@{encodeURIComponent(item()?['messageId'])}/messageType/@{encodeURIComponent('groupchat')}"
                        }
                    },
                    "Condition": {
                        "type": "If",
                        "expression": {
                            "or": [
                                {
                                    "contains": [
                                        "@body('Get_message_details')?['body']?['content']",
                                        "shit"
                                    ]
                                }
                            ]
                        },
                        "actions": {
                            "Send_email_(V2)": {
                                "type": "ApiConnection",
                                "inputs": {
                                    "host": {
                                        "connection": {
                                            "name": "@parameters('$connections')['gmail']['connectionId']"
                                        }
                                    },
                                    "method": "post",
                                    "body": {
                                        "To": "mishrasamir739@gmail.com",
                                        "Importance": "High",
                                        "Subject": "⚠️ Teams Chat Words Violation",
                                        "Body": "<p class=\"editor-paragraph\"><b><strong class=\"editor-text-bold\">‼️ This message contains offensive words.</strong></b></p><br><p class=\"editor-paragraph\">-Message: @{body('Get_message_details')?['body']?['content']}</p><p class=\"editor-paragraph\">-From: @{body('Get_message_details')?['from']?['user']?['displayName']}</p><p class=\"editor-paragraph\">-Time: @{body('Get_message_details')?['lastModifiedDateTime']}</p>"
                                    },
                                    "path": "/v2/Mail"
                                }
                            }
                        },
                        "else": {
                            "actions": {}
                        },
                        "runAfter": {
                            "Get_message_details": [
                                "Succeeded"
                            ]
                        }
                    }
                },
                "runAfter": {}
            }
        },
        "outputs": {},
        "parameters": {
            "$connections": {
                "type": "Object",
                "defaultValue": {}
            }
        }
    },
    "parameters": {
        "$connections": {
            "type": "Object",
            "value": {
                "teams": {
                    "id": "/subscriptions/5e942409-9895-49e8-b6c2-3b2dfa341205/providers/Microsoft.Web/locations/canadacentral/managedApis/teams",
                    "connectionId": "/subscriptions/5e942409-9895-49e8-b6c2-3b2dfa341205/resourceGroups/teams-assignment/providers/Microsoft.Web/connections/teams",
                    "connectionName": "teams"
                },
                "gmail": {
                    "id": "/subscriptions/5e942409-9895-49e8-b6c2-3b2dfa341205/providers/Microsoft.Web/locations/canadacentral/managedApis/gmail",
                    "connectionId": "/subscriptions/5e942409-9895-49e8-b6c2-3b2dfa341205/resourceGroups/teams-assignment/providers/Microsoft.Web/connections/gmail",
                    "connectionName": "gmail"
                }
            }
        }
    }
}
```

### Demo Video
[Click Here](https://youtu.be/e08tpj-L_c0)
