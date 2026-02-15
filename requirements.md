# Requirements Document: CodeLens AI

## Introduction

CodeLens AI is an AI-powered developer learning and productivity companion designed specifically for learners and developers of Bharat. The system addresses the critical gap between academic knowledge and industry-ready skills by enabling users to understand complex real-world codebases through intelligent analysis, visualization, and personalized learning path generation.

The platform targets the next 100 million developers in India, with special focus on Tier-2 and Tier-3 cities where mentorship and structured guidance are limited. Users upload project files (ZIP or folders) for comprehensive analysis without requiring direct GitHub integration, making it accessible to learners with varying levels of infrastructure access.

## Glossary

- **CodeLens_System**: The complete AI-powered platform for code analysis and learning
- **Upload_Handler**: Component responsible for receiving and validating uploaded project files
- **Code_Parser**: Component that extracts Abstract Syntax Trees (AST) and code structure
- **Architecture_Visualizer**: Component that generates visual representations of code architecture
- **Execution_Simulator**: Component that simulates step-by-step code execution flow
- **Skill_Analyzer**: Component that identifies required skills from codebase and compares with user profile
- **Learning_Path_Generator**: Component that creates personalized 30-day learning roadmaps
- **Health_Scorer**: Component that calculates project quality metrics
- **Refactor_Advisor**: Component that identifies code improvements and security issues
- **Interview_Preparer**: Component that generates interview questions from uploaded projects
- **User_Profile**: Stored data about user's skill level, languages known, and learning preferences
- **Project_Metadata**: Extracted information about uploaded codebase structure and dependencies
- **Explanation_Mode**: User-selected complexity level (Beginner/Intermediate/Advanced/Interview)
- **Skill_Gap**: Difference between required skills in project and user's current capabilities
- **Health_Score**: Composite metric (0-100) measuring code quality across multiple dimensions
- **Bedrock_Service**: Amazon Bedrock LLM service for AI inference
- **Processing_Pipeline**: AWS Step Functions orchestrated workflow for code analysis

## Requirements

### Requirement 1: Project Upload and Validation

**User Story:** As a learner, I want to upload my project files securely, so that I can analyze codebases without needing GitHub access or complex setup.

#### Acceptance Criteria

1. WHEN a user uploads a ZIP file, THE Upload_Handler SHALL accept files up to 50MB in size
2. WHEN a user uploads individual files or folders, THE Upload_Handler SHALL accept common source code file extensions (.py, .js, .ts, .java, .cpp, .go, .rb, .php, .html, .css, .json, .xml, .yaml, .md)
3. WHEN an uploaded file exceeds size limits, THE Upload_Handler SHALL reject the upload and return a descriptive error message within 2 seconds
4. WHEN a ZIP file is uploaded, THE Upload_Handler SHALL extract and validate the contents before processing
5. WHEN malicious file types are detected (.exe, .dll, .so, .sh with executable permissions), THE Upload_Handler SHALL reject the upload and log the security event
6. WHEN upload is successful, THE CodeLens_System SHALL store files in isolated S3 buckets with user-specific prefixes
7. WHEN files are stored in S3, THE CodeLens_System SHALL encrypt them at rest using AES-256
8. WHEN a user uploads a project, THE CodeLens_System SHALL generate a unique project identifier and associate it with the user's profile

### Requirement 2: Code Parsing and Structure Extraction

**User Story:** As a learner, I want the system to automatically understand my code structure, so that I can see how different parts of my project connect.

#### Acceptance Criteria

1. WHEN a project is uploaded, THE Code_Parser SHALL detect the primary programming language within 3 seconds
2. WHEN source files are identified, THE Code_Parser SHALL generate Abstract Syntax Trees for supported languages (Python, JavaScript, TypeScript, Java, C++, Go)
3. WHEN AST generation completes, THE Code_Parser SHALL extract function definitions, class definitions, import statements, and module dependencies
4. WHEN the codebase exceeds 10,000 lines, THE Code_Parser SHALL process files in parallel using multiple Lambda invocations
5. WHEN parsing fails for a file, THE Code_Parser SHALL log the error and continue processing remaining files
6. WHEN parsing completes, THE CodeLens_System SHALL store the Project_Metadata in DynamoDB with TTL of 90 days
7. WHEN multiple programming languages are detected, THE Code_Parser SHALL identify the primary language based on line count percentage

