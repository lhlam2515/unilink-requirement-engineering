# Non-Functional Requirement Templates by Domain

## Performance

```
REQ-PERF-001: The system SHALL respond to [action] within [X ms] at the [Nth percentile]
              under a load of [N concurrent users].
REQ-PERF-002: The system SHALL process [N transactions/events] per second
              without exceeding [X%] CPU and [X GB] memory.
REQ-PERF-003: Page load time SHALL not exceed [X seconds] on a [connection type] connection.
```

## Security

```
REQ-SEC-001: The system SHALL authenticate users via [mechanism, e.g., OAuth 2.0 / MFA]
             before granting access to any protected resource.
REQ-SEC-002: All data at rest SHALL be encrypted using AES-256 or equivalent.
REQ-SEC-003: All data in transit SHALL be transmitted over TLS 1.2 or higher.
REQ-SEC-004: The system SHALL lock an account after [N] consecutive failed login attempts
             for a minimum of [X minutes].
REQ-SEC-005: The system SHALL maintain an audit log of all [create/update/delete] operations,
             retaining logs for [N months/years].
REQ-SEC-006: The system SHALL pass OWASP Top 10 vulnerability assessment before production release.
```

## Reliability & Availability

```
REQ-REL-001: The system SHALL maintain [99.X%] uptime per calendar month
             (maximum [N minutes] unplanned downtime).
REQ-REL-002: Mean Time To Recovery (MTTR) SHALL not exceed [X hours] following a critical failure.
REQ-REL-003: The system SHALL recover from a crash without data loss for any committed transaction.
REQ-REL-004: Recovery Point Objective (RPO) SHALL be [X hours]; Recovery Time Objective (RTO)
             SHALL be [X hours].
```

## Usability

```
REQ-USE-001: [N%] of target users SHALL be able to complete [core task] without assistance
             on first use, as measured by usability testing.
REQ-USE-002: The system SHALL conform to WCAG 2.1 Level AA accessibility guidelines.
REQ-USE-003: All error messages SHALL include a plain-language description of the problem
             and a suggested corrective action.
REQ-USE-004: The system SHALL support [languages] and display all UI text in the user's
             preferred locale.
```

## Scalability

```
REQ-SCALE-001: The system SHALL support horizontal scaling to handle [N×] peak load
               without architectural changes.
REQ-SCALE-002: The system SHALL support [N] concurrent active users during peak periods
               with no degradation to the performance SLAs defined in REQ-PERF-001.
REQ-SCALE-003: Data storage SHALL scale to [N TB] without requiring schema migration.
```

## Maintainability

```
REQ-MAINT-001: Automated test coverage SHALL be ≥ [N%] for all business-critical modules.
REQ-MAINT-002: All public APIs and services SHALL be documented using [OpenAPI/Javadoc/etc.].
REQ-MAINT-003: The system SHALL produce structured logs (JSON) with correlation IDs
               for all cross-service operations.
REQ-MAINT-004: Deployment to production SHALL be achievable via automated CI/CD pipeline
               in ≤ [N minutes] with zero manual steps.
```

## Compliance (adapt to domain)

```
REQ-COMP-001 [GDPR]: The system SHALL provide a mechanism for users to request export
                     and deletion of their personal data within [72 hours].
REQ-COMP-002 [HIPAA]: All Protected Health Information (PHI) SHALL be stored and transmitted
                      in compliance with HIPAA Security Rule requirements.
REQ-COMP-003 [PCI-DSS]: Cardholder data SHALL never be stored post-authorization;
                         payment processing SHALL be delegated to a PCI-DSS Level 1 provider.
```
