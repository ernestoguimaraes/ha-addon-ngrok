# Documentation

This add-on runs the ngrok v3 client and opens one or more tunnels from ngrok
to services reachable from Home Assistant. The default tunnel exposes the
Home Assistant web interface on port `8123`. A tunnel makes its configured
service reachable from the internet; keep Home Assistant authentication
enabled, and use ngrok access controls or HTTP authentication where appropriate.

## Installation and first-time setup

1. **Add this add-on repository.** In Home Assistant, open **Settings >
   Add-ons > Add-on Store**, open the menu, choose **Repositories**, and add
   <https://github.com/ernestoguimaraes/ha-addon-ngrok>. The repository can
   also be added using the [Home Assistant repository shortcut][add_repo].
2. **Install the add-on.** Find **ngrok Client Installer (Unofficial)** in the
   add-on store and install it. This repository provides builds for `aarch64`
   and `amd64`.
3. **Get an ngrok authentication token.** Sign in to your ngrok account and
   copy the token from the [ngrok dashboard][ngrok_token]. In the add-on's
   **Configuration** tab, enter it in `auth_token`. The token is required to
   authenticate and create tunnels.
4. **Review the tunnel configuration.** The default configuration creates an
   HTTP tunnel named `hass` to port `8123`. For a normal Home Assistant
   installation, you can keep this tunnel and change only the token. If
   Home Assistant listens on another port or you want to expose another
   service, update the tunnel's `addr` as described in [Tunnel options](#option-tunnels).
