# ADR-002 — Environment Separation

## Metadata

- Decision ID: ADR-002
- Date: 2026-11-24
- Status: Accepted
- Owner: Platform Owner
- Scope: Qman Enterprise Platform
- Classification: Public

## Context

The Qman Enterprise Platform combines production workloads,
software development, infrastructure engineering and
technology experimentation.

Production services must remain stable, secure and available
while new technologies and infrastructure configurations
are developed and validated.

The platform currently has two Linux virtual private servers
available within the Qman trust domain.

A separate infrastructure environment exists within the
Talland trust domain.

A clear separation between production, development,
staging and experimental workloads is required to reduce
operational and security risks.

## Decision

Assign dedicated responsibilities to each Qman server.

QMAN-01 will operate exclusively as a production server.

QMAN-02 will operate as a development, staging and
experimental laboratory environment.

The engineering workstation will be used for local
development, infrastructure management and automation.

Talland infrastructure will remain a separate administrative
and security trust domain.

Production workloads must not depend on the availability
of development or experimental environments.

## Architecture Principles

- Separation of environments
- Separation of trust domains
- Security by design
- Least privilege
- Infrastructure as Code
- Configuration as Code
- Production stability
- Controlled deployment promotion
- Reproducible environments
- Cost efficiency
- Operational simplicity

## Environment Overview

| Environment | Purpose | Criticality |
|-------------|---------|-------------|
| QMAN-01 | Production | High |
| QMAN-02 | Development, Staging and Lab | Medium |
| Engineering Workstation | Local development and management | Medium |
| Talland | Independent organizational infrastructure | Independently defined |

## Production Environment

### QMAN-01

Purpose:

- Host approved production applications.
- Provide stable and secure public services.
- Run reproducible and version-controlled deployments.
- Maintain operational reliability.

Initial production workload:

- Notes website (notes.qman.io)

Initial technology stack:

- Ubuntu LTS
- Nginx
- Node.js
- Systemd
- Tailscale
- Restic
- Backblaze B2

Production requirements:

- Minimal operating system installation
- Restricted administrative access
- Automated security updates
- Controlled application deployments
- Versioned deployment artifacts
- Documented rollback procedures
- Validated backup and recovery procedures
- Monitoring and alerting
- TLS encryption for public services

Experimental software must not be installed directly
on the production server.

Production configuration changes must be managed through
approved automation wherever possible.

## Development, Staging and Lab Environment

### QMAN-02

Purpose:

- Validate application deployments.
- Test infrastructure configurations.
- Validate Ansible roles.
- Test security policies.
- Test application upgrades.
- Perform backup and recovery experiments.
- Evaluate new technologies.
- Develop platform engineering capabilities.

Potential technology stack:

- Ubuntu LTS
- Nginx
- Node.js
- Systemd
- Tailscale
- Docker
- Kubernetes
- GitOps tooling
- Monitoring and observability tooling
- Security scanning tools

Technologies will be introduced incrementally based on
documented architectural decisions.

### Development

The development environment supports:

- Application development
- Infrastructure development
- Automation development
- Integration testing
- Configuration validation

### Staging

The staging environment provides a controlled environment
for validating release candidates before production
deployment.

Staging should resemble production where relevant,
including:

- Operating system version
- Application runtime
- Web server configuration
- Deployment procedures
- Security baseline
- Service configuration
- Monitoring configuration

Staging does not need to reproduce production hardware
capacity.

### Lab

The laboratory environment supports:

- Container experiments
- Kubernetes experiments
- GitOps workflows
- Security experiments
- Infrastructure automation
- Technology evaluation
- Platform engineering experiments

Laboratory workloads must be logically isolated from
staging workloads.

Resource limits and network restrictions must be applied
where appropriate.

Experiments requiring privileged host access, destructive
changes or unrestricted network access must not run
alongside active staging validation.

The laboratory environment must not have access to
production credentials or production application data.

### Shared Infrastructure Risk

Development, staging and lab workloads share the same
physical virtual server.

Logical separation does not provide the same security
or availability guarantees as independent servers.

Resource exhaustion, operating system failures and
privileged experiments may affect all environments
hosted on QMAN-02.

This shared infrastructure risk is accepted during the
initial platform development phase.

Additional infrastructure may be introduced when
isolation requirements increase.

## Engineering Workstation

The engineering workstation is the primary interface
for platform development and administration.

Responsibilities:

- Source code development
- Infrastructure development
- Terraform operations
- Ansible development and execution
- Architecture documentation
- GitHub administration
- Secure credential management
- Local testing
- Platform administration

Administrative credentials must be protected using
appropriate workstation security controls.

Workstation requirements:

- Full disk encryption
- Secure credential storage
- Multi-factor authentication
- Restricted SSH private key permissions
- Regular operating system updates
- Secure screen locking

The workstation must not become a runtime dependency
for production services.

## Trust Domain Separation

Qman and Talland represent independent administrative
and security trust domains.

The following must remain separated:

