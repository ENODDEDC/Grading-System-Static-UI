# Grading System API - Design Document

## API Architecture Overview

### Base URL
```
https://api.edumanage.ph/v1/grades
```

### Authentication
- **Protocol**: OAuth2 with PKCE (for admin/teacher platform integration)
- **Token Type**: JWT (JSON Web Tokens)
- **Token Location**: Authorization header: `Bearer {token}`
- **Token Expiry**: 8 hours (extended for teacher workflow)
- **Roles**: Admin, Teacher (no student access)

## API Endpoints

### 1. Authentication Endpoints

#### POST /auth/token
**Purpose**: Exchange authorization code for access token

**Request Body**:
```json
{
  "grant_type": "authorization_code",
  "client_id": "admin_teacher_platform",
  "code": "auth_code_from_oauth_flow",
  "redirect_uri": "https://admin.edumanage.ph/callback",
  "code_verifier": "pkce_code_verifier"
}
```

**Response**:
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 28800,
  "refresh_token": "refresh_token_string",
  "scope": "grades:read grades:write admin:read",
  "user_role": "teacher",
  "user_id": "TEACHER001",
  "assigned_sections": ["HRM-A", "IT-B"]
}
```

#### POST /auth/refresh
**Purpose**: Refresh expired access token

**Request Body**:
```json
{
  "grant_type": "refresh_token",
  "refresh_token": "refresh_token_string",
  "client_id": "admin_teacher_platform"
}
```

### 2. Grade Management Endpoints

#### GET /students/{studentId}/grades
**Purpose**: Get all grades for a specific student (Admin: all students, Teacher: assigned sections only)

**Authorization**: 
- **Admin**: Can access any student
- **Teacher**: Can only access students in assigned sections

**Path Parameters**:
- `studentId` (string, required): Unique student identifier

**Query Parameters**:
- `school_year` (string, optional): e.g., "2023-2024"
- `semester` (string, optional): "1st", "2nd", "Summer"
- `year_level` (string, optional): "Grade 1", "Grade 7", "1st Year", etc.
- `section` (string, optional): Section name
- `quarter` (string, optional): "1st", "2nd", "3rd", "4th"
- `subject` (string, optional): Subject name or code
- `include_components` (boolean, optional): Include grade component breakdown (default: true)

**Example Request**:
```
GET /students/HRM2023-001/grades?school_year=2023-2024&semester=1st&quarter=1st
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Response**:
```json
{
  "status": "success",
  "data": {
    "student": {
      "id": "HRM2023-001",
      "name": "Alexandra Marie Dela Cruz Santos",
      "year_level": "1st Year",
      "section": "HRM-A",
      "program": "Bachelor of Science in Hotel & Restaurant Management"
    },
    "academic_period": {
      "school_year": "2023-2024",
      "semester": "1st",
      "quarter": "1st"
    },
    "subjects": [
      {
        "subject_id": "HRM101",
        "subject_name": "Introduction to Hospitality Industry",
        "subject_code": "HRM 101",
        "units": 3,
        "teacher": {
          "id": "TEACHER001",
          "name": "Prof. Maria Santos",
          "email": "maria.santos@edumanage.ph"
        },
        "grades": {
          "written_works": {
            "weight": 30,
            "assessments": [
              {
                "name": "Quiz 1",
                "score": 95,
                "max_score": 100,
                "date": "2023-09-15"
              },
              {
                "name": "Quiz 2",
                "score": 88,
                "max_score": 100,
                "date": "2023-09-22"
              }
            ],
            "average": 91.5
          },
          "performance_tasks": {
            "weight": 50,
            "assessments": [
              {
                "name": "Project 1",
                "score": 90,
                "max_score": 100,
                "date": "2023-09-30"
              },
              {
                "name": "Lab Activity",
                "score": 85,
                "max_score": 100,
                "date": "2023-10-05"
              }
            ],
            "average": 87.5
          },
          "quarterly_assessment": {
            "weight": 20,
            "assessments": [
              {
                "name": "Final Exam",
                "score": 88,
                "max_score": 100,
                "date": "2023-10-15"
              }
            ],
            "average": 88.0
          },
          "final_grade": 88.3,
          "status": "PASSED",
          "remarks": "Good performance"
        }
      }
    ],
    "summary": {
      "total_subjects": 8,
      "passed_subjects": 8,
      "failed_subjects": 0,
      "gpa": 88.5,
      "academic_status": "REGULAR"
    }
  },
  "metadata": {
    "total_records": 8,
    "page": 1,
    "per_page": 50,
    "generated_at": "2023-10-20T10:30:00Z"
  }
}
```

#### POST /students/{studentId}/grades
**Purpose**: Add new grade entry for a student (Teachers only)

**Authorization**: Teachers can only add grades for students in their assigned sections

**Request Body**:
```json
{
  "subject_id": "HRM101",
  "quarter": "1st",
  "assessment_type": "written_works", // "written_works", "performance_tasks", "quarterly_assessment"
  "assessment_name": "Quiz 1",
  "score": 95,
  "max_score": 100,
  "date": "2023-09-15"
}
```

