## Mission Partner Collaboration

# 1. Overview

Mission partner collaboration extends sovereign and edge deployment models to enable joint and allied operations across organizations using Mattermost and/or Microsoft 365. This architecture delivers a secure, sovereign, and intelligent mission environment that is federated across enterprise and coalition partner networks, enabling interoperability while maintaining compliance and control.

Organizations using Mattermost, Mattermost with Microsoft 365, and Microsoft 365 alone can securely interoperate on joint missions. Federation is achieved using connected workspaces, Matrix connector, guest accounts, and auto-translation to ensure fast, accurate comprehension across globally distributed teams. Additionally, integrating external data feeds, workflow automation, and sovereign AI enables allied and partner users to collaborate at mission speed while enforcing zero-trust policies and maintaining data sovereignty.

Mattermost deployments may be hosted on-premises or in sovereign clouds, enabling allies and partners to retain control over sensitive data while extending interoperability to coalition partner enterprise networks.

---

# 2. Problem Statement

Multi-organizational collaborations face complex communication challenges:
- Different organizations use different platforms (Teams, Mattermost, Matrix)
- Some organizations use both Teams and Mattermost
- External partners need controlled access without full organizational membership
- Organizations may speak different languages
- Security and compliance requirements vary across organizations
- Standard secure communication channels (HTTPS) must be used for inter-org communication

Traditional solutions require everyone to adopt the same platform or use insecure external tools. Organizations need a flexible collaboration framework that:
- Bridges multiple communication platforms
- Provides granular access control for external users
- Enables cross-organizational communication over standard protocols
- Breaks down language barriers with automatic translation
- Maintains security and compliance standards

# 3. Solution Architecture

A multi-layer collaboration approach addresses diverse organizational needs:

