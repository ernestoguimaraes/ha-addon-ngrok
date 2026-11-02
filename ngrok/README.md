# ngrok client installer and configurator addon for Home Assistant

Based on the work of "Dylan Hasler" form "Home Assistant Add-ons - dy1io", https://github.com/dy1io/hassio-addons

## About

This add-on creates ngrok tunnels using the `http`, `tls`, and `tcp` protocols.
It also provides a RESTful API that can be used to integrate your tunnel status
into Home Assistant.
Some ISPs do not allow port forwarding due to port blocking or CG-NAT. If you're
able to setup a port forward, do not use this addon.

**Note**: _ngrok could man-in-the-middle your http tunnels if they wanted.
          Using a TLS tunnel will prevent this because you can control the certificate.
          To use some features of this add-on, you need a paid ngrok account._

See the [documentation](DOCS.md) for installation, configuration options, and
Home Assistant integration.

[github_link]: https://github.com/ernestoguimaraes/ha-addon-ngrok/issues