### Requirement 3: Architecture Visualization Generation

**User Story:** As a learner, I want to see visual diagrams of my project architecture, so that I can understand the overall structure at a glance.

#### Acceptance Criteria

1. WHEN Project_Metadata is available, THE Architecture_Visualizer SHALL generate a folder hierarchy tree representation
2. WHEN module dependencies are extracted, THE Architecture_Visualizer SHALL create a directed graph showing import relationships
3. WHEN API endpoints are detected (REST routes, GraphQL schemas), THE Architecture_Visualizer SHALL generate an API interaction map
4. WHEN database schemas are identified (ORM models, SQL files), THE Architecture_Visualizer SHALL infer and visualize entity relationships
5. WHEN visualization generation completes, THE Architecture_Visualizer SHALL return structured data in JSON format suitable for frontend rendering
6. WHEN the dependency graph has more than 50 nodes, THE Architecture_Visualizer SHALL apply clustering algorithms to group related modules
7. WHEN visualization data is generated, THE CodeLens_System SHALL cache results in DynamoDB for 30 days

### Requirement 4: Execution Trace Simulation

**User Story:** As a learner, I want to see step-by-step execution flow of functions, so that I can understand how code runs without actually executing it.

#### Acceptance Criteria

1. WHEN a user selects a function for simulation, THE Execution_Simulator SHALL identify the entry point and trace all called functions
2. WHEN tracing execution flow, THE Execution_Simulator SHALL generate a sequence of steps showing function calls in order
3. WHEN loops are encountered, THE Execution_Simulator SHALL show iteration logic with sample values for first 3 iterations
4. WHEN recursion is detected, THE Execution_Simulator SHALL visualize the call stack up to depth of 5
5. WHEN variable assignments occur, THE Execution_Simulator SHALL track state changes with example values
6. WHEN conditional branches exist, THE Execution_Simulator SHALL show both paths with conditions clearly labeled
7. WHEN simulation completes, THE Execution_Simulator SHALL return a structured execution trace with timestamps and step numbers
8. WHEN the execution trace exceeds 100 steps, THE Execution_Simulator SHALL provide a summarized view with expandable details

### Requirement 5: AI-Powered Code Explanation

**User Story:** As a learner, I want explanations tailored to my skill level, so that I can understand code without being overwhelmed or under-challenged.

#### Acceptance Criteria

1. WHEN a user requests explanation, THE CodeLens_System SHALL use the selected Explanation_Mode (Beginner/Intermediate/Advanced/Interview)
2. WHEN Explanation_Mode is Beginner, THE Bedrock_Service SHALL generate explanations using simple analogies and avoiding jargon
3. WHEN Explanation_Mode is Intermediate, THE Bedrock_Service SHALL include technical terms with brief definitions
4. WHEN Explanation_Mode is Advanced, THE Bedrock_Service SHALL provide deep technical insights including performance implications
5. WHEN Explanation_Mode is Interview, THE Bedrock_Service SHALL frame explanations as interview talking points with system design context
6. WHEN generating explanations for large files, THE CodeLens_System SHALL chunk code into segments of maximum 500 lines
7. WHEN prompting Bedrock_Service, THE CodeLens_System SHALL include user's known programming languages for better context
8. WHEN LLM response is received, THE CodeLens_System SHALL validate the response contains required sections before returning to user

### Requirement 6: Skill Gap Analysis

**User Story:** As a learner, I want to know what skills I'm missing based on real projects, so that I can focus my learning efforts effectively.

#### Acceptance Criteria

