# ADR-001 — Backup Storage Architecture

## Metadata

- Decision ID: ADR-001
- Date: 2026-11-24
- Status: Accepted
- Owner: Platform Owner
- Scope: Qman Enterprise Platform
- Classification: Public

## Context

The Qman Enterprise Platform requires a secure, reliable,
cost-efficient and independently hosted backup solution.

The platform currently operates a small number of virtual
private servers with a limited amount of persistent
application data.

Application source code is maintained in GitHub.
Infrastructure and server configuration will be managed
through Infrastructure as Code.

Production servers should be replaceable without losing
critical application data or operational configuration.

The backup architecture must support future growth without
introducing unnecessary operational complexity or costs.

## Decision

Use Backblaze B2 Cloud Storage in the European region
as the primary offsite backup destination.

Use Restic for encrypted, incremental and deduplicated
backups.

Automate backup operations using Ansible and systemd timers.

Maintain separate backup repositories and dedicated
credentials for each production server.

AWS is excluded from the current project scope to reduce
operational complexity and administrative overhead.

## Architecture Principles

- Security by design
- Automation first
- Infrastructure as Code
- Least privilege
- Separation of trust domains
- Cost efficiency by design
- Reproducible infrastructure
- Verifiable recovery
- Operational simplicity

## Technology Stack

| Component | Technology |
|-----------|------------|
| Object storage | Backblaze B2 |
| Storage region | EU Central |
| Backup engine | Restic |
| Storage interface | S3-compatible API |
| Client-side encryption | Restic native encryption |
| Server-side encryption | Backblaze B2 encryption |
| Configuration management | Ansible |
| Scheduling | systemd timers |
| Source control | GitHub |
| Monitoring | To be implemented |

## Backup Scope

The backup solution must protect:

- Persistent application data
- Application databases
- User-generated content
- Application configuration
- Required operational configuration
- Critical recovery information

Database backups must be application-consistent.

The following are generally excluded:

- Operating system packages
- Reproducible application builds
- Temporary files
- Package caches
- Node.js dependencies
- Python virtual environments
- Reproducible container images

Application source code is maintained in GitHub.

GitHub does not replace backups of persistent production
data or uncommitted local changes.

## Security Requirements

- All backup buckets must be private.
- Multi-factor authentication must be enabled.
- Client-side encryption is mandatory.
- Server-side encryption must be enabled.
- Master application keys must not be used by workloads.
- Each server must use dedicated application credentials.
- Access must be restricted to the required backup bucket.
- Credentials must never be committed to Git.
- Credentials must be stored with restrictive permissions.
- Recovery passwords must be stored independently.
- Backup traffic must use encrypted transport.
- Credential rotation must be supported.
- Backup deletion protection must be evaluated and tested.

## Trust Domain Separation

Qman and Talland are separate administrative and security
trust domains.

Talland infrastructure and data must not use the Qman
backup environment unless explicitly authorized by the
organization.

Backup credentials, repositories and operational data
must remain isolated between trust domains.

Customer data must be handled according to applicable
contractual and privacy requirements.

## Recovery Objectives

The following initial recovery objectives are proposed:

| Environment | RPO | RTO |
|-------------|-----|-----|
| Notes Production | 24 hours | 2 hours |
| Customer Applications | 4 hours | 2 hours |
| Development | 24 hours | 24 hours |
| Experimental | Best effort | Best effort |

Recovery objectives must be validated through restore tests.

Customer recovery objectives require explicit agreement
and must not be treated as guaranteed service levels
until verified.

## Backup Schedule

Proposed initial schedule:

- Production backups: Daily
- Customer application backups: According to agreed RPO
- Repository integrity checks: Weekly
- Restore verification: Monthly
- Backup configuration review: Quarterly

Backup scheduling must be aligned with the required
recovery objectives.

## Retention Policy

Proposed initial production retention:

- Daily snapshots: 7
- Weekly snapshots: 4
- Monthly snapshots: 12

Restic retention management and Backblaze lifecycle
settings must be aligned.

Retention settings must be validated before automated
snapshot deletion or repository pruning is enabled.

Object version retention must be monitored to prevent
uncontrolled storage growth.

## Recovery Verification

A successful backup operation does not guarantee
successful recovery.

Recovery verification must include:

- Successful snapshot creation
- Repository integrity validation
- Restoration to an isolated location
- File integrity comparison
- Application-level validation where applicable
- Recovery duration measurement
- Documentation of recovery results

Production infrastructure must not be reinstalled or
destroyed until critical data has been backed up and
its recovery has been verified.

## Monitoring and Alerting

The backup solution must eventually provide monitoring
and alerting for:

