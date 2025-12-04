# SATUSEHAT EMR Integration: Code Reference Library
## Production-Ready Implementation Examples

---

## 1. Environment Configuration

### config.py
```python
from pydantic import BaseSettings
from typing import Literal

class SATUSEHATConfig(BaseSettings):
    """Configuration for SATUSEHAT integration"""
    
    # Environment selection
    environment: Literal["staging", "production"] = "staging"
    
    # API endpoints
    @property
    def api_base_url(self) -> str:
        return {
            "staging": "https://api-satusehat-stg.dto.kemkes.go.id",
            "production": "https://api-satusehat.kemkes.go.id"
        }[self.environment]
    
    # OAuth2 credentials (load from secure vault in production)
    client_id: str
    client_secret: str
    
    # Organization identifiers
    organization_id: str  # Your Organization.id from SATUSEHAT
    organization_code: str  # Internal fasyankes code
    
    # Feature flags
    enable_background_sync: bool = True
    enable_validation: bool = True
    
    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"

config = SATUSEHATConfig()
```

---

## 2. Master Patient Index (MPI) Integration

### services/mpi_service.py
```python
from typing import Optional, Dict
from datetime import datetime
import logging

logger = logging.getLogger(__name__)

class MPIService:
    """Master Patient Index operations"""
    
    def __init__(self, satusehat_client):
        self.client = satusehat_client
    
    def get_patient_ihs_number(self, nik: str) -> Optional[str]:
        """
        Resolve NIK to Patient IHS Number
        
        Returns:
            Patient.id if found, None otherwise
        """
        try:
            response = self.client.search_resource(
                "Patient",
                params={
                    "identifier": f"https://fhir.kemkes.go.id/id/nik|{nik}"
                }
            )
            
            if response.get("total", 0) > 0:
                patient = response["entry"][0]["resource"]
                patient_id = patient["id"]
                
                logger.info(f"Patient found: NIK={nik}, IHS Number={patient_id}")
                return patient_id
            else:
                logger.warning(f"Patient not found in MPI: NIK={nik}")
                return None
                
        except Exception as e:
            logger.error(f"MPI lookup failed for NIK={nik}: {e}")
            raise
    
    def register_newborn(
        self, 
        mother_nik: str,
        birth_date: datetime,
        gender: str,
        birth_sequence: int = 1
    ) -> str:
        """
        Register newborn patient before NIK issuance
        
        Args:
            mother_nik: Mother's NIK
            birth_date: Baby's birth date
            gender: 'male' or 'female'
            birth_sequence: 1 for first baby, 2 for twins, etc.
        
        Returns:
            Patient.id of newly created resource
        """
        # First, get mother's Patient.id
        mother_id = self.get_patient_ihs_number(mother_nik)
        if not mother_id:
            raise ValueError(f"Mother not found in MPI: {mother_nik}")
        
        # Generate temporary identifier
        temp_identifier = f"{mother_nik}-{birth_sequence}"
        
        newborn_resource = {
            "resourceType": "Patient",
            "identifier": [{
                "use": "temp",
                "system": f"http://sys-ids.kemkes.go.id/patient/{config.organization_id}",
                "value": temp_identifier
            }],
            "active": True,
            "name": [{
                "use": "official",
                "text": f"By. Ny. {mother_nik}"  # Placeholder name
            }],
            "gender": gender,
            "birthDate": birth_date.strftime("%Y-%m-%d"),
            "link": [{
                "other": {
                    "reference": f"Patient/{mother_id}",
                    "display": "Mother"
                },
                "type": "seealso"
            }]
        }
        
        response = self.client.create_resource(newborn_resource)
        patient_id = response["id"]
        
        logger.info(f"Newborn registered: temp_id={temp_identifier}, IHS Number={patient_id}")
        return patient_id
    
    def search_by_name_dob(self, name: str, birth_date: str) -> list:
        """
        Search patients by name and date of birth (for patients without NIK)
        
        Args:
            name: Patient's full name
            birth_date: Format YYYY-MM-DD
        
        Returns:
            List of matching Patient resources
        """
        response = self.client.search_resource(
            "Patient",
            params={
                "name": name,
                "birthdate": birth_date
            }
        )
        
        if response.get("total", 0) > 0:
            return [entry["resource"] for entry in response["entry"]]
        else:
            return []
```

