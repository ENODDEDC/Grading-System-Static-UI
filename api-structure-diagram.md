# Grading System API Structure Diagram

## System Architecture Flow

```mermaid
graph TB
    subgraph "Student Portal (External)"
        SP[Student Portal Website]
        SL[Student Login Page]
        SD[Student Dashboard]
    end
    
    subgraph "Admin/Teacher System (Your System)"
        AS[Admin System]
        TS[Teacher System]
        GE[Grade Entry Interface]
    end
    
    subgraph "Authentication Layer"
        GA[Google OAuth2]
        JWT[JWT Token Service]
        AUTH[Authorization Service]
    end
    
    subgraph "Grading API (Your Proposed API)"
        API[API Gateway]
        AE[Auth Endpoints]
        GE_API[Grade Endpoints]
        VE[Validation Engine]
    end
    
    subgraph "Database Layer"
        DB[(Grading Database)]
        STUDENTS[(Students Table)]
        GRADES[(Grades Table)]
        SUBJECTS[(Subjects Table)]
    end
    
    %% Authentication Flow
    SP --> SL
    SL --> GA
    GA --> JWT
    JWT --> AUTH
    
    %% API Access Flow
    SP --> API
    AS --> API
    TS --> API
    
    %% API Internal Flow
    API --> AE
    API --> GE_API
    GE_API --> VE
    VE --> DB
    
    %% Database Relations
    DB --> STUDENTS
    DB --> GRADES
    DB --> SUBJECTS
    
    %% Grade Entry Flow
    TS --> GE
    GE --> GE_API
    
    style SP fill:#e1f5fe
    style AS fill:#f3e5f5
    style TS fill:#f3e5f5
    style API fill:#e8f5e8
    style DB fill:#fff3e0
```

## Authentication Flow Sequence

```mermaid
sequenceDiagram
    participant Student as Student Portal
    participant Google as Google OAuth2
    participant API as Grading API
    participant DB as Database
    
    Note over Student,DB: Authentication Process
    Student->>Google: 1. Login Request (email/password)
    Google->>Google: 2. Validate Credentials
    Google->>API: 3. Send Authorization Code
    API->>API: 4. Generate JWT Token
    API->>Student: 5. Return JWT Token
    
    Note over Student,DB: Grade Request Process
    Student->>API: 6. Request Grades (with JWT)
    API->>API: 7. Validate JWT Token
    API->>DB: 8. Query Student Grades
    DB->>API: 9. Return Grade Data
    API->>Student: 10. Send Grade Response (JSON)
```

## API Endpoints Structure

```mermaid
graph LR
    subgraph "Authentication Endpoints"
        A1[POST /auth/token<br/>Login & Get JWT]
        A2[POST /auth/refresh<br/>Refresh Token]
    end
    
    subgraph "Grade Management Endpoints"
        G1[GET /students/{id}/grades<br/>Get Student Grades]
        G2[POST /students/{id}/grades<br/>Add New Grade]
        G3[PUT /students/{id}/grades/{gradeId}<br/>Update Grade]
        G4[DELETE /students/{id}/grades/{gradeId}<br/>Delete Grade]
    end
    
    subgraph "Section Management"
        S1[GET /sections/{id}/grades<br/>Get Section Grades]
        S2[POST /sections/{id}/grades/batch<br/>Batch Grade Input]
    end
    
    subgraph "Utility Endpoints"
        U1[GET /students/{id}/grades/summary<br/>Grade Summary]
        U2[GET /students/{id}/grades/transcript<br/>Official Transcript]
        U3[GET /academic-periods<br/>Available Periods]
        U4[GET /subjects<br/>Subject List]
    end
    
    style A1 fill:#ffcdd2
    style A2 fill:#ffcdd2
    style G1 fill:#c8e6c9
    style G2 fill:#fff9c4
    style G3 fill:#fff9c4
    style G4 fill:#ffcdd2
```

## Data Flow Architecture

```mermaid
graph TD
    subgraph "Input Layer"
        I1[Student Portal Request]
        I2[Admin Dashboard]
        I3[Teacher Interface]
    end
    
    subgraph "API Layer"
        API[Grading API Gateway]
        AUTH[Authentication Middleware]
        VALID[Request Validation]
        ROUTE[Route Handler]
    end
    
    subgraph "Business Logic Layer"
        BL1[Grade Calculation Engine]
        BL2[Permission Manager]
        BL3[Data Formatter]
    end
    
    subgraph "Data Layer"
        DB[(Database)]
        CACHE[(Redis Cache)]
        LOG[(Audit Logs)]
    end
    
    I1 --> API
    I2 --> API
    I3 --> API
    
    API --> AUTH
    AUTH --> VALID
    VALID --> ROUTE
    
    ROUTE --> BL1
    ROUTE --> BL2
    ROUTE --> BL3
    
    BL1 --> DB
    BL2 --> DB
    BL3 --> DB
    
    DB --> CACHE
    BL1 --> LOG
    BL2 --> LOG
    
    style API fill:#4caf50
    style AUTH fill:#ff9800
    style DB fill:#2196f3
```

