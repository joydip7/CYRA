# Design Document: CYRA - AI Career & Autonomous Inbox Agent

## Overview

CYRA is a voice-first AI-powered email management system that combines intelligent classification, autonomous decision-making, and career intelligence extraction. The system follows a layered architecture with clear separation between presentation (React frontend), business logic (FastAPI backend), AI reasoning (Google Gemini + LangGraph), and external services (Gmail API).

The design emphasizes user safety through explicit confirmation for destructive actions, maintains privacy through in-memory processing, and provides transparency through comprehensive action logging. The system uses LangGraph for agent orchestration, enabling complex multi-step workflows while maintaining state consistency.

## Architecture

### High-Level System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Frontend Layer                          │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐     │
│  │ Voice UI     │  │ Dashboard    │  │ Email Management   │     │
│  │ (Web Speech) │  │ (Analytics)  │  │ (Classification)   │     │
│  └──────────────┘  └──────────────┘  └────────────────────┘     │
│                    React.js + Tailwind CSS                      │
└────────────────────────────┬────────────────────────────────────┘
                             │ REST API (HTTPS)
┌────────────────────────────┴────────────────────────────────────┐
│                         Backend Layer                           │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              FastAPI Application Server                  │   │
│  │  ┌────────────┐  ┌────────────┐  ┌─────────────────┐     │   │
│  │  │ Auth       │  │ Email      │  │ Analytics       │     │   │
│  │  │ Controller │  │ Controller │  │ Controller      │     │   │
│  │  └────────────┘  └────────────┘  └─────────────────┘     │   │
│  └──────────────────────────────────────────────────────────┘   │
│                             │                                   │
│  ┌──────────────────────────┴───────────────────────────────┐   │
│  │              LangGraph Agent Orchestrator                │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐   │   │
│  │  │Classifier│  │ Career   │  │Autonomous│  │  Cold   │   │   │
│  │  │  Agent   │  │Intel Agent│  │  Agent   │  │Email Opt│  │   │
│  │  └──────────┘  └──────────┘  └──────────┘  └─────────┘   │   │
│  └──────────────────────────────────────────────────────────┘   │
└────────────────────────────┬────────────────────────────────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
┌───────▼────────┐  ┌────────▼────────┐  ┌───────▼────────┐
│  Google Gemini │  │   Gmail API     │  │   Database     │
│      API       │  │  (OAuth 2.0)    │  │  (PostgreSQL/  │
│ (Classification│  │                 │  │    SQLite)     │
│  & Extraction) │  │                 │  │                │
└────────────────┘  └─────────────────┘  └────────────────┘
```

### Component Responsibilities

**Frontend Layer:**
- Voice Interface: Handles speech-to-text input and text-to-speech output
- Dashboard: Displays analytics and productivity metrics
- Email Management: Shows classified emails with actions and career intelligence
- Session Management: Maintains conversation context in browser state

**Backend Layer:**
- FastAPI Controllers: Handle HTTP requests, validation, and response formatting
- LangGraph Orchestrator: Routes requests to appropriate agents and manages state
- Classifier Agent: Categorizes emails using Gemini API
- Career Intelligence Agent: Extracts structured data from job emails
- Autonomous Agent: Executes automated inbox actions based on rules
- Cold Email Optimizer: Generates and scores outreach emails

**External Services:**
- Google Gemini API: Provides LLM capabilities for classification and extraction
- Gmail API: Provides email access and modification capabilities
- Database: Stores user preferences, action logs, and analytics

## Components and Interfaces

### 1. Authentication Service

**Purpose:** Manages Google OAuth 2.0 authentication and token lifecycle.

**Interface:**
```python
class AuthService:
    def initiate_oauth_flow() -> OAuthURL:
        """Generate OAuth authorization URL with required scopes"""
        
    def handle_oauth_callback(code: str) -> AuthTokens:
        """Exchange authorization code for access and refresh tokens"""
        
    def refresh_access_token(refresh_token: str) -> AccessToken:
        """Obtain new access token using refresh token"""
        
    def revoke_access(user_id: str) -> bool:
        """Revoke OAuth tokens and clear user session"""
        
    def validate_token(access_token: str) -> bool:
        """Verify token validity and expiration"""
```

**Required OAuth Scopes:**
- `https://www.googleapis.com/auth/gmail.readonly` - Read email content
- `https://www.googleapis.com/auth/gmail.modify` - Modify labels, archive, star
- `https://www.googleapis.com/auth/gmail.labels` - Manage custom labels

### 2. Email Classifier Agent

**Purpose:** Categorizes emails into predefined categories using AI.

