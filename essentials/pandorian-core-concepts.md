# Core Concepts

## What is a Guideline?

A guideline is a software development rule or best practice written in plain English that Pandorian's AI automatically enforces across your codebase. Instead of relying on code reviews to catch issues, or configuring complex static analysis tools with dozens of rules, you simply describe what you want in natural language.

**Example Guidelines:**

*"When initializing lists and dictionaries with known quantities, consider pre-sizing to reduce allocations and improve performance."*

*"All configuration values including URLs, timeouts, limits, and paths must be externalized to configuration files or environment-specific sources rather than embedded directly in source code."*

Guidelines capture your team's standards, architectural decisions, security requirements, and performance best practices in a centralized, enforceable format. They work across any programming language and automatically adapt to your codebase's specific context.

### Anatomy of a Guideline

Every guideline consists of:

**Title:** A clear, actionable name that describes the rule (e.g., "Pre-size Collections When Possible" or "Hardcoded Configuration Values Must Be Eliminated")

**Description:** Plain English explanation of what should or shouldn't happen in your code. This is where you define the rule in natural language, including context, reasoning, and any exceptions.

**Tags:** Metadata that helps organize and prioritize guidelines:
- **Category** (Performance, Security, Configuration, Code Quality, etc.)
- **Severity** (Low, Medium, High, Critical)
- **Language** (Python, Java, JavaScript, or language-agnostic)

The AI Code Enforcer analyzes these components to understand intent and context, then searches your codebase for patterns that violate the guideline.

### Why Plain English Works

Traditional static analysis tools require you to learn proprietary rule languages, configure complex patterns, or write custom checks. Pandorian inverts this model. You write guidelines the same way you'd explain them to a teammate: "Don't hardcode API keys," "Always validate user input before database queries," "Use connection pooling for database access."

The AI handles the translation from intent to detection, adapting to different coding styles, frameworks, and contexts within your codebase. This means guidelines remain readable, maintainable, and accessible to your entire engineering organization, not just tooling experts.

---

## How Enforcement Works

Pandorian enforces guidelines through AI-powered static analysis that runs automatically across your repositories. Here's how the system operates:

### 1. Guideline Processing

When you create or update a guideline, Pandorian's AI analyzes the description to understand:
- The underlying pattern or anti-pattern to detect
- Code structures and language constructs involved
- Context that makes a violation legitimate vs. problematic
- Edge cases and exceptions implied by the guideline

This processing happens once per guideline, creating an enforcement model that's applied during scans.

### 2. Repository Scanning

Scans can be triggered:
- **On-demand** through the Pandorian dashboard
- **On Pull Requests** via GitHub App integration
- **On a schedule** (daily, weekly, or custom intervals)
- **Via CI/CD pipeline** using Pandorian's API

During a scan, the AI Code Enforcer:
- Parses your codebase's structure and dependencies
- Analyzes code against all active guidelines
- Identifies specific locations where violations occur
- Generates contextual explanations for each finding
- Provides fix generation capabilities where applicable

Scans are incremental and optimized for large codebases. The system focuses computational resources on changed files in PR contexts while performing comprehensive analysis for scheduled scans.

### 3. Violation Detection

A violation is identified when code patterns conflict with a guideline's requirements. The AI doesn't just match syntax; it understands semantic meaning. For example, the "Hardcoded Configuration Values" guideline detects:

```python
# Violation detected
API_URL = "https://api.example.com/v1"
timeout = 30
max_retries = 3
```

But correctly ignores constants that aren't configuration:

```python
# Not a violation - mathematical constant
PI = 3.14159
MAX_BATCH_SIZE = 100  # Business logic constant, not environment config
```

Each violation includes:
- **Location:** File path, line numbers, and code snippet
- **Explanation:** Why this code violates the guideline
- **Impact:** Potential consequences (security risk, performance degradation, etc.)
- **Fix generation:** Ability to generate context-aware remediation instructions

### 4. Integration Points

Enforcement happens across your development workflow:

