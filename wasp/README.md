# Wasp Project Development Guide

## Project Structure
```
wasp/
├── README.md                       # Project overview and getting started
├── templates/                      # Reusable code templates
│   ├── wasp-config-templates.md    # Wasp configuration examples
│   ├── environment-setup-guide.md  # Environment setup instructions
│   ├── project-structure-guide.md  # Project structure documentation
│   ├── integration-patterns.md     # Common integration patterns
│   └── copy-paste-blocks.md        # Ready-to-use code blocks
│
├── workflows/                      # Development workflows
│   ├── master-workflow.md         # Main development process
│   ├── transition-templates.md    # Stage transition guides
│   ├── validation-checklist.md    # Validation requirements
│   │
│   ├── initialization/           # Project setup
│   │   └── wasp-project-initialization-guide.md
│   │
│   ├── development/             # Development phase
│   │   └── feature-development-workflow.md
│   │
│   ├── mvp/                     # MVP phase
│   │   ├── mvp-deployment-guide.md
│   │   └── mvp-testing-workflow.md
│   │
│   └── post-mvp/               # Post-MVP phase
│       ├── maintenance-workflow.md
│       └── iteration-planning-guide.md
│
├── guides/                         # Comprehensive development guides
│   ├── feature-development-workflow.md   # Feature implementation guide
│   ├── wasp-testing-guide.md             # Testing strategies
│   ├── mvp-deployment-launch-checklist.md # Deployment and launch guide
│   ├── iteration-planning-guide.md       # Iteration management
│   └── maintenance-workflow-guide.md     # Long-term maintenance
│
├── prompts/                    # Development prompts
│   └── project-stages-checklist.md
│
└── ...
```

## Quick Start

1. **Project Setup**
   ```bash
   wasp new my-app
   cd my-app
   ```

2. **Development Workflow**
   - Follow [master-workflow.md](workflows/master-workflow.md)
   - Use templates from [templates/](templates/)
   - Validate using [validation-checklist.md](workflows/validation-checklist.md)

## Comprehensive Development Guides

### Development Lifecycle Guides
1. [Feature Development Workflow](guides/feature-development-workflow.md)
   - Structured approach to implementing features
   - Wasp-specific implementation patterns
   - Best practices for feature creation

2. [Wasp Testing Guide](guides/wasp-testing-guide.md)
   - Comprehensive testing strategies
   - Unit, integration, and E2E testing
   - Performance and security testing

3. [MVP Deployment and Launch Checklist](guides/mvp-deployment-launch-checklist.md)
   - Deployment preparation
   - Infrastructure setup
   - Launch readiness validation

4. [Iteration Planning Guide](guides/iteration-planning-guide.md)
   - Feedback collection mechanisms
   - Feature prioritization
   - Continuous improvement strategies

5. [Maintenance Workflow Guide](guides/maintenance-workflow-guide.md)
   - System health monitoring
   - Performance optimization
   - Long-term application maintenance

## Guide Categories

### Development Process
- Comprehensive feature implementation guidelines
- Wasp-specific development patterns
- Best practices and common pitfalls

### Quality Assurance
- Testing strategies
- Performance validation
- Security considerations

### Deployment
- Launch preparation
- Infrastructure configuration
- Deployment workflows

### Iteration Management
- Feedback-driven development
- Feature prioritization
- Continuous improvement

### Maintenance
- System health monitoring
- Performance optimization
- Long-term application sustainability

## Recommended Workflow

1. **Project Initialization**
   - Review [Project Stages Checklist](prompts/project-stages-checklist.md)
   - Follow [Wasp Project Initialization Guide](workflows/initialization/wasp-project-initialization-guide.md)

2. **Feature Development**
   - Use [Feature Development Workflow](guides/feature-development-workflow.md)
   - Implement using [Testing Guide](guides/wasp-testing-guide.md)

3. **MVP Preparation**
   - Follow [MVP Deployment Checklist](guides/mvp-deployment-launch-checklist.md)
   - Validate using [Iteration Planning Guide](guides/iteration-planning-guide.md)

4. **Post-Launch**
   - Maintain using [Maintenance Workflow Guide](guides/maintenance-workflow-guide.md)

## Best Practices

- Always refer to the latest guide versions
- Adapt guides to your specific project needs
- Continuously update and refine your workflow
- Encourage team collaboration and knowledge sharing

## Documentation Overview

### 1. Templates
- **Configuration Templates**: Standard Wasp configurations
- **Environment Setup**: Development environment setup
- **Project Structure**: Recommended project organization
- **Integration Patterns**: Common integration solutions
- **Code Blocks**: Ready-to-use code snippets

### 2. Workflows
- **Master Workflow**: Complete development process
- **Transition Templates**: Stage-to-stage transitions
- **Validation Checklist**: Quality requirements
- **Phase-specific Guides**: Detailed phase instructions

### 3. Development Phases

#### Initialization
- Project setup
- Environment configuration
- Initial structure

#### Development
- Feature implementation
- Testing
- Documentation

#### MVP
- Deployment
- Testing
- Launch preparation

#### Post-MVP
- Maintenance
- Scaling
- Iteration planning

## Contributing

1. Review the documentation structure
2. Follow the workflow guidelines
3. Use provided templates
4. Maintain documentation

## Support

- [Wasp Documentation](https://wasp-lang.dev/docs)
- [GitHub Repository](https://github.com/wasp-lang/wasp)
- [Community Discord](https://discord.gg/rzdnErX)