1. WHEN Project_Metadata is analyzed, THE Skill_Analyzer SHALL extract required technical concepts (frameworks, design patterns, algorithms, data structures)
2. WHEN skills are extracted, THE Skill_Analyzer SHALL compare them against the User_Profile skill inventory
3. WHEN comparing skills, THE Skill_Analyzer SHALL classify each Skill_Gap as Critical, Important, or Nice-to-have based on frequency in codebase
4. WHEN a skill appears in more than 30% of files, THE Skill_Analyzer SHALL mark it as Critical
5. WHEN a skill appears in 10-30% of files, THE Skill_Analyzer SHALL mark it as Important
6. WHEN a skill appears in less than 10% of files, THE Skill_Analyzer SHALL mark it as Nice-to-have
7. WHEN skill analysis completes, THE Skill_Analyzer SHALL return a ranked list of gaps with learning priority scores
8. WHEN no gaps are found, THE Skill_Analyzer SHALL recommend advanced topics for skill expansion

### Requirement 7: Personalized Learning Path Generation

**User Story:** As a learner with limited time, I want a structured 30-day learning plan, so that I can systematically fill my skill gaps.

#### Acceptance Criteria

1. WHEN Skill_Gap analysis is complete, THE Learning_Path_Generator SHALL create a 30-day roadmap structured by weeks
2. WHEN generating daily plans, THE Learning_Path_Generator SHALL allocate 1 hour for learning resources, 1-1.5 hours for practice problems, and 30 minutes for mini projects or revision
3. WHEN the user has 2-4 hours available daily, THE Learning_Path_Generator SHALL ensure total daily time does not exceed 3 hours
4. WHEN Critical gaps exist, THE Learning_Path_Generator SHALL prioritize them in Week 1-2
5. WHEN Important gaps exist, THE Learning_Path_Generator SHALL schedule them in Week 2-3
6. WHEN Nice-to-have gaps exist, THE Learning_Path_Generator SHALL schedule them in Week 4
7. WHEN generating learning resources, THE Learning_Path_Generator SHALL include free resources accessible in India (YouTube, freeCodeCamp, GeeksforGeeks, official documentation)
8. WHEN generating practice problems, THE Learning_Path_Generator SHALL link to LeetCode, HackerRank, or Codeforces problems with appropriate difficulty
9. WHEN the roadmap is generated, THE CodeLens_System SHALL store it in DynamoDB associated with the user and project
10. WHEN a user completes a day's tasks, THE CodeLens_System SHALL track progress and update completion percentage

### Requirement 8: Project Health Score Calculation

**User Story:** As a developer, I want to know the quality of my codebase, so that I can identify areas needing improvement.

#### Acceptance Criteria

1. WHEN Project_Metadata is available, THE Health_Scorer SHALL calculate a composite Health_Score from 0 to 100
2. WHEN calculating Health_Score, THE Health_Scorer SHALL use the formula: (0.25 × Readability) + (0.20 × Modularity) + (0.20 × Security) + (0.15 × Documentation) + (0.20 × Test_Coverage)
3. WHEN measuring Readability, THE Health_Scorer SHALL analyze average function length, variable naming conventions, and code complexity (cyclomatic complexity)
4. WHEN measuring Modularity, THE Health_Scorer SHALL evaluate separation of concerns, coupling between modules, and cohesion within modules
5. WHEN measuring Security, THE Health_Scorer SHALL scan for common vulnerabilities (hardcoded secrets, SQL injection patterns, XSS vulnerabilities, insecure dependencies)
6. WHEN measuring Documentation, THE Health_Scorer SHALL calculate percentage of functions with docstrings and presence of README files
7. WHEN measuring Test_Coverage, THE Health_Scorer SHALL detect test files and estimate coverage based on test-to-code ratio
8. WHEN Health_Score is below 50, THE Health_Scorer SHALL flag the project as needing significant improvement
9. WHEN Health_Score is between 50-75, THE Health_Scorer SHALL flag the project as moderate quality
10. WHEN Health_Score is above 75, THE Health_Scorer SHALL flag the project as good quality
11. WHEN scoring completes, THE Health_Scorer SHALL provide detailed breakdown of each dimension with specific improvement suggestions

### Requirement 9: Refactor and Improvement Suggestions

**User Story:** As a developer, I want specific suggestions to improve my code, so that I can write better quality software.

#### Acceptance Criteria