## Request/Response Data Structure

```mermaid
graph TB
    subgraph "API Request Structure"
        REQ[HTTP Request]
        METHOD[Method: GET/POST/PUT/DELETE]
        URL[URL: /students/{id}/grades]
        HEADERS[Headers: Authorization Bearer JWT]
        PARAMS[Query Params: sy, semester, yearLevel, section]
        BODY[Request Body: Grade Data (POST/PUT only)]
    end
    
    subgraph "API Response Structure"
        RESP[HTTP Response]
        STATUS[Status Code: 200, 400, 401, 404, 500]
        RESPHEADERS[Response Headers: Content-Type, Cache-Control]
        RESPBODY[Response Body: JSON Data]
    end
    
    subgraph "JSON Response Format"
        JSON[JSON Structure]
        STAT[status: success/error]
        DATA[data: Grade Information]
        META[metadata: Timestamp, API Version]
        ERROR[error: Error Details (if failed)]
    end
    
    REQ --> METHOD
    REQ --> URL
    REQ --> HEADERS
    REQ --> PARAMS
    REQ --> BODY
    
    RESP --> STATUS
    RESP --> RESPHEADERS
    RESP --> RESPBODY
    
    RESPBODY --> JSON
    JSON --> STAT
    JSON --> DATA
    JSON --> META
    JSON --> ERROR
    
    style REQ fill:#e3f2fd
    style RESP fill:#e8f5e8
    style JSON fill:#fff3e0
```

## Security & Access Control

```mermaid
graph TB
    subgraph "User Roles"
        STUDENT[Student<br/>Read Own Grades Only]
        TEACHER[Teacher<br/>Read/Write Assigned Sections]
        ADMIN[Admin<br/>Read/Write All Data]
    end
    
    subgraph "Authentication Methods"
        OAUTH[Google OAuth2]
        JWT_AUTH[JWT Token Validation]
        BEARER[Bearer Token Authorization]
    end
    
    subgraph "Access Control"
        RBAC[Role-Based Access Control]
        PERM[Permission Validation]
        RATE[Rate Limiting]
    end
    
    subgraph "Security Measures"
        HTTPS[HTTPS Encryption]
        CORS[CORS Policy]
        AUDIT[Audit Logging]
    end
    
    STUDENT --> OAUTH
    TEACHER --> OAUTH
    ADMIN --> OAUTH
    
    OAUTH --> JWT_AUTH
    JWT_AUTH --> BEARER
    
    BEARER --> RBAC
    RBAC --> PERM
    PERM --> RATE
    
    RATE --> HTTPS
    HTTPS --> CORS
    CORS --> AUDIT
    
    style STUDENT fill:#c8e6c9
    style TEACHER fill:#fff9c4
    style ADMIN fill:#ffcdd2
    style OAUTH fill:#e1f5fe
```

## Integration Points

```mermaid
graph LR
    subgraph "External Systems"
        SP[Student Portal<br/>React/Vue/Angular]
        LMS[Learning Management System]
        SIS[Student Information System]
        MOBILE[Mobile App]
    end
    
    subgraph "Your Grading API"
        API[Grading API Gateway]
        ENDPOINTS[REST Endpoints]
        WEBHOOK[Webhooks (Future)]
    end
    
    subgraph "Internal Systems"
        ADMIN[Admin Dashboard]
        TEACHER[Teacher Interface]
        REPORTS[Report Generator]
        BACKUP[Backup System]
    end
    
    SP --> API
    LMS --> API
    SIS --> API
    MOBILE --> API
    
    API --> ENDPOINTS
    API --> WEBHOOK
    
    ENDPOINTS --> ADMIN
    ENDPOINTS --> TEACHER
    ENDPOINTS --> REPORTS
    ENDPOINTS --> BACKUP
    
    style API fill:#4caf50
    style SP fill:#2196f3
    style ADMIN fill:#ff9800
```

## Usage Instructions

1. **Copy any diagram** you want to show your professor
2. **Paste into Markdown file** or documentation
3. **Use in presentations** - most tools support Mermaid
4. **Explain the flow** using the visual diagrams

## Key Points for Your Professor

- **Authentication Flow**: Shows how Google OAuth2 works with JWT tokens
- **API Structure**: Clear endpoint organization and methods
- **Data Flow**: How requests move through the system
- **Security**: Role-based access and authentication layers
- **Integration**: How external student portals connect to your API

These diagrams visualize exactly what your `api-demo.html` demonstrates in working code!