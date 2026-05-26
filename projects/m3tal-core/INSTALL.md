# M3TAL Core Installation Guide

Follow these steps to deploy the M3TAL engine and setup the core components.

---

## 1. Prerequisites
M3TAL orchestrates and routes containerized services on your host. You **must** have these packages installed:
* **Docker Engine** (v20.10+)
* **Docker Compose V2** (runnable as `docker compose`)

Verify that the requirements are satisfied:
```bash
docker --version && docker compose version
```

---

## 2. APT Installation
We distribute the M3TAL system package through an APT repository. Add the repository keys and install the CLI/Daemon system services:

```bash
# 1. Download and trust the repository GPG key
curl -fsSL https://jakej985-rgb.github.io/m3tal-apt-key/public.key | sudo gpg --dearmor -o /usr/share/keyrings/m3tal-archive-keyring.gpg

# 2. Add the M3TAL APT repository to package sources
echo "deb [signed-by=/usr/share/keyrings/m3tal-archive-keyring.gpg] https://jakej985-rgb.github.io/m3tal-apt-key stable main" | sudo tee /etc/apt/sources.list.d/m3tal.list

# 3. Update repositories and install the package
sudo apt update && sudo apt install -y m3tal
```

---

## 3. Configuration Setup
Run the setup wizard to verify and write system configurations to `/etc/m3tal/.env`:

```bash
sudo m3tal config wizard
```
This wizard configures:
* Port bindings (`HTTP_PORT`, `DASHBOARD_PORT`)
* Expose modes (direct port `local` mode vs. reverse-proxied `traefik` mode)
* Local domain mappings (`DOMAIN`)
* State paths and credentials.

---

## 4. Launching Services
Run the system stacks and start the dashboard console:

### Launch All Stacks
Start all core stacks (including the Traefik router/gateway) defined in the `/docker/` directory:
```bash
m3tal up
```

### Start the Dashboard
Fetch the latest container image and start the Dashboard interface:
```bash
m3tal dash up
```

---

## 5. Dashboard Authentication
Access the Dashboard at the URL corresponding to your configuration:
* **Local Mode** (`DASHBOARD_EXPOSE_MODE=local`): `http://YOUR_SERVER_IP:8082`
* **Traefik Mode** (`DASHBOARD_EXPOSE_MODE=traefik`): `http://dash.YOUR_DOMAIN`

Login using these default credentials:
* **Username**: `admin`
* **Password**: The password you entered during `m3tal config wizard`.

To update the admin password later:
```bash
sudo m3tal dashpass
```