1. WHEN code analysis completes, THE Refactor_Advisor SHALL identify code smells (long functions, duplicate code, large classes, long parameter lists)
2. WHEN code smells are found, THE Refactor_Advisor SHALL provide specific line numbers and file paths
3. WHEN analyzing naming, THE Refactor_Advisor SHALL flag non-descriptive variable names (single letters, abbreviations without context)
4. WHEN performance issues are detected (nested loops with high complexity, inefficient algorithms), THE Refactor_Advisor SHALL suggest optimizations with complexity analysis
5. WHEN security vulnerabilities are found, THE Refactor_Advisor SHALL classify severity as High, Medium, or Low
6. WHEN High severity vulnerabilities exist, THE Refactor_Advisor SHALL provide code examples showing secure alternatives
7. WHEN generating suggestions, THE Refactor_Advisor SHALL prioritize based on impact and effort (Quick Wins, High Impact, Long Term)
8. WHEN suggestions are generated, THE CodeLens_System SHALL return them grouped by category with actionable next steps

### Requirement 10: Interview Preparation Mode

**User Story:** As a job seeker, I want interview questions based on my project, so that I can confidently discuss my work in technical interviews.

#### Acceptance Criteria

1. WHEN Interview Mode is activated, THE Interview_Preparer SHALL generate 10-15 technical questions based on the uploaded project
2. WHEN generating questions, THE Interview_Preparer SHALL cover code explanation, design decisions, scalability considerations, and trade-offs
3. WHEN the project uses specific frameworks, THE Interview_Preparer SHALL include framework-specific questions
4. WHEN system design aspects are present, THE Interview_Preparer SHALL generate system design questions with expected discussion points
5. WHEN generating resume bullets, THE Interview_Preparer SHALL create 3-5 achievement-oriented statements following STAR format
6. WHEN creating viva-style questions, THE Interview_Preparer SHALL include "why" and "how" questions about implementation choices
7. WHEN questions are generated, THE Interview_Preparer SHALL provide model answers with key points to cover

### Requirement 11: User Authentication and Profile Management

**User Story:** As a user, I want to securely access my account and manage my learning profile, so that my progress and projects are saved.

#### Acceptance Criteria

1. WHEN a new user signs up, THE CodeLens_System SHALL use Amazon Cognito for authentication
2. WHEN a user creates an account, THE CodeLens_System SHALL collect skill level (1-5 scale), known programming languages, target role, and preferred Explanation_Mode
3. WHEN a user logs in, THE CodeLens_System SHALL issue a JWT token valid for 24 hours
4. WHEN a token expires, THE CodeLens_System SHALL require re-authentication
5. WHEN a user updates their profile, THE CodeLens_System SHALL validate inputs and update DynamoDB within 2 seconds
6. WHEN a user accesses their dashboard, THE CodeLens_System SHALL display all uploaded projects with analysis status
7. WHEN a user deletes a project, THE CodeLens_System SHALL remove all associated data from S3 and DynamoDB within 5 minutes

### Requirement 12: Processing Pipeline Orchestration

**User Story:** As a system administrator, I want reliable processing of uploaded projects, so that users receive consistent and timely results.

#### Acceptance Criteria

1. WHEN a project is uploaded, THE Processing_Pipeline SHALL orchestrate analysis using AWS Step Functions
2. WHEN the pipeline starts, THE Processing_Pipeline SHALL execute steps in sequence: Upload Validation → Code Parsing → Architecture Visualization → Skill Analysis → Health Scoring → Learning Path Generation
3. WHEN any step fails, THE Processing_Pipeline SHALL retry up to 3 times with exponential backoff
4. WHEN retries are exhausted, THE Processing_Pipeline SHALL log the failure and notify the user with a descriptive error message
5. WHEN processing small projects (under 1000 lines), THE Processing_Pipeline SHALL complete within 30 seconds
6. WHEN processing medium projects (1000-5000 lines), THE Processing_Pipeline SHALL complete within 90 seconds
7. WHEN processing large projects (5000-10000 lines), THE Processing_Pipeline SHALL complete within 180 seconds
8. WHEN a step completes, THE Processing_Pipeline SHALL update the project status in DynamoDB
9. WHEN the entire pipeline completes, THE Processing_Pipeline SHALL send a notification to the user

### Requirement 13: Cost Optimization and Resource Management

**User Story:** As a platform operator, I want to minimize AWS costs while maintaining performance, so that the service remains sustainable and scalable.

#### Acceptance Criteria