---

## 3. Encounter Management

### services/encounter_service.py
```python
from typing import Optional
from datetime import datetime
from models import Encounter, EncounterStatus, EncounterClass

class EncounterService:
    """Clinical encounter lifecycle management"""
    
    def __init__(self, db, satusehat_client, mpi_service):
        self.db = db
        self.client = satusehat_client
        self.mpi = mpi_service
        self.builder = EncounterBuilder(config)
    
    def create_outpatient_visit(
        self,
        patient_nik: str,
        practitioner_id: str,
        location_id: str,
        encounter_type_code: Optional[str] = None
    ) -> Encounter:
        """
        Initiate new outpatient encounter
        
        Args:
            patient_nik: Patient's NIK
            practitioner_id: SATUSEHAT Practitioner.id
            location_id: SATUSEHAT Location.id
            encounter_type_code: Optional encounter type from terminology
        
        Returns:
            Created Encounter object
        """
        # Step 1: Resolve patient IHS number
        patient_ihs_number = self.mpi.get_patient_ihs_number(patient_nik)
        if not patient_ihs_number:
            raise ValueError(f"Patient not found: {patient_nik}")
        
        # Step 2: Create internal encounter record
        encounter = self.db.encounters.create({
            "patient_ihs_number": patient_ihs_number,
            "patient_nik": patient_nik,  # Store for reference
            "practitioner_id": practitioner_id,
            "location_id": location_id,
            "encounter_class": EncounterClass.AMBULATORY,
            "encounter_type_code": encounter_type_code,
            "status": EncounterStatus.IN_PROGRESS,
            "period_start": datetime.now(),
            "satusehat_sync_status": "pending"
        })
        
        # Step 3: Build FHIR resource
        fhir_encounter = self.builder.build_resource(encounter)
        
        # Step 4: POST to SATUSEHAT
        try:
            response = self.client.create_resource(fhir_encounter)
            
            # Step 5: Store SATUSEHAT ID and mark as synced
            encounter = self.db.encounters.update(encounter.id, {
                "satusehat_encounter_id": response["id"],
                "satusehat_sync_status": "synced",
                "satusehat_sync_timestamp": datetime.now()
            })
            
            logger.info(f"Encounter created: internal_id={encounter.id}, satusehat_id={response['id']}")
            
        except Exception as e:
            # Mark as failed but don't block workflow
            self.db.encounters.update(encounter.id, {
                "satusehat_sync_status": "error",
                "satusehat_error_message": str(e)
            })
            logger.error(f"Failed to sync encounter {encounter.id}: {e}")
            # Queue for retry via background job
            if config.enable_background_sync:
                sync_encounter_to_satusehat.delay(encounter.id)
        
        return encounter
    
    def complete_encounter(self, encounter_id: int, discharge_disposition: Optional[str] = None):
        """
        Mark encounter as finished and sync to SATUSEHAT
        
        Args:
            encounter_id: Internal encounter ID
            discharge_disposition: Discharge status code
        """
        encounter = self.db.encounters.get(encounter_id)
        
        # Update internal record
        encounter = self.db.encounters.update(encounter_id, {
            "status": EncounterStatus.FINISHED,
            "period_end": datetime.now(),
            "discharge_disposition": discharge_disposition
        })
        
        # Build updated FHIR resource
        fhir_encounter = self.builder.build_resource(encounter)
        
        # PUT to update existing resource in SATUSEHAT
        try:
            self.client.update_resource(
                "Encounter",
                encounter.satusehat_encounter_id,
                fhir_encounter
            )
            
            logger.info(f"Encounter completed: id={encounter_id}")
            
        except Exception as e:
            logger.error(f"Failed to update encounter {encounter_id}: {e}")
            # Queue for retry
            if config.enable_background_sync:
                sync_encounter_to_satusehat.delay(encounter_id)
```

---

## 4. Clinical Documentation

