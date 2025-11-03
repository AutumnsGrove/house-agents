# House Planner + House Coder Workflow

## Scenario

You need to add email validation to your user registration system. This involves planning the changes, implementing them across multiple files, and ensuring everything works correctly.

## Step 1: Plan with House Planner

**Your prompt:**
```
Use house-planner to create a plan for adding email validation to the user registration system. Users should not be able to register with invalid email addresses.
```

**Expected output:**
```markdown
## Execution Plan: Email Validation for User Registration

### Overview
Add comprehensive email validation to user registration with regex validation, database constraints, and tests.

### Clarifications & Assumptions
- **Assumption**: Using regex pattern for email format validation
- **Assumption**: Adding database constraint to prevent invalid emails at DB level
- **Assumption**: Keeping existing registration flow, adding validation layer

### Task Breakdown

**Task 1: Create email validation utility**
- **Agent**: house-coder
- **Scope**: Create utils/validators.py with email validation function
- **Complexity**: Low (30 lines)
- **Estimated Lines**: 30
- **Context for Agent**: "Create utils/validators.py with validate_email(email) function using regex pattern r'^[\w\.-]+@[\w\.-]+\.\w+$'. Raise ValueError for invalid formats. Add type hints and docstring."
- **Dependencies**: None

**Task 2: Update user registration endpoint**
- **Agent**: house-coder
- **Scope**: Add email validation to POST /api/users/register
- **Complexity**: Low (15 lines)
- **Estimated Lines**: 15
- **Context for Agent**: "Import validate_email from utils/validators. Call it before creating user in routes/users.py register function. Return 400 error with message for invalid emails."
- **Dependencies**: Task 1 (needs validation function)

**Task 3: Add database constraint**
- **Agent**: house-coder
- **Scope**: Add CHECK constraint to users table
- **Complexity**: Low (20 lines)
- **Estimated Lines**: 20
- **Context for Agent**: "Create migration file migrations/003_add_email_constraint.sql adding CHECK constraint on users.email column. Use same regex pattern as validator."
- **Dependencies**: Task 1 (should match validation pattern)

**Task 4: Add validation tests**
- **Agent**: house-coder
- **Scope**: Test email validation function and endpoint
- **Complexity**: Medium (60 lines)
- **Estimated Lines**: 60
- **Context for Agent**: "Add tests to tests/test_validators.py for valid/invalid emails. Add tests to tests/test_users.py for registration endpoint with invalid emails. Follow existing test patterns."
- **Dependencies**: Tasks 1, 2 (needs implementation)

**Task 5: Run test suite**
- **Agent**: house-bash
- **Scope**: Execute pytest and verify all tests pass
- **Complexity**: Low
- **Estimated Lines**: N/A
- **Context for Agent**: "Run 'pytest tests/test_validators.py tests/test_users.py -v' and verify all new tests pass"
- **Dependencies**: Task 4 (needs tests)

### Execution Sequence
Task 1 (validator) → Task 2 (endpoint), Task 3 (constraint) → Task 4 (tests) → Task 5 (run tests)

**Total Estimated Scope**: ~125 lines of code
**Estimated Complexity**: Low-Medium
**Risks**: None identified

### Files to be Created/Modified
- `utils/validators.py` (new, ~30 lines)
- `routes/users.py` (modified, ~15 lines)
- `migrations/003_add_email_constraint.sql` (new, ~20 lines)
- `tests/test_validators.py` (new, ~35 lines)
- `tests/test_users.py` (modified, ~25 lines)

### Recommendations
1. Execute tasks in sequence to ensure each builds on previous
2. Verify validator works before updating endpoint
3. Ensure regex pattern matches between validator and DB constraint
```

## Step 2: Execute Task 1 with House Coder

**Your prompt:**
```
Use house-coder to create utils/validators.py with validate_email(email) function using regex pattern r'^[\w\.-]+@[\w\.-]+\.\w+$'. Raise ValueError for invalid formats. Add type hints and docstring.
```

**Expected output:**
```markdown
## Code Changes: Created Email Validator

### Status
✅ Success

### Summary
Created new validators module with email validation function using regex pattern matching.

### Files Modified
- `utils/validators.py` - Created new file with email validation

### Key Changes

**1. Email Validation Function**
Location: `utils/validators.py:1-12`
```python
import re
from typing import str

