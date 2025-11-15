# Lab 5 Report: Servlet & MVC Pattern Implementation

## WHAT: Project Overview

This project implements a **Student Management System** using the **MVC (Model-View-Controller)** architectural pattern. The application allows users to perform CRUD (Create, Read, Update, Delete) operations on student records through a web interface. The system consists of:

- **Model Layer**: `Student.java` - JavaBean representing student data
- **DAO Layer**: `StudentDAO.java` - Data Access Object handling database operations
- **Controller Layer**: `StudentController.java` - Servlet managing request routing and business logic
- **View Layer**: JSP files using JSTL and EL for presentation

## WHY: Purpose and Benefits

### Why MVC Pattern?

1. **Separation of Concerns**: Each layer has a specific responsibility
   - Model: Data structure and business logic
   - View: User interface and presentation
   - Controller: Request handling and coordination

2. **Maintainability**: Changes in one layer don't affect others
   - Database changes only affect DAO
   - UI changes only affect JSP files
   - Business logic changes only affect Controller

3. **Reusability**: Components can be reused across different views
   - Same DAO can serve multiple controllers
   - Same model can be used in different contexts

4. **Testability**: Each component can be tested independently
   - DAO can be tested with unit tests
   - Controller logic can be tested separately

### Why JSTL Instead of Scriptlets?

1. **Cleaner Code**: JSTL provides cleaner, more readable syntax
2. **Security**: Reduces risk of script injection
3. **Maintainability**: Easier for frontend developers to understand
4. **Best Practices**: Follows modern Java web development standards

### Why PreparedStatement?

1. **Security**: Prevents SQL injection attacks
2. **Performance**: Better query optimization by database
3. **Type Safety**: Ensures correct data types are used

## HOW: Implementation Details

### Exercise 1: Model Layer Implementation

#### Student.java (JavaBean)
**What**: A Plain Old Java Object (POJO) representing a student entity.

**How**:
- Declared private attributes: `id`, `studentCode`, `fullName`, `email`, `major`, `createdAt`
- Created no-arg constructor for JavaBean compliance
- Created parameterized constructor (without id) for creating new students
- Implemented getters and setters for all attributes
- Overrode `toString()` method for debugging and logging

**Why**: JavaBeans follow a standard pattern that makes them compatible with frameworks, JSP EL expressions, and serialization.

#### StudentDAO.java (Data Access Object)
**What**: A class responsible for all database interactions.

**How**:
- Defined database connection constants (URL, username, password, driver)
- Implemented `getConnection()` method using JDBC DriverManager
- Used try-with-resources for automatic resource management
- Implemented CRUD methods:
  - `getAllStudents()`: Retrieves all students using SELECT query
  - `getStudentById(int id)`: Retrieves single student by ID
  - `addStudent(Student student)`: Inserts new student using INSERT
  - `updateStudent(Student student)`: Updates existing student using UPDATE
  - `deleteStudent(int id)`: Removes student using DELETE

**Why**: 
- Centralizes database logic in one place
- Makes it easy to change database or add connection pooling later
- Try-with-resources ensures connections are always closed, preventing memory leaks

### Exercise 2: Controller Layer Implementation

#### StudentController.java (Servlet)
**What**: A servlet that acts as the controller, handling all HTTP requests and coordinating between Model and View.

**How**:
- Annotated with `@WebServlet("/student")` for URL mapping
- Initialized `StudentDAO` in `init()` method (called once when servlet loads)
- Implemented `doGet()` method to handle GET requests:
  - Extracts `action` parameter from request
  - Routes to appropriate method based on action (list, new, edit, delete)
- Implemented `doPost()` method to handle POST requests:
  - Routes to insert or update methods
- Created handler methods:
  - `listStudents()`: Gets all students from DAO, sets as request attribute, forwards to list view
  - `showNewForm()`: Forwards to form view for adding new student
  - `showEditForm()`: Gets student by ID, sets as request attribute, forwards to form view
  - `insertStudent()`: Extracts form parameters, creates Student object, calls DAO, redirects with message
  - `updateStudent()`: Gets existing student, updates fields, calls DAO, redirects with message
  - `deleteStudent()`: Extracts ID, calls DAO delete method, redirects with message

**Why**:
- Single servlet handles all student-related operations
- Separation of concerns: Controller doesn't know about database details
- Uses redirect-after-post pattern to prevent duplicate submissions
- Request attributes allow passing data to JSP views

### Exercise 3: View Layer Implementation

#### student-list.jsp
**What**: JSP page displaying a table of all students with action buttons.