1. WHEN invoking Bedrock_Service, THE CodeLens_System SHALL limit prompt tokens to maximum 4000 tokens per request
2. WHEN generating explanations, THE CodeLens_System SHALL cache LLM responses in DynamoDB with 7-day TTL for identical code segments
3. WHEN a cached response exists, THE CodeLens_System SHALL return it instead of calling Bedrock_Service
4. WHEN processing large projects, THE CodeLens_System SHALL use Lambda functions with appropriate memory allocation (512MB-3GB based on project size)
5. WHEN Lambda functions are idle, THE CodeLens_System SHALL allow them to scale to zero
6. WHEN S3 objects are older than 90 days, THE CodeLens_System SHALL transition them to S3 Glacier for archival
7. WHEN DynamoDB tables are queried, THE CodeLens_System SHALL use on-demand billing mode to avoid over-provisioning
8. WHEN CloudWatch logs are generated, THE CodeLens_System SHALL set retention period to 30 days

### Requirement 14: Security and Sandboxing

**User Story:** As a platform operator, I want to ensure uploaded code cannot harm the system, so that the platform remains secure for all users.

#### Acceptance Criteria

1. WHEN files are uploaded, THE Upload_Handler SHALL scan for malicious patterns using AWS Lambda with isolated execution environment
2. WHEN code is parsed, THE Code_Parser SHALL never execute user-uploaded code
3. WHEN storing files in S3, THE CodeLens_System SHALL use bucket policies preventing public access
4. WHEN accessing S3 objects, THE CodeLens_System SHALL use pre-signed URLs valid for maximum 1 hour
5. WHEN Lambda functions process code, THE CodeLens_System SHALL use IAM roles with least privilege permissions
6. WHEN data is transmitted, THE CodeLens_System SHALL enforce HTTPS/TLS 1.2 or higher
7. WHEN secrets are needed (API keys, database credentials), THE CodeLens_System SHALL store them in AWS Secrets Manager
8. WHEN a security event is detected, THE CodeLens_System SHALL log it to CloudWatch and trigger SNS notification

### Requirement 15: Monitoring and Observability

**User Story:** As a platform operator, I want comprehensive monitoring of system health, so that I can proactively address issues.

#### Acceptance Criteria

1. WHEN Lambda functions execute, THE CodeLens_System SHALL log execution time, memory usage, and errors to CloudWatch
2. WHEN API Gateway receives requests, THE CodeLens_System SHALL log request count, latency, and error rates
3. WHEN Bedrock_Service is invoked, THE CodeLens_System SHALL track token usage and cost per request
4. WHEN system errors occur, THE CodeLens_System SHALL create CloudWatch alarms for error rates exceeding 5%
5. WHEN response times exceed 10 seconds, THE CodeLens_System SHALL trigger latency alarms
6. WHEN alarms are triggered, THE CodeLens_System SHALL send notifications via SNS to operations team
7. WHEN analyzing system health, THE CodeLens_System SHALL provide dashboards showing key metrics (requests per minute, success rate, average latency, cost per user)

### Requirement 16: Scalability and High Availability

**User Story:** As a platform serving millions of users, I want the system to scale automatically, so that performance remains consistent during peak usage.

#### Acceptance Criteria

1. WHEN user traffic increases, THE CodeLens_System SHALL automatically scale Lambda functions to handle concurrent requests
2. WHEN Lambda concurrency reaches 80% of account limit, THE CodeLens_System SHALL trigger scaling alarms
3. WHEN DynamoDB experiences high read/write traffic, THE CodeLens_System SHALL automatically scale capacity using on-demand mode
4. WHEN API Gateway receives requests, THE CodeLens_System SHALL implement throttling at 100 requests per second per user
5. WHEN throttling limits are exceeded, THE CodeLens_System SHALL return HTTP 429 with retry-after header
6. WHEN deploying updates, THE CodeLens_System SHALL use blue-green deployment strategy to ensure zero downtime
7. WHEN a region experiences outage, THE CodeLens_System SHALL maintain 99.9% availability through multi-AZ deployment
8. WHEN S3 stores data, THE CodeLens_System SHALL use Standard storage class with 99.999999999% durability
