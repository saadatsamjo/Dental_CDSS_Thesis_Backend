This file is a merged representation of a subset of the codebase, containing specifically included files, combined into a single document by Repomix.

<file_summary>
This section contains a summary of this file.

<purpose>
This file contains a packed representation of a subset of the repository's contents that is considered the most important context.
It is designed to be easily consumable by AI systems for analysis, code review,
or other automated processes.
</purpose>

<file_format>
The content is organized as follows:

1. This summary section
2. Repository information
3. Directory structure
4. Repository files (if enabled)
5. Multiple file entries, each consisting of:

- File path as an attribute
- Full contents of the file
  </file_format>

<usage_guidelines>

- This file should be treated as read-only. Any changes should be made to the
  original repository files, not this packed version.
- When processing this file, use the file path to distinguish
  between different files in the repository.
- Be aware that this file may contain sensitive information. Handle it with
  the same level of security as you would the original repository.
  </usage_guidelines>

<notes>
- Some files may have been excluded based on .gitignore rules and Repomix's configuration
- Binary files are not included in this packed representation. Please refer to the Repository Structure section for a complete list of file paths, including binary files
- Only files matching these patterns are included: config/**/*, scripts/**/*, documents/**/*, app/**/*, test_radiograph/**/*, evaluation_results.json
- Files matching patterns in .gitignore are excluded
- Files matching default ignore patterns are excluded
- Files are sorted by Git change count (files with more changes are at the bottom)
</notes>

</file_summary>

<directory_structure>
app/
cdss_engine/
**init**.py
fusion_engine.py
routes.py
schemas.py
tooth_validator.py
database/
**init**.py
connection.py
helpers/
**init**.py
time.py
model_registry/
**init**.py
RAGsystem/
**init**.py
chains.py
embeddings.py
llm_client.py
llm_providers.py
response_normalizer.py
retriever.py
routes.py
schemas.py
shared/
**init**.py
cost_calculator.py
cost_routes.py
evaluation_routes.py
model_specs.py
performance_tracker.py
system_models/
appointment_model/
appointment_model.py
appointment_schemas.py
clinical_note_model/
clinical_note_model.py
clinical_note_schemas.py
diagnosis_model/
diagnosis_model.py
diagnosis_schemas.py
encounter_model/
encounter_model.py
encounter_schemas.py
facility_model/
facility_model.py
facility_schemas.py
patient_model/
patient_model.py
patient_schemas.py
prescription_model/
prescription_model.py
prescription_schemas.py
system_services/
**init**.py
create_appointment.py
create_clinical_note.py
create_diagnosis.py
create_encounter.py
create_facility.py
create_patient.py
create_prescription.py
system_routes.py
users/
auth_token_model/
token_model.py
token_schemas.py
password_reset_token/
password_reset_token_model.py
password_reset_token_schema.py
user_models/
schemas.py
user_model.py
**init**.py
auth_dependencies.py
auth_emails.py
auth_routers.py
auth_services.py
security.py
visionsystem/
**init**.py
biomedclip_client.py
claude_vision_client.py
florence_client.py
gemini_vision_client.py
gemma3_client.py
gpt4_client.py
groq_vision_client.py
image_processor.py
llava_client.py
llava_med_client.py
routes.py
test_tooth_number_recognition.py
vision_client.py
vision_schemas.py
**init**.py
main.py
config/
**init**.py
appconfig.py
config_schemas.py
ragconfig.py
reset_config_route.py
visionconfig.py
documents/
STG-2013.pdf
STG-2021.pdf
scripts/
**init**.py
ingest_documents.py
test_vision_models.py
test_radiograph/
test_image.jpg
evaluation_results.json
</directory_structure>

<files>
This section contains the contents of the repository's files.

<file path="app/cdss_engine/__init__.py">

</file>

<file path="app/cdss_engine/schemas.py">
# app/cdss_engine/schemas.py
"""
CDSS Request/Response Schemas - Updated with NoImageProvided handling
"""
from pydantic import BaseModel, Field
from typing import List, Optional, Literal, Union, Dict, Any
from datetime import datetime

# ============================================================================

# PATIENT HISTORY

# ============================================================================

class PatientHistory(BaseModel):
"""Patient history from database + current complaint."""
patient_id: Optional[str] = None
age: Optional[int] = None
gender: Optional[str] = None
chief_complaint: Optional[str] = None
medical_history: List[str] = Field(default_factory=list)
current_medications: List[str] = Field(default_factory=list)
allergies: List[str] = Field(default_factory=list)
previous_dental_work: Optional[str] = None
symptoms_duration: Optional[str] = None
pain_level: Optional[int] = Field(None, ge=0, le=10)

# ============================================================================

# VISION ANALYSIS

# ============================================================================

class ImageObservation(BaseModel):
"""Structured radiograph analysis from vision model."""
structured_findings: Optional[Dict[str, Any]] = Field(
None,
description="Highly structured pathology findings (caries, bone loss, etc.)"
)
raw_description: str = Field(..., description="Full clinical analysis from vision model")
pathology_summary: str = Field(..., description="Focused pathology findings")
focused_tooth: Optional[str] = Field(None, description="Tooth number that was the focus of analysis")
image_quality_score: float = Field(0.5, description="0-1 score for image quality")
diagnostic_confidence: float = Field(0.5, description="0-1 score for diagnostic confidence")
overall_confidence: Literal["high", "medium", "low"] = Field(
"medium",
description="Categorical confidence level",
alias="confidence"
)
model_used: str = Field(..., description="Which vision model was used")

    class Config:
        populate_by_name = True

class NoImageProvided(BaseModel):
"""Indicator that no image was provided in the request."""
message: str = Field(default="No radiograph image was provided for this consultation")
image_required: bool = Field(default=False, description="Whether image is required for this case")

# ============================================================================

# KNOWLEDGE RETRIEVAL

# ============================================================================

class RetrievedKnowledge(BaseModel):
"""Retrieved clinical guidelines chunk."""
content: str
pages: List[int]
relevance_score: Optional[float] = None
source: str = "Clinical Guidelines"

# ============================================================================

# CLINICAL RECOMMENDATION

# ============================================================================

class ClinicalRecommendation(BaseModel):
"""Final structured clinical recommendation."""
diagnosis: str = Field(..., description="Primary clinical diagnosis")
differential_diagnoses: List[str] = Field(
default_factory=list,
description="Alternative diagnoses to consider"
)
recommended_management: str = Field(..., description="Treatment plan and next steps")
reference_pages: List[int] = Field(
default_factory=list,
description="Page numbers from guidelines used (integers only, no 'Page X' format)"
)
confidence_level: Literal["high", "medium", "low"] = "medium"
llm_provider: str = Field(default="ollama", description="Which LLM generated this")

# ============================================================================

# CDSS REQUEST

# ============================================================================

class CDSSRequest(BaseModel):
"""Request for CDSS recommendation."""
patient_id: int = Field(..., description="Patient database ID")
chief_complaint: str = Field(..., description="Main clinical complaint/question")
user_id: Optional[int] = Field(1, description="Requesting doctor/admin ID")

    class Config:
        json_schema_extra = {
            "example": {
                "patient_id": 123,
                "chief_complaint": "severe tooth pain, sensitivity to cold",
                "user_id": 1
            }
        }

# ============================================================================

# CDSS RESPONSE (Improved Structure)

# ============================================================================

class CDSSResponse(BaseModel):
"""Complete CDSS response with improved organization."""

    # === PRIMARY RESULT ===
    recommendation: ClinicalRecommendation

    # === SUPPORTING DATA ===
    image_observations: Union[ImageObservation, NoImageProvided] = Field(
        ...,
        description="Radiograph analysis if image was provided, or NoImageProvided message"
    )

    knowledge_sources: List[RetrievedKnowledge] = Field(
        default_factory=list,
        description="Retrieved guideline chunks"
    )

    # === REASONING TRACE ===
    reasoning_chain: str = Field(
        ...,
        description="Step-by-step reasoning process"
    )

    # === METADATA ===
    processing_metadata: dict = Field(
        default_factory=dict,
        description="Performance metrics and model information"
    )

    timestamp: datetime = Field(default_factory=datetime.utcnow)

    class Config:
        json_schema_extra = {
            "example": {
                "recommendation": {
                    "diagnosis": "Irreversible pulpitis, tooth #30",
                    "differential_diagnoses": [
                        "Acute apical periodontitis",
                        "Symptomatic irreversible pulpitis"
                    ],
                    "recommended_management": "Root canal therapy indicated for tooth #30. Consider referral to endodontist given periapical radiolucency.",
                    "reference_pages": [45, 46, 47],
                    "confidence_level": "high",
                    "llm_provider": "gpt-4"
                },
                "image_observations": {
                    "raw_description": "Periapical radiograph shows tooth #30 with large occlusal caries extending into pulp...",
                    "pathology_summary": "Deep caries, periapical radiolucency 4mm",
                    "confidence": "high",
                    "model_used": "gpt-4-vision"
                },
                "processing_metadata": {
                    "total_time_seconds": 8.45,
                    "vision_provider": "gpt4v",
                    "llm_provider": "gpt-4",
                    "patient_id": 123,
                    "user_id": 1
                }
            }
        }

</file>

<file path="app/cdss_engine/tooth_validator.py">
# app/cdss_engine/tooth_validator.py

"""
Tooth Number Validator - Enforce Clinical Consistency
This is a DETERMINISTIC validation layer (no AI involved)
"""
import logging
import re
from typing import List, Dict, Optional

logger = logging.getLogger(**name**)

class ToothValidator:
"""
Validate and correct tooth number inconsistencies.
Uses anatomical rules to enforce clinical accuracy.
"""

    # Universal Numbering System (US/Canadian) - most common in North America
    UNIVERSAL_QUADRANTS = {
        "UR": list(range(1, 9)),    # Upper Right: 1-8
        "UL": list(range(9, 17)),   # Upper Left: 9-16
        "LL": list(range(17, 25)),  # Lower Left: 17-24
        "LR": list(range(25, 33)),  # Lower Right: 25-32
    }

    # FDI World Dental Federation Numbering (International)
    FDI_QUADRANTS = {
        "UR": list(range(11, 19)),  # 11-18
        "UL": list(range(21, 29)),  # 21-28
        "LL": list(range(31, 39)),  # 31-38
        "LR": list(range(41, 49)),  # 41-48
    }

    def validate_and_fix(
        self,
        focus_tooth: str,
        vision_teeth: List[str],
        diagnosis: str,
        image_obs: Dict
    ) -> Dict:
        """
        Main validation method - enforces consistency across all outputs.

        Args:
            focus_tooth: The tooth number user requested (e.g., "47")
            vision_teeth: Teeth identified by vision model (e.g., ["23", "24", "25"])
            diagnosis: The diagnosis text from LLM
            image_obs: Full image observation dict

        Returns:
            dict with:
            - valid: bool (True if no issues)
            - issues: List[str] (descriptions of problems found)
            - fixes: Dict (corrected values)
        """
        issues = []
        fixes = {}

        # Convert to int for processing
        try:
            focus_num = int(focus_tooth) if focus_tooth else None
            vision_nums = [int(t) for t in vision_teeth if t and str(t).strip()]
        except (ValueError, TypeError) as e:
            issues.append(f"Invalid tooth number format: {e}")
            return {"valid": False, "issues": issues, "fixes": fixes}

        # RULE 1: Vision teeth must be in same quadrant as focus tooth
        if focus_num and vision_nums:
            quadrant = self._get_quadrant(focus_num)
            valid_teeth = [t for t in vision_nums if self._get_quadrant(t) == quadrant]

            if len(valid_teeth) < len(vision_nums):
                removed = set(vision_nums) - set(valid_teeth)
                issues.append(
                    f"Removed teeth {removed} - not in same quadrant as focus tooth #{focus_num}"
                )
                fixes["corrected_teeth_visible"] = [str(t) for t in valid_teeth] or [focus_tooth]
                logger.warning(f"⚠️  Tooth quadrant mismatch - removed {removed}")

        # RULE 2: Periapical X-ray can't show more than 5 teeth
        if len(vision_nums) > 5:
            issues.append(
                f"Periapical X-ray cannot show {len(vision_nums)} teeth (maximum 5)"
            )
            # Keep teeth closest to focus tooth
            if focus_num:
                distances = [(abs(t - focus_num), t) for t in vision_nums]
                closest = sorted(distances)[:3]
                fixes["corrected_teeth_visible"] = [str(t[1]) for t in closest]
                logger.warning(f"⚠️  Too many teeth - keeping 3 closest to #{focus_num}")
            else:
                # No focus tooth - just keep first 3
                fixes["corrected_teeth_visible"] = [str(t) for t in vision_nums[:3]]

        # RULE 3: Diagnosis MUST mention focus tooth
        if focus_num:
            # Check if tooth number appears in diagnosis
            tooth_mentioned = (
                str(focus_num) in diagnosis or
                f"#{focus_num}" in diagnosis or
                f"tooth {focus_num}" in diagnosis.lower()
            )

            if not tooth_mentioned:
                issues.append(
                    f"Diagnosis missing focus tooth #{focus_num}"
                )
                # Add tooth number to diagnosis
                diagnosis_clean = diagnosis.rstrip('.')
                fixes["corrected_diagnosis"] = f"{diagnosis_clean}, tooth #{focus_num}."
                logger.warning(f"⚠️  Diagnosis missing tooth number - adding #{focus_num}")

        # RULE 4: Vision model's focused_tooth must match requested focus
        if focus_num and image_obs.get("structured_findings"):
            vision_focus = image_obs["structured_findings"].get("focused_tooth")
            if vision_focus:
                try:
                    vision_focus_num = int(vision_focus)
                    if vision_focus_num != focus_num:
                        issues.append(
                            f"Vision model focused on tooth #{vision_focus} instead of requested #{focus_num}"
                        )
                        fixes["corrected_focused_tooth"] = focus_tooth
                        logger.warning(f"⚠️  Vision focus mismatch: {vision_focus} vs {focus_num}")
                except (ValueError, TypeError):
                    pass

        # RULE 5: Validate FDI vs Universal numbering consistency
        numbering_system = self._detect_numbering_system(focus_num)
        if numbering_system:
            for tooth in vision_nums:
                tooth_system = self._detect_numbering_system(tooth)
                if tooth_system and tooth_system != numbering_system:
                    issues.append(
                        f"Mixed numbering systems detected: {numbering_system} vs {tooth_system}"
                    )
                    logger.warning(f"⚠️  Mixed numbering: {numbering_system} and {tooth_system}")

        # Log results
        if issues:
            logger.warning(f"⚠️  Validation found {len(issues)} issue(s)")
            for i, issue in enumerate(issues, 1):
                logger.warning(f"     {i}. {issue}")
        else:
            logger.info(f"✅ Validation passed - no issues found")

        return {
            "valid": len(issues) == 0,
            "issues": issues,
            "fixes": fixes
        }

    def _get_quadrant(self, tooth_num: int) -> Optional[str]:
        """
        Determine which quadrant a tooth is in.
        Supports both Universal (1-32) and FDI (11-48) systems.
        """
        # Universal system (1-32)
        if 1 <= tooth_num <= 8:
            return "UR"
        if 9 <= tooth_num <= 16:
            return "UL"
        if 17 <= tooth_num <= 24:
            return "LL"
        if 25 <= tooth_num <= 32:
            return "LR"

        # FDI system (11-48)
        if 11 <= tooth_num <= 18:
            return "UR"
        if 21 <= tooth_num <= 28:
            return "UL"
        if 31 <= tooth_num <= 38:
            return "LL"
        if 41 <= tooth_num <= 48:
            return "LR"

        logger.warning(f"⚠️  Tooth #{tooth_num} not in valid range")
        return None

    def _detect_numbering_system(self, tooth_num: int) -> Optional[str]:
        """Detect if tooth uses Universal or FDI numbering"""
        if 1 <= tooth_num <= 32:
            return "Universal"
        if 11 <= tooth_num <= 48:
            return "FDI"
        return None

    def get_adjacent_teeth(self, tooth_num: int, range_size: int = 2) -> List[int]:
        """
        Get teeth within N positions of target tooth.
        Useful for periapical X-ray expectations.
        """
        system = self._detect_numbering_system(tooth_num)

        if system == "Universal":
            # Universal system (1-32)
            return list(range(
                max(1, tooth_num - range_size),
                min(32, tooth_num + range_size) + 1
            ))
        elif system == "FDI":
            # FDI system - stay within quadrant
            quadrant = self._get_quadrant(tooth_num)
            if quadrant in self.FDI_QUADRANTS:
                quad_teeth = self.FDI_QUADRANTS[quadrant]
                return [t for t in quad_teeth if abs(t - tooth_num) <= range_size]

        return [tooth_num]

    def is_valid_periapical_combination(
        self,
        teeth: List[int],
        focus_tooth: int
    ) -> bool:
        """
        Check if a list of teeth is a valid periapical X-ray combination.

        Rules:
        - All teeth must be in same quadrant
        - Maximum 5 teeth
        - Must be adjacent (no gaps > 2 teeth)
        """
        if len(teeth) > 5:
            return False

        # Check same quadrant
        focus_quad = self._get_quadrant(focus_tooth)
        if not all(self._get_quadrant(t) == focus_quad for t in teeth):
            return False

        # Check adjacency
        sorted_teeth = sorted(teeth)
        for i in range(len(sorted_teeth) - 1):
            if sorted_teeth[i+1] - sorted_teeth[i] > 2:
                return False  # Gap too large

        return True

# Global instance

tooth_validator = ToothValidator()
</file>

<file path="app/database/__init__.py">

</file>

<file path="app/database/connection.py">
# app/database/connection.py

from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from sqlalchemy.orm import declarative_base
from config.appconfig import settings

# For async operations, use create_async_engine and make sure your DB_URL starts with postgresql+asyncpg://

DATABASE_URL = settings.DB_URL

# Create async engine

engine = create_async_engine(
DATABASE_URL,
future=True, # echo=True,
)

# Create async session factory

AsyncSessionLocal = async*sessionmaker(
engine,
class*=AsyncSession,
expire_on_commit=False,
autocommit=False,
autoflush=False,
)

# Base class for models

Base = declarative_base()

print(f"✅ Successfully Connected to database: {settings.DB_NAME}")

from typing import AsyncGenerator

# Dependency to get database session

async def get_db() -> AsyncGenerator[AsyncSession, None]:
async with AsyncSessionLocal() as session:
try:
yield session
await session.commit() # print("Session committed successfully")
except Exception as e:
print(f"Error committing session in get_db: {e}")
raise
finally:
await session.close()
</file>

<file path="app/helpers/__init__.py">

</file>

<file path="app/helpers/time.py">
# app/helpers/time.py

from datetime import datetime, timezone

def utcnow():
return datetime.now(timezone.utc)
</file>

<file path="app/model_registry/__init__.py">
# app/model_registry/__init__.py

# Register all models here

# User models

from app.users.user_models.user_model import User
from app.users.auth_token_model.token_model import Token
from app.users.password_reset_token.password_reset_token_model import PasswordResetToken

# System models

from app.system_models.patient_model.patient_model import Patient
from app.system_models.facility_model.facility_model import Facility
from app.system_models.appointment_model.appointment_model import Appointment
from app.system_models.encounter_model.encounter_model import Encounter
from app.system_models.diagnosis_model.diagnosis_model import Diagnosis
from app.system_models.prescription_model.prescription_model import Prescription
from app.system_models.clinical_note_model.clinical_note_model import ClinicalNote
</file>

<file path="app/RAGsystem/__init__.py">

</file>

<file path="app/RAGsystem/embeddings.py">
# app/RAGsystem/embeddings.py

from langchain_ollama import OllamaEmbeddings
from langchain_huggingface import HuggingFaceEmbeddings
from config.ragconfig import rag_settings
import logging

logger = logging.getLogger(**name**)

class EmbeddingProvider:
"""Factory for embedding models with automatic provider switching."""

    _instance = None
    _embeddings = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def get_embeddings(self):
        """Get or initialize embeddings based on configuration."""
        if self._embeddings is None:
            self._embeddings = self._create_embeddings()
        return self._embeddings

    def _create_embeddings(self):
        provider = rag_settings.EMBEDDING_PROVIDER
        model_name = rag_settings.current_embedding_model

        logger.info(f"Initializing {provider} embeddings: {model_name}")

        if provider == "ollama":
            return OllamaEmbeddings(model=model_name)

        elif provider == "huggingface":
            device = rag_settings.effective_device
            return HuggingFaceEmbeddings(
                model_name=model_name,
                model_kwargs={"device": device},
                encode_kwargs={
                    "normalize_embeddings": True,
                    "batch_size": 32 if device == "cuda" else 8
                }
            )

        else:
            raise ValueError(f"Unknown embedding provider: {provider}")

    def refresh(self):
        """Force re-initialization (e.g., after config change)."""
        self._embeddings = None
        return self.get_embeddings()

# Global instance

embedding_provider = EmbeddingProvider()
</file>

<file path="app/RAGsystem/response_normalizer.py">
# app/RAGsystem/response_normalizer.py
"""
Response Normalization Layer
Enforces consistent JSON schema regardless of LLM variations
Ensures mandatory reference pages are always present
"""
from typing import Dict, Any, List, Optional
import logging

logger = logging.getLogger(**name**)

def normalize_rag_response(raw_data: Dict[str, Any]) -> Dict[str, Any]:
"""
Normalize LLM response to consistent schema.

    Handles variations like:
    - "pharmacological" vs "pharmacological_treatment"
    - "non-pharmacological" vs "non_pharmacological_treatment"
    - Different nesting structures

    Returns standardized structure with mandatory fields.
    """

    # Standard schema template
    normalized = {
        "diagnosis": extract_diagnosis(raw_data),
        "differential_diagnoses": extract_differential_diagnoses(raw_data),
        "recommended_management": {
            "pharmacological": {
                "analgesics": [],
                "antibiotics": []
            },
            "non_pharmacological": [],
            "follow_up": "Not specified"
        },
        "precautions": [],
        "reference_pages": []
    }

    # Extract management section (handles various keys)
    mgmt = (
        raw_data.get("management") or
        raw_data.get("recommended_management") or
        {}
    )

    # Normalize pharmacological treatment
    pharm = extract_pharmacological(mgmt)
    normalized["recommended_management"]["pharmacological"] = pharm

    # Normalize non-pharmacological treatment
    non_pharm = extract_non_pharmacological(mgmt)
    normalized["recommended_management"]["non_pharmacological"] = non_pharm

    # Extract precautions
    precautions = extract_precautions(raw_data.get("precautions") or mgmt.get("precautions"))
    normalized["precautions"] = precautions

    # Collect all reference pages
    normalized["reference_pages"] = extract_all_pages(normalized)

    # Validation: Warn if no references found
    if not normalized["reference_pages"]:
        logger.warning("⚠️  No reference pages found in LLM response - this should not happen!")

    return normalized

def extract_diagnosis(data: Dict[str, Any]) -> str:
"""Extract diagnosis from various possible locations"""
return data.get("diagnosis") or "Not specified"

def extract_differential_diagnoses(data: Dict[str, Any]) -> List[str]:
"""Extract differential diagnoses list"""
diff = data.get("differential_diagnoses") or data.get("differentials") or []
return list(diff) if isinstance(diff, list) else []

def extract_pharmacological(mgmt: Dict[str, Any]) -> Dict[str, List[Dict]]:
"""
Extract and normalize pharmacological treatments.

    Handles:
    - "pharmacological" or "pharmacological_treatment"
    - Different drug categorizations
    """
    pharm_data = (
        mgmt.get("pharmacological") or
        mgmt.get("pharmacological_treatment") or
        {}
    )

    result = {
        "analgesics": [],
        "antibiotics": []
    }

    if not isinstance(pharm_data, dict):
        return result

    # Extract analgesics
    analgesics = pharm_data.get("analgesics") or []
    result["analgesics"] = [
        normalize_drug(drug, "analgesic") for drug in analgesics
    ]

    # Extract antibiotics
    antibiotics = pharm_data.get("antibiotics") or []
    result["antibiotics"] = [
        normalize_drug(drug, "antibiotic") for drug in antibiotics
    ]

    return result

def normalize_drug(drug: Dict[str, Any], drug_type: str) -> Dict[str, Any]:
"""
Normalize drug information to consistent format.

    Ensures:
    - name field
    - dose field (combines adult_dose/children_dose if needed)
    - reference_page field
    """
    if not isinstance(drug, dict):
        return {"name": str(drug), "dose": "Not specified", "reference_page": None}

    # Extract name
    name = drug.get("name") or "Unknown"

    # Extract dose (handle variations)
    dose = drug.get("dose")
    if not dose:
        # Try adult_dose + children_dose
        adult = drug.get("adult_dose")
        children = drug.get("children_dose")
        if adult and children:
            dose = f"Adult: {adult}; Children: {children}"
        elif adult:
            dose = adult
        elif children:
            dose = children
        else:
            dose = "Not specified"

    # Extract reference page
    ref_page = extract_page_number(drug)

    return {
        "name": name,
        "dose": dose,
        "reference_page": ref_page,
        "type": drug_type
    }

def extract_non_pharmacological(mgmt: Dict[str, Any]) -> List[Dict[str, Any]]:
"""
Extract and normalize non-pharmacological treatments.

    Handles various formats:
    - "non-pharmacological" or "non_pharmacological_treatment"
    - Nested arrays under keys like "drainage", "extraction", "treatment"
    - Direct arrays
    """
    non_pharm = (
        mgmt.get("non_pharmacological") or
        mgmt.get("non-pharmacological") or
        mgmt.get("non_pharmacological_treatment") or
        {}
    )

    treatments = []

    if isinstance(non_pharm, dict):
        # Iterate through all keys (drainage, extraction, treatment, etc.)
        for key, value in non_pharm.items():
            if isinstance(value, list):
                for item in value:
                    treatments.append(normalize_treatment(item, key))
            elif isinstance(value, dict):
                treatments.append(normalize_treatment(value, key))

    elif isinstance(non_pharm, list):
        # Direct array of treatments
        for item in non_pharm:
            treatments.append(normalize_treatment(item))

    return treatments

def normalize_treatment(item: Any, category: Optional[str] = None) -> Dict[str, Any]:
"""Normalize a single non-pharmacological treatment"""
if not isinstance(item, dict):
return {
"description": str(item),
"category": category or "general",
"reference_page": None
}

    # Extract description (various possible keys)
    description = (
        item.get("type") or
        item.get("name") or
        item.get("action") or
        item.get("description") or
        str(item)
    )

    # Extract additional details
    details = item.get("for") or item.get("dose") or ""
    if details:
        description = f"{description} ({details})"

    # Extract reference page
    ref_page = extract_page_number(item)

    return {
        "description": description,
        "category": category or "general",
        "reference_page": ref_page
    }

def extract_precautions(precautions_data: Any) -> List[Dict[str, Any]]:
"""Extract and normalize precautions/contraindications"""
if not precautions_data:
return []

    result = []

    if isinstance(precautions_data, dict):
        # Nested structure like {"pregnancy": "...", "systemic_diseases": "..."}
        for condition, note in precautions_data.items():
            if isinstance(note, list):
                for item in note:
                    result.append({
                        "condition": condition.replace("_", " ").title(),
                        "note": item.get("note") if isinstance(item, dict) else str(item),
                        "reference_page": extract_page_number(item) if isinstance(item, dict) else None
                    })
            else:
                result.append({
                    "condition": condition.replace("_", " ").title(),
                    "note": str(note),
                    "reference_page": None
                })

    elif isinstance(precautions_data, list):
        for item in precautions_data:
            if isinstance(item, dict):
                result.append({
                    "condition": item.get("condition") or "General",
                    "note": item.get("note") or str(item),
                    "reference_page": extract_page_number(item)
                })
            else:
                result.append({
                    "condition": "General",
                    "note": str(item),
                    "reference_page": None
                })

    return result

def extract_page_number(item: Dict[str, Any]) -> Optional[int]:
"""
Extract page number from various possible field names.

    Tries: reference_page, ref_page, page, ref
    """
    if not isinstance(item, dict):
        return None

    # Try various field names
    for key in ["reference_page", "ref_page", "page", "ref"]:
        value = item.get(key)
        if value is not None:
            try:
                # Handle string pages like "43" or numeric
                return int(str(value).strip())
            except (ValueError, AttributeError):
                pass

    return None

def extract_all_pages(data: Dict[str, Any]) -> List[int]:
"""
Recursively extract all page numbers from the normalized response.

    Returns sorted list of unique page numbers.
    """
    pages = set()

    def recurse(obj):
        if isinstance(obj, dict):
            for key, value in obj.items():
                if "page" in key.lower() and value is not None:
                    try:
                        pages.add(int(str(value).strip()))
                    except (ValueError, TypeError, AttributeError):
                        pass
                else:
                    recurse(value)
        elif isinstance(obj, list):
            for item in obj:
                recurse(item)

    recurse(data)

    # Return sorted list
    return sorted(list(pages))

# ============================================================================

# VALIDATION

# ============================================================================

def validate_response(data: Dict[str, Any]) -> None:
"""
Validate that response has all required fields.

    Raises HTTPException if critical fields missing.
    """
    errors = []

    # Check diagnosis
    if not data.get("diagnosis") or data["diagnosis"] == "Not specified":
        errors.append("Missing diagnosis")

    # Check management exists
    mgmt = data.get("recommended_management", {})
    if not mgmt:
        errors.append("Missing recommended_management")

    # Check for at least some treatment
    pharm = mgmt.get("pharmacological", {})
    non_pharm = mgmt.get("non_pharmacological", [])

    has_analgesics = len(pharm.get("analgesics", [])) > 0
    has_antibiotics = len(pharm.get("antibiotics", [])) > 0
    has_non_pharm = len(non_pharm) > 0

    if not (has_analgesics or has_antibiotics or has_non_pharm):
        errors.append("No treatment recommendations found")

    # Warn if no reference pages (don't fail, just log)
    if not data.get("reference_pages"):
        logger.warning("⚠️  Response missing reference pages - citations required for clinical trust")

    # Raise if critical errors
    if errors:
        from fastapi import HTTPException
        raise HTTPException(
            status_code=500,
            detail=f"Invalid LLM response: {', '.join(errors)}"
        )

</file>

<file path="app/shared/__init__.py">

</file>

<file path="app/shared/evaluation_routes.py">
# app/shared/evaluation_routes.py

"""
Evaluation & Benchmarking API
Generates thesis-ready tables and analysis from real test data
"""
from typing import Dict, List

from fastapi import APIRouter, HTTPException

from app.shared.model_specs import (
HARDWARE_TIERS,
LLM_MODEL_SPECS,
VISION_MODEL_SPECS,
get_deployment_category,
get_model_spec,
)
from app.shared.performance_tracker import performance_tracker

router = APIRouter(prefix="/evaluation", tags=["evaluation"])

@router.get("/dashboard")
async def get_professional_dashboard():
"""
Complete professional dashboard for thesis.

    Includes:
    - Summary statistics
    - Multiple chart types (bar, scatter, grouped bar)
    - Performance summary table
    - Deployment recommendations
    - Proper provider labels (cloud models show platform: groq, google, openai, etc.)
    """
    from datetime import datetime
    from app.shared.model_specs import get_model_spec

    all_results = performance_tracker.results

    if not all_results:
        return {
            "status": "no_data",
            "message": "No evaluation data. Run /api/vision/test_vision_models first"
        }

    # ═══════════════════════════════════════════════════════════
    # GROUP AND AGGREGATE RESULTS
    # ═══════════════════════════════════════════════════════════
    model_groups = {}
    for result in all_results:
        if result.model_name not in model_groups:
            model_groups[result.model_name] = []
        model_groups[result.model_name].append(result)

    detailed_data = []

    for model_name, results in model_groups.items():
        spec = get_model_spec(model_name)

        # Determine proper provider label
        if spec:
            if spec.provider == "local":
                provider_label = "Local"
            elif spec.provider == "cloud":
                # Use platform name for cloud models
                provider_label = spec.platform.capitalize()  # groq → Groq, google → Google
            else:
                provider_label = "Unknown"

            display_name = spec.display_name
            min_ram_gb = spec.min_ram_gb
        else:
            # Fallback for models not in spec database
            provider_label = "Unknown"
            display_name = model_name
            min_ram_gb = 8

        # Aggregate metrics
        num_tests = len(results)
        avg_time_ms = sum(r.inference_time_ms for r in results) / num_tests
        avg_time_sec = avg_time_ms / 1000

        # Resources
        peak_ram_values = [r.peak_ram_mb for r in results if r.peak_ram_mb]
        avg_peak_ram_mb = sum(peak_ram_values) / len(peak_ram_values) if peak_ram_values else None

        cpu_values = [r.avg_cpu_percent for r in results if r.avg_cpu_percent is not None]
        avg_cpu = sum(cpu_values) / len(cpu_values) if cpu_values else None

        model_size_gb = results[0].model_size_gb

        # Cost
        cost_values = [r.cost_usd for r in results if r.cost_usd is not None]
        avg_cost = sum(cost_values) / len(cost_values) if cost_values else 0.0
        annual_cost = avg_cost * 50 * 365

        # Quality
        conf_values = [r.confidence_score for r in results if r.confidence_score is not None]
        avg_confidence = sum(conf_values) / len(conf_values) if conf_values else None

        citation_values = [r.citation_count for r in results]
        avg_citations = sum(citation_values) / len(citation_values)

        tps_values = [r.tokens_per_second for r in results if r.tokens_per_second]
        avg_tps = sum(tps_values) / len(tps_values) if tps_values else None

        detailed_data.append({
            "model": display_name,
            "raw_model_name": model_name,
            "provider": provider_label,  # ✅ PROPER LABELS
            "avg_inference_time_sec": round(avg_time_sec, 1),
            "model_size_gb": model_size_gb,
            "min_ram_required_gb": min_ram_gb,
            "avg_peak_ram_mb": round(avg_peak_ram_mb, 2) if avg_peak_ram_mb else None,
            "avg_cpu_percent": round(avg_cpu, 1) if avg_cpu else None,
            "cost_per_analysis_usd": round(avg_cost, 6),
            "annual_cost_50_daily_usd": round(annual_cost, 2),
            "avg_confidence_score": round(avg_confidence, 3) if avg_confidence else None,
            "avg_citation_count": round(avg_citations, 1),
            "avg_tokens_per_second": round(avg_tps, 2) if avg_tps else None,
            "num_tests": num_tests
        })

    # Sort by speed (fastest first)
    detailed_data.sort(key=lambda x: x["avg_inference_time_sec"])

    # ═══════════════════════════════════════════════════════════
    # SUMMARY STATISTICS
    # ═══════════════════════════════════════════════════════════
    local_models = [m for m in detailed_data if m["provider"] == "Local"]
    cloud_models = [m for m in detailed_data if m["provider"] != "Local" and m["provider"] != "Unknown"]

    fastest_model = detailed_data[0] if detailed_data else None

    cheapest_cloud = None
    if cloud_models:
        cheapest_cloud = min(cloud_models, key=lambda x: x["cost_per_analysis_usd"])

    best_local = None
    if local_models:
        best_local = min(local_models, key=lambda x: x["avg_inference_time_sec"])

    summary = {
        "total_models": len(detailed_data),
        "total_tests": len(all_results),
        "local_models": len(local_models),
        "cloud_models": len(cloud_models),
        "fastest_model": fastest_model["model"] if fastest_model else "N/A",
        "fastest_speed_sec": fastest_model["avg_inference_time_sec"] if fastest_model else None,
        "cheapest_cloud": cheapest_cloud["model"] if cheapest_cloud else "N/A",
        "cheapest_cloud_cost": cheapest_cloud["cost_per_analysis_usd"] if cheapest_cloud else None,
        "best_local": best_local["model"] if best_local else "N/A",
        "best_local_speed": best_local["avg_inference_time_sec"] if best_local else None
    }

    # ═══════════════════════════════════════════════════════════
    # CHARTS (Multiple Visualizations)
    # ═══════════════════════════════════════════════════════════
    charts = {
        "speed_comparison_bar": {
            "type": "bar",
            "title": "Inference Speed Comparison",
            "data": [
                {
                    "model": m["model"],
                    "value": m["avg_inference_time_sec"],
                    "unit": "seconds",
                    "color": "#10b981" if m["provider"] == "Local" else "#3b82f6"
                }
                for m in detailed_data
            ]
        },

        "cost_comparison_bar": {
            "type": "bar",
            "title": "Annual Cost (50 cases/day)",
            "data": [
                {
                    "model": m["model"],
                    "value": m["annual_cost_50_daily_usd"],
                    "unit": "USD/year",
                    "category": "Local (CAPEX)" if m["provider"] == "Local" else "Cloud (OPEX)"
                }
                for m in sorted(detailed_data, key=lambda x: x["annual_cost_50_daily_usd"])
            ]
        },

        "pareto_scatter": {
            "type": "scatter",
            "title": "Speed vs Quality Pareto Frontier",
            "x_label": "Speed (seconds)",
            "y_label": "Quality (%)",
            "data": [
                {
                    "model": m["model"],
                    "x": m["avg_inference_time_sec"],
                    "y": (m["avg_confidence_score"] * 100) if m["avg_confidence_score"] else 50,
                    "size": m["model_size_gb"] if isinstance(m["model_size_gb"], (int, float)) else 1,
                    "color": "#10b981" if m["provider"] == "Local" else "#3b82f6"
                }
                for m in detailed_data
            ]
        },

        "resource_requirements": {
            "type": "grouped_bar",
            "title": "Hardware Requirements",
            "data": [
                {
                    "model": m["model"],
                    "ram_gb": m["min_ram_required_gb"],
                    "size_gb": m["model_size_gb"] if isinstance(m["model_size_gb"], (int, float)) else 0,
                    "peak_ram_gb": round(m["avg_peak_ram_mb"] / 1024, 2) if m["avg_peak_ram_mb"] else 0
                }
                for m in detailed_data
            ]
        },

        "confidence_scores": {
            "type": "bar",
            "title": "Model Confidence Scores",
            "data": [
                {
                    "model": m["model"],
                    "value": round(m["avg_confidence_score"] * 100, 1) if m["avg_confidence_score"] else None,
                    "unit": "%"
                }
                for m in detailed_data
            ]
        }
    }

    # ═══════════════════════════════════════════════════════════
    # PERFORMANCE SUMMARY TABLE
    # ═══════════════════════════════════════════════════════════
    performance_table = [
        {
            "Model": m["model"],
            "Speed (s)": m["avg_inference_time_sec"],
            "Cost/Analysis": f"${m['cost_per_analysis_usd']:.5f}",
            "RAM (GB)": m["min_ram_required_gb"],
            "Provider": m["provider"],
            "Confidence": f"{m['avg_confidence_score']*100:.1f}%" if m["avg_confidence_score"] else "N/A",
            "Model Size (GB)": m["model_size_gb"] if m["model_size_gb"] else "Cloud"
        }
        for m in detailed_data
    ]

    # ═══════════════════════════════════════════════════════════
    # DEPLOYMENT RECOMMENDATIONS (Tanzania Context)
    # ═══════════════════════════════════════════════════════════
    recommendations = {}

    # Ultra-low cost (smallest local model)
    if local_models:
        smallest_local = min(
            [m for m in local_models if isinstance(m["model_size_gb"], (int, float))],
            key=lambda x: x["model_size_gb"],
            default=None
        )
        if smallest_local:
            recommendations["tanzania_rural"] = {
                "model": smallest_local["model"],
                "annual_cost": "$0",
                "reasoning": f"Smallest local model ({smallest_local['model_size_gb']}GB), fully offline, $0 opex"
            }

        # Best local performance
        recommendations["best_local"] = {
            "model": best_local["model"],
            "annual_cost": "$0",
            "reasoning": f"Fastest local model ({best_local['avg_inference_time_sec']}s), fully offline"
        }

    # Best cloud value
    if cheapest_cloud:
        recommendations["tanzania_urban"] = {
            "model": cheapest_cloud["model"],
            "annual_cost": f"${cheapest_cloud['annual_cost_50_daily_usd']:.2f}",
            "reasoning": f"Cheapest cloud option (${cheapest_cloud['cost_per_analysis_usd']:.5f}/case), best value with internet"
        }

    # Speed critical
    recommendations["speed_critical"] = {
        "model": fastest_model["model"] if fastest_model else "N/A",
        "annual_cost": f"${fastest_model['annual_cost_50_daily_usd']:.2f}" if fastest_model else "N/A",
        "reasoning": f"Fastest inference ({fastest_model['avg_inference_time_sec']}s), ideal for emergency triage"
    }

    # Hybrid recommendation
    if local_models and cloud_models:
        recommendations["hybrid_recommended"] = {
            "vision_model": smallest_local["model"] if smallest_local else "Local model",
            "llm_model": cheapest_cloud["model"] if cheapest_cloud else "Cloud LLM",
            "reasoning": "Local vision (offline) + cloud LLM (when connected) = best of both worlds",
            "annual_cost": f"${cheapest_cloud['annual_cost_50_daily_usd']:.2f}" if cheapest_cloud else "$0-50"
        }

    # ═══════════════════════════════════════════════════════════
    # RETURN COMPLETE DASHBOARD
    # ═══════════════════════════════════════════════════════════
    return {
        "summary": summary,
        "charts": charts,
        "tables": {
            "performance_summary": performance_table,
            "detailed_comparison": detailed_data  # Full data for advanced users
        },
        "deployment_recommendations": recommendations,
        "metadata": {
            "generated_at": datetime.now().isoformat(),
            "total_tests": len(all_results),
            "evaluation_file": str(performance_tracker.results_file),
            "confidence_explanation": "/api/evaluation/confidence-explanation"
        }
    }

@router.get("/comparison")
async def get_model_comparison():
"""
Get complete model comparison data for thesis.

    Returns combined specs + performance data.
    """
    return performance_tracker.get_comparison_data()

@router.get("/thesis-table-5-1")
async def get_table_5_1_model_specs():
"""
Table 5.1: Vision Model Specifications

    Columns: Model, Provider, Size (GB), Parameters (B), Min RAM (GB), Deployment
    """
    rows = []

    for model_name, spec in VISION_MODEL_SPECS.items():
        rows.append(
            {
                "model": spec.display_name,
                "provider": spec.platform.capitalize(),
                "size_gb": f"{spec.size_gb:.1f}" if spec.size_gb else "Cloud",
                "params_b": f"{spec.params_billions:.1f}" if spec.params_billions else "N/A",
                "min_ram_gb": spec.min_ram_gb,
                "deployment": "Local/Offline" if spec.provider == "local" else "Cloud/Online",
            }
        )

    # Sort: local first, then by size
    rows.sort(
        key=lambda x: (
            0 if x["deployment"].startswith("Local") else 1,
            float(x["size_gb"]) if x["size_gb"] != "Cloud" else 999,
        )
    )

    return {
        "table_number": "5.1",
        "title": "Vision Model Specifications and Requirements",
        "columns": [
            "Model",
            "Provider",
            "Size (GB)",
            "Parameters (B)",
            "Min RAM (GB)",
            "Deployment",
        ],
        "rows": rows,
        "caption": "Complete specifications for all vision models evaluated in the CDSS system. Local models run entirely offline on the specified hardware, while cloud models require internet connectivity but minimal local resources.",
    }

@router.get("/thesis-table-5-2")
async def get_table_5_2_performance():
"""
Table 5.2: Performance Comparison (Actual Measurements)

    Columns: Model, Avg Speed (s), Cost/Analysis, Cost/1000, Deployment
    """
    comparison = performance_tracker.get_comparison_data()

    if not comparison["models"]:
        raise HTTPException(
            status_code=404,
            detail="No performance data available. Run /api/vision/test_vision_models first.",
        )

    rows = []
    for model in comparison["models"]:
        rows.append(
            {
                "model": model["model"],
                "avg_speed_sec": model["avg_speed_sec"],
                "cost_per_analysis": f"${model['cost_per_analysis']:.5f}",
                "cost_per_1000": f"${model['cost_per_1000']:.2f}",
                "deployment": "Local" if model["provider"] == "local" else "Cloud",
                "hardware_tier": model["hardware_tier"].replace("_", " ").title(),
            }
        )

    return {
        "table_number": "5.2",
        "title": "Vision Model Performance: Speed and Cost Analysis",
        "columns": [
            "Model",
            "Avg Speed (s)",
            "Cost/Analysis",
            "Cost/1000 Analyses",
            "Deployment",
            "Hardware Tier",
        ],
        "rows": rows,
        "caption": f"Performance metrics from {comparison['summary']['total_tests_run']} actual test runs on periapical dental X-rays. Speed measured as average inference time. Costs calculated using February 2026 API pricing for cloud models; local models have zero operational cost.",
        "note": "All measurements performed on MacBook Pro M4 (16GB RAM) under identical conditions.",
    }

@router.get("/thesis-table-5-3")
async def get_table_5_3_deployment_scenarios():
"""
Table 5.3: Deployment Scenarios for Tanzania Context

    Columns: Scenario, Models, Volume, Annual Cost, Use Case
    """
    comparison = performance_tracker.get_comparison_data()

    # Find best models for each category
    local_models = [m for m in comparison["models"] if m["provider"] == "local"]
    cloud_cheap = [
        m
        for m in comparison["models"]
        if m["provider"] == "cloud" and m["cost_per_analysis"] < 0.001
    ]
    cloud_balanced = [
        m
        for m in comparison["models"]
        if m["provider"] == "cloud" and 0.001 <= m["cost_per_analysis"] < 0.01
    ]
    cloud_premium = [
        m
        for m in comparison["models"]
        if m["provider"] == "cloud" and m["cost_per_analysis"] >= 0.01
    ]

    best_local = min(local_models, key=lambda x: x["avg_speed_sec"]) if local_models else None
    best_cloud_cheap = (
        min(cloud_cheap, key=lambda x: x["cost_per_analysis"]) if cloud_cheap else None
    )
    best_cloud_fast = min(cloud_cheap, key=lambda x: x["avg_speed_sec"]) if cloud_cheap else None
    best_premium = cloud_premium[0] if cloud_premium else None

    scenarios = [
        {
            "scenario": "Rural Health Center",
            "models": best_local["model"] if best_local else "Gemma 3 4B",
            "volume": "20 cases/day",
            "annual_cost": "$0",
            "use_case": "Offline rural clinics, no internet",
            "hardware": "Mac Mini M4 ($800 one-time)",
            "connectivity": "Offline",
        },
        {
            "scenario": "District Hospital",
            "models": f"{best_local['model'] if best_local else 'Gemma 3 4B'} + {best_cloud_cheap['model'] if best_cloud_cheap else 'Gemini Flash'}",
            "volume": "50 cases/day",
            "annual_cost": (
                f"${best_cloud_cheap['annual_cost_50_daily']:.0f}" if best_cloud_cheap else "$7"
            ),
            "use_case": "Hybrid: local vision + cloud LLM",
            "hardware": "Mac Mini M4 + mobile data",
            "connectivity": "Intermittent",
        },
        {
            "scenario": "Urban Hospital",
            "models": best_cloud_cheap["model"] if best_cloud_cheap else "Gemini Flash",
            "volume": "100 cases/day",
            "annual_cost": (
                f"${(best_cloud_cheap['annual_cost_50_daily'] * 2):.0f}"
                if best_cloud_cheap
                else "$15"
            ),
            "use_case": "Fully cloud-based, lowest opex",
            "hardware": "Standard PC ($500)",
            "connectivity": "Reliable broadband",
        },
        {
            "scenario": "Emergency Triage",
            "models": best_cloud_fast["model"] if best_cloud_fast else "Groq Llama 11B",
            "volume": "30 cases/day",
            "annual_cost": (
                f"${(best_cloud_fast['cost_per_analysis'] * 30 * 365):.0f}"
                if best_cloud_fast
                else "$20"
            ),
            "use_case": "Speed-critical, 2-3s response",
            "hardware": "Any device",
            "connectivity": "Online required",
        },
        {
            "scenario": "Tertiary Referral",
            "models": best_premium["model"] if best_premium else "GPT-4o",
            "volume": "10 cases/day",
            "annual_cost": (
                f"${(best_premium['cost_per_analysis'] * 10 * 365):.0f}" if best_premium else "$91"
            ),
            "use_case": "Complex cases, second opinions",
            "hardware": "Any device",
            "connectivity": "Online required",
        },
    ]

    return {
        "table_number": "5.3",
        "title": "Deployment Scenarios for Tanzanian Healthcare Context",
        "columns": [
            "Scenario",
            "Model(s)",
            "Volume",
            "Annual Cost",
            "Use Case",
            "Hardware",
            "Connectivity",
        ],
        "rows": scenarios,
        "caption": "Recommended deployment configurations tailored to Tanzania's healthcare infrastructure. Rural scenarios prioritize offline capability ($0 opex), while urban scenarios leverage cloud services for lower capital costs. Annual costs assume daily case volumes over 365 days.",
        "context": {
            "tanzania_dentist_ratio": "1:360,000",
            "who_recommendation": "1:7,500",
            "rural_internet_penetration": "~30%",
            "urban_internet_penetration": "~80%",
        },
    }

@router.get("/thesis-figure-5-1-data")
async def get_figure_5_1_pareto_data():
"""
Figure 5.1: Speed vs Accuracy Pareto Frontier

    Returns data for plotting speed-accuracy tradeoff
    """
    comparison = performance_tracker.get_comparison_data()

    # Prepare data points for scatter plot
    data_points = []
    for model in comparison["models"]:
        # Use confidence as proxy for accuracy (or manually add accuracy data)
        accuracy = model.get("avg_confidence", 0.85) * 100  # Convert to percentage
        speed_sec = model["avg_speed_sec"]

        data_points.append(
            {
                "model": model["model"],
                "speed_sec": speed_sec,
                "accuracy_pct": accuracy,
                "cost_tier": (
                    "Free"
                    if model["cost_per_analysis"] == 0
                    else (
                        "Low"
                        if model["cost_per_analysis"] < 0.001
                        else "Medium" if model["cost_per_analysis"] < 0.01 else "High"
                    )
                ),
            }
        )

    return {
        "figure_number": "5.1",
        "title": "Speed-Accuracy Pareto Frontier of Vision Models",
        "data_points": data_points,
        "axes": {
            "x_axis": "Inference Speed (seconds, log scale)",
            "y_axis": "Diagnostic Confidence (%)",
        },
        "caption": "Tradeoff between inference speed and diagnostic confidence across all evaluated models. Models on the Pareto frontier (top-left) offer optimal speed-accuracy combinations. Color indicates cost tier.",
        "plot_type": "scatter",
        "recommended_library": "matplotlib or plotly",
    }

@router.get("/latex-tables")
async def get_latex_formatted_tables():
"""
Get all thesis tables in LaTeX format.

    Ready to copy-paste into thesis.
    """
    table_5_1 = await get_table_5_1_model_specs()
    table_5_2 = await get_table_5_2_performance()
    table_5_3 = await get_table_5_3_deployment_scenarios()

    # Generate LaTeX for Table 5.1
    latex_5_1 = f"""

\\begin{{table}}[htbp]
\\centering
\\caption{{{table_5_1['title']}}}
\\label{{tab:model-specs}}
\\begin{{tabular}}{{llcccc}}
\\toprule
{' & '.join(table*5_1['columns'])} \\\\
\\midrule
"""
for row in table_5_1["rows"]:
values = [
str(row[col.lower().replace(" ", "*").replace("(", "").replace(")", "")])
for col in table_5_1["columns"]
]
latex_5_1 += " & ".join(values) + " \\\\\n"

    latex_5_1 += """\\bottomrule

\\end{tabular}
\\end{table}
"""

    # Similar for other tables...

    return {
        "table_5_1": latex_5_1,
        "note": "Copy these LaTeX tables directly into your thesis. Requires booktabs package.",
    }

@router.get("/markdown-tables")
async def get_markdown_formatted_tables():
"""
Get all thesis tables in Markdown format.

    For README or documentation.
    """
    table_5_2 = await get_table_5_2_performance()

    # Generate Markdown
    markdown = f"## {table_5_2['title']}\n\n"
    markdown += "| " + " | ".join(table_5_2["columns"]) + " |\n"
    markdown += "|" + "|".join(["---"] * len(table_5_2["columns"])) + "|\n"

    for row in table_5_2["rows"]:
        values = [
            str(
                row[
                    col.lower()
                    .replace(" ", "_")
                    .replace("(", "")
                    .replace(")", "")
                    .replace("/", "_")
                ]
            )
            for col in table_5_2["columns"]
        ]
        markdown += "| " + " | ".join(values) + " |\n"

    markdown += f"\n*{table_5_2['caption']}*\n"

    return {"markdown": markdown, "tables": ["5.1", "5.2", "5.3"]}

@router.get("/export-csv")
async def export_to_csv():
"""Export all results to CSV for external analysis"""
performance_tracker.export_csv()
return {"message": "Results exported to evaluation_results.csv"}

@router.get("/status")
async def get_evaluation_status():
"""Get current evaluation status"""
comparison = performance_tracker.get_comparison_data()

    return {
        "total_tests_run": comparison["summary"]["total_tests_run"],
        "models_tested": comparison["summary"]["total_models_tested"],
        "local_models": comparison["summary"]["local_models"],
        "cloud_models": comparison["summary"]["cloud_models"],
        "data_file": str(performance_tracker.results_file),
    }

@router.post("/reset")
async def reset_evaluation_data():
"""
Clear all evaluation data for fresh test run.

    Use this before running comprehensive evaluation.
    """
    performance_tracker.clear()
    return {"message": "Evaluation data cleared. Ready for fresh test run."}

@router.get("/confidence-explanation")
async def get_confidence_explanation():
"""
Explain how confidence_score is determined.

    Use this in thesis methodology section.
    """
    return {
        "metric_name": "confidence_score",
        "scale": "0.0 to 1.0 (0% to 100%)",
        "source": "Model-reported intrinsic metric",

        "methodology": {
            "description": "Each vision model self-reports diagnostic certainty as part of structured output",
            "extraction_point": "Extracted from 'diagnostic_confidence' field in model JSON response",
            "fallback_strategy": "If not provided by model, assign baseline: 0.9 (premium), 0.7-0.8 (open-source), 0.5 (classifier-only)"
        },

        "interpretation": {
            "high_confidence": ">0.85 - Model is highly certain, image quality good, pathology clear",
            "moderate_confidence": "0.70-0.85 - Normal range for most cases",
            "low_confidence": "<0.70 - Ambiguous findings, poor image quality, or out-of-distribution case"
        },

        "factors_affecting_confidence": [
            "Image quality (clarity, contrast, positioning)",
            "Pathology obviousness (clear lesion vs subtle changes)",
            "Training data similarity (common vs rare conditions)",
            "Model architecture (larger models generally more confident)"
        ],

        "se_ml_perspective": {
            "uncertainty_quantification": "Critical for safety-critical AI systems",
            "trust_calibration": "Helps clinicians know when to verify AI output",
            "quality_control": "Low confidence triggers human review workflow",
            "reliability_indicator": "Track confidence trends over deployment"
        },

        "thesis_usage": {
            "chapter_3_methodology": "Explain as model-intrinsic quality metric",
            "chapter_5_results": "Report avg confidence per model with std deviation",
            "defense_answer": "Confidence is self-reported by model based on image quality, feature clarity, and training similarity"
        },

        "citation_for_defense": {
            "reference": "Guo et al., 'On Calibration of Modern Neural Networks', ICML 2017",
            "relevance": "Standard approach for uncertainty quantification in ML systems"
        }
    }

</file>

<file path="app/system_models/appointment_model/appointment_model.py">
# app/system_models/appointment_model/appointment_model.py
from sqlalchemy import Column, Integer, String, Date, DateTime, Text, ForeignKey
from sqlalchemy.orm import relationship
from app.database.connection import Base
from datetime import datetime

class Appointment(Base):
**tablename** = "appointments"
id = Column(Integer, primary_key=True, index=True)
patient_id = Column(Integer, ForeignKey("patients.id"))
appointment_date = Column(Date)
appointment_time = Column(String)
appointment_type = Column(String)
appointment_status = Column(String)
appointment_notes = Column(Text)
facility_id = Column(Integer, ForeignKey("facilities.id"))
created_at = Column(DateTime, default=datetime.utcnow)
updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
patient = relationship("Patient", back_populates="appointments")
facility = relationship("Facility", back_populates="appointments")
</file>

<file path="app/system_models/appointment_model/appointment_schemas.py">
# app/system_models/appointment_model/appointment_schemas.py
from typing import Optional, List, Literal
from datetime import date, datetime
from pydantic import BaseModel, Field, field_validator

class AppointmentBase(BaseModel):
patient_id: int
appointment_date: date
appointment_time: str
appointment_type: str
appointment_status: str
appointment_notes: str
facility_id: Optional[int] = None

class AppointmentCreate(AppointmentBase):
pass

class AppointmentUpdate(AppointmentBase):
pass

class AppointmentResponse(AppointmentBase):
id: int
created_at: datetime
updated_at: datetime
class Config:
from_attributes = True
</file>

<file path="app/system_models/clinical_note_model/clinical_note_model.py">
# app/system_models/clinical_note_model/clinical_note_model.py
from sqlalchemy import Column, Integer, String, Text, DateTime, ForeignKey
from sqlalchemy.orm import relationship
from app.database.connection import Base
from app.helpers.time import utcnow

class ClinicalNote(Base):
**tablename** = "clinical_notes"

    id = Column(Integer, primary_key=True)
    encounter_id = Column(Integer, ForeignKey("encounters.id"), nullable=False)
    author_id = Column(Integer, ForeignKey("users.id"), nullable=False)

    note_type = Column(String, default="SOAP")
    content = Column(Text, nullable=False)

    created_at = Column(DateTime(timezone=True), default=utcnow)

    encounter = relationship("Encounter", back_populates="notes")
    author = relationship("User")

</file>

<file path="app/system_models/clinical_note_model/clinical_note_schemas.py">
# app/system_models/clinical_note_model/clinical_note_schemas.py
from typing import Optional
from datetime import datetime
from pydantic import BaseModel

class ClinicalNoteBase(BaseModel):
encounter_id: int
author_id: int
note_type: str = "SOAP"
content: str

class ClinicalNoteCreate(ClinicalNoteBase):
pass

class ClinicalNoteUpdate(BaseModel):
note_type: Optional[str] = None
content: Optional[str] = None

class ClinicalNoteResponse(ClinicalNoteBase):
id: int
created_at: datetime

    class Config:
        from_attributes = True

</file>

<file path="app/system_models/diagnosis_model/diagnosis_model.py">
# app/system_models/diagnosis_model/diagnosis_model.py
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import relationship
from app.database.connection import Base

class Diagnosis(Base):
**tablename** = "diagnoses"

    id = Column(Integer, primary_key=True)
    encounter_id = Column(Integer, ForeignKey("encounters.id"), nullable=False)

    code = Column(String, nullable=True)  # ICD-10 / SNOMED
    description = Column(String, nullable=False)
    diagnosis_type = Column(String, default="primary")

    encounter = relationship("Encounter", back_populates="diagnoses")

</file>

<file path="app/system_models/diagnosis_model/diagnosis_schemas.py">
# app/system_models/diagnosis_model/diagnosis_schemas.py
from typing import Optional
from pydantic import BaseModel

class DiagnosisBase(BaseModel):
encounter_id: int
code: Optional[str] = None
description: str
diagnosis_type: str = "primary"

class DiagnosisCreate(DiagnosisBase):
pass

class DiagnosisUpdate(BaseModel):
code: Optional[str] = None
description: Optional[str] = None
diagnosis_type: Optional[str] = None

class DiagnosisResponse(DiagnosisBase):
id: int

    class Config:
        from_attributes = True

</file>

<file path="app/system_models/encounter_model/encounter_model.py">
# app/system_models/encounter_model/encounter_model.py
from sqlalchemy import Column, Integer, String, DateTime, ForeignKey
from sqlalchemy.orm import relationship
from app.database.connection import Base
from app.helpers.time import utcnow

class Encounter(Base):
**tablename** = "encounters"

    id = Column(Integer, primary_key=True, index=True)

    patient_id = Column(Integer, ForeignKey("patients.id"), nullable=False)
    facility_id = Column(Integer, ForeignKey("facilities.id"), nullable=False)
    provider_id = Column(Integer, ForeignKey("users.id"), nullable=False)

    reason_for_visit = Column(String, nullable=True)
    encounter_type = Column(String, default="outpatient")  # outpatient, inpatient, emergency
    started_at = Column(DateTime(timezone=True), default=utcnow)
    closed_at = Column(DateTime(timezone=True), nullable=True)

    patient = relationship("Patient", back_populates="encounters")
    provider = relationship("User")
    diagnoses = relationship("Diagnosis", back_populates="encounter", cascade="all, delete-orphan")
    prescriptions = relationship("Prescription", back_populates="encounter", cascade="all, delete-orphan")
    notes = relationship("ClinicalNote", back_populates="encounter", cascade="all, delete-orphan")

</file>

<file path="app/system_models/encounter_model/encounter_schemas.py">
# app/system_models/encounter_model/encounter_schemas.py
from typing import Optional
from datetime import datetime
from pydantic import BaseModel

class EncounterBase(BaseModel):
patient_id: int
facility_id: int
provider_id: int
reason_for_visit: Optional[str]
encounter_type: str = "outpatient"

class EncounterCreate(EncounterBase):
pass

class EncounterResponse(EncounterBase):
id: int
started_at: datetime
closed_at: Optional[datetime]

    class Config:
        from_attributes = True

</file>

<file path="app/system_models/facility_model/facility_model.py">
# app/system_models/facility_model/facility_model.py
from sqlalchemy import Column, Integer, String, DateTime
from sqlalchemy.orm import relationship
from app.database.connection import Base
from app.helpers.time import utcnow

class Facility(Base):
**tablename** = "facilities"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, nullable=False, index=True)
    address = Column(String, nullable=True)
    phone = Column(String, nullable=True)

    created_at = Column(DateTime(timezone=True), default=utcnow)
    updated_at = Column(DateTime(timezone=True), default=utcnow, onupdate=utcnow)

    # Relationships
    users = relationship("User", back_populates="facility")
    patients = relationship("Patient", back_populates="facility")
    appointments = relationship("Appointment", back_populates="facility")

    def __repr__(self):
        return f"<Facility(id={self.id}, name='{self.name}')>"

</file>

<file path="app/system_models/facility_model/facility_schemas.py">
# app/system_models/facility_model/facility_schemas.py
from typing import Optional, List, Literal
from datetime import date, datetime
from pydantic import BaseModel, Field, field_validator

# facilities

class FacilityBase(BaseModel):
name: str
address: Optional[str] = None
phone: Optional[str] = None

class FacilityCreate(FacilityBase):
pass

class FacilityUpdate(FacilityBase):
name: Optional[str] = None

class FacilityResponse(FacilityBase):
id: int
created_at: datetime
updated_at: datetime
class Config:
from_attributes = True
</file>

<file path="app/system_models/patient_model/patient_model.py">
# app/system_models/patient_model/patient_model.py
from sqlalchemy import Column, Integer, String, Date, DateTime, ForeignKey, CheckConstraint
from sqlalchemy.orm import relationship
from app.database.connection import Base
from app.helpers.time import utcnow

class Patient(Base):
**tablename** = "patients"

    id = Column(Integer, primary_key=True, index=True)

    first_name = Column(String, nullable=False)
    last_name = Column(String, nullable=False)
    dob = Column(Date, nullable=True)
    gender = Column(String, nullable=True)
    contact_info = Column(String, nullable=True)

    facility_id = Column(Integer, ForeignKey("facilities.id"), nullable=False)

    created_at = Column(DateTime(timezone=True), default=utcnow)
    updated_at = Column(DateTime(timezone=True), default=utcnow, onupdate=utcnow)

    __table_args__ = (
        CheckConstraint("gender IN ('male', 'female', 'other')", name="check_gender_values"),
    )

    facility = relationship("Facility", back_populates="patients")
    encounters = relationship("Encounter", back_populates="patient", cascade="all, delete-orphan")
    appointments = relationship("Appointment", back_populates="patient")

    def __repr__(self):
        return f"<Patient {self.id}: {self.first_name} {self.last_name}>"

</file>

<file path="app/system_models/patient_model/patient_schemas.py">
# app/system_models/patient_model/patient_schemas.py
from typing import Optional, Literal
from datetime import date, datetime
from pydantic import BaseModel, field_validator

GENDER = Literal["male", "female", "other"]

class PatientBase(BaseModel):
first_name: str
last_name: str
dob: Optional[date]
gender: Optional[GENDER]
contact_info: Optional[str]
facility_id: int

class PatientCreate(PatientBase):
pass

class PatientUpdate(BaseModel):
first_name: Optional[str]
last_name: Optional[str]
dob: Optional[date]
gender: Optional[GENDER]
contact_info: Optional[str]

class PatientResponse(PatientBase):
id: int
created_at: datetime
updated_at: datetime

    class Config:
        from_attributes = True

</file>

<file path="app/system_models/prescription_model/prescription_model.py">
# app/system_models/prescription_model/prescription_model.py
from sqlalchemy import Column, Integer, String, DateTime, ForeignKey
from sqlalchemy.orm import relationship
from app.database.connection import Base
from app.helpers.time import utcnow

class Prescription(Base):
**tablename** = "prescriptions"

    id = Column(Integer, primary_key=True)
    encounter_id = Column(Integer, ForeignKey("encounters.id"), nullable=False)

    drug_name = Column(String, nullable=False)
    dosage = Column(String)
    frequency = Column(String)
    duration = Column(String)
    route = Column(String)

    prescribed_at = Column(DateTime(timezone=True), default=utcnow)

    encounter = relationship("Encounter", back_populates="prescriptions")

</file>

<file path="app/system_models/prescription_model/prescription_schemas.py">
# app/system_models/prescription_model/prescription_schemas.py
from typing import Optional
from datetime import datetime
from pydantic import BaseModel

class PrescriptionBase(BaseModel):
encounter_id: int
drug_name: str
dosage: Optional[str] = None
frequency: Optional[str] = None
duration: Optional[str] = None
route: Optional[str] = None

class PrescriptionCreate(PrescriptionBase):
pass

class PrescriptionUpdate(BaseModel):
drug_name: Optional[str] = None
dosage: Optional[str] = None
frequency: Optional[str] = None
duration: Optional[str] = None
route: Optional[str] = None

class PrescriptionResponse(PrescriptionBase):
id: int
prescribed_at: datetime

    class Config:
        from_attributes = True

</file>

<file path="app/system_services/__init__.py">

</file>

<file path="app/system_services/create_appointment.py">
from sqlalchemy.ext.asyncio import AsyncSession
from app.system_models.appointment_model.appointment_model import Appointment
from app.system_models.appointment_model.appointment_schemas import AppointmentCreate

async def create_appointment(db: AsyncSession, appointment: AppointmentCreate):
"""Create a new appointment."""
db_appointment = Appointment(\*\*appointment.model_dump())
db.add(db_appointment)
await db.commit()
await db.refresh(db_appointment)
return db_appointment
</file>

<file path="app/system_services/create_clinical_note.py">
from sqlalchemy.ext.asyncio import AsyncSession
from app.system_models.clinical_note_model.clinical_note_model import ClinicalNote
from app.system_models.clinical_note_model.clinical_note_schemas import ClinicalNoteCreate

async def create_clinical_note(db: AsyncSession, clinical_note: ClinicalNoteCreate):
"""Create a new clinical note."""
db_clinical_note = ClinicalNote(\*\*clinical_note.model_dump())
db.add(db_clinical_note)
await db.commit()
await db.refresh(db_clinical_note)
return db_clinical_note
</file>

<file path="app/system_services/create_diagnosis.py">
from sqlalchemy.ext.asyncio import AsyncSession
from app.system_models.diagnosis_model.diagnosis_model import Diagnosis
from app.system_models.diagnosis_model.diagnosis_schemas import DiagnosisCreate

async def create_diagnosis(db: AsyncSession, diagnosis: DiagnosisCreate):
"""Create a new diagnosis."""
db_diagnosis = Diagnosis(\*\*diagnosis.model_dump())
db.add(db_diagnosis)
await db.commit()
await db.refresh(db_diagnosis)
return db_diagnosis
</file>

<file path="app/system_services/create_encounter.py">
from sqlalchemy.ext.asyncio import AsyncSession
from app.system_models.encounter_model.encounter_model import Encounter
from app.system_models.encounter_model.encounter_schemas import EncounterCreate

async def create_encounter(db: AsyncSession, encounter: EncounterCreate):
"""Create a new encounter."""
db_encounter = Encounter(\*\*encounter.model_dump())
db.add(db_encounter)
await db.commit()
await db.refresh(db_encounter)
return db_encounter
</file>

<file path="app/system_services/create_facility.py">
# app/system_services/create_facility.py
from sqlalchemy.ext.asyncio import AsyncSession
from app.system_models.facility_model.facility_model import Facility
from app.system_models.facility_model.facility_schemas import FacilityCreate

async def create_facility(db: AsyncSession, facility: FacilityCreate):
"""Create a new facility."""
db_facility = Facility(\*\*facility.model_dump())
db.add(db_facility)
await db.commit()
await db.refresh(db_facility)
return db_facility
</file>

<file path="app/system_services/create_patient.py">
# app/system_services/create_patient.py
from fastapi import Depends
from sqlalchemy.ext.asyncio import AsyncSession

from app.users.auth_dependencies import get_current_user_id
from app.system_models.patient_model.patient_model import Patient
from app.system_models.patient_model.patient_schemas import PatientCreate

async def create_patient(
db: AsyncSession,
patient: PatientCreate,
):
"""Create a new patient."""
db_patient = Patient(\*\*patient.model_dump())
db.add(db_patient)
await db.commit()
await db.refresh(db_patient)
return db_patient
</file>

<file path="app/system_services/create_prescription.py">
from sqlalchemy.ext.asyncio import AsyncSession
from app.system_models.prescription_model.prescription_model import Prescription
from app.system_models.prescription_model.prescription_schemas import PrescriptionCreate

async def create_prescription(db: AsyncSession, prescription: PrescriptionCreate):
"""Create a new prescription."""
db_prescription = Prescription(\*\*prescription.model_dump())
db.add(db_prescription)
await db.commit()
await db.refresh(db_prescription)
return db_prescription
</file>

<file path="app/system_services/system_routes.py">
# app/system_services/system_routes.py
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.ext.asyncio import AsyncSession

from app.database.connection import get_db
from app.users.auth_dependencies import get_current_admin, get_current_user_id
from app.system_models.appointment_model.appointment_schemas import (
AppointmentCreate,
AppointmentResponse,
)
from app.system_models.clinical_note_model.clinical_note_schemas import (
ClinicalNoteCreate,
ClinicalNoteResponse,
)
from app.system_models.diagnosis_model.diagnosis_schemas import (
DiagnosisCreate,
DiagnosisResponse,
)
from app.system_models.encounter_model.encounter_schemas import (
EncounterCreate,
EncounterResponse,
)
from app.system_models.facility_model.facility_schemas import (
FacilityCreate,
FacilityResponse,
)
from app.system_models.patient_model.patient_schemas import (
PatientCreate,
PatientResponse,
)
from app.system_models.prescription_model.prescription_schemas import (
PrescriptionCreate,
PrescriptionResponse,
)
from app.system_services.create_appointment import create_appointment
from app.system_services.create_clinical_note import create_clinical_note
from app.system_services.create_diagnosis import create_diagnosis
from app.system_services.create_encounter import create_encounter
from app.system_services.create_facility import create_facility
from app.system_services.create_patient import create_patient
from app.system_services.create_prescription import create_prescription

router = APIRouter()

@router.post("/create_facility", response_model=FacilityResponse)
async def create_facility_endpoint(
facility: FacilityCreate,
db: AsyncSession = Depends(get_db),
admin: bool = Depends(get_current_admin),
):
"""Create a new facility."""
try:
db_facility = await create_facility(db, facility)
return db_facility
except Exception as e:
raise HTTPException(status_code=500, detail=str(e))

@router.post("/create_patient", response_model=PatientResponse)
async def create_patient_endpoint(
patient: PatientCreate,
db: AsyncSession = Depends(get_db),
user_id: int = Depends(get_current_user_id),
):
"""Create a new patient."""
try:
db_patient = await create_patient(db, patient)
return db_patient
except Exception as e:
raise HTTPException(status_code=500, detail=str(e))

@router.post("/create_appointment", response_model=AppointmentResponse)
async def create_appointment_endpoint(
appointment: AppointmentCreate, db: AsyncSession = Depends(get_db)
):
"""Create a new appointment."""
try:
db_appointment = await create_appointment(db, appointment)
return db_appointment
except Exception as e:
raise HTTPException(status_code=500, detail=str(e))

@router.post("/create_clinical_note", response_model=ClinicalNoteResponse)
async def create_clinical_note_endpoint(
clinical_note: ClinicalNoteCreate,
db: AsyncSession = Depends(get_db),
user_id: int = Depends(get_current_user_id),
):
"""Create a new clinical note."""
try:
db_clinical_note = await create_clinical_note(db, clinical_note)
return db_clinical_note
except Exception as e:
raise HTTPException(status_code=500, detail=str(e))

@router.post("/create_diagnosis", response_model=DiagnosisResponse)
async def create_diagnosis_endpoint(diagnosis: DiagnosisCreate, db: AsyncSession = Depends(get_db)):
"""Create a new diagnosis."""
try:
db_diagnosis = await create_diagnosis(db, diagnosis)
return db_diagnosis
except Exception as e:
raise HTTPException(status_code=500, detail=str(e))

@router.post("/create_encounter", response_model=EncounterResponse)
async def create_encounter_endpoint(encounter: EncounterCreate, db: AsyncSession = Depends(get_db)):
"""Create a new encounter."""
try:
db_encounter = await create_encounter(db, encounter)
return db_encounter
except Exception as e:
raise HTTPException(status_code=500, detail=str(e))

@router.post("/create_prescription", response_model=PrescriptionResponse)
async def create_prescription_endpoint(
prescription: PrescriptionCreate, db: AsyncSession = Depends(get_db)
):
"""Create a new prescription."""
try:
db_prescription = await create_prescription(db, prescription)
return db_prescription
except Exception as e:
raise HTTPException(status_code=500, detail=str(e))
</file>

<file path="app/users/auth_token_model/token_model.py">
# app/users/auth_token_model/token_model.py
from sqlalchemy import Column, Integer, String, DateTime, Boolean, ForeignKey
from sqlalchemy.orm import relationship
from datetime import datetime
from app.database.connection import Base
from app.helpers.time import utcnow

class Token(Base):
**tablename** = "tokens"

    id = Column(Integer, primary_key=True, index=True)
    token_string = Column(String, unique=True, index=True, nullable=False)
    token_type = Column(String, nullable=False)
    user_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    expires_at = Column(DateTime(timezone=True), nullable=False)
    is_revoked = Column(Boolean, default=False)
    created_at = Column(DateTime(timezone=True), default=utcnow)
    updated_at = Column(DateTime(timezone=True), default=utcnow, onupdate=utcnow)

    user = relationship("User", back_populates="tokens")

</file>

<file path="app/users/auth_token_model/token_schemas.py">
# app/users/auth_token_model/token_schemas.py
from typing import  Literal
from datetime import date, datetime
from pydantic import BaseModel

# Allowed token types as constants

TOKEN_TYPE = Literal["access", "refresh"]

# tokens schemas

class TokenBase(BaseModel):
token_string: str
token_type: TOKEN_TYPE
user_id: int
expires_at: datetime
is_revoked: bool = False

class TokenCreate(TokenBase):
pass

class TokenResponse(TokenBase):
pass
</file>

<file path="app/users/password_reset_token/password_reset_token_model.py">
# app/users/password_reset_token/password_reset_token_model.py
from sqlalchemy import Column, Integer, String, Boolean, DateTime, ForeignKey
from sqlalchemy.orm import relationship
from app.database.connection import Base
from app.helpers.time import utcnow
from datetime import datetime

class PasswordResetToken(Base):
**tablename** = "password_reset_tokens"

    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey("users.id"), nullable=False, index=True)
    token = Column(String, unique=True, index=True, nullable=False)
    expires_at = Column(DateTime(timezone=True), nullable=False)
    is_used = Column(Boolean, default=False)
    created_at = Column(DateTime(timezone=True), default=utcnow)

    # Relationships
    user = relationship("User", back_populates="reset_tokens")

    def __repr__(self):
        return f"<PasswordResetToken(id={self.id}, user_id={self.user_id}, is_used={self.is_used})>"

</file>

<file path="app/users/password_reset_token/password_reset_token_schema.py">
# app/users/password_reset_token/password_reset_token_schema.py
from pydantic import BaseModel
from datetime import datetime
from typing import Optional

class PasswordResetTokenBase(BaseModel):
token: str
is_used: bool = False
expires_at: datetime

class PasswordResetTokenCreate(PasswordResetTokenBase):
user_id: int

class PasswordResetTokenResponse(PasswordResetTokenBase):
id: int
user_id: int
created_at: datetime

    class Config:
        from_attributes = True

</file>

<file path="app/users/user_models/schemas.py">
# app/users/user_models/schemas.py

from datetime import datetime
from typing import List, Literal, Optional

from pydantic import BaseModel, ConfigDict, EmailStr, Field, field_validator

# Allowed values as constants

ROLES = Literal["admin", "doctor"]

class UserBase(BaseModel):
email: EmailStr
model_config = ConfigDict(from_attributes=True)

# ✅ Request schema for registration

class UserRegister(BaseModel):
email: EmailStr
password: str = Field(..., min_length=8, max_length=100)
first_name: Optional[str] = None
last_name: Optional[str] = None
is_active: bool = True
is_verified: bool = False
created_at: datetime = Field(default_factory=datetime.utcnow)
updated_at: datetime = Field(default_factory=datetime.utcnow)
role: ROLES = "doctor"

    @field_validator("email", mode="before")
    def normalize_email(cls, v):
        # sourcery skip: assign-if-exp, reintroduce-else
        if isinstance(v, str):
            return v.strip().lower()
        return v

    @field_validator("role")
    def validate_role_field(cls, v):
        if v not in ["admin", "doctor"]:
            raise ValueError(f"Invalid role value: {v}")
        return v

# ✅ Response schema for user registration

class UserRegisterResponse(UserBase):
id: int
email: str
first_name: Optional[str] = None
last_name: Optional[str] = None

# ✅ User login request

class UserLogin(BaseModel):
email: EmailStr
password: str

# ✅ Response schema for user info

class UserResponse(BaseModel):
id: int
email: str
is_active: bool
is_verified: bool
created_at: datetime
first_name: Optional[str] = None
last_name: Optional[str] = None
model_config = ConfigDict(from_attributes=True)

class UserInDB(UserRegister):
id: int
is_active: bool
role: ROLES

    hashed_password: str  # For internal use, not API responses

class UserList(BaseModel):
users: List[UserResponse]
total: int

class UserPublic(BaseModel):
id: int
role: ROLES

# ✅ Response schema for user login

class UserLoginResponse(BaseModel):
access_token: str
refresh_token: str
token_type: str
user: UserResponse

# ✅ Response schema for user logout

class UserLogoutResponse(BaseModel):
message: str

# ✅ Response schema for token refresh

class UserRefreshResponse(BaseModel):
access_token: str
refresh_token: str
token_type: str

# ✅ Request schema for forgot password

class UserForgotPassword(BaseModel):
email: EmailStr

# ✅ Request schema for reset password

class UserResetPassword(BaseModel):
token: str
new_password: str = Field(..., min_length=8, max_length=100)

# ✅ Request schema for verify email

class UserVerifyEmail(BaseModel):
email: EmailStr
verification_code: str

# ✅ Request schema for change password (authenticated)

class UserChangePassword(BaseModel):
current_password: str
new_password: str = Field(..., min_length=8, max_length=100)
</file>

<file path="app/users/user_models/user_model.py">
# app/users/user_models/user_model.py
from typing import Optional
from pydantic import BaseModel, Field
from config.appconfig import settings
from sqlalchemy import Column, Integer, String, Boolean, DateTime, CheckConstraint, ForeignKey
from sqlalchemy.orm import relationship
from app.database.connection import Base
from app.helpers.time import utcnow

class User(Base):
**tablename** = "users"

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True, nullable=False)
    hashed_password = Column(String, nullable=False)
    is_active = Column(Boolean, default=True)
    is_verified = Column(Boolean, default=False)
    created_at = Column(DateTime(timezone=True), default=utcnow)
    updated_at = Column(DateTime(timezone=True), default=utcnow, onupdate=utcnow)
    verification_code = Column(String, nullable=True)
    role = Column(String, default="doctor", nullable=False)

    facility_id = Column(Integer, ForeignKey("facilities.id"), nullable=True)
    facility = relationship("Facility", back_populates="users")
    tokens = relationship("Token", back_populates="user")
    reset_tokens = relationship("PasswordResetToken", back_populates="user")

    # Optional: Add more fields as needed
    first_name = Column(String, nullable=True)
    last_name = Column(String, nullable=True)



    # Add check constraints for validation at database level
    __table_args__ = (
        CheckConstraint("role IN ('admin', 'doctor')", name='check_role_values'),
    )

    def __repr__(self):
        return f"<User(id={self.id}, first_name='{self.first_name}', last_name='{self.last_name}', email='{self.email}')>"

</file>

<file path="app/users/__init__.py">

</file>

<file path="app/users/auth_dependencies.py">
# app/users/auth_dependencies.py
# Centralized Authentication Dependencies

from typing import Optional
from fastapi import Depends, HTTPException, status, Cookie
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from sqlalchemy import select, and\_
from sqlalchemy.ext.asyncio import AsyncSession

from app.database.connection import get_db
from app.users.user_models.user_model import User
from app.users.auth_token_model.token_model import Token
from app.users.security import decode_token

# Security schemes

security_scheme = HTTPBearer(auto_error=False) # Don't auto-raise for cookie fallback

async def get_current_user(
credentials: Optional[HTTPAuthorizationCredentials] = Depends(security_scheme),
access_token_cookie: Optional[str] = Cookie(None, alias="access_token"),
db: AsyncSession = Depends(get_db)
) -> User:
"""
Get current authenticated user with PROPER token revocation check.

    Accepts token from EITHER:
    - Authorization: Bearer header (for mobile/Postman)
    - httpOnly cookie (for web browsers)

    Validates:
    1. JWT signature and expiry
    2. Token exists in database
    3. Token is not revoked (THE CRITICAL FIX)
    4. User exists and is active

    Raises 401 if any validation fails.
    """
    # Extract token from header or cookie
    token_string = None
    if credentials:
        token_string = credentials.credentials
    elif access_token_cookie:
        token_string = access_token_cookie
    else:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Not authenticated. Provide token in Authorization header or cookie.",
            headers={"WWW-Authenticate": "Bearer"},
        )

    # 1. Decode JWT (validates signature + expiry)
    payload = decode_token(token_string)
    if not payload or payload.get("type") != "access":
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid or expired access token",
            headers={"WWW-Authenticate": "Bearer"},
        )

    # 2. Extract user identifier
    user_email = payload.get("sub")
    if not user_email:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Token missing user identifier",
            headers={"WWW-Authenticate": "Bearer"},
        )

    # ═══════════════════════════════════════════════════════════════
    # 3. THE FIX: Check token revocation status in database
    # ═══════════════════════════════════════════════════════════════
    token_record = await db.execute(
        select(Token).where(
            and_(
                Token.token_string == token_string,
                Token.token_type == "access"
            )
        )
    )
    token_obj = token_record.scalars().first()

    # Token not in DB → invalid (shouldn't happen but be defensive)
    if not token_obj:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Token not found in database. Please log in again.",
            headers={"WWW-Authenticate": "Bearer"},
        )

    # Token revoked → user logged out
    if token_obj.is_revoked:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Token has been revoked. Please log in again.",
            headers={"WWW-Authenticate": "Bearer"},
        )
    # ═══════════════════════════════════════════════════════════════

    # 4. Fetch user from database
    result = await db.execute(
        select(User).where(User.email == user_email)
    )
    user = result.scalars().first()

    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="User not found",
            headers={"WWW-Authenticate": "Bearer"},
        )

    # 5. Check user status
    if not user.is_active:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="User account is inactive"
        )

    # Optional: Check email verification
    # if not user.is_verified:
    #     raise HTTPException(
    #         status_code=status.HTTP_403_FORBIDDEN,
    #         detail="Email not verified"
    #     )

    return user

async def get_current_user_id(
current_user: User = Depends(get_current_user)
) -> int:
"""
Get just the user ID.
Convenience wrapper around get_current_user.
"""
return current_user.id

async def get_current_admin(
current_user: User = Depends(get_current_user)
) -> User:
"""
Require admin role.
Raises 403 if user is not admin.
"""
if current_user.role != "admin":
raise HTTPException(
status_code=status.HTTP_403_FORBIDDEN,
detail="Admin privileges required"
)
return current_user

async def get_current_user_optional(
credentials: Optional[HTTPAuthorizationCredentials] = Depends(
HTTPBearer(auto_error=False)
),
access_token_cookie: Optional[str] = Cookie(None, alias="access_token"),
db: AsyncSession = Depends(get_db)
) -> Optional[User]:
"""
Get user if authenticated, otherwise None.
For endpoints that work with or without auth.
"""
if not credentials and not access_token_cookie:
return None

    try:
        return await get_current_user(credentials, access_token_cookie, db)
    except HTTPException:
        return None  # Invalid/expired token → treat as anonymous

</file>

<file path="app/users/auth_emails.py">
# app/users/auth_emails.py

from config.appconfig import settings
from fastapi import HTTPException
import resend
import os

resend.api_key = settings.RESEND_API_KEY

# =================================================

# ✅ send registration email with verification code

# =================================================

def send_registration_email_with_verification_code(email, verification_code, first_name="User"):
if not first_name:
first_name = "User"

    r = resend.Emails.send(
        {
            "from": "support@medivarse.com",
            "to": email,
            "subject": "Verify your email!",
            "html": f"<p>Hello {first_name}. \
        Welcome to dentoplus. \
        To verify your account, use the code below when prompted to enter your verification code. \
        This code is meant to not be shared to anyone</p>"
            f"<p><strong> {verification_code} </strong></p>"
            f"<p><strong> dentoplus </strong></p>",
        }
    )

# =================================================

# ✅ send reset password link with token in email

# =================================================

def send_reset_password_link_with_token_in_email(email, reset_link, first_name="User"):
if not first_name:
first_name = "User"

    r = resend.Emails.send(
        {
            "from": "support@medivarse.com",
            "to": email,
            "subject": "Reset your password!",
            "html": f"<p> Hello {first_name}. We are sorry to hear that you have been having trouble logging in on your dentoplus account. \
          To reset your password, click the link below</p>"
            f"<p><a href='{reset_link}'>{reset_link}</a></p>"
            "<p>You can only use this link once, not to be shared to anyone</p>"
            f"<p><strong> dentoplus </strong></p>",
        }
    )

</file>

<file path="app/users/auth_routers.py">
# app/users/auth_routers.py

from fastapi import APIRouter, Depends, HTTPException, Body
from app.users.auth_services import (
registering_user,
authenticate_user,
login_user,
refresh_access_token,
logout_user,
create_password_reset_link,
verify_email_with_code,
update_password,
reset_password_with_token,
)
from app.users.user_models.schemas import (
UserRegister,
UserLogin,
UserResponse,
UserRegisterResponse,
UserLoginResponse,
UserLogoutResponse,
UserRefreshResponse,
UserForgotPassword,
UserResetPassword,
UserVerifyEmail,
UserChangePassword,
)
from app.users.user_models.user_model import User
from typing import Optional, Tuple
from app.database.connection import get_db
from sqlalchemy.ext.asyncio import AsyncSession
from app.users.auth_dependencies import get_current_user

router = APIRouter()

# ============================================================

# ✅ REGISTER

# ============================================================

@router.post("/register", response_model=UserRegisterResponse)
async def register_user(user_data: UserRegister, db: AsyncSession = Depends(get_db)) -> UserRegisterResponse:
return await registering_user(user_data, db)

# ============================================================

# ✅ AUTHENTICATE USER (LOGIN)

# ============================================================

@router.post("/login", response_model=UserLoginResponse)
async def login(user_data: UserLogin, db: AsyncSession = Depends(get_db)) -> UserLoginResponse:
access_token, refresh_token, user = await login_user(user_data, db)
return UserLoginResponse(
access_token=access_token,
refresh_token=refresh_token,
token_type="bearer",
user=user

    )

# ============================================================

# ✅ REFRESH TOKEN

# ============================================================

@router.post("/refresh", response_model=UserRefreshResponse)
async def refresh_token(
refresh_token: str = Body(..., embed=True),
db: AsyncSession = Depends(get_db)
) -> UserRefreshResponse:
access_token, new_refresh_token = await refresh_access_token(refresh_token, db)
return UserRefreshResponse(
access_token=access_token,
refresh_token=new_refresh_token,
token_type="bearer"
)

# ============================================================

# ✅ LOGOUT USER

# ============================================================

# Note: This requires the token to be passed. Since we use Bearer token, we can extract it.

# We will use a dependency to get the token string if we want to revoke it.

from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
security = HTTPBearer()

@router.post("/logout", response_model=UserLogoutResponse)
async def logout(
current_user: User = Depends(get_current_user),
db: AsyncSession = Depends(get_db)
) -> UserLogoutResponse:
await logout_user(current_user, db)
return UserLogoutResponse(message="Successfully logged out of all devices")

# ============================================================

# ✅ CHANGE PASSWORD

# ============================================================

@router.post("/change-password", response_model=UserLogoutResponse)
async def change_password(
data: UserChangePassword,
current_user: User = Depends(get_current_user),
db: AsyncSession = Depends(get_db)
) -> UserLogoutResponse:
await update_password(current_user, data.current_password, data.new_password, db)
return {"message": "Password changed successfully"}

# ============================================================

# ✅ FORGOT PASSWORD

# ============================================================

@router.post("/forgot-password")
async def forgot_password(
data: UserForgotPassword,
db: AsyncSession = Depends(get_db)
):
message, token = await create_password_reset_link(data.email, db) # In production, send email here.
return {"message": message, "reset_token": token}

# ============================================================

# ✅ RESET PASSWORD

# ============================================================

@router.post("/reset-password")
async def reset_password(
data: UserResetPassword,
db: AsyncSession = Depends(get_db)
): # Verify token and reset password
await reset_password_with_token(data.token, data.new_password, db)
return {"message": "Password reset successful"}

# ============================================================

# ✅ VERIFY EMAIL

# ============================================================

@router.post("/verify-email")
async def verify_email(
data: UserVerifyEmail,
db: AsyncSession = Depends(get_db)
):
from sqlalchemy.future import select
result = await db.execute(select(User).where(User.email == data.email))
user = result.scalars().first()
if not user:
raise HTTPException(status_code=404, detail="User not found")

    await verify_email_with_code(user, data.verification_code, db)
    return {"message": "Email verified successfully"}

</file>

<file path="app/users/auth_services.py">
from fastapi import Depends, HTTPException, status
from sqlalchemy.future import select
from sqlalchemy.ext.asyncio import AsyncSession
from typing import Optional

from app.users.user_models.schemas import UserRegisterResponse, UserLogin, UserResponse, UserRegister
from app.users.user_models.user_model import User
from app.database.connection import get_db
from app.users.security import (
get_password_hash,
verify_password,
create_access_token,
create_refresh_token,
generate_password_reset_token,
decode_token,
generate_verification_code,
get_password_hash
)
from app.users.auth_token_model.token_model import Token
from app.users.password_reset_token.password_reset_token_model import PasswordResetToken
from app.users.auth_emails import send_registration_email_with_verification_code, send_reset_password_link_with_token_in_email
from config.appconfig import settings
from app.helpers.time import utcnow
from datetime import timedelta

# ============================================================

# ✅ REGISTER A NEW USER

# ============================================================

async def registering_user (user_data: UserRegister, db: AsyncSession = Depends(get_db)) -> UserRegisterResponse: # Check if user already exists
result = await db.execute(select(User).where(User.email == user_data.email))
existing_user = result.scalars().first()
if existing_user:
raise HTTPException(
status_code=status.HTTP_400_BAD_REQUEST,
detail="User with this email already exists"
)

    # Hash password
    hashed_password = get_password_hash(user_data.password)

    # Create new user
    new_user = User(
        email=user_data.email,
        hashed_password=hashed_password,
        first_name=user_data.first_name,
        last_name=user_data.last_name,
        is_active=user_data.is_active,
        is_verified=user_data.is_verified,
        role=user_data.role,
        verification_code=generate_verification_code() # Generate verification code on signup
    )

    db.add(new_user)
    await db.commit()
    await db.refresh(new_user)

    # Send verification email
    try:
        send_registration_email_with_verification_code(new_user.email, new_user.verification_code, new_user.first_name)
    except Exception as e:
        print(f"Error sending registration email: {e}")

    return new_user

# ============================================================

# ✅ AUTHENTICATE USER

# ============================================================

async def authenticate_user(
email: str, password: str, db: AsyncSession
) -> Optional[User]:
result = await db.execute(select(User).where(User.email == email))
user = result.scalars().first()

    if not user:
        return None

    if not verify_password(password, user.hashed_password):
        return None

    return user

# ============================================================

# ✅ LOGIN USER

# ============================================================

async def login_user(user_data: UserLogin, db: AsyncSession) -> tuple[str, str, UserResponse]:
user = await authenticate_user(user_data.email, user_data.password, db)
if not user:
raise HTTPException(
status_code=status.HTTP_401_UNAUTHORIZED,
detail="Incorrect email or password",
headers={"WWW-Authenticate": "Bearer"},
)

    if not user.is_active:
         raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Inactive user"
        )

    # Create tokens
    access_token = await create_access_token(
        data={"sub": user.email, "user_id": user.id, "role": user.role},
        db=db
    )
    refresh_token = await create_refresh_token(
        data={"sub": user.email, "user_id": user.id, "role": user.role},
        db=db
    )

    return access_token, refresh_token, user

# ============================================================

# ✅ REFRESH ACCESS TOKEN

# ============================================================

async def refresh_access_token(
refresh_token: str, db: AsyncSession
) -> tuple[str, str]: # Verify refresh token
payload = decode_token(refresh_token)
if not payload or payload.get("type") != "refresh":
raise HTTPException(
status_code=status.HTTP_401_UNAUTHORIZED,
detail="Invalid refresh token",
headers={"WWW-Authenticate": "Bearer"},
)

    user_email = payload.get("sub")
    if not user_email:
         raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid refresh token payload",
            headers={"WWW-Authenticate": "Bearer"},
        )

    # Check if token exists and is valid in DB (optional but recommended)
    result = await db.execute(select(Token).where(Token.token_string == refresh_token))
    stored_token = result.scalars().first()

    if not stored_token or stored_token.is_revoked:
         raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Refresh token revoked or invalid",
            headers={"WWW-Authenticate": "Bearer"},
        )

    # Get user to ensure they still exist/active
    result = await db.execute(select(User).where(User.email == user_email))
    user = result.scalars().first()

    if not user or not user.is_active:
         raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="User inactive or not found",
            headers={"WWW-Authenticate": "Bearer"},
        )

    # Create new access token
    access_token = await create_access_token(
        data={"sub": user.email, "user_id": user.id, "role": user.role},
        db=db
    )

    # Optionally rotate refresh token
    new_refresh_token = await create_refresh_token(
        data={"sub": user.email, "user_id": user.id, "role": user.role},
        db=db
    )

    # Revoke old refresh token
    stored_token.is_revoked = True
    await db.commit()

    return access_token, new_refresh_token

# ============================================================

# ✅ LOGOUT USER (Global Revocation)

# ============================================================

async def logout_user(user: User, db: AsyncSession) -> None: # Revoke all tokens (access and refresh) for this user
from sqlalchemy import update

    await db.execute(
        update(Token)
        .where(Token.user_id == user.id)
        .values(is_revoked=True)
    )
    await db.commit()

# ============================================================

# ✅ CREATE PASSWORD RESET LINK WITH THE RESET TOKEN ON IT

# ============================================================

async def create_password_reset_link(
email: str, db: AsyncSession
) -> tuple[str, str | None]:
result = await db.execute(select(User).where(User.email == email))
user = result.scalars().first()

    if not user:
        # Don't reveal user existence
        return "If your email is registered, you will receive a password reset link.", None

    reset_token = generate_password_reset_token()


    # Store token in DB

    expires_at = utcnow() + timedelta(minutes=15) # 15 minutes expiry

    new_reset_token = PasswordResetToken(
        user_id=user.id,
        token=reset_token,
        expires_at=expires_at
    )
    db.add(new_reset_token)
    await db.commit()

    reset_link = f"{settings.FRONTEND_URL}/reset-password?token={reset_token}"
    try:
        send_reset_password_link_with_token_in_email(email, reset_link, user.first_name)
    except Exception as e:
        print(f"Error sending reset password email: {e}")

    return "Password reset link sent to your email", reset_token

# ============================================================

# ✅ CHANGE/UPDATE PASSWORD

# ============================================================

async def update_password(
user: User, current_password: str, new_password: str, db: AsyncSession
) -> None:
if not verify_password(current_password, user.hashed_password):
raise HTTPException(
status_code=status.HTTP_400_BAD_REQUEST,
detail="Incorrect current password"
)

    user.hashed_password = get_password_hash(new_password)
    db.add(user)
    await db.commit()

# ============================================================

# ✅ VERIFY EMAIL

# ============================================================

async def verify_email_with_code(
user: User, verification_code: str, db: AsyncSession
) -> None:
if user.is_verified:
return # Already verified

    if user.verification_code != verification_code:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Invalid verification code"
        )

    user.is_verified = True
    # Clear code after verification
    user.verification_code = None
    db.add(user)
    await db.commit()

# ============================================================

# ✅ RESET PASSWORD WITH TOKEN

# ============================================================

async def reset_password_with_token(token: str, new_password: str, db: AsyncSession) -> None: # 1. Verify token exists and is valid

    result = await db.execute(select(PasswordResetToken).where(PasswordResetToken.token == token))
    stored_token = result.scalars().first()

    if not stored_token:
        raise HTTPException(status_code=400, detail="Invalid reset token")

    if stored_token.is_used:
        raise HTTPException(status_code=400, detail="Token already used")

    if stored_token.expires_at < utcnow():
        raise HTTPException(status_code=400, detail="Token expired")

    # 2. Get User
    result = await db.execute(select(User).where(User.id == stored_token.user_id))
    user = result.scalars().first()

    if not user:
        raise HTTPException(status_code=404, detail="User not found")

    # 3. Update Password
    user.hashed_password = get_password_hash(new_password)
    stored_token.is_used = True # Mark token as used

    db.add(user)
    db.add(stored_token)
    await db.commit()

</file>

<file path="app/users/security.py">
# app/users/security.py

from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, and\_
from passlib.context import CryptContext
from datetime import datetime, timedelta
from typing import Optional, Dict, Any
from config.appconfig import settings
from app.helpers.time import utcnow
from jose import JWTError, jwt
import secrets
import random

# Password hashing context

pwd_context = CryptContext(schemes=["argon2"], deprecated="auto")

# ============================================================

# ✅ Verify Password

# ============================================================

def verify_password(plain_password: str, hashed_password: str) -> bool:
"""Verify a password against its hash."""
return pwd_context.verify(plain_password, hashed_password)

# ============================================================

# ✅ Get Password Hash

# ============================================================

def get_password_hash(password: str) -> str:
"""Hash a password."""
return pwd_context.hash(password)

# ============================================================

# ✅ Create Access Token

# ============================================================

async def create_access_token(
data: Dict[str, Any],
db: AsyncSession,
expires_delta: Optional[timedelta] = None
) -> str:
"""Create a JWT access token and store it as active."""
to_encode = data.copy()

    if expires_delta:
        expire = utcnow() + expires_delta
    else:
        expire = utcnow() + timedelta(minutes=settings.ACCESS_TOKEN_EXPIRY)

    to_encode.update({"exp": expire, "type": "access"})
    encoded_jwt = jwt.encode(to_encode, settings.SECRET_KEY, algorithm=settings.ALGORITHM)

    # Store as active token
    from app.users.auth_token_model.token_model import Token
    token = Token(
        token_string=encoded_jwt,
        token_type="access",
        user_id=data.get("user_id"),
        expires_at=expire
    )
    db.add(token)
    await db.commit()

    return encoded_jwt

# ============================================================

# ✅ Create Refresh Token

# ============================================================

async def create_refresh_token(
data: Dict[str, Any],
db: AsyncSession,
expires_delta: Optional[timedelta] = None
) -> str:
"""Create a JWT refresh token and store it as active."""
to_encode = data.copy()

    if expires_delta:
        expire = utcnow() + expires_delta
    else:
        expire = utcnow() + timedelta(days=settings.REFRESH_TOKEN_EXPIRY)

    to_encode.update({"exp": expire, "type": "refresh"})
    encoded_jwt = jwt.encode(to_encode, settings.SECRET_KEY, algorithm=settings.ALGORITHM)

    # Store token
    from app.users.auth_token_model.token_model import Token
    token = Token(
        token_string=encoded_jwt,
        token_type="refresh",
        user_id=data.get("user_id"),
        expires_at=expire
    )
    db.add(token)
    await db.commit()


    return encoded_jwt

# ============================================================

# ✅ Decode Token

# ============================================================

def decode_token(token: str) -> Optional[Dict[str, Any]]:
"""Decode and verify a JWT token."""
try:
payload = jwt.decode(
token, settings.SECRET_KEY, algorithms=[settings.ALGORITHM]
)
return payload
except JWTError:
return None

# ============================================================

# ✅ Generate Password Reset Token

# ============================================================

def generate_password_reset_token() -> str:
"""Generate a secure random token for password reset."""
return secrets.token_urlsafe(32)

# ============================================================

# ✅ Get Token Expiry

# ============================================================

def get_token_expiry(token_type: str = "access") -> datetime:
"""Get expiry datetime for a token."""
if token_type == "refresh":
return utcnow() + timedelta(days=settings.REFRESH_TOKEN_EXPIRY)
return utcnow() + timedelta(minutes=settings.ACCESS_TOKEN_EXPIRY)

# ============================================================

# ✅ Generate Verification Code

# ============================================================

def generate_verification_code():
"""Generate a random 6-digit verification code."""
return str(random.randint(100000, 999999))
</file>

<file path="app/visionsystem/__init__.py">

</file>

<file path="app/visionsystem/biomedclip_client.py">
# app/visionsystem/biomedclip_client.py
"""
BiomedCLIP Client - FIXED (uses open_clip which is already installed)
open-clip-torch>=3.2.0 is in pyproject.toml - this is the REAL implementation.

BiomedCLIP is an open_clip model, NOT a standard transformers model.
Previous code used AutoModel.from_pretrained() which fails because there is no
pytorch_model.bin - weights are in open_clip format only.

Correct loading: open_clip.create_model_and_transforms('hf-hub:microsoft/BiomedCLIP-...')
"""
import logging
from PIL import Image
import torch

logger = logging.getLogger(**name**)

DENTAL_PATHOLOGY_CANDIDATES = [
"periapical abscess",
"dental caries",
"periodontal bone loss",
"normal tooth",
"root canal treatment",
"dental restoration",
"impacted tooth",
"tooth fracture",
]

class BiomedCLIPClient:
"""BiomedCLIP pathology classifier - uses open_clip (correct method for this model)"""

    _instance = None
    _model = None
    _preprocess = None
    _tokenizer = None
    _load_attempted = False

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def _lazy_load_model(self):
        """Load BiomedCLIP via open_clip - the ONLY correct way to load this model."""
        if self._load_attempted:
            return
        self._load_attempted = True

        logger.info("🔄 Loading BiomedCLIP via open_clip (correct method)...")
        try:
            import open_clip

            self._model, _, self._preprocess = open_clip.create_model_and_transforms(
                "hf-hub:microsoft/BiomedCLIP-PubMedBERT_256-vit_base_patch16_224"
            )
            self._tokenizer = open_clip.get_tokenizer(
                "hf-hub:microsoft/BiomedCLIP-PubMedBERT_256-vit_base_patch16_224"
            )
            self._model.eval()
            logger.info("✅ BiomedCLIP loaded successfully via open_clip")

        except Exception as e:
            logger.error(f"❌ Failed to load BiomedCLIP: {e}")
            self._model = None
            raise

    def classify_pathology(self, image: Image.Image) -> dict:
        """
        Classify dental pathology using BiomedCLIP CLIP-style similarity scoring.
        Returns prediction, confidence, and all category scores.
        """
        self._lazy_load_model()

        if self._model is None:
            raise RuntimeError("BiomedCLIP model failed to load")

        logger.info("🔬 Running BiomedCLIP dental pathology classification...")

        try:
            # Preprocess image
            image_tensor = self._preprocess(image).unsqueeze(0)

            # Tokenize text labels with radiological context prefix
            text_inputs = [f"an X-ray showing {label}" for label in DENTAL_PATHOLOGY_CANDIDATES]
            text_tokens = self._tokenizer(text_inputs)

            with torch.no_grad():
                image_features = self._model.encode_image(image_tensor)
                text_features = self._model.encode_text(text_tokens)

                # Normalize embeddings
                image_features = image_features / image_features.norm(dim=-1, keepdim=True)
                text_features = text_features / text_features.norm(dim=-1, keepdim=True)

                # Cosine similarity scaled by 100
                similarity = (100.0 * image_features @ text_features.T)[0]

                # Softmax probabilities
                probs = torch.nn.functional.softmax(similarity, dim=0)

            top_idx = probs.argmax().item()
            top_prob = probs[top_idx].item()

            all_scores = {
                DENTAL_PATHOLOGY_CANDIDATES[i]: round(probs[i].item(), 4)
                for i in range(len(DENTAL_PATHOLOGY_CANDIDATES))
            }

            logger.info(f"✅ BiomedCLIP: {DENTAL_PATHOLOGY_CANDIDATES[top_idx]} ({top_prob:.1%})")

            return {
                "prediction": DENTAL_PATHOLOGY_CANDIDATES[top_idx],
                "confidence": top_prob,
                "all_scores": all_scores,
            }

        except Exception as e:
            logger.error(f"❌ BiomedCLIP classification failed: {e}")
            raise

    def analyze_image(self, image: Image.Image, prompt: str = None) -> dict:
        """Backward compatibility - returns formatted text of classification."""
        result = self.classify_pathology(image)

        lines = [
            "BiomedCLIP Dental Pathology Classification",
            "=" * 45,
            f"Primary Finding: {result['prediction'].upper()}",
            f"Confidence:      {result['confidence']:.1%}",
            "",
            "All Category Scores:",
        ]
        for pathology, score in sorted(
            result["all_scores"].items(), key=lambda x: x[1], reverse=True
        ):
            bar = "█" * int(score * 30)
            lines.append(f"  {pathology:<28} {score:.1%}  {bar}")

        return {
            "text": "\n".join(lines),
            "input_tokens": None,
            "output_tokens": None,
        }

# Global instance

biomedclip_client = BiomedCLIPClient()
</file>

<file path="app/visionsystem/claude_vision_client.py">
# app/visionsystem/claude_vision_client.py
"""
Claude Vision Client - Optimized for Dental Radiographs
"""
from PIL import Image
import base64
from io import BytesIO
import os
import logging
from anthropic import Anthropic
from config.visionconfig import vision_settings

logger = logging.getLogger(**name**)

# Dental radiograph analysis prompt for Claude

DENTAL_XRAY_PROMPT = """You are analyzing a DENTAL PERIAPICAL RADIOGRAPH.

                TASK: Identify all pathologies and anomalies in this dental X-ray image.

                IMAGE CONTEXT:
                - This is a medical diagnostic image (periapical radiograph)
                - Shows tooth/teeth with roots and surrounding bone
                - Purpose: Detect abnormalities requiring clinical intervention

                REQUIRED ANALYSIS:

                1. IMAGE TYPE & QUALITY:
                - Confirm: periapical radiograph
                - Diagnostic quality assessment

                2. VISIBLE ANATOMY:
                - Tooth/teeth number (if identifiable)
                - Crown, root(s), pulp chamber
                - Alveolar bone, PDL, lamina dura

                3. PATHOLOGY IDENTIFICATION:

                DENTAL CARIES:
                - Dark areas in crown = decay
                - Specify: tooth, surface, depth

                PERIAPICAL PATHOLOGY:
                - Dark zone at root tip = abscess/cyst
                - Measure size, note characteristics

                BONE LOSS:
                - Reduced bone height around root
                - Type: horizontal/vertical
                - Severity: mild/moderate/severe

                RESTORATIONS:
                - Bright white = metal fillings
                - Quality assessment

                ROOT PATHOLOGY:
                - Resorption, fractures, calcifications

                4. RADIOGRAPHIC INTERPRETATION:
                - Radiolucent (dark) = less dense tissue, decay, infection
                - Radiopaque (bright) = dense tissue, metal, bone

                5. CLINICAL ASSESSMENT:
                - Primary diagnosis
                - Severity and urgency
                - Treatment implications

                BE SPECIFIC: Use tooth numbers, surface names, measurements when possible.
                USE DENTAL TERMINOLOGY: Proper clinical language.
                STATE CLEARLY: If no pathology detected, say so explicitly.

                Provide your detailed analysis:"""

class ClaudeVisionClient:
"""Claude Vision client for dental radiograph analysis."""

    _instance = None
    _client = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def _get_client(self):
        if self._client is None:
            # Try vision settings first, then fall back to environment variables
            api_key = vision_settings.ANTHROPIC_API_KEY or os.getenv("ANTHROPIC_API_KEY") or os.getenv("CLAUDE_API_KEY")
            if not api_key:
                raise ValueError("Anthropic API key not found. Please set ANTHROPIC_API_KEY in .env")
            self._client = Anthropic(api_key=api_key)
        return self._client

    def _get_model_name(self):
        """Get Claude model from vision settings."""
        return vision_settings.CLAUDE_VISION_MODEL

    def _encode_image(self, image: Image.Image) -> str:
        """Convert PIL Image to base64."""
        buffer = BytesIO()
        image.save(buffer, format="PNG")
        return base64.b64encode(buffer.getvalue()).decode()

    def analyze_image(self, image: Image.Image, prompt: str = None) -> dict:
        """Analyze dental radiograph using Claude Vision."""
        client = self._get_client()
        b64_image = self._encode_image(image)

        analysis_prompt = prompt or DENTAL_XRAY_PROMPT

        logger.info("Analyzing dental radiograph with Claude Vision...")

        try:
            response = client.messages.create(
                model=self._get_model_name(),
                max_tokens=vision_settings.VISION_MAX_TOKENS,
                temperature=vision_settings.VISION_TEMPERATURE,
                messages=[
                    {
                        "role": "user",
                        "content": [
                            {
                                "type": "image",
                                "source": {
                                    "type": "base64",
                                    "media_type": "image/png",
                                    "data": b64_image
                                }
                            },
                            {
                                "type": "text",
                                "text": analysis_prompt
                            }
                        ]
                    }
                ]
            )

            result = response.content[0].text
            usage = response.usage
            logger.info(f"Claude analysis complete: {len(result)} characters")
            if usage:
                logger.info(f"   Token usage: {usage.input_tokens} input, {usage.output_tokens} output")
                return {
                    "text": result,
                    "input_tokens": usage.input_tokens,
                    "output_tokens": usage.output_tokens,
                }
            else:
                return {"text": result, "input_tokens": None, "output_tokens": len(result.split())}

        except Exception as e:
            logger.error(f"Claude Vision analysis failed: {e}")
            raise

    def analyze_clinical_image(self, image: Image.Image) -> dict:
        """Comprehensive dental radiograph analysis."""
        # Single comprehensive analysis (Claude is good with complex prompts)
        analysis_result = self.analyze_image(image, DENTAL_XRAY_PROMPT)
        detailed = analysis_result["text"]
        input_tokens = analysis_result.get("input_tokens", 0)
        output_tokens = analysis_result.get("output_tokens", 0)

        # Optional pathology-focused summary
        pathology = None
        if vision_settings.DUAL_PROMPT_ANALYSIS:
            try:
                pathology_prompt = """List only the pathological findings from this dental X-ray:
                    - Caries: present/absent, details if present
                    - Periapical lesions: present/absent, details if present
                    - Bone loss: present/absent, details if present
                    - Other abnormalities: present/absent, details if present

                    Format as concise bullet points."""
                pathology_result = self.analyze_image(image, pathology_prompt)
                pathology = pathology_result["text"]
                input_tokens += pathology_result.get("input_tokens", 0)
                output_tokens += pathology_result.get("output_tokens", 0)
            except Exception as e:
                logger.error(f"Pathology summary failed: {e}")

        return {
            "detailed_description": detailed,
            "region_findings": pathology or "See detailed description",
            "model": self._get_model_name(),
            "input_tokens": input_tokens,
            "output_tokens": output_tokens
        }

# Global instance

claude_vision_client = ClaudeVisionClient()
</file>

<file path="app/visionsystem/florence_client.py">
# app/visionsystem/florence_client.py
import os

os.environ["TRANSFORMERS_ATTN_IMPLEMENTATION"] = "eager"

import logging

import torch
from PIL import Image
from transformers import AutoModelForCausalLM, AutoProcessor, GenerationConfig

from config.visionconfig import vision_settings

logger = logging.getLogger(**name**)

class FlorenceClient:
"""Production-grade Florence-2 client for clinical image analysis."""

    _instance = None
    _model = None
    _processor = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def _get_device(self):
        if torch.cuda.is_available():
            return "cuda"
        return "cpu"

    def load_model(self):
        """Load Florence-2 with clinical settings."""
        if self._model is not None:
            return self._processor, self._model

        logger.info(f"Loading Florence-2: {vision_settings.FLORENCE_MODEL_NAME}")
        device = self._get_device()

        self._processor = AutoProcessor.from_pretrained(
            vision_settings.FLORENCE_MODEL_NAME, trust_remote_code=True
        )

        self._model = AutoModelForCausalLM.from_pretrained(
            vision_settings.FLORENCE_MODEL_NAME,
            trust_remote_code=True,
            attn_implementation="eager",
            dtype=torch.float32,
            device_map=None,
        )

        self._model = self._model.to(device)
        self._model.eval()

        logger.info(f"✅Florence-2 ready on {device}")
        return self._processor, self._model

    def analyze_image(self, image: Image.Image, prompt: str = None) -> dict:
        """Clinical image analysis."""
        processor, model = self.load_model()
        device = next(model.parameters()).device

        task_prompt = prompt or "<MORE_DETAILED_CAPTION>"

        if image.mode != "RGB":
            image = image.convert("RGB")

        # Process inputs
        inputs = processor(text=task_prompt, images=image, return_tensors="pt")

        # Move to device
        inputs = {k: v.to(device) if isinstance(v, torch.Tensor) else v for k, v in inputs.items()}

        # Create generation config to override model defaults
        gen_config = GenerationConfig(
            max_new_tokens=1024,
            num_beams=1,
            do_sample=False,
            eos_token_id=processor.tokenizer.eos_token_id,
            pad_token_id=processor.tokenizer.pad_token_id,
            use_cache=False,  # Disable KV cache to avoid beam search bug
        )

        # Generate
        with torch.no_grad():
            generated_ids = model.generate(
                input_ids=inputs["input_ids"],
                pixel_values=inputs["pixel_values"],
                generation_config=gen_config,
            )

        # Decode
        generated_text = processor.batch_decode(generated_ids, skip_special_tokens=False)[0]

        # Parse
        try:
            parsed = processor.post_process_generation(
                generated_text, task=task_prompt, image_size=(image.width, image.height)
            )

            if isinstance(parsed, dict) and task_prompt in parsed:
                result = parsed[task_prompt]
                logger.info(f"Analysis complete: {len(str(result))} chars")
                return {
                    "text": str(result),
                    "input_tokens": None,
                    "output_tokens": None,
                }
            else:
                return {
                    "text": str(parsed),
                    "input_tokens": None,
                    "output_tokens": None,
                }

        except Exception as e:
            logger.warning(f"Post-processing failed: {e}")
            # Fallback: clean up raw text
            cleaned = generated_text.replace(task_prompt, "").strip()
            cleaned = cleaned.replace("</s>", "").replace("<s>", "").strip()
            return {
                "text": cleaned,
                "input_tokens": None,
                "output_tokens": None,
            }

    def analyze_clinical_image(self, image: Image.Image) -> dict:
        """Comprehensive clinical analysis with multiple prompts."""
        detailed_result = self.analyze_image(image, "<MORE_DETAILED_CAPTION>")
        region_result = self.analyze_image(image, "<DENSE_REGION_CAPTION>")
        return {
            "detailed_description": detailed_result["text"],
            "region_findings": region_result["text"],
            "model": vision_settings.FLORENCE_MODEL_NAME,
        }

florence_client = FlorenceClient()
</file>

<file path="app/visionsystem/gpt4_client.py">
# app/visionsystem/gpt4_client.py
"""
GPT-4 Vision Client - Optimized for Dental Radiographs
FIXED: Using gpt-4o (current model, not deprecated)
"""
import os
import base64
import logging
from io import BytesIO

from PIL import Image
from openai import OpenAI

from config.visionconfig import vision_settings

logger = logging.getLogger(**name**)

# Dental radiograph prompt for GPT-4V

DENTAL_XRAY_PROMPT = """Analyze this DENTAL PERIAPICAL RADIOGRAPH (X-ray image).

            YOUR TASK: Identify all pathologies and clinical abnormalities present in this dental X-ray.

            IMAGE INFORMATION:
            - Type: Periapical radiograph (shows tooth root and surrounding bone)
            - Purpose: Diagnostic evaluation for dental pathology

            SYSTEMATIC ANALYSIS REQUIRED:

            1. IMAGE VERIFICATION:
            - Confirm this is a periapical dental X-ray
            - Assess diagnostic quality

            2. ANATOMICAL STRUCTURES:
            - Identify visible tooth/teeth (number if possible)
            - Crown, root(s), pulp chamber, root canals
            - Alveolar bone, periodontal ligament, lamina dura

            3. PATHOLOGY DETECTION (PRIMARY FOCUS):

            A. DENTAL CARIES:
                - Radiolucent (dark) areas in tooth crown
                - Location: mesial, distal, occlusal, buccal, lingual
                - Depth: enamel, dentin, approaching pulp

            B. PERIAPICAL PATHOLOGY:
                - Radiolucent area at root apex = infection/abscess/cyst
                - Size estimation (mm)
                - Characteristics: well-defined, diffuse
                - PDL widening

            C. BONE LOSS:
                - Horizontal bone loss: reduced bone height
                - Vertical bone loss: angular defects
                - Distance from CEJ to alveolar crest

            D. PULPAL STATUS:
                - Pulp stones or calcifications
                - Chamber obliteration
                - Previous root canal treatment evident

            E. ROOT ABNORMALITIES:
                - Resorption (internal/external)
                - Fractures
                - Morphological variations

            F. RESTORATIONS:
                - Radiopaque (bright) materials = fillings, crowns
                - Quality: overhangs, recurrent decay
                - Material type if identifiable

            4. RADIOGRAPHIC TERMINOLOGY:
            - RADIOLUCENT (dark) = Lower density: caries, infection, bone loss
            - RADIOPAQUE (bright) = Higher density: enamel, restorations, bone

            5. CLINICAL SIGNIFICANCE:
            - Primary diagnosis
            - Severity level: Mild / Moderate / Severe
            - Clinical urgency: Routine / Prompt / Urgent
            - Treatment implications

            RESPONSE FORMAT:
            - Be SPECIFIC with tooth numbers and surfaces
            - Use PROPER DENTAL TERMINOLOGY
            - Provide MEASUREMENTS when possible
            - State clearly if NO PATHOLOGY DETECTED

            Provide your detailed clinical analysis:"""

class GPT4VisionClient:
"""GPT-4 Vision client for dental radiograph analysis."""

    _instance = None
    _client = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def _get_client(self):
        if self._client is None:
            api_key = os.getenv("OPENAI_API_KEY")
            if not api_key:
                raise ValueError("OPENAI_API_KEY not found in environment")
            self._client = OpenAI(api_key=api_key)
        return self._client

    def _get_model_name(self):
        """Get GPT model - FIXED to use gpt-4o instead of deprecated gpt-4-vision-preview."""
        return "gpt-4o"  # Current model with vision capabilities

    def _encode_image(self, image: Image.Image) -> str:
        """Convert PIL Image to base64."""
        buffer = BytesIO()
        image.save(buffer, format="PNG")
        return base64.b64encode(buffer.getvalue()).decode()

    def analyze_image(self, image: Image.Image, prompt: str = None) -> dict:
        """Analyze dental radiograph using GPT-4o."""
        client = self._get_client()
        b64_image = self._encode_image(image)
        model_name = self._get_model_name()

        analysis_prompt = prompt or DENTAL_XRAY_PROMPT

        logger.info(f"Analyzing dental radiograph with {model_name}...")

        try:
            response = client.chat.completions.create(
                model=model_name,
                messages=[
                    {
                        "role": "user",
                        "content": [
                            {"type": "text", "text": analysis_prompt},
                            {
                                "type": "image_url",
                                "image_url": {
                                    "url": f"data:image/png;base64,{b64_image}",
                                    "detail": "high",  # High detail for clinical accuracy
                                },
                            },
                        ],
                    }
                ],
                max_tokens=vision_settings.VISION_MAX_TOKENS,
                temperature=vision_settings.VISION_TEMPERATURE,
            )

            result = response.choices[0].message.content
            usage = response.usage
            logger.info(f"GPT-4o analysis complete: {len(result)} characters")
            if usage:
                logger.info(f"   Token usage: {usage.prompt_tokens} prompt, {usage.completion_tokens} completion")
                return {
                    "text": result,
                    "input_tokens": usage.prompt_tokens,
                    "output_tokens": usage.completion_tokens,
                }
            else:
                # Fallback if usage is not reported for some reason
                return {"text": result, "input_tokens": None, "output_tokens": len(result.split())}

        except Exception as e:
            logger.error(f"GPT-4o analysis failed: {e}")
            raise

    def analyze_clinical_image(self, image: Image.Image) -> dict:
        """Comprehensive dental radiograph analysis."""
        # Main detailed analysis
        analysis_result = self.analyze_image(image, DENTAL_XRAY_PROMPT)
        detailed = analysis_result["text"]
        input_tokens = analysis_result.get("input_tokens", 0)
        output_tokens = analysis_result.get("output_tokens", 0)

        # Optional pathology summary
        pathology = None
        if vision_settings.DUAL_PROMPT_ANALYSIS:
            try:
                pathology_prompt = """Provide a concise pathology summary from this dental X-ray:

                        PATHOLOGY CHECKLIST:
                        1. Caries: Yes/No (if yes: location, depth)
                        2. Periapical lesion: Yes/No (if yes: size, tooth affected)
                        3. Bone loss: Yes/No (if yes: type, severity)
                        4. Restorations: Yes/No (if yes: type, condition)
                        5. Other findings: List any other abnormalities

                        Format as structured bullet points."""
                pathology_result = self.analyze_image(image, pathology_prompt)
                pathology = pathology_result["text"]
                input_tokens += pathology_result.get("input_tokens", 0)
                output_tokens += pathology_result.get("output_tokens", 0)
            except Exception as e:
                logger.error(f"Pathology summary failed: {e}")

        return {
            "detailed_description": detailed,
            "region_findings": pathology or "See detailed description",
            "model": self._get_model_name(),
            "input_tokens": input_tokens,
            "output_tokens": output_tokens
        }

# Global instance

gpt4v_client = GPT4VisionClient()
</file>

<file path="app/visionsystem/groq_vision_client.py">
# app/visionsystem/groq_vision_client.py
"""
Groq Vision Client - Ultra-fast vision inference using LPU
Supports: Llama 3.2 90B Vision, 11B Vision
Speed: 5-20x faster than GPU-based providers
Cost: ~$0.001 per image (100x cheaper than GPT-4V)
"""
import logging
import base64
from io import BytesIO
from PIL import Image
from typing import Dict

logger = logging.getLogger(**name**)

class GroqVisionClient:
"""Groq LPU-accelerated vision model client"""

    _instance = None
    _client = None
    _load_attempted = False

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def _lazy_load_client(self):
        """Initialize Groq client on first use"""
        if self._load_attempted:
            return
        self._load_attempted = True

        try:
            from config.visionconfig import vision_settings
            import os

            api_key = vision_settings.GROQ_API_KEY or os.getenv("GROQ_API_KEY")
            if not api_key:
                logger.warning("⚠️  GROQ_API_KEY not set - Groq vision will fail")
                return

            from groq import Groq
            self._client = Groq(api_key=api_key)
            logger.info(f"✅ Groq client initialized with model: {vision_settings.GROQ_VISION_MODEL}")

        except ImportError:
            logger.error("❌ Groq SDK not installed. Run: pip install groq")
        except Exception as e:
            logger.error(f"❌ Groq client initialization failed: {e}")

    def analyze_image(self, image: Image.Image, prompt: str) -> dict:
        """
        Analyze dental radiograph using Groq vision model.

        Args:
            image: PIL Image object
            prompt: Text prompt for analysis

        Returns:
            Dict with 'text', 'input_tokens', 'output_tokens'
        """
        from config.visionconfig import vision_settings

        self._lazy_load_client()

        if not self._client:
            raise RuntimeError("Groq client not initialized. Check API key.")

        logger.info(f"🚀 Analyzing with Groq {vision_settings.GROQ_VISION_MODEL}...")

        # Convert PIL Image to base64
        buffered = BytesIO()
        image.save(buffered, format="PNG")
        img_base64 = base64.b64encode(buffered.getvalue()).decode("utf-8")

        try:
            # Groq uses OpenAI-compatible API
            response = self._client.chat.completions.create(
                model=vision_settings.GROQ_VISION_MODEL,
                messages=[
                    {
                        "role": "user",
                        "content": [
                            {
                                "type": "text",
                                "text": prompt
                            },
                            {
                                "type": "image_url",
                                "image_url": {
                                    "url": f"data:image/png;base64,{img_base64}"
                                }
                            }
                        ]
                    }
                ],
                temperature=vision_settings.VISION_TEMPERATURE,
                max_tokens=vision_settings.VISION_MAX_TOKENS,
            )

            result = response.choices[0].message.content
            usage = response.usage
            logger.info(f"✅ Groq analysis complete: {len(result)} chars")
            if usage:
                logger.info(f"   Token usage: {usage.prompt_tokens} prompt, {usage.completion_tokens} completion")
                return {
                    "text": result,
                    "input_tokens": usage.prompt_tokens,
                    "output_tokens": usage.completion_tokens,
                }
            else:
                return {"text": result, "input_tokens": None, "output_tokens": len(result.split())}

        except Exception as e:
            error_msg = str(e)
            logger.error(f"❌ Groq analysis failed: {error_msg}")

            # Handle specific errors
            if "quota" in error_msg.lower() or "limit" in error_msg.lower():
                raise RuntimeError(f"Groq API quota exceeded: {error_msg}")
            elif "auth" in error_msg.lower() or "key" in error_msg.lower():
                raise RuntimeError(f"Groq API authentication failed: {error_msg}")
            else:
                raise RuntimeError(f"Groq analysis error: {error_msg}")

# Global instance

groq_vision_client = GroqVisionClient()
</file>

<file path="app/visionsystem/image_processor.py">
# app/visionsystem/image_processor.py (Updated)
from PIL import Image
import io
from typing import Union
from config.visionconfig import vision_settings
import logging

logger = logging.getLogger(**name**)

class ImageProcessor:
"""Handle image preprocessing for clinical analysis."""

    @staticmethod
    def preprocess_image(image_bytes: bytes, max_size: int = None) -> Image.Image:
        """
        Preprocess uploaded image for vision model.
        Converts to RGB, resizes if too large, and ensures proper format.
        """
        max_size = max_size or vision_settings.MAX_IMAGE_SIZE

        try:
            # Loading image from bytes
            image = Image.open(io.BytesIO(image_bytes))
            logger.info(f"Loaded image: {image.format}, mode={image.mode}, size={image.size}")

            # Converting to RGB if necessary
            if image.mode != 'RGB':
                logger.info(f"Converting image from {image.mode} to RGB")
                image = image.convert('RGB')

            # Resizing if too large (maintain aspect ratio)
            if max(image.size) > max_size:
                ratio = max_size / max(image.size)
                new_size = (int(image.size[0] * ratio), int(image.size[1] * ratio))
                image = image.resize(new_size, Image.Resampling.LANCZOS)
                logger.info(f"Resized image to {new_size}")

            # Ensuring image is in proper format for transformers
            # Saving to buffer and reload to ensure clean format
            buffer = io.BytesIO()
            image.save(buffer, format='PNG')
            buffer.seek(0)
            image = Image.open(buffer)

            return image

        except Exception as e:
            logger.error(f"Image preprocessing failed: {e}")
            raise ValueError(f"Invalid image file: {e}")

    @staticmethod
    def validate_image(content_type: str, size_bytes: int) -> bool:
        """Validate uploaded image meets requirements."""
        if content_type not in vision_settings.SUPPORTED_FORMATS:
            raise ValueError(f"Unsupported format: {content_type}. Use: {vision_settings.SUPPORTED_FORMATS}")

        # 10MB limit
        if size_bytes > 10 * 1024 * 1024:
            raise ValueError("Image too large. Max 10MB.")

        return True

</file>

<file path="app/visionsystem/llava_client.py">
# app/visionsystem/llava_client.py
"""
LLaVA Vision Client - FIXED with proper instance export
"""
import base64
from io import BytesIO
from PIL import Image, ImageEnhance
import ollama
import logging
from config.visionconfig import vision_settings

logger = logging.getLogger(**name**)

# Dental radiograph analysis prompt (unchanged from your original)

DENTAL_XRAY_PROMPT = """You are analyzing a DENTAL PERIAPICAL RADIOGRAPH (X-ray).
PURPOSE: Identify pathologies, anomalies, and clinical findings in this dental X-ray image.

CRITICAL CONTEXT:

- This is a MEDICAL IMAGE, specifically a dental X-ray
- You are looking at tooth/teeth and surrounding bone structures
- Your job is to detect ABNORMALITIES and PATHOLOGIES

SYSTEMATIC ANALYSIS REQUIRED:

1. IMAGE TYPE CONFIRMATION
2. ANATOMICAL STRUCTURES
3. PATHOLOGY DETECTION (MOST IMPORTANT)
4. RADIOGRAPHIC TERMINOLOGY
5. CLINICAL SIGNIFICANCE

OUTPUT REQUIREMENTS:
✅ Be SPECIFIC
✅ Use DENTAL TERMINOLOGY
✅ State SEVERITY levels
✅ If NO pathology found, state clearly

Begin your analysis:"""

PATHOLOGY_CHECKLIST_PROMPT = """FOCUSED PATHOLOGY DETECTION:
Answer YES or NO for each category, with specific details if YES:

1. CARIES (Cavities)
2. PERIAPICAL LESION/ABSCESS
3. BONE LOSS
4. ROOT CANAL TREATMENT
5. RESTORATIONS (Fillings/Crowns)
6. OTHER ABNORMALITIES

FORMAT: Structured bullet points, clinical terminology."""

class LlavaClient:
"""LLaVA client optimized for dental radiograph analysis."""

    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def _get_model_name(self):
        """Get LLaVA model name from settings."""
        return vision_settings.LLAVA_MODEL

    def _encode_image(self, image: Image.Image) -> str:
        """Convert PIL Image to base64 with optional X-ray enhancement."""
        if vision_settings.ENHANCE_CONTRAST:
            # Enhance contrast for better X-ray visibility
            enhancer = ImageEnhance.Contrast(image)
            image = enhancer.enhance(vision_settings.CONTRAST_FACTOR)

            # Enhance brightness
            brightness_enhancer = ImageEnhance.Brightness(image)
            image = brightness_enhancer.enhance(vision_settings.BRIGHTNESS_FACTOR)

        buffer = BytesIO()
        image.save(buffer, format="PNG")
        return base64.b64encode(buffer.getvalue()).decode()

    def analyze_image(self, image: Image.Image, prompt: str = None) -> dict:
        """
        Analyze dental image using LLaVA.

        Args:
            image: PIL Image (dental radiograph)
            prompt: Custom prompt or None for default dental analysis

        Returns:
            Dict with 'text', 'input_tokens', 'output_tokens'
        """
        model_name = self._get_model_name()

        # Use dental-specific prompt if none provided
        analysis_prompt = prompt or DENTAL_XRAY_PROMPT

        # Convert image to base64
        b64_image = self._encode_image(image)

        logger.info(f"Analyzing dental radiograph with {model_name}...")

        try:
            # Call Ollama with clinical settings
            response = ollama.chat(
                model=model_name,
                messages=[
                    {
                        "role": "system",
                        "content": """You are an expert dental radiologist with 20+ years of experience in periapical radiograph interpretation.

You specialize in detecting dental pathologies including caries, periapical lesions, bone loss, and root pathology.
Your analyses are precise, use proper dental terminology, and focus on clinically significant findings."""
},
{
"role": "user",
"content": analysis_prompt,
"images": [b64_image]
}
],
options={
"temperature": vision_settings.VISION_TEMPERATURE,
"top_p": 0.9,
"top_k": 40,
"num_predict": vision_settings.VISION_MAX_TOKENS,
}
)

            result = response["message"]["content"]
            logger.info(f"LLaVA analysis complete: {len(result)} characters")

            # Extract token counts
            input_tokens = response.get("prompt_eval_count")
            output_tokens = response.get("eval_count")
            logger.info(f"   Token usage: {input_tokens} prompt, {output_tokens} completion")

            return {
                "text": result,
                "input_tokens": input_tokens,
                "output_tokens": output_tokens,
            }

        except Exception as e:
            logger.error(f"LLaVA analysis failed: {e}")
            raise

    def analyze_clinical_image(self, image: Image.Image) -> dict:
        """
        Comprehensive dental radiograph analysis with dual prompts.

        Returns:
            dict with detailed_description and region_findings (pathology summary)
        """
        input_tokens = 0
        output_tokens = 0

        # Main detailed analysis
        try:
            analysis_result = self.analyze_image(image, DENTAL_XRAY_PROMPT)
            detailed = analysis_result["text"]
            input_tokens += analysis_result.get("input_tokens", 0)
            output_tokens += analysis_result.get("output_tokens", 0)
        except Exception as e:
            logger.error(f"Detailed analysis failed: {e}")
            detailed = f"Analysis failed: {str(e)}"

        # Pathology-focused checklist (if dual prompt enabled)
        pathology = None
        if vision_settings.DUAL_PROMPT_ANALYSIS:
            try:
                pathology_result = self.analyze_image(image, PATHOLOGY_CHECKLIST_PROMPT)
                pathology = pathology_result["text"]
                input_tokens += pathology_result.get("input_tokens", 0)
                output_tokens += pathology_result.get("output_tokens", 0)
            except Exception as e:
                logger.error(f"Pathology checklist failed: {e}")
                pathology = "Pathology assessment unavailable"

        return {
            "detailed_description": detailed,
            "region_findings": pathology or "See detailed description",
            "model": self._get_model_name(),
            "input_tokens": input_tokens,
            "output_tokens": output_tokens
        }

# CRITICAL: Global instance export

llava_client = LlavaClient()
</file>

<file path="app/visionsystem/test_tooth_number_recognition.py">
# app/visionsystem/test_tooth_number_recognition.py

from PIL import Image
import ollama

models = [
"llava:7b",
"llava:13b",
"llama3.2-vision"
]

image_path = "test_xray.jpg"
prompt = "What teeth are visible in this X-ray? List tooth numbers only."

for model in models:
print(f"\nTesting {model}...")
response = ollama.chat(
model=model,
messages=[{
'role': 'user',
'content': prompt,
'images': [image_path]
}]
)
print(response['message']['content'])
</file>

<file path="app/__init__.py">

</file>

<file path="app/main.py">
# app/main.py
from dotenv import load_dotenv

load_dotenv()

import logging.config
from contextlib import asynccontextmanager

from fastapi import FastAPI

from config.appconfig import settings

# Apply logging configuration

logging.config.dictConfig(settings.LOGGING_CONFIG)

# Import routers

from app.cdss_engine.routes import router as cdss_router
from app.RAGsystem.routes import router as rag_router
from app.shared.cost_routes import router as cost_router
from app.system_services.system_routes import router as system_router
from app.users.auth_routers import router as auth_router
from app.visionsystem.routes import router as vision_router
from app.shared.evaluation_routes import router as evaluation_router

# Import configurations

from config.ragconfig import rag_settings
from config.reset_config_route import router as reset_config_route
from config.visionconfig import vision_settings

@asynccontextmanager
async def lifespan(app: FastAPI):
"""Application lifespan events.""" # Startup
print("\n")
print("\n===============================================================================")
print("===============================================================================")
print(f" 🚀 Starting CDSS Server")
print(
f" ✅ Vision Model: {vision_settings.VISION_MODEL_PROVIDER} - {vision_settings.current_vision_model}"
)
print(
f" ✅ RAG Embeddings Provider: {rag_settings.EMBEDDING_PROVIDER} - {rag_settings.current_embedding_model}"
)
print(f" ✅ Chunk Size: {rag_settings.CHUNK_SIZE}")
print(f" ✅ Chunk Overlap: {rag_settings.CHUNK_OVERLAP}")
print(f" ✅ LLM Provider: {rag_settings.LLM_PROVIDER} - {rag_settings.current_llm_model}")
print(f" ✅ Retrieval Top-K: {rag_settings.RETRIEVAL_K}")
print(f" ✅ Retrival Type: {rag_settings.RETRIEVER_TYPE}")
if rag_settings.RETRIEVER_TYPE == "similarity_score_threshold":
print(f" ✅ Similarity Threshold: {rag_settings.SIMILARITY_THRESHOLD}")
if rag_settings.RETRIEVER_TYPE == "mmr":
print(f" ✅ Fetch K: {rag_settings.FETCH_K}")
print(f" ✅ Lambda Multiplier: {rag_settings.LAMBDA_MULT}")
if rag_settings.EMBEDDING_PROVIDER == "huggingface":
print(f" ✅ Computing Device: {rag_settings.effective_device}")
print("===============================================================================")
print("===============================================================================\n")
yield # Shutdown
print("👋 Shutting down")

app = FastAPI(
title="Clinical Decision Support System",
description="Multimodal CDSS with Vision, RAG, and Clinical Decision Support",
version="1.0.0",
lifespan=lifespan,
)

# Include routers with prefixes

app.include_router(auth_router, prefix="/api/auth", tags=["Authentication"])
app.include_router(rag_router, prefix="/api/rag", tags=["Legacy RAG"])
app.include_router(vision_router, prefix="/api/vision", tags=["Vision Analysis"])
app.include_router(cdss_router, prefix="/api/cdss", tags=["Clinical Decision Support"])
app.include_router(system_router, prefix="/api/system", tags=["System Services"])
app.include_router(reset_config_route, prefix="/api/system")
app.include_router(cost_router, prefix="/api")
app.include_router(evaluation_router, prefix="/api")

if **name** == "**main**":
import uvicorn

    uvicorn.run("app.main:app", host="0.0.0.0", port=8000, reload=True)

</file>

<file path="config/__init__.py">

</file>

<file path="config/appconfig.py">
# app/core/config.py

from pydantic_settings import BaseSettings
from pydantic import Field, ConfigDict, model_validator
from typing import Optional

class AppConfig(BaseSettings): # General Settings
APP_NAME: str = Field(default="My FastAPI App", env="APP_NAME")
ENVIRONMENT: str = Field(default="development", env="ENVIRONMENT")
DEBUG: bool = Field(default=True, env="DEBUG")

    # Database Settings
    DB_USER: str = Field(..., env="DB_USER")
    DB_PASSWORD: str = Field(..., env="DB_PASSWORD")
    DB_HOST: str = Field(default="localhost", env="DB_HOST")
    DB_PORT: str = Field(default="5432", env="DB_PORT")
    DB_NAME: str = Field(..., env="DB_NAME")

    # JWT Settings
    SECRET_KEY: str = Field(..., env="SECRET_KEY")
    ALGORITHM: str = Field(default="HS256", env="ALGORITHM")
    ACCESS_TOKEN_EXPIRY: int = Field(default=30, env="ACCESS_TOKEN_EXPIRY")
    REFRESH_TOKEN_EXPIRY: int = Field(default=60, env="REFRESH_TOKEN_EXPIRY")

    # URLs
    BASE_URL: str = Field(..., env="BASE_URL")
    FRONTEND_URL: str = Field(..., env="FRONTEND_URL")

    # Email Settings
    RESEND_API_KEY: str = Field(..., env="RESEND_API_KEY")

    # Cookie Settings
    COOKIE_DOMAIN: Optional[str] = Field(default=None, env="COOKIE_DOMAIN")
    COOKIE_SECURE: bool = Field(default=False, env="COOKIE_SECURE")
    COOKIE_SAMESITE: str = Field(default="lax", env="COOKIE_SAMESITE")
    ACCESS_TOKEN_COOKIE_NAME: str = Field(default="access_token", env="ACCESS_TOKEN_COOKIE_NAME")
    REFRESH_TOKEN_COOKIE_NAME: str = Field(default="refresh_token", env="REFRESH_TOKEN_COOKIE_NAME")

    @model_validator(mode='after')
    def adjust_for_environment(self):
        """Automatically adjust settings based on ENVIRONMENT variable from .env file"""
        if  self.ENVIRONMENT == "production":
            # Production settings
            self.COOKIE_SECURE = True
            self.COOKIE_SAMESITE = "strict"
            self.ACCESS_TOKEN_COOKIE_NAME = "__Host-access_token"
            self.REFRESH_TOKEN_COOKIE_NAME = "__Host-refresh_token"
            self.COOKIE_DOMAIN = None

            # You can add more production overrides here
            # if self.ACCESS_TOKEN_EXPIRY > 30:
            #     self.ACCESS_TOKEN_EXPIRY = 15  # Force shorter tokens in prod

        return self

    @property
    def DB_URL(self) -> str:
        """Async URL for your FastAPI application"""
        return f"postgresql+asyncpg://{self.DB_USER}:{self.DB_PASSWORD}@{self.DB_HOST}:{self.DB_PORT}/{self.DB_NAME}"

    @property
    def DB_URL_SYNC(self) -> str:
        """Sync URL for Alembic migrations"""
        return f"postgresql://{self.DB_USER}:{self.DB_PASSWORD}@{self.DB_HOST}:{self.DB_PORT}/{self.DB_NAME}"

    model_config = ConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        extra="ignore",
        case_sensitive=False
    )

    LOGGING_CONFIG: dict = {
        "version": 1,
        "disable_existing_loggers": False,
        "formatters": {
            "default": {
                "()": "uvicorn.logging.DefaultFormatter",
                "fmt": "%(levelname)s:     %(message)s",
                "use_colors": True,
            },
        },
        "handlers": {
            "default": {
                "formatter": "default",
                "class": "logging.StreamHandler",
                "stream": "ext://sys.stderr",
            },
        },
        "loggers": {
            "": {"handlers": ["default"], "level": "INFO"}, # Root logger
            "uvicorn.error": {"handlers": ["default"], "level": "INFO"},
            "uvicorn.access": {"propagate": False}, # Let uvicorn handle its own access logs
        },
    }


settings = AppConfig()
</file>

<file path="config/reset_config_route.py">
# config/reset_config_route.py
from fastapi import APIRouter, Depends
from app.users.auth_dependencies import get_current_admin
from app.users.user_models.user_model import User

router = APIRouter(prefix="/admin", tags=["admin"])

@router.post("/reset-all-configs")
async def reset_all_configs(
admin: User = Depends(get_current_admin)
):
"""
Reset all system configs to file defaults.
Admin-only operation.
"""
from config.ragconfig import rag_settings
from config.visionconfig import vision_settings

    # Reload settings from files
    rag_settings.__init__()
    vision_settings.__init__()

    return {
        "message": "All configs reset to defaults",
        "reset_by": admin.email
    }

</file>

<file path="scripts/__init__.py">
# scripts package
</file>

<file path="scripts/test_vision_models.py">
#!/usr/bin/env python3

# scripts/test_vision_models.py

"""
Quick Vision Model Tester
Tests all available vision models on the same image
"""
import os
import sys

# Disable proxy if needed

os.environ["NO_PROXY"] = "\*"

from PIL import Image
import ollama

# ============================================================================

# CONFIGURATION

# ============================================================================

# Path to test X-ray image

IMAGE_PATH = "test_xray.jpg" # UPDATE THIS PATH

# Test prompt

PROMPT = """Analyze this periapical dental X-ray.

CRITICAL TASK: Identify which teeth are visible.

INSTRUCTIONS:

1. Use Universal Tooth Numbering (1-32)
2. List ONLY the tooth numbers that are actually visible
3. A periapical X-ray typically shows 2-4 adjacent teeth
4. Be specific and accurate - do NOT guess

Response format: Teeth visible: [list numbers separated by commas]

What teeth do you see?"""

# Models to test

VISION_MODELS = [
"llava:7b", # Current (small, fast, less accurate)
"llava:13b", # Larger (slower, more accurate)
"llama3.2-vision", # Latest from Meta
]

# ============================================================================

# TEST EXECUTION

# ============================================================================

def test_model(model_name: str, image_path: str, prompt: str):
"""Test a single vision model"""
print(f"\n{'='*70}")
print(f"🔍 Testing: {model_name}")
print(f"{'='*70}")

    try:
        # Check if model exists
        try:
            ollama.show(model_name)
        except:
            print(f"⚠️  Model not found. Download with: ollama pull {model_name}")
            return None

        # Check if image exists
        if not os.path.exists(image_path):
            print(f"❌ Image not found: {image_path}")
            return None

        # Run vision analysis
        print(f"📸 Analyzing image: {image_path}")
        response = ollama.chat(
            model=model_name,
            messages=[{
                'role': 'user',
                'content': prompt,
                'images': [image_path]
            }]
        )

        result = response['message']['content']
        print(f"\n📋 Response:\n{result}\n")

        return result

    except Exception as e:
        print(f"❌ Error: {e}")
        import traceback
        traceback.print_exc()
        return None

def main():
"""Main test execution"""
print("\n" + "="*70)
print("VISION MODEL COMPARISON TEST")
print("="*70)
print(f"\nTest Image: {IMAGE_PATH}")
print(f"Models to test: {len(VISION_MODELS)}")

    # Check if image exists
    if not os.path.exists(IMAGE_PATH):
        print(f"\n❌ ERROR: Image not found at {IMAGE_PATH}")
        print(f"Please update IMAGE_PATH in this script or provide image")
        sys.exit(1)

    # Test each model
    results = {}
    for model in VISION_MODELS:
        result = test_model(model, IMAGE_PATH, PROMPT)
        results[model] = result

    # Summary
    print("\n" + "="*70)
    print("SUMMARY")
    print("="*70)

    for model, result in results.items():
        status = "✅ Success" if result else "❌ Failed"
        print(f"{model:20s} {status}")

    print("\n" + "="*70)
    print("Test complete!")
    print("="*70)

    # Next steps
    print("\n📝 NEXT STEPS:")
    print("1. Review the responses above")
    print("2. Document which model identified teeth correctly")
    print("3. Update config/visionconfig.py with best model:")
    print("   LLAVA_MODEL = 'llava:13b'  # or llama3.2-vision")
    print("4. Restart your application")

if **name** == "**main**":
main()
</file>

<file path="app/cdss_engine/fusion_engine.py">
# app/cdss_engine/fusion_engine.py
"""
CDSS Fusion Engine - Updated with:
1. Structured vision output support
2. Scientific confidence scoring
3. Tooth number enforcement in diagnosis
4. Enhanced prompting for accuracy
"""
import logging
import time
from typing import Dict, List, Optional

from sqlalchemy import desc, select
from sqlalchemy.ext.asyncio import AsyncSession

from app.cdss_engine.schemas import (
CDSSResponse,
ClinicalRecommendation,
ImageObservation,
NoImageProvided,
PatientHistory,
RetrievedKnowledge,
)
from app.cdss_engine.tooth_validator import tooth_validator
from app.RAGsystem.llm_client import llm_client
from app.RAGsystem.retriever import RetrieverFactory
from app.system_models.clinical_note_model.clinical_note_model import ClinicalNote
from app.system_models.patient_model.patient_model import Patient
from app.visionsystem.image_processor import ImageProcessor
from app.visionsystem.vision_client import vision_client
from config.ragconfig import rag_settings
from config.visionconfig import vision_settings

logger = logging.getLogger(**name**)

class CDSSFusionEngine:
"""
Production-ready CDSS with: - Structured vision output - Scientific confidence scoring - Tooth number enforcement - Enhanced clinical accuracy
"""

    def __init__(self):
        self.retriever_factory = RetrieverFactory()
        logger.info("🏥 CDSS Fusion Engine initialized")

    async def fetch_patient_history_with_notes(
        self, patient_id: int, db: AsyncSession, num_notes: int = 3
    ) -> tuple[PatientHistory, List[dict]]:
        """Fetch patient demographics + recent clinical notes."""
        logger.info(f"\n{'='*70}")
        logger.info(f"STEP 1: FETCH PATIENT HISTORY + CLINICAL NOTES")
        logger.info(f"{'='*70}")
        logger.info(f"📋 Patient ID: {patient_id}")

        try:
            result = await db.execute(select(Patient).where(Patient.id == patient_id))
            patient = result.scalar_one_or_none()

            if not patient:
                logger.error(f"❌ Patient {patient_id} not found")
                raise ValueError(f"Patient with ID {patient_id} not found")

            # Calculate age
            age = None
            if patient.dob:
                from datetime import datetime

                today = datetime.now().date()
                age = (
                    today.year
                    - patient.dob.year
                    - ((today.month, today.day) < (patient.dob.month, patient.dob.day))
                )

            logger.info(f"✅Patient: {patient.first_name} {patient.last_name}")
            logger.info(f"   Age: {age}, Gender: {patient.gender}")

            # Fetch clinical notes
            logger.info(f"\n📝 Fetching clinical notes (up to {num_notes})...")

            notes_query = (
                select(ClinicalNote)
                .join(ClinicalNote.encounter)
                .where(ClinicalNote.encounter.has(patient_id=patient_id))
                .order_by(desc(ClinicalNote.created_at))
                .limit(num_notes)
            )

            notes_result = await db.execute(notes_query)
            clinical_notes_raw = notes_result.scalars().all()

            formatted_notes = []
            if clinical_notes_raw:
                logger.info(f"✅Retrieved {len(clinical_notes_raw)} clinical note(s):")
                for i, note in enumerate(clinical_notes_raw, 1):
                    formatted_notes.append(
                        {
                            "date": note.created_at.strftime("%Y-%m-%d %H:%M"),
                            "type": note.note_type,
                            "content": note.content,
                            "author_id": note.author_id,
                        }
                    )
                    logger.info(
                        f"   [{i}] {note.created_at.strftime('%Y-%m-%d')} - {note.note_type}"
                    )
                    logger.info(f"       {note.content[:100]}...")
            else:
                logger.info(f"   No previous clinical notes found")

            patient_history = PatientHistory(
                patient_id=str(patient.id),
                age=age,
                gender=patient.gender,
                chief_complaint=None,
                medical_history=[],
                current_medications=[],
                allergies=[],
                previous_dental_work=None,
                symptoms_duration=None,
                pain_level=None,
            )

            return patient_history, formatted_notes

        except Exception as e:
            logger.error(f"❌ Database error: {e}", exc_info=True)
            raise

    def analyze_radiograph(
        self,
        image_bytes: bytes,
        clinical_complaint: str,
        patient_info: str,
        tooth_numbers: Optional[List[str]] = None,
        clinical_notes: Optional[List[dict]] = None,
    ) -> ImageObservation:
        """
        Analyze radiograph with structured output and tooth number focus.
        """
        logger.info(f"\n{'='*70}")
        logger.info(f"STEP 2: ANALYZE RADIOGRAPH")
        logger.info(f"{'='*70}")
        logger.info(f"🔍 Vision Model Provider: {vision_settings.VISION_MODEL_PROVIDER}")
        logger.info(f"🔍 Vision Model: {vision_settings.current_vision_model}")
        logger.info(f"📝 Chief Complaint: {clinical_complaint}")
        if tooth_numbers:
            logger.info(f"🦷 Focus Teeth: {', '.join(tooth_numbers)}")
            logger.info(f"CRITICAL: Analysis must focus on tooth/teeth {tooth_numbers[0]}")

        image = ImageProcessor.preprocess_image(image_bytes)
        logger.info(f"✅ Image preprocessed: {image.size}")

        # Build context
        enhanced_context = clinical_complaint
        if tooth_numbers:
            tooth_list = ", ".join(tooth_numbers)
            enhanced_context = f"{clinical_complaint}. Focus on tooth/teeth: {tooth_list}"

        # Build clinical notes text
        clinical_notes_text = None
        if clinical_notes:
            notes_parts = []
            for note in clinical_notes:
                notes_parts.append(f"[{note['type']}]: {note['content']}")
            clinical_notes_text = " | ".join(notes_parts)

        # Log context
        logger.info(f"\n{'─'*70}")
        logger.info(f"📤 CONTEXT SENT TO VISION MODEL:")
        logger.info(f"{'─'*70}")
        logger.info(f"Enhanced Context: {enhanced_context}")
        if clinical_notes_text:
            logger.info(f"Clinical Notes: {clinical_notes_text[:200]}...")
        if tooth_numbers:
            logger.info(f"PRIMARY FOCUS TOOTH: #{tooth_numbers[0]}")
        logger.info(f"{'─'*70}\n")

        # Call vision client with tooth number
        result = vision_client.analyze_dental_radiograph(
            image,
            context=enhanced_context,
            clinical_notes=clinical_notes_text,
            tooth_number=tooth_numbers[0] if tooth_numbers else None,
        )

        # Extract scores
        confidence_score = result.get("confidence_score", 0.5)
        quality_score = result.get("image_quality_score", 0.5)

        # Derive confidence level from scores
        overall_score = (confidence_score + quality_score) / 2
        if overall_score >= 0.8:
            confidence_level = "high"
        elif overall_score >= 0.5:
            confidence_level = "medium"
        else:
            confidence_level = "low"

        logger.info(f"✅ Analysis complete")
        logger.info(f"   Vision model Provider: {vision_settings.VISION_MODEL_PROVIDER}")
        logger.info(f"   Vision model: {vision_settings.current_vision_model}")
        logger.info(f"   Image Quality Score: {quality_score:.2f}")
        logger.info(f"   Diagnostic Confidence: {confidence_score:.2f}")
        logger.info(f"   Overall Confidence: {confidence_level}")

        # Log structured findings if available
        if result.get("structured_findings"):
            logger.info(f"\n{'─'*70}")
            logger.info(f"📊 STRUCTURED IMAGE ANALYSIS:")
            logger.info(f"{'─'*70}")
            findings = result["structured_findings"]
            logger.info(f"Focused Tooth: #{findings.get('focused_tooth', 'Not specified')}")
            logger.info(f"Teeth Visible: {', '.join(findings.get('teeth_visible', []))}")
            logger.info(f"Image Quality: {findings.get('image_quality', 'unknown')}")
            logger.info(f"\nPathology Findings:")
            logger.info(f"  Caries: {'Yes' if findings.get('caries', {}).get('present') else 'No'}")
            if findings.get("caries", {}).get("present"):
                logger.info(f"    Location: {findings['caries'].get('location')}")
            logger.info(
                f"  Periapical: {'Yes' if findings.get('periapical_pathology', {}).get('present') else 'No'}"
            )
            if findings.get("periapical_pathology", {}).get("present"):
                logger.info(
                    f"    Location: Tooth {findings['periapical_pathology'].get('location')}"
                )
            logger.info(
                f"  Bone Loss: {'Yes' if findings.get('bone_loss', {}).get('present') else 'No'}"
            )
            logger.info(f"\nPrimary Finding: {findings.get('primary_finding', 'None')}")
            logger.info(f"Severity: {findings.get('severity', 'unknown')}")
            logger.info(f"Urgency: {findings.get('urgency', 'unknown')}")
            logger.info(f"{'─'*70}\n")

        return ImageObservation(
            structured_findings=result.get("structured_findings"),
            raw_description=result["detailed_description"],
            pathology_summary=result["pathology_summary"],
            model_used=vision_settings.current_vision_model,
            focused_tooth=tooth_numbers[0] if tooth_numbers else None,
            image_quality_score=quality_score,
            diagnostic_confidence=confidence_score,
            overall_confidence=confidence_level,
        )

    def retrieve_knowledge(
        self,
        clinical_complaint: str,
        image_obs=None,  # ImageObservation | None
    ):
        """
        Retrieve knowledge with a clean, pathology-specific query.

        FIX: Do NOT use pathology_summary (may contain TOOTH_NUMBER_HERE junk).
        Instead, build a clean query from structured_findings directly.
        """

        logger.info(f"\n{'='*70}")
        logger.info(f"STEP 3: RETRIEVE CLINICAL KNOWLEDGE")
        logger.info(f"{'='*70}")

        # ── Build a clean, specific retrieval query ──────────────────────────
        query_parts = []

        # 1. Extract actual pathologies from structured findings (clean, no placeholders)
        if image_obs and image_obs.structured_findings:
            sf = image_obs.structured_findings
            tooth = image_obs.focused_tooth or sf.get("focused_tooth", "")

            pathologies = []
            if sf.get("caries", {}).get("present"):
                severity = sf["caries"].get("severity", "")
                pathologies.append(f"{severity} dental caries".strip())

            if sf.get("periapical_pathology", {}).get("present"):
                pathologies.append("periapical abscess periapical pathology treatment")

            if sf.get("bone_loss", {}).get("present"):
                bl = sf["bone_loss"]
                pathologies.append(
                    f"{bl.get('severity', '')} {bl.get('type', '')} periodontal bone loss".strip()
                )

            if sf.get("root_canal_treatment", {}).get("present"):
                pathologies.append("root canal treatment endodontic")

            urgency = sf.get("urgency", "")
            severity = sf.get("severity", "")

            if pathologies:
                query_parts.append(f"Dental diagnosis and treatment for: {', '.join(pathologies)}")
                if tooth and tooth not in ("TOOTH_NUMBER_HERE", "the most symptomatic tooth"):
                    query_parts.append(f"Tooth {tooth}")
                if urgency:
                    query_parts.append(f"Urgency: {urgency}")
            else:
                # Fallback if no specific pathology
                query_parts.append(f"Dental pain management lower jaw tooth")

        else:
            # No image - use complaint only
            query_parts.append(clinical_complaint)

        enhanced_query = ". ".join(query_parts)

        logger.info(f"🔍 Building retrieval query:")
        logger.info(f"   Query: {enhanced_query}")
        logger.info(f"\n{'─'*70}")
        logger.info(f"📤 QUERY SENT TO RETRIEVER:")
        logger.info(f"{'─'*70}")
        logger.info(f"{enhanced_query}")
        logger.info(f"{'─'*70}\n")
        logger.info(f"⚙️  Configuration:")
        logger.info(f"   Retriever: {rag_settings.RETRIEVER_TYPE}")
        logger.info(f"   K: {rag_settings.RETRIEVAL_K}")

        try:
            retriever = self.retriever_factory.get_retriever()
            docs = retriever.invoke(enhanced_query)
            logger.info(f"✅Retrieved {len(docs)} chunks")

            if len(docs) == 0:
                logger.warning("⚠️  ZERO CHUNKS RETRIEVED")
                return [], []

            knowledge = []
            available_pages = set()

            logger.info(f"📖 Retrieved Knowledge:")
            for i, doc in enumerate(docs, 1):
                pages = doc.metadata.get("pages", [])
                if not pages and "page" in doc.metadata:
                    pages = [doc.metadata["page"] + 1]
                source = doc.metadata.get("source", "Guidelines")
                preview = doc.page_content[:80].replace("\n", " ")
                logger.info(f"   [{i}] Pages {pages}: {preview}...")
                knowledge.append(
                    RetrievedKnowledge(
                        content=doc.page_content,
                        pages=pages,
                        relevance_score=doc.metadata.get("score"),
                        source=source,
                    )
                )
                available_pages.update(pages)

            sorted_pages = sorted(list(available_pages))
            logger.info(f"\n📄 Total unique pages retrieved: {sorted_pages}")
            return knowledge, sorted_pages

        except Exception as e:
            logger.error(f"❌ Retrieval error: {e}", exc_info=True)
            return [], []

    def _calculate_confidence(
        self,
        image_obs: Optional[ImageObservation],
        knowledge: List[RetrievedKnowledge],
        clinical_notes: List[dict],
        tooth_number_specified: bool,
    ) -> tuple[float, dict]:
        """
        Calculate scientific confidence score based on multiple factors.

        Returns:
            (overall_score, factor_breakdown)
        """
        factors = {}

        # 1. Image quality factor (0-1)
        if image_obs:
            factors["image_quality"] = image_obs.image_quality_score
            factors["radiographic_confidence"] = image_obs.diagnostic_confidence
        else:
            factors["image_quality"] = 0.0
            factors["radiographic_confidence"] = 0.0

        # 2. Knowledge availability factor (0-1)
        if len(knowledge) >= 5:
            factors["knowledge_availability"] = 1.0
        elif len(knowledge) >= 3:
            factors["knowledge_availability"] = 0.8
        elif len(knowledge) >= 1:
            factors["knowledge_availability"] = 0.5
        else:
            factors["knowledge_availability"] = 0.2

        # 3. Clinical context factor (0-1)
        clinical_context_score = 0.5  # baseline
        if clinical_notes:
            clinical_context_score += 0.3
        if tooth_number_specified:
            clinical_context_score += 0.2
        factors["clinical_context"] = min(clinical_context_score, 1.0)

        # 4. Consistency factor (0-1) - Do findings align?
        # If we have both image and knowledge, they should agree
        if image_obs and knowledge:
            # This is simplified - could be more sophisticated
            factors["consistency"] = 0.8
        else:
            factors["consistency"] = 0.6

        # Calculate weighted overall score
        weights = {
            "image_quality": 0.25,
            "radiographic_confidence": 0.30,
            "knowledge_availability": 0.25,
            "clinical_context": 0.10,
            "consistency": 0.10,
        }

        overall_score = sum(factors[k] * weights[k] for k in factors.keys())

        return overall_score, factors

    def fuse_and_reason(
        self,
        patient_history: PatientHistory,
        clinical_notes: List[dict],
        clinical_complaint: str,
        image_obs: Optional[ImageObservation],
        knowledge: List[RetrievedKnowledge],
        available_pages: List[int],
        tooth_numbers: Optional[List[str]] = None,
        image_analysis_error: Optional[str] = None,
    ) -> ClinicalRecommendation:
        """Generate recommendation with tooth number enforcement."""
        logger.info(f"\n{'='*70}")
        logger.info(f"STEP 4: GENERATE STRUCTURED RECOMMENDATION")
        logger.info(f"{'='*70}")
        logger.info(f"🤖 LLM Provider: {rag_settings.LLM_PROVIDER}")
        logger.info(f"🤖 LLM Model: {rag_settings.current_llm_model}")

        knowledge_available = len(knowledge) > 0

        # Calculate scientific confidence
        confidence_score, confidence_factors = self._calculate_confidence(
            image_obs, knowledge, clinical_notes, bool(tooth_numbers)
        )

        # Derive confidence level
        if confidence_score >= 0.75:
            confidence_level = "high"
        elif confidence_score >= 0.45:
            confidence_level = "medium"
        else:
            confidence_level = "low"

        logger.info(f"\n📊 CONFIDENCE CALCULATION:")
        logger.info(f"   Overall Score: {confidence_score:.3f}")
        logger.info(f"   Level: {confidence_level}")
        logger.info(f"   Factors:")
        for factor, score in confidence_factors.items():
            logger.info(f"     - {factor}: {score:.3f}")

        # Building and enhancing contexts
        patient_ctx = f"""Patient Demographics:
                - ID: {patient_history.patient_id}
                - Age: {patient_history.age or 'Unknown'}, Gender: {patient_history.gender or 'Unknown'}

                Current Complaint: {clinical_complaint}"""

        if clinical_notes:
            patient_ctx += f"\n\nRecent Clinical Notes ({len(clinical_notes)}):\n"
            for i, note in enumerate(clinical_notes, 1):
                patient_ctx += f"[{i}] {note['date']} - {note['type']}: {note['content']}\n"

        # CRITICAL: Emphasize tooth number in image context
        image_ctx = ""
        if image_obs:
            image_ctx = f"""Radiographic Analysis ({image_obs.model_used}):"""

            if image_obs.focused_tooth:
                image_ctx += f"\n\n CRITICAL - FOCUSED ANALYSIS ON TOOTH #{image_obs.focused_tooth}"
                image_ctx += f"\nAll radiographic findings relate to tooth #{image_obs.focused_tooth} unless explicitly stated otherwise.\n"

            if image_obs.structured_findings:
                import json

                image_ctx += f"\n\nSTRUCTURED FINDINGS:\n{json.dumps(image_obs.structured_findings, indent=2)}\n"

            image_ctx += f"\n\nNARRATIVE ANALYSIS:\n{image_obs.raw_description}\n"
            image_ctx += f"\nKEY PATHOLOGY:\n{image_obs.pathology_summary}"
            image_ctx += f"\n\nImage Quality: {image_obs.image_quality_score:.2f}/1.0"
            image_ctx += f"\nDiagnostic Confidence: {image_obs.diagnostic_confidence:.2f}/1.0"
        elif image_analysis_error:
            image_ctx = f"⚠️ Image provided but ANALYSIS FAILED: {image_analysis_error}"
        else:
            image_ctx = "⚠️ No radiograph provided."

        # Guidelines context
        if knowledge_available:
            guidelines_ctx = "Clinical Guidelines:\n\n"
            for i, k in enumerate(knowledge, 1):
                pages = f"Pages {k.pages}" if k.pages else ""
                guidelines_ctx += f"[{i}] {pages}\n{k.content}\n\n"
        else:
            guidelines_ctx = "⚠️ No clinical guidelines retrieved."

        # Log contexts
        logger.info(f"\n{'─'*70}")
        logger.info(f"📤 CONTEXT SENT TO LLM:")
        logger.info(f"{'─'*70}")
        logger.info(f"\n=== PATIENT CONTEXT ===")
        logger.info(f"{patient_ctx[:300]}...")
        logger.info(f"\n=== IMAGE CONTEXT ===")
        logger.info(f"{image_ctx[:500]}...")
        logger.info(f"\n=== KNOWLEDGE CONTEXT ===")
        logger.info(f"{guidelines_ctx[:300]}...")
        logger.info(f"{'─'*70}\n")

        # ENHANCED QUERY WITH TOOTH NUMBER ENFORCEMENT
        enhanced_query = clinical_complaint
        if tooth_numbers:
            enhanced_query += f"\n\nCRITICAL INSTRUCTION: The diagnosis MUST reference tooth #{tooth_numbers[0]}. Do NOT diagnose a different tooth number."

        try:
            logger.info(f"⚙️  Calling LLM with structured output...")

            result = llm_client.generate_clinical_recommendation(
                patient_context=patient_ctx,
                image_findings=image_ctx,
                retrieved_knowledge=guidelines_ctx,
                query=enhanced_query,
                knowledge_available=knowledge_available,
                available_pages=available_pages,
            )

            logger.info(f"✅Structured recommendation generated")
            logger.info(f"   Diagnosis: {result['diagnosis'][:100]}...")
            logger.info(f"   Reference pages: {result['reference_pages']}")

            return ClinicalRecommendation(
                diagnosis=result["diagnosis"],
                differential_diagnoses=result["differential_diagnoses"],
                recommended_management=result["recommended_management"],
                reference_pages=result["reference_pages"],
                confidence_score=confidence_score,
                confidence_level=confidence_level,
                confidence_factors=confidence_factors,
                llm_provider=rag_settings.LLM_PROVIDER,
            )

        except Exception as e:
            logger.error(f"❌ Generation failed: {e}", exc_info=True)

            return ClinicalRecommendation(
                diagnosis=f"Error: {str(e)}",
                differential_diagnoses=[],
                recommended_management="Consult supervising dentist.",
                reference_pages=[],
                confidence_score=0.0,
                confidence_level="low",
                confidence_factors={"error": 0.0},
                llm_provider=rag_settings.LLM_PROVIDER,
            )

    async def provide_final_recommendation(
        self,
        user_id: int,
        patient_id: int,
        chief_complaint: str,
        db: AsyncSession,
        image_bytes: Optional[bytes] = None,
        tooth_numbers: Optional[List[str]] = None,
    ) -> CDSSResponse:
        """Main CDSS pipeline with NoImageProvided support."""
        start_time = time.time()

        logger.info(f"\n{'#'*70}")
        logger.info(f"# CDSS PIPELINE - {time.strftime('%Y-%m-%d %H:%M:%S')}")
        logger.info(f"{'#'*70}")
        logger.info(f"User: (Dentist ID) {user_id}")
        logger.info(f"Patient: {patient_id}")
        logger.info(f"Complaint: {chief_complaint}")
        if tooth_numbers:
            logger.info(f"Tooth Numbers: {', '.join(tooth_numbers)}")
        logger.info(f"Config: {rag_settings.RETRIEVER_TYPE}")
        if rag_settings.RETRIEVER_TYPE == "similarity_score_threshold":
            logger.info(f"Similarity threshold: {rag_settings.SIMILARITY_THRESHOLD}")
        elif rag_settings.RETRIEVER_TYPE == "mmr":
            logger.info(f"Diversity (Lambda multiplier): {rag_settings.LAMBDA_MULT}")
            logger.info(f"Fetch K (number of results to return): {rag_settings.FETCH_K}")

        # Pipeline - fetch patient history and notes
        patient_history, clinical_notes = await self.fetch_patient_history_with_notes(
            patient_id, db, 3
        )
        patient_history.chief_complaint = chief_complaint

        # CRITICAL: Initialize both variables
        image_obs = None
        no_image_message = None
        analysis_error = None

        # Try image analysis if image provided
        if image_bytes:
            image_obs = self.analyze_radiograph(
                image_bytes,
                chief_complaint,
                f"{patient_history.age}y {patient_history.gender}",
                tooth_numbers,
                clinical_notes,
            )

        # CRITICAL FIX: Create NoImageProvided if no successful image analysis
        if image_obs is None:
            no_image_message = NoImageProvided(
                message="No radiograph image was provided for this consultation",
                image_required=False,
            )
            logger.info(f"  \n⚠️  No image analysis available")

        knowledge, available_pages = self.retrieve_knowledge(chief_complaint, image_obs)

        recommendation = self.fuse_and_reason(
            patient_history,
            clinical_notes,
            chief_complaint,
            image_obs,
            knowledge,
            available_pages,
            tooth_numbers,
            image_analysis_error=analysis_error
        )

        # ============================================================
        # TOOTH NUMBER VALIDATION (ADD THIS ENTIRE BLOCK)
        # ============================================================
        if tooth_numbers and image_obs:
            logger.info(f"{'='*70}")
            logger.info(f"🔍 VALIDATING TOOTH NUMBER CONSISTENCY")
            logger.info(f"{'='*70}")

            validation = tooth_validator.validate_and_fix(
                focus_tooth=tooth_numbers[0],
                vision_teeth=(
                    image_obs.structured_findings.get("teeth_visible", [])
                    if image_obs.structured_findings
                    else []
                ),
                diagnosis=recommendation.diagnosis,
                image_obs=image_obs.__dict__ if hasattr(image_obs, "__dict__") else image_obs,
            )

            if not validation["valid"]:
                logger.warning(f"⚠️  Validation issues detected:")
                for issue in validation["issues"]:
                    logger.warning(f"     - {issue}")

                # Apply fixes
                if "corrected_diagnosis" in validation["fixes"]:
                    recommendation.diagnosis = validation["fixes"]["corrected_diagnosis"]
                    logger.info(f"✅ FIXED: Updated diagnosis")

                if "corrected_teeth_visible" in validation["fixes"]:
                    if image_obs.structured_findings:
                        image_obs.structured_findings["teeth_visible"] = validation["fixes"][
                            "corrected_teeth_visible"
                        ]
                        logger.info(f"✅ FIXED: Corrected visible teeth")
            else:
                logger.info(f"✅ All validations passed")

            logger.info(f"{'='*70}")
        # ============================================================
        # END VALIDATION
        # ============================================================

        elapsed = time.time() - start_time

        logger.info(f"\n{'#'*70}")
        logger.info(f"✅✅✅✅ COMPLETED IN {elapsed:.2f}s✅✅✅✅")
        logger.info(f"{'#'*70}")
        logger.info(f"Diagnosis: {recommendation.diagnosis[:80]}...")
        logger.info(f"Knowledge: {len(knowledge)} chunks")
        logger.info(f"Notes: {len(clinical_notes)} items")
        logger.info(f"Reference Pages: {recommendation.reference_pages}")
        logger.info(f"Confidence: {recommendation.confidence_level}")
        logger.info(f"{'#'*70}\n")

        return CDSSResponse(
            recommendation=recommendation,
            image_observations=(
                image_obs if image_obs else no_image_message
            ),  # CRITICAL: Use proper object
            knowledge_sources=knowledge,
            reasoning_chain=f"""Analysis:
                    1. Patient: {patient_id}, {patient_history.age}y {patient_history.gender}
                    2. Complaint: {chief_complaint}
                    3. Tooth Numbers: {', '.join(tooth_numbers) if tooth_numbers else 'Not specified'}
                    4. Notes: {len(clinical_notes)} clinical notes
                    5. Image: {'Yes (' + image_obs.model_used + ')' if image_obs else 'No image'}
                    6. Knowledge: {len(knowledge)} chunks
                    7. Diagnosis: {recommendation.diagnosis}
                    8. Confidence: {recommendation.confidence_level}""",
            processing_metadata={
                "total_time_seconds": round(elapsed, 2),
                "knowledge_chunks": len(knowledge),
                "clinical_notes_count": len(clinical_notes),
                "retriever_type": rag_settings.RETRIEVER_TYPE,
                "diversity": (
                    rag_settings.LAMBDA_MULT if rag_settings.RETRIEVER_TYPE == "mmr" else None
                ),
                "fetch_k": rag_settings.FETCH_K if rag_settings.RETRIEVER_TYPE == "mmr" else None,
                "similarity_threshold": (
                    rag_settings.SIMILARITY_THRESHOLD
                    if rag_settings.RETRIEVER_TYPE == "similarity_score_threshold"
                    else None
                ),
                "llm_provider": rag_settings.LLM_PROVIDER,
                "llm_model": rag_settings.current_llm_model,
                "vision_provider": vision_settings.VISION_MODEL_PROVIDER if image_obs else "N/A",
                "vision_model": vision_settings.current_vision_model if image_obs else "N/A",
                "embedding_provider": rag_settings.EMBEDDING_PROVIDER,
                "embedding_model": rag_settings.current_embedding_model,
                "user_id": user_id,
                "patient_id": patient_id,
                "tooth_number": tooth_numbers or [],
            },
        )

</file>

<file path="app/cdss_engine/routes.py">
# app/cdss_engine/routes.py
"""
CDSS Routes - Updated with tooth_number support
"""
from fastapi import APIRouter, UploadFile, File, Form, HTTPException, Depends
from typing import Optional, List
from sqlalchemy.ext.asyncio import AsyncSession

from app.cdss_engine.fusion_engine import CDSSFusionEngine
from app.cdss_engine.schemas import CDSSRequest, CDSSResponse
from app.database.connection import get_db
from app.users.auth_dependencies import get_current_user, get_current_user_id

router = APIRouter()
cdss_engine = CDSSFusionEngine()

@router.post("/provide_final_recommendation", response_model=CDSSResponse)
async def provide_final_recommendation(
patient_id: int = Form(..., description="Patient database ID"),
chief_complaint: str = Form(..., description="Main clinical complaint/question"),
tooth_number: Optional[str] = Form(None, description="Tooth number(s) affected (e.g., '36' or '36,37')"),
image: UploadFile = File(..., description="Dental radiograph (REQUIRED)"),
db: AsyncSession = Depends(get_db),
user_id: int = Depends(get_current_user_id)
):
"""
Main CDSS Endpoint - Complete Clinical Decision Support

    This endpoint orchestrates:
    1. Fetch patient history from database
    2. Analyze radiograph (REQUIRED) with configured vision model
    3. Retrieve relevant clinical guidelines (RAG)
    4. Generate structured clinical recommendation with configured LLM

    **Form Parameters:**
    - **patient_id**: Patient database ID (required)
    - **chief_complaint**: Main complaint or clinical question (required)
    - **tooth_number**: Tooth number(s) affected (optional, e.g., '36' or '36,37')
    - **image**: Dental periapical radiograph (REQUIRED, JPEG/PNG)

    **Configuration:**
    - Vision Model: Set in config/visionconfig.py (VISION_MODEL_PROVIDER)
    - LLM: Set in config/ragconfig.py (LLM_PROVIDER)
    - Embeddings: Set in config/ragconfig.py (EMBEDDING_PROVIDER)

    **Returns:**
    ```json
    {
        "recommendation": {
            "diagnosis": "Primary diagnosis",
            "differential_diagnoses": ["Alt 1", "Alt 2"],
            "recommended_management": "Treatment plan",
            "reference_pages": [100, 101, 102]
        },
        "image_observations": {
            "raw_description": "Full radiograph analysis",
            "pathology_summary": "Key findings",
            "confidence": "high",
            "model_used": "llava"
        },
        "processing_metadata": {
            "vision_provider": "llava",
            "llm_provider": "gpt-4",
            "total_time_seconds": 8.5
        }
    }
    ```
    """
    try:
        # Validate that image is provided and not empty
        if not image or image.size == 0:
            raise HTTPException(
                status_code=400,
                detail="Radiograph image is REQUIRED for this analysis."
            )

        # Parse tooth numbers if provided
        tooth_numbers = None
        if tooth_number:
            # Support comma-separated tooth numbers
            tooth_numbers = [t.strip() for t in tooth_number.split(',')]

        # Read image
        image_bytes = await image.read()

        # Execute CDSS pipeline
        result = await cdss_engine.provide_final_recommendation(
            patient_id=patient_id,
            chief_complaint=chief_complaint,
            db=db,
            image_bytes=image_bytes,
            tooth_numbers=tooth_numbers,
            user_id=user_id
        )

        return result

    except HTTPException:
        raise
    except ValueError as e:
        # Patient not found or validation error
        raise HTTPException(status_code=404, detail=str(e))
    except Exception as e:
        # Other errors
        raise HTTPException(
            status_code=500,
            detail=f"CDSS processing error: {str(e)}"
        )

@router.post("/recommendation_without_radiograph", response_model=CDSSResponse)
async def recommendation_from_json(
request: CDSSRequest,
db: AsyncSession = Depends(get_db),
user_id: int = Depends(get_current_user_id)
):
"""
CDSS Recommendation from JSON payload (no image upload).

    Use when:
    - No radiograph available
    - Testing with JSON payloads
    - Integrating from other systems

    **Request Body:**
    ```json
    {
        "patient_id": 123,
        "chief_complaint": "severe toothache lower right",
        "user_id": 1
    }
    ```
    """
    try:
        result = await cdss_engine.provide_final_recommendation(
            patient_id=request.patient_id,
            chief_complaint=request.chief_complaint,
            db=db,
            image_bytes=None,
            tooth_numbers=None,
            user_id=user_id
        )
        return result
    except ValueError as e:
        raise HTTPException(status_code=404, detail=str(e))
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

from fastapi import status
from fastapi.responses import JSONResponse
from sqlalchemy import text
from pydantic import BaseModel
from typing import Any, Dict

# Health check response models

class ComponentHealth(BaseModel):
status: str
provider: Optional[str] = None
details: Dict[str, Any] = {}

class SystemHealth(BaseModel):
overall_status: str
database: ComponentHealth
vision_model: ComponentHealth
llm: ComponentHealth
embeddings: ComponentHealth
retriever: ComponentHealth

@router.get("/systemconfig", response_model=SystemHealth, tags=["System Health"])
async def cdss_health_check(db: AsyncSession = Depends(get_db)):
"""Check CDSS engine configuration and health of dependent services."""
from config.visionconfig import vision_settings
from config.ragconfig import rag_settings
from app.visionsystem.vision_client import vision_client
from app.RAGsystem.llm_client import llm_client

    components: Dict[str, Dict] = {}
    is_healthy = True

    # 1. Database Health
    try:
        await db.execute(text("SELECT 1"))
        components["database"] = {"status": "healthy", "provider": "postgresql"}
    except Exception as e:
        is_healthy = False
        components["database"] = {
            "status": "unhealthy",
            "provider": "postgresql",
            "details": {"error": str(e)},
        }

    # 2. Vision Model Health
    try:
        info = vision_client.get_model_info()
        components["vision_model"] = {
            "status": "healthy",
            "provider": vision_settings.VISION_MODEL_PROVIDER,
            "details": info,
        }
    except Exception as e:
        is_healthy = False
        components["vision_model"] = {
            "status": "unhealthy",
            "provider": vision_settings.VISION_MODEL_PROVIDER,
            "details": {"error": str(e)},
        }

    # 3. LLM Health
    try:
        info = llm_client.get_model_info()
        components["llm"] = {
            "status": "healthy",
            "provider": rag_settings.LLM_PROVIDER,
            "details": info,
        }
    except Exception as e:
        is_healthy = False
        components["llm"] = {
            "status": "unhealthy",
            "provider": rag_settings.LLM_PROVIDER,
            "details": {"error": str(e)},
        }

    # 4. Embeddings Configuration
    try:
        components["embeddings"] = {
            "status": "healthy",
            "provider": rag_settings.EMBEDDING_PROVIDER,
            "details": {
                "model": rag_settings.current_embedding_model,
                "chunk_size": rag_settings.CHUNK_SIZE,
                "overlap": rag_settings.CHUNK_OVERLAP,
            },

        }
    except Exception as e:
        is_healthy = False
        components["embeddings"] = {
            "status": "unhealthy",
            "provider": rag_settings.EMBEDDING_PROVIDER,
            "details": {"error": str(e)},
        }

    # 5. Retriever Configuration
    try:
        retriever_details = {
            "type": rag_settings.RETRIEVER_TYPE,
            "k": rag_settings.RETRIEVAL_K,
        }
        if rag_settings.RETRIEVER_TYPE == "mmr":
            retriever_details["lambda_multiplier"] = rag_settings.LAMBDA_MULT
            retriever_details["fetch_k"] = rag_settings.FETCH_K
        elif rag_settings.RETRIEVER_TYPE == "similarity_score_threshold":
            retriever_details["similarity_threshold"] = rag_settings.SIMILARITY_THRESHOLD

        components["retriever"] = {
            "status": "healthy",
            "provider": "langchain",
            "details": retriever_details,
        }
    except Exception as e:
        is_healthy = False
        components["retriever"] = {
            "status": "unhealthy",
            "provider": "langchain",
            "details": {"error": str(e)},
        }

    # Final response assembly
    response_payload = {"overall_status": "healthy" if is_healthy else "unhealthy", **components}

    http_status = status.HTTP_200_OK if is_healthy else status.HTTP_503_SERVICE_UNAVAILABLE

    return JSONResponse(content=response_payload, status_code=http_status)

</file>

<file path="app/RAGsystem/chains.py">
# app/RAGsystem/chains.py
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough, RunnableLambda, RunnableParallel
from config.ragconfig import rag_settings
from app.RAGsystem.llm_providers import get_llm_by_provider
import logging

logger = logging.getLogger(**name**)

class ClinicalRAGChain:
"""LCEL-based Clinical RAG Chain."""

    def __init__(self, retriever):
        self.retriever = retriever
        # Get LLM instance based on provider
        self.llm = get_llm_by_provider()

        # Clinical system prompt
        self.system_prompt = """You are a Clinical Decision Support System analyzing dental/oral health guidelines.

Your task is to provide an evidence-based, structured JSON response based strictly on the retrieved context.

RESPONSE JSON SCHEMA:
{{
  "diagnosis": "string",
  "differential_diagnoses": ["string"],
  "recommended_management": {{
    "pharmacological": {{
      "analgesics": [{{"name": "string", "dose": "string", "reference_page": int}}],
"antibiotics": [{{"name": "string", "dose": "string", "reference_page": int}}]
}},
"non_pharmacological": [{{"description": "string", "category": "string", "reference_page": int}}],
"follow_up": "string"
}},
"precautions": [{{"condition": "string", "note": "string", "reference_page": int}}]
}}

RULES:

1.  STRICTLY use the provided clinical guidelines context. Do not add outside knowledge.
2.  Cite the specific page number for EVERY recommendation by setting the "reference_page" field.
3.  If information is insufficient, state: "Insufficient information in retrieved guidelines."
4.  Distinguish between "pharmacological" and "non_pharmacological" treatments.
5.  Note any "precautions" or contraindications mentioned in the context.
6.  Your FINAL output must be ONLY the resulting JSON object. Do not include markdown formatting (e.g., `json ... `) or any other descriptive text.
    """

            self.prompt = ChatPromptTemplate.from_messages([
                ("system", self.system_prompt),
                ("human", """Clinical Guidelines Context:
                {context}

                Question: {question}

                Provide a structured clinical answer with page citations.""")
            ])

        def format_docs(self, docs):
            """Format documents with clear provenance."""
            formatted = []
            for i, doc in enumerate(docs, 1):
                pages = doc.metadata.get('pages', [])
                if not pages and 'page' in doc.metadata:
                    pages = [doc.metadata['page'] + 1]  # Convert 0-indexed to 1-indexed

                page_str = f"Pages {', '.join(map(str, pages))}" if pages else "Unknown"
                formatted.append(f"[{i}] [{page_str}]\n{doc.page_content}")

            return "\n\n---\n\n".join(formatted)

        def create_chain(self):
            """Build LCEL chain, including raw retrieved docs in output."""

            # This will get the question and context, and raw documents
            setup_and_retrieve = RunnableParallel(
                question=RunnablePassthrough(), # Original question comes in here
                retrieved_docs=self.retriever, # Retrieve docs based on question
            ).assign(
                context=lambda x: self.format_docs(x["retrieved_docs"]) # Format docs into context string
            )

            # Now, combine this with the LLM call
            full_chain = setup_and_retrieve | {
                "answer": self.prompt | self.llm | StrOutputParser(),
                "retrieved_docs": lambda x: x["retrieved_docs"] # Pass raw docs through to the end
            }

            return full_chain

        def invoke(self, question: str):
            """Execute chain and log details."""
            logger.info(f"Processing RAG query: {question[:100]}...")
            llm_model_name = self.llm.model if hasattr(self.llm, 'model') else "Unknown LLM Model"
            logger.info(f"----------------------- LLM Model Used: {llm_model_name} ------------------\n") # Log model name

            chain = self.create_chain()
            result = chain.invoke(question)

            answer = result["answer"]
            retrieved_docs = result["retrieved_docs"]

            logger.info(f"Retrieved {len(retrieved_docs)} chunks.")
            for i, doc in enumerate(retrieved_docs):
                logger.info(f"------------------------------ Chunk {i+1} -----------------------------------")
                logger.info(f"Content: {doc.page_content[:200]}...") # Log highlights (first 200 chars)
                logger.info(f"Source: {doc.metadata.get('source', 'Unknown')}, Page: {doc.metadata.get('page', 'Unknown')}")
                logger.info("===============================================================================\n\n")

            return answer

# Legacy support

def answer_question(question: str, retriever):
"""Backward compatible wrapper."""
chain = ClinicalRAGChain(retriever)
return chain.invoke(question)
</file>

<file path="app/RAGsystem/llm_client.py">
# app/RAGsystem/llm_client.py
"""
LLM Client - FIXED with simple text parsing (no complex Pydantic)
This version uses regex-based extraction instead of structured output for reliability with Ollama
"""
import logging
import re
from typing import Dict, List, Optional

from config.ragconfig import rag_settings
from app.RAGsystem.llm_providers import get_llm_by_provider

logger = logging.getLogger(**name**)

class LLMClient:
"""LLM client with simple text-based output parsing (reliable with all providers)"""

    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def generate_clinical_recommendation(
        self,
        patient_context: str,
        image_findings: str,
        retrieved_knowledge: str,
        query: str,
        knowledge_available: bool,
        available_pages: List[int] = None,
    ) -> Dict:
        """
        Generate clinical recommendation using SIMPLE text format.

        This approach is MORE RELIABLE than JSON/Pydantic with local LLMs.
        """
        provider = rag_settings.LLM_PROVIDER
        logger.info(f"🤖 Generating recommendation with {provider} - Model: {rag_settings.current_llm_model}")

        # Extract pages from retrieved knowledge if not provided
        if available_pages is None and knowledge_available:
            available_pages = self._extract_available_pages(retrieved_knowledge)
        elif not knowledge_available:
            available_pages = []

        # Build SIMPLE prompt (no JSON schema)
        prompt = self._build_simple_prompt(
            patient_context,
            image_findings,
            retrieved_knowledge,
            query,
            available_pages
        )

        # Get LLM client
        llm = self._get_llm_client()

        try:
            # Invoke LLM
            logger.info(f"⚙️  Calling LLM...")
            response = llm.invoke(prompt)

            # Parse simple text response
            result = self._parse_simple_response(response.content, available_pages)

            logger.info(f"✅ Generated recommendation successfully")
            logger.info(f"   Diagnosis: {result['diagnosis'][:80]}...")
            logger.info(f"   Reference pages: {result['reference_pages']}")

            return result

        except Exception as e:
            logger.error(f"❌ Generation failed: {e}")
            import traceback
            traceback.print_exc()
            return self._fallback_response()

    def _get_llm_client(self):
        """Get simple LLM client (no structured output)"""
        return get_llm_by_provider()

    def _build_simple_prompt(
        self,
        patient_context: str,
        image_findings: str,
        retrieved_knowledge: str,
        query: str,
        available_pages: List[int]
    ) -> str:
        """Build SIMPLE text prompt (much more reliable than JSON)"""

        pages_str = ", ".join(map(str, available_pages)) if available_pages else "None"

        prompt = f"""You are a clinical decision support system for dentistry.

                === PATIENT INFORMATION ===
                {patient_context}

                === RADIOGRAPHIC FINDINGS ===
                {image_findings}

                === CLINICAL GUIDELINES ===
                {retrieved_knowledge}

                === AVAILABLE PAGES ===
                You may ONLY reference these page numbers: {pages_str}
                DO NOT invent or make up any page numbers not in this list.

                === CLINICAL QUERY ===
                {query}

                === CRITICAL INSTRUCTIONS ===
                1. EXTRACT the tooth number from the patient context or clinical notes
                2. Your diagnosis MUST explicitly mention that specific tooth number
                3. Use ONLY page numbers from the available list above
                4. Format your response EXACTLY as shown below (do not add extra sections)

                === RESPONSE FORMAT ===
                Use this EXACT structure (include the section headers):

                DIAGNOSIS: [Write complete diagnosis here, MUST include tooth number]

                DIFFERENTIAL:
                - [Alternative diagnosis 1]
                - [Alternative diagnosis 2]

                MANAGEMENT:
                1. [First treatment step]
                2. [Second treatment step]
                3. [Third treatment step]
                4. [Fourth treatment step if needed]

                PAGES: [comma-separated page numbers only, e.g., 385, 388, 401]

                Now provide your clinical recommendation:"""

        return prompt

    def _parse_simple_response(self, text: str, available_pages: List[int]) -> Dict:
        """
        Parse simple text format using regex (deterministic, no LLM involved).
        This is MUCH more reliable than JSON parsing with local LLMs.
        """
        logger.info(f"📥 Parsing response ({len(text)} chars)...")

        # Extract diagnosis
        diag_match = re.search(
            r'DIAGNOSIS:\s*(.+?)(?:\n\n|DIFFERENTIAL:|$)',
            text,
            re.DOTALL | re.IGNORECASE
        )
        diagnosis = diag_match.group(1).strip() if diag_match else "Unable to generate diagnosis"

        # Extract differentials
        diff_match = re.search(
            r'DIFFERENTIAL:\s*(.+?)(?:\n\n|MANAGEMENT:|$)',
            text,
            re.DOTALL | re.IGNORECASE
        )
        differentials = []
        if diff_match:
            diff_text = diff_match.group(1)
            # Extract lines starting with dash or number
            diff_lines = re.findall(r'[-•]\s*(.+?)(?:\n|$)', diff_text)
            differentials = [d.strip() for d in diff_lines if d.strip()][:3]  # Max 3

        # Extract management
        mgmt_match = re.search(
            r'MANAGEMENT:\s*(.+?)(?:\n\n|PAGES:|$)',
            text,
            re.DOTALL | re.IGNORECASE
        )
        management = mgmt_match.group(1).strip() if mgmt_match else "Consult supervising dentist for treatment plan."

        # Extract pages
        pages_match = re.search(
            r'PAGES:\s*(.+?)(?:\n|$)',
            text,
            re.IGNORECASE
        )
        pages = []
        if pages_match:
            # Extract all numbers
            page_nums = re.findall(r'\d+', pages_match.group(1))
            # Filter to only available pages (validation)
            pages = [int(p) for p in page_nums if int(p) in available_pages]

        logger.info(f"✅ Parsed successfully:")
        logger.info(f"   Diagnosis: {len(diagnosis)} chars")
        logger.info(f"   Differentials: {len(differentials)}")
        logger.info(f"   Management: {len(management)} chars")
        logger.info(f"   Pages: {pages}")

        return {
            "diagnosis": diagnosis,
            "differential_diagnoses": differentials,
            "recommended_management": management,
            "reference_pages": pages
        }

    def _extract_available_pages(self, retrieved_knowledge: str) -> List[int]:
        """
        Extract all page numbers from retrieved knowledge context.
        Ensures we only reference pages that were actually retrieved.
        """
        available_pages = set()

        # Match patterns like "Pages [385]" or "[1] Pages [385, 386]"
        page_patterns = [
            r'Pages?\s*\[([0-9,\s]+)\]',  # Pages [385]
            r'Pages?\s*([0-9,\s]+)',      # Pages 385, 386
            r'\[Pages?\s*([0-9,\s]+)\]',  # [Pages 385]
        ]

        for pattern in page_patterns:
            matches = re.findall(pattern, retrieved_knowledge, re.IGNORECASE)
            for match in matches:
                page_nums = re.findall(r'\d+', match)
                available_pages.update(int(p) for p in page_nums)

        sorted_pages = sorted(list(available_pages))
        logger.info(f"📖 Extracted pages from knowledge: {sorted_pages}")
        return sorted_pages

    def _fallback_response(self) -> Dict:
        """Fallback response if generation completely fails"""
        return {
            "diagnosis": "Unable to generate recommendation - system error. Please consult supervising dentist.",
            "differential_diagnoses": [],
            "recommended_management": "1. Comprehensive clinical examination; 2. Additional diagnostic imaging if needed; 3. Consult supervising dentist for diagnosis and treatment planning.",
            "reference_pages": []
        }

    def get_model_info(self) -> Dict:
        """Get current LLM configuration"""
        return {
            "provider": rag_settings.LLM_PROVIDER,
            "model": rag_settings.current_llm_model,
            "temperature": rag_settings.LLM_TEMPERATURE,
            "structured_output": False,  # Using simple text parsing
            "parser": "regex",
            "reliable": True  # Text parsing is much more reliable
        }

# Global instance

llm_client = LLMClient()
</file>

<file path="app/RAGsystem/llm_providers.py">
# app/RAGsystem/llm_providers.py
"""
Additional LLM Provider Implementations for RAG System
Groq: Ultra-fast LPU inference
Gemini: Google's multimodal LLM with large context
"""
import logging
from typing import Dict, List
from config.ragconfig import rag_settings

logger = logging.getLogger(**name**)

def get_groq_llm():
"""
Get Groq LLM for text generation.
Uses Groq's LPU for 10-20x faster inference than GPU.
"""
try:
from langchain_groq import ChatGroq
import os

        api_key = rag_settings.GROQ_API_KEY or os.getenv("GROQ_API_KEY")
        if not api_key:
            raise ValueError("GROQ_API_KEY not set in environment")

        llm = ChatGroq(
            model=rag_settings.GROQ_LLM_MODEL,
            temperature=rag_settings.LLM_TEMPERATURE,
            max_tokens=rag_settings.MAX_TOKENS,
            groq_api_key=api_key,
        )

        logger.info(f"✅ Groq LLM initialized: {rag_settings.GROQ_LLM_MODEL}")
        return llm

    except ImportError:
        raise ImportError("langchain-groq not installed. Run: pip install langchain-groq")
    except Exception as e:
        raise RuntimeError(f"Groq LLM initialization failed: {e}")

def get_gemini_llm():
"""
Get Gemini LLM for text generation.
Supports up to 2M token context window.
"""
try:
from langchain_google_genai import ChatGoogleGenerativeAI
import os

        api_key = rag_settings.GEMINI_API_KEY or os.getenv("GEMINI_API_KEY")
        if not api_key:
            raise ValueError("GEMINI_API_KEY not set in environment")

        llm = ChatGoogleGenerativeAI(
            model=rag_settings.GEMINI_LLM_MODEL,
            temperature=rag_settings.LLM_TEMPERATURE,
            max_output_tokens=rag_settings.MAX_TOKENS,
            google_api_key=api_key,
        )

        logger.info(f"✅ Gemini LLM initialized: {rag_settings.GEMINI_LLM_MODEL}")
        return llm

    except ImportError:
        raise ImportError("langchain-google-genai not installed. Run: pip install langchain-google-genai")
    except Exception as e:
        raise RuntimeError(f"Gemini LLM initialization failed: {e}")

def get_llm_by_provider(provider: str = None):
"""
Get LLM instance based on provider.
Factory function for all LLM providers.

    Args:
        provider: Override rag_settings.LLM_PROVIDER

    Returns:
        LangChain LLM instance
    """
    provider = provider or rag_settings.LLM_PROVIDER

    if provider == "ollama":
        from langchain_ollama import ChatOllama
        return ChatOllama(
            model=rag_settings.OLLAMA_LLM_MODEL,
            temperature=rag_settings.LLM_TEMPERATURE,
            format="json",
        )

    elif provider == "openai":
        from langchain_openai import ChatOpenAI
        return ChatOpenAI(
            model=rag_settings.OPENAI_LLM_MODEL,
            temperature=rag_settings.LLM_TEMPERATURE,
            max_tokens=rag_settings.MAX_TOKENS,
            api_key=rag_settings.OPENAI_API_KEY,
        )

    elif provider == "claude":
        from langchain_anthropic import ChatAnthropic
        return ChatAnthropic(
            model=rag_settings.CLAUDE_LLM_MODEL,
            temperature=rag_settings.LLM_TEMPERATURE,
            max_tokens=rag_settings.MAX_TOKENS,
            api_key=rag_settings.CLAUDE_API_KEY,
        )

    elif provider == "groq":
        return get_groq_llm()

    elif provider == "gemini":
        return get_gemini_llm()

    else:
        raise ValueError(f"Unknown LLM provider: {provider}")

</file>

<file path="app/RAGsystem/retriever.py">
# app/RAGsystem/retriever.py
"""
Retriever Factory - Fixed for all retriever types
"""
from langchain_chroma import Chroma
from langchain_classic.retrievers import MultiQueryRetriever, ContextualCompressionRetriever
from langchain_classic.retrievers.document_compressors import LLMChainExtractor
from app.RAGsystem.embeddings import embedding_provider
from app.RAGsystem.llm_providers import get_llm_by_provider
from config.ragconfig import rag_settings
import logging

logger = logging.getLogger(**name**)

class RetrieverFactory:
"""Factory for creating different retrieval strategies."""

    def __init__(self, persist_dir: str = None):
        self.persist_dir = persist_dir or rag_settings.PERSIST_DIR
        self.embeddings = embedding_provider.get_embeddings()
        self.vectorstore = None

    def _get_vectorstore(self):
        """Lazy load vectorstore."""
        if self.vectorstore is None:
            self.vectorstore = Chroma(
                persist_directory=self.persist_dir,
                embedding_function=self.embeddings
            )
        return self.vectorstore

    def create_similarity_retriever(self, k: int = None):
        """
        Standard similarity-based retrieval.
        DOES NOT support score_threshold.
        """
        k = k or rag_settings.RETRIEVAL_K

        logger.info(f"Creating similarity retriever (k={k})")
        return self._get_vectorstore().as_retriever(
            search_type="similarity",
            search_kwargs={"k": k}
        )

    def create_mmr_retriever(self, k: int = None, fetch_k: int = None, lambda_mult: float = None):
        """
        Maximal Marginal Relevance - balances relevance with diversity.
        """
        k = k or rag_settings.RETRIEVAL_K
        fetch_k = fetch_k or rag_settings.FETCH_K
        lambda_mult = lambda_mult or rag_settings.LAMBDA_MULT

        logger.info(f"{'-'*70}")
        logger.info(f"Creating MMR retriever (k={k}, fetch_k={fetch_k}, lambda={lambda_mult})")
        logger.info(f"{'-'*70}")

        return self._get_vectorstore().as_retriever(
            search_type="mmr",
            search_kwargs={
                "k": k,
                "fetch_k": fetch_k,
                "lambda_mult": lambda_mult
            }
        )

    def create_multi_query_retriever(self, llm_model: str = None):
        """
        Generates multiple query variations to improve recall.
        """
        base_retriever = self.create_mmr_retriever(k=4, fetch_k=15, lambda_mult=0.5)

        # Use provider-aware LLM
        llm = get_llm_by_provider()

        logger.info("Creating multi-query retriever")
        multi_query = MultiQueryRetriever.from_llm(
            retriever=base_retriever,
            llm=llm
        )

        return multi_query

    def create_compression_retriever(self, base_retriever=None):
        """
        Uses LLM to filter irrelevant content from retrieved documents.
        """
        # Use provider-aware LLM
        llm = get_llm_by_provider()
        compressor = LLMChainExtractor.from_llm(llm)

        base = base_retriever or self.create_mmr_retriever()

        logger.info("Creating compression retriever")
        return ContextualCompressionRetriever(
            base_compressor=compressor,
            base_retriever=base
        )

    def create_similarity_score_threshold_retriever(self, k: int = None, threshold: float = None):
        """
        Similarity-based retriever with score threshold.
        """
        k = k or rag_settings.RETRIEVAL_K
        threshold = threshold or rag_settings.SIMILARITY_THRESHOLD

        logger.info(f"Creating similarity score threshold retriever (k={k}, threshold={threshold})")
        return self._get_vectorstore().as_retriever(
            search_type="similarity_score_threshold",
            search_kwargs={
                "k": k,
                "score_threshold": threshold
            }
        )

    def get_retriever(self, retriever_type: str = None):
        """
        Get retriever based on configuration.
        """
        rtype = retriever_type or rag_settings.RETRIEVER_TYPE

        if rtype == "similarity":
            return self.create_similarity_retriever()
        elif rtype == "mmr":
            return self.create_mmr_retriever()
        elif rtype == "multi_query":
            return self.create_multi_query_retriever()
        elif rtype == "similarity_score_threshold":
            return self.create_similarity_score_threshold_retriever()
        else:
            raise ValueError(f"Unknown retriever type: {rtype}")

def get_retriever(persist_dir: str = None, strategy: str = None):
"""Convenience function for backward compatibility."""
factory = RetrieverFactory(persist_dir)
return factory.get_retriever(strategy)
</file>

<file path="app/RAGsystem/routes.py">
# app/RAGsystem/routes.py - COMPLETE UPDATE
"""
RAG Routes with Schema Normalization
Ensures consistent response structure for frontend
"""
import logging
import os
import shutil
import json
from pathlib import Path
from typing import Dict, Any

from fastapi import APIRouter, Depends, File, HTTPException, UploadFile
from pydantic import BaseModel
from starlette.concurrency import run_in_threadpool

from app.RAGsystem.chains import ClinicalRAGChain
from app.RAGsystem.embeddings import embedding_provider
from app.RAGsystem.retriever import RetrieverFactory
from app.RAGsystem.schemas import DocumentUploadResponse, QuestionPayload, RAGResponse
from app.RAGsystem.response_normalizer import normalize_rag_response, validate_response
from app.users.auth_dependencies import get_current_user_id
from config.config_schemas import RAGConfigRequest, RAGConfigResponse
from config.ragconfig import rag_settings
from scripts.ingest_documents import ingest_pdf

logger = logging.getLogger(**name**)
router = APIRouter()

@router.post("/answer_question", response_model=RAGResponse)
async def answer_question_endpoint(payload: QuestionPayload):
"""
RAG endpoint with guaranteed consistent schema.

    Features:
    - Normalizes LLM response variations to single standard format
    - Ensures reference pages are always present
    - Validates response before returning

    Returns standardized JSON suitable for frontend consumption.
    """
    if not payload.question:
        raise HTTPException(status_code=400, detail="Question is required")

    try:
        # Get RAG chain and invoke
        retriever = RetrieverFactory().get_retriever()
        chain = ClinicalRAGChain(retriever)
        raw_answer = chain.invoke(payload.question)

        logger.info(f"📝 Raw LLM response: {len(raw_answer)} chars")

        # Parse JSON
        try:
            # Robust JSON cleaning
            cleaned = raw_answer.strip()
            # Remove markdown code blocks if present
            if cleaned.startswith("```"):
                # Handle ```json or just ```
                lines = cleaned.split("\n")
                if lines[0].startswith("```"):
                    lines = lines[1:]
                if lines[-1].startswith("```"):
                    lines = lines[:-1]
                cleaned = "\n".join(lines).strip()

            parsed_data = json.loads(cleaned)
            logger.info("✅ Parsed LLM response as JSON")
        except json.JSONDecodeError as e:
            logger.error(f"❌ LLM returned invalid JSON: {e}")
            logger.error(f"   Raw response head: {raw_answer[:100]}...")
            raise HTTPException(
                status_code=500,
                detail="LLM did not return valid JSON. Please try again or contact support."
            )

        # NORMALIZE - Convert to standard schema
        normalized_data = normalize_rag_response(parsed_data)
        logger.info(f"🔄 Normalized response - found {len(normalized_data.get('reference_pages', []))} reference pages")

        # VALIDATE - Ensure all required fields present
        validate_response(normalized_data)
        logger.info("✅ Response validated successfully")

        return RAGResponse(
            answer=normalized_data,
            retrieval_strategy=rag_settings.RETRIEVER_TYPE,
            llm_provider=rag_settings.LLM_PROVIDER,
            model_used=rag_settings.current_llm_model
        )

    except HTTPException:
        raise  # Re-raise HTTP exceptions as-is
    except Exception as e:
        logger.error(f"❌ RAG endpoint error: {e}", exc_info=True)
        raise HTTPException(
            status_code=500,
            detail=f"Internal error processing question: {str(e)}"
        )

@router.get("/config", response_model=RAGConfigResponse)
async def get_rag_config():
"""Get current RAG system configuration."""
return RAGConfigResponse( # LLM
llm_provider=rag_settings.LLM_PROVIDER,
current_llm_model=rag_settings.current_llm_model,
ollama_llm_model=rag_settings.OLLAMA_LLM_MODEL,
openai_llm_model=rag_settings.OPENAI_LLM_MODEL,
claude_llm_model=rag_settings.CLAUDE_LLM_MODEL,
groq_llm_model=rag_settings.GROQ_LLM_MODEL,
gemini_llm_model=rag_settings.GEMINI_LLM_MODEL, # Embedding
embedding_provider=rag_settings.EMBEDDING_PROVIDER,
current_embedding_model=rag_settings.current_embedding_model,
ollama_embedding_model=rag_settings.OLLAMA_EMBEDDING_MODEL,
hf_embedding_model=rag_settings.HF_EMBEDDING_MODEL, # Retrieval
retriever_type=rag_settings.RETRIEVER_TYPE,
retrieval_k=rag_settings.RETRIEVAL_K,
fetch_k=rag_settings.FETCH_K,
lambda_mult=rag_settings.LAMBDA_MULT,
similarity_threshold=rag_settings.SIMILARITY_THRESHOLD, # Generation
llm_temperature=rag_settings.LLM_TEMPERATURE,
max_tokens=rag_settings.MAX_TOKENS, # Processing
chunk_size=rag_settings.CHUNK_SIZE,
chunk_overlap=rag_settings.CHUNK_OVERLAP,
pdf_path=rag_settings.PDF_PATH,
persist_dir=rag_settings.PERSIST_DIR,
device=rag_settings.effective_device,
)

@router.post("/config", response_model=RAGConfigResponse)
async def update_rag_config(config: RAGConfigRequest):
"""Update RAG configuration with cache invalidation."""
updated_fields = []

    # LLM Provider and Models
    if config.llm_provider is not None:
        rag_settings.LLM_PROVIDER = config.llm_provider
        updated_fields.append(f"llm_provider → {config.llm_provider}")

    if config.ollama_llm_model is not None:
        rag_settings.OLLAMA_LLM_MODEL = config.ollama_llm_model
        updated_fields.append(f"ollama_llm_model → {config.ollama_llm_model}")

    if config.openai_llm_model is not None:
        rag_settings.OPENAI_LLM_MODEL = config.openai_llm_model
        updated_fields.append(f"openai_llm_model → {config.openai_llm_model}")

    if config.claude_llm_model is not None:
        rag_settings.CLAUDE_LLM_MODEL = config.claude_llm_model
        updated_fields.append(f"claude_llm_model → {config.claude_llm_model}")

    if config.groq_llm_model is not None:
        rag_settings.GROQ_LLM_MODEL = config.groq_llm_model
        updated_fields.append(f"groq_llm_model → {config.groq_llm_model}")

    if config.gemini_llm_model is not None:
        rag_settings.GEMINI_LLM_MODEL = config.gemini_llm_model
        updated_fields.append(f"gemini_llm_model → {config.gemini_llm_model}")

    # Embedding Provider
    if config.embedding_provider is not None:
        old_provider = rag_settings.EMBEDDING_PROVIDER
        rag_settings.EMBEDDING_PROVIDER = config.embedding_provider
        updated_fields.append(f"embedding_provider → {config.embedding_provider}")

        # Cache invalidation
        if old_provider != config.embedding_provider:
            embedding_provider.refresh()
            RetrieverFactory()._retriever = None
            logger.info("🔄 Cleared embedding cache")

    if config.ollama_embedding_model is not None:
        rag_settings.OLLAMA_EMBEDDING_MODEL = config.ollama_embedding_model
        updated_fields.append(f"ollama_embedding_model → {config.ollama_embedding_model}")
        embedding_provider.refresh()

    if config.hf_embedding_model is not None:
        rag_settings.HF_EMBEDDING_MODEL = config.hf_embedding_model
        updated_fields.append(f"hf_embedding_model → {config.hf_embedding_model}")
        embedding_provider.refresh()

    # Retrieval Settings
    if config.retriever_type is not None:
        old_type = rag_settings.RETRIEVER_TYPE
        rag_settings.RETRIEVER_TYPE = config.retriever_type
        updated_fields.append(f"retriever_type → {config.retriever_type}")

        if old_type != config.retriever_type:
            RetrieverFactory()._retriever = None
            logger.info("🔄 Cleared retriever cache")

    if config.retrieval_k is not None:
        rag_settings.RETRIEVAL_K = config.retrieval_k
        updated_fields.append(f"retrieval_k → {config.retrieval_k}")

    if config.fetch_k is not None:
        rag_settings.FETCH_K = config.fetch_k
        updated_fields.append(f"fetch_k → {config.fetch_k}")

    if config.lambda_mult is not None:
        rag_settings.LAMBDA_MULT = config.lambda_mult
        updated_fields.append(f"lambda_mult → {config.lambda_mult}")

    if config.similarity_threshold is not None:
        rag_settings.SIMILARITY_THRESHOLD = config.similarity_threshold
        updated_fields.append(f"similarity_threshold → {config.similarity_threshold}")

    # Generation Settings
    if config.llm_temperature is not None:
        rag_settings.LLM_TEMPERATURE = config.llm_temperature
        updated_fields.append(f"llm_temperature → {config.llm_temperature}")

    if config.max_tokens is not None:
        rag_settings.MAX_TOKENS = config.max_tokens
        updated_fields.append(f"max_tokens → {config.max_tokens}")

    # Document Processing
    if config.chunk_size is not None:
        rag_settings.CHUNK_SIZE = config.chunk_size
        updated_fields.append(f"chunk_size → {config.chunk_size}")

    if config.chunk_overlap is not None:
        rag_settings.CHUNK_OVERLAP = config.chunk_overlap
        updated_fields.append(f"chunk_overlap → {config.chunk_overlap}")

    logger.info(f"📝 RAG Config Updated: {len(updated_fields)} field(s)")
    for field in updated_fields:
        logger.info(f"   • {field}")

    return await get_rag_config()

@router.post("/upload_document", response_model=DocumentUploadResponse)
async def upload_document(
file: UploadFile = File(...),
user_id: int = Depends(get_current_user_id)
):
"""Upload and ingest PDF into vector store."""
if not file.filename.endswith('.pdf'):
raise HTTPException(status_code=400, detail="Only PDF files are supported")

    temp_path = Path(f"/tmp/{file.filename}")

    try:
        # Save uploaded file
        with open(temp_path, "wb") as buffer:
            shutil.copyfileobj(file.file, buffer)

        # Ingest into vector store
        await run_in_threadpool(ingest_pdf, str(temp_path))

        # Clear retriever cache
        RetrieverFactory()._retriever = None

        logger.info(f"✅ Uploaded and ingested: {file.filename}")

        return DocumentUploadResponse(
            message="Document uploaded and ingested successfully",
            filename=file.filename,
            success=True
        )

    except Exception as e:
        logger.error(f"❌ Document upload failed: {e}")
        raise HTTPException(status_code=500, detail=str(e))

    finally:
        if temp_path.exists():
            temp_path.unlink()

</file>

<file path="app/RAGsystem/schemas.py">
# app/RAGsystem/schemas.py

from typing import Any, Dict, List, Optional, Union

from pydantic import BaseModel, Field

# ============================================================================

# PYDANTIC OUTPUT SCHEMAS

# ============================================================================

class ClinicalRecommendationOutput(BaseModel):
"""Structured output schema for LLM recommendations."""

    diagnosis: str = Field(..., description="Primary clinical diagnosis as a single string")
    differential_diagnoses: list[str] = Field(
        default_factory=list, description="List of alternative diagnoses to consider"
    )
    recommended_management: str = Field(
        ..., description="Complete treatment plan as a single string with numbered steps"
    )
    reference_pages: list[int] = Field(
        default_factory=list,
        description="Page numbers from clinical guidelines (integers only). Only include pages that were actually used from the retrieved knowledge. Do NOT invent page numbers.",
    )

    # class Config:
    #     json_schema_extra = {
    #         "example": {
    #             "diagnosis": "Irreversible pulpitis with periapical abscess, tooth #{tooth_number}",
    #             "differential_diagnoses": ["Acute apical periodontitis", "Cracked tooth syndrome"],
    #             "recommended_management": "1. Emergency treatment: Pulpectomy or extraction; 2. Pain management with NSAIDs (Ibuprofen 400mg TID); 3. Antibiotics if systemic involvement (Amoxicillin 500mg TID for 7 days); 4. Referral to endodontist for definitive root canal therapy; 5. Follow-up radiograph in 3-6 months.",
    #             "reference_pages": [100, 101, 352],
    #         }
    #     }

class QuestionPayload(BaseModel):
question: str

class RAGResponse(BaseModel):
answer: Union[Dict[str, Any], str] # Can be dict or string
retrieval_strategy: str
llm_provider: Optional[str] = None
model_used: Optional[str] = None

class DocumentUploadResponse(BaseModel):
message: str
filename: str
success: bool

class PharmacologicalTreatment(BaseModel):
"""Standard pharmacological treatment structure"""

    analgesics: List[Dict[str, Any]] = Field(default_factory=list)
    antibiotics: List[Dict[str, Any]] = Field(default_factory=list)

class RecommendedManagement(BaseModel):
"""Standard management structure"""

    pharmacological: PharmacologicalTreatment
    non_pharmacological: List[Dict[str, Any]] = Field(default_factory=list)
    follow_up: str = "Not specified"

class StandardizedRAGAnswer(BaseModel):
"""Enforced standard structure for ALL RAG responses"""

    diagnosis: str = Field(..., description="Primary diagnosis")
    differential_diagnoses: List[str] = Field(
        default_factory=list, description="Alternative diagnoses to consider"
    )
    recommended_management: RecommendedManagement
    precautions: List[Dict[str, Any]] = Field(
        default_factory=list, description="Contraindications and warnings"
    )
    reference_pages: List[int] = Field(
        default_factory=list, description="Page numbers from clinical guidelines (REQUIRED)"
    )

    class Config:
        json_schema_extra = {
            "example": {
                "diagnosis": "Periapical abscess, tooth #47",
                "differential_diagnoses": [
                    "Acute apical periodontitis",
                    "Symptomatic irreversible pulpitis",
                ],
                "recommended_management": {
                    "pharmacological": {
                        "analgesics": [
                            {
                                "name": "Ibuprofen",
                                "dose": "400mg 8 hourly",
                                "reference_page": 44,
                                "type": "analgesic",
                            }
                        ],
                        "antibiotics": [
                            {
                                "name": "Amoxicillin",
                                "dose": "500mg 8 hourly for 5-7 days",
                                "reference_page": 45,
                                "type": "antibiotic",
                            }
                        ],
                    },
                    "non_pharmacological": [
                        {
                            "description": "Extraction of offending tooth under local anesthesia",
                            "category": "extraction",
                            "reference_page": 43,
                        },
                        {
                            "description": "Establish drainage with incision",
                            "category": "drainage",
                            "reference_page": 43,
                        },
                    ],
                    "follow_up": "Review in 24-48 hours if symptoms persist",
                },
                "precautions": [
                    {
                        "condition": "Pregnancy",
                        "note": "Adjust antibiotic - avoid tetracyclines",
                        "reference_page": 47,
                    }
                ],
                "reference_pages": [43, 44, 45, 47],
            }
        }

</file>

<file path="app/shared/cost_calculator.py">
# app/shared/cost_calculator.py
"""
Cost Calculator for Vision and LLM Models
Provides pricing estimates and cost comparison across all providers
Updated: February 2026
"""
from typing import Dict, List, Tuple
from dataclasses import dataclass

# ============================================================================

# PRICING DATA (per 1M tokens, USD)

# ============================================================================

@dataclass
class ModelPricing:
"""Pricing for a single model"""
input_price: float # Per 1M input tokens
output_price: float # Per 1M output tokens
notes: str = ""

# Vision Model Pricing (estimates for dental X-ray analysis)

# Note: Vision models typically charge per image + tokens

VISION_MODEL_PRICING = { # Premium Cloud (Proprietary)
"gpt-4o": ModelPricing(2.50, 10.00, "Best accuracy, expensive"),
"gpt4v": ModelPricing(2.50, 10.00, "Alias for GPT-4o"),
"gpt-4-vision-preview": ModelPricing(10.00, 30.00, "Legacy model"),
"claude-3-5-sonnet-20241022": ModelPricing(3.00, 15.00, "Excellent reasoning"),
"claude": ModelPricing(3.00, 15.00, "Alias for Claude 3.5 Sonnet"),
"claude-3-opus-20240229": ModelPricing(15.00, 75.00, "Highest capability"),

    # Fast Cloud (New additions)
    "llama-3.2-90b-vision-preview": ModelPricing(0.59, 0.79, "Groq - ultra-fast"),
    "llama-3.2-11b-vision-preview": ModelPricing(0.18, 0.18, "Groq - faster, cheaper"),
    "groq": ModelPricing(0.18, 0.18, "Alias for Groq Llama 3.2"),
    "gemini-2.5-flash": ModelPricing(0.075, 0.30, "Google - best value"),
    "gemini": ModelPricing(0.075, 0.30, "Alias for Gemini 2.5 Flash"),
    "gemini-1.5-pro": ModelPricing(1.25, 5.00, "Google - high capability"),
    "gemini-1.5-flash": ModelPricing(0.075, 0.30, "Google - fast"),

    # Local Models (Free)
    "llava:13b": ModelPricing(0.00, 0.00, "Local Ollama - best open-source"),
    "llava_med": ModelPricing(0.00, 0.00, "Local Ollama - medical specialist"),
    "llama3.2-vision": ModelPricing(0.00, 0.00, "Local Ollama - Meta latest"),
    "gemma3:12b": ModelPricing(0.00, 0.00, "Local Ollama - Google multimodal"),
    "gemma3:4b": ModelPricing(0.00, 0.00, "Local Ollama - lightweight"),
    "gemma3": ModelPricing(0.00, 0.00, "Local Ollama - gemma3 alias"),
    "biomedclip": ModelPricing(0.00, 0.00, "Local - pathology classifier"),

}

# LLM Pricing (text-only, for RAG recommendations)

LLM_MODEL_PRICING = { # Cloud - Premium
"gpt-4o": ModelPricing(2.50, 10.00, "OpenAI latest"),
"gpt-4-turbo-preview": ModelPricing(10.00, 30.00, "OpenAI legacy"),
"claude-3-5-sonnet-20241022": ModelPricing(3.00, 15.00, "Anthropic best"),

    # Cloud - Fast/Cheap (New)
    "llama-3.3-70b-versatile": ModelPricing(0.59, 0.79, "Groq - 70B fast"),
    "groq": ModelPricing(0.59, 0.79, "Alias for Groq Llama 3.3"),
    "mixtral-8x7b-32768": ModelPricing(0.24, 0.24, "Groq - MoE model"),
    "gemini-2.5-flash": ModelPricing(0.075, 0.30, "Google - best value"),
    "gemini": ModelPricing(0.075, 0.30, "Alias for Gemini 2.5 Flash"),
    "gemini-1.5-pro": ModelPricing(1.25, 5.00, "Google - 2M context"),

    # Local - Free
    "llama3.1:8b": ModelPricing(0.00, 0.00, "Local Ollama"),
    "mixtral:8x7b": ModelPricing(0.00, 0.00, "Local Ollama"),
    "gemma3:4b": ModelPricing(0.00, 0.00, "Local Ollama"),

}

# ============================================================================

# COST ESTIMATION FUNCTIONS

# ============================================================================

def estimate_vision_cost(
model: str,
num_images: int = 1,
avg_input_tokens: int = 1000,
avg_output_tokens: int = 500
) -> Dict[str, float]:
"""
Estimate cost for vision model inference.

    Args:
        model: Model identifier
        num_images: Number of images to analyze
        avg_input_tokens: Average input tokens (prompt + image)
        avg_output_tokens: Average output tokens

    Returns:
        Dict with cost breakdown
    """
    pricing = VISION_MODEL_PRICING.get(model)
    if not pricing:
        return {"error": f"Unknown model: {model}"}

    # Calculate per-request cost
    input_cost = (avg_input_tokens / 1_000_000) * pricing.input_price
    output_cost = (avg_output_tokens / 1_000_000) * pricing.output_price
    per_image_cost = input_cost + output_cost

    return {
        "model": model,
        "cost_per_image": round(per_image_cost, 6),
        "total_cost": round(per_image_cost * num_images, 4),
        "cost_per_1000_images": round(per_image_cost * 1000, 2),
        "pricing_notes": pricing.notes,
    }

def estimate_llm_cost(
model: str,
num_requests: int = 1,
avg_input_tokens: int = 2000,
avg_output_tokens: int = 500
) -> Dict[str, float]:
"""
Estimate cost for LLM text generation.

    Args:
        model: Model identifier
        num_requests: Number of requests
        avg_input_tokens: Average input tokens (context + prompt)
        avg_output_tokens: Average output tokens

    Returns:
        Dict with cost breakdown
    """
    pricing = LLM_MODEL_PRICING.get(model)
    if not pricing:
        return {"error": f"Unknown model: {model}"}

    input_cost = (avg_input_tokens / 1_000_000) * pricing.input_price
    output_cost = (avg_output_tokens / 1_000_000) * pricing.output_price
    per_request_cost = input_cost + output_cost

    return {
        "model": model,
        "cost_per_request": round(per_request_cost, 6),
        "total_cost": round(per_request_cost * num_requests, 4),
        "cost_per_1000_requests": round(per_request_cost * 1000, 2),
        "pricing_notes": pricing.notes,
    }

def compare_vision_models(
num_images: int = 1000,
avg_input_tokens: int = 1000,
avg_output_tokens: int = 500
) -> List[Dict]:
"""
Compare costs across all vision models.

    Returns:
        List of dicts sorted by cost (cheapest first)
    """
    comparisons = []

    for model, pricing in VISION_MODEL_PRICING.items():
        cost_data = estimate_vision_cost(
            model, num_images, avg_input_tokens, avg_output_tokens
        )
        comparisons.append(cost_data)

    # Sort by total cost
    return sorted(comparisons, key=lambda x: x.get("total_cost", float('inf')))

def compare_llm_models(
num_requests: int = 1000,
avg_input_tokens: int = 2000,
avg_output_tokens: int = 500
) -> List[Dict]:
"""
Compare costs across all LLM models.

    Returns:
        List of dicts sorted by cost (cheapest first)
    """
    comparisons = []

    for model, pricing in LLM_MODEL_PRICING.items():
        cost_data = estimate_llm_cost(
            model, num_requests, avg_input_tokens, avg_output_tokens
        )
        comparisons.append(cost_data)

    return sorted(comparisons, key=lambda x: x.get("total_cost", float('inf')))

def estimate_cdss_pipeline_cost(
vision_model: str,
llm_model: str,
num_cases: int = 100
) -> Dict:
"""
Estimate total cost for complete CDSS pipeline (vision + RAG + LLM).

    Assumptions:
    - Vision: 1000 input tokens (image + prompt), 500 output tokens
    - LLM: 2000 input tokens (context + retrieved docs), 500 output tokens

    Returns:
        Dict with cost breakdown
    """
    vision_cost_data = estimate_vision_cost(vision_model, num_cases, 1000, 500)
    llm_cost_data = estimate_llm_cost(llm_model, num_cases, 2000, 500)

    total_cost = vision_cost_data.get("total_cost", 0) + llm_cost_data.get("total_cost", 0)

    return {
        "pipeline": f"{vision_model} + {llm_model}",
        "num_cases": num_cases,
        "vision_cost": vision_cost_data.get("total_cost", 0),
        "llm_cost": llm_cost_data.get("total_cost", 0),
        "total_cost": round(total_cost, 4),
        "cost_per_case": round(total_cost / num_cases, 6) if num_cases > 0 else 0,
        "annual_cost_50_daily": round((total_cost / num_cases) * 50 * 365, 2) if num_cases > 0 else 0,
    }

def get_cheapest_options() -> Dict[str, str]:
"""
Get the cheapest model for each category.

    Returns:
        Dict mapping category to cheapest model
    """
    # Find cheapest non-zero cost models
    vision_costs = [
        (model, pricing.input_price + pricing.output_price)
        for model, pricing in VISION_MODEL_PRICING.items()
        if pricing.input_price > 0 or pricing.output_price > 0
    ]

    llm_costs = [
        (model, pricing.input_price + pricing.output_price)
        for model, pricing in LLM_MODEL_PRICING.items()
        if pricing.input_price > 0 or pricing.output_price > 0
    ]

    cheapest_vision_cloud = min(vision_costs, key=lambda x: x[1])[0] if vision_costs else None
    cheapest_llm_cloud = min(llm_costs, key=lambda x: x[1])[0] if llm_costs else None

    return {
        "cheapest_vision_cloud": cheapest_vision_cloud,
        "cheapest_llm_cloud": cheapest_llm_cloud,
        "cheapest_vision_local": "gemma3:4b",  # Smallest local multimodal
        "cheapest_llm_local": "gemma3:4b",     # Smallest local LLM
        "best_value_vision": "gemini-2.5-flash",  # Best accuracy/cost ratio
        "best_value_llm": "gemini-2.5-flash",     # Best accuracy/cost ratio
        "fastest_vision": "llama-3.2-11b-vision-preview",  # Groq 11B
        "fastest_llm": "llama-3.3-70b-versatile",  # Groq 70B
    }

</file>

<file path="app/shared/cost_routes.py">
# app/shared/cost_routes.py
"""
Cost Comparison API Endpoints
Provides pricing estimates for all vision and LLM models
"""
from fastapi import APIRouter
from app.shared.cost_calculator import (
    compare_vision_models,
    compare_llm_models,
    estimate_cdss_pipeline_cost,
    get_cheapest_options,
)

router = APIRouter(prefix="/cost", tags=["cost-analysis"])

@router.get("/vision-comparison")
async def get_vision_cost_comparison(num_images: int = 1000):
"""
Compare costs across all vision models.

    Returns pricing for local (free) and cloud models.

    Example: GET /api/cost/vision-comparison?num_images=1000
    """
    comparison = compare_vision_models(num_images)

    return {
        "num_images": num_images,
        "comparison": comparison,
        "summary": {
            "cheapest_cloud": comparison[0] if comparison and comparison[0]["cost_per_image"] > 0 else None,
            "most_expensive": comparison[-1] if comparison else None,
            "local_models": [m for m in comparison if m["cost_per_image"] == 0],
        },
        "assumptions": {
            "avg_input_tokens": 1000,
            "avg_output_tokens": 500,
            "note": "Costs are estimates. Actual costs may vary."
        }
    }

@router.get("/llm-comparison")
async def get_llm_cost_comparison(num_requests: int = 1000):
"""
Compare costs across all LLM models.

    Example: GET /api/cost/llm-comparison?num_requests=1000
    """
    comparison = compare_llm_models(num_requests)

    return {
        "num_requests": num_requests,
        "comparison": comparison,
        "summary": {
            "cheapest_cloud": comparison[0] if comparison and comparison[0]["cost_per_request"] > 0 else None,
            "most_expensive": comparison[-1] if comparison else None,
            "local_models": [m for m in comparison if m["cost_per_request"] == 0],
        },
        "assumptions": {
            "avg_input_tokens": 2000,
            "avg_output_tokens": 500,
            "note": "Costs are estimates. Actual costs may vary."
        }
    }

@router.get("/pipeline-estimate")
async def get_pipeline_cost_estimate(
vision_model: str = "llava:13b",
llm_model: str = "llama3.1:8b",
num_cases: int = 100
):
"""
Estimate cost for complete CDSS pipeline (vision + RAG + LLM).

    Example scenarios:
    - Ultra-low cost: vision_model=gemma3:4b, llm_model=gemma3:4b
    - Best value: vision_model=gemini-2.5-flash, llm_model=gemini-2.5-flash
    - Ultra-fast: vision_model=llama-3.2-11b-vision-preview, llm_model=llama-3.3-70b-versatile
    - Best accuracy: vision_model=gpt-4o, llm_model=gpt-4o

    Example: GET /api/cost/pipeline-estimate?vision_model=groq&llm_model=gemini&num_cases=100
    """
    estimate = estimate_cdss_pipeline_cost(vision_model, llm_model, num_cases)

    return {
        "estimate": estimate,
        "deployment_context": {
            "tanzania_rural_clinic": {
                "cases_per_day": 50,
                "annual_cases": 50 * 365,
                "annual_cost": estimate["cost_per_case"] * 50 * 365 if estimate.get("cost_per_case") else 0,
            },
            "urban_hospital": {
                "cases_per_day": 200,
                "annual_cases": 200 * 365,
                "annual_cost": estimate["cost_per_case"] * 200 * 365 if estimate.get("cost_per_case") else 0,
            }
        }
    }

@router.get("/recommendations")
async def get_cost_recommendations():
"""
Get recommended model combinations for different deployment scenarios.

    Returns:
    - Cheapest options (local)
    - Best value (cloud)
    - Fastest (cloud)
    - Best accuracy (cloud)
    """
    cheapest = get_cheapest_options()

    # Calculate actual costs for each scenario
    scenarios = {
        "ultra_low_cost": {
            "vision": "gemma3:4b",
            "llm": "gemma3:4b",
            "description": "Fully local, offline-capable",
            "cost_per_case": "$0.00",
            "annual_cost_50_daily": "$0",
            "use_case": "Rural clinics, privacy-critical, no internet",
            "speed": "Medium (40-60s per case)",
            "accuracy": "Good (85%)",
        },
        "best_value_cloud": {
            "vision": cheapest["best_value_vision"],
            "llm": cheapest["best_value_llm"],
            "description": "Google Gemini - best accuracy/cost ratio",
            "cost_per_case": "$0.0004",
            "annual_cost_50_daily": "$7.30",
            "use_case": "Connected clinics, high volume",
            "speed": "Fast (3-5s per case)",
            "accuracy": "Very Good (89%)",
        },
        "ultra_fast": {
            "vision": cheapest["fastest_vision"],
            "llm": cheapest["fastest_llm"],
            "description": "Groq LPU - 10x faster than GPU",
            "cost_per_case": "$0.002",
            "annual_cost_50_daily": "$36.50",
            "use_case": "Emergency triage, speed-critical",
            "speed": "Ultra-fast (2-3s per case)",
            "accuracy": "Very Good (87%)",
        },
        "best_accuracy": {
            "vision": "gpt-4o",
            "llm": "gpt-4o",
            "description": "OpenAI GPT-4o - highest accuracy",
            "cost_per_case": "$0.025",
            "annual_cost_50_daily": "$456.25",
            "use_case": "Tertiary hospitals, complex cases, second opinions",
            "speed": "Medium (8-12s per case)",
            "accuracy": "Excellent (92%)",
        },
        "hybrid_recommended": {
            "vision": "gemma3:4b",
            "llm": "gemini-2.5-flash",
            "description": "Local vision + cloud LLM (recommended for Tanzania)",
            "cost_per_case": "$0.0003",
            "annual_cost_50_daily": "$5.48",
            "use_case": "Offline X-ray analysis + online recommendations when available",
            "speed": "Mixed (60s local, 3s cloud when online)",
            "accuracy": "Very Good (88%)",
        }
    }

    return {
        "recommendations": scenarios,
        "model_catalog": cheapest,
        "notes": [
            "Costs are estimates based on February 2026 pricing",
            "Accuracy percentages are approximate based on test set performance",
            "Speed estimates assume typical periapical X-ray analysis",
            "Annual costs assume 50 cases/day, 365 days/year",
            "Local models (Gemma3, LLaVA) have $0 operating cost but require upfront hardware"
        ]
    }

@router.get("/tanzania-deployment")
async def get_tanzania_deployment_analysis():
"""
Specific cost analysis for Tanzania deployment context.

    Context:
    - Dentist-to-population ratio: 1:360,000 (vs WHO recommendation 1:7,500)
    - Limited internet connectivity in rural areas
    - Budget constraints
    - Need for offline capability
    """

    # Scenario 1: Rural clinic (offline, 20 cases/day)
    rural_scenario = estimate_cdss_pipeline_cost("gemma3:4b", "gemma3:4b", 20 * 365)

    # Scenario 2: District hospital (semi-connected, 50 cases/day)
    district_scenario = estimate_cdss_pipeline_cost("gemma3:4b", "gemini-2.5-flash", 50 * 365)

    # Scenario 3: Urban hospital (connected, 100 cases/day)
    urban_scenario = estimate_cdss_pipeline_cost("gemini-2.5-flash", "gemini-2.5-flash", 100 * 365)

    return {
        "context": {
            "dentist_ratio": "1:360,000",
            "who_recommendation": "1:7,500",
            "gap": "48x shortage",
            "internet_penetration": "~30% rural, ~80% urban"
        },
        "deployment_scenarios": {
            "rural_health_center": {
                "location": "Remote villages",
                "connectivity": "Offline or intermittent",
                "volume": "20 cases/day",
                "recommended_stack": "Gemma3 4B (local) for both vision and LLM",
                "hardware_requirements": "Mac Mini M4 or equivalent ($800 one-time)",
                "annual_opex": "$0",
                "total_first_year": "$800",
                "analysis": rural_scenario
            },
            "district_hospital": {
                "location": "Small towns",
                "connectivity": "Intermittent internet",
                "volume": "50 cases/day",
                "recommended_stack": "Gemma3 4B (vision, offline) + Gemini Flash (LLM, when online)",
                "hardware_requirements": "Mac Mini M4 ($800) + mobile data ($20/month)",
                "annual_opex": "$240 internet + $5 API = $245",
                "total_first_year": "$1,045",
                "analysis": district_scenario
            },
            "urban_hospital": {
                "location": "Dar es Salaam, Arusha",
                "connectivity": "Reliable broadband",
                "volume": "100 cases/day",
                "recommended_stack": "Gemini Flash (both vision and LLM)",
                "hardware_requirements": "Standard PC ($500)",
                "annual_opex": "$50/month internet + $15 API = $780",
                "total_first_year": "$1,280",
                "analysis": urban_scenario
            }
        },
        "comparison_baseline": {
            "gpt4_premium_stack": {
                "annual_cost_50_daily": "$456 API + $50 internet = $9,125",
                "note": "Not feasible for Tanzania context"
            },
            "human_dentist_alternative": {
                "annual_salary": "$12,000 - $24,000",
                "training_years": "5-7 years",
                "availability": "Severe shortage",
                "note": "CDSS augments, doesn't replace human expertise"
            }
        },
        "recommendation": {
            "tier_1_rural": "Gemma3 local ($800 hardware, $0/year opex)",
            "tier_2_district": "Hybrid Gemma3 + Gemini ($800 hardware, $245/year opex)",
            "tier_3_urban": "Gemini cloud ($500 hardware, $780/year opex)",
            "national_deployment_cost": "~$2M first year for 100 sites across all tiers"
        }
    }

</file>

<file path="app/shared/model_specs.py">
# app/shared/model_specs.py

"""
Complete Model Specifications Database
Used for thesis tables and deployment analysis
All data verified as of February 2026
"""
from dataclasses import dataclass
from typing import Optional, Dict

@dataclass
class ModelSpecs:
"""Complete specifications for a vision or LLM model"""
name: str
display_name: str
size_gb: Optional[float] # None for cloud models
params_billions: Optional[float]
requires_gpu: bool
min_ram_gb: int
min_vram_gb: Optional[int] # For GPU models
quantization: Optional[str]
provider: str # "local" or "cloud"
platform: str # "ollama", "huggingface", "openai", "anthropic", "groq", "google"
supports_vision: bool
context_window: Optional[int] # Tokens

# ============================================================================

# VISION MODELS SPECIFICATIONS

# ============================================================================

VISION_MODEL_SPECS: Dict[str, ModelSpecs] = { # ── Local Models (Ollama) ──
"llava:13b": ModelSpecs(
name="llava:13b",
display_name="LLaVA 13B",
size_gb=7.4,
params_billions=13.0,
requires_gpu=False,
min_ram_gb=16,
min_vram_gb=None,
quantization="Q4_0",
provider="local",
platform="ollama",
supports_vision=True,
context_window=4096
),

    "llava:latest": ModelSpecs(
        name="llava:latest",
        display_name="LLaVA 7B",
        size_gb=4.7,
        params_billions=7.0,
        requires_gpu=False,
        min_ram_gb=8,
        min_vram_gb=None,
        quantization="Q4_0",
        provider="local",
        platform="ollama",
        supports_vision=True,
        context_window=4096
    ),

    "llama3.2-vision": ModelSpecs(
        name="llama3.2-vision",
        display_name="Llama 3.2 Vision 11B",
        size_gb=7.9,
        params_billions=11.0,
        requires_gpu=False,
        min_ram_gb=16,
        min_vram_gb=None,
        quantization="Q4_0",
        provider="local",
        platform="ollama",
        supports_vision=True,
        context_window=128000
    ),

    "gemma3:4b": ModelSpecs(
        name="gemma3:4b",
        display_name="Gemma 3 4B",
        size_gb=2.5,
        params_billions=4.0,
        requires_gpu=False,
        min_ram_gb=8,
        min_vram_gb=None,
        quantization="Q4_0",
        provider="local",
        platform="ollama",
        supports_vision=True,
        context_window=8192
    ),

    "gemma3:12b": ModelSpecs(
        name="gemma3:12b",
        display_name="Gemma 3 12B",
        size_gb=7.0,
        params_billions=12.0,
        requires_gpu=False,
        min_ram_gb=16,
        min_vram_gb=None,
        quantization="Q4_0",
        provider="local",
        platform="ollama",
        supports_vision=True,
        context_window=8192
    ),

    # ── Local Models (HuggingFace) ──
    "biomedclip": ModelSpecs(
        name="biomedclip",
        display_name="BiomedCLIP",
        size_gb=0.5,
        params_billions=None,
        requires_gpu=False,
        min_ram_gb=4,
        min_vram_gb=None,
        quantization=None,
        provider="local",
        platform="huggingface",
        supports_vision=True,
        context_window=77
    ),

    "florence": ModelSpecs(
        name="florence",
        display_name="Florence-2 Large",
        size_gb=1.6,
        params_billions=0.77,
        requires_gpu=False,
        min_ram_gb=8,
        min_vram_gb=None,
        quantization=None,
        provider="local",
        platform="huggingface",
        supports_vision=True,
        context_window=1024
    ),

    # ── Cloud Models ──
    "gpt-4o": ModelSpecs(
        name="gpt-4o",
        display_name="GPT-4o Vision",
        size_gb=None,
        params_billions=None,  # Not disclosed
        requires_gpu=False,
        min_ram_gb=4,  # Only for API client
        min_vram_gb=None,
        quantization=None,
        provider="cloud",
        platform="openai",
        supports_vision=True,
        context_window=128000
    ),
    "gpt4v": ModelSpecs(
        name="gpt4v",
        display_name="GPT-4o Vision (Alias)",
        size_gb=None,
        params_billions=None,
        requires_gpu=False,
        min_ram_gb=4,
        min_vram_gb=None,
        quantization=None,
        provider="cloud",
        platform="openai",
        supports_vision=True,
        context_window=128000
    ),

    "claude-3-5-sonnet-20241022": ModelSpecs(
        name="claude-3-5-sonnet-20241022",
        display_name="Claude 3.5 Sonnet",
        size_gb=None,
        params_billions=None,
        requires_gpu=False,
        min_ram_gb=4,
        min_vram_gb=None,
        quantization=None,
        provider="cloud",
        platform="anthropic",
        supports_vision=True,
        context_window=200000
    ),
    "claude": ModelSpecs(
        name="claude",
        display_name="Claude 3.5 Sonnet (Alias)",
        size_gb=None,
        params_billions=None,
        requires_gpu=False,
        min_ram_gb=4,
        min_vram_gb=None,
        quantization=None,
        provider="cloud",
        platform="anthropic",
        supports_vision=True,
        context_window=200000
    ),

    "llama-3.2-11b-vision-preview": ModelSpecs(
        name="llama-3.2-11b-vision-preview",
        display_name="Groq Llama 3.2 11B Vision",
        size_gb=None,
        params_billions=11.0,
        requires_gpu=False,
        min_ram_gb=4,
        min_vram_gb=None,
        quantization=None,
        provider="cloud",
        platform="groq",
        supports_vision=True,
        context_window=8192
    ),

    "gemini-1.5-flash": ModelSpecs(
        name="gemini-1.5-flash",
        display_name="Gemini 1.5 Flash",
        size_gb=None,
        params_billions=None,
        requires_gpu=False,
        min_ram_gb=4,
        min_vram_gb=None,
        quantization=None,
        provider="cloud",
        platform="google",
        supports_vision=True,
        context_window=1000000
    ),
    "gemini": ModelSpecs(
        name="gemini",
        display_name="Gemini 2.5 Flash (Alias)",
        size_gb=None,
        params_billions=None,
        requires_gpu=False,
        min_ram_gb=4,
        min_vram_gb=None,
        quantization=None,
        provider="cloud",
        platform="google",
        supports_vision=True,
        context_window=1000000
    ),

}

# ============================================================================

# LLM MODELS SPECIFICATIONS (Text-only)

# ============================================================================

LLM_MODEL_SPECS: Dict[str, ModelSpecs] = { # ── Local ──
"llama3.1:8b": ModelSpecs(
name="llama3.1:8b",
display_name="Llama 3.1 8B",
size_gb=4.7,
params_billions=8.0,
requires_gpu=False,
min_ram_gb=8,
min_vram_gb=None,
quantization="Q4_0",
provider="local",
platform="ollama",
supports_vision=False,
context_window=128000
),

    "mixtral:8x7b": ModelSpecs(
        name="mixtral:8x7b",
        display_name="Mixtral 8x7B",
        size_gb=26.0,
        params_billions=46.7,
        requires_gpu=False,
        min_ram_gb=32,
        min_vram_gb=None,
        quantization="Q4_0",
        provider="local",
        platform="ollama",
        supports_vision=False,
        context_window=32000
    ),

    "llama-3.3-70b-versatile": ModelSpecs(
        name="llama-3.3-70b-versatile",
        display_name="Groq Llama 3.3 70B",
        size_gb=None,
        params_billions=70.0,
        requires_gpu=False,
        min_ram_gb=4,
        min_vram_gb=None,
        quantization=None,
        provider="cloud",
        platform="groq",
        supports_vision=False,
        context_window=8192
    ),
    "gemma3": ModelSpecs(
        name="gemma3",
        display_name="Gemma 3 4B",
        size_gb=2.5,
        params_billions=4.0,
        requires_gpu=False,
        min_ram_gb=8,
        min_vram_gb=None,
        quantization="Q4_0",
        provider="local",
        platform="ollama",
        supports_vision=True,
        context_window=8192
    ),

    "llava_med": ModelSpecs(
        name="llava_med",
        display_name="LLaVA-Med 13B",
        size_gb=7.4,
        params_billions=13.0,
        requires_gpu=False,
        min_ram_gb=16,
        min_vram_gb=None,
        quantization="Q4_0",
        provider="local",
        platform="ollama",
        supports_vision=True,
        context_window=4096
    ),

    "groq": ModelSpecs(
        name="groq",
        display_name="Groq Llama 3.2 11B",
        size_gb=None,  # Cloud model
        params_billions=11.0,
        requires_gpu=False,
        min_ram_gb=4,
        min_vram_gb=None,
        quantization=None,
        provider="cloud",
        platform="groq",  # ← This makes it show as "Groq"!
        supports_vision=True,
        context_window=8192
    ),

}

# ============================================================================

# PRICING DATABASE (per 1M tokens + per image)

# ============================================================================

MODEL_PRICING = { # Vision models - Premium
"gpt-4o": {
"input_per_1m": 2.50,
"output_per_1m": 10.00,
"per_image": 0.00765, # 1024x1024 image
"currency": "USD"
},
"gpt4v": {
"input_per_1m": 2.50,
"output_per_1m": 10.00,
"per_image": 0.00765,
"currency": "USD"
},
"claude-3-5-sonnet-20241022": {
"input_per_1m": 3.00,
"output_per_1m": 15.00,
"per_image": 0.012,
"currency": "USD"
},
"claude-3-haiku-20240307": {
"input_per_1m": 0.25,
"output_per_1m": 1.25,
"per_image": 0.0,
"currency": "USD"
},
"claude": {
"input_per_1m": 3.00,
"output_per_1m": 15.00,
"per_image": 0.012,
"currency": "USD"
},

    # Vision models - Fast/Cheap
    "llama-3.2-11b-vision-preview": {
        "input_per_1m": 0.18,
        "output_per_1m": 0.18,
        "per_image": 0.0,
        "currency": "USD"
    },
    "meta-llama/llama-4-scout-17b-16e-instruct": {
        "input_per_1m": 0.10,
        "output_per_1m": 0.10,
        "per_image": 0.0,
        "currency": "USD"
    },
    "groq": {
        "input_per_1m": 0.18,
        "output_per_1m": 0.18,
        "per_image": 0.0,
        "currency": "USD"
    },

    "gemini-1.5-flash": {
        "input_per_1m": 0.075,
        "output_per_1m": 0.30,
        "per_image": 0.0,
        "currency": "USD"
    },
    "gemini-2.5-flash": {
        "input_per_1m": 0.075,
        "output_per_1m": 0.30,
        "per_image": 0.0,
        "currency": "USD"
    },
    "gemini": {
        "input_per_1m": 0.075,
        "output_per_1m": 0.30,
        "per_image": 0.0,
        "currency": "USD"
    },

    # LLM models
    "llama-3.3-70b-versatile": {
        "input_per_1m": 0.59,
        "output_per_1m": 0.79,
        "per_image": 0.0,
        "currency": "USD"
    },

    # Local models (all free)
    **{
        model: {"input_per_1m": 0.0, "output_per_1m": 0.0, "per_image": 0.0, "currency": "USD"}
        for model in [
            "llava:13b", "llava:latest", "llama3.2-vision", "gemma3:4b", "gemma3:12b",
            "biomedclip", "florence", "llama3.1:8b", "mixtral:8x7b", "gemma3", "llava_med"
        ]
    }

}

# ============================================================================

# HARDWARE REQUIREMENTS TIERS

# ============================================================================

HARDWARE_TIERS = {
"minimal": {
"description": "Entry-level laptop",
"ram_gb": 8,
"storage_gb": 50,
"suitable_models": ["llava:latest", "gemma3:4b", "biomedclip"],
"estimated_cost_usd": 500
},
"recommended": {
"description": "Modern workstation (Mac Mini M4, etc.)",
"ram_gb": 16,
"storage_gb": 100,
"suitable_models": [
"llava:13b", "llama3.2-vision", "gemma3:12b", "llama3.1:8b"
],
"estimated_cost_usd": 800
},
"high_performance": {
"description": "Server-grade hardware",
"ram_gb": 32,
"storage_gb": 500,
"suitable_models": ["mixtral:8x7b", "all_models"],
"estimated_cost_usd": 2000
},
"cloud": {
"description": "API-based, any device",
"ram_gb": 4,
"storage_gb": 10,
"suitable_models": [
"gpt-4o", "claude-3-5-sonnet-20241022",
"llama-3.2-11b-vision-preview", "gemini-1.5-flash"
],
"estimated_cost_usd": 300
}
}

# ============================================================================

# HELPER FUNCTIONS

# ============================================================================

def get_model_spec(model_name: str) -> Optional[ModelSpecs]:
"""Get specifications for a model"""
return VISION_MODEL_SPECS.get(model_name) or LLM_MODEL_SPECS.get(model_name)

def get_hardware_tier_for_model(model_name: str) -> str:
"""Determine minimum hardware tier needed"""
spec = get_model_spec(model_name)
if not spec:
return "unknown"

    if spec.provider == "cloud":
        return "cloud"
    elif spec.min_ram_gb >= 32:
        return "high_performance"
    elif spec.min_ram_gb >= 16:
        return "recommended"
    else:
        return "minimal"

def calculate_cost(
model_name: str,
input_tokens: int,
output_tokens: int,
num_images: int = 1
) -> float:
"""Calculate actual cost for a request"""
pricing = MODEL_PRICING.get(model_name, {})

    input_cost = (input_tokens / 1_000_000) * pricing.get("input_per_1m", 0)
    output_cost = (output_tokens / 1_000_000) * pricing.get("output_per_1m", 0)
    image_cost = num_images * pricing.get("per_image", 0)

    return input_cost + output_cost + image_cost

def get_deployment_category(model_name: str) -> str:
"""Categorize model for deployment recommendations"""
spec = get_model_spec(model_name)
if not spec:
return "unknown"

    if spec.provider == "local":
        if spec.size_gb and spec.size_gb < 5:
            return "ultra_low_cost_local"
        else:
            return "standard_local"
    else:
        # Cloud models
        pricing = MODEL_PRICING.get(model_name, {})
        typical_cost = calculate_cost(model_name, 1000, 500, 1)

        if typical_cost < 0.001:
            return "ultra_fast_cloud"
        elif typical_cost < 0.01:
            return "balanced_cloud"
        else:
            return "premium_cloud"

</file>

<file path="app/shared/performance_tracker.py">
# app/shared/performance_tracker.py

"""
Performance Tracking System
Collects REAL metrics during vision/LLM tests for thesis analysis
"""
import json
import time
import psutil
import os
from datetime import datetime
from typing import Dict, List, Optional
from dataclasses import dataclass, asdict
from pathlib import Path

from app.shared.model_specs import get_hardware_tier_for_model, get_model_spec, calculate_cost, MODEL_PRICING

@dataclass
class PerformanceMetrics:
"""
Performance metrics for ML/Software Engineering thesis.

    Focus: System performance, resource efficiency, cost economics.
    NOT clinical details (those belong in medical evaluation).
    """
    # Identification & Core Metrics (Non-Default)
    model_name: str
    test_id: str
    timestamp: str
    inference_time_ms: float
    peak_ram_mb: float

    # ══════════════════════════════════════════════════════════
    # SYSTEM PERFORMANCE METRICS (Core SE/ML) - With Defaults
    # ══════════════════════════════════════════════════════════

    # Latency (critical for real-time systems)
    time_to_first_token_ms: Optional[float] = None  # For streaming models
    tokens_per_second: Optional[float] = None

    # Throughput (scalability)
    concurrent_requests: int = 1
    queue_wait_time_ms: Optional[float] = None

    # ══════════════════════════════════════════════════════════
    # RESOURCE EFFICIENCY (Deployment Planning)
    # ══════════════════════════════════════════════════════════

    # Memory usage
    avg_ram_mb: Optional[float] = None
    gpu_memory_mb: Optional[float] = None

    # CPU/GPU utilization
    avg_cpu_percent: Optional[float] = None
    avg_gpu_percent: Optional[float] = None

    # Storage
    model_size_gb: Optional[float] = None
    cache_size_mb: Optional[float] = None

    # ══════════════════════════════════════════════════════════
    # COST ECONOMICS (TCO Analysis)
    # ══════════════════════════════════════════════════════════

    # Token counts (for cost calculation)
    input_tokens: Optional[int] = None
    output_tokens: Optional[int] = None
    total_tokens: Optional[int] = None

    # Actual costs
    cost_usd: Optional[float] = None
    cost_per_token: Optional[float] = None

    # ══════════════════════════════════════════════════════════
    # QUALITY METRICS (ML Model Performance)
    # ══════════════════════════════════════════════════════════

    # Response quality
    response_complete: bool = True  # All required fields present
    schema_valid: bool = True  # Matches expected schema
    citation_count: int = 0  # Number of reference pages cited
    avg_citation_quality: Optional[float] = None  # 0-1 score

    # Model confidence (if available)
    confidence_score: Optional[float] = None
    image_quality_score: Optional[float] = None  # 0-1 score for image quality

    # Clinical Summary (for qualitative comparison)
    primary_finding: Optional[str] = None
    teeth_visible: Optional[List[str]] = None

    # ══════════════════════════════════════════════════════════
    # RELIABILITY METRICS (System Stability)
    # ══════════════════════════════════════════════════════════

    # Success/failure tracking
    request_success: bool = True
    error_type: Optional[str] = None
    error_message: Optional[str] = None
    retry_count: int = 0

    # ══════════════════════════════════════════════════════════
    # METADATA (Test Context)
    # ══════════════════════════════════════════════════════════

    # Test configuration
    test_type: str = "vision_analysis"  # vision_analysis, rag_query, full_cdss
    model_provider: str = "local"  # local, cloud
    hardware_tier: str = "recommended"  # minimal, recommended, high_performance
    context_provided: Optional[bool] = None
    tooth_number_provided: Optional[bool] = None

class PerformanceTracker:
"""Tracks and persists performance metrics"""

    def __init__(self, results_file: str = "evaluation_results.json"):
        self.results_file = Path(results_file)
        self.results: List[PerformanceMetrics] = []
        self._load_existing()

    def _load_existing(self):
        """Load existing results from file"""
        if self.results_file.exists():
            try:
                with open(self.results_file, 'r') as f:
                    data = json.load(f)
                    self.results = [
                        PerformanceMetrics(**item) for item in data
                    ]
            except Exception as e:
                print(f"Warning: Could not load existing results: {e}")
                self.results = []

    def record(
        self,
        model_name: str,
        inference_time_ms: float,
        # Quality metrics
        response_complete: bool = True,
        schema_valid: bool = True,
        citation_count: int = 0,
        confidence_score: Optional[float] = None,
        image_quality_score: Optional[float] = None,
        avg_citation_quality: Optional[float] = None,
        # Clinical Summary (for qualitative comparison)
        primary_finding: Optional[str] = None,
        teeth_visible: Optional[List[str]] = None,
        # Cost metrics
        input_tokens: Optional[int] = None,
        output_tokens: Optional[int] = None,
        cost_usd: Optional[float] = None,
        # System metrics
        tokens_per_second: Optional[float] = None,
        time_to_first_token_ms: Optional[float] = None,
        # Reliability
        request_success: bool = True,
        error_type: Optional[str] = None,
        error_message: Optional[str] = None,
        # Metadata
        test_type: str = "vision_analysis",
        model_provider: str = "local",
        context_provided: Optional[bool] = None,
        tooth_number_provided: Optional[bool] = None
    ):
        """
        Record performance metrics.

        Focus on:
        - System performance ✅
        - Resource efficiency ✅
        - Cost economics ✅
        - Quality/reliability ✅
        """
        # Get system metrics
        process = psutil.Process(os.getpid())
        peak_ram_mb = process.memory_info().rss / 1024 / 1024
        avg_cpu = process.cpu_percent(interval=0.1)

        # Calculate derived metrics
        if input_tokens and output_tokens:
            total_tokens = input_tokens + output_tokens
            cost_usd = cost_usd or calculate_cost(model_name, input_tokens, output_tokens)
            cost_per_token = cost_usd / total_tokens if total_tokens > 0 else None
        elif output_tokens:
            # Handle cases where only output tokens are known (common for local models)
            total_tokens = output_tokens
            cost_usd = cost_usd or calculate_cost(model_name, 0, output_tokens)
            cost_per_token = cost_usd / total_tokens if total_tokens > 0 else None
        else:
            total_tokens = None
            cost_per_token = None

        if output_tokens and inference_time_ms > 0:
            tokens_per_second = (output_tokens / inference_time_ms) * 1000

        # Get model specs
        spec = get_model_spec(model_name)
        model_size_gb = spec.size_gb if spec else None
        hardware_tier = get_hardware_tier_for_model(model_name)

        # Create metric
        metric = PerformanceMetrics(
            model_name=model_name,
            test_id=f"{model_name}_{datetime.now().strftime('%Y%m%d_%H%M%S')}",
            timestamp=datetime.now().isoformat(),
            # Performance
            inference_time_ms=inference_time_ms,
            tokens_per_second=tokens_per_second,
            time_to_first_token_ms=time_to_first_token_ms,
            # Resources
            peak_ram_mb=peak_ram_mb,
            avg_cpu_percent=avg_cpu,
            model_size_gb=model_size_gb,
            # Cost
            input_tokens=input_tokens,
            output_tokens=output_tokens,
            total_tokens=total_tokens,
            cost_usd=cost_usd,
            cost_per_token=cost_per_token,
            # Quality
            response_complete=response_complete,
            schema_valid=schema_valid,
            citation_count=citation_count,
            avg_citation_quality=avg_citation_quality,
            confidence_score=confidence_score,
            image_quality_score=image_quality_score,
            primary_finding=primary_finding,
            teeth_visible=teeth_visible,
            # Reliability
            request_success=request_success,
            error_type=error_type,
            error_message=error_message,
            # Metadata
            test_type=test_type,
            model_provider=model_provider,
            hardware_tier=hardware_tier,
            context_provided=context_provided,
            tooth_number_provided=tooth_number_provided
        )

        self.results.append(metric)
        self._save()

    def _save(self):
        """Save results to JSON file"""
        with open(self.results_file, 'w') as f:
            json.dump(
                [asdict(r) for r in self.results],
                f,
                indent=2
            )

    def get_stats(self, model_name: str) -> Dict:
        """Get aggregated statistics for a model"""
        model_results = [r for r in self.results if r.model_name == model_name]

        if not model_results:
            return {
                "model": model_name,
                "num_tests": 0,
                "status": "no_data"
            }

        times = [r.inference_time_ms for r in model_results]
        costs = [r.cost_usd for r in model_results if r.cost_usd is not None]
        confidences = [r.confidence_score for r in model_results if r.confidence_score]

        return {
            "model": model_name,
            "num_tests": len(model_results),
            "speed": {
                "avg_ms": sum(times) / len(times),
                "min_ms": min(times),
                "max_ms": max(times),
                "avg_sec": sum(times) / len(times) / 1000
            },
            "cost": {
                "avg_usd": sum(costs) / len(costs) if costs else 0.0,
                "total_usd": sum(costs) if costs else 0.0
            },
            "quality": {
                "avg_confidence": sum(confidences) / len(confidences) if confidences else None
            }
        }

    def get_all_stats(self) -> List[Dict]:
        """Get stats for all tested models"""
        unique_models = set(r.model_name for r in self.results)
        return [self.get_stats(model) for model in unique_models]

    def clear(self):
        """Clear all results (use for fresh evaluation run)"""
        self.results = []
        self._save()

    def export_csv(self, filepath: str = "evaluation_results.csv"):
        """Export results to CSV for analysis"""
        import csv

        with open(filepath, 'w', newline='') as f:
            if not self.results:
                return

            writer = csv.DictWriter(f, fieldnames=asdict(self.results[0]).keys())
            writer.writeheader()
            for result in self.results:
                writer.writerow(asdict(result))

    def get_comparison_data(self) -> Dict:
        """
        Get data formatted for thesis comparison tables.

        Returns dict with all stats + model specs combined.
        """
        from app.shared.model_specs import get_model_spec, get_hardware_tier_for_model

        comparison = []

        for model_name in set(r.model_name for r in self.results):
            stats = self.get_stats(model_name)
            spec = get_model_spec(model_name)

            if not spec:
                continue

            # Calculate typical cost for comparison
            pricing = MODEL_PRICING.get(model_name, {})
            typical_cost = calculate_cost(model_name, 1000, 500, 1)

            comparison.append({
                "model": spec.display_name,
                "provider": spec.provider,
                "platform": spec.platform,
                "size_gb": spec.size_gb if spec.size_gb else "Cloud",
                "params_b": spec.params_billions if spec.params_billions else "N/A",
                "min_ram_gb": spec.min_ram_gb,
                "hardware_tier": get_hardware_tier_for_model(model_name),
                "avg_speed_sec": round(stats["speed"]["avg_sec"], 1),
                "cost_per_analysis": typical_cost,
                "cost_per_1000": typical_cost * 1000,
                "annual_cost_50_daily": typical_cost * 50 * 365,
                "avg_confidence": stats["quality"]["avg_confidence"],
                "num_tests": stats["num_tests"]
            })

        # Sort by cost (free first, then ascending)
        comparison.sort(key=lambda x: (
            0 if x["cost_per_analysis"] == 0 else 1,
            x["cost_per_analysis"]
        ))

        return {
            "models": comparison,
            "summary": {
                "total_models_tested": len(comparison),
                "local_models": len([m for m in comparison if m["provider"] == "local"]),
                "cloud_models": len([m for m in comparison if m["provider"] == "cloud"]),
                "total_tests_run": sum(m["num_tests"] for m in comparison)
            }
        }

# Global tracker instance

performance_tracker = PerformanceTracker()
</file>

<file path="app/visionsystem/gemini_vision_client.py">
# app/visionsystem/gemini_vision_client.py
"""
Google Gemini Vision Client - UPDATED for new google.genai SDK
"""
import logging

from PIL import Image

logger = logging.getLogger(**name**)

class GeminiVisionClient:
"""Google Gemini client using NEW SDK (google.genai)"""

    _instance = None
    _client = None
    _model_id = None
    _load_attempted = False

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def _lazy_load_client(self):
        """Initialize with NEW google.genai SDK"""
        if self._load_attempted:
            return
        self._load_attempted = True

        try:
            import os

            from config.visionconfig import vision_settings

            api_key = vision_settings.GEMINI_API_KEY or os.getenv("GEMINI_API_KEY")
            if not api_key:
                logger.warning("⚠️  GEMINI_API_KEY not set")
                return

            # NEW SDK import
            from google import genai

            self._client = genai.Client(api_key=api_key)
            self._model_id = vision_settings.GEMINI_VISION_MODEL

            logger.info(f"✅ Gemini client initialized: {self._model_id}")

        except ImportError:
            logger.error("❌ google-genai not installed. Run: pip install google-genai")
        except Exception as e:
            logger.error(f"❌ Gemini failed: {e}")

    def get_model_info(self) -> dict:
        """Get model metadata for health checks"""
        from config.visionconfig import vision_settings
        self._lazy_load_client()

        return {
            "provider": "gemini",
            "model": self._model_id or vision_settings.GEMINI_VISION_MODEL,
            "status": "ready" if self._client else "error",
            "api_type": "google.genai",
            "reliable": True if self._client else False
        }

    def analyze_image(self, image: Image.Image, prompt: str) -> dict:
        """Analyze with NEW SDK"""
        from config.visionconfig import vision_settings

        self._lazy_load_client()

        if not self._client:
            raise RuntimeError("Gemini client not initialized")

        logger.info(f"🌟 Analyzing with Gemini {self._model_id}...")

        try:
            import base64
            from io import BytesIO

            # Convert PIL to base64
            buffered = BytesIO()
            image.save(buffered, format="PNG")
            img_base64 = base64.b64encode(buffered.getvalue()).decode()

            # NEW SDK API
            response = self._client.models.generate_content(
                model=self._model_id,
                contents=[prompt, {"inline_data": {"mime_type": "image/png", "data": img_base64}}],
                config={
                    "response_mime_type": "application/json",
                }
            )

            result = response.text
            usage = response.usage_metadata
            logger.info(f"✅ Gemini complete: {len(result)} chars")
            if usage:
                logger.info(f"   Token usage: {usage.prompt_token_count} prompt, {usage.candidates_token_count} completion")
                return {
                    "text": result,
                    "input_tokens": usage.prompt_token_count,
                    "output_tokens": usage.candidates_token_count,
                }
            else:
                return {"text": result, "input_tokens": None, "output_tokens": len(result.split())}

        except Exception as e:
            logger.error(f"❌ Gemini failed: {e}")
            raise RuntimeError(f"Gemini error: {e}")

gemini_vision_client = GeminiVisionClient()
</file>

<file path="app/visionsystem/gemma3_client.py">
# app/visionsystem/gemma3_client.py
"""
Gemma 3 Vision Client - Google's latest multimodal model via Ollama
"""
import base64
from io import BytesIO
from PIL import Image
import ollama
import logging
from config.visionconfig import vision_settings

logger = logging.getLogger(**name**)

class Gemma3VisionClient:
"""Gemma 3 client optimized for local dental radiograph analysis via Ollama."""

    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def _get_model_name(self):
        """Get Gemma 3 model name from settings."""
        return vision_settings.GEMMA3_MODEL

    def _encode_image(self, image: Image.Image) -> str:
        """Convert PIL Image to base64 for Ollama."""
        buffer = BytesIO()
        image.save(buffer, format="PNG")
        return base64.b64encode(buffer.getvalue()).decode()

    def analyze_image(self, image: Image.Image, prompt: str) -> dict:
        """
        Analyze dental image using Gemma 3.
        """
        model_name = self._get_model_name()
        b64_image = self._encode_image(image)

        logger.info(f"🌟 Analyzing with Gemma 3 ({model_name}) via Ollama...")

        try:
            response = ollama.chat(
                model=model_name,
                format="json",
                messages=[
                    {
                        "role": "system",
                        "content": "You are an expert dental radiologist. Provide precise, clinical analysis of dental X-rays."
                    },
                    {
                        "role": "user",
                        "content": prompt,
                        "images": [b64_image]
                    }
                ],
                options={
                    "temperature": vision_settings.VISION_TEMPERATURE,
                    "num_predict": vision_settings.VISION_MAX_TOKENS,
                }
            )

            result = response["message"]["content"]
            logger.info(f"✅ Gemma 3 analysis complete: {len(result)} chars")

            # Extract token counts
            input_tokens = response.get("prompt_eval_count")
            output_tokens = response.get("eval_count")
            logger.info(f"   Token usage: {input_tokens} prompt, {output_tokens} completion")

            return {
                "text": result,
                "input_tokens": input_tokens,
                "output_tokens": output_tokens,
            }

        except Exception as e:
            logger.error(f"❌ Gemma 3 analysis failed: {e}")
            raise

# Global instance

gemma3_vision_client = Gemma3VisionClient()
</file>

<file path="app/visionsystem/llava_med_client.py">
# app/visionsystem/llava_med_client.py
"""
LLaVA-Med Client - FIXED
The microsoft/llava-med-v1.5-mistral-7b model uses a custom architecture
(llava_mistral) that is not supported by the current transformers library.

SOLUTION: Route LLaVA-Med requests through Ollama using the llava:13b model
with a medical specialist system prompt. This is functionally equivalent for
our use case (dental radiograph analysis) and avoids the transformers issue.

When transformers adds support for llava_mistral in a future release,
restore the HuggingFace code from the comment block below.
"""
from config.ragconfig import rag_settings
import logging
from PIL import Image
import ollama

from config.visionconfig import vision_settings

logger = logging.getLogger(**name**)

class LlavaMedClient:
"""
LLaVA-Med client - currently routing through Ollama with medical prompt.

    The original HuggingFace implementation fails because microsoft/llava-med-v1.5-mistral-7b
    uses model_type='llava_mistral' which is not registered in the current transformers.

    This workaround uses llava:13b (already downloaded) with a medical specialist
    system prompt that mimics LLaVA-Med's medical focus.
    """

    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def _get_model_name(self) -> str:
        """ Return the current vision model name for Ollama routing """
        return vision_settings.LLAVA_MODEL

    def analyze_image(self, image: Image.Image, prompt: str) -> dict:
        """
        Analyze image using llava:13b with medical specialist system prompt.
        This approximates LLaVA-Med behavior using available local models.
        """
        import base64
        from io import BytesIO
        from PIL import ImageEnhance

        model_name = self._get_model_name()
        logger.info(f"🔬 LLaVA-Med (via {model_name} with medical prompt)...")

        # Enhance contrast for better X-ray visibility
        enhancer = ImageEnhance.Contrast(image)
        enhanced = enhancer.enhance(1.5)

        # Convert to base64
        buffer = BytesIO()
        enhanced.save(buffer, format="PNG")
        b64_image = base64.b64encode(buffer.getvalue()).decode()

        # Medical specialist system prompt that approximates LLaVA-Med
        medical_system_prompt = """You are LLaVA-Med, a medical vision-language model specialized in clinical image analysis.

You have been fine-tuned on medical datasets including dental radiographs, CT scans, and clinical photography.
Your role is to provide expert medical image interpretation with clinical precision.

For dental radiographs specifically, you excel at:

- Identifying caries (cavities) as radiolucent (dark) areas in tooth structure
- Detecting periapical pathology (abscesses, granulomas) as dark halos around root tips
- Assessing bone levels and identifying periodontal bone loss
- Recognizing root canal treatments (radiopaque fills in root canals)
- Identifying restorations (amalgam, composite, crowns)
- Counting and identifying all teeth visible in the radiograph

You always provide structured, clinically actionable findings."""

        try:
            response = ollama.chat(
                model=model_name,
                messages=[
                    {
                        "role": "system",
                        "content": medical_system_prompt
                    },
                    {
                        "role": "user",
                        "content": prompt,
                        "images": [b64_image]
                    }
                ],
                options={
                    "temperature": 0.1,  # Low temp for consistent medical output
                    "top_p": 0.9,
                    "num_predict": vision_settings.VISION_MAX_TOKENS,
                }
            )

            result = response["message"]["content"]
            logger.info(f"✅ LLaVA-Med response: {len(result)} chars")

            # Extract token counts
            input_tokens = response.get("prompt_eval_count")
            output_tokens = response.get("eval_count")
            logger.info(f"   Token usage: {input_tokens} prompt, {output_tokens} completion")

            return {
                "text": result,
                "input_tokens": input_tokens,
                "output_tokens": output_tokens,
            }

        except Exception as e:
            logger.error(f"❌ LLaVA-Med (Ollama) failed: {e}")
            raise

    def analyze_clinical_image(self, image: Image.Image) -> dict:
        """Backward compatibility."""
        try:
            analysis_result = self.analyze_image(image, "Analyze this dental radiograph.")
            detailed = analysis_result["text"]
            input_tokens = analysis_result.get("input_tokens", 0)
            output_tokens = analysis_result.get("output_tokens", 0)

            return {
                "detailed_description": detailed,
                "region_findings": "See detailed description",
                "model": f"llava_med_via_{self._get_model_name()}",
                "input_tokens": input_tokens,
                "output_tokens": output_tokens
            }
        except Exception as e:
            return {
                "detailed_description": f"Error: {e}",
                "region_findings": "Analysis failed",
                "model": "llava_med",
            }

# Global instance

llava_med_client = LlavaMedClient()
</file>

<file path="app/visionsystem/routes.py">
# app/visionsystem/routes.py
"""
Vision System Routes
1. llava:7b -> llava:latest (actual installed model name from ollama list)
2. BiomedCLIP properly loads via open_clip (open-clip-torch is installed)
3. LLaVA-Med routes through Ollama with medical system prompt
4. GPT-4V and Claude gracefully report billing errors without crashing
"""
import logging
import time
from typing import Optional

from fastapi import APIRouter, Depends, File, Form, HTTPException, UploadFile

from app.visionsystem.image_processor import ImageProcessor
from app.visionsystem.vision_client import vision_client
from app.visionsystem.vision_schemas import VisionAnalysisResponse
from config.config_schemas import VisionConfigRequest, VisionConfigResponse
from config.visionconfig import vision_settings
from app.users.auth_dependencies import get_current_user_id
from app.shared.performance_tracker import performance_tracker

router = APIRouter()
logger = logging.getLogger(**name**)

@router.post("/analyze_image", response_model=VisionAnalysisResponse)
async def analyze_dental_radiograph(
file: UploadFile = File(...),
context: str = Form(None),
tooth_number: str = Form(None),
user_id: int = Depends(get_current_user_id)
):
"""Analyze a single dental radiograph using the configured vision model."""
logger.info(f"\n{'='*70}")
logger.info(f"SINGLE MODEL VISION ANALYSIS")
logger.info(f"{'='*70}")
logger.info(f"🔍 Vision Model Provider: {vision_settings.VISION_MODEL_PROVIDER}")
logger.info(f"🔍 Vision Model: {vision_settings.current_vision_model}")
if context:
logger.info(f"📝 Chief Complaint: {context}")
if tooth_number:
logger.info(f"🦷 Focus Tooth: #{tooth_number}")

    content = await file.read()
    image = ImageProcessor.preprocess_image(content)

    start_time = time.time()
    result = vision_client.analyze_dental_radiograph(
        image, context=context, tooth_number=tooth_number
    )
    elapsed = time.time() - start_time
    logger.info(f"✅ Analysis complete in {elapsed:.2f}s")

    return VisionAnalysisResponse(
        detailed_description=result["detailed_description"],
        pathology_summary=result.get("pathology_summary", ""),
        model_used=f"{vision_settings.VISION_MODEL_PROVIDER} - {vision_settings.current_vision_model}",
        processing_time_ms=elapsed * 1000,
        focused_tooth=tooth_number,
        image_quality_score=result.get("image_quality_score", 0.5),
        diagnostic_confidence=result.get("confidence_score", 0.5),
        structured_findings=result.get("structured_findings"),
        probabilities=result.get("structured_findings", {}).get("probabilities") if result.get("structured_findings") else None,
    )

@router.post("/test_vision_models")
async def test_all_vision_models(
file: UploadFile = File(...),
context: Optional[str] = Form(None),
tooth_number: Optional[str] = Form(None),
):
"""
TEST ENDPOINT: Compare ALL available vision models on the same image.

    Models tested:
    - llava:13b       (Ollama - best open-source)
    - llama3.2-vision (Ollama - Meta latest)
    - llava:latest    (Ollama - baseline, installed as llava:latest not llava:7b)
    - llava_med       (Ollama + medical system prompt, routes via llava:13b)
    - biomedclip      (open_clip - real BiomedCLIP classification)
    - florence        (HuggingFace Florence-2-base)
    - gpt4v           (OpenAI - skipped if no credits)
    - claude          (Anthropic - skipped if no credits)
    """
    logger.info(f"\n{'='*70}")
    logger.info(f"MULTI-MODEL VISION COMPARISON TEST")
    logger.info(f"{'='*70}")
    if context:
        logger.info(f"📝 Test Context: {context}")
    if tooth_number:
        logger.info(f"🦷 Test Tooth: #{tooth_number}")

    content = await file.read()
    image = ImageProcessor.preprocess_image(content)

    # (provider_type, ollama_model_override_or_None, label)
    models_to_test = [
        ("florence", None, "Florence-2 - Not recommended"),
        ("biomedclip", None, "BiomedCLIP - Pathology classifier (open_clip)"),
        ("llava", "llava:latest", "llava:latest - Baseline (7B)"),
        ("llava", "llava:13b", "llava:13b - Best open-source"),
        ("llava", "llama3.2-vision", "llama3.2-vision - Meta latest"),
        ("llava_med", None, "LLaVA-Med - Medical specialist (via llava:13b)"),
        ("gemma3", None, "Gemma 3 4B - Google multimodal local"),
        ("groq", None, "Groq Llama 3.2 90B - Ultra-fast cloud"),
        ("gemini", None, "Gemini 2.0 Flash - Google cloud"),
        ("claude", None, "Claude 3.5 Sonnet - Medical reasoning"),
        ("gpt4v", None, "GPT-4 Vision - Best accuracy"),
    ]

    results = {}
    original_provider = vision_settings.VISION_MODEL_PROVIDER
    original_llava_model = vision_settings.LLAVA_MODEL

    for provider, ollama_model, description in models_to_test:
        logger.info(f"\n{'─'*70}")
        logger.info(f"🔄 Testing: {provider} ({description})")
        logger.info(f"{'─'*70}")

        model_label = ollama_model if ollama_model else provider
        result_key = f"{provider}_{model_label.replace(':', '_').replace('.', '_').replace('-', '_')}"

        try:
            # Switch provider
            vision_settings.VISION_MODEL_PROVIDER = provider
            vision_client._clients = {}  # Force reload of client

            # Also switch the Ollama model if specified
            if ollama_model:
                vision_settings.LLAVA_MODEL = ollama_model
                logger.info(f"   🔧 Ollama model set to: {ollama_model}")

            start = time.time()
            result = vision_client.analyze_dental_radiograph(
                image, context=context, tooth_number=tooth_number
            )
            elapsed = (time.time() - start) * 1000

            # Record performance metrics
            input_tokens = result.get("input_tokens")
            output_tokens = result.get("output_tokens")
            structured = result.get("structured_findings")

            # Fallback for models that don't return structured findings (Florence, BiomedCLIP)
            primary_finding = None
            if structured:
                primary_finding = structured.get("primary_finding")
            else:
                primary_finding = result.get("pathology_summary")

            performance_tracker.record(
                model_name=model_label,
                inference_time_ms=elapsed,
                confidence_score=result.get("confidence_score"),
                image_quality_score=result.get("image_quality_score"),
                primary_finding=primary_finding,
                teeth_visible=structured.get("teeth_visible") if structured else None,
                input_tokens=input_tokens,
                output_tokens=output_tokens if output_tokens is not None else len(result.get("detailed_description", "").split()),
                context_provided=context is not None,
                tooth_number_provided=tooth_number is not None,
            )
            logger.info(f"📊 Recorded performance metrics for {model_label}")

            if result.get("error"):
                logger.error(f"❌ {provider} returned error: {result['error']}")
                results[result_key] = {
                    "success": False,
                    "description": description,
                    "error": result["error"],
                }
                continue

            results[result_key] = {
                "success": True,
                "description": description,
                "model_used": model_label,
                "structured_findings": structured,
                "teeth_visible": structured.get("teeth_visible") if structured else None,
                "primary_finding": structured.get("primary_finding") if structured else None,
                "pathology_summary": result.get("pathology_summary", ""),
                "image_quality": result.get("image_quality_score", 0.0),
                "confidence": result.get("confidence_score", 0.0),
                "time_ms": round(elapsed, 2),
            }

            logger.info(f"✅ Success - {elapsed:.0f}ms")
            if structured:
                logger.info(f"   Teeth: {structured.get('teeth_visible', [])}")
                logger.info(f"   Finding: {str(structured.get('primary_finding', ''))[:70]}...")

        except Exception as e:
            err_str = str(e)
            # Classify error type for cleaner reporting
            if "insufficient_quota" in err_str or "credit balance" in err_str:
                err_msg = "API billing: no credits. Add credits to use this model."
            elif "404" in err_str and "not found" in err_str:
                err_msg = f"Model not installed in Ollama: {model_label}"
            else:
                err_msg = err_str[:200]

            logger.error(f"❌ {provider} failed: {err_msg}")
            results[result_key] = {
                "success": False,
                "description": description,
                "error": err_msg,
            }
        finally:
            # Always restore after each test
            vision_settings.VISION_MODEL_PROVIDER = original_provider
            vision_settings.LLAVA_MODEL = original_llava_model
            vision_client._clients = {}

    # Final restore
    vision_settings.VISION_MODEL_PROVIDER = original_provider
    vision_settings.LLAVA_MODEL = original_llava_model

    successful = sum(1 for r in results.values() if r.get("success"))
    logger.info(f"\n{'='*70}")
    logger.info(f"TEST COMPLETE")
    logger.info(f"{'='*70}")
    logger.info(f"✅ {successful}/{len(results)} models succeeded")

    return {
        "test_context": {
            "context": context,
            "tooth_number": tooth_number,
            "models_tested": len(results),
            "successful": successful,
        },
        "results": results,
        "recommendation": {
            "best_open_source": "llava:13b or llama3.2-vision",
            "fastest_baseline": "llava:latest (same as llava:7b)",
            "medical_specialist": "llava_med (llava:13b + medical prompt)",
            "pathology_classifier": "biomedclip (BiomedCLIP via open_clip)",
            "best_accuracy": "gpt4v or claude (requires API credits)",
        },
    }

@router.get("/config", response_model=VisionConfigResponse)
async def get_vision_config():
"""
Get current vision system configuration.

    Returns all vision settings including:
    - Active vision model provider and model name
    - Inference parameters (temperature, max_tokens)
    - Prompt engineering settings
    - Image preprocessing options
    """
    return VisionConfigResponse(
        # Model Selection
        vision_model_provider=vision_settings.VISION_MODEL_PROVIDER,
        current_vision_model=vision_settings.current_vision_model,
        llava_model=vision_settings.LLAVA_MODEL,
        gemma3_model=vision_settings.GEMMA3_MODEL,
        groq_vision_model=vision_settings.GROQ_VISION_MODEL,
        gemini_vision_model=vision_settings.GEMINI_VISION_MODEL,
        openai_vision_model=vision_settings.OPENAI_VISION_MODEL,
        claude_vision_model=vision_settings.CLAUDE_VISION_MODEL,
        florence_model_name=vision_settings.FLORENCE_MODEL_NAME,
        llava_med_model=vision_settings.LLAVA_MED_MODEL,
        biomedclip_model=vision_settings.BIOMEDCLIP_MODEL,

        # Inference Settings
        vision_temperature=vision_settings.VISION_TEMPERATURE,
        vision_max_tokens=vision_settings.VISION_MAX_TOKENS,
        # Prompt Engineering
        include_clinical_notes_in_vision_model_prompt=vision_settings.INCLUDE_CLINICAL_NOTES_IN_VISION_MODEL_PROMPT,
        # Image Processing
        enhance_contrast=vision_settings.ENHANCE_CONTRAST,
        contrast_factor=vision_settings.CONTRAST_FACTOR,
        brightness_factor=vision_settings.BRIGHTNESS_FACTOR,
    )

@router.post("/config", response_model=VisionConfigResponse)
async def update_vision_config(config: VisionConfigRequest):
"""
Update vision configuration (in-memory only, resets on restart).

    Supports partial updates — send only the fields you want to change.

    Important cache invalidations:
    - Changing vision_model_provider → clears vision client cache
    - Changing llava_model → clears cache and forces model reload

    Example request:
    ```json
    {
        "vision_model_provider": "llava",
        "llava_model": "llama3.2-vision",
        "vision_temperature": 0.0
    }
    ```
    """
    updated_fields = []

    # ── Model Selection ──
    if config.vision_model_provider is not None:
        vision_settings.VISION_MODEL_PROVIDER = config.vision_model_provider
        # Clear vision client cache when provider changes
        vision_client._clients = {}
        updated_fields.append(
            f"vision_model_provider → {config.vision_model_provider} (cache cleared)"
        )

    if config.llava_model is not None:
        vision_settings.LLAVA_MODEL = config.llava_model
        # Force reload of LLaVA client
        if "llava" in vision_client._clients:
            del vision_client._clients["llava"]
        updated_fields.append(f"llava_model → {config.llava_model} (cache cleared)")

    if config.gemma3_model is not None:
        vision_settings.GEMMA3_MODEL = config.gemma3_model
        vision_client._clients = {}
        updated_fields.append(f"gemma3_model → {config.gemma3_model}")

    if config.groq_vision_model is not None:
        vision_settings.GROQ_VISION_MODEL = config.groq_vision_model
        vision_client._clients = {}
        updated_fields.append(f"groq_vision_model → {config.groq_vision_model}")

    if config.gemini_vision_model is not None:
        vision_settings.GEMINI_VISION_MODEL = config.gemini_vision_model
        vision_client._clients = {}
        updated_fields.append(f"gemini_vision_model → {config.gemini_vision_model}")

    # ── Inference Settings ──
    if config.vision_temperature is not None:
        vision_settings.VISION_TEMPERATURE = config.vision_temperature
        updated_fields.append(f"vision_temperature → {config.vision_temperature}")

    if config.vision_max_tokens is not None:
        vision_settings.VISION_MAX_TOKENS = config.vision_max_tokens
        updated_fields.append(f"vision_max_tokens → {config.vision_max_tokens}")

    # ── Prompt Engineering ──
    if config.include_clinical_notes_in_vision_model_prompt is not None:
        vision_settings.INCLUDE_CLINICAL_NOTES_IN_VISION_MODEL_PROMPT = (
            config.include_clinical_notes_in_vision_model_prompt
        )
        updated_fields.append(
            f"include_clinical_notes → {config.include_clinical_notes_in_vision_model_prompt}"
        )

    # ── Image Processing ──
    if config.enhance_contrast is not None:
        vision_settings.ENHANCE_CONTRAST = config.enhance_contrast
        updated_fields.append(f"enhance_contrast → {config.enhance_contrast}")

    if config.contrast_factor is not None:
        vision_settings.CONTRAST_FACTOR = config.contrast_factor
        updated_fields.append(f"contrast_factor → {config.contrast_factor}")

    if config.brightness_factor is not None:
        vision_settings.BRIGHTNESS_FACTOR = config.brightness_factor
        updated_fields.append(f"brightness_factor → {config.brightness_factor}")

    # Log changes
    logger.info(f"📝 Vision Config Updated: {len(updated_fields)} field(s)")
    for field in updated_fields:
        logger.info(f"   • {field}")

    # Return updated config
    return await get_vision_config()

</file>

<file path="app/visionsystem/vision_client.py">
# app/visionsystem/vision_client.py

import logging
import json
from typing import Dict, Optional
from PIL import Image
from config.visionconfig import vision_settings

logger = logging.getLogger(**name**)

def \_build_json_schema(tooth_number: str = None) -> str:
"""Build JSON schema with actual tooth number baked in — no more placeholder copying."""
focus = tooth_number if tooth_number else "the most symptomatic tooth" # Derive FDI quadrant hint from tooth number
if tooth_number and tooth_number.isdigit():
t = int(tooth_number)
if 41 <= t <= 48:
quadrant_hint = "41-48 range (lower right)"
elif 31 <= t <= 38:
quadrant_hint = "31-38 range (lower left)"
elif 11 <= t <= 18:
quadrant_hint = "11-18 range (upper right)"
elif 21 <= t <= 28:
quadrant_hint = "21-28 range (upper left)"
else:
quadrant_hint = "FDI range near " + tooth_number
else:
quadrant_hint = "FDI two-digit numbers only (e.g. 46, 47, 48)"

    return f"""RESPOND WITH ONLY THIS JSON:
            {{
            "teeth_visible": ["USE FDI NUMBERING ONLY - teeth in {quadrant_hint} - list only 3-5 actually visible"],
            "image_quality": "good/fair/poor",
            "focused_tooth": "{focus}",
            "caries": {{
                "present": true_or_false,
                "location": "tooth and surface, or null",
                "severity": "mild/moderate/severe or null",
                "notes": "describe dark crown areas, or null"
            }},
            "periapical_pathology": {{
                "present": true_or_false,
                "location": "tooth number, or null",
                "size_mm": null,
                "characteristics": "shape/border of radiolucency, or null",
                "notes": "periapical observations, or null"
            }},
            "bone_loss": {{
                "present": true_or_false,
                "type": "horizontal/vertical/null",
                "location": "between which teeth, or null",
                "severity": "mild/moderate/severe or null",
                "notes": "bone level description, or null"
            }},
            "root_canal_treatment": {{
                "present": true_or_false,
                "location": "tooth number, or null",
                "quality": "adequate/inadequate/null",
                "notes": "RCT appearance, or null"
            }},
            "restorations": {{
                "present": true_or_false,
                "type": "amalgam/composite/crown/null",
                "location": "tooth and surface, or null",
                "condition": "satisfactory/defective/null",
                "notes": "restoration description, or null"
            }},
            "other_abnormalities": [],
            "primary_finding": "Describe main finding on tooth {focus} based on what you see",
            "severity": "normal/mild/moderate/severe",
            "urgency": "routine/prompt/urgent/emergency",
            "image_quality_score": 0.8,
            "diagnostic_confidence": 0.7,
            "interpretation_notes": null,
            "narrative_summary": "2-3 sentence clinical summary of findings on tooth {focus}"
            }}

            RULES — MUST FOLLOW:
            - teeth_visible: FDI two-digit numbers ONLY (e.g. 46, 47, 48) — NEVER use 1-32 Universal numbering
            - Maximum 5 teeth in teeth_visible — periapical X-rays show 3-5 teeth
            - Tooth {focus} MUST be included in teeth_visible
            - If dark areas in crown → caries present=true
            - If dark halo at root tip → periapical_pathology present=true
            - primary_finding: write what YOU SEE, not a template phrase
            """

class VisionClient:
"""Unified interface to ALL vision models."""

    _instance = None
    _clients = {}

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def _get_client(self):
        provider = vision_settings.VISION_MODEL_PROVIDER
        if provider not in self._clients:
            logger.info(f"🔄 Loading {provider} vision client...")
            if provider == "llava":
                from app.visionsystem.llava_client import llava_client
                self._clients["llava"] = llava_client
            elif provider == "llava_med":
                from app.visionsystem.llava_med_client import llava_med_client
                self._clients["llava_med"] = llava_med_client
            elif provider == "biomedclip":
                from app.visionsystem.biomedclip_client import biomedclip_client
                self._clients["biomedclip"] = biomedclip_client
            elif provider == "gpt4v":
                from app.visionsystem.gpt4_client import gpt4v_client
                self._clients["gpt4v"] = gpt4v_client
            elif provider == "claude":
                from app.visionsystem.claude_vision_client import claude_vision_client
                self._clients["claude"] = claude_vision_client
            elif provider == "florence":
                from app.visionsystem.florence_client import florence_client
                self._clients["florence"] = florence_client
            elif provider == "groq":
                from app.visionsystem.groq_vision_client import groq_vision_client
                self._clients["groq"] = groq_vision_client
            elif provider == "gemini":
                from app.visionsystem.gemini_vision_client import gemini_vision_client
                self._clients["gemini"] = gemini_vision_client
            elif provider == "gemma3":
                from app.visionsystem.gemma3_client import gemma3_vision_client
                self._clients["gemma3"] = gemma3_vision_client
            else:
                raise ValueError(f"Unknown vision provider: {provider}")
            logger.info(f"✅ {provider} client loaded")
        return self._clients[provider]

    def get_model_info(self) -> Dict:
        """
        Gets model info from the currently configured vision client.
        Delegates to the active client's get_model_info method if it exists,
        otherwise provides basic info from settings.
        """
        client = self._get_client()

        if hasattr(client, "get_model_info"):
            try:
                return client.get_model_info()
            except Exception as e:
                logger.error(f"Underlying vision client get_model_info() failed: {e}")
                raise

        # logger.warning(
        #     f"Vision provider '{vision_settings.VISION_MODEL_PROVIDER}' "
        #     f"client does not have a get_model_info method. Returning config data."
        # )
        return {
            "provider": vision_settings.VISION_MODEL_PROVIDER,
            "model": vision_settings.current_vision_model,
            "temperature": vision_settings.VISION_TEMPERATURE,
            "max_tokens": vision_settings.VISION_MAX_TOKENS,
            "reliable": False,
            "notes": "This client does not implement a live health check. Info is from static config.",
        }

    def analyze_dental_radiograph(
        self,
        image: Image.Image,
        context: str = None,
        clinical_notes: str = None,
        tooth_number: str = None
    ) -> Dict:
        client = self._get_client()
        provider = vision_settings.VISION_MODEL_PROVIDER
        logger.info(f"🔍 Analyzing dental radiograph with {provider}...: Vision Model: {vision_settings.current_vision_model}")
        if context:
            logger.info(f"   📋 Context: {context[:80]}...")
        if tooth_number:
            logger.info(f"   🦷 FOCUS TOOTH: #{tooth_number}")
        if provider == "florence":
            return self._analyze_with_florence(client, image, context, tooth_number)
        elif provider == "biomedclip":
            return self._analyze_with_biomedclip(client, image, tooth_number)
        else:
            return self._analyze_with_structured_output(client, provider, image, context, clinical_notes, tooth_number)

    def _analyze_with_florence(self, client, image, context=None, tooth_number=None):
        logger.info("⚠️  Florence-2: Using simple task token (no context support)")
        try:
            response_dict = client.analyze_image(image, "<MORE_DETAILED_CAPTION>")
            response_text = response_dict["text"]
            logger.info(f"Analysis complete: {len(response_text)} chars")

            # Use a snippet of the description as the summary
            summary = response_text[:200] + "..." if len(response_text) > 200 else response_text

            return {"structured_findings": None, "narrative_analysis": response_text, "model": "florence",
                    "detailed_description": response_text, "pathology_summary": summary,
                    "confidence_score": 0.5, "image_quality_score": 0.5,
                    "input_tokens": None, "output_tokens": None}
        except Exception as e:
            return self._error_response("florence", str(e))

    def _analyze_with_biomedclip(self, client, image, tooth_number=None):
        logger.info("🔬 BiomedCLIP: Running pathology classification...")
        try:
            result = client.classify_pathology(image)
            if "error" in result:
                return self._error_response("biomedclip", result["error"])

            # The analyze_image method returns the text-based bar chart
            analysis_result = client.analyze_image(image)
            detailed_description = analysis_result['text']

            # This is the key clinical finding for BiomedCLIP
            summary = f"Classification: {result['prediction']} ({result['confidence']:.1%})"

            # Put the breakdown into structured_findings so it's accessible as data
            return {
                "structured_findings": {
                    "probabilities": result.get("all_scores"),
                    "primary_finding": result["prediction"],
                    "confidence": result["confidence"]
                },
                "narrative_analysis": summary,
                "model": "biomedclip",
                "detailed_description": detailed_description,
                "pathology_summary": summary,
                "confidence_score": result["confidence"],
                "image_quality_score": 0.5,
                "input_tokens": None,
                "output_tokens": None
            }
        except Exception as e:
            return self._error_response("biomedclip", str(e))

    def _analyze_with_structured_output(self, client, provider, image, context=None, clinical_notes=None, tooth_number=None):
        prompt = self._build_structured_prompt(context, clinical_notes, tooth_number)
        logger.info(f"📤 Sending structured prompt to {provider}")
        logger.info(f"   Prompt length: {len(prompt)} chars")
        try:
            response_dict = client.analyze_image(image, prompt)
            response_text = response_dict["text"]
            input_tokens = response_dict.get("input_tokens")
            output_tokens = response_dict.get("output_tokens")

            logger.info(f"{provider.upper()} analysis complete: {len(response_text)} characters")

            parsed_response = self._parse_structured_response(response_text, provider, tooth_number)
            parsed_response["input_tokens"] = input_tokens
            parsed_response["output_tokens"] = output_tokens
            return parsed_response

        except Exception as e:
            logger.error(f"❌ {provider} analysis failed: {e}")
            raise

    def _build_structured_prompt(self, context=None, clinical_notes=None, tooth_number=None):
        prompt = """You are an expert dental radiologist analyzing a periapical X-ray.

                MANDATORY:
                1. Respond with ONLY valid JSON - no preamble, no markdown, no explanation
                2. Periapical X-rays show 3-5 adjacent teeth — list all using FDI numbering
                3. PATHOLOGY IS EXPECTED — look carefully and report what you actually see

                READING GUIDE:
                - DARK areas (radiolucent) = PATHOLOGY: caries, abscess, bone loss
                - BRIGHT areas (radiopaque) = enamel, fillings, healthy bone
                - Caries: dark spots/shadows in tooth crown
                - Periapical abscess: dark halo around root tip
                - Bone loss: reduced bone height between teeth

                """
        if tooth_number or context:
            prompt += "CLINICAL CONTEXT:\n"
            if tooth_number:
                prompt += f"⚠️  FOCUS TOOTH: #{tooth_number} — patient has pain here. Pathology is LIKELY PRESENT.\n"
                prompt += f"   Examine tooth #{tooth_number} with extreme care.\n"
            if context:
                prompt += f"Chief Complaint: {context}\n"
            if clinical_notes and vision_settings.INCLUDE_CLINICAL_NOTES_IN_VISION_MODEL_PROMPT:
                prompt += f"Notes: {clinical_notes[:200]}\n"
            prompt += "\n"

        # KEY FIX: inject actual tooth number so model never sees "TOOTH_NUMBER_HERE"
        prompt += _build_json_schema(tooth_number)
        prompt += "\nANALYZE THE X-RAY AND RESPOND WITH JSON ONLY:"
        return prompt

    def _parse_structured_response(self, response: str, provider: str, tooth_number: str = None) -> Dict:
        logger.info(f"📥 Parsing response from {provider} ({len(response)} chars)")
        try:
            cleaned = response.strip()
            for prefix in ["```json", "```"]:
                if cleaned.startswith(prefix):
                    cleaned = cleaned[len(prefix):]
            if cleaned.endswith("```"):
                cleaned = cleaned[:-3]
            cleaned = cleaned.strip()

            structured_data = json.loads(cleaned)

            # KEY FIX: Override focused_tooth with actual value BEFORE formatting
            # This ensures the placeholder never leaks into narrative/summary strings
            if tooth_number:
                structured_data["focused_tooth"] = tooth_number

            # Format narrative/summary using corrected data
            detailed = self._format_narrative(structured_data)
            summary = self._format_summary(structured_data)

            logger.info("✅ Successfully parsed JSON")
            logger.info(f"   Focused tooth: {structured_data.get('focused_tooth', 'N/A')}")
            logger.info(f"   Finding: {str(structured_data.get('primary_finding', 'N/A'))[:70]}...")

            return {
                "structured_findings": structured_data,
                "narrative_analysis": structured_data.get("narrative_summary", ""),
                "model": provider,
                "detailed_description": detailed,
                "pathology_summary": summary,
                "confidence_score": structured_data.get("diagnostic_confidence", 0.5),
                "image_quality_score": structured_data.get("image_quality_score", 0.5),
            }
        except json.JSONDecodeError as e:
            logger.error(f"❌ JSON parse failed: {e}")
            raise RuntimeError(f"Vision model {provider} did not return valid JSON. Please try again so we get structured output.")

    def _format_narrative(self, data: Dict) -> str:
        parts = ["RADIOGRAPHIC ANALYSIS",
                 f"Teeth visible: {', '.join(data.get('teeth_visible', []))}",
                 f"Image quality: {data.get('image_quality', 'unknown')}"]
        if data.get("focused_tooth"):
            parts.append(f"PRIMARY FOCUS: Tooth #{data['focused_tooth']}\n")
        findings = []
        if data.get("caries", {}).get("present"):
            c = data["caries"]
            findings.append(f"CARIES: {c.get('location')}, {c.get('severity')}")
        if data.get("periapical_pathology", {}).get("present"):
            p = data["periapical_pathology"]
            findings.append(f"PERIAPICAL: Tooth {p.get('location')}")
        if data.get("bone_loss", {}).get("present"):
            b = data["bone_loss"]
            findings.append(f"BONE LOSS: {b.get('severity')} {b.get('type')}")
        if data.get("root_canal_treatment", {}).get("present"):
            findings.append(f"RCT: {data['root_canal_treatment'].get('location')}")
        if data.get("restorations", {}).get("present"):
            r = data["restorations"]
            findings.append(f"RESTORATION: {r.get('type')} at {r.get('location')}")
        for a in data.get("other_abnormalities", []):
            findings.append(f"OTHER: {a}")
        if findings:
            parts.append("\nPATHOLOGY DETECTED:")
            parts.extend([f"  ✓ {f}" for f in findings])
        else:
            parts.append("\nNo significant pathology detected")
        parts.append(f"\nPrimary Finding: {data.get('primary_finding', 'None')}")
        parts.append(f"Severity: {data.get('severity', 'unknown')}")
        parts.append(f"Urgency: {data.get('urgency', 'unknown')}")
        if data.get("narrative_summary"):
            parts.append(f"\nSummary: {data['narrative_summary']}")
        return "\n".join(parts)

    def _format_summary(self, data: Dict) -> str:
        parts = []
        if data.get("focused_tooth"):
            parts.append(f"**Focused on tooth #{data['focused_tooth']}**\n")
        caries = data.get("caries", {})
        periapical = data.get("periapical_pathology", {})
        bone = data.get("bone_loss", {})
        rct = data.get("root_canal_treatment", {})
        resto = data.get("restorations", {})
        parts.extend([
            f"CARIES: {'✓ YES - ' + str(caries.get('location', '')) if caries.get('present') else 'No'}",
            f"PERIAPICAL: {'✓ YES - ' + str(periapical.get('location', '')) if periapical.get('present') else 'No'}",
            f"BONE LOSS: {'✓ YES - ' + str(bone.get('severity', '')) if bone.get('present') else 'No'}",
            f"RCT: {'✓ YES' if rct.get('present') else 'No'}",
            f"RESTORATIONS: {'✓ YES - ' + str(resto.get('type', '')) if resto.get('present') else 'No'}",
            f"\nPrimary Finding: {data.get('primary_finding', 'None')}",
            f"Severity: {data.get('severity', 'unknown')}",
            f"Urgency: {data.get('urgency', 'unknown')}",
        ])
        return "\n".join(parts)

    def _error_response(self, model: str, error: str) -> Dict:
        return {"structured_findings": None, "narrative_analysis": f"Analysis failed: {error}",
                "model": model, "detailed_description": f"Error: {error}",
                "pathology_summary": "Analysis failed", "confidence_score": 0.0,
                "image_quality_score": 0.0, "error": error}

# Global instance

vision_client = VisionClient()
</file>

<file path="app/visionsystem/vision_schemas.py">
# app/visionsystem/vision_schemas.py
"""
Structured schemas for vision model outputs to ensure consistent, parseable results
"""
from pydantic import BaseModel, Field
from typing import List, Literal, Optional, Dict, Any

class VisionAnalysisResponse(BaseModel):
"""Response from vision analysis endpoint."""

    # Structured findings
    structured_findings: Optional[Dict[str, Any]] = Field(
        None,
        description="Structured pathology findings if available"
    )

    # Full probabilities (BiomedCLIP)
    probabilities: Optional[Dict[str, float]] = Field(
        None,
        description="Full probability distribution for classification models"
    )

    # Narrative components
    detailed_description: str = Field(..., description="Complete analysis")
    pathology_summary: str = Field(..., description="Key findings")

    # Metadata
    model_used: str = Field(..., description="Vision model used")
    processing_time_ms: float = Field(..., description="Processing time")
    focused_tooth: Optional[str] = Field(None, description="Focused tooth if specified")

    # Scientific confidence
    image_quality_score: float = Field(default=0.5, ge=0.0, le=1.0)
    diagnostic_confidence: float = Field(default=0.5, ge=0.0, le=1.0)
    confidence_level: Literal["high", "medium", "low"] = Field(default="medium")

class CariesFinding(BaseModel):
"""Structured caries finding"""
present: bool = Field(..., description="Whether caries are present")
location: Optional[str] = Field(None, description="Tooth number and surface (e.g., '23 mesial')")
severity: Optional[Literal["early", "moderate", "deep"]] = Field(None)
notes: Optional[str] = Field(None, description="Additional details")

class PeriapicalFinding(BaseModel):
"""Structured periapical pathology finding"""
present: bool = Field(..., description="Whether periapical lesion/abscess present")
location: Optional[str] = Field(None, description="Tooth number (e.g., '23')")
size_mm: Optional[float] = Field(None, description="Approximate size in mm if measurable")
characteristics: Optional[str] = Field(None, description="Well-defined, diffuse, etc.")
notes: Optional[str] = Field(None)

class BoneLossFinding(BaseModel):
"""Structured bone loss finding"""
present: bool = Field(..., description="Whether bone loss is present")
type: Optional[Literal["horizontal", "vertical", "mixed"]] = Field(None)
location: Optional[str] = Field(None, description="Which teeth affected")
severity: Optional[Literal["mild", "moderate", "severe"]] = Field(None)
notes: Optional[str] = Field(None)

class RootCanalTreatment(BaseModel):
"""Structured root canal treatment assessment"""
present: bool = Field(..., description="Whether RCT is present")
location: Optional[str] = Field(None, description="Tooth number")
quality: Optional[Literal["adequate", "short", "overfilled", "underfilled"]] = Field(None)
notes: Optional[str] = Field(None)

class RestorationFinding(BaseModel):
"""Structured restoration finding"""
present: bool = Field(..., description="Whether restorations are present")
type: Optional[str] = Field(None, description="Amalgam, composite, crown, etc.")
location: Optional[str] = Field(None, description="Tooth number and surface")
condition: Optional[Literal["intact", "defective", "overhanging", "recurrent decay"]] = Field(None)
notes: Optional[str] = Field(None)

class OtherAbnormality(BaseModel):
"""Any other abnormal finding"""
description: str = Field(..., description="Description of the abnormality")
location: Optional[str] = Field(None)
clinical_significance: Optional[Literal["urgent", "prompt", "routine", "monitor"]] = Field(None)

class StructuredPathologyFindings(BaseModel):
"""Complete structured pathology findings from radiograph"""

    # Anatomical assessment
    teeth_visible: List[str] = Field(
        default_factory=list,
        description="List of tooth numbers visible (e.g., ['23', '24', '25'])"
    )
    image_quality: Literal["excellent", "good", "adequate", "poor"] = Field(
        ...,
        description="Overall radiograph quality"
    )

    # Pathology findings
    caries: CariesFinding = Field(..., description="Caries findings")
    periapical_pathology: PeriapicalFinding = Field(..., description="Periapical findings")
    bone_loss: BoneLossFinding = Field(..., description="Bone loss findings")
    root_canal_treatment: RootCanalTreatment = Field(..., description="RCT findings")
    restorations: RestorationFinding = Field(..., description="Restoration findings")
    other_abnormalities: List[OtherAbnormality] = Field(
        default_factory=list,
        description="Any other abnormal findings"
    )

    # Clinical assessment
    primary_radiographic_finding: str = Field(
        ...,
        description="Main finding based on radiograph alone"
    )
    severity: Literal["normal", "mild", "moderate", "severe"] = Field(
        ...,
        description="Overall severity based on radiographic findings"
    )
    urgency: Literal["emergency", "urgent", "prompt", "routine"] = Field(
        ...,
        description="Recommended urgency level"
    )

    # Confidence scoring
    image_quality_score: float = Field(
        ...,
        ge=0.0,
        le=1.0,
        description="0-1 score for image quality (affects diagnostic confidence)"
    )
    diagnostic_confidence: float = Field(
        ...,
        ge=0.0,
        le=1.0,
        description="0-1 score for confidence in the radiographic interpretation"
    )
    interpretation_notes: Optional[str] = Field(
        None,
        description="Any limitations or areas of uncertainty in interpretation"
    )

class VisionAnalysisResult(BaseModel):
"""Complete vision analysis result with both narrative and structured data"""

    # Narrative analysis (for context/review)
    narrative_analysis: str = Field(
        ...,
        description="Full narrative radiographic interpretation"
    )

    # Structured findings (for programmatic use)
    structured_findings: StructuredPathologyFindings = Field(
        ...,
        description="Structured, parseable pathology findings"
    )

    # Clinical context integration
    clinical_context_considered: bool = Field(
        default=False,
        description="Whether clinical context was provided and considered"
    )
    focused_tooth_number: Optional[str] = Field(
        None,
        description="Specific tooth number that was focus of analysis if provided"
    )

    # Model metadata
    model_used: str = Field(..., description="Vision model used for analysis")

</file>

<file path="config/config_schemas.py">
# config/config_schemas.py
"""
Configuration Schemas - COMPLETE WITH ALL MODELS
Includes: Groq, Gemini, Gemma3 support
"""
from pydantic import BaseModel, Field
from typing import Literal, Optional

# ============================================================================

# VISION SYSTEM CONFIGURATION

# ============================================================================

class VisionConfigRequest(BaseModel):
"""Request to update vision system configuration."""

    # Model Selection
    vision_model_provider: Optional[Literal[
        "llava",        # Ollama: llava:13b, llama3.2-vision
        "gemma3",       # Ollama: gemma3:4b, gemma3:12b (NEW)
        "llava_med",    # HuggingFace: Medical specialist
        "biomedclip",   # HuggingFace: Pathology classifier
        "florence",     # HuggingFace: General vision
        "gpt4v",        # OpenAI: Premium cloud
        "claude",       # Anthropic: Premium cloud
        "groq",         # Groq: Ultra-fast cloud (NEW)
        "gemini"        # Google: Multimodal cloud (NEW)
    ]] = Field(None, description="Vision model provider")

    llava_model: Optional[str] = Field(
        None,
        description="Ollama LLaVA model",
        examples=["llava:13b", "llama3.2-vision", "llava:latest"]
    )

    gemma3_model: Optional[str] = Field(
        None,
        description="Ollama Gemma3 model (NEW)",
        examples=["gemma3:4b", "gemma3:12b"]
    )

    groq_vision_model: Optional[str] = Field(
        None,
        description="Groq vision model (NEW)",
        examples=["meta-llama/llama-4-scout-17b-16e-instruct", "meta-llama/llama-4-maverick-17b-128e-instruct"]
    )

    gemini_vision_model: Optional[str] = Field(
        None,
        description="Google Gemini vision model (NEW)",
        examples=["gemini-2.0-flash", "gemini-1.5-pro"]
    )

    # Inference Settings
    vision_temperature: Optional[float] = Field(
        None, ge=0.0, le=2.0, description="Temperature for vision model generation"
    )
    vision_max_tokens: Optional[int] = Field(
        None, ge=100, le=4000, description="Maximum tokens in vision response"
    )

    # Prompt Engineering
    include_clinical_notes_in_vision_model_prompt: Optional[bool] = Field(
        None, description="Include clinical notes in vision prompt"
    )

    # Image Processing
    enhance_contrast: Optional[bool] = Field(
        None, description="Apply contrast enhancement to X-rays"
    )
    contrast_factor: Optional[float] = Field(
        None, ge=1.0, le=3.0, description="Contrast enhancement factor"
    )
    brightness_factor: Optional[float] = Field(
        None, ge=0.5, le=2.0, description="Brightness adjustment factor"
    )

class VisionConfigResponse(BaseModel):
"""Current vision system configuration."""

    # Model Selection
    vision_model_provider: str
    current_vision_model: str
    llava_model: str
    gemma3_model: str  # NEW
    groq_vision_model: str  # NEW
    gemini_vision_model: str  # NEW
    openai_vision_model:str #NEW
    claude_vision_model:str #NEW
    florence_model_name:str #NEW
    llava_med_model:str #NEW
    biomedclip_model:str #NEW

    # Inference Settings
    vision_temperature: float
    vision_max_tokens: int

    # Prompt Engineering
    include_clinical_notes_in_vision_model_prompt: bool

    # Image Processing
    enhance_contrast: bool
    contrast_factor: float
    brightness_factor: float

# ============================================================================

# RAG SYSTEM CONFIGURATION

# ============================================================================

class RAGConfigRequest(BaseModel):
"""Request to update RAG system configuration."""

    # LLM Provider and Models
    llm_provider: Optional[Literal[
        "ollama",   # Local: Free
        "openai",   # Cloud: Premium
        "claude",   # Cloud: Premium
        "groq",     # Cloud: Ultra-fast (NEW)
        "gemini"    # Cloud: Google (NEW)
    ]] = Field(None, description="LLM provider for generating recommendations")

    ollama_llm_model: Optional[str] = Field(
        None,
        description="Ollama model name",
        examples=["llama3.1:8b", "mixtral:8x7b", "gemma3:4b"]
    )
    openai_llm_model: Optional[str] = Field(
        None,
        description="OpenAI model name",
        examples=["gpt-4o", "gpt-4-turbo-preview"]
    )
    claude_llm_model: Optional[str] = Field(
        None,
        description="Claude model name",
        examples=["claude-3-5-sonnet-20241022"]
    )
    groq_llm_model: Optional[str] = Field(
        None,
        description="Groq model name (NEW)",
        examples=["llama-3.3-70b-versatile", "mixtral-8x7b-32768"]
    )
    gemini_llm_model: Optional[str] = Field(
        None,
        description="Gemini model name (NEW)",
        examples=["gemini-2.0-flash", "gemini-1.5-pro"]
    )

    # Embedding Provider and Models
    embedding_provider: Optional[Literal["ollama", "huggingface"]] = Field(
        None, description="Embedding provider for document retrieval"
    )
    ollama_embedding_model: Optional[str] = Field(
        None,
        description="Ollama embedding model",
        examples=["nomic-embed-text", "mxbai-embed-large"]
    )
    hf_embedding_model: Optional[str] = Field(
        None,
        description="HuggingFace embedding model",
        examples=["abhinand/MedEmbed-large-v0.1", "BAAI/bge-large-en-v1.5"]
    )

    # Retrieval Settings
    retriever_type: Optional[Literal["similarity", "mmr", "multi_query", "similarity_score_threshold"]] = Field(
        None, description="Retrieval strategy"
    )
    retrieval_k: Optional[int] = Field(
        None, ge=1, le=20, description="Number of document chunks to retrieve"
    )
    fetch_k: Optional[int] = Field(
        None, ge=5, le=50, description="MMR: Number of candidates to consider"
    )
    lambda_mult: Optional[float] = Field(
        None, ge=0.0, le=1.0, description="MMR: 1.0=relevance, 0.0=diversity"
    )
    similarity_threshold: Optional[float] = Field(
        None, ge=0.0, le=1.0, description="Minimum similarity score"
    )

    # LLM Generation Settings
    llm_temperature: Optional[float] = Field(
        None, ge=0.0, le=2.0, description="Temperature"
    )
    max_tokens: Optional[int] = Field(
        None, ge=100, le=4000, description="Maximum tokens in LLM response"
    )

    # Document Processing
    chunk_size: Optional[int] = Field(
        None, ge=500, le=5000, description="Text chunk size"
    )
    chunk_overlap: Optional[int] = Field(
        None, ge=0, le=500, description="Overlap between chunks"
    )

class RAGConfigResponse(BaseModel):
"""Current RAG system configuration."""

    # LLM
    llm_provider: str
    current_llm_model: str
    ollama_llm_model: str
    openai_llm_model: str
    claude_llm_model: str
    groq_llm_model: str  # NEW
    gemini_llm_model: str  # NEW

    # Embedding
    embedding_provider: str
    current_embedding_model: str
    ollama_embedding_model: str
    hf_embedding_model: str

    # Retrieval
    retriever_type: str
    retrieval_k: int
    fetch_k: int
    lambda_mult: float
    similarity_threshold: Optional[float]

    # Generation
    llm_temperature: float
    max_tokens: int

    # Processing
    chunk_size: int
    chunk_overlap: int
    pdf_path: str
    persist_dir: str
    device: str

# ============================================================================

# COMBINED SYSTEM STATUS

# ============================================================================

class SystemConfigResponse(BaseModel):
"""Complete system configuration (RAG + Vision combined)."""
rag: RAGConfigResponse
vision: VisionConfigResponse
</file>

<file path="config/ragconfig.py">
# config/ragconfig.py
"""
RAG System Configuration - COMPLETE WITH ALL LLM PROVIDERS
Controls knowledge retrieval and LLM settings for clinical recommendations
Supports: Ollama, OpenAI, Claude, Groq, Gemini
"""
import os
from pathlib import Path
from typing import Literal, Optional

import torch
from pydantic import Field
from pydantic_settings import BaseSettings

# Calculate the project root

BASE_DIR = Path(**file**).resolve().parent.parent

class RAGSettings(BaseSettings):
"""Configuration for Clinical Decision Support RAG System"""

    # ============================================================================
    # LLM SELECTION (for generating clinical recommendations)
    # ============================================================================
    LLM_PROVIDER: Literal["ollama", "openai", "claude", "groq", "gemini"] = "ollama"

    # ── Ollama LLM Settings (Local, Free) ──
    # OLLAMA_LLM_MODEL: str = "llama3.1:8b"  # Recommended
    # OLLAMA_LLM_MODEL: str = "llama3:8b"
    OLLAMA_LLM_MODEL: str = "gemma3:4b"
    # Alternatives: "llama3:8b", "mixtral:8x7b", "gemma3:4b"

    # ── OpenAI Settings (Cloud, Paid) ──
    OPENAI_API_KEY: str = Field(default="", env="OPENAI_API_KEY")
    OPENAI_LLM_MODEL: str = "gpt-4o"
    # Alternatives: "gpt-4-turbo-preview", "gpt-3.5-turbo"

    # ── Claude Settings (Cloud, Paid) ──
    CLAUDE_API_KEY: str = Field(default="", env="CLAUDE_API_KEY")
    CLAUDE_LLM_MODEL: str = "claude-3-haiku-20240307"
    # Alternatives: "claude-3-opus-20240229", "claude-3-haiku-20240307"

    # ── Groq Settings (NEW - Cloud, Ultra-fast, Free tier) ──
    # Free tier: 7,000 requests/day!
    # Best for: Speed-critical applications
    GROQ_API_KEY: str = Field(default="", env="GROQ_API_KEY")
    GROQ_LLM_MODEL: str = "llama-3.3-70b-versatile"
    # Alternatives: "mixtral-8x7b-32768", "llama-3.1-70b-versatile"

    # ── Gemini Settings (NEW - Google Cloud, Competitive pricing) ──
    # Free tier: 1,500 requests/day
    # Best for: Long context (2M tokens), multimodal reasoning
    GEMINI_API_KEY: str = Field(default="", env="GEMINI_API_KEY")
    GEMINI_LLM_MODEL: str = "gemini-2.5-pro"
    # Alternatives: "gemini-1.5-pro", "gemini-1.5-flash", "gemini-2.5-pro", "gemini-2.5-flash"

    # ============================================================================
    # EMBEDDING MODEL SELECTION (for document retrieval)
    # ============================================================================
    EMBEDDING_PROVIDER: Literal["ollama", "huggingface"] = "ollama"

    # ── Ollama Embedding Settings ──
    OLLAMA_EMBEDDING_MODEL: str = "nomic-embed-text"
    # Alternatives: "mxbai-embed-large"

    # ── HuggingFace Embedding Settings ──
    HF_EMBEDDING_MODEL: str = "neuml/pubmedbert-base-embeddings"
    # Alternatives: "abhinand/MedEmbed-large-v0.1", "BAAI/bge-large-en-v1.5"

    # ============================================================================
    # RETRIEVAL SETTINGS
    # ============================================================================
    RETRIEVER_TYPE: Literal["similarity", "mmr", "multi_query", "similarity_score_threshold"] = (
        "mmr"
    )
    RETRIEVAL_K: int = 8  # Number of chunks to retrieve
    FETCH_K: int = 20  # MMR: Candidates to consider
    LAMBDA_MULT: float = 1.0  # MMR: 1.0=relevance, 0.0=diversity
    SIMILARITY_THRESHOLD: Optional[float] = 0.6  # Min similarity score

    # ============================================================================
    # LLM GENERATION SETTINGS
    # ============================================================================
    LLM_TEMPERATURE: float = 0.0  # 0=deterministic (clinical use)
    MAX_TOKENS: int = 1500
    FORMAT: Literal["json", "markdown"] = "json"
    FORCE_JSON_OUTPUT: bool = True

    # ============================================================================
    # DOCUMENT PROCESSING
    # ============================================================================
    CHUNK_SIZE: int = 1500  # Characters per chunk
    CHUNK_OVERLAP: int = 100  # Overlap for context continuity
    PDF_PATH: str = str(BASE_DIR / "documents" / "stg.pdf")

    # ============================================================================
    # VECTOR STORE
    # ============================================================================
    PERSIST_DIR: str = str(BASE_DIR / "chroma_db")

    # ============================================================================
    # DEVICE CONFIGURATION
    # ============================================================================
    DEVICE: str = "auto"  # Options: "auto", "cuda", "mps", "cpu"

    class Config:
        env_file = ".env"
        extra = "ignore"

    # ========================================================================
    # COMPUTED PROPERTIES
    # ========================================================================
    @property
    def resolved_pdf_path(self) -> Path:
        """Get absolute path to PDF document."""
        path = Path(self.PDF_PATH)
        return path if path.is_absolute() else BASE_DIR / path

    @property
    def resolved_persist_dir(self) -> Path:
        """Get absolute path to vector store directory."""
        path = Path(self.PERSIST_DIR)
        return path if path.is_absolute() else BASE_DIR / path

    @property
    def effective_device(self) -> str:
        """Determine compute device based on configuration."""
        if self.DEVICE == "auto":
            if torch.cuda.is_available():
                return "cuda"
            elif torch.backends.mps.is_available():
                return "mps"
            else:
                return "cpu"
        return self.DEVICE

    @property
    def current_embedding_model(self) -> str:
        """Get active embedding model based on provider."""
        if self.EMBEDDING_PROVIDER == "ollama":
            return self.OLLAMA_EMBEDDING_MODEL
        return self.HF_EMBEDDING_MODEL

    @property
    def current_llm_model(self) -> str:
        """Get active LLM model based on provider."""
        provider_map = {
            "ollama": self.OLLAMA_LLM_MODEL,
            "openai": self.OPENAI_LLM_MODEL,
            "claude": self.CLAUDE_LLM_MODEL,
            "groq": self.GROQ_LLM_MODEL,
            "gemini": self.GEMINI_LLM_MODEL,
        }
        return provider_map.get(self.LLM_PROVIDER, self.OLLAMA_LLM_MODEL)

rag_settings = RAGSettings()
</file>

<file path="config/visionconfig.py">
# config/visionconfig.py
"""
Vision Model Configuration - COMPLETE WITH ALL MODELS
Supports:
  LOCAL: LLaVA, Gemma3, LLaVA-Med, BiomedCLIP, Florence
  CLOUD: GPT-4V, Claude, Groq, Gemini
"""
from typing import Literal

from pydantic import Field
from pydantic_settings import BaseSettings

class VisionSettings(BaseSettings):
"""Configuration for all vision models in the system"""

    # ========================================================================
    # PRIMARY VISION MODEL SELECTION
    # ========================================================================
    VISION_MODEL_PROVIDER: Literal[
        "florence",  # HuggingFace: General vision (not recommended)
        "biomedclip",  # HuggingFace: Pathology classifier
        "llava",  # Ollama: llava:13b, llama3.2-vision, llava:latest
        "llava_med",  # HuggingFace: Medical specialist
        "gemma3",  # Ollama: gemma3:4b, gemma3:12b
        "groq",  # Groq: Ultra-fast, cheap
        "gemini",  # Google: Multimodal, large context
        "claude",  # Anthropic: Excellent reasoning
        "gpt4v",  # OpenAI: Best accuracy, expensive
    ] = "gpt4v"

    # ========================================================================
    # LOCAL MODELS - OLLAMA (Free, No API)
    # ========================================================================

    # ── LLaVA Settings ──
    # Available: llava:13b (recommended), llama3.2-vision (Meta latest), llava:latest
    # LLAVA_MODEL: str = "llava:13b"
    LLAVA_MODEL: str = "llava:latest"
    # LLAVA_MODEL: str = "llama3.2-vision"

    # ── Gemma 3 Settings (NEW - Google's multimodal) ──
    # Available: gemma3:4b (recommended for 16GB RAM), gemma3:12b (better accuracy)
    # Note: gemma3:4b is vision-capable, gemma3:1b is text-only
    GEMMA3_MODEL: str = "gemma3:4b"

    # ── LLaVA-Med Settings (HuggingFace transformers) ──
    # Medical-specific vision model
    LLAVA_MED_MODEL: str = "mradermacher/llava-med-v1.5-mistral-7b-GGUF"
    LLAVA_MED_DEVICE: str = "cpu"

    # ── BiomedCLIP Settings (HuggingFace open_clip) ──
    # Medical pathology classifier
    BIOMEDCLIP_MODEL: str = "microsoft/BiomedCLIP-PubMedBERT_256-vit_base_patch16_224"
    BIOMEDCLIP_DEVICE: str = "cpu"

    # ── Florence Settings (HuggingFace - not recommended for dental) ──
    FLORENCE_MODEL_NAME: str = "microsoft/Florence-2-large"

    # ========================================================================
    # CLOUD MODELS - API (Paid/Free Tier)
    # ========================================================================

    # ── OpenAI Settings ──
    OPENAI_API_KEY: str = Field(default="", env="OPENAI_API_KEY")
    OPENAI_VISION_MODEL: str = "gpt-4o"  # or "gpt-4-vision-preview"

    # ── Anthropic Settings ──
    ANTHROPIC_API_KEY: str = Field(default="", env="ANTHROPIC_API_KEY")
    CLAUDE_VISION_MODEL: str = "claude-3-haiku-20240307"

    # ── Groq Settings (NEW - Ultra-fast LPU inference) ──
    # Free tier: 7,000 requests/day!
    # Models: meta-llama/llama-4-maverick-17b-128e-instruct (best), meta-llama/llama-4-scout-17b-16e-instruct (fast)
    GROQ_API_KEY: str = Field(default="", env="GROQ_API_KEY")
    GROQ_VISION_MODEL: str = (
        "meta-llama/llama-4-scout-17b-16e-instruct"  # or "meta-llama/llama-4-maverick-17b-128e-instruct" foe complex reasoning
    )

    # ── Gemini Settings (NEW - Google's multimodal) ──
    # Free tier: 1,500 requests/day
    # Models: gemini-2.5-flash (stable), gemini-2.5-pro, gemini-1.5-pro (best for reasoning), gemini-1.5-flash (fast)
    GEMINI_API_KEY: str = Field(default="", env="GEMINI_API_KEY")
    GEMINI_VISION_MODEL: str = "gemini-2.5-flash"  # Fast, cheap, good

    # ========================================================================
    # IMAGE PROCESSING
    # ========================================================================
    MAX_IMAGE_SIZE: int = 1024  # Max dimension for preprocessing
    SUPPORTED_FORMATS: list = ["image/jpeg", "image/png", "image/webp"]

    # X-ray contrast enhancement (experimental)
    ENHANCE_CONTRAST: bool = False
    CONTRAST_FACTOR: float = 1.5
    BRIGHTNESS_FACTOR: float = 1.2

    # ========================================================================
    # ANALYSIS SETTINGS
    # ========================================================================
    VISION_TEMPERATURE: float = 0.0  # Deterministic for clinical use
    VISION_MAX_TOKENS: int = 1500
    REQUEST_TIMEOUT: int = 30

    # Include clinical notes in vision prompt
    INCLUDE_CLINICAL_NOTES_IN_VISION_MODEL_PROMPT: bool = True

    # Dual-prompt analysis (detailed + pathology-focused)
    DUAL_PROMPT_ANALYSIS: bool = True

    # ========================================================================
    # HELPER PROPERTIES
    # ========================================================================
    @property
    def current_vision_model(self) -> str:
        """Get the active model name for display"""
        provider_map = {
            "llava": self.LLAVA_MODEL,
            "gemma3": self.GEMMA3_MODEL,
            "llava_med": self.LLAVA_MED_MODEL,
            "biomedclip": self.BIOMEDCLIP_MODEL,
            "florence": self.FLORENCE_MODEL_NAME,
            "gpt4v": self.OPENAI_VISION_MODEL,
            "claude": self.CLAUDE_VISION_MODEL,
            "groq": self.GROQ_VISION_MODEL,
            "gemini": self.GEMINI_VISION_MODEL,
        }
        return provider_map.get(self.VISION_MODEL_PROVIDER, "unknown")

    class Config:
        env_file = ".env"
        extra = "ignore"

vision_settings = VisionSettings()
</file>

<file path="scripts/ingest_documents.py">
#!/usr/bin/env python3

# scripts/ingest_documents.py

# to run the script, run the following command:

# python scripts/ingest_documents.py

"""
Document Ingestion Script
Loads PDF, splits into chunks, embeds, and stores in ChromaDB
"""
import logging
import shutil
import sys
import time
from pathlib import Path

# Add project root to path

sys.path.insert(0, str(Path(**file**).parent.parent))

from langchain_chroma import Chroma
from langchain_community.document_loaders import PyPDFLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter

from app.RAGsystem.embeddings import embedding_provider
from config.ragconfig import rag_settings

# Setup logging

logging.basicConfig(
level=logging.INFO, format="%(asctime)s - %(name)s - %(levelname)s - %(message)s"
)
logger = logging.getLogger(**name**)

def ingest_pdf(pdf_path: str = None):
"""
Load PDF, split into chunks, and store in ChromaDB.
"""
start_time = time.time() # Check if PDF exists
if pdf_path is None:
pdf_path = rag_settings.resolved_pdf_path
else:
pdf_path = Path(pdf_path)

    if not pdf_path.exists():
        print("=======================================================================\n")
        logger.error(f"❌ PDF not found at: {pdf_path}")
        logger.error(f"   Please place your clinical guidelines PDF at: {pdf_path}")
        print("=======================================================================\n")
        return False

    print("=======================================================================\n")
    logger.info(f"📄 Loading PDF: {pdf_path}")
    logger.info(f"   File size: {pdf_path.stat().st_size / 1024 / 1024:.2f} MB")

    # Load PDF
    try:
        loader = PyPDFLoader(str(pdf_path))
        documents = loader.load()
        logger.info(f"✅Loaded {len(documents)} pages from PDF")
        print("=======================================================================\n")
    except Exception as e:
        logger.error(f"❌ Failed to load PDF: {e}")
        print("=======================================================================\n")
        return False

    # Split into chunks
    print("=======================================================================\n")
    logger.info(f"✂️  Splitting into chunks...")
    logger.info(f"   Chunk size: {rag_settings.CHUNK_SIZE}")
    logger.info(f"   Chunk overlap: {rag_settings.CHUNK_OVERLAP}")

    text_splitter = RecursiveCharacterTextSplitter(
        chunk_size=rag_settings.CHUNK_SIZE,
        chunk_overlap=rag_settings.CHUNK_OVERLAP,
        length_function=len,
        separators=["\n\n", "\n", " ", ""],
    )

    chunks = text_splitter.split_documents(documents)
    logger.info(f"✅Created {len(chunks)} chunks")
    print("=======================================================================\n")

    # Add metadata
    for i, chunk in enumerate(chunks):
        chunk.metadata["chunk_id"] = i
        chunk.metadata["source"] = pdf_path.name
        # Preserve page number from PyPDF
        if "page" not in chunk.metadata:
            chunk.metadata["page"] = 0

    # Get embeddings
    print("=======================================================================\n")
    logger.info(f"🔤 Initializing embeddings...")
    logger.info(f"   Provider: {rag_settings.EMBEDDING_PROVIDER}")
    logger.info(f"   Model: {rag_settings.current_embedding_model}")

    embeddings = embedding_provider.get_embeddings()

    # Create/update ChromaDB
    persist_dir = rag_settings.resolved_persist_dir
    logger.info(f"💾 Initializing vector store...")
    logger.info(f"   Directory: {persist_dir}")
    print("=======================================================================\n")

    try:
        # Initialize the client first to manage collections
        import chromadb
        client = chromadb.PersistentClient(path=str(persist_dir))

        # Clear existing collection if it exists to ensure a fresh start
        collection_name = "langchain" # Default name used by LangChain Chroma
        try:
            # Check if collection exists and delete it
            collections = client.list_collections()
            if any(c.name == collection_name for c in collections):
                logger.info(f"   Clearing existing collection: {collection_name}")
                client.delete_collection(name=collection_name)
                logger.info(f"   Collection cleared.")
        except Exception as e:
            logger.warning(f"   Could not clear collection: {e}")

        # Now create the LangChain Chroma instance using the existing client
        vectorstore = Chroma(
            client=client,
            collection_name=collection_name,
            embedding_function=embeddings
        )

        # Add documents to the cleared/new collection
        vectorstore.add_documents(documents=chunks)

        logger.info(f"✅Vector store updated successfully")

        # Verify
        collection = vectorstore._collection
        count = collection.count()
        logger.info(f"✅Stored {count} chunks in ChromaDB")
        print("=======================================================================\n")

        # Test retrieval
        print("=======================================================================\n")
        logger.info(f"\n🧪 Testing retrieval...")
        test_query = "treatment for pulpitis"
        results = vectorstore.similarity_search(test_query, k=3)

        logger.info(f"✅Test query: '{test_query}'")
        logger.info(f"✅Retrieved {len(results)} chunks")
        for i, doc in enumerate(results, 1):
            page = doc.metadata.get("page", "Unknown")
            preview = doc.page_content[:100].replace("\n", " ")
            logger.info(f"   [{i}] Page {page}: {preview}...")
            print("=======================================================================\n")

        print("=======================================================================\n")
        logger.info(f"\n✅ INGESTION COMPLETE!")
        logger.info(f"   Total chunks: {count}")
        logger.info(f"   Persist directory: {persist_dir}")
        logger.info(f"   Ready for retrieval!")

        end_time = time.time()
        duration = end_time - start_time
        num_pages = len(documents)
        logger.info(f"⏱️ Ingestion took {duration:.2f} seconds for {num_pages} pages.")
        print("=======================================================================\n")

        return True

    except Exception as e:
        print("=======================================================================\n")
        logger.error(f"❌ Failed to create vector store: {e}", exc_info=True)
        print("=======================================================================\n")
        return False

if **name** == "**main**":
print("\n" + "=" _ 60)
print(" dentoplus - DOCUMENT INGESTION")
print("=" _ 60 + "\n")
print("=======================================================================\n")

    # Check for CLI argument
    pdf_to_ingest = None
    if len(sys.argv) > 1:
        pdf_to_ingest = sys.argv[1]
        logger.info(f"📌 Using PDF from argument: {pdf_to_ingest}")

    success = ingest_pdf(pdf_to_ingest)

    if success:
        print("\n" + "=" * 60)
        print("   ✅ SUCCESS - Knowledge base ready!")
        print("=" * 60 + "\n")
        print("=======================================================================\n")
        sys.exit(0)
    else:
        print("\n" + "=" * 60)
        print("   ❌ FAILED - Check errors above")
        print("=" * 60 + "\n")
        print("=======================================================================\n")
        sys.exit(1)

</file>

<file path="evaluation_results.json">
[
  {
    "model_name": "florence",
    "test_id": "florence_20260313_023543",
    "timestamp": "2026-03-13T02:35:43.610277",
    "inference_time_ms": 38458.773136138916,
    "peak_ram_mb": 1398.671875,
    "time_to_first_token_ms": null,
    "tokens_per_second": 1.7941289951124864,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": 1.6,
    "cache_size_mb": null,
    "input_tokens": null,
    "output_tokens": 69,
    "total_tokens": 69,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.5,
    "image_quality_score": 0.5,
    "primary_finding": "The image is a black and white X-ray of a tooth. The tooth appears to be in good condition, with no visible cavities or cavities. The X-rays are arranged in a rectangular shape, with the topmost tooth...",
    "teeth_visible": null,
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "minimal",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "biomedclip",
    "test_id": "biomedclip_20260313_023552",
    "timestamp": "2026-03-13T02:35:52.099411",
    "inference_time_ms": 8384.558200836182,
    "peak_ram_mb": 1535.4375,
    "time_to_first_token_ms": null,
    "tokens_per_second": 5.247742212059776,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": 0.5,
    "cache_size_mb": null,
    "input_tokens": null,
    "output_tokens": 44,
    "total_tokens": 44,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.598285973072052,
    "image_quality_score": 0.5,
    "primary_finding": "dental caries",
    "teeth_visible": null,
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "minimal",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "llava:latest",
    "test_id": "llava:latest_20260313_023620",
    "timestamp": "2026-03-13T02:36:20.391926",
    "inference_time_ms": 28188.11583518982,
    "peak_ram_mb": 93.640625,
    "time_to_first_token_ms": null,
    "tokens_per_second": 16.07060232931491,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": 4.7,
    "cache_size_mb": null,
    "input_tokens": 1738,
    "output_tokens": 453,
    "total_tokens": 2191,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.8,
    "image_quality_score": 0.8,
    "primary_finding": "The primary finding on tooth 47 is the presence of mild caries on the crown.",
    "teeth_visible": [
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "minimal",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "llava:13b",
    "test_id": "llava:13b_20260313_023716",
    "timestamp": "2026-03-13T02:37:16.712453",
    "inference_time_ms": 56203.1672000885,
    "peak_ram_mb": 18.765625,
    "time_to_first_token_ms": null,
    "tokens_per_second": 8.807332836559796,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.2,
    "avg_gpu_percent": null,
    "model_size_gb": 7.4,
    "cache_size_mb": null,
    "input_tokens": 1737,
    "output_tokens": 495,
    "total_tokens": 2232,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.7,
    "image_quality_score": 0.8,
    "primary_finding": "Moderate caries and periapical abscess on tooth 47",
    "teeth_visible": [
      "47"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "recommended",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "llama3.2-vision",
    "test_id": "llama3.2-vision_20260313_023827",
    "timestamp": "2026-03-13T02:38:27.280089",
    "inference_time_ms": 70430.89604377747,
    "peak_ram_mb": 18.265625,
    "time_to_first_token_ms": null,
    "tokens_per_second": 4.997806641295728,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": 7.9,
    "cache_size_mb": null,
    "input_tokens": 921,
    "output_tokens": 352,
    "total_tokens": 1273,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.7,
    "image_quality_score": 0.8,
    "primary_finding": "Periapical abscess on tooth 47",
    "teeth_visible": [
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "recommended",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "llava_med",
    "test_id": "llava_med_20260313_023855",
    "timestamp": "2026-03-13T02:38:55.100925",
    "inference_time_ms": 27699.98002052307,
    "peak_ram_mb": 83.515625,
    "time_to_first_token_ms": null,
    "tokens_per_second": 19.747306662124835,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": 7.4,
    "cache_size_mb": null,
    "input_tokens": 1862,
    "output_tokens": 547,
    "total_tokens": 2409,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.8,
    "image_quality_score": 0.8,
    "primary_finding": "Periapical abscess and mild bone loss around tooth 47",
    "teeth_visible": [
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "recommended",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "gemma3",
    "test_id": "gemma3_20260313_023920",
    "timestamp": "2026-03-13T02:39:20.662269",
    "inference_time_ms": 25445.322036743164,
    "peak_ram_mb": 18.625,
    "time_to_first_token_ms": null,
    "tokens_per_second": 20.986175739057323,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": 2.5,
    "cache_size_mb": null,
    "input_tokens": 1205,
    "output_tokens": 534,
    "total_tokens": 1739,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.85,
    "image_quality_score": 0.9,
    "primary_finding": "Extensive caries affecting the crown of tooth 47 with associated periapical inflammation.",
    "teeth_visible": [
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "minimal",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "groq",
    "test_id": "groq_20260313_023925",
    "timestamp": "2026-03-13T02:39:25.474899",
    "inference_time_ms": 4701.628923416138,
    "peak_ram_mb": 89.8125,
    "time_to_first_token_ms": null,
    "tokens_per_second": 84.01343586107568,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": null,
    "cache_size_mb": null,
    "input_tokens": 2734,
    "output_tokens": 395,
    "total_tokens": 3129,
    "cost_usd": 0.0005632199999999999,
    "cost_per_token": 1.7999999999999997e-07,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.8,
    "image_quality_score": 0.9,
    "primary_finding": "periapical radiolucency around root tip of tooth 47 and caries on tooth 46",
    "teeth_visible": [
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "cloud",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "gemini",
    "test_id": "gemini_20260313_023940",
    "timestamp": "2026-03-13T02:39:40.809282",
    "inference_time_ms": 15227.279901504517,
    "peak_ram_mb": 184.5625,
    "time_to_first_token_ms": null,
    "tokens_per_second": 51.48654290662495,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": null,
    "cache_size_mb": null,
    "input_tokens": 1171,
    "output_tokens": 784,
    "total_tokens": 1955,
    "cost_usd": 0.00032302499999999995,
    "cost_per_token": 1.6523017902813297e-07,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.95,
    "image_quality_score": 0.9,
    "primary_finding": "Severe recurrent caries on tooth 47 extending into the pulp, complicated by a large periapical radiolucency indicative of an acute apical abscess.",
    "teeth_visible": [
      "48",
      "47",
      "46",
      "45"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "cloud",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "claude",
    "test_id": "claude_20260313_023947",
    "timestamp": "2026-03-13T02:39:47.242508",
    "inference_time_ms": 6325.575828552246,
    "peak_ram_mb": 189.140625,
    "time_to_first_token_ms": null,
    "tokens_per_second": 71.45594523739203,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": null,
    "cache_size_mb": null,
    "input_tokens": 2007,
    "output_tokens": 452,
    "total_tokens": 2459,
    "cost_usd": 0.024801,
    "cost_per_token": 1.0085807238714925e-05,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.7,
    "image_quality_score": 0.8,
    "primary_finding": "Moderate caries, periapical abscess, and moderate vertical bone loss associated with tooth #47",
    "teeth_visible": [
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "cloud",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "gpt4v",
    "test_id": "gpt4v_20260313_023959",
    "timestamp": "2026-03-13T02:39:59.548416",
    "inference_time_ms": 12195.32299041748,
    "peak_ram_mb": 131.875,
    "time_to_first_token_ms": null,
    "tokens_per_second": 32.881457942121514,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": null,
    "cache_size_mb": null,
    "input_tokens": 1616,
    "output_tokens": 401,
    "total_tokens": 2017,
    "cost_usd": 0.0157,
    "cost_per_token": 7.783837382250868e-06,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.7,
    "image_quality_score": 0.8,
    "primary_finding": "Severe caries on the occlusal surface and periapical radiolucency on tooth 47",
    "teeth_visible": [
      "45",
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "cloud",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "florence",
    "test_id": "florence_20260313_030750",
    "timestamp": "2026-03-13T03:07:50.997378",
    "inference_time_ms": 41565.778970718384,
    "peak_ram_mb": 1366.609375,
    "time_to_first_token_ms": null,
    "tokens_per_second": 1.66001941281091,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": 1.6,
    "cache_size_mb": null,
    "input_tokens": null,
    "output_tokens": 69,
    "total_tokens": 69,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.5,
    "image_quality_score": 0.5,
    "primary_finding": "The image is a black and white X-ray of a tooth. The tooth appears to be in good condition, with no visible cavities or cavities. The X-rays are arranged in a rectangular shape, with the topmost tooth...",
    "teeth_visible": null,
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "minimal",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "biomedclip",
    "test_id": "biomedclip_20260313_030759",
    "timestamp": "2026-03-13T03:07:59.144003",
    "inference_time_ms": 8041.047096252441,
    "peak_ram_mb": 1261.21875,
    "time_to_first_token_ms": null,
    "tokens_per_second": 5.471924175211752,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": 0.5,
    "cache_size_mb": null,
    "input_tokens": null,
    "output_tokens": 44,
    "total_tokens": 44,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.598285973072052,
    "image_quality_score": 0.5,
    "primary_finding": "dental caries",
    "teeth_visible": null,
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "minimal",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "llava:latest",
    "test_id": "llava:latest_20260313_030826",
    "timestamp": "2026-03-13T03:08:26.966636",
    "inference_time_ms": 27718.459844589233,
    "peak_ram_mb": 51.171875,
    "time_to_first_token_ms": null,
    "tokens_per_second": 16.342899372471,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": 4.7,
    "cache_size_mb": null,
    "input_tokens": 1738,
    "output_tokens": 453,
    "total_tokens": 2191,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.8,
    "image_quality_score": 0.8,
    "primary_finding": "The primary finding on tooth 47 is the presence of mild caries on the crown.",
    "teeth_visible": [
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "minimal",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "llava:13b",
    "test_id": "llava:13b_20260313_030922",
    "timestamp": "2026-03-13T03:09:22.669573",
    "inference_time_ms": 55592.007875442505,
    "peak_ram_mb": 20.484375,
    "time_to_first_token_ms": null,
    "tokens_per_second": 8.904157610372332,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.2,
    "avg_gpu_percent": null,
    "model_size_gb": 7.4,
    "cache_size_mb": null,
    "input_tokens": 1737,
    "output_tokens": 495,
    "total_tokens": 2232,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.7,
    "image_quality_score": 0.8,
    "primary_finding": "Moderate caries and periapical abscess on tooth 47",
    "teeth_visible": [
      "47"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "recommended",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "llama3.2-vision",
    "test_id": "llama3.2-vision_20260313_031036",
    "timestamp": "2026-03-13T03:10:36.730830",
    "inference_time_ms": 73933.32815170288,
    "peak_ram_mb": 19.671875,
    "time_to_first_token_ms": null,
    "tokens_per_second": 4.761046321054769,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": 7.9,
    "cache_size_mb": null,
    "input_tokens": 921,
    "output_tokens": 352,
    "total_tokens": 1273,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.7,
    "image_quality_score": 0.8,
    "primary_finding": "Periapical abscess on tooth 47",
    "teeth_visible": [
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "recommended",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "llava_med",
    "test_id": "llava_med_20260313_031104",
    "timestamp": "2026-03-13T03:11:04.463822",
    "inference_time_ms": 27610.857009887695,
    "peak_ram_mb": 89.015625,
    "time_to_first_token_ms": null,
    "tokens_per_second": 16.877418902032677,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": 7.4,
    "cache_size_mb": null,
    "input_tokens": 1862,
    "output_tokens": 466,
    "total_tokens": 2328,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.8,
    "image_quality_score": 0.8,
    "primary_finding": "The primary finding on tooth 47 is a small carious lesion on the crown.",
    "teeth_visible": [
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "recommended",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "gemma3",
    "test_id": "gemma3_20260313_031130",
    "timestamp": "2026-03-13T03:11:30.743473",
    "inference_time_ms": 26169.458866119385,
    "peak_ram_mb": 21.5625,
    "time_to_first_token_ms": null,
    "tokens_per_second": 20.405465880356807,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": 2.5,
    "cache_size_mb": null,
    "input_tokens": 1205,
    "output_tokens": 534,
    "total_tokens": 1739,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.85,
    "image_quality_score": 0.9,
    "primary_finding": "Extensive caries affecting the crown of tooth 47 with associated periapical inflammation.",
    "teeth_visible": [
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "minimal",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "groq",
    "test_id": "groq_20260313_031135",
    "timestamp": "2026-03-13T03:11:35.067652",
    "inference_time_ms": 4213.385105133057,
    "peak_ram_mb": 91.21875,
    "time_to_first_token_ms": null,
    "tokens_per_second": 96.83425317636983,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": null,
    "cache_size_mb": null,
    "input_tokens": 2734,
    "output_tokens": 408,
    "total_tokens": 3142,
    "cost_usd": 0.00056556,
    "cost_per_token": 1.8e-07,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.9,
    "image_quality_score": 0.8,
    "primary_finding": "periapical radiolucency around tooth 47 and large caries on tooth 46",
    "teeth_visible": [
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "cloud",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "gemini",
    "test_id": "gemini_20260313_031152",
    "timestamp": "2026-03-13T03:11:52.614244",
    "inference_time_ms": 17439.740896224976,
    "peak_ram_mb": 172.03125,
    "time_to_first_token_ms": null,
    "tokens_per_second": 38.70470347103099,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.0,
    "avg_gpu_percent": null,
    "model_size_gb": null,
    "cache_size_mb": null,
    "input_tokens": 1171,
    "output_tokens": 675,
    "total_tokens": 1846,
    "cost_usd": 0.000290325,
    "cost_per_token": 1.5727248104008669e-07,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.9,
    "image_quality_score": 0.9,
    "primary_finding": "Extensive deep caries with pulp involvement and a large periapical radiolucency consistent with a chronic periapical abscess associated with tooth 47.",
    "teeth_visible": [
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "cloud",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "claude",
    "test_id": "claude_20260313_031158",
    "timestamp": "2026-03-13T03:11:58.918526",
    "inference_time_ms": 6199.95903968811,
    "peak_ram_mb": 193.984375,
    "time_to_first_token_ms": null,
    "tokens_per_second": 72.09725050417211,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": null,
    "cache_size_mb": null,
    "input_tokens": 2007,
    "output_tokens": 447,
    "total_tokens": 2454,
    "cost_usd": 0.024726,
    "cost_per_token": 1.0075794621026896e-05,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.7,
    "image_quality_score": 0.8,
    "primary_finding": "Moderate caries, periapical abscess, and moderate vertical bone loss associated with tooth #47",
    "teeth_visible": [
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "cloud",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "gpt4v",
    "test_id": "gpt4v_20260313_031226",
    "timestamp": "2026-03-13T03:12:26.563052",
    "inference_time_ms": 27535.63904762268,
    "peak_ram_mb": 35.828125,
    "time_to_first_token_ms": null,
    "tokens_per_second": 14.92611082274788,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": null,
    "cache_size_mb": null,
    "input_tokens": 1616,
    "output_tokens": 411,
    "total_tokens": 2027,
    "cost_usd": 0.0158,
    "cost_per_token": 7.794770596941294e-06,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.7,
    "image_quality_score": 0.8,
    "primary_finding": "Severe caries on the occlusal surface and periapical radiolucency at the root tip of tooth 47",
    "teeth_visible": [
      "45",
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "cloud",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "florence",
    "test_id": "florence_20260313_131629",
    "timestamp": "2026-03-13T13:16:29.817858",
    "inference_time_ms": 40679.78000640869,
    "peak_ram_mb": 1233.78125,
    "time_to_first_token_ms": null,
    "tokens_per_second": 1.6961743644909029,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": 1.6,
    "cache_size_mb": null,
    "input_tokens": null,
    "output_tokens": 69,
    "total_tokens": 69,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.5,
    "image_quality_score": 0.5,
    "primary_finding": "The image is a black and white X-ray of a tooth. The tooth appears to be in good condition, with no visible cavities or cavities. The X-rays are arranged in a rectangular shape, with the topmost tooth...",
    "teeth_visible": null,
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "minimal",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "biomedclip",
    "test_id": "biomedclip_20260313_131634",
    "timestamp": "2026-03-13T13:16:34.363939",
    "inference_time_ms": 4430.078983306885,
    "peak_ram_mb": 823.296875,
    "time_to_first_token_ms": null,
    "tokens_per_second": 9.932102828368915,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": 0.5,
    "cache_size_mb": null,
    "input_tokens": null,
    "output_tokens": 44,
    "total_tokens": 44,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.598285973072052,
    "image_quality_score": 0.5,
    "primary_finding": "dental caries",
    "teeth_visible": null,
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "minimal",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "llava:latest",
    "test_id": "llava:latest_20260313_131703",
    "timestamp": "2026-03-13T13:17:03.988633",
    "inference_time_ms": 29512.518882751465,
    "peak_ram_mb": 36.578125,
    "time_to_first_token_ms": null,
    "tokens_per_second": 15.349418387488265,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": 4.7,
    "cache_size_mb": null,
    "input_tokens": 1738,
    "output_tokens": 453,
    "total_tokens": 2191,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.8,
    "image_quality_score": 0.8,
    "primary_finding": "The primary finding on tooth 47 is the presence of mild caries on the crown.",
    "teeth_visible": [
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "minimal",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "llava:13b",
    "test_id": "llava:13b_20260313_131806",
    "timestamp": "2026-03-13T13:18:07.006762",
    "inference_time_ms": 62838.03296089172,
    "peak_ram_mb": 19.9375,
    "time_to_first_token_ms": null,
    "tokens_per_second": 7.877394894077466,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.2,
    "avg_gpu_percent": null,
    "model_size_gb": 7.4,
    "cache_size_mb": null,
    "input_tokens": 1737,
    "output_tokens": 495,
    "total_tokens": 2232,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.7,
    "image_quality_score": 0.8,
    "primary_finding": "Moderate caries and periapical abscess on tooth 47",
    "teeth_visible": [
      "47"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "recommended",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "llama3.2-vision",
    "test_id": "llama3.2-vision_20260313_131929",
    "timestamp": "2026-03-13T13:19:29.224909",
    "inference_time_ms": 81926.47695541382,
    "peak_ram_mb": 13.109375,
    "time_to_first_token_ms": null,
    "tokens_per_second": 4.296535297026945,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.3,
    "avg_gpu_percent": null,
    "model_size_gb": 7.9,
    "cache_size_mb": null,
    "input_tokens": 921,
    "output_tokens": 352,
    "total_tokens": 1273,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.7,
    "image_quality_score": 0.8,
    "primary_finding": "Periapical abscess on tooth 47",
    "teeth_visible": [
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "recommended",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "llava_med",
    "test_id": "llava_med_20260313_132003",
    "timestamp": "2026-03-13T13:20:03.393461",
    "inference_time_ms": 33855.27300834656,
    "peak_ram_mb": 22.125,
    "time_to_first_token_ms": null,
    "tokens_per_second": 13.882623244069775,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": 7.4,
    "cache_size_mb": null,
    "input_tokens": 1862,
    "output_tokens": 470,
    "total_tokens": 2332,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.8,
    "image_quality_score": 0.8,
    "primary_finding": "The primary finding on tooth 47 is a small carious lesion on the crown.",
    "teeth_visible": [
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "recommended",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "gemma3",
    "test_id": "gemma3_20260313_132030",
    "timestamp": "2026-03-13T13:20:30.563979",
    "inference_time_ms": 27063.286066055298,
    "peak_ram_mb": 21.15625,
    "time_to_first_token_ms": null,
    "tokens_per_second": 19.731528488322816,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": 2.5,
    "cache_size_mb": null,
    "input_tokens": 1205,
    "output_tokens": 534,
    "total_tokens": 1739,
    "cost_usd": 0.0,
    "cost_per_token": 0.0,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.85,
    "image_quality_score": 0.9,
    "primary_finding": "Extensive caries affecting the crown of tooth 47 with associated periapical inflammation.",
    "teeth_visible": [
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "minimal",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "groq",
    "test_id": "groq_20260313_132034",
    "timestamp": "2026-03-13T13:20:34.448221",
    "inference_time_ms": 3773.6778259277344,
    "peak_ram_mb": 50.578125,
    "time_to_first_token_ms": null,
    "tokens_per_second": 115.27216155318136,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.1,
    "avg_gpu_percent": null,
    "model_size_gb": null,
    "cache_size_mb": null,
    "input_tokens": 2734,
    "output_tokens": 435,
    "total_tokens": 3169,
    "cost_usd": 0.00057042,
    "cost_per_token": 1.8e-07,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.9,
    "image_quality_score": 0.8,
    "primary_finding": "Periapical radiolucency around tooth 47 and caries on tooth 46",
    "teeth_visible": [
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "cloud",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "claude",
    "test_id": "claude_20260313_132043",
    "timestamp": "2026-03-13T13:20:43.261491",
    "inference_time_ms": 6214.792013168335,
    "peak_ram_mb": 65.25,
    "time_to_first_token_ms": null,
    "tokens_per_second": 80.45321531928423,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.2,
    "avg_gpu_percent": null,
    "model_size_gb": null,
    "cache_size_mb": null,
    "input_tokens": 2007,
    "output_tokens": 500,
    "total_tokens": 2507,
    "cost_usd": 0.025521000000000002,
    "cost_per_token": 1.0179896290386917e-05,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.7,
    "image_quality_score": 0.8,
    "primary_finding": "Moderate caries on the occlusal surface of tooth #47 with a periapical radiolucent lesion and moderate vertical bone loss between teeth #46 and #47",
    "teeth_visible": [
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "cloud",
    "context_provided": true,
    "tooth_number_provided": true
  },
  {
    "model_name": "gpt4v",
    "test_id": "gpt4v_20260313_132053",
    "timestamp": "2026-03-13T13:20:53.272571",
    "inference_time_ms": 9899.659872055054,
    "peak_ram_mb": 70.453125,
    "time_to_first_token_ms": null,
    "tokens_per_second": 40.00137430153901,
    "concurrent_requests": 1,
    "queue_wait_time_ms": null,
    "avg_ram_mb": null,
    "gpu_memory_mb": null,
    "avg_cpu_percent": 0.3,
    "avg_gpu_percent": null,
    "model_size_gb": null,
    "cache_size_mb": null,
    "input_tokens": 1616,
    "output_tokens": 396,
    "total_tokens": 2012,
    "cost_usd": 0.01565,
    "cost_per_token": 7.778330019880716e-06,
    "response_complete": true,
    "schema_valid": true,
    "citation_count": 0,
    "avg_citation_quality": null,
    "confidence_score": 0.7,
    "image_quality_score": 0.8,
    "primary_finding": "Severe caries on the occlusal surface of tooth 47 with periapical radiolucency indicating possible abscess.",
    "teeth_visible": [
      "45",
      "46",
      "47",
      "48"
    ],
    "request_success": true,
    "error_type": null,
    "error_message": null,
    "retry_count": 0,
    "test_type": "vision_analysis",
    "model_provider": "local",
    "hardware_tier": "cloud",
    "context_provided": true,
    "tooth_number_provided": true
  }
]
</file>

</files>
