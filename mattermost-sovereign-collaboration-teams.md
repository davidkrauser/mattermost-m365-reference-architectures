## Sovereign Collaboration in Microsoft Teams

# 1. Overview

Agencies and critical infrastructure organizations often face strict data sovereignty requirements that restrict the use of public cloud services for sensitive collaboration. Deploying Mattermost for sovereign collaboration within Microsoft Teams and Outlook enables secure, compliant, and resilient communication while maintaining workflow continuity inside familiar Microsoft interfaces.

Mattermost can be hosted on-premises or in sovereign clouds, such as Azure GovCloud or Azure Local, ensuring that messages, files, recordings, and transcriptions remain in compliance-approved systems with encryption and strict policy enforcement.

Unified authentication with Microsoft Entra ID extends your Microsoft enterprise IT investments while delivering the compliance, control, and resilience required for mission-critical operations or out-of-band scenarios.

This document outlines architectural guidance for enabling sovereign collaboration within your Microsoft ecosystem.

---

# 2. Problem Statement

Many organizations operate in hybrid environments where both Microsoft Teams and Mattermost are essential collaboration tools. Teams serves general business communication needs, while Mattermost provides:
- Sovereign communication for classified or sensitive workflows
- Self-hosted data control for regulatory compliance
- Secure collaboration for mission-critical operations

However, requiring users to switch between separate applications creates friction, reduces productivity, and complicates workflows. Organizations need a unified experience that provides seamless access to both platforms without compromising security or data sovereignty.

# 3. Solution Architecture

