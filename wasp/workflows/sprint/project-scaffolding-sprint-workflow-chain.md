# Wasp Project Scaffolding Sprint Workflow Chain

> **⚠️ Workflow Status**: 
> - Experimental Workflow for Wasp Language Development
> - Designed to leverage Wasp's full-stack development capabilities
> - Iterative and adaptive approach to project initialization

## Workflow Objective
Create a comprehensive, production-ready web application using the Wasp language, focusing on rapid development, clean architecture, and full-stack integration.

## Prerequisite Checks
- [ ] Wasp CLI installed (version >= 0.11.0)
- [ ] Node.js (version 16+ recommended)
- [ ] npm or yarn package manager
- [ ] Basic understanding of React and Node.js

## Related Documentation
- [Wasp Workflow README](../README.md)
- [Requirements Generation Guide](../requirements/wasp-requirements-generation-guide.md)
- [Project Initialization Guide](../initialization/wasp-project-initialization-guide.md)

## Workflow Phases

### 1. Project Initialization and Requirements Gathering
- **Objective**: Define clear project scope, features, and technical requirements
- **Inputs**: 
  - High-level project concept
  - Target user personas
  - Key functional and non-functional requirements
- **Outputs**:
  - Detailed project requirements document
  - Initial feature breakdown
  - Technology stack confirmation

#### Prompt Template for Requirements
```markdown
Generate a comprehensive Wasp application specification focusing on declarative full-stack design.

Wasp-Specific Requirements Breakdown:
1. Application Core:
   - Define primary app entities using Wasp's declarative syntax
   - Specify client and server-side components
   - Outline data flow and state management approach

2. Wasp Language Features to Leverage:
   - Authentication methods (email, social login)
   - Routing configuration
   - Database schema and relationships
   - Client-server code co-location strategies

3. Functional Decomposition:
   - Break down features into Wasp routes
   - Define client and server actions
   - Specify data fetching and mutation patterns

4. Integration Considerations:
   - External API integrations
   - Third-party service connections
   - State management across client/server boundary

5. Deployment and Scalability:
   - Target hosting environment
   - Expected user load
   - Performance optimization strategies

Example Wasp Specification Structure:
```wasp
app MyProject {
  title: "Project Management Tool",
  auth: {
    methods: {
      email: {},
      google: {}
    },
    userEntity: User
  }
}

route DashboardRoute { 
  path: "/dashboard", 
  component: Dashboard 
}

entity User {=psl
  id         Int     @id @default(autoincrement())
  email      String  @unique
  name       String?
  projects   Project[]
=psl}
```

### 2. Wasp Project Scaffolding
- **Objective**: Set up initial project structure with Wasp
- **Inputs**: 
  - Validated project requirements
  - Chosen application type
- **Outputs**:
  - Basic Wasp project structure
  - Initial configuration files
  - Placeholder components

#### Scaffolding Commands
```bash
# Initialize new Wasp project
wasp start my-project

# Generate initial project structure
wasp generate crud User
wasp generate page HomePage
```

### 3. Backend Model and Data Layer Design
- **Objective**: Define data models, relationships, and initial database schema using Wasp and Prisma
- **Inputs**:
  - Project requirements
  - Data entity relationships
- **Outputs**:
  - Prisma schema
  - Wasp entity definitions
  - CRUD operation configurations

#### Wasp Data Modeling Prompt
```markdown
Design a comprehensive data model for [PROJECT_NAME] with Wasp-specific considerations:

Modeling Guidelines:
1. Entity Definitions:
   - List all primary entities
   - Define relationships (one-to-one, one-to-many, many-to-many)
   - Specify validation rules and constraints

2. Prisma Schema Considerations:
   - Database field types
   - Unique constraints
   - Relation fields and cascading behaviors

3. Wasp-Specific Modeling:
   - Define authentication requirements
   - Specify access control for entities
   - Create server actions for data manipulation

