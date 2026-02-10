# 🏠 Omada + Matter/Thread Reference

## 📂 1. Essential Folder Setup

You must create these directories on your host machine to match your `docker-compose.yaml`. Without these, your Matter pairing keys and Thread network data will be lost every time you restart.

### Create the Structure

Run this once on your host:

```bash
mkdir -p ~/homeassist/config \
         ~/homeassist/mosquitto/config \
         ~/homeassist/mosquitto/data \
         ~/homeassist/mosquitto/log \
         ~/homeassist/otbr \
         ~/homeassist/matter

```

### Prep the Mosquitto Config

The `mosquitto` container will fail if the config file doesn't exist. Create it now:

```bash
cat <<EOF > ~/homeassist/mosquitto/config/mosquitto.conf
persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log
listener 1883
allow_anonymous true
EOF

```

---

## 🌐 2. Omada Controller: Detailed Config

Omada's "Advanced" features are often aggressive. Here is exactly what to toggle to stop it from interfering with Matter's IPv6 traffic.

### Site Settings (mDNS)

* **Path:** `Settings > Services > mDNS`
* **Toggle:** **Enable mDNS**
* **Services:** Select **All** (Matter uses `_matter._tcp.local` and `_matterc._udp.local`).(Create a custom service if you want to be specific).

### Wired Network (MLD Snooping)

* **Path:** `Settings > Wired Networks > LAN > [Your Network] > Edit`
* **MLD Snooping:** **Enabled**. (This is like IGMP Snooping but for IPv6—mandatory for Thread).
* **Unknown Multicast:** **Forward**.
* *Warning:* If set to **Discard**, your switch will drop the pairing request before HA even sees the device.


* **MLD Querier:** **Enabled**.

### Wireless Settings

* **Path:** `Settings > Wireless Networks > [Your SSID] > Advanced Settings`
* **Multicast-to-Unicast:** **Disabled**. (This feature "optimizes" traffic by converting it, but it often breaks the Matter handshake).
* **Multicast Filtering:** **Disabled**. (Check your WLAN's "Multicast/Broadcast Management" and ensure no rules are blocking the IPv6 multicast range `ff02::/16`).

---

## ⚙️ 3. Host System Setup (`sysctl`)

Matter over Thread requires your host to act as a router for IPv6 traffic.

**Add these to the bottom of `/etc/sysctl.conf`:**

```text
net.ipv6.conf.all.disable_ipv6=0
net.ipv4.conf.all.forwarding=1
net.ipv6.conf.all.forwarding=1
net.ipv6.conf.all.accept_ra_rt_info_max_plen=64
net.ipv6.conf.all.accept_ra=2

```

Apply with: `sudo sysctl -p`