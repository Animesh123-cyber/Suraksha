Here is how the files in the codebase work together across the frontend, parsing engine, and backend:
1. Root & Server Entry Points
server.ts: Full-stack Node.js Express server.
Hosts REST APIs: /api/scan (IaC & configuration posture engine), /api/webhook/dispatch (transmits Slack/Discord alert payloads server-side to avoid browser CORS restrictions), and /api/report/html (generates standalone HTML reports).
Mounts Vite middlewares in development and serves the compiled SPA in production on port 3000.
index.html: HTML shell configuring typography (Plus Jakarta Sans & JetBrains Mono), responsive viewports, and dark theme defaults.
metadata.json: Application metadata, capabilities, and permissions.
package.json: Dependency definitions and npm scripts (dev, build, lint, start).
vite.config.ts: Vite bundler configuration with Tailwind CSS v4 and path aliases.
2. Core Security & Compliance Engine (src/lib/)
src/lib/iacParser.ts:
Custom parser for Infrastructure-as-Code (Terraform HCL and configuration blocks).
Extracts resource types (e.g., aws_s3_bucket, aws_security_group), block names, key-value properties, and line numbers for precise code gutter tracking.
src/lib/cspmRules.ts:
Rule catalog containing security checks covering S3, IAM, Security Groups, RDS, EC2 (IMDSv2), EBS, CloudTrail, and hardcoded secrets.
Each rule provides a check function, CVSS 3.1 score, severity rating (Critical/High/Medium/Low), compliance mappings (CIS AWS, SOC 2, HIPAA, PCI-DSS), remediation advice, and Python boto3 + AWS CLI commands.
src/lib/complianceEngine.ts:
Executes the rules against parsed resource blocks.
Computes the aggregate Security Posture Score (0–100) and calculates control pass/fail percentages for CIS AWS Benchmark v3.0, SOC 2 Type II, HIPAA, and PCI-DSS.
src/lib/webhookDispatcher.ts:
Formats security violation alerts into rich Slack Block Kit payloads, Discord Embeds, or generic SIEM JSON schemas.
Handles automated delivery and dry-run simulations.
src/lib/reportGenerator.ts:
Generates self-contained, printable HTML audit reports and machine-readable JSON (SARIF/CSPM) exports.
src/lib/sampleTemplates.ts:
Pre-packaged IaC scenarios (Vulnerable Startup Stack, CIS Hardened Architecture, EKS Kubernetes Stack, and Blank Workspace).
3. Frontend UI Components (src/components/)
src/components/Header.tsx: Top navigation header adhering to the 3-zone layout contract (Wordmark, single-line navigation tabs, and primary "Export Report" action).
src/components/DashboardView.tsx: Executive overview displaying the 0–100 Posture Score gauge, 4-KPI metric strip (Critical, High, Medium, Passed), compliance framework mini-meters, and top vulnerability action cards.
src/components/DataVerifierView.tsx: Interactive data entry sandbox where users can fill in specific resource parameters (S3, SG, IAM, RDS, EC2, CloudTrail) or paste raw configs, click "Verify Security Posture", and inspect instant verdicts and generated Terraform/Boto3 scripts.
src/components/IacScannerView.tsx: Split-screen code editor with line gutter highlighting, error line badges, filterable finding details, and 1-Click Auto-Remediation that live-patches code in the editor.
src/components/ComplianceView.tsx: Deep-dive compliance matrix displaying control-level audits for CIS AWS Benchmark, SOC 2, HIPAA, and PCI-DSS with evidence links.
src/components/WebhookView.tsx: Webhook configuration console featuring interactive visual feed simulations (Slack & Discord), threshold selectors, payload inspectors, and transmission activity logs.
src/components/RemediationView.tsx: Blue Team console that generates an executable Python script with the boto3 SDK along with AWS CLI terminal commands.
src/components/ExportModal.tsx: Modal for downloading standalone HTML audit reports, raw JSON SARIF files, or copying Markdown summaries.
4. Application State & Styling
src/App.tsx: Root React component coordinating navigation state (activeTab), active IaC source code, scan results, auto-patching mutations, and webhook dispatches.
src/types/cspm.ts: TypeScript interfaces defining Finding, ScanResult, ComplianceFrameworkSummary, WebhookConfig, and ResourceBlock.
src/index.css: Tailwind CSS theme layers and sleek scrollbars for code inspection.