- GitHub organizations
- Infrastructure inventories
- Administrative identities
- Workload identities
- SSH credentials
- Infrastructure state
- Deployment credentials
- Backup repositories
- Application secrets
- Customer and organizational data
- Security policies

Cross-domain access requires explicit organizational
authorization and documented justification.

Talland infrastructure must comply with applicable
organizational policies and security requirements.

Qman infrastructure decisions do not automatically
apply to Talland.

## Network Separation

Administrative access must use authenticated and
authorized private network connections.

Tailscale will be used as the primary administrative
connectivity solution.

Network requirements:

- Production administrative services must not be
  unnecessarily exposed to the public internet.
- Public production services must use encrypted transport.
- Development and laboratory services must be private
  by default.
- Access between environments must be explicitly authorized.
- Laboratory workloads must not have unrestricted access
  to production resources.
- Network policies must follow least privilege principles.

Network architecture and access policies will be defined
in a separate Architecture Decision Record.

## Identity and Access Management

Each environment must use appropriately scoped identities
and credentials.

Requirements:

- Multi-factor authentication for administrative accounts
- Dedicated workload identities
- Environment-specific deployment credentials
- Environment-specific application secrets
- Least privilege access policies
- Secure credential storage
- Credential rotation capabilities
- Auditable administrative access

Development and laboratory identities must not receive
production permissions by default.

Production deployment identities must only receive the
permissions required to deploy approved application
artifacts.

## Deployment Architecture

Application releases must follow a controlled promotion
process.

The intended release workflow is:

1. Local development
2. Source code commit
3. Automated validation
4. Automated testing
5. Security scanning
6. Artifact creation
7. Staging deployment
8. Staging validation
9. Production approval
10. Production deployment
11. Production health verification

The same immutable release artifact should be promoted
from staging to production without rebuilding.

Production deployment must not rely on manually copying
development files or performing an uncontrolled Git pull
on the production server.

Deployment workflows must support rollback to a known
working release.

Detailed deployment architecture will be defined in a
separate Architecture Decision Record.

## Infrastructure as Code

Infrastructure configuration must be version-controlled
and reproducible.

Terraform will be used for infrastructure provisioning
where supported by the infrastructure provider.

Ansible will be used for operating system configuration,
security hardening and application runtime configuration.

Environment-specific configuration must be separated.

Infrastructure requirements:

- Separate environment inventories
- Separate Terraform state where applicable
- Environment-specific variables
- Reusable infrastructure modules
- Reusable Ansible roles
- Explicit environment targeting
- Protection against accidental production changes
- Validation before infrastructure changes are applied

Sensitive infrastructure state must not be stored in
public GitHub repositories.

## Data Separation

Production application data must remain isolated from
development and experimental workloads.

Requirements:

- No production secrets in development environments
- No uncontrolled copying of production databases
- Synthetic or anonymized test data where possible
- Separate database credentials
- Separate application configuration
- Separate persistent storage
- Environment-specific backup policies

Production data may only be restored into a non-production
environment when explicitly authorized and appropriately
protected.

## Backup and Recovery

Backup architecture is defined in ADR-001.

Production environments must have documented and validated
backup and recovery procedures.

Development and laboratory environments should prioritize
reproducibility through source control and Infrastructure
as Code.

Persistent non-production data must be backed up when
its loss would have an unacceptable impact.

Backup requirements:

- Separate production and non-production backup credentials
- Appropriate repository separation
- Environment-specific retention policies
- Encrypted offsite backups
- Periodic recovery verification

Experimental environments must not have access to
production backup credentials.

## Monitoring and Observability

Each environment must provide sufficient visibility into
its operational state.

Production monitoring must prioritize:

- Availability
- Application health
- Resource utilization
- Security events
- Backup status
- Deployment status
- Certificate expiration
- Patch compliance

Development and laboratory monitoring may additionally
support:

- Performance experiments
- Infrastructure testing
- Technology evaluation
- Monitoring configuration development

Monitoring architecture must prevent experimental
workloads from affecting production monitoring reliability.

## Security Requirements

- Production services must use a hardened security baseline.
- Administrative access must be restricted.
- Secrets must never be committed to source control.
- Production and non-production credentials must be separated.
- Software dependencies must be regularly reviewed.
- Security vulnerabilities must be monitored.
- Security updates must be applied according to policy.
- Production changes must be traceable.
- Experimental workloads must not compromise production.
- Privileged laboratory workloads must be explicitly controlled.
- Public documentation must not expose sensitive operational
  information.

## Cost Management

The platform will initially use the two existing Qman
virtual private servers.

Additional infrastructure must only be introduced when
supported by a documented requirement.

Cost management principles:

- Reuse existing infrastructure.
- Avoid unnecessary cloud services.
- Introduce technologies incrementally.
- Monitor infrastructure utilization.
- Apply resource limits to experimental workloads.
- Evaluate capacity before purchasing additional services.
- Maintain proportional operational complexity.

Environment separation must remain proportionate to
platform size, risk and workload requirements.

