# Enterprise to Edge DDIL Operations

## Overview

Disconnected, Denied, Intermittent, and Limited (DDIL) network conditions present unique challenges for enterprise-to-edge operations. Tactical networks may become temporarily isolated from the internet, interrupting direct access to Microsoft 365 services such as Teams and Outlook.

Mattermost enables resilient collaboration by remaining fully operational in DDIL environments. Mission users continue to access self-hosted messaging collaboration, workflow automation, mission planning, AI agents, and audio calls with screen sharing within the tactical network. No internet connectivity is required for collaboration functions, including optional self-hosted LLMs and calling services.

When connectivity is restored, mission users regain access to M365 enterprise systems in addition to collaboration continuity with enterprise users through the embedded Mattermost experience inside their Microsoft Teams and Outlook applications. All mission activity during the period of disconnection becomes available across enterprise and tactical environments after connectivity returns.

This deployment architecture extends sovereign collaboration with Microsoft Teams and Outlook to the tactical edge, enabling fully operational tactical collaboration without reliance on internet connectivity in DDIL scenarios.

---

## Problem Statement

Field operations, remote deployments, and edge environments often experience Disconnected, Intermittent, and Limited (DDIL) network connectivity. Organizations need collaboration solutions that:
- Enable continued operations when internet connectivity is unavailable
- Allow users with internet access to collaborate via M365
- Allow users without internet access to collaborate via Mattermost
- Make all messages immediately available when connectivity is restored

Traditional cloud-only solutions fail in these scenarios, while fully disconnected systems don't integrate with enterprise workflows during normal operations.

## Solution Architecture

A dual-environment deployment strategy provides optimal collaboration in both connected and disconnected states:

