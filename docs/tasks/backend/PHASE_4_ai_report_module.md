# TASK: Backend — Phase 4 — AI Emergency Report Module

**Phase:** 4 (Advanced)  
**Module:** Incidents / AI  
**Priority:** 🟢 Advanced  
**Estimate:** 2 days  
**Depends on:** PHASE_4_ai_bot_module, PHASE_3_notification_module  

---

## Objective

Build an AI-assisted conversational emergency report generator. Residents describe an incident in chat; AI formulates a structured report for the relevant authority and notifies the admin.

---

## Endpoints

| Method | Route | Role | Description |
|--------|-------|------|-------------|
| POST | `/api/v1/emergency/chat` | Auth | Send message in emergency chat |
| POST | `/api/v1/emergency/generate-report` | Auth | Generate structured report from conversation |
| GET | `/api/v1/complexes/{cid}/emergency-reports` | Admin | List emergency reports |
| GET | `/api/v1/emergency-reports/{id}` | Admin, Reporter | Get report detail |

---

## DTOs

### EmergencyChatDto
```csharp
public record EmergencyChatDto(
    EmergencyType EmergencyType, // fire|police|ambulance|power|water|gas
    string Message,              // Required
    List<ConversationMessageDto> History  // Previous messages for context
);

public enum EmergencyType { Fire, Police, Ambulance, Power, Water, Gas }

public record ConversationMessageDto(string Role, string Content); // role: user|assistant
```

### EmergencyChatResponseDto
```csharp
public record EmergencyChatResponseDto(
    string Message,           // Bot's reply (next question)
    bool IsReadyToReport,     // True when bot has enough info
    string? EmergencyPhone    // Phone number for the authority
);
```

### GenerateReportDto
```csharp
public record GenerateReportDto(
    EmergencyType EmergencyType,
    List<ConversationMessageDto> Conversation
);

public record GenerateReportResponseDto(
    Guid ReportId,
    string FormattedReport,   // Plain text, ready to read to operator
    string AuthorityName,     // "Bomberos", "Policía", etc.
    string AuthorityPhone,
    string ReportSummary      // One-line summary for admin notification
);
```

---

## Emergency Type Configuration

```csharp
public static class EmergencyConfig
{
    public static readonly Dictionary<EmergencyType, EmergencyAuthority> Authorities = new()
    {
        [EmergencyType.Fire]      = new("Bomberos", "119"),
        [EmergencyType.Police]    = new("Policía Nacional", "123"),
        [EmergencyType.Ambulance] = new("Ambulancias", "125"),
        [EmergencyType.Power]     = new("EPM / Empresa de Energía", "115"),
        [EmergencyType.Water]     = new("Acueducto y Alcantarillado", "116"),
        [EmergencyType.Gas]       = new("Gas Natural", "164"),
    };
}
```

---

## Conversational AI Flow

### System Prompt (per emergency type)

```csharp
// Fire example:
var systemPrompt = $"""
    You are an emergency assistant for a residential complex.
    The resident is reporting a FIRE emergency.
    Your job is to collect all necessary information to generate a report for the fire department.
    
    Ask CONCISE questions to collect:
    - Exact location (building, floor, apartment)
    - What is burning / what they can see
    - Are there any injuries? How many people need help?
    - Is the building being evacuated?
    - Has 119 been called already?
    
    Current complex: {complexName}
    Address: {complexAddress}
    Reporting resident: {residentName}, Apartment {apartmentNumber}
    
    Language: Respond in the same language as the user.
    When you have enough information, end your message with: [READY_TO_REPORT]
    """;
```

### Report Generation Prompt

```csharp
var reportPrompt = $"""
    Based on the following emergency conversation, generate a structured emergency report 
    for the fire department (Bomberos). The report should be:
    - Clear and concise
    - Include all key facts
    - Formatted as if being read over the phone to the operator
    
    Complex: {complexName}
    Address: {complexAddress}
    
    Conversation:
    {conversationText}
    
    Generate the report in Spanish.
    """;
```

---

## Incident Report Module (Manual Reports)

**Also included in this task:**

| Method | Route | Role | Description |
|--------|-------|------|-------------|
| GET | `/api/v1/complexes/{cid}/incidents` | Admin | List incidents (paginated) |
| POST | `/api/v1/complexes/{cid}/incidents` | Auth | Submit incident report |
| GET | `/api/v1/incidents/{id}` | Admin, Reporter | Get incident |
| PUT | `/api/v1/incidents/{id}/assign` | Admin | Assign to staff |
| PUT | `/api/v1/incidents/{id}/status` | Admin | Update status |

**AI Triage:**
```csharp
// After incident submission, call Azure OpenAI:
var prompt = $"""
    Categorize the priority of this incident report for a residential complex:
    Category: {incident.Category}
    Description: {incident.Description}
    
    Return one of: Critical, High, Medium, Low
    And a brief (1-sentence) reason.
    """;
```

---

## Acceptance Criteria

- [ ] Emergency chat collects incident info conversationally
- [ ] Bot detects when it has enough info ([READY_TO_REPORT])
- [ ] Report generation produces clear, structured text
- [ ] Report saved to DB and admin notified instantly
- [ ] Emergency phone number shown to resident in UI
- [ ] Manual incident reports triaged by AI with priority
- [ ] Admin can update incident status and assignee

---

## Unit Tests

- [ ] Emergency type → authority name/phone mapping
- [ ] Report generation prompt format
- [ ] `EmergencyReportService` — saves report and triggers notification

---

## Integration Tests

- [ ] POST `/emergency/chat` → bot responds with follow-up question
- [ ] POST `/emergency/generate-report` → returns structured report
- [ ] GET `/complexes/{cid}/emergency-reports` → paginated list
