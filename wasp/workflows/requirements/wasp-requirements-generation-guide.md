# Wasp Project Requirements Generation Guide

## Purpose
This guide provides a structured approach to generating comprehensive requirements for Wasp-based web applications, ensuring clear, actionable, and technically feasible project specifications.

## Requirements Specification Template

### 1. Project Overview
- **Project Name**: 
- **Brief Description**:
- **Primary Objective**:

### 2. Stakeholder Analysis
- **Target Users**:
  - Primary user personas
  - User roles and access levels
- **Stakeholder Goals**:
  - Business objectives
  - User experience expectations

### 3. Functional Requirements
#### Core Features
- **Feature 1**:
  - Description
  - User interactions
  - Acceptance criteria
- **Feature 2**:
  - Description
  - User interactions
  - Acceptance criteria

### 4. Technical Requirements
#### System Architecture
- **Frontend**:
  - Preferred React component structure
  - State management approach
- **Backend**:
  - Data models
  - Authentication requirements
  - API endpoint specifications

#### Performance Expectations
- **Response Time**:
- **Concurrent Users**:
- **Data Volume**:

### 5. Non-Functional Requirements
- **Security**:
  - Authentication mechanisms
  - Data protection strategies
- **Scalability**:
  - Expected growth
  - Horizontal scaling considerations
- **Compliance**:
  - Regulatory requirements
  - Data privacy standards

### 6. Integration Requirements
- **External Services**:
  - Third-party API integrations
  - Payment gateways
  - Authentication providers
- **Data Sources**:
  - Internal and external data connections

### 7. Constraints and Limitations
- **Technical Constraints**:
  - Browser compatibility
  - Device support
- **Regulatory Constraints**:
  - Industry-specific regulations
  - Data handling restrictions

### 8. Wasp-Specific Considerations
- **Recommended Wasp Features**:
  - Declarative routing
  - Built-in authentication
  - Client-server code co-location
- **Potential Challenges**:
  - Identified complexity areas
  - Mitigation strategies

## Requirements Validation Checklist
- [ ] Requirements are specific and measurable
- [ ] Aligned with Wasp language capabilities
- [ ] Reviewed by technical and business stakeholders
- [ ] Feasibility assessed

## Example Wasp-Specific Requirement
```wasp
app MyProject {
  title: "Project Management App",
  auth: {
    methods: {
      email: {},
      google: {}
    }
  }
}

route DashboardRoute { 
  path: "/dashboard", 
  component: Dashboard 
}
```

## Revision History
- **Version**: 1.0.0
- **Last Updated**: 2024-12-31
- **Author**: AI Assistant

## Guidance Notes
- Be as specific as possible
- Focus on user value
- Consider Wasp's unique capabilities
- Remain flexible for iterative development