**Interface:**
```python
class EmailClassifier:
    def classify_email(email: Email) -> Classification:
        """
        Classify single email into category with confidence score
        Returns: Classification(category, confidence, reasoning)
        """
        
    def classify_batch(emails: List[Email]) -> List[Classification]:
        """Classify multiple emails in parallel (up to 50)"""
        
    def get_category_summary(classifications: List[Classification]) -> CategorySummary:
        """Aggregate classifications into category counts"""
```

**Classification Schema:**
```python
class Classification:
    category: Literal["Urgent", "Job/Internship", "Promotional", "Meeting", "General"]
    confidence: float  # 0.0 to 1.0
    reasoning: str     # Brief explanation of classification
    email_id: str
    timestamp: datetime
```

**Classification Logic:**
- Priority order: Urgent > Job/Internship > Meeting > Promotional > General
- Urgent signals: "ASAP", "urgent", "deadline today", "immediate action"
- Job signals: "job opening", "internship", "career opportunity", "apply now"
- Meeting signals: "calendar invite", "meeting request", "schedule"
- Promotional signals: "unsubscribe", "promotional", "offer", "discount"

### 3. Career Intelligence Engine

**Purpose:** Extracts structured data from job/internship emails.

**Interface:**
```python
class CareerIntelligenceEngine:
    def extract_job_data(email: Email) -> JobOpportunity:
        """Extract structured fields from job email"""
        
    def extract_multiple_opportunities(email: Email) -> List[JobOpportunity]:
        """Handle emails containing multiple job postings"""
        
    def normalize_date(date_str: str) -> datetime:
        """Convert various date formats to ISO 8601"""
        
    def normalize_compensation(comp_str: str) -> Compensation:
        """Parse and normalize salary/stipend information"""
```

**Job Opportunity Schema:**
```python
class JobOpportunity:
    role: Optional[str]
    company: Optional[str]
    deadline: Optional[datetime]  # ISO 8601 format
    location: Optional[str]
    compensation: Optional[Compensation]
    required_skills: List[str]
    email_id: str
    extraction_confidence: float
    
class Compensation:
    amount: Optional[float]
    currency: str  # ISO 4217 code (USD, EUR, INR)
    period: Literal["hourly", "monthly", "yearly", "one-time"]
```

### 4. Autonomous Agent

**Purpose:** Executes automated inbox actions based on classification rules.

**Interface:**
```python
class AutonomousAgent:
    def is_enabled(user_id: str) -> bool:
        """Check if autonomous mode is enabled for user"""
        
    def enable_autonomous_mode(user_id: str) -> bool:
        """Enable autonomous decision making"""
        
    def disable_autonomous_mode(user_id: str) -> bool:
        """Disable autonomous decision making"""
        
    def execute_action(email_id: str, classification: Classification) -> ActionResult:
        """Execute appropriate action based on classification"""
        
    def log_action(action: Action) -> bool:
        """Record action in audit log"""
```

**Action Rules:**
```python
AUTONOMOUS_RULES = {
    "Promotional": "archive",
    "Job/Internship": "star",
    "Urgent": "highlight"
}
```

**Action Schema:**
```python
class Action:
    action_type: Literal["archive", "star", "highlight", "label"]
    email_id: str
    timestamp: datetime
    user_id: str
    triggered_by: Literal["autonomous", "manual"]
    
class ActionResult:
    success: bool
    action: Action
    error: Optional[str]
```

### 5. Cold Email Optimizer

**Purpose:** Generates and analyzes cold outreach emails.

**Interface:**
```python
class ColdEmailOptimizer:
    def generate_email(context: EmailContext) -> GeneratedEmail:
        """Generate cold email based on provided context"""
        
    def analyze_tone(email_content: str) -> ToneAnalysis:
        """Analyze email tone (professional, casual, aggressive, etc.)"""
        
    def calculate_personalization_score(email_content: str, context: EmailContext) -> int:
        """Score personalization level (0-100)"""
        
    def estimate_response_likelihood(email: GeneratedEmail) -> ResponseLikelihood:
        """Estimate probability of receiving response"""
        
    def regenerate_with_feedback(email: GeneratedEmail, feedback: str) -> GeneratedEmail:
        """Improve email based on user feedback"""
```

**Email Generation Schema:**
```python
class EmailContext:
    recipient_name: Optional[str]
    recipient_company: Optional[str]
    purpose: str
    key_points: List[str]
    tone_preference: Literal["professional", "casual", "friendly"]
    
class GeneratedEmail:
    subject: str
    body: str
    tone_analysis: ToneAnalysis
    personalization_score: int
    response_likelihood: ResponseLikelihood
    
class ToneAnalysis:
    primary_tone: str
    confidence: float
    suggestions: List[str]
    
class ResponseLikelihood:
    level: Literal["Low", "Medium", "High"]
    score: float  # 0.0 to 1.0
    factors: List[str]  # Reasons for the score
```

