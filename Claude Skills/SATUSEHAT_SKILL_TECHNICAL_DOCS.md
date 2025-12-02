# SATUSEHAT Knowledge Base Skill - Technical Documentation

## Architecture Overview

### Design Philosophy

This skill implements a **dynamic documentation retrieval pattern** rather than a static knowledge base approach. Key architectural decisions:

1. **Zero Maintenance Burden**: No manual documentation updates required
2. **Real-Time Accuracy**: Always retrieves current SATUSEHAT documentation
3. **Intelligent Routing**: Automatic tool selection based on query characteristics
4. **Progressive Disclosure**: Reference files loaded only when needed
5. **Token Efficiency**: Minimal context window usage while maintaining comprehensive coverage

### Skill Components

```
satusehat-knowledge-base/
├── SKILL.md                           # Core instruction file (~8KB)
│   ├── Frontmatter (YAML)            # Triggering mechanism
│   └── Body (Markdown)               # Search strategy & response format
│
└── references/                        # Loaded on-demand
    ├── quick_reference.md            # Lookup tables & endpoints (~9KB)
    ├── search_patterns.md            # Query construction guide (~9.5KB)
    └── workflow_examples.md          # Implementation scenarios (~14KB)
```

**Total Package Size**: ~40KB (minimal context footprint)

## How It Works

### Triggering Mechanism

The skill activates when queries match patterns in the `description` field:

**Primary Triggers**:
- Explicit mentions: "SATUSEHAT", "SATUSEHAT Platform"
- Technical terms: "FHIR resources", "interoperability", "MPI", "MSI", "KFA"
- Workflow contexts: "rawat jalan", "IGD", "rawat inap", "kefarmasian"
- Standards: "ICD-10", "LOINC", "SNOMED-CT", "HL7 FHIR R4"

**Context Awareness**:
- Indonesian healthcare digital transformation
- Indonesia's national health information system
- Healthcare data exchange in Indonesia

### Tool Selection Logic

The skill implements a **decision tree approach** for tool selection:

```
Query Analysis
    │
    ├─── Contains "overview" OR "apa itu" OR "general"?
    │    └─── → Use Brave Search
    │
    ├─── Requests "implementation" OR "example" OR "API"?
    │    └─── → Use Tavily Search
    │
    ├─── Asks about "workflow" OR "alur" OR "proses"?
    │    └─── → Use Tavily Search (specific domain)
    │
    └─── Needs "documentation" OR "specification"?
         └─── → Use Tavily Search (targeted domain)
```

### Search Query Construction

**Brave Search Patterns**:
```python
# Pattern: General understanding
query = f"SATUSEHAT {topic} overview"
# Example: "SATUSEHAT platform overview"

# Pattern: Conceptual questions
query = f"Apa itu {concept} SATUSEHAT"
# Example: "Apa itu Master Patient Index SATUSEHAT"
```

**Tavily Search Patterns**:
```python
# Pattern: Resource implementation
query = f"{resource} FHIR SATUSEHAT {context}"
# Example: "Patient FHIR SATUSEHAT rawat jalan"

# Pattern: API details
query = f"SATUSEHAT {endpoint} API {action}"
# Example: "SATUSEHAT authentication API token"

# Domain filtering
include_domains = ["satusehat.kemkes.go.id/platform/docs/id/"]
```

### Response Assembly

```
1. Search Execution
   ├─ Construct optimized query
   ├─ Select appropriate tool
   └─ Apply domain filtering

2. Result Processing
   ├─ Extract relevant information
   ├─ Synthesize from multiple sources
   └─ Verify completeness

3. Response Formatting
   ├─ Direct answer (Indonesian/English)
   ├─ Context and background
   ├─ Code examples if applicable
   ├─ Official documentation links
   └─ Related topics suggestions
```

## MCP Integration

### Required MCP Servers

**Brave MCP Server**:
```json
{
  "mcpServers": {
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "YOUR_BRAVE_API_KEY"
      }
    }
  }
}
```

**Tavily MCP Server**:
```json
{
  "mcpServers": {
    "tavily": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "TAVILY_API_KEY",
        "mcp/tavily"
      ],
      "env": {
        "TAVILY_API_KEY": "YOUR_TAVILY_API_KEY"
      }
    }
  }
}
```

### MCP Tool Usage

**brave_web_search**:
- General documentation searches
- High-level platform information
- Policy and regulatory context
- News and announcements

**tavily-search**:
- Technical documentation extraction
- API specifications
- Code examples and payloads
- Implementation guides

**tavily-extract** (when needed):
- Deep content extraction from specific URLs
- Comprehensive documentation page retrieval
- Multiple page content synthesis

## Customization Guide

### Modifying Search Strategy

**File**: `SKILL.md` → Search Strategy section

