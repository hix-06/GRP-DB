# ClassMate: Database Design and Implementation

## Overview
ClassMate is an academic organization application designed to help students manage their educational resources, communications, and schedules. It consists of both a mobile application (offline-first approach) and a web application (online approach), both utilizing Supabase as the backend service provider.

## 1. Database Architecture Comparison

### Mobile Application Database
- **Type**: Local database with occasional synchronization (offline-first)
- **Purpose**: Optimized for offline usage and performance on mobile devices

### Web Application Database
- **Type**: Cloud-based database (online-only)
- **Purpose**: Designed for consistent access across multiple devices with persistent internet connection

## 2. Mobile Application Database Structure

### Tables Overview

**`chats`**: Stores information about chat groups based on university cohorts  
**`messages`**: Contains all text and voice messages linked to specific chats  
**`summaries`**: Stores metadata about educational resources shared by users  

### Storage Buckets
1. **summaries**: Stores PDF files and educational materials  
2. **voice-messages**: Stores voice recordings from the chat feature  

### Enhanced Relationship Diagram (Mobile App)

| chats                                   | messages                                | summaries                               |
|-----------------------------------------|-----------------------------------------|-----------------------------------------|
| ```                                     | ```                                     | ```                                     |
| +---------------+                       | +---------------+                       | +---------------+                       |
| |     chats     |                       | |   messages    |                       | |   summaries   |                       |
| |---------------|                       | |---------------|                       | |---------------|                       |
| | id (PK)       |                       | | id (PK)       |                       | | id (PK)       |                       |
| | chat_name     |                       | | chat_id (FK)  |<----------------------| | university    |                       |
| | admin_id      |<----------------------+ | sender_id     |                       | | timestamp     |                       |
| | university    |                       | | sender_name   |                       | | faculty       |                       |
| | faculty       |                       | | content       |                       | | academic_year |                       |
| | academic_year |                       | | created_at    |                       | | file_url      |                       |
| | created_at    |                       | | is_voice      |                       | | file_name     |                       |
| | admin_name    |                       | | voice_url     |                       | | subject       |                       |
| +---------------+                       | +---------------+                       | | uploaded_by   |                       |
|                                         |                                         | +---------------+                       |
| ```                                     | ```                                     | ```                                     |

**Key Relationships:**
- One chat can have multiple messages (1:many)
- Messages reference back to their sender
- Both chats and summaries use university, faculty, and academic_year as filtering attributes

## 3. Web Application Database Structure and Auth System

### Understanding Auth.users

**Auth.users** is a special system table provided by Supabase authentication service that:
- Stores user credentials and authentication information
- Manages email verification, password resets, and session tokens
- Serves as the foundation for user identity throughout the application
- Is automatically created and managed by Supabase Auth
- Cannot be directly modified by application code

**Auth.users schema includes:**
- id (UUID): Primary identifier used throughout the application
- email: User's verified email address
- phone: Optional phone number
- created_at: Account creation timestamp
- last_sign_in_at: Most recent login timestamp
- confirmed_at: Email verification timestamp

### Tables Overview

**`students`**: User profile information linked to authentication system  
**`files`**: Personal files uploaded by users  
**`schedules`**: Class schedule entries with time and location information  
**`archive`**: Shared educational resources available to cohort members  
**`tasks`**: Personal tasks and assignments with deadlines  

### Enhanced Relationship Diagram (Web App)

| Auth.users                              | students                                | files                                   | schedules                               | archive                                 | tasks                                   |
|-----------------------------------------|-----------------------------------------|-----------------------------------------|-----------------------------------------|-----------------------------------------|-----------------------------------------|
| ```                                     | ```                                     | ```                                     | ```                                     | ```                                     | ```                                     |
| +---------------+                       | +---------------+                       | +---------------+                       | +---------------+                       | +---------------+                       | +---------------+                       |
| |  Auth.users   |                       | |   students    |                       | |    files      |                       | |   schedules   |                       | |    archive    |                       | |     tasks     |                       |
| |---------------|                       | |---------------|                       | |---------------|                       | |---------------|                       | |---------------|                       | |---------------|                       |
| | id (PK)       |<----------------------+ | user_id (PK/FK)|                       | | id (PK)       |                       | | id (PK)       |                       | | id (PK)       |                       | | id (PK)       |                       |
| | email         |                       | | full_name     |                       | | user_id (FK)  |<----------------------+ | user_id (FK)  |<----------------------+ | user_id (FK)  |<----------------------+ | user_id (FK)  |<----------------------+ |
| | phone         |                       | | email         |                       | | file_name     |                       | | class_name    |                       | | user_name     |                       | | title         |                       |
| | created_at    |                       | | faculty       |                       | | file_path     |                       | | day           |                       | | faculty       |                       | | description   |                       |
| | last_sign_in  |                       | | academic_year |                       | | file_type     |                       | | start_time    |                       | | academic_year |                       | | due_date      |                       |
| | confirmed_at  |                       | | created_at    |                       | | file_size     |                       | | end_time      |                       | | file_name     |                       | | priority      |                       |
| +---------------+                       | +---------------+                       | | uploaded_at   |                       | | room          |                       | | file_path     |                       | | completed     |                       |
|                                         |                                         | +---------------+                       | | professor     |                       | | file_type     |                       | | created_at    |                       |
|                                         |                                         |                                         | | created_at    |                       | | file_size     |                       | +---------------+                       |
|                                         |                                         |                                         | +---------------+                       | | uploaded_at   |                       |                                         |
|                                         |                                         |                                         |                                         | +---------------+                       |                                         |
| ```                                     | ```                                     | ```                                     | ```                                     | ```                                     | ```                                     |