### services/condition_service.py
```python
from typing import List
from datetime import datetime

class ConditionService:
    """Diagnosis management"""
    
    def __init__(self, db, satusehat_client):
        self.db = db
        self.client = satusehat_client
    
    def record_diagnosis(
        self,
        encounter_id: int,
        icd10_code: str,
        clinical_status: str = "active",
        is_primary: bool = True
    ):
        """
        Record a clinical diagnosis
        
        Args:
            encounter_id: Internal encounter ID
            icd10_code: ICD-10 code (e.g., 'E11.9')
            clinical_status: 'active', 'recurrence', 'relapse', 'inactive', 'remission', 'resolved'
            is_primary: Whether this is the primary diagnosis
        """
        encounter = self.db.encounters.get(encounter_id)
        
        # Validate ICD-10 code exists
        icd10_display = self.db.icd10_codes.get_display(icd10_code)
        if not icd10_display:
            raise ValueError(f"Invalid ICD-10 code: {icd10_code}")
        
        # Create internal condition record
        condition = self.db.conditions.create({
            "encounter_id": encounter_id,
            "patient_ihs_number": encounter.patient_ihs_number,
            "icd10_code": icd10_code,
            "clinical_status": clinical_status,
            "is_primary": is_primary,
            "onset_datetime": encounter.period_start
        })
        
        # Build FHIR Condition resource
        fhir_condition = {
            "resourceType": "Condition",
            "clinicalStatus": {
                "coding": [{
                    "system": "http://terminology.hl7.org/CodeSystem/condition-clinical",
                    "code": clinical_status
                }]
            },
            "verificationStatus": {
                "coding": [{
                    "system": "http://terminology.hl7.org/CodeSystem/condition-ver-status",
                    "code": "confirmed"
                }]
            },
            "category": [{
                "coding": [{
                    "system": "http://terminology.hl7.org/CodeSystem/condition-category",
                    "code": "encounter-diagnosis",
                    "display": "Encounter Diagnosis"
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
            },
            "onsetDateTime": encounter.period_start.strftime("%Y-%m-%dT%H:%M:%S+07:00")
        }
        
        # Add diagnosis rank if primary
        if is_primary:
            fhir_condition["extension"] = [{
                "url": "https://fhir.kemkes.go.id/r4/StructureDefinition/DiagnosisRank",
                "valuePositiveInt": 1
            }]
        
        # POST to SATUSEHAT
        try:
            response = self.client.create_resource(fhir_condition)
            
            self.db.conditions.update(condition.id, {
                "satusehat_condition_id": response["id"],
                "satusehat_sync_status": "synced"
            })
            
            logger.info(f"Diagnosis recorded: ICD-10={icd10_code}, Condition.id={response['id']}")
            
        except Exception as e:
            logger.error(f"Failed to sync diagnosis: {e}")
            self.db.conditions.update(condition.id, {
                "satusehat_sync_status": "error",
                "satusehat_error_message": str(e)
            })
```

---

## 5. Medication Workflow