### 6. Session Manager

**Purpose:** Maintains conversation context across user interactions.

**Interface:**
```python
class SessionManager:
    def create_session(user_id: str) -> Session:
        """Initialize new conversation session"""
        
    def add_interaction(session_id: str, interaction: Interaction) -> bool:
        """Add user command or system response to session"""
        
    def get_context(session_id: str, lookback: int = 10) -> List[Interaction]:
        """Retrieve recent conversation history"""
        
    def resolve_reference(session_id: str, reference: str) -> Optional[str]:
        """Resolve references like 'this email' to actual email ID"""
        
    def clear_session(session_id: str) -> bool:
        """Clear session context (after 30 min inactivity)"""
```

**Session Schema:**
```python
class Session:
    session_id: str
    user_id: str
    created_at: datetime
    last_activity: datetime
    interactions: List[Interaction]
    
class Interaction:
    type: Literal["user_command", "system_response"]
    content: str
    timestamp: datetime
    referenced_emails: List[str]
    structured_data: Optional[dict]
```

### 7. Voice Interface

**Purpose:** Handles speech-to-text and text-to-speech using Web Speech API.

**Interface (Frontend):**
```typescript
class VoiceInterface {
    startListening(): Promise<string>
    // Activate microphone and convert speech to text
    
    stopListening(): void
    // Stop speech recognition
    
    speak(text: string): Promise<void>
    // Convert text to speech and play audio
    
    stopSpeaking(): void
    // Stop current speech synthesis
    
    isSupported(): boolean
    // Check if Web Speech API is available
    
    handleError(error: SpeechRecognitionError): void
    // Handle recognition errors and fallback to text
}
```

**Supported Voice Commands:**
- "Read urgent emails"
- "Show job emails"
- "Archive promotions"
- "What's in my inbox?"
- "Reply to this email politely"
- "Generate cold email for [context]"

### 8. Analytics Dashboard

**Purpose:** Displays productivity metrics and insights.

**Interface:**
```python
class AnalyticsDashboard:
    def get_metrics(user_id: str, time_period: TimePeriod) -> Metrics:
        """Retrieve aggregated metrics for specified period"""
        
    def calculate_time_saved(actions: List[Action]) -> int:
        """Estimate time saved in minutes based on actions"""
        
    def get_category_breakdown(user_id: str, time_period: TimePeriod) -> CategoryBreakdown:
        """Get email distribution across categories"""
```

**Metrics Schema:**
```python
class Metrics:
    total_emails_filtered: int
    career_emails_detected: int
    promotions_archived: int
    time_saved_minutes: int
    period: TimePeriod
    
class TimePeriod:
    type: Literal["today", "week", "month", "all_time"]
    start_date: datetime
    end_date: datetime
    
class CategoryBreakdown:
    urgent: int
    job_internship: int
    promotional: int
    meeting: int
    general: int
```

**Time Saved Calculation:**
- Email classification: 10 seconds saved per email
- Career data extraction: 2 minutes saved per job email
- Promotional archive: 5 seconds saved per email
- Cold email generation: 15 minutes saved per email

### 9. Gmail API Client

**Purpose:** Interfaces with Gmail API for email operations.

**Interface:**
```python
class GmailAPIClient:
    def list_emails(user_id: str, query: str, max_results: int) -> List[Email]:
        """Fetch emails matching query"""
        
    def get_email(user_id: str, email_id: str) -> Email:
        """Retrieve full email content"""
        
    def modify_email(user_id: str, email_id: str, modifications: Modifications) -> bool:
        """Apply labels, archive, star, etc."""
        
    def send_email(user_id: str, email: OutgoingEmail) -> bool:
        """Send email (requires explicit confirmation)"""
        
    def create_label(user_id: str, label_name: str) -> Label:
        """Create custom Gmail label"""
```

**Email Schema:**
```python
class Email:
    id: str
    thread_id: str
    subject: str
    from_address: str
    to_addresses: List[str]
    body: str
    snippet: str
    date: datetime
    labels: List[str]
    
class Modifications:
    add_labels: List[str]
    remove_labels: List[str]
    star: Optional[bool]
    archive: Optional[bool]
```

### 10. LangGraph Agent Orchestrator

**Purpose:** Manages agent state and routes requests to appropriate agents.

