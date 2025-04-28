# Tailscale + WSL + Docker Compose: Detailed Setup Guide

This guide explains how to expose a web service running in Docker Compose inside WSL (Windows Subsystem for Linux) to your Tailscale tailnet using Tailscale Serve. It covers all steps, troubleshooting, and best practices.

---

## Prerequisites

- **WSL2** (recommended)
- **Docker Compose** running your backend (e.g., on port 3000)
- **Tailscale** installed on both Windows and WSL
- Your service is reachable at `localhost:<PORT>` inside WSL

---

## 1. Install Tailscale in WSL

```sh
curl -fsSL https://tailscale.com/install.sh | sh
```

---

## 2. Start Tailscale as Root in WSL

Tailscale Serve requires `tailscaled` to run as root to proxy privileged ports (like 443).

```sh
# Clean up any old state (optional, but recommended if you had issues)
sudo tailscaled --cleanup

# Start tailscaled in the background
sudo tailscaled &

# Bring up your WSL node in the tailnet
sudo tailscale up
```

Authenticate as instructed (usually via a browser link).

---

## 3. Confirm Tailscale Node in WSL

Check your node appears in your tailnet:

```sh
tailscale status
```
You should see your WSL node (e.g., `thor-1`) with an IP like `100.x.x.x`.

---

## 4. Start Your Docker Compose Service

From your project directory:

```sh
docker compose up -d
```

Check the service is running and healthy:

```sh
docker compose ps
```
Look for something like `0.0.0.0:3000->3000/tcp`.

---

## 5. Confirm Your Service is Reachable in WSL

```sh
curl -I http://localhost:3000
```
You should see `HTTP/1.1 200 OK` or similar.

---

## 6. Expose the Service with Tailscale Serve

**With Tailscale v1.36+ the syntax is:**

```sh
sudo tailscale serve --bg localhost:3000
```

This will proxy all HTTPS traffic from your tailnet to your service at `localhost:3000`.

---

## 7. Check Serve Status

```sh
sudo tailscale serve status
```
You should see:

```
https://<your-node>.ts.net (tailnet only)
|-- / proxy http://localhost:3000
```

---

## 8. Test from Another Tailscale Device

On another device in your tailnet (e.g., your MacBook Pro):

```sh
curl -v https://<your-node>.ts.net/
```
Or open the URL in your browser.

---

## 9. Troubleshooting

### a. Connection Refused or Timeout
- Make sure `tailscaled` is running as root in WSL.
- Make sure your service is listening on the correct port and bound to `localhost`.
- Disable any firewall in WSL: `sudo ufw disable`
- Restart `tailscaled` if needed: `sudo tailscaled --cleanup && sudo tailscaled &`
- Reapply `tailscale serve` as above.

### b. DNS Issues
- Confirm DNS resolves: `nslookup <your-node>.ts.net`
- If DNS fails, use the Tailscale IP directly for testing.

### c. Tailscale ACLs
- Ensure your tailnet ACLs allow access between devices: https://login.tailscale.com/admin/acls

### d. WSL Networking
- WSL2 is required for full networking support.
- If issues persist, try restarting WSL and/or your Windows host.

---

## 10. Updating Environment Variables

If you expose your service via a different Tailscale node, update environment variables (e.g., `.env`) to use the correct URL:

```
SERVER_URL=https://<your-node>.ts.net
```

---

## 11. References
- [Tailscale Serve Documentation](https://tailscale.com/kb/1242/tailscale-serve)
- [Tailscale WSL Guide](https://tailscale.com/kb/1153/wsl/)
- [Tailscale Funnel (for public access)](https://tailscale.com/kb/1223/funnel/)

---

## Example: Full Command Sequence

```sh
# 1. Install Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

# 2. Start tailscaled as root
sudo tailscaled &

# 3. Authenticate
sudo tailscale up

# 4. Start your service
cd /path/to/your/project
docker compose up -d

# 5. Confirm service is up
curl -I http://localhost:3000

# 6. Expose with Tailscale Serve
sudo tailscale serve --bg localhost:3000

# 7. Check status
sudo tailscale serve status

# 8. Test from another device
curl -v https://<your-node>.ts.net/
```

---

## Notes
- The `--bg` flag keeps `tailscale serve` running in the background.
- For public access, use Tailscale Funnel instead of Serve.
- Always run `tailscaled` as root for privileged ports.

---

This guide should help you or any team member reproduce this setup on any WSL + Docker Compose environment with Tailscale. If you encounter new errors, consult the Tailscale docs or reach out for support.