**Response**:
```json
{
  "status": "success",
  "data": {
    "grade_id": "GRADE_12345",
    "student_id": "HRM2023-001",
    "subject_id": "HRM101",
    "assessment": {
      "name": "Quiz 1",
      "type": "written_works",
      "score": 95,
      "max_score": 100,
      "percentage": 95.0
    },
    "updated_averages": {
      "written_works_average": 91.5,
      "subject_final_grade": 88.3
    },
    "created_at": "2023-09-15T14:30:00Z"
  }
}
```

#### PUT /students/{studentId}/grades/{gradeId}
**Purpose**: Update existing grade entry (Teachers only)

**Authorization**: Teachers can only update grades they created

**Request Body**:
```json
{
  "score": 98,
  "max_score": 100,
  "assessment_name": "Quiz 1 (Revised)"
}
```

#### DELETE /students/{studentId}/grades/{gradeId}
**Purpose**: Delete grade entry (Teachers only)

**Authorization**: Teachers can only delete grades they created

#### GET /sections/{sectionId}/grades
**Purpose**: Get all grades for a specific section (Teachers: assigned sections, Admin: all sections)

**Query Parameters**:
- `subject_id` (string, optional): Filter by subject
- `quarter` (string, optional): Filter by quarter
- `assessment_type` (string, optional): Filter by assessment type

**Response**:
```json
{
  "status": "success",
  "data": {
    "section": {
      "id": "HRM-A",
      "name": "HRM Section A",
      "year_level": "1st Year",
      "program": "Hotel & Restaurant Management"
    },
    "students": [
      {
        "student_id": "HRM2023-001",
        "name": "Alexandra Marie Dela Cruz Santos",
        "subjects": [
          {
            "subject_id": "HRM101",
            "subject_name": "Introduction to Hospitality Industry",
            "grades": {
              "written_works": { "average": 91.5, "count": 2 },
              "performance_tasks": { "average": 87.5, "count": 2 },
              "quarterly_assessment": { "average": 88.0, "count": 1 },
              "final_grade": 88.3
            }
          }
        ]
      }
    ]
  }
}
```

#### POST /sections/{sectionId}/grades/batch
**Purpose**: Batch grade input for multiple students (Teachers only)

**Request Body**:
```json
{
  "subject_id": "HRM101",
  "quarter": "1st",
  "assessment_type": "written_works",
  "assessment_name": "Quiz 1",
  "max_score": 100,
  "grades": [
    {
      "student_id": "HRM2023-001",
      "score": 95
    },
    {
      "student_id": "HRM2023-002", 
      "score": 88
    }
  ]
}
```

#### GET /students/{studentId}/grades/summary
**Purpose**: Get grade summary for a student

**Query Parameters**:
- `school_year` (string, optional)
- `semester` (string, optional)

**Response**:
```json
{
  "status": "success",
  "data": {
    "student": {
      "id": "HRM2023-001",
      "name": "Alexandra Marie Dela Cruz Santos"
    },
    "academic_periods": [
      {
        "school_year": "2023-2024",
        "semester": "1st",
        "quarters": [
          {
            "quarter": "1st",
            "gpa": 88.5,
            "total_subjects": 8,
            "passed_subjects": 8,
            "status": "COMPLETED"
          },
          {
            "quarter": "2nd",
            "gpa": 89.2,
            "total_subjects": 8,
            "passed_subjects": 8,
            "status": "IN_PROGRESS"
          }
        ]
      }
    ],
    "overall_gpa": 88.85,
    "academic_standing": "GOOD_STANDING"
  }
}
```

#### GET /students/{studentId}/grades/transcript
**Purpose**: Get official transcript of records

**Query Parameters**:
- `format` (string, optional): "json" or "pdf" (default: "json")
- `include_failed` (boolean, optional): Include failed subjects (default: true)

**Response** (JSON format):
```json
{
  "status": "success",
  "data": {
    "student": {
      "id": "HRM2023-001",
      "name": "Alexandra Marie Dela Cruz Santos",
      "program": "Bachelor of Science in Hotel & Restaurant Management",
      "date_enrolled": "2023-08-15",
      "expected_graduation": "2027-04-30"
    },
    "academic_record": [
      {
        "school_year": "2023-2024",
        "year_level": "1st Year",
        "semester": "1st",
        "subjects": [
          {
            "subject_code": "HRM 101",
            "subject_name": "Introduction to Hospitality Industry",
            "units": 3,
            "final_grade": 88.3,
            "status": "PASSED"
          }
        ],
        "semester_gpa": 88.5,
        "total_units": 24,
        "units_earned": 24
      }
    ],
    "summary": {
      "cumulative_gpa": 88.5,
      "total_units_enrolled": 24,
      "total_units_earned": 24,
      "academic_standing": "GOOD_STANDING"
    }
  }
}
```

### 3. Utility Endpoints

#### GET /academic-periods
**Purpose**: Get available academic periods

