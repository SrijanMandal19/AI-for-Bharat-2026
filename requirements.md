# Requirements Document

## Introduction

My Campus Buddy is an AI-powered mentoring system designed to help Indian college students (particularly from Tier-2 and Tier-3 cities) navigate academics, discover opportunities, develop skills, and prepare career applications. The system provides multilingual support with voice capabilities, operates on AWS infrastructure, and must function effectively in low-bandwidth environments.

## Glossary

- **System**: The My Campus Buddy application
- **Student**: A registered user of the system, typically an undergraduate in an Indian university
- **Profile**: A structured collection of student data including education, skills, projects, and interests
- **Notice**: An academic document containing rules, deadlines, or announcements
- **Opportunity**: A scholarship, internship, hackathon, or job posting
- **RAG**: Retrieval-Augmented Generation system using indexed documents
- **Lite_Mode**: A reduced-bandwidth mode with text-only responses and no automatic voice playback
- **Voice_Pipeline**: The complete flow of Speech-to-Text → Translation → LLM → Translation → Text-to-Speech
- **Match_Score**: A calculated percentage indicating alignment between student profile and opportunity requirements
- **PII**: Personally Identifiable Information including Aadhaar numbers, phone numbers, emails, roll numbers
- **Resume_Builder**: The component that generates ATS-friendly resumes from student profiles
- **Knowledge_Base**: Amazon Bedrock Knowledge Base containing indexed academic documents
- **Search_Agent**: Amazon Bedrock Agent that performs live web searches for opportunities
- **Skill_Scout**: The component that curates free learning resources and creates learning paths

## Requirements

### Requirement 1: User Authentication and Profile Management

**User Story:** As a student, I want to create and maintain my academic profile, so that the system can provide personalized recommendations and generate tailored application materials.

#### Acceptance Criteria

1. THE System SHALL support authentication via email/OTP or Google login
2. WHEN a student completes registration, THE System SHALL create a Profile with fields for degree, year, branch, CGPA, skills, projects, and interests
3. WHEN a student updates their Profile, THE System SHALL persist changes to DynamoDB within 2 seconds
4. THE System SHALL allow students to access their Profile without integration with college ERP systems
5. WHEN Profile data is transmitted, THE System SHALL detect and mask PII before sending to LLM services

### Requirement 2: Academic Process Navigator

**User Story:** As a student, I want to understand complex academic notices and rules in simple language, so that I can meet deadlines and avoid penalties.

#### Acceptance Criteria

1. WHEN a student uploads an academic Notice, THE System SHALL store it in S3 and index it in the Knowledge_Base
2. WHEN a student asks about academic rules or deadlines, THE RAG SHALL retrieve relevant Notice content and generate a simplified explanation
3. THE System SHALL extract deadline dates from Notice content and present them in a structured format
4. WHEN explaining consequences, THE System SHALL provide context about penalties, fees, or academic impact
5. THE System SHALL respond to academic queries within 3 seconds in standard mode

### Requirement 3: Document and Form Explainer

**User Story:** As a student, I want line-by-line explanations of forms and legal documents in my preferred language, so that I can understand what I'm signing and avoid mistakes.

#### Acceptance Criteria

1. WHEN a student uploads a document image, THE System SHALL extract text using OCR capabilities
2. WHEN a student requests explanation, THE System SHALL use a vision-capable model to analyze document structure
3. THE System SHALL generate line-by-line explanations highlighting critical fields and risks
4. WHERE a student selects a vernacular language, THE System SHALL provide explanations in that language
5. THE System SHALL support explanation of forms, affidavits, and administrative documents

### Requirement 4: Opportunity Discovery Agent

**User Story:** As a student, I want to discover scholarships, internships, and hackathons that match my eligibility, so that I can apply to relevant opportunities without manual searching.

#### Acceptance Criteria

1. WHEN a student searches for opportunities, THE Search_Agent SHALL perform live web searches using Lambda-invoked search APIs
2. THE Search_Agent SHALL filter results based on Profile fields including degree, year, branch, CGPA, and location
3. WHEN presenting opportunities, THE System SHALL display at least 3 valid results with verified links
4. THE System SHALL verify opportunity links through HTTP response validation and domain credibility filtering
5. THE System SHALL categorize opportunities as scholarships, internships, hackathons, or jobs
6. THE System SHALL extract and display eligibility criteria, deadlines, and application links for each opportunity
7. THE System SHALL NOT store scraped opportunity data permanently

### Requirement 5: Proactive Nudging System

**User Story:** As a student, I want timely reminders about deadlines and opportunities, so that I don't miss important applications or submissions.

#### Acceptance Criteria