## Alternatives Considered

### Single Shared Production and Lab Server

Advantages:

- Lower infrastructure costs
- Reduced initial configuration effort

Disadvantages:

- Increased production security risks
- Resource contention
- Uncontrolled operational dependencies
- Higher risk of production outages

Decision: Not selected.

### Dedicated Server for Every Environment

Advantages:

- Strong infrastructure isolation
- Independent resource allocation
- Reduced shared failure domains

Disadvantages:

- Increased infrastructure costs
- Additional maintenance requirements
- Increased operational complexity

Decision: Deferred until justified by platform growth.

### QMAN-02 as Customer Production

Advantages:

- Dedicated infrastructure for customer workloads
- Separation from personal production applications

Disadvantages:

- No dedicated remote development and staging environment
- Reduced capacity for infrastructure experimentation
- Limited opportunity to validate production deployments

Decision: Not selected for the current platform phase.

### Local Development and Lab Only

Advantages:

- No additional server requirements
- Reduced hosting costs
- Direct control over experimental workloads

Disadvantages:

- Workstation availability dependency
- Reduced similarity to remote production infrastructure
- Limited remote integration testing

Decision: Suitable as a supplementary development
environment, but not as the primary staging environment.

## Known Risks

| Risk | Mitigation |
|------|------------|
| Lab workloads affect staging | Resource limits and logical isolation |
| Lab workloads compromise the shared host | Restrict privileged experiments |
| Production configuration drift | Ansible and automated validation |
| Accidental production deployment | Environment-specific identities and approval controls |
| Production secrets exposed to development | Dedicated credentials and secret isolation |
| Unauthorized cross-environment access | Explicit network access policies |
| Staging differs from production | Shared configuration roles and runtime standards |
| Resource exhaustion | Monitoring and resource limits |
| Accidental infrastructure destruction | Separate state, validation and approval |
| Excessive operational complexity | Incremental implementation and regular review |

## Current Limitations

- The target environments have not yet been rebuilt.
- Network access policies have not yet been finalized.
- Environment-specific workload identities have not yet
  been configured.
- Automated deployment promotion has not yet been implemented.
- Staging infrastructure has not yet been validated.
- Laboratory isolation has not yet been implemented.
- Central monitoring has not yet been established.
- Provider-specific Terraform capabilities require validation.
- Existing production and customer data must be reviewed
  before destructive infrastructure changes are performed.
- Shared staging and lab infrastructure does not provide
  independent host-level isolation.

These limitations must be addressed during the relevant
implementation phases.

## Implementation Roadmap

1. Document environment responsibilities.
2. Establish separate infrastructure inventories.
3. Define network architecture.
4. Define identity and access architecture.
5. Define deployment architecture.
6. Validate infrastructure provider capabilities.
7. Establish Terraform foundations.
8. Establish Ansible foundations.
9. Establish the Linux security baseline.
10. Rebuild QMAN-02 after data review and authorization.
11. Configure development and staging environments.
12. Validate automated application deployments.
13. Validate security and recovery procedures.
14. Rebuild QMAN-01 after data review and authorization.
15. Deploy the initial production application.
16. Implement monitoring and alerting.
17. Introduce isolated laboratory workloads.
18. Evaluate container and Kubernetes technologies.

## Acceptance Criteria

- [ ] Environment responsibilities documented.
- [ ] Qman and Talland trust domains separated.
- [ ] Separate infrastructure inventories established.
- [ ] Production and non-production credentials separated.
- [ ] Network access policies implemented.
- [ ] Production security baseline implemented.
- [ ] Development environment operational.
- [ ] Staging environment operational.
- [ ] Laboratory isolation validated.
- [ ] Automated staging deployment successful.
- [ ] Production deployment promotion validated.
- [ ] Production rollback procedure validated.
- [ ] Environment-specific backup policies implemented.
- [ ] Monitoring and alerting operational.
- [ ] Infrastructure documentation completed.

## Consequences

The selected architecture provides a clear separation
between production workloads and infrastructure
development activities.

It enables controlled experimentation without requiring
additional virtual private servers during the initial
platform development phase.

Production stability takes priority over experimental
activities.

QMAN-02 provides a dedicated environment for developing
and validating infrastructure automation, application
deployments and emerging platform technologies.

The shared staging and laboratory infrastructure requires
appropriate resource management and security controls.

Future infrastructure expansion must be justified by
documented capacity, security or availability requirements.

## Review Policy

Review this decision:

- Quarterly
- After a significant infrastructure change
- After a security incident
- When production workload requirements change
- When isolation requirements increase
- When infrastructure capacity becomes insufficient
- Before introducing customer production workloads

## Related Architecture Decision Records

- ADR-001 — Backup Storage Architecture

Planned architecture decisions:

- ADR-003 — Network Architecture
- ADR-004 — Deployment Architecture
- ADR-005 — Security Architecture

## Revision History

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-11-24 | Initial environment separation decision |

---

Qman Enterprise Platform

Secure by design. Automated by default.

