# MCP Security Incident Response System

AI-powered security operations that enable natural language investigation and automated containment of AWS threats — turning GuardDuty alerts into actionable incident response.

## Overview

Security teams face a critical gap between threat detection and response. GuardDuty generates findings, but investigating and containing threats still requires manual AWS console navigation, CLI commands, and context-switching between tools. This system bridges that gap by bringing AWS security operations directly into an AI assistant.

The solution combines two components: an MCP server that exposes GuardDuty investigation capabilities to Claude Desktop, and an EventBridge-triggered Lambda that automatically isolates compromised EC2 instances. When GuardDuty detects a threat, the Lambda immediately contains it by swapping security groups and tagging the instance with forensic metadata, while simultaneously alerting the security team via Slack. The analyst can then use natural language to investigate the finding, generate incident reports, check isolation status, and restore access once remediation is complete — all without leaving their chat interface.

This demonstrates production-ready incident response automation: defence-in-depth with automated containment, human-in-the-loop investigation through conversational AI, and full audit trails through tagging and logging.

## Architecture

The system operates through two parallel workflows. The **automated containment path** starts when GuardDuty detects a medium-to-high severity finding on an EC2 instance. EventBridge captures the finding and invokes the Lambda function, which creates an isolation security group with no ingress/egress rules, applies it to the compromised instance, stores the original security groups in instance tags for later restoration, and sends a formatted Slack notification with finding details.

The **investigation path** runs through the MCP server, which exposes five tools to Claude Desktop: listing all findings with pagination support, deep-diving into specific findings to extract attacker IP, timeline, and resource details, generating markdown incident reports, checking if instances are currently isolated, and reversing isolation by restoring original security groups. The MCP server communicates directly with AWS APIs using local credentials and includes graceful fallback to mock data for demonstration purposes.

## Tech Stack

**Infrastructure**: AWS GuardDuty, EventBridge, Lambda, EC2, Terraform

**Automation**: Model Context Protocol SDK, Node.js, Slack Webhooks

**Security**: IAM roles with least-privilege policies, security group isolation pattern

## Key Decisions

- **Security group swap for isolation**: Rather than stopping instances (losing volatile memory evidence) or modifying NACL rules (affecting other resources), the system replaces security groups with a no-rule isolation group. This preserves forensic state while ensuring complete network containment, with original groups stored in tags for easy restoration.

- **MCP over custom API**: Building on the Model Context Protocol rather than a custom REST API means the security tools integrate natively with Claude Desktop. Analysts use natural language instead of learning new interfaces, reducing mean-time-to-respond while maintaining the AI's ability to chain multiple operations intelligently.

- **EventBridge pattern matching**: The event rule filters for both medium (≥4) and high severity findings specifically targeting EC2 instances. This catches brute force attacks and cryptomining activity while avoiding noise from low-severity findings, balancing automation coverage with alert fatigue.

## Screenshots

![Slack security alert](screenshots/slack-message.png)

![Claude investigation](screenshots/claude-investigate.png)

![Instance restoration](screenshots/claude-restore.png)

![GuardDuty investigation](screenshots/claude-guardduty.png)

![Claude response](screenshots/claude-response.png)

## Author

**Noah Frost**

- Website: [noahfrost.co.uk](https://noahfrost.co.uk)
- GitHub: [github.com/nfroze](https://github.com/nfroze)
- LinkedIn: [linkedin.com/in/nfroze](https://linkedin.com/in/nfroze)