# Security: Model & Threat Analysis

Documentation of Kimi's security architecture, container isolation, and threat model.

## Files

- **[security-model.md](security-model.md)** - Container isolation strategy, open ports, CORS configuration, threat model analysis

## Key Security Aspects

### Container Isolation
- Kimi runs in isolated containers
- Processes have restricted system access
- Network exposure is limited to defined ports

### Port Exposure
- Kernel server: Port 8888
- Chrome DevTools: Port 9223
- Frontend: Port 5000 (configurable)

### Data Protection
- User uploads stored in isolated /mnt/kimi
- Agent workspaces isolated per environment
- No direct host filesystem access

## Threat Model

The security model considers:
- Container escape risks
- Network-based attacks
- Resource exhaustion
- Privilege escalation

## Related Documentation

- **Infrastructure**: See [../infrastructure/](../infrastructure/)
- **Runtime services**: See [../runtime/](../runtime/)