**Response**:
```json
{
  "status": "success",
  "data": {
    "school_years": [
      {
        "year": "2023-2024",
        "status": "CURRENT",
        "start_date": "2023-08-15",
        "end_date": "2024-05-30"
      },
      {
        "year": "2022-2023",
        "status": "COMPLETED",
        "start_date": "2022-08-15",
        "end_date": "2023-05-30"
      }
    ],
    "semesters": ["1st", "2nd", "Summer"],
    "quarters": ["1st", "2nd", "3rd", "4th"]
  }
}
```

#### GET /subjects
**Purpose**: Get available subjects for filtering

**Query Parameters**:
- `year_level` (string, optional)
- `program` (string, optional)

**Response**:
```json
{
  "status": "success",
  "data": {
    "subjects": [
      {
        "id": "HRM101",
        "code": "HRM 101",
        "name": "Introduction to Hospitality Industry",
        "units": 3,
        "year_level": "1st Year",
        "semester": "1st"
      }
    ]
  }
}
```

## Error Responses

### Standard Error Format
```json
{
  "status": "error",
  "error": {
    "code": "INVALID_STUDENT_ID",
    "message": "The provided student ID does not exist or you don't have permission to access it",
    "details": {
      "student_id": "INVALID123",
      "timestamp": "2023-10-20T10:30:00Z"
    }
  }
}
```

### HTTP Status Codes
- `200 OK`: Successful request
- `400 Bad Request`: Invalid parameters or malformed request
- `401 Unauthorized`: Missing or invalid authentication token
- `403 Forbidden`: Insufficient permissions
- `404 Not Found`: Student or resource not found
- `429 Too Many Requests`: Rate limit exceeded
- `500 Internal Server Error`: Server error

### Common Error Codes
- `INVALID_TOKEN`: JWT token is invalid or expired
- `INSUFFICIENT_PERMISSIONS`: User doesn't have required permissions
- `INVALID_STUDENT_ID`: Student ID not found or inaccessible
- `INVALID_PARAMETERS`: Request parameters are invalid
- `RATE_LIMIT_EXCEEDED`: Too many requests from client
- `RESOURCE_NOT_FOUND`: Requested resource doesn't exist

## Security Considerations

### Access Control Matrix
| Role | Own Grades | Student Grades (Assigned) | All Grades | Grade Input/Update |
|------|------------|---------------------------|------------|-------------------|
| Teacher | ✅ Read | ✅ Read/Write (Assigned Sections) | ❌ | ✅ (Assigned Sections) |
| Admin | ✅ Read | ✅ Read/Write | ✅ Read/Write | ✅ (Override Capability) |

### Rate Limiting
- **Teachers**: 500 requests per minute  
- **Admins**: 1000 requests per minute

### Data Privacy
- Teachers can only access grades for sections they are assigned to teach
- Admins can access all grade data across the institution
- All API calls are logged for audit purposes with user identification
- Grade modifications are tracked with teacher/admin attribution

## Performance Specifications

### Response Time Targets
- Single student grade query: < 200ms
- Grade summary query: < 300ms
- Transcript generation: < 500ms
- Bulk queries (admin): < 1000ms

### Caching Strategy
- Student grade data cached for 5 minutes
- Academic period data cached for 1 hour
- Subject metadata cached for 24 hours
- Cache invalidation on grade updates

### Database Optimization
- Indexes on student_id, school_year, semester, subject_id
- Partitioning by school_year for historical data
- Read replicas for grade queries
- Connection pooling for high concurrency

## Integration Points

### Admin/Teacher Platform Integration
```javascript
// Example JavaScript integration for admin/teacher platform
const gradesAPI = new GradesAPI({
  baseURL: 'https://api.edumanage.ph/v1/grades',
  clientId: 'admin_teacher_platform',
  redirectUri: 'https://admin.edumanage.ph/callback'
});

// Teacher: Get grades for assigned section
const sectionGrades = await gradesAPI.getSectionGrades('HRM-A', {
  subject_id: 'HRM101',
  quarter: '1st'
});

// Teacher: Add new grade
const newGrade = await gradesAPI.addGrade('HRM2023-001', {
  subject_id: 'HRM101',
  quarter: '1st',
  assessment_type: 'written_works',
  assessment_name: 'Quiz 1',
  score: 95,
  max_score: 100
});

// Admin: Get institutional grade analytics
const analytics = await gradesAPI.getGradeAnalytics({
  school_year: '2023-2024',
  semester: '1st'
});
```

### Existing System Integration
- **Database Integration**: Connect to existing student information system
- **Authentication**: Integrate with current admin/teacher login system
- **UI Integration**: Embed grade management into existing admin dashboard
- **Reporting**: Connect to existing report generation system

### Webhook Support (Future Enhancement)
```json
{
  "event": "grade_updated",
  "student_id": "HRM2023-001",
  "subject_id": "HRM101",
  "quarter": "1st",
  "timestamp": "2023-10-20T10:30:00Z"
}
```

## API Documentation
- OpenAPI 3.0 specification available at `/docs/openapi.json`
- Interactive API documentation at `/docs`
- Postman collection available for testing
- SDK libraries for JavaScript, Python, and PHP