**Key Relationships:**
- Auth.users (1:1) students: Each authenticated user has exactly one student profile
- Auth.users (1:many) files: A user can upload multiple personal files
- Auth.users (1:many) schedules: A user can create multiple class schedule entries
- Auth.users (1:many) archive: A user can contribute multiple files to the archive
- Auth.users (1:many) tasks: A user can create multiple tasks and assignments

## 4. Database-Centric Application Summary

### Mobile Application Database Functions

1. **User Identification and Context**
   - Stores basic user information for local identification
   - Uses university, faculty, and academic_year as cohort filtering attributes
   - Does not require formal authentication

2. **Communication System**
   - Manages chat groups in the `chats` table
   - Stores messages with metadata in the `messages` table
   - Supports both text and voice messages with references to voice files

3. **Educational Resource Management**
   - Tracks shared educational resources in the `summaries` table
   - Uses field-based filtering to show relevant content
   - Maintains file metadata with links to actual files

### Web Application Database Functions

1. **User Authentication and Profiles**
   - Leverages Supabase Auth system through `auth.users`
   - Maintains extended user profiles in `students` table
   - Links all user data through the UUID foreign key

2. **Personal File Management**
   - Stores user-specific files in the `files` table
   - Associates files with authenticated users
   - Maintains file metadata for organization

3. **Schedule Management**
   - Records class schedules in the `schedules` table 
   - Stores temporal information (day, start_time, end_time)
   - Includes location and professor information

4. **Resource Sharing System**
   - Facilitates file sharing through the `archive` table
   - Uses faculty and academic_year for content filtering
   - Tracks file provenance through user associations

5. **Task Management**
   - Maintains user tasks in the `tasks` table
   - Stores deadline information for notifications
   - Tracks completion status for progress

## 5. Key Differences Between Mobile and Web Database Implementations

### Schema Differences

| Aspect | Mobile Database | Web Database |
|--------|----------------|--------------|
| **Authentication** | None (local profile only) | Full Supabase Auth integration |

### Data Storage Differences

| Aspect | Mobile Database | Web Database | Advantage |
|--------|----------------|--------------|-----------|
| **Storage Type** | Local with remote sync | Cloud-primary | Mobile: Offline access / Web: Universal access |
| **Data Querying** | Local queries with periodic sync to Supabase | Direct cloud queries to Supabase | Mobile: Fast local access, limited by sync / Web: Real-time, requires internet |

### Foreign Key Dependency Differences

| Aspect | Mobile Database | Web Database |
|--------|----------------|--------------|
| **Foreign Key Structure** | Relies on `chat_id` and `sender_id` for relationships between `chats` and `messages` tables | Universal dependency on `Auth.users.id` as the primary foreign key across `students`, `files`, `schedules`, `archive`, and `tasks` tables |
| **Identity Foundation** | No formal authentication; uses local identifiers like `admin_id` and `sender_id` | `Auth.users.id` serves as the foundation for user identity, linking all user-related data across tables |

## 6. Conclusion: Platform-Specific Database Design

ClassMate demonstrates strategic database design by tailoring each platform's database structure to match its technical constraints and usage patterns:

1. **Mobile Database Strengths**:
   - Optimized for offline operation
   - Focused on communication and quick resource access
   - Efficient data storage for device limitations

2. **Web Database Strengths**:
   - Robust authentication and security
   - Comprehensive organization tools
   - Shared resources with fine-grained access control

3. **Overall Architecture Benefits**:
   - Both platforms handle specific use cases optimally
   - Databases designed for their deployment environment
   - Backend services (Supabase) provide consistency

This dual-platform approach with specialized database designs provides students with a comprehensive solution that delivers both mobility and power, adapting to their changing academic needs.