**Connected State:**
- All users access unified Teams/Mattermost interface via [Mission Collaboration plugin](https://github.com/mattermost/mattermost-plugin-msteams-devsecops)
- Enterprise users collaborate with edge users seamlessly via Mattermost

**Disconnected State:**
- Users with internet access continue using M365 (Teams, Outlook, etc.)
- Users without internet access use Mattermost exclusively via native clients
- Mattermost operates independently at the edge location
- All Mattermost messages are stored locally and remain accessible

**Reconnection:**
- Internet connectivity is restored to the edge Mattermost server
- Users with internet access can again view Mattermost via Mission Collaboration plugin in Teams
- All messages sent during disconnection are immediately available

---

## Core Mattermost Capabilities

This use case benefits from Mattermost's foundational features:

### Message Collaboration
Core Mattermost functionality providing secure, real-time messaging with channels, direct messages, group conversations, file sharing, and rich formatting.

### Workflow Automation
The [Playbooks plugin](https://github.com/mattermost/mattermost-plugin-playbooks) enables teams to create and run structured, repeatable workflows with:
- Customizable checklists for standardized procedures
- Process automation to streamline repetitive tasks
- Retrospectives for continuous improvement
- Real-time collaboration during workflow execution

### Project Tracking
The [Boards plugin](https://github.com/mattermost/mattermost-plugin-boards) provides integrated project management capabilities as a self-hosted alternative to Trello, Notion, and Asana, featuring:
- Kanban boards and table views
- Multilingual support
- Direct integration within Mattermost
- Collaborative task management

### AI Agents
The [Agents plugin](https://github.com/mattermost/mattermost-plugin-agents) integrates AI capabilities directly into Mattermost, offering:
- Multiple AI assistants with specialized capabilities
- Thread and channel summarization
- Action item extraction from discussions
- Meeting transcription and summarization
- Semantic search across workspace content
- Support for multiple LLM backends: Azure OpenAI, OpenAI, Anthropic, or self-hosted models (Ollama, vLLM)
- Requires PostgreSQL with pgvector extension

### Audio Calls and Screenshare
The [Calls plugin](https://github.com/mattermost/mattermost-plugin-calls) enables real-time voice calling and screen sharing directly within channels:
- Voice calls between channel participants
- Screen sharing capabilities
- Integrated communication without external tools

---

## Architecture Components

The deployment architecture includes the following components. For detailed technical specifications, see the [Mattermost Server Architecture documentation](https://docs.mattermost.com/deployment-guide/server/server-architecture.html).

**Enterprise Network**: Network containing enterprise services and continuous access to Microsoft 365. The network may have a firewall or access gateway protecting egress and ingress, such as network policies, IP allowlists, or WAFs depending on your networking configurations.

**Enterprise Users**: Users within the enterprise network boundary accessing client applications for M365 services and Mattermost.

**Tactical Network**: Operates independently during disconnections, hosting a self-contained Mattermost deployment to ensure mission users maintain collaboration capabilities.

**Mission Users**: Tactical or edge users who may be intermittently disconnected from enterprise systems and Microsoft services.

**Identity Provider**: Authentication for mission users at the tactical edge requires a locally hosted identity provider to ensure authentication services remain operational during disconnected periods. Enterprise users authenticate to M365 applications using Microsoft Entra ID.

- **Local Identity Provider**: Deployed within the tactical network to provide authentication services during DDIL conditions. Recommended options include Keycloak (open-source IdP with OIDC/SAML support), Active Directory with ADFS, or OpenLDAP with an OIDC bridge. The local IdP serves as the primary authentication source for Mattermost and maintains an independent user directory that operates without internet connectivity.
- **Microsoft Entra ID**: Cloud-based identity provider for enterprise users accessing M365 services. When connectivity permits, the local IdP can federate with Entra ID to synchronize user accounts, credentials, and group memberships, ensuring consistency across enterprise and tactical environments.

**Client Applications**:
- **Microsoft 365 Desktop Apps**: Teams and Outlook with embedded Mattermost application.
  - **When internet connected**: Enables seamless enterprise-to-edge collaboration within a familiar interface.
  - **When internet disconnected**: Microsoft services are unreachable, but embedded Mattermost application remains fully operational for tactical collaboration.
- **Mattermost Desktop Apps**: Access Mattermost via desktop or web apps in addition to the embedded views from Teams and Outlook. (Optional)
- **Mattermost Mobile Apps**: Access Mattermost via iPhone and Android apps, with support for ID-only push notifications to ensure compliance with data sovereignty requirements. (Optional when connectivity permits)

**Mattermost Deployment**: Mattermost deployed for sovereign tactical collaboration on local infrastructure, such as Azure Local, supporting data residency regulations and disconnected operations.

**Mattermost Server**: Core application server handling tactical collaboration workloads, including:
- **Messaging Collaboration**: DDIL-ready 1:1, group messaging, and structured channel collaboration with rich integration capabilities and enterprise-grade search. For deployments with over 5 million posts, [Elasticsearch can be hosted locally](https://docs.mattermost.com/administration-guide/scale/scaling-for-enterprise.html#enterprise-search) for optimized search performance with dedicated indexing.
- **Workflow Automation** (Optional): Playbooks provide structure, monitoring and automation for repeatable processes built-in to your local Mattermost deployment.
- **Project Tracking** (Optional): Boards enables project management capabilities built-in to your local Mattermost deployment.
- **AI Agents** (Optional): AI Agents run against a local LLM hosted within your tactical network.
- **Audio & Screenshare** (Optional): Calls offers native real-time self-hosted audio calls and screen sharing within your tactical network.
- **Performance Optimization** (Optional): Redis for in-memory caching significantly improves performance for installations over 100,000 users by providing efficient caching mechanisms, all hosted within the tactical network. See [Scaling for Enterprise](https://docs.mattermost.com/administration-guide/scale/scaling-for-enterprise.html) for detailed performance tuning guidance.

**Proxy Server**: The proxy server handles HTTP(S) routing within the cluster, directing traffic between the server and clients accessing Mattermost services. NGINX is recommended for load balancing with support for WebSocket connections, health check endpoints, and sticky sessions. The proxy layer provides SSL termination and distributes client traffic across application servers.

**PostgreSQL Database**: Stores persistent application data on a PostgreSQL v13+ database hosted locally within your tactical network. For high availability, configure a master/read replica architecture with connection pooling. Optimize PostgreSQL settings including connection limits, work memory allocation, and autovacuum parameters. Support for search replicas enables dedicated database resources for search operations.

**Object Storage**: File uploads, images, and attachments are stored outside the application node on an S3-compatible store, such as MinIO, hosted locally within your tactical network. For high availability deployments, object storage must be shared across all application servers using network-attached storage (NAS) or S3-compatible object storage with erasure coding or replication for durability.

**Recording Instance**: `calls-offloader` job service to offload heavy processing tasks from Mattermost Calls to self-hosted infrastructure within your tactical network, such as recordings, transcriptions, and live captioning. (Optional)

**Self-hosted Integrations**: Custom apps, plugins, and webhooks can be deployed within your tactical network. (Optional)

**Self-hosted LLM**: Locally hosted OpenAI compatible LLM for agentic powered collaboration within your tactical network. (Optional)

**Microsoft Global Network**: World-wide network of Microsoft data centers, delivering public cloud services when internet connectivity permits.

---

## Key Components Detail

### Identity Provider

Authentication for mission users at the tactical edge requires a locally hosted identity provider to ensure authentication services remain operational during disconnected periods.

#### User Authentication in DDIL Environments

DDIL environments require authentication infrastructure that remains fully operational without internet connectivity. Relying solely on cloud-based identity providers such as Microsoft Entra ID creates a critical single point of failure when tactical networks become disconnected. To ensure mission users maintain authentication capabilities, deploy a locally hosted identity provider within the tactical network.

Deploy an identity solution such as Keycloak (open-source IdP with OIDC/SAML support), Active Directory with ADFS, or OpenLDAP with an OIDC bridge within the tactical network. The local IdP serves as the primary authentication source for Mattermost and maintains an independent user directory that operates without internet connectivity. When connectivity permits, the local IdP can federate with Entra ID to synchronize user accounts, credentials, and group memberships. User accounts must be provisioned in the local IdP before disconnection occurs to ensure authentication services remain available throughout DDIL conditions. See [Air-Gapped Deployment](https://docs.mattermost.com/deployment-guide/server/air-gapped-deployment.html) for detailed configuration guidance for disconnected environments.

### Mattermost Server Deployment

The Mattermost server provides tactical collaboration capabilities when deployed on local infrastructure, such as Azure Local, supporting data residency regulations and disconnected operations.

#### High Availability and Fault Tolerance

Deploy Mattermost in a [cluster-based architecture](https://docs.mattermost.com/administration-guide/scale/high-availability-cluster-based-deployment.html) to ensure continued availability during outages or hardware failures at the tactical edge. All cluster nodes must reside within the same tactical network location to ensure optimal performance and consistent network latency. High availability requires redundant infrastructure across each critical component:

- **Application servers**: Scale horizontally across multiple nodes (64-bit x86 architecture) with a load balancer distributing client traffic. Servers communicate via gossip protocol for cluster coordination and leader election. All nodes must maintain identical `config.json` configurations and synchronized time via Network Time Protocol (NTP).
- **Database layer**: Use PostgreSQL master/read replica architecture with connection pooling and automatic failover. Configure read replicas to offload query traffic and search replicas for dedicated search operations. Optimize database connection limits, work memory settings, and autovacuum parameters for performance.
- **Object storage**: Configure S3-compatible backends with erasure coding or replication for durability. All application servers must access shared file storage (NAS or S3-compatible local storage) to ensure consistent data availability.
- **Monitoring**: Deploy Prometheus and Grafana for comprehensive cluster health monitoring and performance metrics collection at the edge.

**Cluster Operations**: Rolling updates can be performed for dot releases without service interruption. Configuration changes can typically be applied without full service restarts (server restart takes approximately 5 seconds; database schema updates may require up to 30 seconds). Use environment variables for configuration management and mmctl for cluster administration.

#### Compliance and Retention

Sovereign environments often require strict enforcement of retention policies, legal hold, and export controls. Configure Mattermost's built-in compliance features to meet organizational mandates.

- Enable compliance export and monitoring to produce auditable exports of message data and user activity logs.
- Configure message retention and legal hold policies to align with applicable regulations.
- Integrate with your organization's eDiscovery and archiving systems as required.

### AI Agents Plugin

The [Agents plugin](https://github.com/mattermost/mattermost-plugin-agents) integrates AI capabilities directly into Mattermost, offering multiple AI assistants, thread and channel summarization, semantic search, and meeting transcription.

#### Self-hosted LLMs

Deploy an OpenAI compatible LLM on tactical infrastructure to ensure AI capabilities remain fully sovereign and operational in disconnected scenarios. A self-hosted LLM can power message and call summarization, semantic search, and mission-tuned AI agents without relying on public cloud AI services. This guarantees compliance with strict data handling mandates and enables AI-enhanced workflows to function locally, even during extended disconnections.

### Calls Plugin

The [Calls plugin](https://github.com/mattermost/mattermost-plugin-calls) enables real-time voice calling and screen sharing directly within channels.

#### Self-hosted Audio & Screensharing

Effective collaboration at the tactical edge requires all voice and screen sharing capabilities remain operational without reliance on the internet or third-party services. Deploy Mattermost Calls in a self-hosted configuration, including:

- The `rtcd` service for scalable, low-latency media routing hosted on-premises. Run multiple `rtcd` nodes for redundancy.
- The `calls-offloader` service offloads heavy processing tasks like recording, transcription and live captioning to a locally hosted compliance-approved job server.

### Mobile Apps

Mattermost provides native mobile applications for iPhone and Android, with support for ID-only push notifications to ensure compliance with data sovereignty requirements.

#### ID-only Push Notifications

To prevent sensitive message content from being transmitted to external notification services such as Apple Push Notification Service (APNS) and Firebase Cloud Messaging (FCM), configure Mattermost to use ID-only push notifications. In this mode, only a message identifier is sent to public push notification services, and the client retrieves the content securely from the Mattermost server over an encrypted channel.