5. **Find the proxy address and configure `trusted_proxies`.** First, try
   opening the tunnel's public URL. If Home Assistant rejects the request as
   coming from an untrusted proxy, open **Settings > System > Logs** and inspect
   the Home Assistant Core log. Find the message about a request received from
   an untrusted reverse proxy; it includes the source IP address that Core
   actually sees. Use that address (or the narrow subnet containing the
   add-on's address) in `trusted_proxies`. This is the proxy's address on the
   local Home Assistant network, not your browser's public IP or the ngrok
   public URL.

   Add the value under the `http:` section of Home Assistant's
   `configuration.yaml`, alongside `use_x_forwarded_for: true`. For example,
   replace the sample subnet below with the address or subnet identified from
   your Core log:

   ```yaml
   http:
     use_x_forwarded_for: true
     trusted_proxies:
       - 172.30.32.1
   ```

   The example address is illustrative; use the exact source IP from your log.
   If the add-on's source address can change, configure only the smallest
   subnet that contains its possible addresses. Do not add a broad range such
   as `0.0.0.0/0`. See [Home Assistant's reverse proxy
   documentation][trusted_proxies_docs] for details about proxy configuration.
6. **Start the add-on.** Open its **Info** tab and select **Start**. Optionally
   enable **Start on boot** so the tunnel starts automatically after a reboot.
7. **Restart Home Assistant Core** after changing `configuration.yaml`, then
   open the add-on's **Web UI**. The ngrok inspector is available on TCP port
   `4040`; the Web UI shows the active tunnel and its public URL.

If you change the add-on configuration later, save it and restart the add-on.
If you change Home Assistant's `trusted_proxies` configuration, restart
Home Assistant Core as well.

### Verify the tunnel

After both the add-on and Home Assistant Core are running:

1. Check the add-on **Log** tab for startup errors. A successful start should
   show the ngrok client starting without an authentication or connection
   error.
2. Open the add-on's **Web UI**. Confirm that the `hass` tunnel is listed and
   copy its public URL.
3. Open that URL in a browser that can reach the internet. It should show the
   Home Assistant sign-in page.

If Home Assistant reports an untrusted proxy, use the Core log procedure above
to find the source address and update `trusted_proxies`. If the tunnel does
not appear, check that `auth_token` is valid and that `addr` points to a
service and port reachable from the add-on.

## Add-on configuration

Enter these values in the add-on's **Configuration** tab. This is the add-on
options object; do not include an extra top-level `options:` key.

```yaml
log_level: info
auth_token: YOUR_NGROK_AUTH_TOKEN
region: us
tunnels:
  - name: hass
    proto: http
    addr: 8123
    inspect: false
    auth: ""
```

The add-on defaults are `log_level: info`, `region: us`, and one `hass` HTTP
tunnel to port `8123`. The default `auth_token` is empty, so replace it with
your own token before starting. Keep the token private; the add-on stores its
generated ngrok configuration with owner-only permissions.

### Option: `log_level`

Sets the add-on log verbosity. Supported values are `crit`, `warn`, `error`,
`info`, and `debug`. The default is `info`. Use `debug` when troubleshooting
and return to `info` afterward.

### Option: `auth_token`

Your ngrok authentication token. This value is required to authenticate with
ngrok. Treat it like a password and do not post it in logs, screenshots, or
public issue reports.

### Option: `region`

Selects the ngrok region where the client hosts its tunnels. Supported values:

| Value | Location |
| :---: | -------- |
| `us` | United States |
| `eu` | Europe |
| `ap` | Asia/Pacific |
| `au` | Australia |
| `sa` | South America |
| `jp` | Japan |
| `in` | India |

The default is `us`.

### Option: `tunnels`

`tunnels` is a list, so each tunnel is a separate item beginning with `-`.
Each item requires a unique `name`, a `proto`, and an `addr`. The add-on
starts all configured tunnels. The default is:

```yaml
tunnels:
  - name: hass
    proto: http
    addr: 8123
    inspect: false
    auth: ""
```

`addr` accepts a port from `1` to `65535` or an IPv4 `address:port`. When you
provide only a port, the add-on targets `172.30.32.1:<port>`; this is why the
default `8123` forwards to Home Assistant. For example, to forward to a
different local service on port `8080`, set `addr: 8080`. IPv6 addresses and
hostnames are not accepted by the add-on's configuration schema.

| Option | Protocol | Description |
| ------ | -------- | ----------- |
| `name`* | all | Unique tunnel name, used as the key in the generated ngrok configuration and client API. |
| `proto`* | all | Tunnel protocol: `http`, `tls`, or `tcp`. |
| `addr`* | all | Destination port (`1`-`65535`) or IPv4 address and port (`address:port`). |
| `inspect` | http | Enables HTTP request inspection. Defaults to `false`. |
| `auth` | http | HTTP Basic Authentication credentials in `username:password` form. Defaults to an empty string. |
| `host_header` | http | Rewrites the HTTP `Host` header; omit to leave it unchanged. |
| `bind_tls` | http | Set to `true` to bind an HTTPS endpoint or `false` to bind an HTTP endpoint. |
| `subdomain` | http, tls | Requests a specific ngrok subdomain, subject to your ngrok account and domain configuration. |
| `hostname` | http, tls | Requests a specific hostname; configure/reserve the domain with ngrok and point its DNS as required by ngrok. |
| `crt` | tls | Path to a PEM TLS certificate for terminating TLS before forwarding locally. |
| `key` | tls | Path to the matching PEM TLS private key. |
| `client_cas` | tls | Path to a PEM certificate authority used to verify incoming TLS client certificates. |
| `remote_addr` | tcp | Remote TCP address in IPv4 `address:port` form. |
| `metadata` | all | User-defined metadata exposed through the ngrok service API. |

*Required by the add-on schema.

The options other than `name`, `proto`, and `addr` are optional in the schema.
Protocol-specific options are passed through to ngrok, which may reject an
option that is not valid for the selected protocol. See [ngrok's tunnel
configuration documentation][ngrok_docs_tunnels] for protocol details and
account/domain requirements.

### Example: expose another HTTP service

To expose a service available on port `8080` as well as Home Assistant, add a
second list item. Give it a distinct name:

```yaml
tunnels:
  - name: hass
    proto: http
    addr: 8123
    inspect: false
    auth: ""
  - name: my-service
    proto: http
    addr: 8080
    inspect: false
    auth: ""
```

## Home Assistant integration

The ngrok client API can be used to read tunnel status and public URLs. The
add-on exposes the inspector/API on port `4040` by default. From Home Assistant,
use an address for the Home Assistant host that is reachable from Core; do not
assume `localhost` works, because it refers to the Core container itself.
If you changed the add-on's host port mapping, use that port instead.

### Example: create a sensor for the public URL

Add a REST sensor to Home Assistant's `configuration.yaml` (or to a package
that is included by your configuration):

```yaml
sensor:
  - platform: rest
    resource: http://HOME_ASSISTANT_HOST:4040/api/tunnels/hass
    name: Home Assistant URL
    value_template: "{{ value_json.public_url }}"
```

Replace `HOME_ASSISTANT_HOST` with a hostname or IP address of the Home
Assistant host that Core can reach. Replace `hass` in the URL path if you
renamed the tunnel. After editing the configuration, check it in Home
Assistant and restart Core. The sensor entity will be
`sensor.home_assistant_url`.

For other tunnel details and API endpoints, see [ngrok's client API
documentation][ngrok_docs_api].

[add_repo]: https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Fernestoguimaraes%2Fha-addon-ngrok%2F
[ngrok_token]: https://dashboard.ngrok.com/get-started/your-authtoken
[ngrok_docs_tunnels]: https://ngrok.com/docs#tunnel-definitions
[ngrok_docs_api]: https://ngrok.com/docs#client-api
[trusted_proxies_docs]: https://www.home-assistant.io/integrations/http#reverse-proxies