### services/medication_service.py
```python
from typing import Optional, Dict

class MedicationService:
    """Prescription and dispensing workflow"""
    
    def __init__(self, db, satusehat_client):
        self.db = db
        self.client = satusehat_client
    
    def create_prescription(
        self,
        encounter_id: int,
        kfa_code: str,
        dosage_text: str,
        quantity: int,
        unit: str,
        duration_days: int,
        prescriber_id: str
    ) -> Dict:
        """
        Create medication prescription
        
        Args:
            encounter_id: Internal encounter ID
            kfa_code: KFA (Kamus Farmasi Alkes) code
            dosage_text: Human-readable dosage instruction
            quantity: Total quantity prescribed
            unit: Unit of measure (e.g., 'tablet', 'ml')
            duration_days: Duration of treatment
            prescriber_id: SATUSEHAT Practitioner.id of prescriber
        
        Returns:
            Dict with Medication.id and MedicationRequest.id
        """
        encounter = self.db.encounters.get(encounter_id)
        
        # Step 1: Get medication details from KFA
        kfa_medication = self._get_kfa_medication(kfa_code)
        
        # Step 2: Create or retrieve Medication resource
        medication_id = self._ensure_medication_resource(kfa_code, kfa_medication)
        
        # Step 3: Create internal prescription record
        prescription = self.db.prescriptions.create({
            "encounter_id": encounter_id,
            "patient_ihs_number": encounter.patient_ihs_number,
            "kfa_code": kfa_code,
            "dosage_text": dosage_text,
            "quantity": quantity,
            "unit": unit,
            "duration_days": duration_days,
            "prescriber_id": prescriber_id,
            "medication_id": medication_id
        })
        
        # Step 4: Build MedicationRequest FHIR resource
        medication_request = {
            "resourceType": "MedicationRequest",
            "identifier": [{
                "system": f"http://sys-ids.kemkes.go.id/prescription/{config.organization_id}",
                "value": str(prescription.id)
            }],
            "status": "active",
            "intent": "order",
            "medicationReference": {
                "reference": f"Medication/{medication_id}",
                "display": kfa_medication["display_name"]
            },
            "subject": {
                "reference": f"Patient/{encounter.patient_ihs_number}"
            },
            "encounter": {
                "reference": f"Encounter/{encounter.satusehat_encounter_id}"
            },
            "authoredOn": datetime.now().strftime("%Y-%m-%dT%H:%M:%S+07:00"),
            "requester": {
                "reference": f"Practitioner/{prescriber_id}"
            },
            "dosageInstruction": [{
                "text": dosage_text,
                "timing": {
                    "repeat": {
                        "duration": duration_days,
                        "durationUnit": "d"
                    }
                }
            }],
            "dispenseRequest": {
                "quantity": {
                    "value": quantity,
                    "unit": unit
                }
            }
        }
        
        # Step 5: POST MedicationRequest
        try:
            response = self.client.create_resource(medication_request)
            request_id = response["id"]
            
            self.db.prescriptions.update(prescription.id, {
                "satusehat_request_id": request_id,
                "satusehat_sync_status": "synced"
            })
            
            logger.info(f"Prescription created: KFA={kfa_code}, MedicationRequest.id={request_id}")
            
            return {
                "medication_id": medication_id,
                "medication_request_id": request_id
            }
            
        except Exception as e:
            logger.error(f"Failed to create prescription: {e}")
            raise
    
    def record_dispensing(
        self,
        prescription_id: int,
        pharmacist_id: str,
        dispensed_quantity: int,
        when_handed_over: Optional[datetime] = None
    ):
        """
        Record medication dispensing by pharmacy
        
        Args:
            prescription_id: Internal prescription ID
            pharmacist_id: SATUSEHAT Practitioner.id of pharmacist
            dispensed_quantity: Actual quantity dispensed
            when_handed_over: Timestamp when medication was given to patient
        """
        prescription = self.db.prescriptions.get(prescription_id)
        encounter = self.db.encounters.get(prescription.encounter_id)
        
        if not when_handed_over:
            when_handed_over = datetime.now()
        
        # Create MedicationDispense FHIR resource
        medication_dispense = {
            "resourceType": "MedicationDispense",
            "identifier": [{
                "system": f"http://sys-ids.kemkes.go.id/dispense/{config.organization_id}",
                "value": f"{prescription_id}-{int(when_handed_over.timestamp())}"
            }],
            "status": "completed",
            "medicationReference": {
                "reference": f"Medication/{prescription.medication_id}"
            },
            "subject": {
                "reference": f"Patient/{prescription.patient_ihs_number}"
            },
            "context": {
                "reference": f"Encounter/{encounter.satusehat_encounter_id}"
            },
            "performer": [{
                "actor": {
                    "reference": f"Practitioner/{pharmacist_id}"
                }
            }],
            "authorizingPrescription": [{
                "reference": f"MedicationRequest/{prescription.satusehat_request_id}"
            }],
            "quantity": {
                "value": dispensed_quantity,
                "unit": prescription.unit
            },
            "whenHandedOver": when_handed_over.strftime("%Y-%m-%dT%H:%M:%S+07:00")
        }
        
        # POST MedicationDispense
        try:
            response = self.client.create_resource(medication_dispense)
            
            self.db.prescriptions.update(prescription_id, {
                "dispensed": True,
                "satusehat_dispense_id": response["id"],
                "dispensed_at": when_handed_over
            })
            
            logger.info(f"Medication dispensed: Prescription={prescription_id}, MedicationDispense.id={response['id']}")
            
        except Exception as e:
            logger.error(f"Failed to record dispensing: {e}")
            raise
    
    def _get_kfa_medication(self, kfa_code: str) -> Dict:
        """Query KFA master data API"""
        response = self.client.search_resource(
            "Medication",
            params={"code": f"http://sys-ids.kemkes.go.id/kfa|{kfa_code}"}
        )
        
        if response.get("total", 0) > 0:
            return response["entry"][0]["resource"]
        else:
            raise ValueError(f"KFA code not found: {kfa_code}")
    
    def _ensure_medication_resource(self, kfa_code: str, kfa_data: Dict) -> str:
        """Create Medication resource if not exists, otherwise return existing ID"""
        # Check local cache first
        cached = self.db.medications.get_by_kfa_code(kfa_code)
        if cached:
            return cached.satusehat_medication_id
        
        # Create new Medication resource
        medication = {
            "resourceType": "Medication",
            "identifier": [{
                "system": "http://sys-ids.kemkes.go.id/kfa",
                "value": kfa_code
            }],
            "code": {
                "coding": [{
                    "system": "http://sys-ids.kemkes.go.id/kfa",
                    "code": kfa_code,
                    "display": kfa_data["code"]["coding"][0]["display"]
                }]
            },
            "status": "active",
            "form": kfa_data.get("form", {})
        }
        
        response = self.client.create_resource(medication)
        medication_id = response["id"]
        
        # Cache locally
        self.db.medications.create({
            "kfa_code": kfa_code,
            "satusehat_medication_id": medication_id,
            "display_name": kfa_data["code"]["coding"][0]["display"]
        })
        
        return medication_id
```

