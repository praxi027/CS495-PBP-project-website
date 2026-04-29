# Practice Before The Patient - Feature Usage Guide

## Overview

This document explains how to use each completed feature in Practice Before The Patient, what each feature produces, how the features connect to one another, and where the related implementation lives in the project.

---

## Table of Contents

1. [User Roles & Access Model](#user-roles--access-model)
2. [Feature Walkthroughs](#feature-walkthroughs)
   - [Home & Role-Based Navigation](#home--role-based-navigation)
   - [Simulation Runner](#simulation-runner)
   - [Student Grades Page](#student-grades-page)
   - [Scenario Editor](#scenario-editor)
   - [AI Scenario Generation](#ai-scenario-generation)
   - [Class Management](#class-management)
   - [Assignment Grading](#assignment-grading)
   - [Admin Management](#admin-management)
   - [Settings & Dev Access Switching](#settings--dev-access-switching)
   - [Database-Backed Persistence](#database-backed-persistence)
   - [API Documentation & Testing](#api-documentation--testing)
3. [How Completed Features Connect Together](#how-completed-features-connect-together)

---

## User Roles & Access Model

The application currently supports three stored roles plus one development-focused access mode.

| Access Type | Capabilities |
|-------------|--------------|
| **Student** | Complete assigned simulations and view grades |
| **Teacher** | Create and edit scenarios, manage classes, assign scenarios, and grade submissions |
| **Admin** | All teacher permissions plus admin user management tools |
| **Guest / Dev Practice Mode** | Access the simulation experience for testing, with limited class-linked grading and saved student submission workflows |

**Current authentication note:** Until OAuth is implemented, the project uses a simplified development access model. There is one admin access and testing flow used to reach admin-only functionality instead of a full production login system. Role switching and temporary admin access are being used as placeholders for real UA-authenticated access.

**Project links:**
- `PracticeBeforeThePatient.Web/Components/Layout/NavMenu.razor`
- `PracticeBeforeThePatient.Api/Controllers/AccessController.cs`

---

## Feature Walkthroughs

### Home & Role-Based Navigation

**How to use:**
1. Open the web application.
2. Use the left navigation menu to move between pages.
3. Review the links available for the current access level.

**Result:**
- Users only see the pages relevant to their role.
- Students see **Grades**.
- Teachers see **Scenario Editor** and **Class Management**.
- Admins also see **Admin Management**.

**Links to other features:**
- Navigation is the entry point to Simulation, Grades, Scenario Editor, Class Management, Assignment Grading, and Admin Management.

**Project links:**
- `PracticeBeforeThePatient.Web/Components/Layout/NavMenu.razor`
- `PracticeBeforeThePatient.Api/Controllers/AccessController.cs`

### Simulation Runner

**How to use:**
1. Open the **Simulation** page.
2. Select an available scenario or assignment from the dropdown if more than one is available.
3. Read the patient narration.
4. Choose the next action.
5. Enter reasoning for the choice.
6. Click **Submit** for that step, then **Continue**.
7. Repeat until the scenario is complete.
8. At the end, download the response summary if needed.

**Result:**
- The student progresses through a branching clinical case.
- The system records the student's reasoning and final response for the assignment when the simulation is linked to a class assignment.
- The visit summary compares what the student chose against the recommended path.

**Links to other features:**
- Assigned scenarios come from **Class Management**.
- Available scenario access comes from the Access API.
- Completed assigned work can later appear in **Assignment Grading** for instructors.
- Submitted work and instructor feedback appear on the **Grades** page for students.

**Project links:**
- `PracticeBeforeThePatient.Web/Components/Pages/Simulation.razor`
- `PracticeBeforeThePatient.Web/Components/Pages/Simulation.razor.cs`
- `PracticeBeforeThePatient.Api/Controllers/AssignmentsController.cs`
- `PracticeBeforeThePatient.Api/Controllers/ScenariosController.cs`
- `PracticeBeforeThePatient.Api/Controllers/AccessController.cs`

### Student Grades Page

**How to use:**
1. Sign in or switch into a student account.
2. Open the **Grades** page.
3. Review the list of assignments.
4. If an assignment has not been submitted, use the **Go to assignment** button to open the Simulation page.
5. If a submission exists, use **View response** to open the detailed grade view.

**Result:**
- Students can see assignment status, due dates, numeric grades, feedback, submission timestamps, grading timestamps, and the response that was graded.

**Example outcomes:**
- Assigned work that has not been submitted shows as **Not done** and links to the Simulation page.
- Submitted work without a grade shows as **Not graded yet**.
- Graded work shows the numeric score and allows the student to review the graded response.

**Links to other features:**
- Unsubmitted assignments link directly back to **Simulation**.
- Grade data comes from class assignments and instructor grading actions.
- Instructors create the grade entries from **Assignment Grading**.

**Project links:**
- `PracticeBeforeThePatient.Web/Components/Pages/Grades.razor`
- `PracticeBeforeThePatient.Api/Controllers/ClassManagementController.cs`
- `GET /api/classes/me/grades`

### Scenario Editor

**How to use:**
1. Open **Scenario Editor** as a teacher or admin.
2. Select an existing scenario from the dropdown, or generate a new one.
3. Edit the title and description.
4. Expand or collapse nodes to review the branching tree.
5. Edit node content, prompts, and choices.
6. Save changes.

**Result:**
- The scenario definition is updated and stored for future simulation use.

**Links to other features:**
- Scenarios created or edited here can be assigned to classes in **Class Management**.
- Generated scenarios load directly into the editor and can be refined before use.

**Project links:**
- `PracticeBeforeThePatient.Web/Components/Pages/ScenarioEditor.razor`
- `PracticeBeforeThePatient.Web/Components/Pages/ScenarioEditor.razor.cs`
- `PracticeBeforeThePatient.Web/Components/Pages/NodeEditor.razor`
- `PracticeBeforeThePatient.Api/Controllers/ScenariosController.cs`
- `GET /api/scenarios`
- `GET /api/scenarios/{scenarioId}`
- `PUT /api/scenarios/{scenarioId}`

### AI Scenario Generation

**How to use:**
1. Open **Scenario Editor**.
2. Enter a topic describing the clinical case to generate.
3. Optionally enter a custom scenario ID.
4. Choose branch depth.
5. Click **Generate Scenario**.
6. Wait for the generated case to be saved and loaded into the editor.
7. Review and edit the generated scenario before assigning it.

**Result:**
- The app creates a new branching scenario using Gemini and saves it into the project data store.

**Links to other features:**
- The generated scenario becomes available in **Scenario Editor** immediately.
- It can then be assigned to a class through **Class Management**.
- It can later be completed by students in **Simulation**.

**Project links:**
- `PracticeBeforeThePatient.Web/Components/Pages/ScenarioEditor.razor`
- `PracticeBeforeThePatient.Api/Controllers/ScenariosController.cs`
- `PracticeBeforeThePatient.Api/Services/LlmService.cs`
- `POST /api/scenarios/generate`

### Class Management

**How to use:**
1. Open **Class Management** as a teacher or admin.
2. Create a class.
3. Select the class from the dropdown.
4. Add or remove teachers.
5. Add or remove students by email.
6. Add assignments that connect a class to a scenario.
7. Set or edit due dates.
8. Delete assignments or classes if needed.
9. Use the **Grade** button on an assignment to open instructor grading.

**Result:**
- The app stores class rosters, teacher assignments, student enrollments, and scenario assignments.
- Multiple teachers can be attached to the same class.

**Example outcomes:**
- A class can have more than one teacher.
- A student added to a class can see assigned scenarios tied to that class.
- An assignment created here becomes the source for both student submission tracking and instructor grading.

**Links to other features:**
- Students added here receive access to assigned scenarios in **Simulation**.
- Assignments created here appear in the **Grades** page and **Assignment Grading** workflow.
- The **Grade** button links directly to the **Assignment Grading** page.

**Project links:**
- `PracticeBeforeThePatient.Web/Components/Pages/ClassManagement.razor`
- `PracticeBeforeThePatient.Api/Controllers/ClassManagementController.cs`
- `GET /api/classes`
- `POST /api/classes`
- `POST /api/classes/{classId}/students`
- `DELETE /api/classes/{classId}/students`
- `POST /api/classes/{classId}/teachers`
- `DELETE /api/classes/{classId}/teachers`
- `GET /api/classes/{classId}/assignments`
- `POST /api/classes/{classId}/assignments`
- `PUT /api/classes/{classId}/assignments/{assignmentId}`
- `PUT /api/classes/{classId}/assignments/{assignmentId}/due`
- `DELETE /api/classes/{classId}/assignments/{assignmentId}`

### Assignment Grading

**How to use:**
1. Open **Class Management**.
2. Choose a class and assignment.
3. Click **Grade**.
4. Review which students have submitted work.
5. Open a submitted response if needed.
6. Enter a score from 0 to 100.
7. Enter optional written feedback.
8. Save the grade.

**Result:**
- The student's submission receives a grade and optional feedback.
- The grading result becomes visible on the student **Grades** page.

**Example outcomes:**
- A submitted assignment appears as **Submitted** for the instructor.
- After grading, the student can open **Grades** and see the score, feedback, and graded response.

**Links to other features:**
- Receives assignments and student rosters from **Class Management**.
- Reads student submission text produced by the **Simulation** workflow.
- Sends final grade results to the student **Grades** page.

**Project links:**
- `PracticeBeforeThePatient.Web/Components/Pages/AssignmentGrading.razor`
- `PracticeBeforeThePatient.Api/Controllers/ClassManagementController.cs`
- `PUT /api/classes/{classId}/assignments/{assignmentId}/grades`

### Admin Management

**How to use:**
1. Use the admin access and testing flow to enter admin mode.
2. Open **Admin Management** as an admin user.
3. Review the list of users with elevated access.
4. Add a teacher or admin.
5. Update a user role if needed.
6. Remove elevated access if needed.

**Result:**
- The application updates which users have teacher or admin privileges.
- This is currently the main admin-focused workflow used until OAuth is implemented.

**Links to other features:**
- Access level changes affect which pages appear in navigation.
- Teachers added here can use **Class Management** and **Scenario Editor**.

**Project links:**
- `PracticeBeforeThePatient.Web/Components/Pages/AdminManagement.razor`
- `PracticeBeforeThePatient.Api/Controllers/AdminUsersController.cs`

### Settings & Dev Access Switching

**How to use:**
1. Open the **Settings** page or admin access and testing flow when available.
2. Switch the current development user or theme for testing.
3. Use temporary admin access for admin-only workflows in development.

**Result:**
- Developers and testers can simulate different user roles and themes without full production authentication.
- This is the temporary replacement for the final OAuth-based login flow.

**Links to other features:**
- Role switching changes the available navigation items and page access.
- Theme changes affect the visual presentation across the app.

**Project links:**
- `PracticeBeforeThePatient.Web/Components/Pages/Settings.razor`
- `PracticeBeforeThePatient.Web/Components/Pages/Demo.razor`
- `PracticeBeforeThePatient.Api/Controllers/AccessController.cs`
- `GET /api/access`
- `POST /api/access/dev-user`
- `POST /api/access/theme`
- `POST /api/access/temporary-admin/login`
- `POST /api/access/temporary-admin/logout`

### Database-Backed Persistence

**How to use:**
1. Start the application through Docker Compose or the supported backend setup.
2. Let the API apply migrations on startup.
3. Use the app normally through Scenario Editor, Class Management, Simulation, and Grading.

**Result:**
- Classes, users, teacher relationships, assignments, submissions, grades, and scenarios are stored in PostgreSQL instead of temporary JSON-only storage.

**Links to other features:**
- This is the persistence layer supporting nearly every completed feature.
- Scenario Editor writes scenarios.
- Class Management writes classes and assignments.
- Simulation writes submissions.
- Assignment Grading writes grades and feedback.

**Project links:**
- `PracticeBeforeThePatient.Api/Data`
- `PracticeBeforeThePatient.Api/Data/Migrations`
- `PracticeBeforeThePatient.Api/PracticeBeforeThePatient.Api.csproj`

### API Documentation & Testing

**How to use:**
1. Run the API in `Development` to access Swagger.
2. Open the Swagger UI to review and test endpoints.
3. Run the test suite with `dotnet test`.

**Result:**
- Developers can inspect endpoints, validate request and response behavior, and verify major project workflows through automated tests.

**Links to other features:**
- Swagger documents the API used by the web frontend.
- Tests validate access, scenarios, class management, grading, and entity relationships.

**Project links:**
- `PracticeBeforeThePatient.Tests`
- `README.md`
- `MAINTENANCE.md`

---

## How Completed Features Connect Together

The main student workflow moves through the application in this order:

1. A teacher creates or edits a scenario.
2. A teacher creates a class and assignment.
3. A student receives access to the assigned scenario.
4. The student completes the scenario in **Simulation**.
5. An instructor grades the submission.
6. The student reviews the grade and feedback on the **Grades** page.

This flow ties together the major finished features in the system:
- **Scenario Editor** and **AI Scenario Generation** create the learning content.
- **Class Management** connects that content to students.
- **Simulation Runner** captures the student's work.
- **Assignment Grading** evaluates the submission.
- **Grades** returns the result to the student.
