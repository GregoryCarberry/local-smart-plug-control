# OPNsense: reusable local-only policy for smart plugs

**Verified on 21 November 2026:** four locally integrated plugs on an IoT VLAN kept working through Home Assistant after their manufacturers' apps reported them offline.

This document describes a **sanitised pattern**, not an importable firewall configuration. It deliberately omits real MAC addresses, IP ranges, DNS resolver addresses and credentials from my home network.

## Goal and scope

Allow Home Assistant (on a trusted network) to communicate with selected smart plugs on an IoT VLAN, while preventing those plugs from opening new connections to Internet destinations. Do **not** indiscriminately block every IoT device: some integrations still need cloud services.

In this lab:
- **TCP Smart Plug 1 and 2:** Home Assistant **LocalTuya** integration, using previously recovered per-device local keys.
- **Tapo P110 1 and 2:** built-in **TP-Link Smart Home** integration, with Third-Party Compatibility enabled.
- Devices have fixed DHCP reservations. All four are members of one OPNsense **Host(s)** alias, `IOT_LOCAL_ONLY`.
- The IPv4 block is applied on the **IoT** interface and named **Block Internet - Local-Only IoT Devices**.
- IoT interface **IPv6 Configuration Type** was verified as `None`. Audit IPv6 separately if enabled in another setup.

## Recommended implementation sequence

### 1. Verify local HA control *before* blocking

Confirm each plug works in its chosen Home Assistant integration and record its real IP/MAC mapping privately. Prefer DHCP reservations. Check that the trusted HA host can initiate connections to the IoT devices over the relevant interface firewall rules. Your specific allow policy may differ; do not assume a general trusted-to-IoT rule exists.

### 2. Create an alias

Go to **Firewall → Aliases** and add:

| Field | Value |
| --- | --- |
| Name | `IOT_LOCAL_ONLY` |
| Type | `Host(s)` |
| Content | IP addresses of the individually tested devices |
| Description | IoT devices managed locally without cloud access |

Keep the real addresses in **your firewall**, not this public repository. Initially I tested the two Tapo P110s, then expanded the same alias to both TCP plugs after the first stage passed.

### 3. Position an IPv4 block above the general Internet allow

Under **Firewall → Rules → IOT**, create or edit a rule:

| Setting | Example value |
| --- | --- |
| Action | **Block** |
| Interface / direction | **IOT / In** |
| TCP/IP version | **IPv4** |
| Protocol | **Any** |
| Source | `IOT_LOCAL_ONLY` |
| Source port | Any |
| Destination / port | Any / Any |
| Description | `Block Internet - Local-Only IoT Devices` |
| Logging | Enable temporarily to validate; reduce log noise afterwards if no longer needed |

My effective **top-down rule order** was:

1. Permit IoT-to-local-Pi-hole DNS (TCP/UDP 53).
2. Other narrowly scoped Chromecast/Roku exceptions.
3. Block IoT-initiated connections to private networks.
4. **Block source `IOT_LOCAL_ONLY` to any destination.**
5. Allow the remaining IoT devices to reach the Internet.

Because OPNsense is stateful, replies to legitimate **Home Assistant-initiated** connections can return even though the devices themselves cannot initiate new Internet sessions. The position and interaction with existing firewall and floating rules matter. Keep other devices' access unchanged.

**DNS nuance:** the existing *earlier* allow to Pi-hole means the blocked plugs can still resolve domains using the local resolver. This policy is **not** a complete ban on all local traffic or a proof that no DNS-derived information can ever travel externally through other services.

### 4. Clear existing states selectively

After Save → Apply, a plug may retain an already established cloud connection.

1. Visit **Firewall → Diagnostics → States**.
2. Search for **one device's address**.
3. Inspect and **kill only the matching states**.
4. Repeat on the next device after verifying the previous one.

Do **not** reset the entire firewall state table.

OPNsense documents both [alias and established-state interactions](https://docs.opnsense.org/manual/aliases.html) and [filtered state removal](https://docs.opnsense.org/manual/diagnostics_firewall.html).

### 5. Test each device and record the distinction

For each plug, check:
- Firewall Live View reports the plug's new attempted Internet connections **blocked**.
- Its manufacturer app reports it offline (and where available, HA's cloud diagnostic says `Disconnected`).
- Home Assistant still switches it on and off and updates the relevant sensors.
- Control also works through *remote access to Home Assistant* if that remote-access service is already securely configured.

**Actual results:** the Tapos' cloud diagnostics became disconnected after about one minute and about 3–4 minutes respectively. The Tapo app showed both offline. After the two TCP plugs were added to the alias and their established states cleared, the TCP app also showed both offline. All **four** plugs still worked through HA on the LAN and when connecting to HA over 4G/5G. LocalTuya remained operational; Tapo on/off and sensors remained operational.

One Tapo was also tested with known power levels before cloud isolation, approximately 700/1,000/1,700 W. No quantitative latency comparison or long-term power-history graph was completed.

### 6. Rollback

If a particular device stops functioning:
1. Remove its address from `IOT_LOCAL_ONLY` or temporarily disable **only** the associated block rule, then Apply.
2. Inspect and clear that device's states if required, then recheck the vendor app and HA.
3. Restore the rule after understanding why that device needs cloud access.

For future firmware updates, temporarily permitting vendor access may be necessary. Record any permanent exceptions separately rather than weakening the entire IoT isolation policy.

## Boundaries of the finding

- This was a real-world **local operation test without active vendor-cloud connections**, not a formal proof that no information could ever leave the network.
- The tested IoT interface had IPv6 disabled. With routed IPv6, additional rules could be necessary.
- No prolonged-offline, restart or power-loss soak test was undertaken.
- Disconnected vendor apps are a deliberate trade-off. Cloud-only functions, scenes or remote-app features may no longer work.
- Only add future devices to the alias after checking their own integration's local functionality and dependencies.

For device-specific setup, see [Tapo P110](tapo-p110.md) and [TCP Smart / LocalTuya](home-assistant.md).