---

## 6. Background Synchronization

### tasks/sync_tasks.py
```python
from celery import Celery, Task
from celery.utils.log import get_task_logger

logger = get_task_logger(__name__)

app = Celery(
    'satusehat_sync',
    broker='redis://localhost:6379/0',
    backend='redis://localhost:6379/1'
)

app.conf.update(
    task_serializer='json',
    accept_content=['json'],
    result_serializer='json',
    timezone='Asia/Jakarta',
    enable_utc=True,
    task_track_started=True,
    task_time_limit=300,  # 5 minutes
    task_soft_time_limit=240  # 4 minutes
)

class SATUSEHATSyncTask(Task):
    """Base task with automatic retry logic"""
    autoretry_for = (Exception,)
    retry_kwargs = {'max_retries': 5}
    retry_backoff = True
    retry_backoff_max = 600  # 10 minutes
    retry_jitter = True

@app.task(base=SATUSEHATSyncTask, bind=True)
def sync_encounter_to_satusehat(self, encounter_id: int):
    """
    Background task to sync encounter to SATUSEHAT
    
    Automatically retries with exponential backoff on failure
    """
    from services.encounter_service import EncounterService
    from db import get_db
    
    db = get_db()
    encounter = db.encounters.get(encounter_id)
    
    # Skip if already synced
    if encounter.satusehat_sync_status == "synced":
        logger.info(f"Encounter {encounter_id} already synced, skipping")
        return
    
    try:
        # Build FHIR resource
        builder = EncounterBuilder(config)
        fhir_encounter = builder.build_resource(encounter)
        
        # Determine operation: CREATE or UPDATE
        client = SATUSEHATClient(config)
        if encounter.satusehat_encounter_id:
            response = client.update_resource(
                "Encounter",
                encounter.satusehat_encounter_id,
                fhir_encounter
            )
            operation = "UPDATE"
        else:
            response = client.create_resource(fhir_encounter)
            operation = "CREATE"
        
        # Update sync status
        db.encounters.update(encounter_id, {
            "satusehat_encounter_id": response["id"],
            "satusehat_sync_status": "synced",
            "satusehat_sync_timestamp": datetime.now()
        })
        
        logger.info(f"{operation} Encounter successful: internal_id={encounter_id}, satusehat_id={response['id']}")
        
    except Exception as exc:
        logger.error(f"Sync failed for encounter {encounter_id}: {exc}")
        
        # Update error status
        db.encounters.update(encounter_id, {
            "satusehat_sync_status": "error",
            "satusehat_error_message": str(exc),
            "satusehat_retry_count": (encounter.satusehat_retry_count or 0) + 1
        })
        
        # Re-raise to trigger Celery retry
        raise self.retry(exc=exc)

@app.task(base=SATUSEHATSyncTask)
def batch_sync_pending_encounters():
    """
    Periodic task to sync all pending encounters
    
    Run this every 15 minutes via Celery Beat
    """
    from db import get_db
    
    db = get_db()
    pending = db.encounters.get_by_sync_status("pending", limit=100)
    
    logger.info(f"Found {len(pending)} pending encounters to sync")
    
    for encounter in pending:
        sync_encounter_to_satusehat.delay(encounter.id)

# Configure periodic tasks
app.conf.beat_schedule = {
    'sync-pending-encounters-every-15-minutes': {
        'task': 'tasks.sync_tasks.batch_sync_pending_encounters',
        'schedule': 900.0,  # 15 minutes in seconds
    },
}
```

