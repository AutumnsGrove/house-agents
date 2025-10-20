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

#### Example 1: Implementing a TODO Comment

**You:**
```
Use house-coder to implement the TODO in auth.py for email validation
```

**House Coder Agent Returns:**
```
## Code Changes: Email Validation Implementation

### Status
✅ Success

### Summary
Implemented email validation in the register_user function. Added regex pattern matching and error handling for invalid email formats.

### Files Modified
- `auth/auth.py` - Added email validation logic
- `tests/test_auth.py` - Added validation test cases

### Key Changes

**1. Email Validation in register_user**
Location: `auth/auth.py:23-27`
```python
import re

def register_user(email, password):
    if not re.match(r'^[\w\.-]+@[\w\.-]+\.\w+$', email):
        raise ValueError("Invalid email format")
    # ... rest of registration logic
```
Rationale: Prevents registration with malformed email addresses

**2. Test Coverage**
Location: `tests/test_auth.py:45-52`
```python
def test_invalid_email_validation():
    with pytest.raises(ValueError, match="Invalid email format"):
        register_user("invalid-email", "password123")

def test_valid_email_passes():
    result = register_user("user@example.com", "password123")
    assert result is not None
```
Rationale: Ensures validation works correctly

### Tests
- ✅ Syntax check: Passed
- ✅ Import check: Passed
- ✅ Quick tests: 2/2 new tests passed

### Recommendations
- Consider using email-validator library for more robust validation
- Add integration test for full registration flow

### Token Savings
Processed ~8,200 tokens → Returned ~780 tokens (~90% savings)
```

---

#### Example 2: Fixing Import Errors

**You:**
```
Use house-coder to fix the import errors in utils.py
```

**House Coder Agent Returns:**
```
## Code Changes: Import Error Fixes

### Status
✅ Success

### Summary
Fixed 3 import errors in utils.py by updating module paths to match new project structure.

### Files Modified
- `utils/utils.py` - Corrected import statements

### Key Changes

**1. Updated Import Paths**
Location: `utils/utils.py:1-5`
```python
# Before:
from helpers import format_date
from validators import check_email

# After:
from lib.helpers import format_date
from lib.validators import check_email
```
Rationale: Modules were moved to lib/ directory in recent refactor

### Tests
- ✅ Syntax check: Passed
- ✅ Import check: All imports resolved
- ⚠️ Tests not run (no test file found for utils.py)

### Recommendations
- Consider adding tests for utils.py functions
- Verify format_date and check_email work as expected in new structure

### Token Savings
Processed ~2,100 tokens → Returned ~420 tokens (~80% savings)
```

---

#### Example 3: Adding Error Handling

**You:**
```
Use house-coder to add try-catch error handling to the login function
```

**House Coder Agent Returns:**
```
## Code Changes: Error Handling for Login Function

### Status
✅ Success

### Summary
Wrapped login logic in try-except blocks to handle authentication errors gracefully and added proper logging.

### Files Modified
- `api/auth.py` - Added error handling to login function
- `utils/logger.py` - Added auth error logging format

### Key Changes

**1. Login Error Handling**
Location: `api/auth.py:34-47`
```python
def login(username, password):
    try:
        user = authenticate(username, password)
        if not user:
            logger.warning(f"Failed login attempt for user: {username}")
            return None

        token = generate_session_token(user.id)
        return {"user": user, "token": token}

    except AuthenticationError as e:
        logger.error(f"Authentication error for {username}: {e}")
        return None
    except DatabaseError as e:
        logger.critical(f"Database error during login: {e}")
        raise
```
Rationale: Prevents unhandled exceptions and provides better error visibility

### Tests
- ✅ Syntax check: Passed
- ✅ Import check: Passed
- ✅ Existing tests: 5/5 passed

### Recommendations
- Add specific tests for each error case (AuthenticationError, DatabaseError)
- Consider rate limiting for failed login attempts

### Token Savings
Processed ~5,800 tokens → Returned ~690 tokens (~88% savings)
```

---

### House Planner Examples

#### Example 1: Planning New Feature Development

**You:**
```
Use house-planner to plan adding a user authentication system with JWT tokens
```

