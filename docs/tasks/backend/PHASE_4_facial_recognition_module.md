# TASK: Backend — Phase 4 — Facial Recognition Module

**Phase:** 4 (Advanced)  
**Module:** Security / AI  
**Priority:** 🟢 Advanced  
**Estimate:** 3 days  
**Depends on:** PHASE_2_owner_tenant_module  

---

## Objective

Allow residents to optionally enroll facial data for contactless entry. Guards' station cameras recognize faces and display resident info automatically.

---

## Endpoints

| Method | Route | Role | Description |
|--------|-------|------|-------------|
| POST | `/api/v1/users/face-enrollment` | Owner, Tenant | Enroll face (upload photos) |
| DELETE | `/api/v1/users/face-enrollment` | Owner, Tenant | Remove face enrollment |
| GET | `/api/v1/users/face-enrollment/status` | Owner, Tenant | Check enrollment status |
| POST | `/api/v1/complexes/{cid}/face-recognition/identify` | Guard | Identify person from camera image |

---

## DTOs

### EnrollFaceDto
```csharp
public record EnrollFaceDto(
    List<string> PhotoUrls  // 3-5 photos uploaded to Blob; Azure Face API requires URLs
);
```

### FaceEnrollmentStatusDto
```csharp
public record FaceEnrollmentStatusDto(
    bool IsEnrolled,
    DateTimeOffset? EnrolledAt,
    int PhotoCount
);
```

### IdentifyFaceDto
```csharp
public record IdentifyFaceDto(
    string ImageUrl  // URL of frame captured by guard station camera
);
```

### FaceIdentificationResultDto
```csharp
public record FaceIdentificationResultDto(
    bool Identified,
    double? Confidence,        // 0.0 - 1.0
    string? ResidentName,
    string? PhotoUrl,
    string? ApartmentNumber,
    string? ResidentType,      // "owner"|"tenant"
    bool IsAuthorized,
    string Status              // "identified"|"not_recognized"|"low_confidence"
);
```

---

## Azure Face API Integration

```csharp
// Infrastructure/Services/FacialRecognitionService.cs
public class FacialRecognitionService : IFacialRecognitionService
{
    private readonly FaceClient _faceClient;
    private readonly string _personGroupId;  // Per complex: "complex-{complexId}"

    public async Task<string> EnrollFaceAsync(Guid userId, List<string> photoUrls)
    {
        // 1. Ensure PersonGroup exists for the complex
        await EnsurePersonGroupAsync(_personGroupId);
        
        // 2. Create Person in the group
        var person = await _faceClient.PersonGroupPerson.CreateAsync(
            _personGroupId, name: userId.ToString());
        
        // 3. Add photos to the person
        foreach (var url in photoUrls)
        {
            await _faceClient.PersonGroupPerson.AddFaceFromUrlAsync(
                _personGroupId, person.PersonId, url);
        }
        
        // 4. Train the group (async, takes seconds)
        await _faceClient.PersonGroup.TrainAsync(_personGroupId);
        
        return person.PersonId.ToString();
    }

    public async Task<FaceIdentificationResult> IdentifyAsync(string complexId, string imageUrl)
    {
        // 1. Detect faces in the image
        var detectedFaces = await _faceClient.Face.DetectWithUrlAsync(imageUrl);
        if (!detectedFaces.Any())
            return new(Status: "not_recognized");
        
        // 2. Identify against the complex's person group
        var faceIds = detectedFaces.Select(f => f.FaceId!.Value).ToList();
        var results = await _faceClient.Face.IdentifyAsync(
            faceIds, _personGroupId, confidenceThreshold: 0.75);
        
        var topResult = results.FirstOrDefault()?.Candidates.FirstOrDefault();
        if (topResult is null)
            return new(Status: "not_recognized");
        
        return new(
            Identified: true,
            Confidence: topResult.Confidence,
            PersonId: topResult.PersonId.ToString()
        );
    }

    public async Task RemoveFaceAsync(string faceId, string personId)
    {
        await _faceClient.PersonGroupPerson.DeleteAsync(_personGroupId, Guid.Parse(personId));
    }
}
```

---

## Enrollment Flow

```
1. Resident takes 3-5 photos via app camera
2. Photos uploaded to Azure Blob (POST /media/upload)
3. Resident calls POST /face-enrollment with photo URLs
4. Backend:
   a. Calls Azure Face API to create person and add faces
   b. Stores face_enrollment record with azure person_id
   c. Triggers PersonGroup training (async)
5. Enrollment status → "enrolled"
```

---

## Guard Station Identification Flow

```
1. Camera at entrance captures frame → uploads to Blob
2. Guard station calls POST /face-recognition/identify with image URL
3. Backend:
   a. Detects face in image via Azure Face API
   b. Identifies against complex's PersonGroup
   c. Maps PersonId → face_enrollment → owner/tenant record
   d. Returns resident info + authorization status
4. Guard station displays: resident photo, name, apartment, AUTHORIZED/DENIED
5. Access log entry created automatically
```

---

## Privacy & Compliance

- Facial enrollment is **100% optional** for residents
- Clear opt-in consent required before enrollment (acknowledged in API via consent flag)
- Residents can delete their enrollment at any time (data removed from Azure Face API)
- Face data never stored locally; only Azure Person IDs stored in DB
- Privacy notice must be shown in UI before enrollment screen
- Comply with Colombia's Habeas Data law (Ley 1581 de 2012)

---

## PersonGroup Management

- One PersonGroup per complex: `complex-{complexId}`
- PersonGroup training is triggered:
  1. After each new enrollment
  2. After each enrollment deletion
  3. Nightly (re-training job, Hangfire)

```csharp
// Jobs/FacePersonGroupTrainingJob.cs (nightly)
var complexIds = await _uow.Complexes.GetAllActiveIdsAsync();
foreach (var cid in complexIds)
{
    await _facialRecognitionService.TrainGroupAsync($"complex-{cid}");
}
```

---

## Business Rules

- Minimum 3 photos required for enrollment
- Maximum 5 photos per enrollment
- Photos must meet Azure Face API quality requirements (face clearly visible, well-lit)
- If recognition confidence < 0.75 → "low_confidence" status → manual verification by guard
- Enrollment removal → immediate (cannot enter without physical ID until re-enrolled)

---

## Acceptance Criteria

- [ ] Resident can enroll face via photo upload
- [ ] Enrollment stored in Azure Face API + DB record
- [ ] Identification returns resident info for enrolled faces
- [ ] Unknown faces return "not_recognized" status
- [ ] Low confidence (< 0.75) returns "low_confidence" → guard prompted for manual verification
- [ ] Removing enrollment deletes data from Azure Face API
- [ ] Access log entry created on every identification attempt
- [ ] Consent flag required before enrollment accepted

---

## Unit Tests

- [ ] `FacialRecognitionService.EnrollFaceAsync` — maps response to face enrollment record
- [ ] `IdentifyFaceCommandHandler` — creates access log entry

---

## Integration Tests

- [ ] POST `/face-enrollment` → 201 with enrollment status
- [ ] DELETE `/face-enrollment` → 204, record removed
- [ ] POST `/face-recognition/identify` (unknown image) → 200 with not_recognized status