**Pull Request Checks:** Violations appear as PR comments with detailed context, allowing developers to address issues before merge.

**Dashboard Review:** Engineering managers review aggregated violations and prioritize remediation efforts.

**Feedback Mechanism (Coming Soon):** Mark violations as accepted, false positives, or provide feedback directly in the UI to help improve detection accuracy and document intentional exceptions.

**IDE Integration (Coming Soon):** The VSCode plugin will show violations directly in your editor with inline context.

**CI/CD Gates:** Configure pipelines to fail builds when high-severity violations are introduced, preventing issues from reaching production.

---

## Understanding Scans

A scan is Pandorian's analysis of a repository against your active guidelines. Scans provide visibility into where your codebase diverges from established standards.

### Scan Types

**Full Scan:** Analyzes all repositories in your organization against all active guidelines. Best for comprehensive organizational audits, understanding technical debt across your entire codebase, and getting a complete view of standards compliance.

**Repository Scan:** Analyzes a specific repository in its entirety against all active guidelines. Ideal for deep-diving into a particular codebase, onboarding new repositories, or conducting focused assessments of individual projects.

**PR Scan:** Analyzes only the changes in a pull request against all active guidelines. Optimized for speed, these scans provide fast feedback during development without re-analyzing unchanged code.

### What Gets Scanned

Pandorian scans source code files across all languages in your repository. The system automatically:
- Identifies file types and languages
- Applies language-specific and language-agnostic guidelines
- Respects `.gitignore` and Pandorian-specific exclusion rules
- Focuses on application code (typically excludes `node_modules`, `vendor`, etc.)

You configure scan scope through repository settings, including file patterns to include or exclude.

### Scan Results

Each scan produces:

**Violation Summary:** Total count by severity (Critical, High, Medium, Low), aggregated by guideline and by file.

**Detailed Findings:** Individual violations with context, showing exactly where and why code violates guidelines.

**Fix Instructions:** Context-aware remediation guidance that you can generate for each violation.

Results remain accessible in the Pandorian dashboard, with historical data showing how your codebase evolves over time.

### Scan Performance

Scan duration varies based on repository size, complexity, and the number of active guidelines. Factors that influence scan time include:
- Number of files and lines of code
- Programming languages and frameworks used
- Complexity of active guidelines
- System load and resource availability

Incremental PR scans are optimized for speed and typically provide feedback within a minute or two, ensuring fast development velocity. Full repository scans may take longer depending on codebase size, but are designed to run asynchronously without blocking your workflow.

---

## Violations & Fixes

### What Makes a Violation

A violation occurs when code doesn't comply with a guideline's requirements. Violations aren't always bugs; they're deviations from your team's defined standards. This could mean:
- A security anti-pattern that creates risk
- A performance pattern that degrades efficiency
- A configuration approach that complicates deployment
- An architectural decision that conflicts with team conventions

The severity tag (Low, Medium, High, Critical) helps teams prioritize remediation. Critical violations might block deployments, while Low violations inform longer-term refactoring efforts.

### Understanding Violation Context

Each violation includes rich context:

**Code Location:** Precise file path and line numbers, with the problematic code snippet highlighted.

**Guideline Reference:** Which guideline was violated, with full description and rationale.

**Why It Matters:** Explanation of the specific issue in this code, including potential consequences.

**Example:** For "Hardcoded Configuration Values Must Be Eliminated":
```
Violation in src/api/client.py, line 15:
  API_URL = "https://api.example.com/v1"

This hardcoded URL prevents environment-specific configuration. 
If the API endpoint changes between development, staging, and 
production, code changes are required instead of simple config updates.
This violates deployment best practices and increases change risk.
```

### Generate Fix Feature

For each violation, Pandorian can generate context-aware fix instructions tailored to your specific code. When you click "Generate Fix," the system creates a detailed markdown file containing:

**Problem Analysis:** A breakdown of why the current code violates the guideline, including the specific patterns detected and their implications.

**Recommended Approach:** Step-by-step guidance on how to remediate the violation, explaining the reasoning behind each step.