**House Planner Agent Returns:**
```
## Execution Plan: JWT Authentication System

### Overview
Implementing JWT-based authentication with token generation, validation, and refresh functionality. System will replace existing session-based auth.

### Clarifications & Assumptions
- **Assumption**: Using PyJWT library (already in requirements.txt)
- **Assumption**: Tokens expire after 24 hours, refresh tokens after 7 days
- **Assumption**: Keeping backward compatibility with existing user model
- **Clarification Needed**: Should we support multiple active sessions per user?

### Task Breakdown

**Task 1: Research existing authentication patterns**
- **Agent**: house-research
- **Scope**: Find all auth-related code in codebase
- **Complexity**: Low
- **Estimated Lines**: N/A (research only)
- **Context for Agent**: "Search for authentication, login, session management patterns. Focus on api/auth.py, models/user.py, and middleware/auth.py"
- **Dependencies**: None

**Task 2: Create JWT utilities module**
- **Agent**: house-coder
- **Scope**: Create auth/jwt_utils.py with token generation and validation
- **Complexity**: Medium (80 lines)
- **Estimated Lines**: 80
- **Context for Agent**: "Create jwt_utils.py with: generate_token(user_id, expiry_hours), validate_token(token), refresh_token(old_token). Use PyJWT, handle expiration, signature validation. Follow error handling pattern from auth/password.py"
- **Dependencies**: Task 1 (informed by research)

**Task 3: Add JWT fields to User model**
- **Agent**: house-coder
- **Scope**: Update models/user.py and create migration
- **Complexity**: Low (30 lines)
- **Estimated Lines**: 30
- **Context for Agent**: "Add 'refresh_token' and 'refresh_token_expiry' fields to User model. Create alembic migration. Follow pattern in models/user.py for field definitions"
- **Dependencies**: None (parallel with Task 2)

**Task 4: Update login endpoint**
- **Agent**: house-coder
- **Scope**: Modify api/auth.py login to return JWT tokens
- **Complexity**: Medium (60 lines)
- **Estimated Lines**: 60
- **Context for Agent**: "Update POST /api/auth/login to generate and return JWT access token + refresh token. Store refresh token in DB. Match response format: {access_token, refresh_token, expires_in}"
- **Dependencies**: Task 2, Task 3

**Task 5: Create token refresh endpoint**
- **Agent**: house-coder
- **Scope**: Add POST /api/auth/refresh endpoint
- **Complexity**: Low (40 lines)
- **Estimated Lines**: 40
- **Context for Agent**: "Create refresh endpoint that validates refresh token, generates new access token. Follow existing endpoint pattern in api/auth.py"
- **Dependencies**: Task 4

**Task 6: Update authentication middleware**
- **Agent**: house-coder
- **Scope**: Modify middleware/auth.py to validate JWT tokens
- **Complexity**: Medium (70 lines)
- **Estimated Lines**: 70
- **Context for Agent**: "Update auth middleware to check Authorization: Bearer header, validate JWT token, set user context. Handle expired tokens gracefully"
- **Dependencies**: Task 2

**Task 7: Add comprehensive tests**
- **Agent**: house-coder
- **Scope**: Create tests for all JWT functionality
- **Complexity**: High (150 lines)
- **Estimated Lines**: 150
- **Context for Agent**: "Add tests in tests/test_jwt.py covering: token generation, validation, expiration, refresh flow, invalid tokens, missing tokens. Follow test pattern in tests/test_auth.py"
- **Dependencies**: Task 6

**Task 8: Run test suite**
- **Agent**: house-bash
- **Scope**: Execute full test suite with coverage
- **Complexity**: Low
- **Context for Agent**: "Run 'uv run pytest tests/ -v --cov=auth --cov=api' and report results"
- **Dependencies**: Task 7

### Execution Sequence

**Phase 1: Research & Setup** (Parallel)
- Task 1 (research)
- Task 3 (DB migration)

**Phase 2: Core Implementation** (Sequential)
- Task 2 (JWT utils) → Task 4 (login) → Task 5 (refresh) → Task 6 (middleware)

**Phase 3: Testing & Verification**
- Task 7 (tests) → Task 8 (run tests)

**Total Estimated Scope**: ~420 lines of new code
**Estimated Complexity**: Medium-High
**Estimated Time**: 3-4 hours

### Files to be Created/Modified
- `auth/jwt_utils.py` (new, ~80 lines)
- `migrations/004_add_jwt_fields.py` (new, ~30 lines)
- `models/user.py` (modified, +10 lines)
- `api/auth.py` (modified, ~100 lines)
- `middleware/auth.py` (modified, ~70 lines)
- `tests/test_jwt.py` (new, ~150 lines)

### Risks & Considerations
- **Breaking Change**: Existing session-based clients will need updates
- **Security**: Ensure tokens stored securely, no exposure in logs
- **Performance**: Token validation on every request - consider caching

### Recommendations for Main Claude
1. Execute Phase 1 tasks in parallel
2. After Task 2, review JWT implementation for security best practices
3. After Task 8, if tests pass, plan deployment strategy
4. Consider adding rate limiting to login/refresh endpoints
5. Document token format and authentication flow for API consumers
```

---

#### Example 2: Planning Large Refactoring

**You:**
```
Use house-planner to plan refactoring the entire API layer from synchronous to async/await
```

**House Planner Agent Returns:**
```
## Execution Plan: Async/Await API Refactoring

### Overview
Convert synchronous Flask API to asynchronous FastAPI using async/await patterns. This is a large architectural change requiring careful coordination.

### Clarifications & Assumptions
- **Question for User**: Do you want to migrate to FastAPI or add async support to Flask with Quart?
  - Option A: FastAPI (cleaner async, better performance, breaking changes)
  - Option B: Quart (minimal code changes, maintains Flask compatibility)
- **Assumption**: Using async database driver (asyncpg for PostgreSQL)
- **Assumption**: Gradual migration - run both sync and async endpoints during transition

*[Planner would wait for user's framework choice before continuing with detailed plan]*

### Recommendations
This is a complex refactoring (500+ lines across 15+ files). Consider:
1. Breaking into smaller phases (auth endpoints → user endpoints → admin endpoints)
2. Running both sync and async versions in parallel during migration
3. Comprehensive integration testing at each phase
4. Main Claude should coordinate this as a multi-sprint effort
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
