# Palo Alto Perimeter Security Lab

[Read the Full Lab Notes (PDF)](./palo_alto_perimeter_security_deployment.pdf)

## Overview
These lab notes document the configuration, troubleshooting, and validation of a segmented enterprise perimeter using a Palo Alto Networks firewall. Starting from a baseline image, the deployment establishes Layer 3 security zones, virtual routing, dynamic Source NAT, App-ID security policies, and unified threat inspection across internal, extranet, and external networks.

## Network Architecture and Routing
- **External WAN (`ethernet1/1` | `Internet` zone):** Configured at `203.0.113.20/24` for outbound internet connectivity.
- **Internal Network (`ethernet1/2` | `Users` zone):** Configured at `192.168.1.1/24` with an `Allow-Ping` management profile for local diagnostics.
- **Extranet Segment (`ethernet1/3` | `Extranet` zone):** Configured at `192.168.50.1/24` to isolate partner servers.
- **Virtual Routing (`VR-1`):** Binds all three Layer 3 interfaces with a static `Default-Route` directing outbound traffic toward the internet gateway.
- **Source NAT (`Users_to_Internet`, `Extranet_to_Internet`):** Applies dynamic IP and port translation mapped to `ethernet1/1` (`203.0.113.20/24`) so private internal subnets reach external resources without exposing internal addressing.

## Security Policies and Threat Inspection
- **Audited Default Deny:** Overrode `interzone-default` to enable `Log at Session End` so unauthorized cross-zone traffic is recorded for monitoring.
- **URL Filtering Policy (`Block_Bad_URLs`):** Explicitly blocks outbound connections to hacking, phishing, malware, and unknown threat categories.
- **App-ID Policies (`Users_to_Extranet`, `Users_to_Internet`, `Extranet_to_Internet`):** Restricts traffic strictly to approved applications (`dns`, `ping`, `ssh`, `ssl`, and `web-browsing`) by zone pair using color-coded administrative tags.
- **Unified Profile Group (`Corp-Security-Group`):** Consolidates `Corp-AV` (Antivirus), `Corp-AS` (Anti-Spyware), `Corp-Vuln` (Vulnerability Protection), `Corp-URL` (URL Filtering), `Corp-FB` (File Blocking), and `Corp-WF` (WildFire) across every active allow rule for simultaneous deep packet inspection.

## Verification and Testing
- **Routing and Gateway Reachability:** Verified zero percent packet loss from internal hosts to `192.168.1.1` and across source-bound CLI pings from `203.0.113.20` and `192.168.1.1`.
- **Outbound Internet and DNS:** Confirmed successful client pings to `8.8.8.8` and `google.com`, plus encrypted HTTPS web browsing to `paloaltonetworks.com`.
- **Extranet Access:** Validated HTTP connectivity to the internal extranet server (`192.168.50.80`) and SSH access plus outbound routing from `192.168.50.150`.
- **Negative Security Controls:** Confirmed immediate connection reset (`ERR_CONNECTION_RESET`) when attempting to browse to blocked domain `hacker9.com`.
