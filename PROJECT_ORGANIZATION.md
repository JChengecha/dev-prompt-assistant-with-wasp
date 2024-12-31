# Project Prompt Library Organizational Strategy

## 1. Prompt Categories Directory Structure
```
prompts/
├── architecture/
│   ├── general/
│   └── assistant-specific/
├── code-analysis/
│   ├── general/
│   └── assistant-specific/
├── coding/
│   ├── general/
│   └── assistant-specific/
├── documentation/
│   ├── general/
│   └── assistant-specific/
├── learning/
│   ├── general/
│   └── assistant-specific/
├── maintenance/
│   ├── general/
│   └── assistant-specific/
├── planning/
│   ├── general/
│   └── assistant-specific/
│       └── wasp/
│           └── project-initialization-prompt.md
├── requirements/
│   ├── general/
│   └── assistant-specific/
│       └── wasp/
│           └── requirements-gathering-prompt.md
├── testing/
│   ├── general/
│   └── assistant-specific/
└── [other prompt categories]
```

## 2. Workflow Categories Directory Structure
```
workflows/
├── general/                   # Generic workflows applicable across assistants
│   ├── sprint/                # Sprint workflows not tied to specific assistants
│   └── maintenance/           # Maintenance workflows not tied to specific assistants
└── assistant-specific/        # Assistant-specific workflows
    ├── aider/
    │   ├── sprint/
    │   └── maintenance/
    ├── github-copilot/
    │   ├── sprint/
    │   └── maintenance/
    ├── claude/
    │   ├── sprint/
    │   └── maintenance/
    └── wasp/
        ├── sprint/
        │   ├── project-scaffolding-workflow.md
        │   ├── post-scaffolding-workflow.md
        │   └── iterative-refinement-workflow.md
        ├── features/
        │   └── wasp-feature-tracking.md
        └── README.md
```

## 3. Workflow Strategy
- Workflows are categorized by:
  * Development phase (sprint, maintenance)
  * Generality (general vs. assistant-specific)
- Each workflow represents a connected chain of prompts
- Workflows can:
  * Span multiple prompt categories
  * Include verification and validation steps
  * Support different development scenarios

## 4. Metadata Strategy
- Each prompt has a corresponding `.meta.md` file
- Metadata files provide:
  * AI assistant compatibility
  * SDLC phase information
  * Complexity ratings
  * Usage guidelines

## 5. Key Design Principles
- Flexibility: Easy to add new prompts and workflows
- Discoverability: Clear, intuitive structure
- Extensibility: Support for various development contexts
- Consistency: Standardized prompt and workflow organization

## Component Organization

### 1. Workflows
Organized collections of development processes:
- Project scaffolding
- Development sprints
- Feature implementation
- Testing procedures

### 2. Prompts
Structured AI instructions for:
- Project initialization
- Requirements gathering
- Technical specifications
- Implementation guidance

### 3. Documentation
Comprehensive guides covering:
- Getting started
- Development guidelines
- Best practices
- Troubleshooting

## Workflow Integration

### Wasp Development
1. Project Initialization
   - Requirements gathering
   - Technical setup
   - Configuration

2. Development Process
   - Feature implementation
   - Testing
   - Documentation
   - Deployment

3. Maintenance
   - Updates
   - Optimization
   - Feature tracking

## Version Control

### Current Versions
- Wasp: 0.11.x
- Workflow: 1.0.0
- Documentation: 1.0.0

### Compatibility
- Node.js: 16.x+
- React: 18.x
- Prisma: Latest stable

## Best Practices

### 1. File Organization
- Maintain clear directory structure
- Use consistent naming conventions
- Keep related files together
- Document organization changes

### 2. Documentation
- Keep README files updated
- Document version changes
- Maintain clear guidelines
- Include examples

### 3. Version Control
- Track compatibility requirements
- Document breaking changes
- Maintain change logs
- Version documentation

## Contributing Guidelines

### 1. Adding Content
- Follow existing structure
- Maintain documentation
- Include version info
- Test thoroughly

### 2. Updates
- Keep versions aligned
- Update documentation
- Test changes
- Review dependencies

## Support Resources

### Documentation
- Getting started guides
- Development guidelines
- Troubleshooting guides
- API references

### Community
- Issue tracking
- Discussion forums
- Feature requests
- Bug reports

## 6. Wasp Project Guides

### Development Guides
- [Feature Development Workflow](/wasp/guides/feature-development-workflow.md)
- [Wasp Testing Guide](/wasp/guides/wasp-testing-guide.md)
- [MVP Deployment and Launch Checklist](/wasp/guides/mvp-deployment-launch-checklist.md)
- [Iteration Planning Guide](/wasp/guides/iteration-planning-guide.md)
- [Maintenance Workflow Guide](/wasp/guides/maintenance-workflow-guide.md)

### Guide Categories
- **Development Process**: Comprehensive guides for feature implementation
- **Quality Assurance**: Testing and validation strategies
- **Deployment**: Launch and release management
- **Iteration Management**: Planning and continuous improvement
- **Maintenance**: Long-term application health and optimization