def validate_email(email: str) -> bool:
    """
    Validate email format using regex pattern.

    Args:
        email: Email address to validate

    Returns:
        True if valid

    Raises:
        ValueError: If email format is invalid
    """
    pattern = r'^[\w\.-]+@[\w\.-]+\.\w+$'
    if not re.match(pattern, email):
        raise ValueError(f"Invalid email format: {email}")
    return True
```
Rationale: Provides reusable email validation for use across the application

### Tests
- ✅ Syntax check: Passed
- ✅ Import check: Passed

### Recommendations
- Add tests before using in production
- Consider using dedicated library like email-validator for production use
- Pattern doesn't support all RFC 5322 valid emails (international domains, etc.)

### Token Savings
Processed ~2,100 tokens → Returned ~450 tokens (79% savings)
```

## Step 3: Execute Task 2 with House Coder

**Your prompt:**
```
Use house-coder to import validate_email from utils/validators and call it before creating user in routes/users.py register function. Return 400 error with message for invalid emails.
```

**Expected output:**
```markdown
## Code Changes: Added Email Validation to Registration

### Status
✅ Success

### Summary
Added email validation to user registration endpoint with proper error handling.

### Files Modified
- `routes/users.py` - Added validation and error handling

### Key Changes

**1. Import Statement**
Location: `routes/users.py:5`
```python
from utils.validators import validate_email
```
Rationale: Import validation function

**2. Validation Call**
Location: `routes/users.py:45-49`
```python
@app.route('/api/users/register', methods=['POST'])
def register():
    email = request.json.get('email')
    try:
        validate_email(email)
    except ValueError as e:
        return jsonify({"error": str(e)}), 400
    # ... rest of registration logic
```
Rationale: Validates email before database operations, returns clear error message

### Tests
- ✅ Syntax check: Passed
- ✅ Import check: Passed

### Recommendations
- Add tests for this endpoint with invalid emails
- Consider logging failed validation attempts for security monitoring

### Token Savings
Processed ~3,800 tokens → Returned ~520 tokens (86% savings)
```

## Step 4: Execute Remaining Tasks

Continue using house-coder for Tasks 3 and 4:

**Task 3:**
```
Use house-coder to create migration file migrations/003_add_email_constraint.sql adding CHECK constraint on users.email column using the same regex pattern
```

**Task 4:**
```
Use house-coder to add tests to tests/test_validators.py for valid/invalid emails and tests/test_users.py for registration with invalid emails
```

## Step 5: Run Tests

**Your prompt:**
```
Use house-bash to run pytest tests/test_validators.py tests/test_users.py -v and verify all new tests pass
```

**Expected output:**
```markdown
## Command Execution: Test Suite

### Status
✅ Success - All tests passed

### Summary
Executed pytest on validator and user tests. All 8 new tests passed.

### Test Results

**tests/test_validators.py:**
- ✅ test_valid_email_simple (PASSED)
- ✅ test_valid_email_with_dots (PASSED)
- ✅ test_invalid_email_no_at (FAILED → PASSED after fix)
- ✅ test_invalid_email_no_domain (PASSED)

**tests/test_users.py:**
- ✅ test_register_with_valid_email (PASSED)
- ✅ test_register_with_invalid_email_returns_400 (PASSED)
- ✅ test_register_with_missing_email (PASSED)
- ✅ test_register_error_message_format (PASSED)

**Metrics:**
- Total: 8 tests
- Passed: 8
- Failed: 0
- Duration: 1.23s

### Recommendations
None - all tests passing, validation working as expected
```

## Summary

This workflow demonstrates the house-planner → house-coder collaboration:

1. **House Planner** analyzed the requirement, created a detailed execution plan with task breakdown and dependencies
2. **House Coder** executed each implementation task efficiently, returning condensed summaries
3. **House Bash** verified everything worked by running tests

### Benefits

- **Clear planning**: house-planner provides roadmap before any code is written
- **Efficient execution**: house-coder handles implementation without polluting main context
- **Token savings**: ~85-90% reduction vs implementing in main conversation
- **Systematic approach**: Each task builds on previous, following dependency chain

### When to Use This Pattern

Use house-planner → house-coder workflow when:
- Adding features that span multiple files (3+)
- Requirements need clarification
- You want a clear plan before implementation
- Changes have dependencies that need coordination
- You're unsure about implementation approach

Don't use this pattern for:
- Single-file changes
- Simple bug fixes
- Well-defined, straightforward tasks
- Emergency hotfixes (too much planning overhead)
