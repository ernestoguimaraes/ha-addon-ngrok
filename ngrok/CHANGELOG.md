## Changes

### 21.0.10.3.0
- Upgrade the Home Assistant base image to v21.0.10.
- Drop armv7 support to match the base image architectures.
- Mask the ngrok auth token in add-on configuration and keep it out of debug logs.
- Use the current Bashio app port function.

### 2.1.10
- log version of client
- v3 stable client installed and configured
- further branding work.

### 2.1.9
- Repo and addon to https://github.com/pssc/ha-addon-ngrok

### 2.1.8

- Fix S6 error fixes [#12](https://github.com/dy1io/hassio-addons/issues/12)
- Require auth token (ngrok now requires an account to use their service)


### 2.1.7

**Note**: You will need to uninstall and reinstall to go to v2.1.7

- add archived notice @dy1io (#33)
- Fix missing build arg @dy1io (#32)
- Update Docs for Trusted-Proxies Change @dy1io (#31)
- Rename @dy1io (#30)
- Require local build
