# GCP Networking & Firewall

## Info to Collect
- **Rule name**: Descriptive name for the firewall rule
- **Direction**: Ingress (incoming) or Egress (outgoing)
- **Action**: Allow or Deny
- **Targets**: Which instances the rule applies to (all, specific tags, specific SA)
- **Source/Destination**: IP ranges, tags, or service accounts
- **Protocols and ports**: e.g., TCP:80,443 or UDP:53
- **Priority**: Lower number = higher priority (default 1000)
- **Network**: Which VPC network (default is "default")

## Prerequisites
- API enabled: `compute.googleapis.com`
- User needs `compute.firewalls.create` permission

## Console Steps

### Creating a Firewall Rule
1. Navigate to `https://console.cloud.google.com/networking/firewalls/add?project=PROJECT_ID`
2. wait_for "Create a firewall rule" page
3. take_snapshot to find form elements
4. fill the rule name
5. Select the network
6. Set priority if not default
7. Select direction (Ingress/Egress)
8. Select action (Allow/Deny)

### Target Configuration
9. Select target type:
   - "All instances in the network"
   - "Specified target tags" → fill the tag
   - "Specified service account" → fill the SA email

### Source/Destination
10. For Ingress rules: set source IP ranges (e.g., `0.0.0.0/0` for anywhere, or specific CIDR)
11. For Egress rules: set destination IP ranges

### Protocols and Ports
12. Select "Specified protocols and ports"
13. Check TCP and/or UDP
14. fill the port numbers (e.g., `80,443` or `8080-8090`)

### Create
15. Click "Create"
16. wait_for the rule to appear in the firewall rules list
17. take_screenshot to confirm

## Common Firewall Patterns

### Allow HTTP/HTTPS
- Direction: Ingress, Allow
- Source: `0.0.0.0/0`
- Protocols: TCP:80,443
- Target: tag `http-server`

### Allow SSH
- Direction: Ingress, Allow
- Source: `0.0.0.0/0` (or restrict to your IP)
- Protocols: TCP:22
- Target: tag `ssh-server`

### Allow Internal Communication
- Direction: Ingress, Allow
- Source: `10.128.0.0/9` (default VPC range)
- Protocols: all
- Target: all instances

## Common Issues
- **Rule order matters**: Lower priority number wins. Check for conflicting rules.
- **Default deny**: GCP has implicit deny-all ingress and allow-all egress by default
- **Tags must match**: Instance must have the target tag for the rule to apply
- **Propagation**: Rules take effect within seconds but occasionally take up to a minute