1. WHEN the System extracts a deadline from a Notice, THE System SHALL create a nudge reminder 3 days before the deadline
2. WHEN sending a nudge, THE System SHALL include context, relevant link, and suggested next step
3. THE System SHALL NOT send nudges without actionable content
4. WHEN seasonal internship cycles begin, THE System SHALL proactively suggest relevant opportunities based on Profile
5. THE System SHALL allow students to dismiss or snooze nudges

### Requirement 6: Career Application Engine - Resume Builder

**User Story:** As a student, I want to generate professional resumes automatically from my profile, so that I can apply to opportunities quickly without manual formatting.

#### Acceptance Criteria

1. WHEN a student requests resume generation, THE Resume_Builder SHALL create an ATS-friendly PDF from Profile data within 5 seconds
2. THE Resume_Builder SHALL include education, skills, projects, and experience from the Profile
3. WHEN Profile data is updated, THE System SHALL reflect changes in subsequently generated resumes
4. THE System SHALL provide downloadable PDF format for resumes
5. THE Resume_Builder SHALL use professional formatting suitable for Indian job markets

### Requirement 7: Career Application Engine - Opportunity Matching

**User Story:** As a student, I want to see how well I match with job opportunities, so that I can prioritize applications and identify skill gaps.

#### Acceptance Criteria

1. WHEN a student views an opportunity, THE System SHALL calculate a Match_Score comparing Profile skills with job requirements
2. THE System SHALL extract keywords from job descriptions to determine required skills
3. WHEN displaying Match_Score, THE System SHALL show percentage, missing skills, and suggested improvements
4. THE Match_Score SHALL use heuristic comparison of Profile skills against extracted job keywords
5. THE System SHALL highlight skills present in Profile that match job requirements

### Requirement 8: Career Application Engine - Cover Letter Generation

**User Story:** As a student, I want AI-generated cover letters tailored to specific opportunities, so that I can submit strong applications without writing from scratch.

#### Acceptance Criteria

1. WHEN a student requests a cover letter, THE System SHALL generate content using Profile data and job description
2. THE System SHALL offer tone options including simple English, formal, and confident
3. THE System SHALL generate cover letters within 5 seconds
4. THE System SHALL tailor content to highlight relevant Profile skills matching the opportunity
5. THE System SHALL provide editable text output for student customization

### Requirement 9: Multilingual and Voice Support

**User Story:** As a vernacular-medium student, I want to interact with the system in my preferred Indian language using voice, so that language is not a barrier to accessing guidance.

#### Acceptance Criteria

1. THE System SHALL support 7 Indian languages: English, Hindi, Bengali, Marathi, Telugu, Tamil, and Gujarati
2. WHEN a student speaks a query, THE Voice_Pipeline SHALL transcribe speech using Amazon Transcribe
3. WHERE the input language differs from English, THE System SHALL translate to English using Amazon Translate before LLM processing
4. WHERE the output language differs from English, THE System SHALL translate LLM responses using Amazon Translate
5. WHEN voice output is enabled, THE System SHALL synthesize speech using Amazon Polly
6. THE Voice_Pipeline SHALL complete end-to-end processing within 3 seconds
7. WHERE Lite_Mode is enabled, THE System SHALL disable automatic voice playback and provide text-only responses
8. THE System SHALL allow students to toggle voice support on or off

### Requirement 10: Personalized Skill Scout

**User Story:** As a student, I want curated learning paths using free resources, so that I can develop industry-relevant skills without financial barriers.

#### Acceptance Criteria

1. WHEN a student requests skill development guidance, THE Skill_Scout SHALL assess current skill level from Profile
2. THE Skill_Scout SHALL search for high-quality, paywall-free learning resources
3. THE System SHALL structure learning resources into timeline-based learning paths
4. THE Skill_Scout SHALL provide 100% free resources with no paid content
5. WHEN presenting learning paths, THE System SHALL include resource links, estimated time, and learning objectives
6. THE Skill_Scout SHALL map syllabus topics to industry-relevant skills when requested

### Requirement 11: Privacy and Data Protection

**User Story:** As a student, I want my personal information protected, so that sensitive data is not exposed or misused.

#### Acceptance Criteria

1. WHEN processing student data, THE System SHALL detect PII using Amazon Comprehend or regex-based masking
2. THE System SHALL mask Aadhaar numbers, phone numbers, emails, and roll numbers before sending to LLM services
3. THE System SHALL store Profile data securely in DynamoDB with appropriate access controls
4. WHERE students upload documents, THE System SHALL process them ephemerally unless explicitly saved to personal vault
5. WHEN documents are saved, THE System SHALL store them in S3 with student-specific access permissions
6. THE System SHALL follow minimal retention principles for processed data