**Implementation Examples:** Code samples showing compliant patterns in your repository's language and style, adapted to your specific context.

**Testing Considerations:** Suggestions for how to verify the fix works correctly and doesn't introduce regressions.

This markdown file provides everything you need to understand and implement the fix yourself. You can also feed this markdown file directly to your AI code-writing tool (like GitHub Copilot, Cursor, or Claude), which will use the context-aware instructions to generate the actual code fix for you automatically.

The instructions are context-aware, meaning they consider:
- Your codebase's existing patterns and conventions
- The specific frameworks and libraries you're using
- Related code that might need coordinated changes
- Edge cases specific to your implementation

Whether you implement fixes manually or use an AI tool to generate them, the markdown provides the complete context needed. Some violations require architectural decisions or business logic understanding that only developers can provide.

### False Positives and False Negatives

No static analysis system achieves perfect accuracy. While Pandorian's AI-powered approach significantly improves detection compared to traditional tools, you may occasionally encounter both false positives and false negatives.

**False Positives:** Legitimate code flagged as a violation. This happens when the AI identifies a pattern that matches the guideline but doesn't understand specific context that makes the code acceptable.

When this occurs:
- **Refine the guideline:** Update the guideline description to clarify exceptions and provide more context about when the pattern is acceptable
- **Contact support:** Share the false positive with our team to help improve detection accuracy (coming soon: mark violations as accepted directly in the UI)
- **Temporary workaround:** Document accepted violations in your team's guidelines (coming soon: exclusion rules to skip specific patterns or files during scans)

**False Negatives:** Violations that should be detected but aren't. This can happen when code violates the spirit of a guideline using patterns the AI hasn't yet learned to recognize.

When you discover false negatives:
- **Refine the guideline:** Make the description more explicit about patterns to detect
- **Provide examples:** Add specific code examples in the guideline description
- **Report patterns:** Let Pandorian know about missed patterns to improve detection

Your feedback in both cases trains the system. Over time, the AI learns your codebase's specific patterns and context, improving accuracy for your organization's unique needs.

---

## Use Cases: Making the Unautomatable Automatic

Pandorian's magic isn't just automating what linters already do—it's making the previously impossible suddenly trivial. Write any standard in plain English, and 60 seconds later it's automatically enforced across your entire codebase. Here's what becomes possible:

### Enforce Tribal Knowledge

**The Challenge:** Your most experienced engineers carry critical knowledge in their heads. They catch subtle issues in code review that no tool can detect: missing error context, incorrect architectural patterns, business logic mistakes. This expertise doesn't scale, doesn't transfer to new hires, and disappears when senior engineers leave or get overwhelmed.

Traditional linters can't help because this knowledge is too nuanced, too context-specific, and changes too frequently to encode in complex rule configurations.

**What Becomes Possible:**

Turn expert knowledge into automated enforcement by simply writing what your senior engineers have been saying in code reviews:

*"All API error responses must include the request correlation ID from the incoming request headers for distributed tracing"*

No linter can enforce this. It requires understanding request flow, context propagation, and your specific error handling architecture. With Pandorian, you write it in plain English and it's instantly enforced.

*"When adding database queries in user-facing request handlers, consider if the query should use read replicas to reduce load on the primary database"*

This isn't about syntax—it's about architectural thinking and performance considerations. Traditional tools can't detect "should consider" patterns or understand your specific database architecture. Pandorian can.

**The Result:** Senior engineer expertise becomes organizational knowledge that scales infinitely. New developers get the same quality of feedback as if a principal engineer reviewed every line of their code.

---

### Your Architecture, Automated

**The Challenge:** Every organization has architectural decisions that are critical but impossible to enforce automatically. Your microservices should follow certain patterns. Your modules should be organized a specific way. Services should communicate through designated channels. These aren't generic best practices—they're YOUR specific design decisions.

You document them in Confluence. You mention them in onboarding. You catch violations in code review. But there's no way to automatically enforce "use OUR pattern for OUR architecture."

