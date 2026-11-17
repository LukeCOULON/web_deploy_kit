# Web Deployer

A standalone toolkit that can be copied into the root directory of a Python project.

The application continues running on the user's PC: **Web Deploy Kit** provides a local service and does not deploy the application to a remote hosting provider.

---

## Installation

Copy the `web_deploy/` folder and the `deploy.py` file into the root directory of your project.

Run:

```bash
python deploy.py
```

At startup, choose between the **Simple** and **Advanced** versions. Both interfaces allow you to select the Python file that starts your server.

* The **Simple** version provides the essential server, port, proxy, and virtual environment settings.
* The **Advanced** version additionally includes HTTPS, Cloudflare Tunnel, access controls, and network diagnostics.

In the Advanced version, you can uncheck **Public Proxy (Caddy)** to run the application in local-only mode.

Once the server is running, the **ENABLE PUBLIC PROXY** button allows you to start Caddy without stopping the server. The toolkit asks for confirmation before exposing the application to the public Internet.

### Automatic Installation via Git

**Windows (CMD):**
```cmd
git clone https://github.com/LukeCOULON/web_deploy_kit.git "%TEMP%\web_deploy_kit" && xcopy "%TEMP%\web_deploy_kit\*" "." /E /I /Y && rmdir /S /Q "%TEMP%\web_deploy_kit" && python deploy.py
```

**Linux (Terminal):**
```bash
git clone https://github.com/LukeCOULON/web_deploy_kit.git /tmp/web_deploy_kit && \
cp -rn /tmp/web_deploy_kit/. . && \
rm -rf /tmp/web_deploy_kit && \
python3 deploy.py
```

> **Important:** The toolkit files are copied directly into the current directory without creating a subfolder. No files with matching names must already exist. Therefore, the target directory should be empty.

### Uninstallation

**Windows (CMD):**
```cmd
rmdir /S /Q web_deploy 2>nul & del /Q deploy.py web_deploy_config.json 2>nul & rmdir /S /Q .venv 2>nul
```

**Linux (Terminal):**
```bash
rm -rf web_deploy deploy.py web_deploy_config.json .venv
```

> **Warning:** These commands also delete `.venv`, which may contain dependencies specific to your project. Check its contents and back it up if necessary before running the command. The commands do not delete logs stored in the user's home directory.

---

## Example

If your project has the following structure:

```text
my_project/
├── app/
│   └── server.py
├── web_deploy/
└── deploy.py
```

Select:
```text
app/server.py
```

The server must listen on the configured address and port.

Web Deploy Kit uses **Caddy v2** as a reverse proxy for HTTP and HTTPS:
```text
0.0.0.0:8080 -> local application 127.0.0.1:8000
```

Install Caddy separately before deployment. The toolkit checks its version and validates its configuration before starting it.

Caddy proxies requests directly to the application and handles WebSocket upgrades, streaming responses, and long-running requests.

The toolkit does not download or update the Caddy binary. The application can remain bound to `127.0.0.1`.

---

## Currently Available Features

Web Deploy Kit currently covers the stabilization phase and V4 UX/network functionality:

* Local public proxy using Caddy v2 for HTTP and HTTPS.
* Strict validation of the Caddy binary, Caddyfile, and ports before startup.
* Optional Basic Authentication, IP/CIDR allowlisting, rate limiting, and daily upload quotas.
* Explicitly disabling and restoring public access without stopping the local server.
* Python framework detection (Flask, FastAPI, Django, and common commands).
* Server and proxy monitoring with configurable health checks.
* Local network diagnostics: public IP, CGNAT, double NAT, DNS, firewall, and NAT forwarding.
* Optional Cloudflare Quick Tunnel for situations where the router or network does not allow direct public access.
* Status dashboard displaying the server, proxy, tunnel, and public exposure status.

*Features not yet delivered or fully formalized include multi-application management and complete automated installation/uninstallation packaging.*

---

## Virtual Environment and Dependencies

During deployment, the **Create/use a venv and automatically install dependencies** option is enabled by default.

The toolkit:
1. Creates `.venv/` in the project root if it does not already exist.
2. Updates `pip` inside this virtual environment.
3. Installs dependencies from `requirements.txt`, if available.
4. Otherwise, if `pyproject.toml` or `setup.py` is present, installs the project using `pip install .`.
5. Starts the server using the Python interpreter from the virtual environment rather than the system Python.

To disable this behavior, uncheck the corresponding option in the interface.

---

## Important Network & Exposure Modes

The toolkit does not automatically configure every Internet router or gateway. To make your website accessible over the Internet, configure port forwarding (NAT) on your router to redirect incoming traffic to the computer running the toolkit.

**Example:**
```text
WAN Port 8080 -> Local PC IP:8080
```

A dynamic public IP address may change over time. For a proper public-facing service, a domain name, HTTPS, and an appropriate dynamic DNS solution are recommended.

The toolkit clearly distinguishes between **three exposure modes**:

