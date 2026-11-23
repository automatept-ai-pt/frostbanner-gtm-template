# FrostBanner – Cookie Consent Banner for Google Tag Manager

Official Google Tag Manager template for [FrostBanner](https://frostbanner.com), a cookie consent banner with Google Consent Mode v2 and a consent log.

The template:

- sets Google Consent Mode v2 defaults as soon as the container loads (everything denied except `security_storage`, or your own defaults by region; `security_storage` is always granted);
- applies a returning visitor's saved choice right away, from the `fb_consent` cookie;
- loads the FrostBanner widget from `https://app.frostbanner.com/v1/fb.js` with your Site ID, in GTM mode (`frostBannerConfig.gtm = true`): the widget then sends no `gtag('consent', …)` commands of its own, only the template passes consent to Google;
- passes every choice the widget confirms to Tag Manager with `updateConsentState`;
- if the banner asks the visitor again (for example after the site's categories changed), sets a restored choice back to denied;
- optionally pushes a `frostbanner_consent_update` event to the data layer.

## How consent flows

1. Default: everything denied except `security_storage` (or your region table).
2. Returning visitor: the saved choice from the `fb_consent` cookie is applied at once, so Google tags get it before the widget loads. No data layer event yet.
   - A choice older than **Consent lifetime, days** (default 180, checked against the record's date `ts`) is not applied.
   - Set the same value as the consent lifetime (`maxAgeDays`) in your FrostBanner settings. With a longer value in the template, a choice the widget already treats as expired is applied until the widget asks again. With a shorter value, returning visitors start denied until the widget confirms their choice.
   - Changed categories and a raised `consentVersion` are checked by the widget only, not by the template. In that case the saved choice is applied until the widget loads, then the widget shows the banner again (`shown`) and consent goes back to denied. Google tags that fire before that (for example on Initialization or the first page view) can still use the old choice. If that is not acceptable for you, switch off **Apply a returning visitor's saved choice immediately**.
3. The widget loads and reports:
   - `change` — a choice is applied. The template updates consent and pushes `frostbanner_consent_update` with the widget's `source`: `restore` (saved choice confirmed), `accept_all`, `reject_all`, `custom` (banner), `outside_region` or `withdrawn` (a choice made before the widget started is withdrawn);
   - `shown` — the banner asks the visitor. If a saved choice was applied in step 2, consent goes back to denied until the visitor chooses.

   The widget can start while its script is still running (cached settings, page already parsed), before the template has subscribed to these events. So right after the script loads the template also reads `FrostBanner.getStatus()`:
   - `'pending'` — the widget has not started yet: wait for `shown` / `change`;
   - `'shown'` — the banner is on screen and there is no choice: same as the `shown` event;
   - `'decided'` — a choice is applied: the template takes it from `FrostBanner.getConsent()` and its source from `FrostBanner.getSource()`, same as a `change` event.

   `getStatus()` and `getSource()` must be present in the widget served from `app.frostbanner.com`. If the widget has no `getStatus()` (an older version), a choice applied from the cookie goes back to denied right after the script loads, and the template then follows `change` events only. If that older widget had already confirmed the choice while its script ran, no further `change` comes, so the returning visitor stays denied for the rest of that page view (safe, but tags lose that page's data until the visitor chooses again).

   If your site calls `FrostBanner.open()` or `FrostBanner.show()` after the widget script has loaded but before the widget has started, and the template reads the status in that moment, the status is already `'shown'`: a restored choice goes back to denied, and returns to granted a moment later when the widget confirms it (`change` with `restore`). Tags see a short denied period. The window is narrow, but "after the page has loaded" is not enough to avoid it: the widget can wait up to 2.5 s for its settings. Call `open()` / `show()` only after the widget has started, for example from a visitor's click on a "Cookie settings" link or after a `shown` / `change` event.
4. If the widget cannot load, the tag fails (`gtmOnFailure`), defaults and any restored choice stay as they are, and no event is pushed.

## Setup

1. Sign in to the [FrostBanner dashboard](https://app.frostbanner.com/app/), add your site and copy the **Site ID** from the **Install** tab.
2. In Google Tag Manager, open **Templates** → **Tag Templates** → **Search Gallery**, find **FrostBanner – Cookie Consent Banner** and add it to your workspace.
3. Create a new tag with this template, paste your Site ID.
4. Set the trigger to **Consent Initialization - All Pages**.
5. Remove any FrostBanner `<script>` you added to the site by hand, so the banner does not load twice.
6. Preview, check the banner and the consent state in Tag Assistant, then publish.

Full guide: <https://frostbanner.com/guides/google-tag-manager/>

## Fields

| Field | What it does |
| --- | --- |
| Site ID | Your FrostBanner site ID. Required. Letters, numbers, `-` and `_` (up to 64), checked again in code when the value comes from a variable. |
| Default consent by region | Optional table. Empty = everything except `security_storage` denied in all regions. Region codes are ISO 3166-2 (`PT`, `US-CA`). `security_storage` is always granted, even if listed as denied. |
| Wait for update | Milliseconds Google tags wait for the visitor's choice. Default 500. |
| Apply a returning visitor's saved choice immediately | Reads the `fb_consent` cookie. On by default. Set back to denied if the banner asks again. |
| Consent lifetime, days | A saved choice older than this is not applied. Default 180. Must match the consent lifetime (`maxAgeDays`) in your FrostBanner settings. |
| Push a `frostbanner_consent_update` event | Data layer event each time the widget confirms a choice, with `frostbanner.analytics`, `frostbanner.marketing`, `frostbanner.functional` (true/false) and `frostbanner.source` (`restore`, `accept_all`, `reject_all`, `custom`, `outside_region`, `withdrawn`). Never pushed from the cookie alone. On by default. |
| `ads_data_redaction`, `url_passthrough` | Optional Google settings. Off by default. |

## How FrostBanner categories map to Google consent types

| FrostBanner category | Google consent types |
| --- | --- |
| Necessary | `security_storage` (always granted) |
| Analytics | `analytics_storage` |
| Marketing | `ad_storage`, `ad_user_data`, `ad_personalization` |
| Functional | `functionality_storage`, `personalization_storage` |

## Permissions

| Permission | Why |
| --- | --- |
| Accesses consent state (write) | Sets defaults and updates consent. |
| Reads cookie `fb_consent` | Applies a returning visitor's saved choice. |
| Injects `https://app.frostbanner.com/v1/fb.js` | Loads the banner. |
| Accesses globals `frostBannerConfig` (read, write), `FrostBanner.on`, `FrostBanner.getStatus`, `FrostBanner.getConsent`, `FrostBanner.getSource` (execute), `dataLayer` (read, write) | Passes the Site ID and GTM mode to the widget, follows its `change` and `shown` events, reads its status, choice and source after it loads, pushes the optional event. |
| Writes `ads_data_redaction`, `url_passthrough` | Optional Google settings. |
| Logs to console (debug only) | Messages in Preview mode. |

## Support

support@frostbanner.com · Issues: <https://github.com/automatept-ai-pt/frostbanner-gtm-template/issues>

## License

Apache 2.0, see [LICENSE](LICENSE).
