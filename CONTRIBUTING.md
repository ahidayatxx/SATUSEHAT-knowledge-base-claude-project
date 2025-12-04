# Contributing to SATUSEHAT-Compliant EMR Development Framework

Thank you for your interest in contributing! This framework benefits greatly from real-world implementation experiences shared by the community.

## 🎯 Types of Contributions

### 1. **Implementation Experiences**
Share lessons learned from your SATUSEHAT integration:
- Architecture decisions and trade-offs
- Performance optimization techniques
- Error handling patterns
- Workflow customizations

### 2. **Code Examples**
Provide working code snippets for:
- Specific FHIR resource types
- Integration patterns
- Error handling
- Testing utilities

### 3. **Documentation Improvements**
Enhance existing documentation:
- Clarify ambiguous sections
- Add diagrams and visualizations
- Translate content to Bahasa Indonesia
- Update for new SATUSEHAT versions

### 4. **Bug Reports**
Report issues with:
- Code examples that don't work
- Outdated SATUSEHAT API references
- Incorrect FHIR resource mappings
- Documentation errors

## 📋 Contribution Guidelines

### Before You Start

1. **Search Existing Issues**: Check if your contribution has already been discussed
2. **Read the Documentation**: Familiarize yourself with the framework structure
3. **Test Against SATUSEHAT**: Validate code examples in staging environment

### Code Contributions

#### Requirements
- **Tested**: All code must be tested against SATUSEHAT staging environment
- **Documented**: Include inline comments and usage examples
- **Error Handling**: Implement proper exception handling
- **Style**: Follow PEP 8 for Python code

#### Example Contribution Structure
```python
"""
FHIR Resource: ServiceRequest (Laboratory Orders)
Use Case: Rawat Jalan Laboratory Testing
SATUSEHAT Version: 7.20
Last Updated: 2025-12-02
Author: Your Name
"""

def create_lab_order(encounter_id: int, loinc_code: str, specimen_type: str):
    """
    Create laboratory service request
    
    Args:
        encounter_id: Internal encounter identifier
        loinc_code: LOINC code for test (e.g., '2339-0' for Glucose)
        specimen_type: Type of specimen (e.g., 'blood', 'urine')
    
    Returns:
        ServiceRequest.id from SATUSEHAT
    
    Example:
        >>> create_lab_order(123, '2339-0', 'blood')
        'ServiceRequest/abc123def456'
    
    Raises:
        ValueError: If LOINC code not found
        APIError: If SATUSEHAT transmission fails
    """
    # Implementation here
    pass
```

### Documentation Contributions

#### Structure
- Use Markdown format
- Include code examples for technical content
- Add cross-references to related sections
- Maintain consistent terminology

#### Style Guide
- **Headers**: Use sentence case
- **Code Blocks**: Always specify language (```python, ```json, ```bash)
- **Lists**: Use consistent bullet/numbering style
- **Links**: Use descriptive text, not "click here"

### Testing Requirements

Before submitting:

```bash
# 1. Validate code against SATUSEHAT staging
python test_your_contribution.py

# 2. Check documentation formatting
markdown-lint docs/your_contribution.md

# 3. Verify no sensitive data included
grep -r "client_id\|client_secret\|password" .
```

## 🔄 Contribution Process

### 1. Fork the Repository
```bash
git clone https://github.com/YOUR_USERNAME/SATUSEHAT-Compliant_EMR_Development_Framework.git
cd SATUSEHAT-Compliant_EMR_Development_Framework
```

### 2. Create a Feature Branch
```bash
git checkout -b feature/your-contribution-name
# Or for documentation:
git checkout -b docs/your-doc-improvement
# Or for bug fixes:
git checkout -b fix/issue-description
```

### 3. Make Your Changes
- Keep commits atomic (one logical change per commit)
- Write clear commit messages

### 4. Commit with Descriptive Messages
```bash
git commit -m "feat: Add ServiceRequest implementation for lab orders"
# Or:
git commit -m "docs: Clarify medication workflow prerequisites"
# Or:
git commit -m "fix: Correct ICD-10 terminology binding in Condition example"
```

### 5. Push to Your Fork
```bash
git push origin feature/your-contribution-name
```

### 6. Create Pull Request
- Use the PR template provided
- Link related issues
- Describe testing performed
- Include before/after examples if applicable

## ✅ Pull Request Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Code example
- [ ] Documentation improvement
- [ ] Bug fix
- [ ] Architecture proposal

## Testing
Describe how you tested these changes:
- [ ] Tested against SATUSEHAT staging
- [ ] Code runs without errors
- [ ] Documentation renders correctly

## Checklist
- [ ] Code follows style guidelines
- [ ] Comments added for complex logic
- [ ] Documentation updated
- [ ] No sensitive data included
- [ ] Related issues linked

## Screenshots (if applicable)

## Additional Context
```

## 🚫 What NOT to Contribute

### Security & Privacy
- **NO** actual client credentials (use placeholders: `YOUR_CLIENT_ID`)
- **NO** real patient data (use SATUSEHAT dummy patients)
- **NO** production URLs or identifiers
- **NO** proprietary algorithms from commercial EMRs

### Quality Standards
- **NO** untested code examples
- **NO** code without error handling
- **NO** undocumented functions
- **NO** external dependencies without justification

## 🎖️ Recognition

Contributors will be acknowledged in:
- README.md Contributors section
- Release notes for version including contribution
- Documentation pages for major contributions

## 📞 Questions?

- **Framework Questions**: Open a [Discussion](https://github.com/ahidayatxx/SATUSEHAT-Compliant_EMR_Development_Framework/discussions)
- **Bug Reports**: Create an [Issue](https://github.com/ahidayatxx/SATUSEHAT-Compliant_EMR_Development_Framework/issues)
- **Feature Proposals**: Open a [Discussion](https://github.com/ahidayatxx/SATUSEHAT-Compliant_EMR_Development_Framework/discussions) first

## 📜 License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

Thank you for helping improve SATUSEHAT implementation for the Indonesian healthcare community! 🙏
