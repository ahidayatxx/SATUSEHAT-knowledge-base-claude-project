# Quick Start Guide: SATUSEHAT EMR Implementation

## 🎯 Objective

Get your EMR transmitting validated FHIR resources to SATUSEHAT in **10 weeks** using this structured roadmap.

---

## ✅ Prerequisites Checklist

Before starting implementation:

- [ ] **SATUSEHAT Platform Account**: Register at [satusehat.kemkes.go.id](https://satusehat.kemkes.go.id/platform)
- [ ] **EMR System Ready**: Core patient registration and clinical documentation modules functional
- [ ] **Development Environment**: Staging/sandbox environment available for testing
- [ ] **NIK Data Quality**: Patient NIKs validated and stored in database
- [ ] **Practitioner Credentials**: All clinicians have valid NIKs recorded
- [ ] **Technical Capacity**: Development team familiar with REST APIs and JSON

---

## 📅 10-Week Implementation Timeline

### **Week 1-2: Foundation Setup**

#### Day 1-3: Authentication Infrastructure
```python
# 1. Obtain SATUSEHAT credentials
# - Login to SATUSEHAT Platform
# - Navigate to "Kode Akses API (Sandbox)"
# - Copy client_id and client_secret

# 2. Implement OAuth2 client
from satusehat_client import SATUSEHATClient

config = {
    "environment": "staging",
    "client_id": "YOUR_CLIENT_ID",
    "client_secret": "YOUR_CLIENT_SECRET"
}

client = SATUSEHATClient(config)
token = client._get_token()  # Should succeed
print(f"Token obtained: {token[:20]}...")
```

**Success Criteria**: Successfully obtain access token from staging API.

#### Day 4-7: Organization Registration
```python
# Register your facility as Organization resource
organization_payload = {
    "resourceType": "Organization",
    "identifier": [{
        "system": "http://sys-ids.kemkes.go.id/organization/{org_id}",
        "value": "YOUR_FASYANKES_CODE"
    }],
    "active": True,
    "type": [{
        "coding": [{
            "system": "http://terminology.hl7.org/CodeSystem/organization-type",
            "code": "prov",
            "display": "Healthcare Provider"
        }]
    }],
    "name": "Your Hospital Name"
}

response = client.create_resource(organization_payload)
org_id = response["id"]

# CRITICAL: Store this org_id - you'll need it for every subsequent resource
print(f"Organization ID: {org_id}")
```

**Success Criteria**: Organization.id obtained and stored in configuration.

#### Day 8-10: Location Registration
```python
# Register at least one location (e.g., outpatient clinic)
location_payload = {
    "resourceType": "Location",
    "identifier": [{
        "system": f"http://sys-ids.kemkes.go.id/location/{org_id}",
        "value": "POLI-UMUM-01"
    }],
    "status": "active",
    "name": "Poliklinik Umum",
    "mode": "instance",
    "physicalType": {
        "coding": [{
            "system": "http://terminology.hl7.org/CodeSystem/location-physical-type",
            "code": "ro",
            "display": "Room"
        }]
    },
    "managingOrganization": {
        "reference": f"Organization/{org_id}"
    }
}

response = client.create_resource(location_payload)
location_id = response["id"]

# Store mapping: internal_location_code -> SATUSEHAT Location.id
db.locations.update("POLI-UMUM-01", {"satusehat_id": location_id})
```

**Success Criteria**: At least 3 locations registered (outpatient, inpatient, emergency).

#### Day 11-14: Practitioner Validation
```python
# For each practitioner in your system:
practitioners = db.practitioners.get_all()

for prac in practitioners:
    # Query SATUSEHAT by NIK
    response = client.search_resource(
        "Practitioner",
        params={"identifier": f"https://fhir.kemkes.go.id/id/nik|{prac.nik}"}
    )
    
    if response.get("total", 0) > 0:
        satusehat_id = response["entry"][0]["resource"]["id"]
        db.practitioners.update(prac.id, {"satusehat_id": satusehat_id})
        print(f"✓ Practitioner {prac.name}: {satusehat_id}")
    else:
        print(f"✗ Practitioner {prac.name}: NIK not found in SATUSEHAT")
        # Register new practitioner if needed
```

**Success Criteria**: 100% of active practitioners have SATUSEHAT Practitioner.id.

---

### **Week 3-4: Patient Integration**

#### Day 15-18: MPI Service Implementation
```python
# Implement Master Patient Index integration
from services.mpi_service import MPIService

mpi = MPIService(client)

# Test with dummy patient from SATUSEHAT
test_nik = "9271060312000001"
patient_ihs = mpi.get_patient_ihs_number(test_nik)

assert patient_ihs == "P02478375538", "MPI lookup failed!"
print(f"✓ MPI integration working: {test_nik} -> {patient_ihs}")
```

#### Day 19-21: Patient Registration Workflow
```python
# Add MPI lookup to patient registration process
def register_patient(nik, name, birth_date, gender):
    # 1. Query MPI first
    patient_ihs = mpi.get_patient_ihs_number(nik)
    
    if not patient_ihs:
        raise ValueError(f"Patient not found in national registry: {nik}")
    
    # 2. Create internal patient record
    patient = db.patients.create({
        "nik": nik,
        "patient_ihs_number": patient_ihs,
        "name": name,
        "birth_date": birth_date,
        "gender": gender
    })
    
    return patient
```

**Success Criteria**: Patient registration automatically resolves NIK to Patient.id.

---

### **Week 5-6: Encounter Management**

#### Day 22-28: Encounter Creation
```python
# Implement encounter creation on patient check-in
def create_outpatient_encounter(patient_nik, practitioner_nik, location_code):
    # Resolve references
    patient = db.patients.get_by_nik(patient_nik)
    practitioner = db.practitioners.get_by_nik(practitioner_nik)
    location = db.locations.get_by_code(location_code)
    
    # Create internal encounter
    encounter = db.encounters.create({
        "patient_ihs_number": patient.patient_ihs_number,
        "practitioner_id": practitioner.satusehat_id,
        "location_id": location.satusehat_id,
        "status": "in-progress",
        "period_start": datetime.now()
    })
    
    # Build FHIR resource
    fhir_encounter = EncounterBuilder(config).build_resource(encounter)
    
    # POST to SATUSEHAT
    response = client.create_resource(fhir_encounter)
    
    # Store SATUSEHAT ID
    db.encounters.update(encounter.id, {
        "satusehat_encounter_id": response["id"],
        "satusehat_sync_status": "synced"
    })
    
    return encounter
```

#### Day 29-35: Encounter Completion
```python
def complete_encounter(encounter_id):
    encounter = db.encounters.get(encounter_id)
    
    # Update status
    db.encounters.update(encounter_id, {
        "status": "finished",
        "period_end": datetime.now()
    })
    
    # Rebuild FHIR resource with updated status
    fhir_encounter = EncounterBuilder(config).build_resource(encounter)
    
    # PUT to update in SATUSEHAT
    client.update_resource(
        "Encounter",
        encounter.satusehat_encounter_id,
        fhir_encounter
    )
```

**Success Criteria**: Successfully create and complete 10 test encounters in staging.

---

### **Week 7: Clinical Documentation**

#### Day 36-40: Diagnosis Recording
```python
def record_diagnosis(encounter_id, icd10_code):
    encounter = db.encounters.get(encounter_id)
    
    # Validate ICD-10 code
    icd10_display = db.icd10_codes.get_display(icd10_code)
    if not icd10_display:
        raise ValueError(f"Invalid ICD-10: {icd10_code}")
    
    # Build Condition resource
    condition = {
        "resourceType": "Condition",
        "clinicalStatus": {
            "coding": [{
                "system": "http://terminology.hl7.org/CodeSystem/condition-clinical",
                "code": "active"
            }]
        },
        "category": [{
            "coding": [{
                "system": "http://terminology.hl7.org/CodeSystem/condition-category",
                "code": "encounter-diagnosis"
            }]
        }],
        "code": {
            "coding": [{
                "system": "http://hl7.org/fhir/sid/icd-10",
                "code": icd10_code,
                "display": icd10_display
            }]
        },
        "subject": {
            "reference": f"Patient/{encounter.patient_ihs_number}"
        },
        "encounter": {
            "reference": f"Encounter/{encounter.satusehat_encounter_id}"
        }
    }
    
    response = client.create_resource(condition)
    return response["id"]
```

#### Day 41-42: Vital Signs (Observation)
```python
def record_vital_signs(encounter_id, vital_signs):
    # vital_signs = {"systolic": 120, "diastolic": 80, "pulse": 72, ...}
    
    for code, value in vital_signs.items():
        observation = build_observation(
            encounter_id=encounter_id,
            loinc_code=VITAL_SIGN_CODES[code],
            value=value,
            unit=VITAL_SIGN_UNITS[code]
        )
        client.create_resource(observation)
```

**Success Criteria**: Successfully transmit diagnoses and vital signs for 10 test encounters.

---

### **Week 8: Medication Workflow**

#### Day 43-49: Prescription Flow
```python
def create_prescription(encounter_id, kfa_code, dosage, quantity):
    # 1. Get/Create Medication resource
    medication_id = ensure_medication_resource(kfa_code)
    
    # 2. Create MedicationRequest
    med_request = {
        "resourceType": "MedicationRequest",
        "status": "active",
        "intent": "order",
        "medicationReference": {"reference": f"Medication/{medication_id}"},
        "subject": {"reference": f"Patient/{encounter.patient_ihs_number}"},
        "encounter": {"reference": f"Encounter/{encounter.satusehat_encounter_id}"},
        "dosageInstruction": [{
            "text": dosage,
            "doseAndRate": [{"doseQuantity": {"value": quantity}}]
        }]
    }
    
    response = client.create_resource(med_request)
    return response["id"]
```

**Success Criteria**: Complete prescription → dispensing flow for 5 test medications.

---

### **Week 9: Testing & Validation**

#### Day 50-54: Staging Environment Testing
```bash
# Test with all 10 dummy patients provided by SATUSEHAT
DUMMY_NIKS = [
    "9271060312000001",  # Ardianto Putra
    "9204014804000002",  # Claudia Sintia
    # ... (see SATUSEHAT documentation for full list)
]

# For each patient:
# 1. Create encounter
# 2. Record diagnosis
# 3. Record vital signs
# 4. Create prescription
# 5. Record dispensing
# 6. Complete encounter

# Verify in SATUSEHAT Dashboard:
# - All resources created successfully
# - No validation errors
# - Sync status = "synced"
```

#### Day 55-56: Error Handling Verification
```python
# Test error scenarios:
# - Invalid ICD-10 code
# - Missing practitioner ID
# - Malformed FHIR resource
# - Network timeout
# - SATUSEHAT API downtime

# Verify:
# - Errors logged properly
# - Failed transmissions queued for retry
# - User workflows not blocked
```

**Success Criteria**: 
- 100 test encounters successfully transmitted
- All validation errors resolved
- Retry logic functioning correctly

---

### **Week 10: Production Pilot**

#### Day 57-59: Production Credential Setup
```python
# Switch to production environment
config = {
    "environment": "production",
    "client_id": "PRODUCTION_CLIENT_ID",
    "client_secret": "PRODUCTION_CLIENT_SECRET",
    "organization_id": "YOUR_PRODUCTION_ORG_ID"
}

# Re-register prerequisites in production:
# - Organization (get production ID)
# - Locations (map to production IDs)
# - Practitioners (verify NIKs in production)
```

#### Day 60-63: Single Clinic Pilot
```python
# Select pilot clinic with:
# - High data quality
# - Tech-savvy staff
# - Manageable patient volume (20-30 patients/day)

# Enable SATUSEHAT sync for pilot clinic only
PILOT_LOCATIONS = ["POLI-UMUM-01"]

def should_sync_to_satusehat(location_code):
    return location_code in PILOT_LOCATIONS
```

#### Day 64-67: Monitoring & Iteration
```python
# Daily review metrics:
# - Sync success rate (target: >99%)
# - Average API latency
# - Validation error distribution
# - User-reported issues

# View SATUSEHAT Dashboard:
# - Ringkasan Transaksi FHIR
# - Log Transaksi FHIR
# - Check for any validation warnings
```

#### Day 68-70: Scale-Up Planning
```bash
# Based on pilot success:
# - Document lessons learned
# - Update implementation runbook
# - Train additional staff
# - Plan phased rollout to remaining clinics

# Week 11+: Full facility deployment
```

**Success Criteria**:
- Pilot clinic achieves >99% sync success rate for 1 week
- Zero user workflow disruptions
- All validation errors resolved within 24 hours

---

## 🔧 Troubleshooting Common Issues

### Issue 1: "Reference not found" Error
**Symptom**: HTTP 404 when creating Encounter
**Cause**: Referenced Patient/Practitioner/Location doesn't exist in SATUSEHAT
**Solution**: Verify prerequisites created first, check IDs match exactly

### Issue 2: "Invalid code" Validation Error
**Symptom**: HTTP 422 with OperationOutcome
**Cause**: Wrong terminology system or non-existent code
**Solution**: Verify ICD-10/LOINC/KFA code exists, check system URL format

### Issue 3: Token Expiration
**Symptom**: HTTP 401 Unauthorized
**Cause**: OAuth2 token expired
**Solution**: Implement automatic token refresh 5 minutes before expiry

### Issue 4: Rate Limiting
**Symptom**: HTTP 429 Too Many Requests
**Cause**: Exceeded API rate limits
**Solution**: Add exponential backoff, queue non-urgent transmissions

### Issue 5: Patient Not Found in MPI
**Symptom**: NIK lookup returns zero results
**Cause**: Invalid NIK, patient not registered in national system
**Solution**: Verify NIK with KTP, use alternate identifiers (passport, KK)

---

## 📊 Success Metrics

Track these KPIs throughout implementation:

### Technical Metrics
- **Sync Success Rate**: % resources successfully transmitted (target: >99%)
- **API Latency**: p95 response time (target: <2 seconds)
- **Error Rate**: % failed transmissions (target: <1%)
- **Retry Success**: % errors resolved by automatic retry (target: >90%)

### Data Quality Metrics
- **NIK Coverage**: % patients with valid NIK (target: >95%)
- **ICD-10 Completeness**: % encounters with coded diagnosis (target: 100%)
- **Practitioner Validation**: % staff with SATUSEHAT ID (target: 100%)

### Operational Metrics
- **User Workflow Impact**: Average delay per patient interaction (target: <5 seconds)
- **Support Tickets**: SATUSEHAT-related issues per week (target: declining trend)
- **Dashboard Monitoring**: Daily review compliance (target: 100%)

---

## 📞 Getting Help

### During Implementation
1. **Review Documentation**: Check strategic framework and code reference
2. **SATUSEHAT FAQ**: [faq.kemkes.go.id/category/satusehat-platform](https://faq.kemkes.go.id/category/satusehat-platform)
3. **Postman Testing**: Use official collection for API testing
4. **GitHub Issues**: Report framework-related questions

### Production Support
1. **SATUSEHAT Helpdesk**: Via platform dashboard
2. **Technical Forum**: Connect with other implementers
3. **Regulatory Guidance**: Contact Ministry of Health Digital Health team

---

## ✅ Implementation Checklist

Print this checklist and track progress:

```
FOUNDATION
☐ OAuth2 authentication working
☐ Organization registered (ID stored)
☐ 3+ Locations registered
☐ All practitioners validated
☐ MPI integration functional

CORE WORKFLOWS
☐ Encounter creation working
☐ Encounter completion working
☐ Diagnosis recording working
☐ Vital signs transmission working
☐ Medication workflow complete

QUALITY ASSURANCE
☐ 100 test encounters successful
☐ All validation errors resolved
☐ Error handling tested
☐ Retry logic functioning
☐ Monitoring dashboard configured

PRODUCTION READINESS
☐ Production credentials obtained
☐ Prerequisites re-registered in production
☐ Pilot clinic selected
☐ Staff trained
☐ Rollout plan documented

PILOT DEPLOYMENT
☐ Week 1: >95% sync success
☐ Week 2: >99% sync success
☐ No user workflow disruptions
☐ Lessons learned documented
☐ Scale-up plan approved
```

---

**Remember**: SATUSEHAT compliance is a journey, not a destination. Start small, iterate fast, and scale based on real-world feedback. Good luck! 🚀