**State Graph:**
```python
class AgentState(TypedDict):
    user_id: str
    session_id: str
    user_command: str
    email_ids: List[str]
    classifications: List[Classification]
    job_opportunities: List[JobOpportunity]
    actions_performed: List[Action]
    response: str
    error: Optional[str]
```

**Agent Flow:**
```
START
  ↓
PARSE_COMMAND (Determine intent)
  ↓
  ├─→ CLASSIFY_EMAILS → AUTONOMOUS_ACTIONS → END
  ├─→ EXTRACT_CAREER_DATA → FORMAT_RESPONSE → END
  ├─→ GENERATE_COLD_EMAIL → ANALYZE_EMAIL → END
  ├─→ VOICE_COMMAND → EXECUTE_ACTION → END
  └─→ ANALYTICS_REQUEST → FETCH_METRICS → END
```

**Routing Logic:**
```python
def route_command(state: AgentState) -> str:
    """Determine next agent based on user command"""
    command = state["user_command"].lower()
    
    if "classify" in command or "categorize" in command:
        return "classify_emails"
    elif "job" in command or "career" in command:
        return "extract_career_data"
    elif "cold email" in command or "outreach" in command:
        return "generate_cold_email"
    elif "analytics" in command or "metrics" in command:
        return "analytics_request"
    else:
        return "parse_command"
```

## Data Models

### Database Schema

**Users Table:**
```sql
CREATE TABLE users (
    user_id VARCHAR(255) PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    autonomous_mode_enabled BOOLEAN DEFAULT FALSE,
    preferences JSONB
);
```

