# FrostBanner – Cookie Consent Banner for Google Tag Manager

Official Google Tag Manager template for [FrostBanner](https://frostbanner.com), a cookie consent banner with Google Consent Mode v2 and a consent log.

The template:

- sets Google Consent Mode v2 defaults as soon as the container loads (everything denied except `security_storage`, or your own defaults by region);
- applies a returning visitor's saved choice right away, from the `fb_consent` cookie;
- loads the FrostBanner widget from `https://app.frostbanner.com/v1/fb.js` with your Site ID;
- passes every visitor choice to Tag Manager with `updateConsentState`;
- optionally pushes a `frostbanner_consent_update` event to the data layer.

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
| Site ID | Your FrostBanner site ID. Required. |
| Default consent by region | Optional table. Empty = everything except `security_storage` denied in all regions. Region codes are ISO 3166-2 (`PT`, `US-CA`). |
| Wait for update | Milliseconds Google tags wait for the visitor's choice. Default 500. |
| Apply a returning visitor's saved choice immediately | Reads the `fb_consent` cookie. On by default. |
| Push a `frostbanner_consent_update` event | Data layer event after each choice, with `frostbanner.analytics`, `frostbanner.marketing`, `frostbanner.functional` (true/false) and `frostbanner.source`. On by default. |
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
| Accesses globals `frostBannerConfig`, `FrostBanner.getConsent`, `FrostBanner.on`, `dataLayer` | Passes the Site ID to the widget, reads the visitor's choice, follows each new choice, pushes the optional event. |
| Writes `ads_data_redaction`, `url_passthrough` | Optional Google settings. |
| Logs to console (debug only) | Messages in Preview mode. |

## Support

support@frostbanner.com · Issues: <https://github.com/automatept-ai-pt/frostbanner-gtm-template/issues>

## License

Apache 2.0, see [LICENSE](LICENSE).