[Mattermost Mission Collaboration for M365](https://github.com/mattermost/mattermost-plugin-msteams-devsecops) embeds Mattermost directly within Microsoft Teams and Outlook clients, providing a single-pane-of-glass experience. Users access both Teams channels and Mattermost channels from within the Microsoft ecosystem, eliminating application switching while maintaining separate data sovereignty for each platform.

**Key Benefits:**
- Unified user experience within familiar M365 interface
- Maintained data sovereignty for sensitive Mattermost communications
- Enterprise-grade identity management via Microsoft Entra ID
- Secure, self-hosted Mattermost data with M365 accessibility

---

# 4. Core Mattermost Capabilities

This use case benefits from Mattermost's foundational features:

## 4.1 Message Collaboration
Core Mattermost functionality providing secure, real-time messaging with channels, direct messages, group conversations, file sharing, and rich formatting. Mattermost includes enterprise-grade search capabilities. For large-scale deployments with over 5 million posts, [Elasticsearch or AWS OpenSearch Service](https://docs.mattermost.com/administration-guide/scale/scaling-for-enterprise.html#enterprise-search) provides optimized search performance with dedicated indexing.

## 4.2 Workflow Automation
The [Playbooks plugin](https://github.com/mattermost/mattermost-plugin-playbooks) enables teams to create and run structured, repeatable workflows with:
- Customizable checklists for standardized procedures
- Process automation to streamline repetitive tasks
- Retrospectives for continuous improvement
- Real-time collaboration during workflow execution

## 4.3 Project Tracking
The [Boards plugin](https://github.com/mattermost/mattermost-plugin-boards) provides integrated project management capabilities as a self-hosted alternative to Trello, Notion, and Asana, featuring:
- Kanban boards and table views
- Multilingual support
- Direct integration within Mattermost
- Collaborative task management

## 4.4 AI Agents
The [Agents plugin](https://github.com/mattermost/mattermost-plugin-agents) integrates AI capabilities directly into Mattermost, offering:
- Multiple AI assistants with specialized capabilities
- Thread and channel summarization
- Action item extraction from discussions
- Meeting transcription and summarization
- Semantic search across workspace content
- Support for multiple LLM backends: Azure OpenAI, OpenAI, Anthropic, or self-hosted models (Ollama, vLLM)
- Requires PostgreSQL with pgvector extension

## 4.5 Audio Calls and Screenshare
The [Calls plugin](https://github.com/mattermost/mattermost-plugin-calls) enables real-time voice calling and screen sharing directly within channels:
- Voice calls between channel participants
- Screen sharing capabilities
- Integrated communication without external tools

---

# 5. Architecture

The deployment architecture includes the following components. For detailed technical specifications, see the [Mattermost Server Architecture documentation](https://docs.mattermost.com/deployment-guide/server/server-architecture.html).

## 5.1 Users

Enterprise users within the network boundary accessing client applications for M365 services and Mattermost.

## 5.2 Microsoft Entra ID (Identity Provider)

Users authenticate to M365 applications and Mattermost using single sign-on Entra ID via OpenID Connect (OIDC) or SAML.

## 5.3 Client Applications

- **Microsoft 365 Desktop Apps**: Teams and Outlook with embedded Mattermost application for seamless collaboration within a familiar interface while enforcing regulatory compliance.
- **Mattermost Desktop Apps**: Access Mattermost via desktop or web apps in addition to the embedded views from Teams and Outlook. (Optional)
- **Mattermost Mobile Apps**: Access Mattermost via iPhone and Android apps, with support for ID-only push notifications to ensure compliance with data sovereignty requirements. (Optional)

### 5.3.1 ID-only Push Notifications

To prevent sensitive message content from being transmitted to external notification services such as Apple Push Notification Service (APNS) and Firebase Cloud Messaging (FCM), configure Mattermost to use ID-only push notifications. In this mode, only a message identifier is sent to public push notification services, and the client retrieves the content securely from the Mattermost server over an encrypted channel.

## 5.4 Mattermost Server

Core application server deployed for sovereign collaboration on enterprise-controlled infrastructure or private cloud, such as Azure or Azure Local, to maintain compliance with STIG, FedRAMP, and NIST 800-53 standards. The server provides all capabilities described in the Core Mattermost Capabilities section above. For large-scale deployments, see [Scaling for Enterprise](https://docs.mattermost.com/administration-guide/scale/scaling-for-enterprise.html) for detailed performance tuning guidance.

### 5.4.1 High Availability and Fault Tolerance

Deploy Mattermost in a [cluster-based architecture](https://docs.mattermost.com/administration-guide/scale/high-availability-cluster-based-deployment.html) to ensure continued availability during outages or hardware failures. All cluster nodes must reside within the same datacenter to ensure optimal performance and consistent network latency. High availability requires redundant infrastructure across each critical component:

- **Application servers**: Scale horizontally across multiple nodes (64-bit x86 architecture) with a load balancer distributing client traffic. Servers communicate via gossip protocol for cluster coordination and leader election. All nodes must maintain identical `config.json` configurations and synchronized time via Network Time Protocol (NTP).
- **Database layer**: Use PostgreSQL master/read replica architecture with connection pooling and automatic failover. Configure read replicas to offload query traffic and search replicas for dedicated search operations. Optimize database connection limits, work memory settings, and autovacuum parameters for performance.
- **Object storage**: Configure S3-compatible backends with erasure coding or replication for durability. All application servers must access shared file storage (NAS or S3) to ensure consistent data availability.
- **Monitoring**: Deploy Prometheus and Grafana for comprehensive cluster health monitoring and performance metrics collection.

**Cluster Operations**: Rolling updates can be performed for dot releases without service interruption. Configuration changes can typically be applied without full service restarts (server restart takes approximately 5 seconds; database schema updates may require up to 30 seconds). Use environment variables for configuration management and mmctl for cluster administration.

### 5.4.2 Compliance and Retention

Sovereign environments often require strict enforcement of retention policies, legal hold, and export controls. Configure Mattermost's built-in compliance features to meet agency or sectoral mandates.

- Enable compliance export and monitoring to produce auditable exports of message data and user activity logs.
- Configure message retention and legal hold policies to align with applicable regulations.
- Integrate with your organization's eDiscovery and archiving systems as required.

## 5.5 Proxy Server

The proxy server handles HTTP(S) routing within the cluster, directing traffic between the server and clients accessing Mattermost services. NGINX is recommended for load balancing with support for WebSocket connections, health check endpoints, and sticky sessions. The proxy layer provides SSL termination and distributes client traffic across application servers.

## 5.6 PostgreSQL Database

Stores persistent application data on a PostgreSQL v13+ database, such as Azure Database for PostgreSQL. For high availability, configure a master/read replica architecture with connection pooling. Optimize PostgreSQL settings including connection limits, work memory allocation, and autovacuum parameters. Support for search replicas enables dedicated database resources for search operations.

## 5.7 Object Storage

File uploads, images, and attachments are stored outside the application node on an S3-compatible store, such as MinIO, or Amazon S3. For high availability deployments, object storage must be shared across all application servers using network-attached storage (NAS) or S3-compatible object storage with erasure coding or replication for durability.

## 5.8 Recording Instance

`calls-offloader` job service to offload heavy processing tasks from Mattermost Calls, such as recordings, transcriptions, and live captioning, to enterprise-controlled infrastructure or private cloud. (Optional)

## 5.9 Self-hosted Integrations

Custom apps, plugins, and webhooks can be deployed within the enterprise boundary. (Optional)

## 5.10 Secure Access Layer

A firewall or access gateway protecting entry into the enterprise network. This may include network policies, IP allowlists, or WAFs depending on your networking configurations. (Optional)

## 5.11 Microsoft Global Network

World-wide network of Microsoft data centers, delivering public cloud services including M365 and Azure OpenAI.

## 5.12 Azure OpenAI Service

LLM service used for summarization, AI-enhanced search, and agent-assisted workflows, hosted within the Microsoft Global Network. (Optional)

## 5.13 Calls Plugin

The [Calls plugin](https://github.com/mattermost/mattermost-plugin-calls) enables real-time voice calling and screen sharing directly within channels.

### 5.13.1 Sovereign Audio & Screensharing

Sovereign collaboration requires all voice and screen sharing traffic remain within enterprise-controlled infrastructure and does not traverse third-party services. Deploy Mattermost Calls in a self-hosted configuration to ensure that Microsoft Teams users and Mattermost users collaborate without media ever leaving the sovereign network.

- The `rtcd` service for scalable, low-latency media routing hosted in private cloud or on-premises. Run multiple `rtcd` nodes for redundancy across datacenters or regions.
- The `calls-offloader` service offloads heavy processing tasks like recording, transcription and live captioning to a compliance-approved job server.
<