Example Entity Modeling Prompt:
- Create a project management application with:
  * User entity with authentication
  * Project entity with team members
  * Task entity linked to projects
  * Implement role-based access control

Provide a complete Wasp and Prisma schema that demonstrates:
- Declarative entity relationships
- Authentication integration
- Server-side validation
```

### 4. Frontend Component Architecture
- **Objective**: Design modular, reusable React components optimized for Wasp
- **Inputs**:
  - User interface requirements
  - Component interaction flows
- **Outputs**:
  - Component hierarchy
  - Routing configuration
  - State management approach

#### Component Design Prompt
```markdown
Create a component architecture for [PROJECT_NAME] tailored to Wasp's full-stack paradigm:

Component Design Criteria:
1. Structural Organization:
   - Atomic design principle implementation
   - Modular component breakdown
   - Reusable UI component library

2. Wasp-Specific Frontend Patterns:
   - Leverage Wasp's client-side routing
   - Implement server actions in components
   - Optimize data fetching and state management

3. Responsive and Accessible Design:
   - Mobile-first approach
   - Accessibility WCAG compliance
   - Performance optimization techniques

4. State Management:
   - Client-side state handling
   - Server state synchronization
   - Optimistic UI updates

Example Component Hierarchy:
- Layout Components
  * MainLayout
  * DashboardLayout
- Page Components
  * Dashboard
  * UserProfile
- Atomic Components
  * Button
  * Input
  * DataDisplay

Provide a comprehensive component strategy that demonstrates:
- Modular design
- Wasp routing integration
- Server state management
```

### 5. Authentication and Authorization
- **Objective**: Implement secure user authentication leveraging Wasp's built-in methods
- **Inputs**:
  - User roles
  - Access control requirements
- **Outputs**:
  - Authentication flow
  - Role-based access control
  - Secure route protection

#### Authentication Prompt
```markdown
Design a robust authentication system for [PROJECT_NAME] using Wasp's unique capabilities:

Authentication Requirements:
1. Authentication Methods:
   - Email/password registration
   - Social login providers
   - Passwordless authentication options

2. User Authorization:
   - Role-based access control
   - Granular permission management
   - Session management and security

3. Wasp-Specific Implementation:
   - Declarative auth configuration
   - Server-side authorization checks
   - Secure token management

Example Authentication Specification:
```wasp
app MyProject {
  auth: {
    methods: {
      email: {},
      google: {},
      github: {}
    },
    userEntity: User,
    onAuthFailedRedirectTo: "/login"
  }
}

route ProtectedRoute {
  path: "/dashboard",
  component: Dashboard,
  authRequired: true
}
```

Provide a comprehensive authentication strategy that covers:
- Multiple login methods
- Secure user management
- Role-based access control

### 6. Deployment and CI/CD Preparation
- **Objective**: Set up deployment pipeline and hosting
- **Inputs**:
  - Deployment target
  - Performance requirements
- **Outputs**:
  - Dockerfile
  - GitHub Actions workflow
  - Deployment configuration

### Verification Checklist
- [ ] Project compiles without errors
- [ ] All core features implemented
- [ ] Unit and integration tests created
- [ ] Documentation generated
- [ ] Deployment pipeline configured

## Workflow Completion Criteria
- Fully functional Wasp application
- Clean, modular code structure
- Comprehensive test coverage
- Deployment-ready configuration

## Recommended Tools and Extensions
- Wasp VSCode Extension
- Prisma VSCode Extension
- React Developer Tools
- GitHub Copilot (optional)

## Potential Challenges and Mitigations
1. Wasp Language Learning Curve
   - Leverage official documentation
   - Practice with small example projects
2. Full-stack Integration Complexity
   - Use Wasp's built-in generators
   - Follow recommended architectural patterns

## Version and Compatibility
- Workflow Version: 0.1.0
- Wasp Language Version: 0.11.x
- Recommended React Version: 18.x
- Recommended Node.js Version: 16.x+
