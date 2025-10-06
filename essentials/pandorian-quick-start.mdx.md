# Getting Started

This guide will walk you through setting up Pandorian and experiencing the core value: writing an engineering guideline in plain English, running a scan, and finding violations instantly.

**Time to first scan:** ~10 minutes

## Prerequisites

- GitHub account with access to at least one repository
- Repository admin permissions (required to install the GitHub App)

## Step 1: Sign Up and Connect GitHub

1. Visit [app.pandorian.ai](https://app.pandorian.ai/)
2. Click **Sign Up with GitHub**
3. Authorize Pandorian to access your GitHub account
4. Complete the GitHub App installation:
   - Choose **All repositories** or **Select repositories**
   - Approve the installation

Once connected, you'll see your repositories in the Pandorian dashboard.

## Step 2: Create Your First Guideline

You have two options: create a custom guideline from scratch, or import one from our catalog of 2,079+ pre-built guidelines.

### Option A: Create a Custom Guideline

This is where Pandorian's magic happens—you write guidelines in plain English.

1. Navigate to the **Guidelines** page
2. Click **+ New Guideline** (top right)
3. Select **Create Guideline**
4. Fill out the guideline form:
   - **Guideline ID:** A unique identifier (e.g., `SEC-001`, `PERF-002`)
   - **Labels:** Select a category (Security, Performance, Code Quality, etc.)
   - **Language:** Choose the programming language this applies to
   - **Title:** Clear, actionable name for your rule
   - **Description:** Write your guideline in plain English. Be specific about what should or shouldn't happen in the code.
   - **Severity:** High, Medium, or Low
   - **Enforce On:** Which repositories this guideline applies to (default: ALL)

**Example guideline:**
```
Title: Production Flask Applications Must Use Production-Ready WSGI Servers

Description: Flask's built-in development server (flask run or app.run()) must not be used in production environments. It is single-threaded, not designed to handle concurrent requests efficiently, and lacks the performance optimizations needed for production workloads. Applications must use production WSGI servers like Gunicorn, uWSGI, or Waitress with proper worker configuration to handle multiple concurrent requests and scale effectively.
```

5. Click **Create Guideline**

### Option B: Import from Catalog

Pandorian's catalog contains thousands of engineering best practices across 6 languages (Python, JavaScript, TypeScript, Java, Go, Scala) and multiple categories.

1. Navigate to the **Guidelines** page
2. Click **+ New Guideline** (top right)
3. Select **Import from Catalog**
4. Browse or search the catalog (filter by language, category, or priority)
5. Click the **+** icon next to any guideline to add it to your organization

Popular starting guidelines include environment-specific configuration management, API design standards, and production-readiness checks.

## Step 3: Run Your First Scan

Now that you have at least one guideline, it's time to scan your codebase.

1. Navigate to the **Repositories** page
2. Find the repository you want to scan
3. Click the **▶ Play** button next to the repository name
4. Select **Full Scan** (scans entire repository) or **Repository Scan** (same as full scan)

**Scan duration:**
- Small repositories: Under 60 seconds
- Medium repositories: 60-90 seconds  
- Large repositories: A few minutes

Pandorian's AI analyzes your entire codebase against all active guidelines, understanding context and structure rather than just pattern matching.

## Step 4: Review Scan Results

Once your scan completes, you'll see it in the **Scans** page with a status of **Completed**.

1. Navigate to the **Scans** page
2. Double-click on your completed scan
3. You'll see the **Scan Results** view with:
   - **Non-Compliant Files:** Files that contain violations (shown in red)
   - Each violation lists:
     - The guideline that was violated
     - Severity level
     - Detailed explanation of why this code violates the guideline
     - Specific lines of code involved

### Understanding Violations

Each violation provides rich context:

**Violations List:** Bullet points explaining exactly what was detected and why it's a problem

**Suggested Remediation:** Specific, actionable steps to fix the violation

**View Code:** Click to see the exact code location with line numbers highlighted

This is the moment most teams say "wow"—seeing complex engineering standards automatically enforced with detailed, context-aware explanations.

## Step 5: Take Action on Violations

You have several options for addressing violations:

### View in Context

Click **View Code** to see the violation in your actual codebase with:
- Exact line numbers highlighted
- Full file context
- Guideline details in the side panel

### Generate Fix Instructions

Click **Generate Fix** to create a markdown document with:
- Full context about the violation
- Step-by-step remediation instructions
- Code suggestions

You can feed this markdown directly to AI coding tools like GitHub Copilot, Cursor, or Claude to automatically implement fixes.

### Manual Remediation

Use the suggested remediations to fix violations manually, or share them with your team during code review.

## Next Steps

Now that you've experienced Pandorian's core workflow, you can:

- **Create more guidelines** to capture your organization's specific standards
- **Import additional guidelines** from the catalog for comprehensive coverage
- **Enable PR Scans** to catch violations before they reach your main branch
- **Integrate with CI/CD** to enforce guidelines automatically in your pipeline
- **Review scan history** to track compliance improvements over time

## Common Questions

**How often should I scan?**  
Most teams run full scans weekly or monthly to audit their codebase health, and enable PR scans to catch new violations automatically.

**Can I edit catalog guidelines?**  
Yes, after importing a guideline from the catalog, you can customize the description, severity, and enforcement scope to match your organization's needs.

**What if I get false positives?**  
You can refine the guideline description to be more specific, or contact support for assistance. We're continuously improving detection accuracy.

**Do scans affect my repository?**  
No, Pandorian only reads your code—it never writes or modifies anything in your repository.

## Need Help?

- Check out [Core Concepts](/core-concepts) for deeper understanding of how guidelines work
- Visit [Integration Guides](/integrations) to set up PR scanning and CI/CD enforcement
- Contact support at support@pandorian.ai