- Failed backup operations
- Missing or overdue backups
- Failed integrity checks
- Failed restore tests
- Unexpected storage growth
- Authentication failures
- Recovery objective violations

Monitoring and alerting will be implemented as part of
the platform observability architecture.

## Cost Management

The backup solution must remain proportionate to the
size and requirements of the platform.

Initial target monthly cost: EUR 2 or less.

Cost management requirements:

- Pay-per-use storage
- Regular storage usage reviews
- Controlled snapshot retention
- Monitoring of historical object versions
- Avoidance of unnecessary paid services
- Evaluation of data transfer and API request costs
- Periodic review of storage growth

The monthly cost target is not a guaranteed spending cap.

Additional services require a documented justification.

## Alternatives Considered

### Amazon S3

Advantages:

- Mature enterprise object storage
- Extensive Infrastructure as Code support
- Advanced identity and access management
- Comprehensive security capabilities

Disadvantages:

- Additional cloud platform to administer
- Increased operational complexity
- Additional identity and cost management requirements

Decision: Not selected for the current project phase.

### Hetzner Object Storage

Advantages:

- European hosting
- S3-compatible API
- Predictable pricing

Disadvantages:

- Fixed monthly base cost
- Less cost-efficient for the initial small backup volume

Decision: Not selected.

### Hetzner Storage Box

Advantages:

- Simple remote backup storage
- Predictable pricing

Disadvantages:

- Fixed monthly storage subscription
- No native S3-compatible interface

Decision: Not selected.

### Local Backup Storage

Advantages:

- Low additional cost
- Direct control over backup storage

Disadvantages:

- Limited availability
- Physical location risks
- Dependency on local infrastructure
- Additional hardware maintenance

Decision: Suitable as a supplementary backup destination,
but not as the primary offsite backup solution.

## Known Risks

| Risk | Mitigation |
|------|------------|
| Loss of encryption password | Independent secure password storage |
| Compromised application credentials | Bucket-scoped credentials and rotation |
| Accidental backup deletion | Evaluate deletion protection and Object Lock |
| Uncontrolled storage growth | Retention and usage monitoring |
| Corrupted backup repository | Scheduled integrity checks |
| Unsuccessful recovery | Periodic restore verification |
| Inconsistent database backup | Application-consistent backup procedures |
| Provider unavailability | Evaluate an independent secondary backup destination |

## Current Limitations

- Backup automation has not yet been implemented.
- Production backup coverage has not yet been verified.
- Object Lock is not currently enabled.
- Backup immutability has not yet been validated.
- Central monitoring and alerting are not yet implemented.
- A secondary independent backup destination has not yet
  been established.
- Recovery objectives have not yet been demonstrated.

These limitations must be addressed before the platform
can be considered fully production-ready.

## Implementation Roadmap

1. Establish the Backblaze B2 account.
2. Enable multi-factor authentication.
3. Create private backup storage.
4. Create restricted application credentials.
5. Initialize the Restic repository.
6. Perform an initial encrypted test backup.
7. Verify repository integrity.
8. Perform and document a restore test.
9. Identify critical production data.
10. Perform pre-rebuild production backups.
11. Automate backup configuration using Ansible.
12. Configure systemd backup timers.
13. Implement retention management.
14. Implement monitoring and alerting.
15. Evaluate backup immutability.
16. Establish periodic recovery testing.

## Acceptance Criteria

- [ ] Private European backup storage configured.
- [ ] Dedicated application credentials configured.
- [ ] Client-side encryption verified.
- [ ] Initial test backup completed.
- [ ] Repository integrity verified.
- [ ] Initial restore test completed.
- [ ] Production data successfully backed up.
- [ ] Production recovery procedure verified.
- [ ] Automated backup scheduling implemented.
- [ ] Retention policy implemented and validated.
- [ ] Monitoring and alerting operational.
- [ ] Backup deletion protection evaluated.
- [ ] Recovery documentation completed.

## Consequences

The selected architecture provides a cost-efficient
foundation for professional backup and recovery operations.

It introduces limited additional operational complexity
while supporting future platform growth.

Backup operations must be managed as an integral part
of the production platform lifecycle.

## Review Policy

Review this decision:

- Quarterly
- After a significant infrastructure change
- After a backup or recovery incident
- When recovery requirements change
- When storage costs increase significantly

## References

- https://restic.net/
- https://restic.readthedocs.io/
- https://www.backblaze.com/cloud-storage
- https://www.backblaze.com/docs/cloud-storage

## Revision History

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-11-24 | Initial Hetzner proposal |
| 1.1 | 2026-11-24 | Amazon S3 evaluated and selected |
| 1.2 | 2026-11-24 | Backblaze B2 selected to reduce cost and complexity |

---

Qman Enterprise Platform

Secure by design. Automated by default.