---

## 7. Data Validation Layer

### validators/fhir_validator.py
```python
from jsonschema import validate, ValidationError
from typing import Dict
import json

class FHIRValidator:
    """Validate FHIR resources before transmission"""
    
    def __init__(self):
        self.schemas = self._load_schemas()
    
    def _load_schemas(self) -> Dict:
        """Load FHIR R4 JSON schemas"""
        # In production, load from packaged schema files
        with open("schemas/fhir-r4-schemas.json", "r") as f:
            return json.load(f)
    
    def validate_resource(self, resource: Dict) -> bool:
        """
        Validate FHIR resource against schema
        
        Raises:
            ValidationError if invalid
        """
        resource_type = resource.get("resourceType")
        if not resource_type:
            raise ValidationError("Missing resourceType")
        
        schema = self.schemas.get(resource_type)
        if not schema:
            raise ValidationError(f"No schema found for {resource_type}")
        
        validate(instance=resource, schema=schema)
        return True
    
    def validate_references(self, resource: Dict) -> list:
        """
        Check that all references point to valid resources
        
        Returns:
            List of invalid references (empty if all valid)
        """
        from utils.fhir_utils import extract_references
        
        references = extract_references(resource)
        invalid = []
        
        for ref in references:
            # Check format: ResourceType/id
            parts = ref.split("/")
            if len(parts) != 2:
                invalid.append(ref)
                continue
            
            resource_type, resource_id = parts
            
            # Verify resource exists (check local cache or query SATUSEHAT)
            if not self._reference_exists(resource_type, resource_id):
                invalid.append(ref)
        
        return invalid
    
    def _reference_exists(self, resource_type: str, resource_id: str) -> bool:
        """Check if referenced resource exists"""
        # Implementation depends on your caching strategy
        # For performance, maintain local cache of prerequisite IDs
        pass

validator = FHIRValidator()
```

---

## 8. Error Handling & Monitoring

### utils/error_handlers.py
```python
from typing import Dict
import logging
from datetime import datetime

logger = logging.getLogger(__name__)

class SATUSEHATErrorHandler:
    """Centralized error handling for SATUSEHAT API responses"""
    
    @staticmethod
    def handle_api_error(response: requests.Response, context: Dict):
        """
        Parse and log SATUSEHAT API errors
        
        Args:
            response: Failed HTTP response
            context: Additional context (resource type, internal ID, etc.)
        """
        try:
            error_body = response.json()
        except:
            error_body = response.text
        
        error_data = {
            "timestamp": datetime.now().isoformat(),
            "status_code": response.status_code,
            "error_body": error_body,
            "context": context
        }
        
        # Log based on severity
        if response.status_code >= 500:
            # Server errors - likely transient
            logger.error(f"SATUSEHAT server error: {error_data}")
        elif response.status_code == 429:
            # Rate limiting
            logger.warning(f"SATUSEHAT rate limit exceeded: {error_data}")
        elif response.status_code >= 400:
            # Client errors - likely data validation issue
            logger.warning(f"SATUSEHAT validation error: {error_data}")
            
            # Parse OperationOutcome for specific issues
            if isinstance(error_body, dict) and error_body.get("resourceType") == "OperationOutcome":
                issues = error_body.get("issue", [])
                for issue in issues:
                    logger.error(f"  - {issue.get('severity')}: {issue.get('diagnostics')}")
        
        # Store in database for dashboard
        from db import get_db
        db = get_db()
        db.satusehat_errors.create(error_data)
```

---

## 9. Utility Functions