**OAuth Tokens Table:**
```sql
CREATE TABLE oauth_tokens (
    user_id VARCHAR(255) PRIMARY KEY REFERENCES users(user_id),
    access_token TEXT NOT NULL,
    refresh_token TEXT NOT NULL,
    token_expiry TIMESTAMP NOT NULL,
    scopes TEXT[],
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Email Classifications Table:**
```sql
CREATE TABLE email_classifications (
    id SERIAL PRIMARY KEY,
    user_id VARCHAR(255) REFERENCES users(user_id),
    email_id VARCHAR(255) NOT NULL,
    category VARCHAR(50) NOT NULL,
    confidence FLOAT NOT NULL,
    reasoning TEXT,
    classified_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(user_id, email_id)
);
```

**Job Opportunities Table:**
```sql
CREATE TABLE job_opportunities (
    id SERIAL PRIMARY KEY,
    user_id VARCHAR(255) REFERENCES users(user_id),
    email_id VARCHAR(255) NOT NULL,
    role VARCHAR(255),
    company VARCHAR(255),
    deadline TIMESTAMP,
    location VARCHAR(255),
    compensation_amount FLOAT,
    compensation_currency VARCHAR(10),
    compensation_period VARCHAR(20),
    required_skills TEXT[],
    extraction_confidence FLOAT,
    extracted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Action Logs Table:**
```sql
CREATE TABLE action_logs (
    id SERIAL PRIMARY KEY,
    user_id VARCHAR(255) REFERENCES users(user_id),
    email_id VARCHAR(255) NOT NULL,
    action_type VARCHAR(50) NOT NULL,
    triggered_by VARCHAR(20) NOT NULL,
    success BOOLEAN NOT NULL,
    error_message TEXT,
    performed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Sessions Table:**
```sql
CREATE TABLE sessions (
    session_id VARCHAR(255) PRIMARY KEY,
    user_id VARCHAR(255) REFERENCES users(user_id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_activity TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    interactions JSONB
);
```

**Analytics Cache Table:**
```sql
CREATE TABLE analytics_cache (
    user_id VARCHAR(255) REFERENCES users(user_id),
    period_type VARCHAR(20) NOT NULL,
    period_start TIMESTAMP NOT NULL,
    period_end TIMESTAMP NOT NULL,
    metrics JSONB NOT NULL,
    calculated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, period_type, period_start)
);
```

## API Endpoints

### Authentication Endpoints

**POST /api/auth/login**
- Initiates OAuth flow
- Returns: `{ "auth_url": "https://accounts.google.com/..." }`

**GET /api/auth/callback**
- Handles OAuth callback
- Query params: `code`, `state`
- Returns: `{ "access_token": "...", "user_id": "..." }`

**POST /api/auth/refresh**
- Refreshes access token
- Body: `{ "refresh_token": "..." }`
- Returns: `{ "access_token": "...", "expires_in": 3600 }`

**POST /api/auth/revoke**
- Revokes access and clears tokens
- Returns: `{ "success": true }`

### Email Classification Endpoints

**POST /api/emails/classify**
- Classifies emails
- Body: `{ "email_ids": ["id1", "id2"], "fetch_from_gmail": true }`
- Returns: `{ "classifications": [...], "summary": {...} }`

**GET /api/emails/summary**
- Gets category summary
- Query params: `user_id`
- Returns: `{ "urgent": 2, "job_internship": 3, "promotional": 15, ... }`

### Career Intelligence Endpoints

**POST /api/career/extract**
- Extracts job data from emails
- Body: `{ "email_ids": ["id1", "id2"] }`
- Returns: `{ "opportunities": [...] }`

**GET /api/career/opportunities**
- Lists extracted job opportunities
- Query params: `user_id`, `limit`, `offset`
- Returns: `{ "opportunities": [...], "total": 42 }`

### Autonomous Agent Endpoints

**POST /api/autonomous/enable**
- Enables autonomous mode
- Body: `{ "user_id": "..." }`
- Returns: `{ "enabled": true }`

**POST /api/autonomous/disable**
- Disables autonomous mode
- Body: `{ "user_id": "..." }`
- Returns: `{ "enabled": false }`

**GET /api/autonomous/status**
- Checks autonomous mode status
- Query params: `user_id`
- Returns: `{ "enabled": true }`

**POST /api/autonomous/execute**
- Executes autonomous actions on classified emails
- Body: `{ "classifications": [...] }`
- Returns: `{ "actions": [...], "success_count": 10 }`

### Cold Email Endpoints

**POST /api/cold-email/generate**
- Generates cold email
- Body: `{ "context": {...} }`
- Returns: `{ "email": {...}, "analysis": {...} }`

**POST /api/cold-email/regenerate**
- Regenerates with feedback
- Body: `{ "email": {...}, "feedback": "..." }`
- Returns: `{ "email": {...}, "analysis": {...} }`

**POST /api/cold-email/send**
- Sends cold email (requires confirmation)
- Body: `{ "email": {...}, "confirmation": true }`
- Returns: `{ "sent": true, "message_id": "..." }`

### Session Endpoints

**POST /api/session/create**
- Creates new session
- Body: `{ "user_id": "..." }`
- Returns: `{ "session_id": "...", "created_at": "..." }`

**POST /api/session/interact**
- Adds interaction to session
- Body: `{ "session_id": "...", "command": "..." }`
- Returns: `{ "response": "...", "structured_data": {...} }`

**GET /api/session/context**
- Retrieves session context
- Query params: `session_id`, `lookback`
- Returns: `{ "interactions": [...] }`

**DELETE /api/session/{session_id}**
- Clears session
- Returns: `{ "cleared": true }`

### Analytics Endpoints

**GET /api/analytics/metrics**
- Gets productivity metrics
- Query params: `user_id`, `period` (today|week|month|all_time)
- Returns: `{ "metrics": {...}, "breakdown": {...} }`

**GET /api/analytics/actions**
- Lists recent actions
- Query params: `user_id`, `limit`, `offset`
- Returns: `{ "actions": [...], "total": 156 }`

### Voice Interface Endpoints

**POST /api/voice/command**
- Processes voice command
- Body: `{ "transcript": "...", "session_id": "..." }`
- Returns: `{ "response": "...", "speak": true }`

## Correctness Properties


*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property 1: Email Classification Completeness

*For any* email, when classified by the Email_Classifier, the output SHALL contain exactly one category from the valid set {Urgent, Job/Internship, Promotional, Meeting, General} and a confidence score between 0.0 and 1.0.

**Validates: Requirements 1.1, 1.2**

### Property 2: Classification Summary Accuracy

*For any* list of email classifications, the category summary counts SHALL equal the actual distribution of categories in the classification list.

**Validates: Requirements 1.3**

### Property 3: Classification Priority Ordering

*For any* email containing signals for multiple categories, the Email_Classifier SHALL assign the category with highest priority according to: Urgent > Job/Internship > Meeting > Promotional > General.

**Validates: Requirements 1.4**

### Property 4: Career Data Extraction Structure

*For any* email classified as Job/Internship, the Career_Intelligence_Engine output SHALL contain all required fields (Role, Company, Deadline, Location, Compensation, Required Skills), with missing fields marked as null or "Not specified", and the output SHALL be valid JSON.

**Validates: Requirements 2.1, 2.2, 2.3**

### Property 5: Multiple Opportunity Extraction

*For any* email containing N job opportunities (where N > 1), the Career_Intelligence_Engine SHALL return exactly N separate JobOpportunity objects.

**Validates: Requirements 2.4**

### Property 6: Data Normalization Consistency

*For any* extracted job opportunity, all date fields SHALL conform to ISO 8601 format, and all compensation fields SHALL have consistent structure with amount, currency (ISO 4217), and period.

**Validates: Requirements 2.5, 2.6**

### Property 7: Autonomous Action Mapping

*For any* email classification when Autonomous Decision Mode is enabled, the Autonomous_Agent SHALL execute the correct action: Promotional → archive, Job/Internship → star, Urgent → highlight.

**Validates: Requirements 3.1, 3.2, 3.3**

### Property 8: Autonomous Mode Disable Immediate Effect

*For any* system state where Autonomous Decision Mode transitions from enabled to disabled, no automated actions SHALL execute after the transition.

**Validates: Requirements 3.4**

### Property 9: Action Logging Completeness

*For any* automated action performed, the action log SHALL contain timestamp, email identifier, action type, and triggered_by fields.

**Validates: Requirements 3.5**

### Property 10: Destructive Action Blocking

*For any* destructive action request (send, delete) without explicit confirmation flag, the system SHALL reject the request and not execute the action.

**Validates: Requirements 3.6, 4.1, 5.5**

### Property 11: Autonomous Mode Toggle Availability

*For any* system state, calling disable_autonomous_mode SHALL successfully disable the mode regardless of current state or pending actions.

**Validates: Requirements 4.2**

### Property 12: Data Processing Privacy

*For any* email processed without explicit storage consent flag, the email content SHALL NOT appear in permanent storage (database tables).

**Validates: Requirements 4.3**

### Property 13: Automated Action Visibility

*For any* automated action performed, the action SHALL appear in the action logs table and be retrievable via the actions API endpoint.

**Validates: Requirements 4.6**

### Property 14: Cold Email Generation Completeness

*For any* valid EmailContext input, the Cold_Email_Optimizer SHALL generate output containing subject, body, tone_analysis, personalization_score (0-100), and response_likelihood (Low|Medium|High).

**Validates: Requirements 5.1, 5.2, 5.3, 5.4**

### Property 15: Cold Email Regeneration Produces Change

*For any* generated email, when regenerated with feedback, the new email body SHALL differ from the original email body.

**Validates: Requirements 5.6**

### Property 16: Session Initialization Structure

*For any* user_id, creating a new session SHALL return a Session object with session_id, user_id, created_at, last_activity, and empty interactions list.

**Validates: Requirements 6.1**

### Property 17: Session Context Retrieval

*For any* session with N interactions added, retrieving context SHALL return the interactions in chronological order.

**Validates: Requirements 6.2**

### Property 18: Reference Resolution to Most Recent

*For any* session with multiple email references, resolving "this email" SHALL return the email_id from the most recent interaction containing an email reference.

**Validates: Requirements 6.3**

### Property 19: Structured Output JSON Validity

*For any* request requiring structured output, the response SHALL be valid JSON that can be parsed without errors.

**Validates: Requirements 6.4**

### Property 20: Session Context Window Limit

*For any* session with more than 10 interactions added, retrieving context with default lookback SHALL return only the 10 most recent interactions.

**Validates: Requirements 6.6**

### Property 21: Analytics Metrics Structure

*For any* analytics request, the response SHALL contain all required fields: total_emails_filtered, career_emails_detected, promotions_archived, and time_saved_minutes.

**Validates: Requirements 8.1, 8.2, 8.3**

### Property 22: Time Saved Calculation Accuracy

*For any* set of actions, the calculated time_saved_minutes SHALL equal the sum of: (classification_count × 10/60) + (career_extraction_count × 2) + (archive_count × 5/60) + (cold_email_count × 15).

**Validates: Requirements 8.4**

### Property 23: Token Revocation Cleanup

*For any* user with active tokens and sessions, revoking access SHALL remove all tokens from oauth_tokens table and all sessions from sessions table for that user.

**Validates: Requirements 9.4**

### Property 24: Credential Storage Prohibition

*For any* data stored in the database, no table or field SHALL contain user passwords or raw credentials.

**Validates: Requirements 9.6**

### Property 25: Rate Limit Retry with Backoff

*For any* Gmail API request that receives a rate limit error, the system SHALL retry the request with exponentially increasing delays (1s, 2s, 4s, 8s, ...).

**Validates: Requirements 11.1**

### Property 26: Classification Failure Default Category

*For any* email that cannot be classified (error or low confidence), the system SHALL assign category "General" and create an error log entry.

**Validates: Requirements 11.4**

### Property 27: Database Operation Retry Logic

*For any* database operation that fails, the system SHALL retry up to 3 times before raising an error to the caller.

**Validates: Requirements 11.5**

### Property 28: Error Logging Structure

*For any* error that occurs, the error log SHALL contain timestamp, error_type, and context fields.

**Validates: Requirements 11.6**

### Property 29: Batch Classification Parallelism

*For any* batch of N emails (where N ≤ 50), the classify_batch function SHALL process them concurrently rather than sequentially.

**Validates: Requirements 12.1**

### Property 30: Classification Result Caching

*For any* email classified within the last 24 hours, requesting classification again SHALL return the cached result without calling the Gemini API.

**Validates: Requirements 12.6**

## Error Handling

### Error Categories

**1. External API Errors:**
- Gmail API failures (rate limits, network errors, authentication errors)
- Gemini API failures (service unavailable, quota exceeded, invalid requests)
- OAuth errors (invalid tokens, expired tokens, revoked access)

**2. Data Validation Errors:**
- Invalid email format or structure
- Missing required fields in requests
- Invalid classification categories
- Malformed JSON in requests

**3. Business Logic Errors:**
- Attempting destructive actions without confirmation
- Session not found or expired
- User not authenticated
- Autonomous mode conflicts

**4. System Errors:**
- Database connection failures
- Database query failures
- Cache failures
- Internal server errors

### Error Handling Strategies

**Gmail API Rate Limiting:**
```python
async def handle_gmail_request_with_retry(request_func, max_retries=5):
    """
    Exponential backoff retry for Gmail API rate limits
    """
    for attempt in range(max_retries):
        try:
            return await request_func()
        except RateLimitError as e:
            if attempt == max_retries - 1:
                raise
            delay = 2 ** attempt  # 1s, 2s, 4s, 8s, 16s
            await asyncio.sleep(delay)
            continue
```

**Gemini API Fallback:**
```python
async def classify_with_fallback(email: Email) -> Classification:
    """
    Try Gemini API, fall back to rule-based classification
    """
    try:
        return await gemini_classify(email)
    except GeminiAPIError as e:
        logger.warning(f"Gemini API failed, using fallback: {e}")
        return rule_based_classify(email)

def rule_based_classify(email: Email) -> Classification:
    """
    Simple rule-based classification as fallback
    """
    subject_lower = email.subject.lower()
    body_lower = email.body.lower()
    
    # Urgent signals
    if any(signal in subject_lower or signal in body_lower 
           for signal in ["urgent", "asap", "immediate", "deadline today"]):
        return Classification(category="Urgent", confidence=0.7, reasoning="Rule-based: urgent keywords")
    
    # Job signals
    if any(signal in subject_lower or signal in body_lower 
           for signal in ["job opening", "internship", "career opportunity", "apply now"]):
        return Classification(category="Job/Internship", confidence=0.7, reasoning="Rule-based: job keywords")
    
    # Meeting signals
    if any(signal in subject_lower or signal in body_lower 
           for signal in ["meeting", "calendar", "schedule", "invite"]):
        return Classification(category="Meeting", confidence=0.7, reasoning="Rule-based: meeting keywords")
    
    # Promotional signals
    if any(signal in subject_lower or signal in body_lower 
           for signal in ["unsubscribe", "promotional", "offer", "discount", "sale"]):
        return Classification(category="Promotional", confidence=0.7, reasoning="Rule-based: promotional keywords")
    
    # Default to General
    return Classification(category="General", confidence=0.5, reasoning="Rule-based: no specific signals")
```

**Database Retry Logic:**
```python
async def execute_with_retry(db_operation, max_retries=3):
    """
    Retry database operations with exponential backoff
    """
    for attempt in range(max_retries):
        try:
            return await db_operation()
        except DatabaseError as e:
            if attempt == max_retries - 1:
                logger.error(f"Database operation failed after {max_retries} attempts: {e}")
                raise
            delay = 0.5 * (2 ** attempt)  # 0.5s, 1s, 2s
            await asyncio.sleep(delay)
            continue
```

**Validation Error Responses:**
```python
class ValidationError(Exception):
    def __init__(self, field: str, message: str):
        self.field = field
        self.message = message
        
@app.exception_handler(ValidationError)
async def validation_error_handler(request: Request, exc: ValidationError):
    return JSONResponse(
        status_code=400,
        content={
            "error": "validation_error",
            "field": exc.field,
            "message": exc.message
        }
    )
```

**Authentication Error Handling:**
```python
@app.exception_handler(AuthenticationError)
async def auth_error_handler(request: Request, exc: AuthenticationError):
    return JSONResponse(
        status_code=401,
        content={
            "error": "authentication_error",
            "message": "Invalid or expired authentication token",
            "action": "Please re-authenticate with Google OAuth"
        }
    )
```

**Destructive Action Prevention:**
```python
def validate_destructive_action(action_type: str, confirmation: bool):
    """
    Prevent destructive actions without explicit confirmation
    """
    if action_type in ["send", "delete"] and not confirmation:
        raise DestructiveActionError(
            f"Action '{action_type}' requires explicit confirmation"
        )
```

**Error Logging:**
```python
class ErrorLogger:
    def log_error(self, error: Exception, context: dict):
        """
        Log all errors with structured data
        """
        log_entry = {
            "timestamp": datetime.utcnow().isoformat(),
            "error_type": type(error).__name__,
            "error_message": str(error),
            "context": context,
            "stack_trace": traceback.format_exc()
        }
        logger.error(json.dumps(log_entry))
        
        # Store critical errors in database for monitoring
        if isinstance(error, CriticalError):
            self.store_error_in_db(log_entry)
```

## Testing Strategy

### Dual Testing Approach

CYRA requires both unit testing and property-based testing for comprehensive coverage:

**Unit Tests** focus on:
- Specific examples of email classification (e.g., a known urgent email)
- Edge cases (empty emails, malformed JSON, missing fields)
- Error conditions (API failures, invalid tokens, network errors)
- Integration points (OAuth flow, Gmail API calls, database operations)
- Voice command parsing for specific commands

**Property-Based Tests** focus on:
- Universal properties that hold for all inputs (e.g., all classifications have valid confidence scores)
- Comprehensive input coverage through randomization
- Invariants that must be maintained (e.g., autonomous mode state consistency)
- Round-trip properties (e.g., session serialization/deserialization)

### Property-Based Testing Configuration

**Testing Library:** Use `hypothesis` for Python (backend) and `fast-check` for TypeScript (frontend)

**Test Configuration:**
- Minimum 100 iterations per property test
- Each test must reference its design document property
- Tag format: `# Feature: cyra-ai-inbox-agent, Property {number}: {property_text}`

**Example Property Test:**
```python
from hypothesis import given, strategies as st
import pytest

# Feature: cyra-ai-inbox-agent, Property 1: Email Classification Completeness
@given(email=st.builds(Email, 
                       subject=st.text(min_size=1, max_size=200),
                       body=st.text(min_size=1, max_size=5000)))
@pytest.mark.property_test
async def test_classification_completeness(email):
    """
    For any email, classification output contains exactly one valid category
    and confidence score between 0.0 and 1.0
    """
    classifier = EmailClassifier()
    classification = await classifier.classify_email(email)
    
    # Check category is valid
    valid_categories = {"Urgent", "Job/Internship", "Promotional", "Meeting", "General"}
    assert classification.category in valid_categories
    
    # Check confidence score range
    assert 0.0 <= classification.confidence <= 1.0
    
    # Check required fields present
    assert classification.email_id is not None
    assert classification.timestamp is not None
```

**Example Unit Test:**
```python
@pytest.mark.unit_test
async def test_urgent_email_classification():
    """
    Test specific example: email with 'URGENT' in subject is classified as Urgent
    """
    email = Email(
        id="test123",
        subject="URGENT: Action Required Today",
        body="Please respond immediately.",
        from_address="sender@example.com"
    )
    
    classifier = EmailClassifier()
    classification = await classifier.classify_email(email)
    
    assert classification.category == "Urgent"
    assert classification.confidence > 0.7
```

### Test Coverage Requirements

**Backend (Python/FastAPI):**
- Unit test coverage: minimum 80%
- Property test coverage: all 30 properties implemented
- Integration tests: OAuth flow, Gmail API, database operations
- Error handling tests: all error categories covered

**Frontend (React/TypeScript):**
- Unit test coverage: minimum 75%
- Component tests: all major UI components
- Integration tests: API communication, voice interface
- Accessibility tests: keyboard navigation, screen reader support

**End-to-End Tests:**
- Complete user flows: authentication → classification → autonomous actions
- Voice command flows: voice input → processing → voice output
- Cold email generation flow: context input → generation → analysis → send
- Analytics dashboard: action logging → metric calculation → display

### Testing Tools

**Backend:**
- pytest: Test runner
- hypothesis: Property-based testing
- pytest-asyncio: Async test support
- pytest-mock: Mocking external services
- coverage.py: Code coverage measurement

**Frontend:**
- Vitest: Test runner
- fast-check: Property-based testing
- React Testing Library: Component testing
- MSW (Mock Service Worker): API mocking
- Playwright: E2E testing

**CI/CD:**
- Run all tests on every pull request
- Block merges if tests fail or coverage drops
- Run property tests with 100 iterations in CI
- Run E2E tests on staging environment before production deployment