1. **Local Caddy Proxy:** The service is published through the local machine, with or without HTTPS.
2. **Cloudflare Quick Tunnel:** Public traffic is relayed through Cloudflare without opening a port on the router.
3. **No Public Access:** The server remains local and the application is not exposed.

The tunnel does not guarantee unlimited Internet access or a permanent public service. It provides a temporary URL and depends on an outbound network connection to Cloudflare being permitted. The toolkit displays this mode accurately and never claims that external access is available without evidence from an external network.

---

## Optional Cloudflare Quick Tunnel

When the network is behind CGNAT, double NAT, or a router that blocks public ports, the project can use an outbound Cloudflare tunnel.

**The tunnel mode:**
* Requires the `cloudflared` binary to be installed or configured through `WEB_DEPLOY_CLOUDFLARED`.
* Starts a Quick Tunnel pointing to `http://127.0.0.1:<local_port>`.
* Keeps the local Caddy proxy listening on loopback only, preventing direct exposure without the relay.
* Publishes a URL in the format `https://<name>.trycloudflare.com`.
* Stops before the local proxy and does not retain the URL after an explicit shutdown.

This mode is useful for testing a public-facing service without configuring NAT. However, it does not replace a production deployment with dedicated DNS, permanent certificates, and a complete security policy.

---

## Automatic HTTPS

The public proxy requires **Caddy v2** to be installed on the PC.

**Automatic HTTPS also requires:**
* A public domain name.
* DNS records pointing to the network's public IP address.
* TCP ports `80` and `443` accessible from the Internet.

In HTTPS mode, the public port is `443`. Configure your router to forward:
```text
WAN Port 80  -> PC Port 80
WAN Port 443 -> PC Port 443
```