### Requirement 12: Performance and Scalability

**User Story:** As a student in a low-bandwidth area, I want the system to work effectively on slow connections, so that I can access guidance despite connectivity limitations.

#### Acceptance Criteria

1. WHERE Lite_Mode is enabled, THE System SHALL reduce payload sizes and optimize for low-bandwidth
2. THE System SHALL provide text-only responses in Lite_Mode without automatic media loading
3. THE System SHALL respond to queries within 3 seconds for voice interactions
4. THE System SHALL generate resumes within 5 seconds
5. THE System SHALL use Amazon Bedrock Claude Haiku for speed-critical operations
6. THE System SHALL use Amazon Bedrock Claude Sonnet for reasoning-intensive operations

### Requirement 13: AWS Infrastructure Integration

**User Story:** As a system administrator, I want the application built on AWS services, so that it leverages managed services for reliability and scalability.

#### Acceptance Criteria

1. THE System SHALL use Amazon Bedrock for all generative AI operations
2. THE System SHALL use Amazon Bedrock Agents for Search_Agent and Skill_Scout workflows
3. THE System SHALL use Amazon Bedrock Knowledge Bases for RAG implementation
4. THE System SHALL use AWS Lambda for backend logic and API integrations
5. THE System SHALL use Amazon S3 for document storage
6. THE System SHALL use Amazon DynamoDB for Profile and application data
7. THE System SHALL use Amazon Transcribe for speech-to-text conversion
8. THE System SHALL use Amazon Translate for language translation
9. THE System SHALL use Amazon Polly for text-to-speech synthesis
10. THE System SHALL use Amazon Comprehend for PII detection


## Hackathon MVP Scope

This section defines the implementation scope for the AWS hackathon demonstration, organized into phases based on completion level.

### Phase 1: Fully Implemented Features (Core Demo)

The following features will be fully functional in the hackathon demonstration:

1. **Academic Process Navigator**
   - RAG-based notice explanation using Amazon Bedrock Knowledge Bases
   - Document upload and indexing to S3 and Knowledge Base
   - Query-based retrieval and simplified explanation generation
   - Deadline extraction and structured presentation

2. **Resume Builder**
   - Profile-based PDF generation using structured student data
   - ATS-friendly formatting for Indian job markets
   - Downloadable output within 5-second target
   - Dynamic updates based on profile changes

3. **Opportunity Discovery Agent**
   - Live web search using Amazon Bedrock Agents with Lambda function calling
   - Basic eligibility filtering based on degree, year, branch, and location
   - Result presentation with links, deadlines, and eligibility criteria
   - HTTP validation and domain credibility filtering

4. **Opportunity Match Score**
   - Heuristic keyword comparison between profile skills and job descriptions
   - Match percentage calculation and display
   - Missing skills identification
   - Suggested improvements based on gap analysis

### Phase 2: Prototype or Partial Implementation

The following features will be demonstrated in limited or simulated form:

1. **Proactive Nudging System**
   - Simulated deadline triggers based on sample notices
   - Demonstration of nudge format with context and next steps
   - Manual trigger mechanism for demo purposes

2. **Skill Scout**
   - Basic curated search for free learning resources
   - Simple learning path structure with timeline
   - Limited to 2-3 skill domains for demonstration

3. **Multilingual Voice Pipeline**
   - Full implementation for English
   - One additional Indian language (Hindi) for demonstration
   - Complete Voice_Pipeline flow: Transcribe → Translate → Bedrock → Translate → Polly
   - Remaining languages (Bengali, Marathi, Telugu, Tamil, Gujarati) as post-hackathon expansion

### Post-Hackathon Expansion

The system architecture is modular and designed for incremental feature addition:

- Complete multilingual support for all 7 languages
- Full proactive nudging with automated deadline monitoring
- Comprehensive skill scout with expanded domain coverage
- Cover letter generation engine
- Document and form explainer with OCR
- Enhanced opportunity filtering with CGPA and advanced criteria

## Non-Functional Requirements

### Performance

1. THE System SHALL respond to text queries within 3 seconds under standard network conditions
2. THE Voice_Pipeline SHALL complete end-to-end processing within 3-5 seconds under standard network conditions
3. THE Resume_Builder SHALL generate PDF output within 5 seconds
4. THE Search_Agent SHALL return initial results within 4 seconds of query submission
5. THE System SHALL process document uploads and initiate indexing within 2 seconds

### Scalability

