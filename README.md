# Whirlpool Legacy for Home Assistant

This custom integration uses the Whirlpool Sixth Sense legacy cloud API to expose supported appliances in Home Assistant. This fork extends the laundry controls for specific API144 washer and dryer models. Avoid configuring the same appliance through multiple integrations.

The code also retains upstream air-conditioner, refrigerator, and oven platforms, but the added controls and validation described here concern laundry appliances. Availability and supported options depend on the model and the appliance's current state. This is an unofficial community project, not a Whirlpool product.

## What it provides

- Washer and dryer state, door, remote-control status, and completion-time entities, where reported by the appliance.
- Washer cycle selection and supported temperature, spin, soil, rinse, presoak, Fan Fresh, steam, and dispenser configuration controls.
- Dryer cycle selection and supported temperature, dryness, Wrinkle Shield, static-guard, EcoBoost, and manual-dry-time controls.
- Start, Pause, Resume, and Cancel controls when the appliance permits them. These controls **can operate the appliance**; confirm its condition before using them remotely.
- A washer **Cycle Time Remaining** sensor that counts down locally between appliance updates, freezes while paused, reads `0 min` at completion, and clears when idle. The completion/end-time entity is separate.
- Home Assistant-local **Favorite Cycle** selects and Save/Delete Favorite actions for supported washers and dryers.
- Eleven model-specific washer Specialty (Download & Go) cycles on the validated `WFW9620HBK3`: Activewear, Blankets, Business Casual, Coats & Jackets, Comforters, Diapers, Jeans, Lingerie, Machine Wash Curtains, Sleeping Bags, and Swimwear.

Controls are capability-aware. For example, an option unavailable for the selected cycle is not offered as universally supported. Dryer configuration selects use the dryer's changeability status, not a blanket Remote Control Enable requirement, so some can be available while the dryer is off. Starting a cycle remains a separate operation.

## Installation and setup

Manual installation is the documented method for this source tree:

1. Back up any existing `custom_components/whirlpool` directory and Home Assistant configuration before replacing an installed Whirlpool integration. This package uses the `whirlpool` domain and can override a built-in integration of the same domain.
2. Copy the `whirlpool` directory from this repository's `custom_components` directory to `<Home Assistant config>/custom_components/whirlpool`.
3. Restart Home Assistant. The manifest installs a pinned revision of a `whirlpool-sixth-sense` fork, so Home Assistant needs network access to retrieve that dependency.
4. In **Settings > Devices & services > Add integration**, select Whirlpool Legacy and enter the account credentials, region, and brand requested by the flow. The current flow offers EU/US regions and Whirlpool, Maytag, KitchenAid, and Consul brands. Use the brand and region associated with your account.

Model support is not implied by a brand choice in the setup flow. This fork's Specialty table is explicitly limited to `WFW9620HBK3`; other controls appear only when the underlying library reports them as supported. This integration is intended for installation as a HACS custom repository. It is not submitted to the HACS default store because it intentionally overrides Home Assistant's built-in `whirlpool` domain.

## HACS Custom Repository Installation

Whirlpool Legacy intentionally uses the `whirlpool` domain to replace/override Home Assistant's built-in Whirlpool integration. Because of this domain conflict, Whirlpool Legacy is distributed as a HACS custom repository rather than through the HACS default store.

1. Open HACS.
2. Open **Integrations**.
3. Open **Custom repositories**.
4. Add `https://github.com/aevans0001/Whirlpool-Legacy`.
5. Select **Integration** as the category.
6. Find and install **Whirlpool Legacy**.
7. Restart Home Assistant when requested.
8. In Home Assistant, go to **Settings > Devices & services**, add **Whirlpool Legacy**, and complete the configuration flow.

## Home Assistant-local Favorites

A supported washer or dryer has a **Favorite Cycle** select entity. Favorites are stored locally by Home Assistant, separately per appliance; they do **not** synchronize with the Whirlpool app's native Favorite list. A genuinely fresh installation receives one optional washer example named **Socks**. It can be replaced or deleted like any other Favorite. Deleting it persists across reloads and restarts; it is never recreated merely because its name is missing.