Caddy automatically obtains and renews the certificate. The toolkit does not download the Caddy binary.
* **Official installation documentation:** [https://caddyserver.com/docs/install](https://caddyserver.com/docs/install)

**Certificate issuance may fail if:**
* The domain does not resolve to the correct network.
* The Internet provider uses CGNAT.
* The firewall or router blocks port `80` or `443`.

On Linux, Caddy must also have permission to listen on these ports.

---

## Security

Opening a port may make the application and its features accessible to anyone on the Internet. **Do not expose applications containing sensitive data or applications designed exclusively for local use.**

Confirmation is required before deploying with the public proxy enabled and before activating the proxy after a local-only deployment.

### Network Access Controls
Under **Network Access**, enable **Filter by IP/CIDR** to apply an allowlist of IP addresses or ranges separated by commas.
* An empty list is rejected at startup.
* The access decision uses the peer address forwarded by Caddy, not a direct client-supplied header.
* Rejected addresses receive a `403` response.

### Rate Limiting
**Limit requests/minute/client** is enabled by default at `120` requests per minute.
* Uncheck it to disable the limit.
* Requests exceeding the limit receive a `429` response with a `Retry-After` header.

### Daily Upload Quota
**Daily Upload Quota (MiB)** is disabled by default.
* When enabled, the toolkit adds up the request body sizes announced through `Content-Length` and reserves that amount before transferring the request.
* The counter is stored in: `~/.web_deploy_kit/security_usage.sqlite3`
* It resets at midnight UTC.
* It does not count downloaded responses or headers. A request interrupted after its quota reservation remains counted.
* Uploads without a declared content length (for example, chunked uploads) receive a `411` response when this quota is enabled.

> These security controls only apply when **Public Proxy (Caddy)** is active. They do not replace a firewall and do not protect the application's internal functions.

### HTTPS and Basic Authentication
Without Caddy HTTPS mode, traffic is not encrypted. Do not send sensitive information through the HTTP proxy.
* If **Basic Authentication** is enabled, the toolkit requests the username and password for each deployment.
* Credentials are stored only in memory and are never written to `web_deploy_config.json`.
* The proxy removes the Basic Authentication header before forwarding the request to the application.

### Public Access Controls
The public access button allows you to stop Caddy without stopping the server and subsequently restore access using the same security policy. Depending on its state, the button displays:
* `ENABLE PUBLIC PROXY`
* `DISABLE PUBLIC ACCESS`
* `RESTORE PUBLIC ACCESS`

---

## Current Limitations

* The Python server runs as a child process.
* Caddy v2 must be installed and updated separately. The toolkit does not manage binary downloads.
* Automatic HTTPS requires a domain name and accessible TCP ports `80`/`443`.
* Cloudflare Tunnel is optional and temporary. It does not replace a stable public domain or a permanent security policy.
* Caddy handles WebSocket, streaming, and long-running connections, but these paths could not be tested with the actual binary here.
* Basic Authentication is only available with HTTPS or the Cloudflare HTTPS tunnel.
* The IP allowlist, rate limiter, and daily quota protect the gateway, not the application's internal functions.
* Automatic router configuration is not universal and depends on the router model.
* The selected port must be available and permitted by the firewall.
* The toolkit does not independently verify from the Internet whether the firewall or NAT rule allows external access.
* Multi-application management has not yet been implemented in the active code. Each deployment still uses a single profile, even though the interface and diagnostics are already prepared for future expansion.

---

## Built-in Tutorial

After selecting a version, the interface displays a guide before configuration. The guides explain how to prepare port forwarding on the router. The Advanced version also covers network checks before startup.

**The guide explains:**
1. Where to find the router's NAT / Port Forwarding settings.
2. The detected local IP address of the machine.
3. Which TCP protocol to select.
4. The server's internal port.
5. The external port to open.
6. Why reserving the local IP address through DHCP is recommended.

**Example rule:**
```text
Internal IP    : 192.168.1.42
Protocol       : TCP
External Port  : 8080
Internal Port  : 8000
```

Once you have read the guide, click **I UNDERSTAND — CONTINUE** to open the main interface.

---

## Public IP Address

The interface displays the detected public IP address at the top, along with a clickable link in the following format:
```text
http://PUBLIC_IP:PUBLIC_PORT
```

* The link's port is synchronized in real time with the **Access Port** field.
* The public IP address is refreshed periodically.

**For the link to work over the Internet:**
* The router must forward the public port to the PC.
* The Windows/Linux firewall must allow that port.

*Note: Some Internet providers use CGNAT. In this case, simple port forwarding is not sufficient.*

---

## Automatic Diagnostics

The log translates several common errors into readable messages with short suggested solutions:

* **PORT ALREADY IN USE** — Close the program using the port or select another port.
* **PERMISSION DENIED** — Use a port above 1024 and check permissions.
* **SERVER UNREACHABLE** — Check that the application is listening on the configured server port.
* **FILE NOT FOUND** — Check the project path or the relevant file.
* **MISSING PYTHON DEPENDENCY** — Install the dependency in the virtual environment or add it to `requirements.txt`.
* **INCOMPATIBLE PYTHON DEPENDENCY** — Check the Python version and package compatibility.
* **TIMEOUT** — Check the application startup or increase the timeout.
* **INVALID NETWORK ADDRESS** — Check the listening address used by the server or proxy.
* **INVALID CONFIGURATION** — Check or recreate `web_deploy_config.json`.

---

## Non-Destructive Port Checks

Before startup, Web Deployer checks whether the selected ports are available.
* If a port is already in use, startup is blocked and the log displays the PID and process name whenever the operating system can provide them.
* The toolkit never terminates an external process to free a port. Choose another port or manually stop the program using it.
* During shutdown, the toolkit only manages the server and proxy processes it started itself.
* If normal server shutdown fails, the interface requests confirmation before forcing termination. Without confirmation, the process remains managed and its state is preserved.

---

## Monitoring and Logs

After deployment, the toolkit periodically checks the HTTP response of both the server and the proxy.
* After two failed checks, public access is stopped.
* Automatic restarting is disabled by default. If enabled in the interface, the number of attempts and the checking interval can be configured.
* The **STOP** button closes the proxy first, followed by the server.
* Closing the interface also triggers a clean shutdown of the services.

### Log Storage
Logs are stored in:
```text
~/.web_deploy_kit/logs/web-deployer.log
```

**The log system uses:**
* Log rotation at 2 MiB.
* Retention of five log files.
* Server output included in the logs.
* Private user-only file permissions for the current log file on Linux and macOS.

*Logs may contain information written by the application itself. Review them before sharing.*

---

## Server Detection and Health Checks

The **Detect** button statically analyzes Python files and dependencies to suggest:
* Flask, FastAPI, or Django.
* An entry-point file.
* A port.
* A startup command.

The project code is neither imported nor executed during detection. Review the suggestions before starting the server. They remain editable, and manual selection is available if the framework is unknown.

### Health Check
Under **Health Check**, configure:
* HTTP path
* Expected response code
* Timeout

**Default values:**
```text
Method  : GET
Path    : /
Status  : 200
Timeout : 3 seconds
```

* Periodic checks use the interval configured in the options.
* The local server is checked separately from the Caddy proxy.
* Caddy also performs active upstream health checks.
* Proxy authentication responses (`401`/`403`) are not interpreted as application failures.

---

## Tests

Run the local tests using Python's standard library:

```bash
python3 -m unittest discover -s tests -v
```

---

## Local Port Database

Web Deployer includes a small local SQLite database:
```text
web_deploy/data/ports.db
```

**It contains:**
* Known sensitive ports or ports blocked by certain browsers (`browser_unsafe`).
* Ports frequently reserved for network services and potentially filtered by certain routers or gateways (`router_reserved`).
* Common ports suitable for web services (`recommended`).

The database helps prevent errors before deployment. It does not claim to know the specific rules of every router or Internet provider. Public port filtering depends on the equipment and network.

* The public port is checked before startup.
* A port classified as `browser_unsafe` blocks deployment and suggests alternative ports.
* The interface also allows you to manually check the public port.
