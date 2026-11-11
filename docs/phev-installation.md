# Install and test PHEV-DRAGONS

Add `https://github.com/gnyman/ha-addons` to Home Assistant's app store under
**Settings → Apps → Install app → ⋮ → Repositories**. On older releases these
pages are called Add-ons and Add-on Store. If the repository is already added,
select **Check for updates**. PHEV-DRAGONS version **1.10** should appear in this
repository's section.

Install it. Home Assistant downloads a prebuilt minimal image from public GHCR
packages; it does not compile the bridge on your device. No bridge checkout,
submodule command, or image-registry login is needed on the HA device.

## Moving from ~/local_apps/phev

The repository offers PHEV-DRAGONS with the `phev-dragons` slug. Your old local
PHEV app keeps its `phev` slug and can remain installed alongside it. This is a
new installation with separate configuration. Keep `~/local_apps/phev` as a
rollback copy.

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

This repository pins `phev-dragons/` to an exact commit in
`gnyman/home-assistant-addon-phev2mqtt`. Publish the bridge fork's commits first,
then the add-on source repository. Wait for its image publishing workflow to
succeed for all architectures and verify that the GHCR packages are public.
Finally publish the ha-addons commit that updates this pointer. Publishing the
pointer before its source commit is reachable would break submodule checkout.

For maintainers updating the checkout:

```sh
git submodule update --init phev-dragons
git -C phev-dragons fetch origin phev-dragons
git -C phev-dragons checkout <reviewed-add-on-commit>
git add phev-dragons
git commit
```

The app version must increase when releasing a subsequent update. The source
repository workflow builds, tests, and publishes the images. This repository's legacy image builder skips
PHEV-DRAGONS to avoid duplicate publication.