To save the appliance's **currently reported** cycle configuration, use **Developer Tools > Actions** or an automation with `whirlpool.save_favorite`. Target that appliance's Favorite Cycle select. Saving writes a recipe to Home Assistant storage; **it sends no configuration command to the appliance**. It does not start a cycle.

```yaml
action: whirlpool.save_favorite
target:
  entity_id: select.my_washer_favorite_cycle
data:
  name: Weekend Laundry
```

To remove any recipe:

```yaml
action: whirlpool.delete_favorite
target:
  entity_id: select.my_washer_favorite_cycle
data:
  name: Weekend Laundry
```

Replace the example entity ID with your own washer or dryer Favorite Cycle entity ID. Save and Delete are **actions, not device-page button entities**. Deleting changes only the local saved recipe. Every Favorite, including Socks and previously saved recipes, can be deleted.

Names have whitespace collapsed, must be 1-50 characters, and cannot be `None` (case-insensitively). Saving a Favorite with the same name, ignoring case, replaces the existing recipe for that appliance. The washer and dryer Favorite namespaces are separate.

The washer captures a valid What-to-Wash/How-to-Wash pair and supported current options, including temperature, spin, soil, extra rinse, presoak, Fan Fresh, steam, and applicable dispenser settings. Specialty or utility cycles cannot be saved as ordinary What/How Favorites. The dryer captures its What-to-Dry/How-to-Dry pair and applicable temperature, drying level, Wrinkle Shield, Static Guard, damp-dry signal, EcoBoost, and manual dry time.

**Selecting/applying a Favorite is different from saving one:** selecting a recipe can send cycle-configuration commands to the appliance. It does not itself send Start. Check the appliance and selected settings before applying or starting a cycle. The Favorite label reflects Home Assistant's applied configuration; incoming appliance updates do not independently assert that label.

## Known limitations and troubleshooting

- The library refreshes each discovered appliance sequentially during its approximately five-minute keepalive cycle. A refresh failure for one appliance does not prevent the remaining appliances from being refreshed.
- Specialty cycles are validated for `WFW9620HBK3`, not every washer model. Available regular cycle combinations and options vary by model. The validated Colors + Cold Wash combination remains available; Steam is not offered for Cold Wash, and a Bulky + Sanitize combination lacking a valid appliance value is hidden.
- HA-local Favorites are not cloud-native Favorites. The integration initializes `<config>/.storage/whirlpool.favorites` on the first setup with a supported appliance. Do not edit that storage file manually while Home Assistant is running. Existing installations that used an earlier build with implicit presets require the documented one-time migration before updating, or those implicit presets will not be carried into local storage.
- If an entity is `unknown` or `unavailable`, check the appliance's connectivity and whether it currently permits changes. If new Python code has been installed, restart Home Assistant; a frontend refresh alone does not reload an integration module.

## Development, attribution, and release status

Whirlpool Legacy includes modified portions of [Home Assistant Core's Whirlpool integration](https://github.com/home-assistant/core/tree/dev/homeassistant/components/whirlpool), licensed under Apache License 2.0. The standalone integration carries that license and attribution; the original Home Assistant contributors retain their rights.

The integration depends on a separately maintained, pinned [Whirlpool Legacy library](https://github.com/aevans0001/whirlpool-legacy-library). That dependency is **not bundled** with this repository and is separately licensed under MIT, with its existing copyright and license retained. Generic protocol changes belong there; Home Assistant entities, actions, and local Favorite storage belong in this integration.

The integration's automated tests are in `tests`. They require a separate checkout of the pinned `whirlpool-sixth-sense` library, for example cloned beside this repository. From this repository's root in Windows PowerShell, point `WHISK_LIBRARY_DIR` at that checkout before running them:

```text
$env:WHISK_LIBRARY_DIR = (Resolve-Path '..\whirlpool-legacy-library').Path
py -3.13 -m pytest -q tests
```

This source tree is prepared as a standalone Home Assistant integration repository. A live installation using earlier implicit Favorites still requires the documented offline Favorites migration before updating.