**To add new tool selection criteria**:
```markdown
**Use [New Tool] for:**
- [Query pattern 1]
- [Query pattern 2]
- [Example query]
```

**To adjust domain focus**:
```markdown
Primary documentation source: `your-domain.com/docs/`

Key documentation sections:
- `/section1/` - Description
- `/section2/` - Description
```

### Adding Reference Files

**Create new reference file**:
```bash
references/
└── new_reference.md
```

**Reference from SKILL.md**:
```markdown
## Advanced Topic

For detailed information, see [New Reference](references/new_reference.md)
```

**File structure**:
```markdown
# Reference Title

Load this file when [specific conditions]

## Table of Contents
1. [Section 1](#section-1)
2. [Section 2](#section-2)

## Section 1
[Content organized for easy scanning]

## Section 2
[Tables, code examples, quick lookups]
```

### Extending Query Patterns

**File**: `references/search_patterns.md`

**Add new query template**:
```markdown
#### New Topic Queries
\`\`\`
Template: "[Topic] SATUSEHAT [Context]"
Examples:
- "Example query 1"
- "Example query 2"
\`\`\`
```

### Adding Workflow Examples

**File**: `references/workflow_examples.md`

**New workflow structure**:
```markdown
## New Workflow Name

### Scenario
[Describe the use case]

### Workflow Sequence
\`\`\`
1. Step 1 description
2. Step 2 description
3. Step 3 description
\`\`\`

### Implementation

#### Step 1: [Action]
\`\`\`
Query: "[Optimized search query]"
Tool: [Brave/Tavily]
Domain: [Target documentation section]

[Code example or payload]
\`\`\`
```

## Performance Optimization

### Context Window Management

**Strategy**: Progressive disclosure
- SKILL.md (~8KB): Always loaded
- References (~32KB total): Loaded on-demand
- Search results: Dynamic, cached by MCP

**Token Budget**:
```
Base skill:           ~2,000 tokens
Quick reference:      ~2,500 tokens (optional)
Search patterns:      ~2,800 tokens (optional)
Workflow examples:    ~4,000 tokens (optional)
Search results:       ~3,000-5,000 tokens (per search)
```

### Search Efficiency

**Optimization techniques**:

1. **Domain filtering**: Reduces irrelevant results
```python
include_domains=["satusehat.kemkes.go.id/platform/docs/"]
```

2. **Concise queries**: Shorter queries, better results
```python
# Poor: "How do I implement a patient resource in SATUSEHAT?"
# Better: "Patient FHIR SATUSEHAT implementation"
```

3. **Multi-stage search**: Broad → Narrow
```python
# Stage 1: "SATUSEHAT Encounter overview"
# Stage 2: "SATUSEHAT Encounter rawat jalan create"
```

4. **Result limits**: Control response size
```python
max_results=10  # Brave
max_results=5   # Tavily (more verbose)
```

### Caching Strategy

**MCP-level caching**:
- Search results cached by MCP servers
- Token refresh handled automatically
- Domain-filtered results cached longer

**Skill-level optimization**:
- Reference files loaded once per conversation
- Repeated queries leverage MCP cache
- Common patterns pre-defined in reference files

## Testing and Validation

### Unit Testing

**Test categories**:

1. **Trigger Tests**: Verify skill activation
```
Test: "What is SATUSEHAT?"
Expected: Skill triggers, Brave search used

Test: "Patient FHIR resource implementation"
Expected: Skill triggers, Tavily search used
```

2. **Tool Selection Tests**: Validate routing logic
```
Test: "Apa itu platform SATUSEHAT?"
Expected: Brave search, Indonesian response

Test: "SATUSEHAT authentication OAuth2 flow"
Expected: Tavily search, code examples included
```

3. **Domain Filtering Tests**: Confirm targeting
```
Test: "[Query] site:satusehat.kemkes.go.id"
Expected: Results from official documentation only
```

4. **Reference Loading Tests**: Verify on-demand access
```
Test: "Load quick reference"
Expected: quick_reference.md content displayed

Test: "Show workflow examples"
Expected: workflow_examples.md content displayed
```

### Integration Testing

**End-to-end scenarios**:

```python
# Scenario 1: New developer onboarding
queries = [
    "What is SATUSEHAT platform?",  # Brave → Overview
    "How to register Organization?",  # Tavily → Prerequisites
    "Show Patient resource structure",  # Tavily + quick_reference
    "Complete outpatient workflow"  # Tavily + workflow_examples
]

# Validate:
# - Appropriate tool selection
# - Progressive information depth
# - Reference files loaded when needed
# - Response coherence across queries
```

**Performance benchmarks**:
```
Metric                  Target          Measured
---------------------------------------------------
Response time          < 5s            [Track actual]
Result relevance       > 90%           [User feedback]
Token efficiency       < 10K/response  [Monitor usage]
Cache hit rate         > 60%           [MCP metrics]
```

