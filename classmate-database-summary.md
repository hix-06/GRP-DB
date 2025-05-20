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
```
                     +---------------+
                     |     chats     |
                     |---------------|
                     | id (PK)       |
                     | chat_name     |
                     | admin_id      |<-----+
                     | university    |      |
                     | faculty       |      |
                     | academic_year |      |
                     | created_at    |      |
                     | admin_name    |      |
                     +---------------+      |
                           |                |
                           | 1:many         |
                           v                |
                     +---------------+      |
                     |   messages    |      |
                     |---------------|      |
                     | id (PK)       |      |
                     | chat_id (FK)  |      |
                     | sender_id     |------+
                     | sender_name   |
                     | content       |
                     | created_at    |
                     | is_voice      |
                     | voice_url     |
                     +---------------+
                           
                     +---------------+
                     |   summaries   |
                     |---------------|
                     | id (PK)       |
                     | university    |
                     | timestamp     |
                     | faculty       |
                     | academic_year |
                     | file_url      |
                     | file_name     |
                     | subject       |
                     | uploaded_by   |
                     +---------------+
```

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
```
                         +---------------+
                         |  Auth.users   |
                         |---------------|
                         | id (PK)       |<-------------------------------+
                         | email         |                                |
                         | phone         |                                |
                         | created_at    |                                |
                         | last_sign_in  |                                |
                         | confirmed_at  |                                |
                         +---------------+                                |
                                |                                         |
                                | 1:1                                     |
                                v                                         |
                         +---------------+                                |
                         |   students    |                                |
                         |---------------|                                |
                         | user_id (PK/FK)|                               |
                         | full_name     |                                |
                         | email         |                                |
                         | faculty       |                                |
                         | academic_year |                                |
                         | created_at    |                                |
                         +---------------+                                |
                                                                          |
                         +---------------+                                |
                         |    files      |                                |
                         |---------------|                                |
                         | id (PK)       |                                |
                         | user_id (FK)  |-----------------------------+  |
                         | file_name     |                             |  |
                         | file_path     |                             |  |
                         | file_type     |                             |  |
                         | file_size     |                             |  |
                         | uploaded_at   |                             |  |
                         +---------------+                             |  |
                                                                       |  |
                         +---------------+                             |  |
                         |   schedules   |                             |  |
                         |---------------|                             |  |
                         | id (PK)       |                             |  |
                         | user_id (FK)  |-----------------------------+  |
                         | class_name    |                             |  |
                         | day           |                             |  |
                         | start_time    |                             |  |
                         | end_time      |                             |  |
                         | room          |                             |  |
                         | professor     |                             |  |
                         | created_at    |                             |  |
                         +---------------+                             |  |
                                                                       |  |
                         +---------------+                             |  |
                         |    archive    |                             |  |
                         |---------------|                             |  |
                         | id (PK)       |                             |  |
                         | user_id (FK)  |-----------------------------+  |
                         | user_name     |                             |  |
                         | faculty       |                             |  |
                         | academic_year |                             |  |
                         | file_name     |                             |  |
                         | file_path     |                             |  |
                         | file_type     |                             |  |
                         | file_size     |                             |  |
                         | uploaded_at   |                             |  |
                         +---------------+                             |  |
                                                                       |  |
                         +---------------+                             |  |
                         |     tasks     |                             |  |
                         |---------------|                             |  |
                         | id (PK)       |                             |  |
                         | user_id (FK)  |-----------------------------+  |
                         | title         |                                |
                         | description   |                                |
                         | due_date      |                                |
                         | priority      |                                |
                         | completed     |                                |
                         | created_at    |                                |
                         +---------------+                                |
                                                                          |
```

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

4. **Data Persistence Strategy**
   - Maintains local database for offline operation
   - Stores file references locally with actual files in device storage
   - Synchronizes with cloud when connection available

5. **Querying Patterns**
   - Primarily uses local queries for immediate response
   - Filters content based on user's academic context
   - Orders messages chronologically for conversation flow

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

6. **Row-Level Security**
   - Implements security through Supabase RLS policies
   - Restricts data access based on user_id
   - Ensures users can only modify their own data

## 5. Key Differences Between Mobile and Web Database Implementations

### Schema Differences

| Aspect | Mobile Database | Web Database |
|--------|----------------|--------------|
| **Authentication** | None (local profile only) | Full Supabase Auth integration |
| **Data Ownership** | Based on sender_id/uploaded_by | Based on auth.users foreign keys |
| **Table Structure** | Communication-focused | Organization-focused |
| **Security Model** | Device-level security | Row-level security policies |

### Data Storage Differences

| Mobile Storage | Web Storage | Advantage |
|----------------|-------------|-----------|
| Local with remote sync | Cloud-primary | Mobile: Offline access / Web: Universal access |
| Segregated storage buckets | Integrated file system | Mobile: Performance / Web: Simplicity |
| Limited by device capacity | Limited by cloud quota | Mobile: Portable / Web: Expansive |

### Query Pattern Differences

| Mobile Queries | Web Queries | Performance Impact |
|----------------|-------------|-------------------|
| Local database queries | Cloud database queries | Mobile: Faster / Web: Always current |
| Limited by device capabilities | Limited by network latency | Mobile: Consistent / Web: Variable |
| Optimized for minimal storage | Optimized for maximal features | Mobile: Efficient / Web: Feature-rich |

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