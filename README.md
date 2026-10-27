# AD262 - Agentic CAP: Build MCP Servers and Enterprise AI Agents

[![REUSE status](https://api.reuse.software/badge/github.com/SAP-samples/teched2026-AD262)](https://api.reuse.software/info/github.com/SAP-samples/teched2026-AD262)

## Description

This repository contains the material for the SAP TechEd 2026 session **AD262 - Agentic CAP: Build MCP Servers and Enterprise AI Agents**.

Learn how to turn apps based on SAP Cloud Application Programming Model (CAP) into enterprise AI agents using the Model Context Protocol (MCP) and Agent2Agent protocol. With the integration of the SAP AI Core infrastructure, it's straightforward to embed AI capabilities in CAP applications.

## Overview

**Use Case:** AI-powered invoice matching system that automatically finds and matches incoming invoices with purchase orders using semantic similarity and business rules.

This hands-on session introduces attendees to building enterprise AI agents with CAP by:
- Understanding the Model Context Protocol (MCP) and how it enables AI agents to interact with CAP services
- Creating an MCP server that exposes CAP services to AI agents
- Building intelligent invoice-to-PO matching using semantic search
- Creating conversational AI agents with natural language interfaces
- Integrating SAP AI Core for enterprise-grade recommendations (optional)
- Implementing audit logging and change tracking for AI governance (optional)

**Duration:** 2 hours (core content) + optional advanced topics

**CAP plugins added in the session:**

| Plugin | What it does |
|---|---|
| [`@cap-js/mcp`](https://github.com/cap-js/mcp) | Expose your CAP service as an MCP server — consumable by any AI tool |
| [`@cap-js/agents`](https://github.com/cap-js/agents) | Turn your CAP service into a fully autonomous A2A AI agent |
| [`@cap-js/ai`](https://github.com/cap-js/ai) | Embed AI features (RPT-1 recommendations) directly in your CAP app |

**One S/4HANA Data Product is integrated using `cds import`:**

| API | Version | Used for |
|---|---|---|
| `Supplier` | Data Product | Supplier master data |

## Requirements

To complete the exercises in this repository, you need:

### Software Requirements
- **npm**
- **Node.js** (v22 or higher)
- **@sap/cds-dk** (CAP Development Kit) - latest version
- **Visual Studio Code**
- a AI coding tool like  **Claude Code** (required for AI-assisted development)

### Workshop Materials
- **sap-s4com-Supplier-v1.json** - S/4HANA Supplier data product (provided in root folder. Originally from [SAP Business Accelerator Hub](https://api.sap.com/dataproduct/sap-s4com-Supplier-v1/overview) )
  - Copy this file to your project root when you reach Exercise 2

### Knowledge Prerequisites
- Basic understanding of SAP Cloud Application Programming Model (CAP)
- Familiarity with JavaScript/Node.js
- Basic understanding of REST APIs and OData services
- Basic knowledge of AI agents and prompting (helpful but not required)

### Optional
- **SAP AI Core** access (for optional exercise on SAP-RPT-1 recommendations)
- **SAP BTP** account (for deployment scenarios)

## Exercises

This workshop is structured into progressive exercises:

### Core Exercises (115 minutes)

- **[Exercise 0 - Initialize and Generate App](exercises/ex0/)** (20 min)
  - Initialize CAP project
  - Generate data model for invoices, POs, and match decisions
  - Create sample data
  - Use of the CAP Skills library

- **[Exercise 1 - Build React UI](exercises/ex1/)** (20 min)
  - Generate React application
  - Build invoice and PO matching interface
  - Implement decision workflow

- **[Exercise 2 - Add Supplier Data](exercises/ex2/)** (15 min)
  - Import S/4HANA Supplier data product
  - Add supplier details to invoice view
  - Show payment terms and blocking status

- **[Exercise 3 - Add MCP Server](exercises/ex3/)** (15 min)
  - Expose matching service over MCP
  - Configure Claude Desktop to connect
  - Test conversational queries

- **[Exercise 4 - Add Semantic Search](exercises/ex4/)** (20 min)
  - Install AI and vector search packages
  - Implement semantic search with local embeddings
  - Test meaning-based matching

- **[Exercise 5 - Add Agent with Skills](exercises/ex5/)** (25 min)
  - Configure CAP service as an agent
  - Create AP domain policy skill
  - Test intelligent match proposals

### Optional Advanced Exercises (35 minutes)

- **[Exercise 6 - Add Audit Log and Change Tracking](exercises/ex6/)** (Optional, 15 min)
  - Configure audit logging for data privacy
  - Enable change tracking on match decisions
  - Test audit trail

- **[Exercise 7 - Add SAP-RPT-1 Recommendations](exercises/ex7/)** (Optional, 20 min)
  - Integrate SAP AI Core
  - Use SAP-RPT-1 model for supplier recommendations
  - Add cost savings analysis

## Learning Objectives

By the end of this session, you will be able to:

1. Understand how Model Context Protocol (MCP) enables AI agents to interact with enterprise applications
2. Build an MCP server that exposes CAP services to AI agents
3. Implement Agent2Agent protocol for multi-agent collaboration
4. Create practical AI-powered business processes using CAP and MCP
5. Integrate enterprise AI capabilities with SAP AI Core (optional)
6. Implement governance and compliance features for AI agents (optional)

## Project Structure

```
invoice-matching/           # Your CAP project (created in Exercise 0)
├── db/
│   ├── schema.cds         # Data model (Invoices, POs, Matches)
│   └── data/              # Test data
├── srv/
│   ├── invoice-matching-service.cds   # Service definitions
│   ├── invoice-matching-service.js    # Service implementation
│   ├── recommendation-service.cds     # AI recommendations (optional)
│   ├── mcp-config.cds                 # MCP server configuration
│   ├── agent-config.yaml              # AI agent configuration
│   ├── skills/                        # Custom agent skills
│   └── prompts/                       # Agent prompt templates
├── app/matching/              # React UI (generated in Exercise 4)
├── package.json
└── sap-s4com-Supplier-v1.json  # Supplier data product
```

## Key Technologies

- **SAP Cloud Application Programming Model (CAP)** - Enterprise application framework
- **Model Context Protocol (MCP)** - Standard protocol for AI-application integration
- **Agent2Agent Protocol** - Multi-agent communication standard
- **SAP AI Core** - Enterprise AI infrastructure (optional)
- **Node.js & Express** - Runtime and web framework
- **SQLite/HANA** - Database options


## Further Learning

- [CAP Documentation](https://cap.cloud.sap/)
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [SAP AI Core Documentation](https://help.sap.com/docs/sap-ai-core)


## Contributing
Please read the [CONTRIBUTING.md](./CONTRIBUTING.md) to understand the contribution guidelines.

## Code of Conduct
Please read the [SAP Open Source Code of Conduct](https://github.com/SAP-samples/.github/blob/main/CODE_OF_CONDUCT.md).

## How to obtain support

Support for the content in this repository is available during the actual time of the online session for which this content has been designed. Otherwise, you may request support via the [Issues](../../issues) tab.

## License
Copyright 2026 SAP SE or an SAP affiliate company and AD262 contributors. Please see our [LICENSE](LICENSE) for copyright and license information. Detailed information including third-party components and their licensing/copyright information is available [via the REUSE tool](https://api.reuse.software/info/github.com/SAP/AD262).