**How**:
- Included JSTL taglib directive: `<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>`
- Used `<c:if>` to conditionally display success/error messages from URL parameters
- Used `<c:forEach>` to iterate through students list from request attribute
- Used EL expressions `${student.property}` to display data
- Used `<c:choose>` and `<c:when>` to handle empty list case
- Created links for Edit and Delete actions with confirmation dialog

**Why**:
- No scriptlets means cleaner, more maintainable code
- JSTL provides standard tag library for common operations
- EL expressions are type-safe and null-safe
- Separation: View only displays data, doesn't contain business logic

#### student-form.jsp
**What**: Single JSP form that handles both adding new students and editing existing ones.

**How**:
- Used `<c:choose>` to dynamically set page title (Add vs Edit)
- Used `<c:if>` to conditionally include hidden ID field (only when editing)
- Set form action dynamically: `value="${student != null ? 'update' : 'insert'}"`
- Made student code field readonly when editing: `${student != null ? 'readonly' : 'required'}`
- Pre-filled form fields using EL: `value="${student.fullName}"`
- Dynamic submit button text based on mode

**Why**:
- Single form reduces code duplication
- Conditional logic handles both scenarios elegantly
- Pre-filling values improves user experience
- Readonly student code prevents accidental changes to unique identifier

### Exercise 4: Complete CRUD Integration

**What**: Full integration of all Create, Read, Update, Delete operations.

**How**:
- Completed all DAO methods (getById, add, update, delete)
- Integrated all controller methods with proper error handling
- Added success/error messages via URL parameters
- Implemented redirect-after-post pattern
- Added confirmation dialog for delete operation

**Why**:
- Complete CRUD provides full functionality
- Error handling improves user experience
- Redirect-after-post prevents browser refresh from resubmitting forms
- Confirmation dialogs prevent accidental deletions

## Technical Decisions and Rationale

### 1. URL Parameter Messages vs Request Attributes
**Decision**: Used URL parameters for success/error messages (`?message=...`)

**Why**: 
- Messages persist after redirect
- Can be bookmarked/shared
- Simple to implement

**Trade-off**: Messages visible in URL (but acceptable for this use case)

### 2. Single Form for Add/Edit
**Decision**: One JSP form handles both add and edit operations

**Why**:
- Reduces code duplication
- Easier to maintain (one form to update)
- Consistent UI/UX

### 3. PreparedStatement for All Queries
**Decision**: Used PreparedStatement for all database operations

**Why**:
- Security: Prevents SQL injection
- Performance: Query plans are cached
- Type safety: Database validates types

### 4. Try-With-Resources
**Decision**: Used try-with-resources for all database connections

**Why**:
- Automatic resource management
- Guarantees connection closure even on exceptions
- Cleaner code (no finally blocks needed)

### 5. Redirect After POST
**Decision**: Used `sendRedirect()` after POST operations instead of forward

**Why**:
- Prevents duplicate form submissions on browser refresh
- Follows PRG (Post-Redirect-Get) pattern
- Better user experience

## Architecture Flow

### Request Flow Example: Adding a Student

1. **User Action**: Clicks "Add New Student" button
2. **HTTP Request**: GET `/student?action=new`
3. **Controller**: `doGet()` receives request, routes to `showNewForm()`
4. **Controller**: Forwards to `student-form.jsp`
5. **View**: JSP renders empty form
6. **User Action**: Fills form and submits
7. **HTTP Request**: POST `/student` with form data and `action=insert`
8. **Controller**: `doPost()` receives request, routes to `insertStudent()`
9. **Controller**: Extracts parameters, creates Student object
10. **DAO**: `addStudent()` executes INSERT query
11. **Controller**: Redirects to `/student?action=list&message=Success`
12. **Controller**: `listStudents()` gets all students from DAO
13. **View**: `student-list.jsp` displays updated list with success message

## Key Learning Outcomes

1. **MVC Pattern Understanding**: Clear separation between data (Model), presentation (View), and logic (Controller)

2. **Servlet Lifecycle**: Understanding of `init()`, `doGet()`, `doPost()` methods

3. **JSTL Mastery**: Replaced all scriptlets with JSTL tags and EL expressions

4. **Database Best Practices**: PreparedStatement, try-with-resources, proper connection management

5. **Web Application Architecture**: Request/response cycle, forwarding vs redirecting, request attributes

## Conclusion

This project successfully implements a complete MVC-based web application following Java EE best practices. The separation of concerns makes the codebase maintainable, testable, and scalable. The use of JSTL eliminates scriptlets, making the views cleaner and more secure. All CRUD operations are fully functional with proper error handling and user feedback.

The implementation demonstrates understanding of:
- MVC architectural pattern
- Servlet programming
- JSP with JSTL and EL
- JDBC database operations
- Web application security practices

