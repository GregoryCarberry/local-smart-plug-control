# Tapo P110: Home Assistant integration and cloud-independent control

**Verified:** 21 November 2026, using two UK Tapo P110 plugs. This documents the sequence I followed on my own equipment; firmware, app menus and model revisions can differ.

Unlike the older [TCP Smart devices](tcp-case-study.md), I did not need to extract local encryption keys or write scripts. Most of the Home Assistant configuration took place through the web interface, after initial setup in the Tapo phone app.

## What I verified

- Both P110s were provisioned through the official Tapo app on an isolated IoT Wi-Fi network and assigned reserved IPv4 addresses.
- Both were added individually through Home Assistant's **built-in TP-Link Smart Home** integration, manually by IP address.
- Switching, current, voltage, instantaneous power and energy sensor entities appeared in Home Assistant.
- With one P110 powering a radiator for a supervised measurement test, the HA instantaneous readings roughly matched three known heater levels: **700 W, 1,000 W and 1,700 W**. This checks plausible live watts, **not** accumulated kWh accuracy. I did not record which of the two plugs was used for that load test.
- After an OPNsense rule blocked their outward IPv4 connections, Home Assistant showed **Cloud connection: Disconnected**, and the Tapo app reported both devices offline. **Switching and sensors still worked through Home Assistant.**
- The first plug's cloud status changed after roughly a minute; the second took approximately 3–4 minutes. These are observations, not guaranteed detection timings.

## Prerequisites

- A working Home Assistant instance and two supported Tapo P110 plugs.
- The Tapo app for first-time provisioning and enabling third-party integration.
- A network route from the Home Assistant host to the plugs. My Home Assistant server and plugs sit on different VLANs; no specific network address or Wi-Fi SSID is required by the method.
- Fixed/reserved device addresses, so a manual Home Assistant entry does not become stale after DHCP renewal.
- The appropriate TP-Link account credentials available privately when Home Assistant asks for them. **Do not put credentials in the repository or command output.**

The Home Assistant [TP-Link Smart Home integration documentation](https://www.home-assistant.io/integrations/tplink/) lists the P110 and describes its local connection behaviour and compatibility requirements.

## 1. Provision the plugs and reserve their addresses

1. Pair each plug using the official Tapo mobile app and join it to the intended IoT Wi-Fi network.
2. Check each device's identity in your DHCP server (or in the Tapo app) and reserve a stable IP address for its MAC address. Verify the currently assigned address actually matches its reservation.
3. Identify devices by **MAC address and IP**, rather than assuming physical sticker numbers, app nicknames and setup order all agree. I initially ran into a device-label mix-up while doing exactly this.
4. Confirm each plug switches normally in the Tapo app before adding firewall restrictions.

In this lab, OPNsense provides the DHCP reservations. Avoid publishing actual MAC addresses, SSIDs or device IDs in reusable examples.

## 2. Enable third-party compatibility

Open the official **Tapo app** and enable **Third-Party Compatibility**. In current versions the setting may be under **Tapo Lab**, **Me / Third-Party Services**, or a similarly named location.

This was explicitly enabled for my P110s. Some firmware revisions require it before Home Assistant can authenticate locally. The feature can have security implications, so use a trusted Home Assistant installation and keep the plug firmware and account credentials managed appropriately.

See [TP-Link's explanation](https://www.tp-link.com/us/support/faq/4416/) and [Home Assistant's integration notes](https://www.home-assistant.io/integrations/tplink/).

## 3. Add a P110 through Home Assistant

1. Open Home Assistant in a browser and go to **Settings → Devices & services**.
2. Add the **TP-Link Smart Home** integration if it is not already present.
3. If a plug is not discovered across VLANs, choose the integration's **manual add / host-IP** option and enter its reserved IP.
4. Provide the TP-Link account credentials **only within Home Assistant's private configuration flow**, if prompted.
5. Repeat for the second plug using its own address. Check each device's reported MAC address against the DHCP reservation.
6. Test on/off switching, then inspect available entities for current (A), voltage (V), instantaneous power (W), daily consumption (kWh) and monthly consumption (kWh).

The availability and labels of sensors are firmware/model-dependent. In this lab, both P110s exposed the above measurements. A displayed `0 W` with no load attached is not evidence that power monitoring works: test with an appropriate, attended load.

## 4. Restrict Internet access without blocking Home Assistant

I used an **OPNsense IoT interface firewall rule** targeting a reusable host alias rather than a rule for each plug brand. The rule blocks the selected devices' new outbound IPv4 Internet connections, while Home Assistant can still initiate LAN connections to them over the routed VLANs.

The two Tapos were tested first; later I added both TCP Smart plugs. The practical procedure, rule order, existing-state handling, rollback and limitations are in **[OPNsense local-only IoT policy](local-only-firewall.md)**.

**Do this only once the plugs are functioning through Home Assistant.** Blocking the whole IoT VLAN or its access to the HA server is not equivalent to restricting the plugs' Internet access.

## 5. Verify the outcome

After applying the source-alias firewall restriction:

1. Use OPNsense's live firewall view to confirm the plug's outbound attempts are **blocked**. Observe whether permitted internal DNS traffic is being confused with external access.
2. Clear **only that plug's existing firewall states** if the old cloud connection survives the change. Do not flush the whole router state table.
3. Wait for Home Assistant's **Cloud connection** status to become **Disconnected**. The Tapo mobile app should show the plug offline.
4. With a small, safe test load or no load, switch it on and off from Home Assistant and check its sensor values. Repeat for the second P110.
5. Check local HA access and, if already configured securely, remote access *to Home Assistant* from a separate network. Remote HA access does not mean the P110 has its own cloud connection.

**Observed result:** both Tapo apps offline; both devices still switched and reported sensors in HA. This supports cloud-independent local control. I did not perform a formal audit of every possible egress path.

## Known limitations and sensible precautions

- At this test point the IoT interface's **IPv6 Configuration Type was None**. An IPv4 block alone may be insufficient on another network with routed IPv6.
- An earlier IoT DNS rule still allowed queries to my local Pi-hole. That does not by itself show that Internet access was restored, but zero external information exchange was not proven.
- Firmware updates and functions in the vendor app may need the Internet block lifted temporarily.
- I did **not** test operation across a prolonged outage, repeated reboots, or all firmware revisions.
- A smart plug controlling a heater has additional fire and unattended-operation risks. The radiator test above was a measurement exercise; use a small non-heating load for routine switching demonstrations and follow the appliance manufacturer's instructions.

## Later enhancement

Plot live watts and accumulated kWh in Home Assistant using a History/Statistics card once the sensors' recording and long-term statistics support have been checked. **No graph was created during this project phase.**
