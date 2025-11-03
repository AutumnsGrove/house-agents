# House Agents - Usage Examples

Practical examples for using house agents in real-world scenarios.

## Table of Contents

- [Single Agent Usage](#single-agent-usage)
  - [House Research Examples](#house-research-examples)
  - [House Bash Examples](#house-bash-examples)
  - [House Git Examples](#house-git-examples)
  - [House MCP Examples](#house-mcp-examples)
  - [House Coder Examples](#house-coder-examples)
  - [House Planner Examples](#house-planner-examples)
- [Multi-Agent Workflows](#multi-agent-workflows)
- [Common Patterns](#common-patterns)
- [Anti-Patterns](#anti-patterns)

---

## Single Agent Usage

### House Research Examples

#### Example 1: Finding Security Issues

**You:**
```
Use house-research to find all instances where we're using eval() or innerHTML
```

**House Research Agent Returns:**
```
## Search Results: Potentially Unsafe Code Patterns

### Summary
Found 3 instances of eval() and 7 instances of innerHTML across the codebase.
Most are in legacy code, 2 require immediate attention.

### Key Findings

**1. Direct eval() usage (HIGH RISK)**
- Location: `src/utils/parser.js:42`
- Details: `eval(userInput)` - evaluating unsanitized user input
- Relevance: Critical security vulnerability

**2. innerHTML with user data**
- Location: `src/components/UserProfile.js:89`
- Details: `element.innerHTML = userData.bio` - XSS risk
- Relevance: User-supplied content rendered without sanitization

[... condensed results ...]

### Recommended Actions
1. Replace eval() in parser.js with JSON.parse()
2. Use textContent or DOMPurify for innerHTML cases
3. Audit remaining instances in legacy code
```

**Your Next Message:**
```
Replace the eval() in parser.js with a safe alternative
```

---

#### Example 2: Finding API Endpoints

**You:**
```
Use house-research to find all Express route definitions in the backend
```

**House Research Agent Returns:**
```
## Search Results: Express Routes

### Summary
Found 23 route definitions across 5 files. 12 GET, 8 POST, 3 DELETE routes.

### Key Findings by File

**src/routes/auth.js**
- `POST /api/auth/login` (line 15)
- `POST /api/auth/register` (line 32)
- `POST /api/auth/logout` (line 48)

**src/routes/users.js**
- `GET /api/users` (line 10)
- `GET /api/users/:id` (line 25)
- `PUT /api/users/:id` (line 42)
- `DELETE /api/users/:id` (line 67)

[... more routes ...]

### Patterns Observed
- Auth routes lack rate limiting
- User routes missing input validation middleware
- No API versioning (consider /v1/ prefix)

### Recommended Actions
1. Add rate limiting to auth endpoints
2. Implement input validation middleware
3. Consider API versioning strategy
```

---

### House Bash Examples

#### Example 1: Running Tests

**You:**
```
Use house-bash to run the test suite and analyze any failures
```

**House Bash Agent Returns:**
```
## Execution Results: Test Suite

### Status
❌ Failed - 3 tests failing

### Summary
Ran 47 tests in 2.3s. 44 passed, 3 failed. All failures in auth module.

### Commands Executed
1. `npm test`

### Key Results
- Total tests: 47
- Passed: 44 (93.6%)
- Failed: 3 (6.4%)
- Duration: 2.3 seconds

### Errors Found

**Error 1: JWT Token Validation**
```
FAIL src/auth/jwt.test.js
  ● JWT verification › should reject expired tokens

  Expected expired token to be rejected, but was accepted

  at Object.<anonymous> (src/auth/jwt.test.js:45:7)
```
**Likely Cause**: JWT library not checking expiration or system clock issue
**Suggested Fix**:
1. Check if `jwt.verify()` has `ignoreExpiration: false` option
2. Verify system time is correct (`date` command)
3. Check if test tokens actually expired

**Error 2: Password Hashing**
```
FAIL src/auth/password.test.js
  ● Password hashing › should generate different hashes for same password

  Expected different hashes but got identical results

  at Object.<anonymous> (src/auth/password.test.js:23:7)
```
**Likely Cause**: Salt not being generated or reused across calls
**Suggested Fix**: Ensure bcrypt.hash() generates new salt each time

**Error 3: Login Rate Limiting**
```
FAIL src/auth/rateLimit.test.js
  ● Rate limiting › should block after 5 failed attempts

  Expected request to be blocked but got 200 response

  at Object.<anonymous> (src/auth/rateLimit.test.js:67:7)
```
**Likely Cause**: Rate limiter not properly configured or Redis not running
**Suggested Fix**: Check Redis connection and rate limiter middleware order

### Next Steps
1. Fix JWT expiration checking in auth/jwt.js
2. Review bcrypt salt generation in auth/password.js
3. Verify rate limiter configuration and Redis connection
4. Re-run tests after fixes
```

---

#### Example 2: Building and Analyzing Output

**You:**
```
Use house-bash to run the production build and check for issues
```

**House Bash Agent Returns:**
```
## Execution Results: Production Build

### Status
✅ Success with warnings

### Summary
Build completed in 12.3s. Bundle size increased 15% from last build. 3 warnings found.

### Commands Executed
1. `npm run build`

### Key Results
- Build time: 12.3 seconds
- Output directory: dist/
- Main bundle: 245.3 KB (gzipped: 87.2 KB)
- Vendor bundle: 156.8 KB (gzipped: 52.1 KB)
- Total size: 402.1 KB

### Warnings Found

**Warning 1: Large Bundle Size**
```
WARNING in asset size limit: The following asset(s) exceed the recommended size limit (244 KiB).
  main.js (245.3 KiB)
```
**Suggested Fix**: Consider code splitting or dynamic imports for large dependencies

**Warning 2: Deprecated API Usage**
```
WARNING in ./src/utils/api.js
Module Warning (from ./node_modules/eslint-loader/index.js):
  axios.get() 'transformResponse' option is deprecated
```
**Suggested Fix**: Update to axios v1.0+ API or remove deprecated options

**Warning 3: Source Map Size**
```
WARNING in source-map-loader
Failed to parse source map from 'node_modules/some-package/dist/file.js'
```
**Suggested Fix**: Non-critical - third-party package issue, safe to ignore

### Output Highlights
```
✓ Built in 12.3s
✓ 23 modules transformed
✓ Build artifacts written to dist/

Bundle Analysis:
├─ main.js (245.3 KB)
│  ├─ React (45 KB)
│  ├─ Chart.js (89 KB) ← largest contributor
│  └─ Application code (111.3 KB)
└─ vendor.js (156.8 KB)
```

### Next Steps
1. Consider lazy-loading Chart.js (used on only 2 pages)
2. Update axios to latest version to remove deprecated API warning
3. Monitor bundle size in CI (set threshold at 250 KB)
4. Deploy to staging for testing
```

---

### House Git Examples

#### Example 1: Reviewing a Large Diff

**You:**
```
Use house-git to review my staged changes
```

**House Git Agent Returns:**
```
## Git Analysis: Staged Changes Review

### Status
📊 Summary: 12 files changed, 347 insertions(+), 189 deletions(-)

### Key Changes by Impact

**🔴 CRITICAL (Review Carefully)**
- `src/auth/middleware.js:45-67` - Added new authentication check
- `config/api.js:12` - Changed API endpoint URL
- Relevance: Security and configuration changes

**🟡 MEDIUM (Notable Changes)**
- `api/users/controller.js:102-145` - Refactored user creation logic
- `api/users/validator.js:23-45` - Updated validation rules
- `tests/users.test.js:89-156` - Added new test cases
- Relevance: Core functionality with test coverage

**🟢 MINOR (Low Risk)**
- `styles/components/button.css:234-236` - Updated button hover color
- `docs/API.md:45-48` - Updated API documentation
- `package.json:23` - Bumped dependency version
- Relevance: Non-functional or low-impact changes

### Changes by File Type
- **Source Code**: 8 files (src/, api/)
- **Tests**: 2 files (tests/)
- **Configuration**: 1 file (config/)
- **Documentation**: 1 file (docs/)

### Recommendations
1. Carefully review authentication middleware changes
2. Verify API endpoint URL is correct for production
3. Ensure test coverage is adequate for user controller refactor
4. Consider running tests before committing: `npm test`
```

**Your Next Message:**
```
Run the test suite to verify these changes
```

---

#### Example 2: Branch Comparison Before Merge

**You:**
```
Use house-git to compare feature-user-export with main
```

**House Git Agent Returns:**
```
## Git Analysis: Branch Comparison (feature-user-export vs main)

### Status
📊 Summary: 7 files changed, 156 insertions(+), 23 deletions(-)
📍 Branches: feature-user-export is 8 commits ahead of main

### Key Changes by Impact

**🔴 CRITICAL (Review Carefully)**
- `api/users/export.js:1-89` - NEW FILE: User export endpoint
- `config/permissions.js:34` - Added EXPORT_USERS permission
- Relevance: New feature with permission implications

**🟡 MEDIUM (Notable Changes)**
- `api/users/routes.js:67` - Added new /export endpoint route
- `utils/csv-generator.js:1-45` - NEW FILE: CSV generation utility
- `tests/export.test.js:1-78` - NEW FILE: Export endpoint tests
- Relevance: New functionality with test coverage

**🟢 MINOR (Low Risk)**
- `docs/API.md:123-134` - Documented new export endpoint
- Relevance: Documentation update

### Merge Status
✅ No conflicts detected - safe to merge

### Commit Summary
Recent commits on feature-user-export:
- a1b2c3d "Add user export endpoint"
- e4f5g6h "Implement CSV generation"
- i7j8k9l "Add export tests"
- [... 5 more commits]

### Recommendations
1. Verify EXPORT_USERS permission is properly enforced
2. Test CSV export with large datasets (performance)
3. Ensure export respects user privacy/GDPR requirements
4. Run full test suite before merging: `npm test`
5. Consider adding rate limiting to export endpoint
```

---

#### Example 3: Commit History Analysis

**You:**
```
Use house-git to analyze the last 10 commits on main
```

**House Git Agent Returns:**
```
## Git Analysis: Last 10 Commits on main

### Summary
📅 Timespan: Last 10 commits span 5 days (Jan 10-14)
👥 Authors: 3 contributors (Alice: 6, Bob: 3, Carol: 1)
📊 Total Changes: 47 files, 892 insertions(+), 341 deletions(-)

### Commits by Category

**🔧 Features (4 commits)**
- `a1b2c3d` - Add user export functionality (Alice, Jan 14)
  - Modified: api/users/, utils/, tests/
- `e4f5g6h` - Implement search filters (Bob, Jan 12)
  - Modified: api/search/, components/
- `i7j8k9l` - Add pagination to user list (Alice, Jan 11)
  - Modified: api/users/, components/UserList.js

**🐛 Bug Fixes (3 commits)**
- `m8n9o0p` - Fix authentication timeout (Alice, Jan 13)
  - Modified: src/auth/middleware.js:45
- `q1r2s3t` - Resolve CSV export encoding issue (Bob, Jan 12)
  - Modified: utils/csv-generator.js:23

**📦 Dependencies (2 commits)**
- `u4v5w6x` - Update security dependencies (Carol, Jan 10)
  - Modified: package.json, package-lock.json

**📝 Documentation (1 commit)**
- `y7z8a9b` - Update API documentation (Alice, Jan 14)
  - Modified: docs/API.md

### Files Most Modified
1. `api/users/controller.js` - 4 commits, 234 changes
2. `utils/csv-generator.js` - 3 commits, 156 changes
3. `tests/users.test.js` - 3 commits, 189 changes

### Recommendations
1. Consider creating a release tag for the export feature
2. User-related code is seeing heavy activity - may need refactoring
3. Good test coverage being maintained
```

---

### House MCP Examples

#### Example 1: Configuring an MCP Server

**You:**
```
Use house-mcp to help me configure the Context7 MCP server for library documentation
```

**House MCP Agent Returns:**
```
## Context7 MCP Server Config

**Summary:** Access library documentation directly in Claude Code using Context7 MCP server.

**Configuration (.mcp.json):**
```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@context7/mcp"]
    }
  }
}
```

**Usage:**
```
Ask Claude: "Use context7 to get React hooks documentation"
Or: "Look up Next.js routing in context7"
```

**Critical Notes:**
- No API key required - uses public documentation
- Restart Claude Code after adding config
- Works with 1000+ popular libraries

**Need more?** Ask about: specific libraries, version selection, caching
```

---

#### Example 2: General API Configuration

**You:**
```
Use house-mcp to generate a SendGrid email integration config
```

**House MCP Agent Returns:**
```
## SendGrid Email Integration

**Summary:** Send transactional emails via SendGrid API with Node.js

**Code:**
```javascript
const sgMail = require('@sendgrid/mail');
sgMail.setApiKey(process.env.SENDGRID_API_KEY);

async function sendEmail(to, subject, html) {
  await sgMail.send({
    to,
    from: 'noreply@yourdomain.com', // Must be verified
    subject,
    html
  });
}
```

**Usage:**
```javascript
await sendEmail(
  'user@example.com',
  'Welcome!',
  '<h1>Thanks for signing up</h1>'
);
```

**Critical Notes:**
- Verify sender email in SendGrid dashboard first
- Never commit API keys (use environment variables)
- Free tier: 100 emails/day

**Need more?** Ask about: templates, attachments, tracking, webhooks
```

---

### House Coder Examples

#### Example 1: Fixing Import Errors

**You:**
```
Use house-coder to fix the import error in utils.py
```

**House Coder Agent Returns:**
```
## Code Changes: Fixed Import Error in utils.py

### Status
✅ Success

### Summary
Fixed ModuleNotFoundError by updating import path to include 'lib.' prefix.

### Files Modified
- `src/utils.py` - Updated import statement

### Key Changes

**1. Import Path Correction**
Location: `src/utils.py:3`
```python
# Before: from utils import process_data
from lib.utils import process_data
```
Rationale: Module is located in lib/ directory, not root

### Tests
- ✅ Syntax check: Passed
- ✅ Import check: Passed

### Token Savings
Processed ~2,500 tokens → Returned ~400 tokens (84% savings)
```

---

#### Example 2: Implementing TODO Comments

**You:**
```
Use house-coder to implement the TODO comment in authentication.py for email validation
```

**House Coder Agent Returns:**
```
## Code Changes: Implemented Email Validation

### Status
✅ Success

### Summary
Added regex-based email validation as requested in TODO comment. Raises ValueError for invalid formats.

### Files Modified
- `src/auth/authentication.py` - Added validation function
- `tests/test_authentication.py` - Added validation tests

### Key Changes

**1. Email Validation Function**
Location: `src/auth/authentication.py:45-49`
```python
def validate_email(email: str) -> bool:
    pattern = r'^[\w\.-]+@[\w\.-]+\.\w+$'
    if not re.match(pattern, email):
        raise ValueError(f"Invalid email format: {email}")
    return True
```
Rationale: Implements TODO for email format validation

**2. Test Coverage**
Location: `tests/test_authentication.py:67-71`
```python
def test_email_validation():
    assert validate_email("user@example.com")
    with pytest.raises(ValueError):
        validate_email("invalid-email")
```
Rationale: Added tests to ensure validation works correctly

### Tests
- ✅ Syntax check: Passed
- ✅ Quick tests: 2/2 passed

### Recommendations
- Consider using dedicated library like email-validator for production
- Add tests for edge cases (international domains, etc.)

### Token Savings
Processed ~8,200 tokens → Returned ~650 tokens (92% savings)
```

---

#### Example 3: Small Bug Fix

**You:**
```
Use house-coder to fix the off-by-one error in pagination.py
```

**House Coder Agent Returns:**
```
## Code Changes: Fixed Pagination Off-by-One Error

### Status
✅ Success

### Summary
Corrected range calculation in get_page_numbers() to prevent extra empty page.

### Files Modified
- `src/pagination.py` - Fixed range calculation

### Key Changes

**1. Range Calculation Fix**
Location: `src/pagination.py:23`
```python
# Before: range(0, total_pages + 1)
for page in range(0, total_pages):
```
Rationale: range(0, N) already gives N elements (0 to N-1), +1 created extra page

### Tests
- ✅ Syntax check: Passed
- ✅ Import check: Passed
- ✅ Quick tests: 5/5 passed

### Token Savings
Processed ~1,800 tokens → Returned ~320 tokens (82% savings)
```

---

### House Planner Examples

#### Example 1: Planning JWT Authentication

**You:**
```
Use house-planner to create a plan for adding JWT authentication to the API
```

**House Planner Agent Returns:**
```
## Execution Plan: JWT Authentication System

### Overview
Implement JWT-based authentication for API endpoints with token generation, validation, and middleware.

### Clarifications & Assumptions
- **Assumption**: Using PyJWT library (adding to requirements.txt)
- **Assumption**: Tokens valid for 24 hours
- **Assumption**: Keeping existing password hashing, adding JWT layer

### Task Breakdown

**Task 1: Research existing auth patterns**
- **Agent**: house-research
- **Scope**: Find current authentication and user model code
- **Complexity**: Low
- **Estimated Lines**: N/A (research only)
- **Context for Agent**: "Find all auth-related code: login/logout, password hashing, User model. Focus on auth/ and models/ directories."
- **Dependencies**: None

**Task 2: Install JWT library**
- **Agent**: house-bash
- **Scope**: Add PyJWT to requirements.txt and install
- **Complexity**: Low
- **Estimated Lines**: 1
- **Context for Agent**: "Add 'PyJWT==2.8.0' to requirements.txt, then run pip install -r requirements.txt"
- **Dependencies**: None

**Task 3: Create JWT utilities module**
- **Agent**: house-coder
- **Scope**: Create auth/jwt_utils.py with generate/validate functions
- **Complexity**: Medium (60 lines)
- **Estimated Lines**: 60
- **Context for Agent**: "Create auth/jwt_utils.py with generate_token(user_id) and validate_token(token) functions. Use PyJWT. Follow error handling pattern from auth/password.py."
- **Dependencies**: Task 2 (needs library installed)

**Task 4: Update login endpoint**
- **Agent**: house-coder
- **Scope**: Modify login to return JWT token
- **Complexity**: Low (30 lines)
- **Estimated Lines**: 30
- **Context for Agent**: "Update POST /api/auth/login to generate JWT token after password validation. Return token in response JSON."
- **Dependencies**: Task 3 (needs JWT utilities)

**Task 5: Create auth middleware**
- **Agent**: house-coder
- **Scope**: Add middleware to validate JWT on protected routes
- **Complexity**: Medium (50 lines)
- **Estimated Lines**: 50
- **Context for Agent**: "Create middleware/auth.py that validates JWT tokens. Extract user_id from token, attach to request. Return 401 for invalid tokens."
- **Dependencies**: Task 3 (needs JWT utilities)

**Task 6: Add tests**
- **Agent**: house-coder
- **Scope**: Test JWT generation, validation, and middleware
- **Complexity**: Medium (80 lines)
- **Estimated Lines**: 80
- **Context for Agent**: "Add tests for: token generation, token validation, expired tokens, invalid tokens, middleware. Follow test pattern in tests/test_auth.py."
- **Dependencies**: Tasks 3, 4, 5 (needs all components)

**Task 7: Run test suite**
- **Agent**: house-bash
- **Scope**: Execute pytest and verify all tests pass
- **Complexity**: Low
- **Estimated Lines**: N/A
- **Context for Agent**: "Run 'pytest tests/test_auth.py -v' and verify all JWT tests pass"
- **Dependencies**: Task 6 (needs tests)

### Execution Sequence
**Parallel Track 1:** Task 1 (research) → informs Tasks 3-5
**Parallel Track 2:** Task 2 (install) → Task 3 (JWT utils) → Task 4 (login), Task 5 (middleware) → Task 6 (tests) → Task 7 (run tests)

**Total Estimated Scope**: ~220 lines of code
**Estimated Complexity**: Medium
**Risks**: None identified

### Files to be Created/Modified
- `requirements.txt` (modified, +1 line)
- `auth/jwt_utils.py` (new, ~60 lines)
- `api/auth.py` (modified, ~30 lines)
- `middleware/auth.py` (new, ~50 lines)
- `tests/test_auth.py` (modified, ~80 lines)

### Recommendations
1. Execute research task first to understand current auth structure
2. Install library before implementing JWT utilities
3. Run tests after each major component
4. Consider adding rate limiting to login endpoint (future task)
```

---

#### Example 2: Planning Database Migration

**You:**
```
Use house-planner to plan migrating our database from SQLite to PostgreSQL
```

**House Planner Agent Returns:**
```
## Execution Plan: SQLite to PostgreSQL Migration

### Overview
Migrate application from SQLite to PostgreSQL with schema conversion, data migration, and connection updates.

### Clarifications & Assumptions
- **Assumption**: Using psycopg2 for PostgreSQL connection
- **Assumption**: Development environment migration first, production later
- **Assumption**: Keeping existing ORM/query patterns

### Task Breakdown

**Task 1: Audit current database schema**
- **Agent**: house-research
- **Scope**: Find all database models and SQL files
- **Complexity**: Low
- **Estimated Lines**: N/A
- **Context for Agent**: "Find all database models, schema files, and raw SQL queries. Look in models/, migrations/, and search for 'CREATE TABLE', 'ALTER TABLE'."
- **Dependencies**: None

**Task 2: Install PostgreSQL adapter**
- **Agent**: house-bash
- **Scope**: Install psycopg2-binary
- **Complexity**: Low
- **Estimated Lines**: N/A
- **Context for Agent**: "Add psycopg2-binary to requirements.txt and install"
- **Dependencies**: None

**Task 3: Create PostgreSQL schema**
- **Agent**: house-coder
- **Scope**: Convert SQLite schema to PostgreSQL
- **Complexity**: High (150 lines)
- **Estimated Lines**: 150
- **Context for Agent**: "Create migrations/postgresql_schema.sql converting SQLite schema to PostgreSQL. Update data types: INTEGER→INT, TEXT→VARCHAR, AUTOINCREMENT→SERIAL. Add proper indexes."
- **Dependencies**: Task 1 (needs schema audit)

[... more tasks ...]

### Recommendations
1. Test migration on development copy first
2. Create data backup before production migration
3. Plan for downtime window
4. Consider using migration tool like Alembic for future schema changes
```

---

## Multi-Agent Workflows

### Workflow 1: Refactoring Authentication System

**Step 1: Research Current Implementation**
```
Use house-research to find all authentication-related code
```

**Step 2: Implement Changes**
```
[You make changes based on research findings]
```

**Step 3: Run Tests**
```
Use house-bash to run the auth tests and check for regressions
```

---

### Workflow 2: Adding New API Integration

**Step 1: Research & Configuration**
```
Research SendGrid email API documentation and configure the integration
```

**Step 2: Implementation**
```
[You implement the email sending functionality with main Claude]
```

**Step 3: Testing**
```
Use house-bash to run integration tests for the email service
```

---

### Workflow 3: Codebase Audit

**Step 1: Find Issues**
```
Use house-research to find all console.log statements in production code
```

**Step 2: Remove Debug Code**
```
[You remove or replace console.logs]
```

**Step 3: Verify Changes**
```
Use house-bash to run linter and ensure no debug statements remain
```

---

## Common Patterns

### Pattern 1: Search → Fix → Test

Best for: Bug fixes, code cleanup

```
1. "Use house-research to find all instances of [problem]"
2. [Make fixes]
3. "Use house-bash to run tests and verify fixes"
```

### Pattern 2: Research → Implement → Deploy

Best for: New features, integrations

```
1. "Use house-research to find examples of [integration/pattern]"
2. [Implement the feature with main Claude]
3. "Use house-bash to build and deploy"
```

### Pattern 3: Audit → Research → Implement

Best for: Large refactors, security reviews

```
1. "Use house-research to analyze the [system/module]"
2. [Review findings and plan changes]
3. "Use house-research to find similar patterns in other files"
4. [Implement changes across codebase]
5. "Use house-bash to run full test suite"
```

### Pattern 4: Pre-Merge Review

Best for: Code review, merge safety checks

```
1. "Use house-git to compare feature-branch with main"
2. [Review critical changes identified]
3. "Use house-bash to run tests on feature-branch"
4. [Make merge decision]
```

---

## Anti-Patterns

### ❌ Don't: Use for Simple Tasks

**Bad:**
```
Use house-research to read the auth.js file
```

**Good:**
```
Read src/auth/auth.js
```

*Why:* Single file reads don't need a sub-agent

---

### ❌ Don't: Chain Too Many Agents

**Bad:**
```
Use house-research to find X, then use house-bash to test Y,
then use house-git to review Z, then use house-research to verify...
```

**Good:**
```
Use house-research to find X
[Review results, make decisions]
Use house-bash to test the changes
```

*Why:* Too many agent calls add latency. Let main Claude coordinate.

---

### ❌ Don't: Over-Specify Agent Tasks

**Bad:**
```
Use house-research to search for the word "async" in files matching
*.js pattern in the src directory, excluding node_modules, and return
results grouped by file
```

**Good:**
```
Use house-research to find all async functions in src/
```

*Why:* Let the agent determine search strategy. They're experts.

---

### ❌ Don't: Use When You Need Full Context

**Bad:**
```
Use house-research to help me understand how the auth system works
```

**Good:**
```
Explain how the authentication system in src/auth/ works
```

*Why:* Main Claude is better for explanations and learning. House agents are for heavy operations.

---

## Tips for Best Results

1. **Be Specific About Goals**
   - ✅ "Find all React components with memory leaks"
   - ❌ "Find React stuff"

2. **Let Agents Finish Before Next Step**
   - Review agent results before making decisions
   - Don't chain 5 agents in one message

3. **Use the Right Agent**
   - Searching files? → house-research
   - Analyzing git changes? → house-git
   - Running commands? → house-bash

4. **Trust Agent Expertise**
   - They know which tools to use
   - They know how to condense results
   - Review their work, but don't micromanage

5. **Iterate When Needed**
   - If agent returns too much: "Focus on files modified in last month"
   - If agent returns too little: "Also check the test files"

---

## Getting Help

Having issues with house agents?

1. Check agent files for tool permissions
2. Verify you're using the right agent for the task
3. Try being more specific in your request
4. Break complex tasks into smaller steps

Remember: House agents are specialized workers, not replacements for main Claude. Use them for heavy operations, let main Claude handle coordination and implementation.
