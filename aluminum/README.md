# Aluminum

**Aluminum** is the Windows 11 virtual machine running Plex Media Server and its supporting automation stack. This setup provides seamless media acquisition, storage, and streaming in my homelab environment.

---

## 🏗️ System Overview

- **VM Name**: `aluminum`
- **Hypervisor**: Proxmox VE
- **OS**: Windows 11
- **Static IP**: `192.168.0.253`

---

## 🗂️ Media Directory Layout

Media is stored across two drives:

- `C:\media\movies`
- `C:\media\shows`
- `M:\media\movies`
- `M:\media\shows`

> `M:` is a dedicated HDD volume for additional storage.

---

## 🧰 Application Stack

- **Plex** – Streaming and transcoding engine
- **Prowlarr** – Index aggregator for torrent and Usenet sources
- **Radarr** – Automated movie acquisition
- **Sonarr** – Automated TV show acquisition

---

## 🔁 Media Flow

1. **Prowlarr** indexes torrent sites and provides results to Radarr/Sonarr.
2. **Radarr/Sonarr** manually search for content based on requests.
3. Media is downloaded and placed into the appropriate `C:\media` or `M:\media` directories.
4. **Plex** scans the directories and updates the libraries.

---

## ⚙️ Plex Configuration

- **Scan Paths**: `C:\media`, `M:\media`
- **Transcoding**: High CPU usage allowed (optimized for local hardware)
- **Remote Access**: Enabled for authenticated users

---

## 🤖 Discord Bot Integration

- The Discord bot running in the `magnesium` container has API access to **Radarr** and **Sonarr**.
- Users in the Discord server can request to add movies (Radarr) and TV shows (Sonarr).
- These requests go through an **approval workflow** before being processed.

---

## 👤 User Access

- Two Plex accounts:
  - `Aaron` (primary account)
  - `Other` (guest/secondary access)
- Automation tasks like adding media are restricted to trusted users

---