### Quality Assurance Checklist

- [ ] All queries trigger skill appropriately
- [ ] Tool selection matches query type
- [ ] Domain filtering returns official docs
- [ ] Code examples are complete and valid
- [ ] Indonesian responses use formal language
- [ ] English technical terms used correctly
- [ ] Documentation links are current
- [ ] Reference files load on request
- [ ] Error messages are helpful
- [ ] Response format is consistent

## Deployment

### Prerequisites Check

Before deployment, verify:

```bash
# 1. Claude Desktop installed
which claude-desktop || echo "Install Claude Desktop"

# 2. MCP servers configured
cat ~/.config/Claude/claude_desktop_config.json | grep -E "brave|tavily"

# 3. API keys valid
# Test Brave
curl -H "X-Subscription-Token: $BRAVE_API_KEY" \
  "https://api.search.brave.com/res/v1/web/search?q=test"

# Test Tavily
curl -X POST "https://api.tavily.com/search" \
  -H "Content-Type: application/json" \
  -d '{"api_key":"'$TAVILY_API_KEY'","query":"test"}'
```

### Installation Process

```bash
# 1. Package skill
python3 scripts/package_skill.py /path/to/satusehat-knowledge-base

# 2. Distribute .zip file
# Users upload via Settings → Capabilities → Skills → Upload skill

# 3. Verify installation
# Check ~/.claude/skills/ directory
```

### Post-Installation Verification

```bash
# 1. Test basic query
echo "Ask: Apa itu SATUSEHAT?"

# 2. Test technical query
echo "Ask: SATUSEHAT Patient FHIR resource"

# 3. Test reference loading
echo "Ask: Load quick reference"

# 4. Monitor logs (if available)
tail -f ~/.claude/logs/skills.log
```

## Troubleshooting

### Common Issues

**Issue**: Skill not triggering
```
Diagnosis: Query doesn't match trigger patterns
Solution: Add "SATUSEHAT" explicitly to query
Example: "Show patient structure" → "SATUSEHAT Patient structure"
```

**Issue**: Wrong tool selected
```
Diagnosis: Query ambiguity or pattern mismatch
Solution: Be more specific about information type
Example: "SATUSEHAT info" → "SATUSEHAT platform overview" (Brave)
         or "SATUSEHAT Patient implementation" (Tavily)
```

**Issue**: No search results
```
Diagnosis: Documentation doesn't exist or domain filter too restrictive
Solution: Broaden query or remove domain filter
Example: Try without include_domains parameter
```

**Issue**: Reference file not loading
```
Diagnosis: File path incorrect or file missing
Solution: Verify references/ directory structure
Check: All .md files present and readable
```

### Debug Mode

**Enable verbose logging** (if supported by Claude Desktop):
```json
{
  "debug": true,
  "logLevel": "verbose",
  "skills": {
    "satusehat-knowledge-base": {
      "debugSearchQueries": true,
      "logToolSelection": true
    }
  }
}
```

## Maintenance

### Regular Updates

**Quarterly review**:
- [ ] Verify SATUSEHAT documentation structure unchanged
- [ ] Test critical query patterns
- [ ] Update workflow examples if API changes
- [ ] Refresh terminology code examples
- [ ] Validate reference file accuracy

**As needed**:
- Update search patterns for new SATUSEHAT features
- Add workflow examples for new use cases
- Refine tool selection logic based on usage patterns
- Expand quick reference with frequently needed information

### Monitoring

**Track metrics**:
```python
metrics = {
    "queries_processed": count,
    "tool_selection": {
        "brave": brave_count,
        "tavily": tavily_count
    },
    "reference_loads": {
        "quick_reference": count,
        "search_patterns": count,
        "workflow_examples": count
    },
    "avg_response_time": seconds,
    "cache_hit_rate": percentage
}
```

### Version Control

**Semantic versioning**: MAJOR.MINOR.PATCH

```
1.0.0 → Initial release
1.0.1 → Patch: Minor bug fixes, documentation corrections
1.1.0 → Minor: New reference files, expanded query patterns
2.0.0 → Major: Changed search strategy, skill restructure
```

## Contributing

### Improvement Areas

**High Priority**:
- Additional workflow examples
- More comprehensive error handling guidance
- Expanded terminology mappings
- Advanced search patterns

**Medium Priority**:
- Integration troubleshooting scenarios
- Performance optimization tips
- Multi-language query support
- Vendor-specific implementation notes

**Low Priority**:
- Historical version documentation
- Community-contributed examples
- Extended use case scenarios

### Submission Guidelines

1. Test changes thoroughly
2. Update relevant reference files
3. Validate skill packaging
4. Document changes in commit message
5. Provide example queries demonstrating improvements

---

**Document Version**: 1.0.0  
**Last Updated**: December 2, 2024  
**Maintainer**: SATUSEHAT Integration Community