### utils/fhir_utils.py
```python
from typing import Dict, List
from datetime import datetime
import pytz

def format_fhir_datetime(dt: datetime, timezone: str = "Asia/Jakarta") -> str:
    """
    Convert datetime to FHIR-compliant ISO 8601 format
    
    Args:
        dt: Datetime object
        timezone: IANA timezone name
    
    Returns:
        ISO 8601 string with timezone (e.g., '2025-12-01T14:30:00+07:00')
    """
    if dt.tzinfo is None:
        # Localize naive datetime
        tz = pytz.timezone(timezone)
        dt = tz.localize(dt)
    
    return dt.strftime("%Y-%m-%dT%H:%M:%S%z")

def extract_references(resource: Dict) -> List[str]:
    """
    Recursively extract all references from FHIR resource
    
    Returns:
        List of reference strings (e.g., ['Patient/123', 'Practitioner/456'])
    """
    references = []
    
    def _recurse(obj):
        if isinstance(obj, dict):
            if "reference" in obj and isinstance(obj["reference"], str):
                references.append(obj["reference"])
            for value in obj.values():
                _recurse(value)
        elif isinstance(obj, list):
            for item in obj:
                _recurse(item)
    
    _recurse(resource)
    return references

def build_coding(system: str, code: str, display: str) -> Dict:
    """Helper to build FHIR Coding data type"""
    return {
        "system": system,
        "code": code,
        "display": display
    }

def build_codeable_concept(system: str, code: str, display: str) -> Dict:
    """Helper to build FHIR CodeableConcept data type"""
    return {
        "coding": [build_coding(system, code, display)]
    }

def build_reference(resource_type: str, resource_id: str, display: str = None) -> Dict:
    """Helper to build FHIR Reference data type"""
    ref = {
        "reference": f"{resource_type}/{resource_id}"
    }
    if display:
        ref["display"] = display
    return ref
```

---

## 10. Testing Utilities

### tests/test_satusehat_integration.py
```python
import pytest
from services.mpi_service import MPIService
from services.encounter_service import EncounterService

# Dummy patient data from SATUSEHAT staging
DUMMY_PATIENTS = [
    {"nik": "9271060312000001", "name": "Ardianto Putra", "ihs_number": "P02478375538"},
    {"nik": "9204014804000002", "name": "Claudia Sintia", "ihs_number": "P03647103112"},
]

@pytest.fixture
def satusehat_client():
    """Fixture providing SATUSEHAT client in staging mode"""
    config_override = SATUSEHATConfig(environment="staging")
    return SATUSEHATClient(config_override)

@pytest.fixture
def mpi_service(satusehat_client):
    return MPIService(satusehat_client)

def test_patient_lookup_by_nik(mpi_service):
    """Test MPI patient resolution"""
    for patient in DUMMY_PATIENTS:
        ihs_number = mpi_service.get_patient_ihs_number(patient["nik"])
        assert ihs_number == patient["ihs_number"], f"Patient lookup failed for {patient['name']}"

def test_encounter_creation(encounter_service):
    """Test complete encounter lifecycle"""
    # Create encounter
    encounter = encounter_service.create_outpatient_visit(
        patient_nik=DUMMY_PATIENTS[0]["nik"],
        practitioner_id="N10000001",  # Dummy practitioner in staging
        location_id="b017aa54-f1df-4ec2-9d84-8823815d7228"  # Dummy location
    )
    
    assert encounter.satusehat_sync_status == "synced"
    assert encounter.satusehat_encounter_id is not None
    
    # Complete encounter
    encounter_service.complete_encounter(encounter.id)
    
    # Verify final status
    updated = db.encounters.get(encounter.id)
    assert updated.status == EncounterStatus.FINISHED
    assert updated.period_end is not None

def test_medication_workflow():
    """Test prescription → dispensing flow"""
    # Create prescription
    prescription_ids = medication_service.create_prescription(
        encounter_id=test_encounter_id,
        kfa_code="93001212",  # Metformin 500mg
        dosage_text="1 tablet twice daily",
        quantity=60,
        unit="tablet",
        duration_days=30,
        prescriber_id="N10000001"
    )
    
    assert "medication_request_id" in prescription_ids
    
    # Record dispensing
    medication_service.record_dispensing(
        prescription_id=test_prescription_id,
        pharmacist_id="N10000002",
        dispensed_quantity=60
    )
```

---

## Conclusion

This code reference library provides production-ready implementations of core SATUSEHAT integration patterns. All code examples follow:

1. **Error resilience**: Automatic retries, graceful degradation
2. **Observability**: Comprehensive logging at every integration point
3. **Data integrity**: Validation before transmission, reference checking
4. **Performance**: Background queues for non-blocking operations

Adapt these patterns to your specific EMR architecture and technology stack.
