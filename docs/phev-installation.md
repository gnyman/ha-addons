# Install and test PHEV

Add `https://github.com/gnyman/ha-addons` to Home Assistant's app store under
**Settings → Apps → Install app → ⋮ → Repositories**. On older releases these
pages are called Add-ons and Add-on Store. If the repository is already added,
select **Check for updates**. PHEV version **1.8** should appear in this
repository's section.

Install it. This is a source-built app: Home Assistant builds the image from
its Dockerfile, including the vendored patched bridge. No bridge checkout,
submodule command, or image-registry login is needed on the HA device. The first
build can take several minutes. Build failures are reported in Supervisor logs.

## Moving from ~/local_apps/phev

The local app and repository app have different full IDs, even though both use
the `phev` slug. This is a new installation, not an in-place update of the local
one. The source folder at `~/local_apps/phev` can stay as a rollback copy.

1. Copy the existing app's Configuration values into the new app's Configuration
   tab: MQTT server, username/password, `phev_ip`, and `phev_port`. Keep secrets
   in the HA configuration UI. Use the vehicle address reachable from HA; the
   forwarded address used during development is not necessarily the right one
   for HA itself.
2. Set `debug: false` for normal operation. Debug mode only sleeps and does not
   start the bridge.
3. Stop the old local PHEV app and any other PHEVCTRL bridge before starting the
   new one. The vehicle has a single client slot, and the bridges also share a
   default MQTT client ID.
4. Start the new app and inspect its logs. Confirm that it completes the protocol
   handshake and publishes vehicle updates, rather than repeatedly reconnecting.
5. In HA's MQTT devices/entities, check **Charge State** and **Charge Timer
   Override**. State data is read-only until you deliberately toggle a command.
   The existing charging binary sensor is preserved for existing automations.
6. Leave the old local app stopped, with start-on-boot/watchdog disabled, once
   the replacement is working. To roll back, stop the new app before starting
   the old one.

The new Charge State sensor reports `charging`, `waiting`, `unplugged`, `idle`,
or a timer `error`; unknown data or disconnection makes it unavailable. The
legacy `/charge/charging` topic still reports only `on`/`off`. Override ON means
charge now; override OFF restores timer control. A scheduled waiting state can
coexist with the old charging bit during a transition.

## Publishing an update

This repository pins `phev/` to an exact commit in
`gnyman/home-assistant-addon-phev2mqtt`. Publish the bridge fork's commits first,
then the add-on source repository, and finally the ha-addons commit that updates
this pointer. Publishing the pointer before its source commit is reachable
would break submodule checkout.

For maintainers updating the checkout:

```sh
git submodule update --init phev
git -C phev fetch origin main
git -C phev checkout <reviewed-add-on-commit>
git add phev
git commit
```

The app version must increase when releasing a subsequent update. Source builds
happen on the HA device; this repository's existing image-publishing workflow
skips apps without an `image` property.