## 3.1 Layer 1: External Teams Users → Mattermost Access
Teams users from external organizations receive Mattermost guest accounts and access Mattermost via the [Mission Collaboration plugin](https://github.com/mattermost/mattermost-plugin-msteams-devsecops) embedded in their Teams client.

## 3.2 Layer 2: Mattermost-to-Mattermost Collaboration
Organizations with Mattermost deployments establish secure connections and share specific channels using [Connected Workspaces](https://docs.mattermost.com/administration-guide/onboard/connected-workspaces.html) over standard HTTPS.

## 3.3 Layer 3: Mattermost-to-Matrix Federation
Organizations using Matrix servers connect to Mattermost via the [Matrix bridge plugin](https://github.com/mattermost/mattermost-plugin-matrix-bridge) for bidirectional communication.

## 3.4 Layer 4: Automatic Language Translation
AI-powered [automatic translation](https://github.com/mattermost/mattermost-plugin-channel-translations) enables seamless communication across language barriers in all shared channels.

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

## 5.1 Allied or Partner Networks

Globally distributed and segregated networks for each allied or partner organization.
- Networks may have a firewall or access gateway protecting egress and ingress, such as network policies, IP allowlists, or WAFs depending on networking configurations.
- Networks may operate in contested environments where internet connectivity is intermittent.

## 5.2 Users

Enterprise, allied, and coalition partner users accessing client applications for Mattermost and/or Microsoft 365.

## 5.3 Microsoft Entra ID (Identity Provider)

Partnered organizations using Microsoft 365 services may use Entra ID for unified authentication to M365 and Mattermost applications. (Optional)

## 5.4 Federation Services

- **[Connected Workspaces](https://docs.mattermost.com/administration-guide/onboard/connected-workspaces.html)**: Federated collaboration across partner networks, with seamless synchronization of messages, threads, and files.
- **[Guest Accounts](https://docs.mattermost.com/administration-guide/onboard/guest-accounts.html)**: Secure participation of external mission partners with least-privilege access. (Optional)
- **[Matrix](https://github.com/mattermost/mattermost-plugin-matrix-bridge) & XMPP Interoperability**: Federation with legacy partner systems for cross-domain coalition collaboration. (Optional)

### 5.4.1 Guest Accounts for External Teams Users

[Mattermost Guest Accounts Documentation](https://docs.mattermost.com/administration-guide/onboard/guest-accounts.html)

**Capabilities:**
External Teams users access Mattermost as guests via the Mission Collaboration plugin with controlled permissions:

- **Access Control**: Guests can only access specific channels they're invited to
- **Authentication**: Support for email invitation, AD/LDAP, or SAML 2.0
- **User Identification**: Guests are marked with "GUEST" badge
- **Allowed Actions**: Pin messages, use slash commands, favorite channels, update profile
- **Restrictions**: Cannot invite others, create DMs outside assigned channels, or join open teams

**Management:**
- System admins control who can invite guests
- Admin ability to promote/demote guest roles
- Individual guest account controls
- Guest accounts count as paid users

### 5.4.2 Cross-Organization Mattermost Collaboration

[Connected Workspaces Documentation](https://docs.mattermost.com/administration-guide/onboard/connected-workspaces.html)

**Capabilities:**
Shared channels synchronize messages in real-time between separate Mattermost instances:

- **Real-Time Sync**: Messages, emoji reactions, and file sharing
- **Channel Types**: Support for public and private shared channels
- **Permission Control**: Each server maintains separate access controls
- **Participation Modes**: Read-only or full participation sharing
- **Advanced Features** (v10.10+): Direct messages between remote users, membership sync, remote user discovery

#### Resilient Federation for Joint Operations

Connected workspaces allow federated collaboration across multiple organizations and networks while maintaining local data control of each Mattermost deployment. Messages, threads, and files are securely synchronized between environments, ensuring mission continuity for multinational operations without requiring partners to join a single centralized deployment.

- Enforce zero-trust access and ensure that only authorized mission partners can view or contribute to shared collaboration channels.
- Configure auto-translation in shared channels for seamless multilingual cross-domain collaboration.
- Mattermost instances can operate independently during outages or intermittent connectivity and sync conversations once connectivity returns. For deployments in contested environments with limited connectivity, see [Air-Gapped Deployment](https://docs.mattermost.com/deployment-guide/server/air-gapped-deployment.html) guidance.

Many mission partners continue to operate on legacy systems such as Matrix and XMPP. To enable joint operations without forcing migration, Mattermost supports secure interoperability with these environments for continuity of coalition communications while allowing modernized workflows to extend across federated networks.

- Synchronize Mattermost channels with Matrix or XMPP rooms, allowing messages, threads, and attachments to flow across systems in real-time.
- Each organization maintains control of its data and infrastructure, while interoperability is enabled through federation bridges rather than centralized services.

#### Controlled External Access

Mission partner collaboration may require involving external users such as allied forces, contractors, or coalition partners that do not have Mattermost deployments themselves. Guest accounts provide a controlled mechanism to enable these users to participate in joint mission operations while maintaining strict compliance and security boundaries.

- Guest accounts are restricted to specific teams and channels. This ensures external users only have access to mission-critical resources necessary for their role.
- Guests can be granted access to shared channels, enabling collaboration with additional trusted organizations through connected workspaces.

**Network Security Architecture**: When federating Mattermost instances across partner organizations, deploy Mattermost servers in a demilitarized zone (DMZ) network segment to provide controlled access while protecting internal network resources. This architecture provides defense-in-depth by isolating federation traffic from internal systems.

- **DMZ Deployment**: Position Mattermost application servers in the DMZ network segment, allowing both internal users and external partner federation traffic to access the collaboration platform through controlled network boundaries.
- **VPN Termination**: Terminate site-to-site VPN connections at the network perimeter or DMZ layer, enabling encrypted partner connectivity without exposing internal network infrastructure. VPN tunnels establish secure communication channels between partner organizations over the internet.
- **Firewall Segmentation**: Deploy ingress and egress firewall rules to control traffic flow between the DMZ, internal network, and external partner networks. Restrict database and object storage access to only originate from the DMZ segment where Mattermost servers reside.
- **Federation Traffic Isolation**: Partner federation traffic (Connected Workspaces synchronization over HTTPS port 443/TCP) remains isolated within the DMZ, protecting internal systems while enabling cross-organizational collaboration.

This DMZ architecture enables secure partner collaboration while maintaining network segmentation and enforcing zero-trust principles across organizational boundaries.

### 5.4.3 Matrix Federation

[Mattermost Matrix Bridge Plugin](https://github.com/mattermost/mattermost-plugin-matrix-bridge)

**Capabilities:**
Bidirectional communication between Mattermost and Matrix chat platforms:

- **Real Users**: Preserves authentic user identities, names, and avatars
- **Rich Content**: Syncs message formatting, mentions, threads, and file attachments
- **Emoji Support**: Over 4,400 emoji reactions synchronized
- **Message Lifecycle**: Handles edits, deletions, and reply threads
- **Security**: Prevents message loops and ensures proper authentication

### 5.4.4 Automatic Language Translation

[AI Auto Translations Plugin](https://github.com/mattermost/mattermost-plugin-channel-translations)

**Capabilities:**
Automatic message translation within configured channels:

- **Channel-Specific**: Translation enabled on per-channel basis
- **Multiple Languages**: Configurable target languages
- **User Preferences**: Individual users select their preferred language
- **Inline Display**: Translations appear seamlessly with original messages
- **AI-Powered**: Leverages Agents plugin with LLM backend

## 5.5 Client Applications

- **Mattermost Desktop Apps**: Access Mattermost directly by deploying desktop or web apps in your organization.
- **Mattermost Mobile Apps**: Access Mattermost via iPhone and Android apps, with support for ID-only push notifications to ensure compliance with data sovereignty requirements. (Optional)
- **Microsoft 365 Desktop Apps**: For partnered organizations using Microsoft 365 services, Teams and Outlook can be deployed with the embedded Mattermost application for cross-domain partner collaboration within a familiar interface. (Optional)

### 5.5.1 ID-only Push Notifications

To prevent sensitive message content from being transmitted to external notification services such as Apple Push Notification Service (APNS) and Firebase Cloud Messaging (FCM), configure Mattermost to use ID-only push notifications. In this mode, only a message identifier is sent to public push notification services, and the client retrieves the content securely from the Mattermost server over an encrypted channel.

## 5.6 Mattermost Server

Core application server deployed for sovereign collaboration on private cloud or local infrastructure, such as Azure or Azure Local, to maintain compliance with STIG, FedRAMP, and NIST 800-53 standards. The server provides all capabilities described in the Core Mattermost Capabilities section above, supporting federated collaboration across partner organizations. For large-scale deployments, see [Scaling for Enterprise](https://docs.mattermost.com/administration-guide/scale/scaling-for-enterprise.html) for detailed performance tuning guidance.

### 5.6.1 High Availability and Fault Tolerance

Deploy Mattermost in a [cluster-based architecture](https://docs.mattermost.com/administration-guide/scale/high-availability-cluster-based-deployment.html) to ensure continued availability during outages or hardware failures. All cluster nodes must reside within the same datacenter to ensure optimal performance and consistent network latency. High availability requires redundant infrastructure across each critical component:

- **Application servers**: Scale horizontally across multiple nodes (64-bit x86 architecture) with a load balancer distributing client traffic. Servers communicate via gossip protocol for cluster coordination and leader election. All nodes must maintain identical `config.json` configurations and synchronized time via Network Time Protocol (NTP).
- **Database layer**: Use PostgreSQL master/read replica architecture with connection pooling and automatic failover. Configure read replicas to offload query traffic and search replicas for dedicated search operations. Optimize database connection limits, work memory settings, and autovacuum parameters for performance.
- **Object storage**: Configure S3-compatible backends with erasure coding or replication for durability. All application servers must access shared file storage (NAS or S3) to ensure consistent data availability.
- **Monitoring**: Deploy Prometheus and Grafana for comprehensive cluster health monitoring and performance metrics collection across federated partner networks.

**Cluster Operations**: Rolling updates can be performed for dot releases without service interruption. Configuration changes can typically be applied without full service restarts (server restart takes approximately 5 seconds; database schema updates may require up to 30 seconds). Use environment variables for configuration management and mmctl for cluster administration.

### 5.6.2 Zero-trust Access Controls

Mission partner collaboration environments should adopt zero-trust principles by implementing attribute-based access control (ABAC) to ensure access to mission channels is governed by dynamic attributes such as role, clearance, location, and mission context.

- Restrict channel access based on user attributes rather than static groups.
- Continuously audit ABAC policies to ensure compliance with multinational operational and legal requirements.

### 5.6.3 Compliance and Retention

Sovereign environments often require strict enforcement of retention policies, legal hold, and export controls. Configure Mattermost's built-in compliance features to meet agency or sectoral mandates.

- Enable compliance export and monitoring to produce auditable exports of message data and user activity logs.
- Configure message retention and legal hold policies to align with applicable regulations.
- Integrate with your organization's eDiscovery and archiving systems as required.

## 5.7 Proxy Server

The proxy server handles HTTP(S) routing within the cluster, directing traffic between the server and clients accessing Mattermost services, including requests from users in connected organizations. NGINX is recommended for load balancing with support for WebSocket connections, health check endpoints, and sticky sessions. The proxy layer provides SSL termination and distributes client traffic across application servers.

## 5.8 PostgreSQL Database

Stores persistent application data on a PostgreSQL v13+ database, such as Azure Database for PostgreSQL. For high availability, configure a master/read replica architecture with connection pooling. Optimize PostgreSQL settings including connection limits, work memory allocation, and autovacuum parameters. Support for search replicas enables dedicated database resources for search operations.

## 5.9 Object Storage

File uploads, images, and attachments are stored outside the application node on an S3-compatible store, such as MinIO, or Amazon S3. For high availability deployments, object storage must be shared across all application servers using network-attached storage (NAS) or S3-compatible object storage with erasure coding or replication for durability.

## 5.10 Recording Instance

`calls-offloader` job service to offload heavy processing tasks from Mattermost Calls, such as recordings, transcriptions, and live captioning, to local infrastructure or private cloud. (Optional)

## 5.11 Integration Framework

Custom apps, plugins, and webhooks can be deployed for real-time data integrations and alerting. (Optional)

## 5.12 Self-hosted LLM

Locally hosted OpenAI compatible LLM for agentic powered collaboration. (Optional)

### 5.12.1 Sovereign AI

AI capabilities enhance mission collaboration with summarization, translation, semantic search, and decision support. Sovereign AI ensures these capabilities remain fully under organizational control, without reliance on public cloud services or external data processing. Deploying AI in a self-hosted or compliance-approved environment enables secure, mission-ready augmentation.

- Deploy OpenAI-compatible language models on local or private cloud infrastructure to maintain data sovereignty and ensure offline availability.
- Configure custom agents for summarization, workflow automation, and decision support while enforcing organizational compliance policies.
- Enable multilingual collaboration in shared channels using sovereign AI services to provide real-time translations across partner organizations.
- Embed AI into operational playbooks for automated task execution, situational summaries, and proactive recommendations.
- Allow authorized users from partner organizations to securely access locally hosted LLMs through shared channels in connected workspaces.

## 5.13 Microsoft Global Network

World-wide network of Microsoft data centers, delivering public cloud services including M365 and Azure OpenAI. (Optional)

## 5.14 Calls Plugin

The [Calls plugin](https://github.com/mattermost/mattermost-plugin-calls) enables real-time voice calling and screen sharing directly within channels.

### 5.14.1 Sovereign Audio & Screensharing

Deploy Mattermost Calls in a self-hosted configuration to ensure voice and screen sharing capabilities remain operational without reliance on the internet, and that media traffic does not traverse non-compliant third-party services.

- The `rtcd` service for scalable, low-latency media routing hosted on-premises. Run multiple `rtcd` nodes for redundancy.
- The `calls-offloader` service offloads heavy processing tasks like recording, transcription and live captioning to a locally hosted compliance-approved job server.
