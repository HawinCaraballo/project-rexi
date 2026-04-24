# TASK: Frontend — Phase 4 — AI Chat Bot Screen

**Phase:** 4 (Advanced)  
**Module:** AI / Documents  
**Priority:** 🟢 Advanced  
**Estimate:** 2 days  
**Depends on:** PHASE_1_design_system  

---

## Objective

Build the AI document Q&A chatbot interface for residents, and the document upload management screen for admins.

---

## Routes

| Route | Role | Description |
|-------|------|-------------|
| `/documents` | All | Documents + AI bot |
| `/documents/manage` | Admin | Upload/manage documents |

---

## Documents + Bot Page (`/documents`)

**Layout:** Split panel

**Left panel (35%): Document Library**
- List of uploaded complex documents
- Each item: file icon, name, upload date, download button
- Search input for documents by name

**Right panel (65%): AI Chat**

```
┌─────────────────────────────────────────────────────────────┐
│  🤖 Rexi Assistant                              [ES] [EN]   │
│  ─────────────────────────────────────────────────────────  │
│                                                             │
│  Hola! Soy Rexi. Puedo responder preguntas                 │
│  sobre las normas y documentos del conjunto.               │
│  ¿En qué te puedo ayudar?                     [Bot bubble] │
│                                                             │
│  ¿Puedo tener mascotas en el apartamento?    [User bubble] │
│                                                             │
│  Según el Reglamento de Convivencia           [Bot bubble] │
│  (sección 4.2), se permite tener mascotas                  │
│  siempre que el peso no supere 15 kg...                    │
│  📄 Fuente: Reglamento Convivencia v2.pdf                  │
│                                                             │
│  [No te ayudé? Contactar administración]                   │
│  ─────────────────────────────────────────────────────────  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Escribe tu pregunta...                        [Send]│  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

**Mobile:** Stack vertically (document list collapses to an expandable bottom sheet)

---

## Chat UI Details

**Message bubbles:**
- User: right-aligned, primary color background
- Bot: left-aligned, white/surface card with bot avatar
- Typing indicator: animated 3-dot pulse while waiting for response

**Source citation:**
After bot response, if sources exist:
```
📄 Source: Reglamento Convivencia v2.pdf (page 12)
```
Clicking source → opens document or scrolls to document in left panel

**"Escalate to admin" button:**
Small link below bot messages that had low confidence or user marked as unhelpful.

**Language auto-detect:**
Bot detects language from user input and responds in same language.

**Suggested questions (first load):**
- "¿Cuáles son los horarios del gimnasio?"
- "¿Cómo puedo reservar el salón social?"
- "¿Cuál es la política de mascotas?"

---

## Admin: Document Management (`/documents/manage`)

**Layout:** Standard PageLayout + list

**Document list:**
| Column | Notes |
|--------|-------|
| Name | File name, clickable |
| Type | PDF / Word badge |
| Version | v1, v2... |
| Uploaded By | User name |
| Upload Date | Formatted date |
| Actions | Download, Delete |

**Upload button → Slide-over:**
| Field | Validation |
|-------|-----------|
| Document Name | Required, max 200 |
| Description | Optional |
| File | Required, .pdf or .docx, max 20MB |

**After upload:**
- Shows "Indexing..." spinner
- When indexing complete → "Bot ready to answer questions from this document" toast

**Delete document:**
- Confirmation dialog: "Are you sure? The bot will no longer answer questions from this document."

---

## Chat State Management

```typescript
// src/modules/ai/hooks/useAiChat.ts
interface ChatMessage {
  id: string;
  role: 'user' | 'bot';
  content: string;
  sources?: { documentName: string; excerpt: string }[];
  timestamp: Date;
}

export function useAiChat(complexId: string) {
  const [messages, setMessages] = useState<ChatMessage[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  
  const sendMessage = async (text: string) => {
    // Add user message immediately (optimistic)
    // Call POST /ai/chat with message + history
    // Add bot response
    // Handle errors with error bubble
  };
  
  return { messages, isLoading, sendMessage };
}
```

---

## Acceptance Criteria

- [ ] Document list shows all complex documents
- [ ] Bot chat sends message and displays streamed response
- [ ] Typing indicator shown while waiting for response
- [ ] Source citations shown for document-based answers
- [ ] Suggested questions shown on first load (before any message)
- [ ] "Escalate to admin" link works (sends message to admin notification)
- [ ] Language detection works (user writes in English → bot responds in English)
- [ ] Admin can upload documents
- [ ] Indexing status shown after upload
- [ ] Admin can delete documents with confirmation

---

## Component IDs

```
documents-list
documents-search
ai-chat-window
ai-chat-messages
ai-chat-input
ai-chat-send-btn
ai-chat-typing-indicator
ai-chat-source-citation
ai-chat-escalate-btn
ai-suggested-questions
documents-manage-list
documents-upload-btn
documents-upload-form
documents-upload-file
documents-upload-submit
documents-delete-btn
```