1. THE System SHALL use serverless AWS architecture to enable automatic scaling
2. THE System SHALL leverage AWS Lambda for compute auto-scaling based on request volume
3. THE System SHALL use Amazon DynamoDB with on-demand capacity mode for automatic throughput scaling
4. THE System SHALL use Amazon Bedrock managed service for AI workload scaling
5. THE System SHALL design API endpoints to handle concurrent student requests without degradation

### Security

1. THE System SHALL mask PII (Aadhaar numbers, phone numbers, emails, roll numbers) before LLM invocation
2. THE System SHALL implement role-based access control for student data
3. THE System SHALL isolate document storage per student using S3 bucket policies and IAM roles
4. THE System SHALL encrypt data at rest in S3 and DynamoDB
5. THE System SHALL encrypt data in transit using TLS 1.2 or higher
6. THE System SHALL validate and sanitize all user inputs before processing

### Reliability

1. WHEN Amazon Bedrock service fails, THE System SHALL return a graceful error message and log the failure
2. WHEN search API calls fail, THE System SHALL implement exponential backoff retry logic with maximum 3 attempts
3. WHEN external services timeout, THE System SHALL respond within 10 seconds with partial results or error notification
4. THE System SHALL implement circuit breaker patterns for external API dependencies
5. THE System SHALL log all errors to Amazon CloudWatch for monitoring and debugging

### Availability

1. THE System SHALL use AWS managed services designed for high availability across multiple availability zones
2. THE System SHALL design Lambda functions to be stateless for fault tolerance
3. THE System SHALL use DynamoDB global tables for multi-region data replication (future enhancement)
4. THE System SHALL implement health check endpoints for monitoring system status

### Lite Mode Constraints

1. WHERE Lite_Mode is enabled, THE System SHALL provide text-only responses without voice synthesis
2. WHERE Lite_Mode is enabled, THE System SHALL reduce JSON payload sizes by omitting optional metadata
3. WHERE Lite_Mode is enabled, THE System SHALL disable automatic media loading including images and audio
4. WHERE Lite_Mode is enabled, THE System SHALL compress response data before transmission
5. WHERE Lite_Mode is enabled, THE System SHALL prioritize essential content over supplementary information

## Assumptions and Constraints

### Assumptions

1. Students provide accurate and truthful profile data including degree, year, branch, CGPA, and skills
2. Students have internet connectivity sufficient for HTTP requests (minimum 2G for Lite Mode, 3G+ for standard mode)
3. External search APIs are accessible and return structured data
4. Academic notices and documents are provided in readable formats (PDF, images with clear text)
5. Students have access to email or Google account for authentication
6. Opportunity websites maintain stable URLs and accessible content
7. Free learning resources remain available at provided URLs

### Constraints

1. THE System MUST use AWS services as specified in Requirement 13
2. THE System MUST NOT perform large-scale web scraping or store scraped opportunity data permanently
3. THE System MUST NOT integrate with college ERP systems as a mandatory requirement
4. THE System MUST operate within AWS Free Tier limits during development and hackathon demonstration
5. THE System MUST NOT guarantee 100% accuracy of opportunity eligibility filtering due to variability in source data
6. THE System MUST NOT replace official college communication channels or academic advising
7. THE System MUST limit document storage to reasonable quotas per student (e.g., 100MB per user)
8. THE System MUST NOT provide financial, legal, or medical advice beyond informational guidance

## Error Handling Requirements

### Requirement 14: Graceful Error Handling

**User Story:** As a student, I want clear error messages when the system encounters problems, so that I understand what went wrong and what to do next.

#### Acceptance Criteria

1. WHEN OCR processing fails on an uploaded document, THE System SHALL inform the student that text extraction failed and suggest uploading a clearer image
2. WHEN Amazon Bedrock times out or returns an error, THE System SHALL display a message indicating temporary unavailability and suggest retrying in a few moments
3. WHEN search API calls fail after retry attempts, THE System SHALL inform the student that opportunity search is temporarily unavailable and log the error for investigation
4. WHEN document upload to S3 fails, THE System SHALL notify the student of upload failure and provide option to retry
5. WHEN Knowledge_Base indexing fails, THE System SHALL queue the document for retry and notify the student of delayed availability
6. WHEN translation services fail, THE System SHALL fall back to English response and notify the student of language service unavailability
7. WHEN voice synthesis fails, THE System SHALL provide text-only response and indicate voice output is temporarily unavailable
8. WHEN Profile data validation fails, THE System SHALL highlight specific fields with errors and provide correction guidance
9. WHEN resume generation fails, THE System SHALL identify missing required Profile fields and prompt student to complete them
10. THE System SHALL log all errors to Amazon CloudWatch with sufficient context for debugging and monitoring
