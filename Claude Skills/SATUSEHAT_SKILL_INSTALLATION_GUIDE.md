# SATUSEHAT Knowledge Base Skill

## Overview

The SATUSEHAT Knowledge Base skill transforms Claude into a specialized expert on Indonesia's national health information exchange platform. This skill enables dynamic, real-time documentation retrieval using Brave and Tavily MCP servers, eliminating the need for manual knowledge base maintenance.

**Key Capabilities:**
- Real-time access to official SATUSEHAT documentation
- Intelligent routing between Brave (overview) and Tavily (technical details)
- Comprehensive coverage of FHIR resources, API integration, and clinical workflows
- Built-in quick reference guides and workflow examples
- Optimized search patterns for efficient information retrieval

## Prerequisites

Before installing this skill, ensure you have:

1. **Claude Desktop** installed on your system
2. **Brave MCP Server** configured with valid API key
3. **Tavily MCP Server** configured with valid API key
4. Working knowledge of MCP (Model Context Protocol) configuration

For detailed MCP setup instructions, refer to:
- [Claude Desktop Installation Guide](https://github.com/ahidayatxx/SATUSEHAT-knowledge-base-claude-project/blob/master/CLAUDE_DESKTOP_INSTALLATION.md)
- [MCP Configuration Guide](https://github.com/ahidayatxx/SATUSEHAT-knowledge-base-claude-project/blob/master/DESKTOP_CONFIGURATION.md)

## Installation

### Step 1: Install the Skill

1. Download the `SATUSEHAT-Knowledge-Base-Skill-v1.0.0.zip` file
2. Open Claude Desktop application
3. Go to **Settings** → **Capabilities** → **Skills**
4. Click **"Upload skill"** button
5. Select the `SATUSEHAT-Knowledge-Base-Skill-v1.0.0.zip` file
6. Wait for the upload to complete
7. Toggle the skill **ON** to enable it

**Important Notes:**
- The ZIP file must contain a folder with the `SKILL.md` file inside
- Do NOT extract the ZIP file before uploading
- Ensure you have administrative permissions if using a work account
- Skills may need to be enabled by organization administrators for enterprise accounts

### Step 2: Verify Installation

1. Open Claude Desktop
2. Start a new conversation
3. Ask: "Do you have access to SATUSEHAT documentation?"
4. Claude should confirm the skill is available

### Step 3: Test Functionality

Try these test queries to verify the skill is working:

**Test 1 - General Overview (Brave):**
```
"Apa itu platform SATUSEHAT secara umum?"
```
Expected: High-level explanation of SATUSEHAT ecosystem

**Test 2 - Technical Details (Tavily):**
```
"Bagaimana implementasi Patient resource untuk rawat jalan di SATUSEHAT?"
```
Expected: Detailed FHIR implementation guidance with code examples

**Test 3 - API Integration:**
```
"Jelaskan proses authentication SATUSEHAT dan berikan contoh request"
```
Expected: OAuth2 authentication flow with example code

## How to Use

### Basic Usage

Simply ask questions about SATUSEHAT in natural language. The skill automatically:
1. Determines the appropriate search tool (Brave or Tavily)
2. Constructs optimized search queries
3. Retrieves relevant documentation
4. Synthesizes comprehensive answers with proper citations

### Query Types

**General Questions (Auto-routed to Brave):**
- "Apa itu SATUSEHAT?"
- "Komponen utama platform SATUSEHAT"
- "Perbedaan FHIR R4 dengan implementasi SATUSEHAT"

**Technical Implementation (Auto-routed to Tavily):**
- "Patient resource FHIR SATUSEHAT mandatory fields"
- "SATUSEHAT authentication OAuth2 flow"
- "Alur interoperabilitas rawat jalan lengkap"

**API Integration:**
- "SATUSEHAT API POST Encounter example"
- "MedicationRequest prescription payload structure"
- "Observation vital signs implementation"

**Workflow Guidance:**
- "Bagaimana cara mendaftarkan pasien baru di SATUSEHAT?"
- "Alur pengiriman data laboratorium ke SATUSEHAT"
- "Implementasi resep elektronik di SATUSEHAT"

### Advanced Usage

**Load Reference Files:**
When you need comprehensive reference information, request:
- "Load the SATUSEHAT quick reference" → Access resource tables, endpoints, terminology codes
- "Show workflow examples" → View complete implementation scenarios
- "Explain search patterns" → Understand query construction strategies

**Domain-Specific Searches:**
Specify documentation sections for targeted results:
```
"Search SATUSEHAT FHIR resource specifications for Observation"
"Find authentication documentation in API catalogue"
"Look up Master Patient Index REST API details"
```

**Multi-Step Workflows:**
Request complete integration workflows:
```
"Walk me through complete outpatient visit workflow in SATUSEHAT"
"Explain laboratory order to results submission process"
"Guide me through medication prescription and dispensing"
```

## Skill Structure

### SKILL.md
Core instruction file containing:
- Search strategy and tool selection logic
- Response format guidelines
- Common query patterns
- Technical implementation notes
- Quality assurance criteria

### References Directory

**quick_reference.md** (Load when needed for lookups)
- FHIR resource quick reference table
- API endpoints overview
- Master data resources
- Terminology code systems
- Common error codes
- Integration workflow checklist

**search_patterns.md** (Load for advanced query construction)
- Query templates by topic
- Domain-specific search strategies
- Effective domain filtering
- Query refinement patterns
- Multi-tool search strategies
- Language-specific patterns

**workflow_examples.md** (Load for implementation examples)
- New patient registration & outpatient visit
- Laboratory order & results
- Medication prescription & dispensing
- Emergency department workflow
- Inpatient admission
- Common patterns across workflows

## Features

### 1. Intelligent Tool Routing

The skill automatically selects the optimal search tool based on query characteristics:

- **Brave Search**: Platform overviews, general concepts, policy context
- **Tavily Search**: Technical specifications, code examples, implementation details

### 2. Domain-Focused Retrieval

All searches prioritize official SATUSEHAT documentation:
- Primary source: `satusehat.kemkes.go.id/platform/docs/`
- Structured by topic: playbook, interoperability, FHIR, API catalogue, master data

### 3. Multi-Language Support

Responds appropriately in:
- **Bahasa Indonesia**: Formal language following EYD V guidelines
- **English**: Technical documentation and API references
- **Mixed**: Indonesian context with English technical terms

### 4. Context-Aware Responses

Answers include:
- Direct information requested
- Relevant background context
- Practical code examples
- Official documentation links
- Related topic suggestions

### 5. Built-in Quick References

Access pre-compiled reference material without searching:
- FHIR resource relationships
- API endpoint specifications
- Terminology system URLs
- Common error codes
- Integration checklists

## Use Cases

### For Healthcare IT Developers

**Scenario**: Implementing SATUSEHAT integration in hospital information system

**Queries**:
```
1. "What are the prerequisite resources I need to register first?"
2. "Show me complete Patient resource structure with mandatory fields"
3. "How do I create an outpatient encounter?"
4. "Provide authentication flow with code examples"
```

### For System Integrators

**Scenario**: Building interoperability layer for clinic management system

**Queries**:
```
1. "Explain the difference between MedicationRequest and MedicationDispense"
2. "What's the sequence for submitting laboratory results?"
3. "How do I handle medication dispensing workflow?"
4. "Show me error handling best practices"
```

### For Technical Architects

**Scenario**: Designing SATUSEHAT integration architecture

**Queries**:
```
1. "What are the key components of SATUSEHAT platform?"
2. "Explain the master data dependencies (MPI, MSI, KFA)"
3. "How does FHIR R4 implementation differ in Indonesian context?"
4. "What are the environment URLs for staging and production?"
```

### For Regulatory Compliance Officers

**Scenario**: Understanding SATUSEHAT requirements for facility accreditation

**Queries**:
```
1. "What are the mandatory resources for outpatient interoperability?"
2. "Which terminology standards are required by SATUSEHAT?"
3. "What data privacy requirements apply to SATUSEHAT integration?"
4. "How do I obtain API credentials for my facility?"
```

## Best Practices

### 1. Start with General, Then Dive Deep

```
Step 1: "Apa itu Encounter resource di SATUSEHAT?"
Step 2: "Show me Encounter structure for outpatient visit"
Step 3: "Provide complete Encounter creation example with all mandatory fields"
```

### 2. Specify Workflow Context

Instead of: "How to create patient?"
Better: "How to create patient record for outpatient registration in SATUSEHAT?"

### 3. Request Code Examples

Add "with example" or "show payload" to get implementation-ready code:
```
"Explain SATUSEHAT authentication with complete request example"
"Show MedicationRequest payload structure for oral medication"
```

### 4. Load References When Needed

For extensive lookups, explicitly request reference files:
```
"Load quick reference to see all FHIR resources"
"Show me workflow examples for laboratory integration"
```

### 5. Verify with Official Documentation

Always cross-reference critical implementation details with official documentation links provided in responses.

## Troubleshooting

### Issue: Skill Not Triggering

**Solution**:
- Verify MCP servers are running (Brave and Tavily)
- Check API keys are valid and not expired
- Restart Claude Desktop
- Ensure queries mention "SATUSEHAT" explicitly

### Issue: Incomplete or Outdated Information

**Solution**:
- Request fresh search: "Search for latest SATUSEHAT [topic] documentation"
- Specify documentation version if needed
- Verify you're accessing production vs staging documentation

### Issue: Search Returns No Results

**Solution**:
- Rephrase query with different terminology
- Try broader search first, then narrow down
- Check if feature exists in SATUSEHAT (some features may not be implemented)
- Ask for alternative approaches

### Issue: Mixed Language Responses

**Solution**:
- Specify preferred language explicitly: "Explain in Indonesian" or "Respond in English"
- Use consistent language in your queries
- Technical terms may remain in English as per SATUSEHAT documentation standards

## Maintenance

### Keeping Information Current

This skill retrieves real-time documentation, so:
- No manual updates required
- Documentation changes are automatically reflected
- API credentials must be maintained separately by users
- SATUSEHAT platform updates are accessed dynamically

### Monitoring Performance

Track these metrics:
- Response accuracy and relevance
- Search tool selection appropriateness
- Documentation coverage completeness
- Query resolution success rate

### Providing Feedback

To improve this skill:
1. Note queries that don't return expected results
2. Identify missing or outdated documentation areas
3. Suggest additional reference material needs
4. Report any inconsistencies with official SATUSEHAT documentation

## Limitations

### Scope Limitations

- Retrieves publicly available documentation only
- Cannot access internal SATUSEHAT systems or data
- API credentials must be obtained through official channels
- Implementation support limited to documentation interpretation

### Technical Limitations

- Dependent on MCP server availability and API quotas
- Search results quality depends on official documentation quality
- Cannot execute actual API calls or test implementations
- Cannot access real-time SATUSEHAT platform status

### Disclaimer

This skill provides documentation guidance based on official SATUSEHAT resources. For:
- Official support: Contact SATUSEHAT helpdesk
- API credentials: Register at satusehat.kemkes.go.id/platform
- Regulatory compliance: Consult with Kementerian Kesehatan RI
- Production deployment: Follow official implementation guidelines

## Support and Resources

### Official SATUSEHAT Resources

- **Documentation Portal**: https://satusehat.kemkes.go.id/platform/docs/
- **Platform Homepage**: https://satusehat.kemkes.go.id/
- **Registration**: https://satusehat.kemkes.go.id/platform

### Related Projects

- **GitHub Repository**: https://github.com/ahidayatxx/SATUSEHAT-knowledge-base-claude-project
- **MCP Configuration**: See PROJECT_SETUP.md in repository

### Community and Support

For questions about this skill or SATUSEHAT integration:
1. Review skill reference files first
2. Consult official SATUSEHAT documentation
3. Contact SATUSEHAT support for platform-specific issues
4. Engage with Indonesian digital health community

## Version Information

- **Skill Version**: 1.0.0
- **Created**: December 2024
- **Compatible With**: Claude Desktop with MCP support
- **SATUSEHAT Documentation**: Version 7.20+ (as of December 2024)
- **FHIR Version**: R4

## License and Attribution

This skill provides access to public SATUSEHAT documentation from Kementerian Kesehatan Republik Indonesia. All SATUSEHAT-related content, terminology, and standards remain property of their respective owners.

**Skill Development**: Based on MCP integration patterns from the SATUSEHAT-knowledge-base-claude-project repository.

---

**Installation Date**: _[Auto-populated by Claude Desktop]_
**Last Updated**: December 2, 2024