**What Becomes Possible:**

Enforce your specific architectural decisions as if you had a custom-built tool for your exact codebase:

*"New microservices must follow the standard project structure with /api for endpoints, /services for business logic, /models for data structures, and /tests with matching directory structure"*

Generic linters can't understand your specific project conventions or validate directory structures against business logic organization. Pandorian understands your architecture from the description alone.

*"Inter-service communication must use the ServiceClient library with circuit breaker configuration, not direct HTTP calls or the old RestTemplate approach"*

This requires understanding your specific internal libraries, deprecated patterns, and architectural standards. It's impossible to configure a traditional tool for this without writing custom plugins. With Pandorian, it's one plain English sentence.

**The Result:** Your architectural decisions become automatically enforced. No more "we should be doing this" followed by discovering 30 services that don't. No more teams reinventing patterns because they didn't know the standard.

---

### Cross-Team Consistency at Scale

**The Challenge:** You have multiple teams working in different languages, different repos, different timezones. Each team develops their own interpretation of standards. The Python team handles errors differently than the Java team. The frontend follows different API conventions than the backend. Consistency is impossible when teams operate independently.

You can't use language-specific linters to enforce cross-team patterns, and you can't manually review every commit across dozens of repositories.

**What Becomes Possible:**

Enforce organization-wide standards that work across any language, any framework, any codebase:

*"All datetime objects must use UTC internally; timezone conversion to local time should only happen at the presentation layer"*

This is a semantic architectural rule about data handling, not a syntax rule. Traditional tools can't understand "internal vs. presentation layer" or detect semantic misuse of timezone handling. Pandorian enforces this consistently across Python, Java, JavaScript, and any other language you use.

*"Database migrations must include both upgrade and rollback scripts, with the rollback script tested to ensure it properly reverses the upgrade"*

This requires understanding file relationships, naming conventions, and testing patterns—all specific to your organization. It's not a generic rule anyone can download. Pandorian makes your specific standards enforceable across every repository.

**The Result:** True consistency across your entire engineering organization. A developer moving between teams finds the same standards enforced everywhere. Your architecture stays coherent even as you scale to dozens of teams.

---

### Business Logic Patterns

**The Challenge:** Some of your most important standards aren't about code quality—they're about business correctness. Prices must be calculated a certain way. User permissions must flow through specific services. Analytics events must follow naming conventions so your data pipeline works. Feature flags must be registered before use.

These aren't technical patterns that linters understand. They're YOUR business logic, YOUR domain rules, YOUR operational requirements. There's never been a way to automatically enforce "do the business process correctly."

**What Becomes Possible:**

Enforce business logic patterns and domain-specific rules as if you had written custom validation for every business requirement:

*"Price and currency calculations must use the Money value object type, never raw floats, integers, or string representations"*

This is about domain modeling and business correctness, not code syntax. Traditional tools don't understand your domain objects or why using primitives for money is dangerous in YOUR business. Pandorian understands from the description.

*"Analytics events must follow the naming convention: object_action (e.g., button_clicked, form_submitted, checkout_completed) and be logged through the AnalyticsService, not directly to tracking libraries"*

This combines naming conventions, business process, and architectural requirements—all specific to your analytics pipeline. It's impossible to enforce with traditional tools. With Pandorian, you describe what you need and it's automatically enforced.

**The Result:** Business logic correctness becomes automated. Developers can't accidentally break your data pipeline, miscalculate prices, or bypass permission checks. Your domain knowledge becomes enforceable code standards.

---

## Getting Started

Ready to create your first guideline? Start with a pain point your team faces regularly: a common code review comment, a security issue that keeps appearing, or a performance pattern you're trying to promote.

Write the guideline in plain English, add appropriate tags, and run your first scan. You'll immediately see where your codebase diverges from the standard, with specific locations and detailed context.

From there, expand your guideline catalog to cover more of your team's standards, architectural decisions, and best practices. The AI Code Enforcer scales with your needs, from a handful of critical rules to comprehensive organizational standards.