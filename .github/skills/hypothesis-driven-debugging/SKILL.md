---
name: hypothesis-driven-debugging
description: "Investigate build failures, test errors, or unexpected behavior through systematic minimal reproduction, 3-hypothesis testing, and verification. Always re-run builds and tests after changes."
---

# Hypothesis-Driven Debugging

A systematic, rigorous approach to debugging failures.

## When to Use

- Investigating test failures
- Debugging build errors or compilation failures
- Analyzing unexpected runtime behavior
- Troubleshooting performance regressions
- Examining warning/error message issues

## Core Principles

1. **Always start with a minimal reproduction**
2. **Form multiple competing hypotheses**
3. **Design verification for each hypothesis**
4. **Document findings rigorously**
5. **Re-run builds and tests after every change**

## Process

### Step 1: Create Minimal Reproduction

1. **Extract the failure**:
   ```bash
   # For test failures
   dotnet test --filter "FullyQualifiedName~YourTest"

   # For build failures — isolate the problematic file
   dotnet build YourProject.csproj
   ```

2. **Reduce to essentials**: Remove unrelated code, simplify, verify minimal case still fails.

3. **Document the repro**:
   ```markdown
   ## Minimal Reproduction
   File: test-case.cs
   Command: dotnet test --filter "TestName"
   Expected: <expected behavior>
   Actual: <actual behavior>
   ```

### Step 2: Form 3 Hypotheses

Always form **at least 3 competing hypotheses**:

```markdown
## Hypothesis 1: [Brief description]
**Theory**: The failure occurs because...
**How to verify**: Run/change X and observe Y
**Verification result**: [To be filled]

## Hypothesis 2: [Brief description]
**Theory**: The failure occurs because...
**How to verify**: Add logging at Z
**Verification result**: [To be filled]

## Hypothesis 3: [Brief description]
**Theory**: The failure occurs because...
**How to verify**: Check assumption A by running B
**Verification result**: [To be filled]
```

### Step 3: Verification Methods

#### Code Instrumentation
```csharp
// Temporary debugging
Console.WriteLine($"DEBUG: Value at checkpoint: {someValue}");
System.Diagnostics.Debug.WriteLine($"DEBUG: Entering {nameof(Method)} with {arg}");
```

#### Minimal Test Cases
```csharp
[Fact]
public void Hypothesis1_Verification()
{
    var result = FunctionUnderTest(input);
    Assert.Equal(expected, result);
}
```

#### Build with Different Configurations
```bash
dotnet build -c Debug
dotnet build -c Release
# Compare outputs
```

### Step 4: Document in HYPOTHESIS.md

```markdown
# Hypothesis Investigation

## Issue Summary
Brief description of failure.

## Minimal Reproduction
[Code/commands]

## Hypotheses

### Hypothesis 1: Race condition in async disposal
**Theory**: Task disposed before reaching terminal state.
**Verification result**: ✅ CONFIRMED — adding await before dispose fixed it.

### Hypothesis 2: Missing null check
**Verification result**: ❌ DENIED — null check exists, different code path.

### Hypothesis 3: Configuration mismatch
**Verification result**: ⚠️ PARTIAL — related but not root cause.

## Resolution
[Final fix and verification]
```

### Step 5: Always Re-run Tests

After implementing any fix:

1. Build: `dotnet build -c Release`
2. Run affected tests: `dotnet test --filter "ClassName~Affected"`
3. Verify minimal repro passes
4. Run related tests — confirm no regressions
5. Document results with pass/fail counts and timing

## Anti-Patterns

❌ Skip the minimal reproduction
❌ Form only one hypothesis
❌ Make changes without verification
❌ Claim "fixed" without build evidence
❌ Say "pre-existing" or "infra issue"

✅ Start with smallest repro
✅ Consider multiple explanations
✅ Verify each hypothesis
✅ Always re-run build and tests
✅ Document commands, timings, results
