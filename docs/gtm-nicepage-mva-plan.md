# GTM Minimum Viable Analytics Plan for Nicepage

This plan translates the current Next.js website into a Nicepage-ready Google Tag Manager implementation that can be deployed in under two hours.

## Architecture Review

### Current route inventory

The existing site uses the Next.js App Router with one public content page and one health API route. The public homepage is defined in `app/page.tsx`; the global shell is defined in `app/layout.tsx`. Health aliases are configured as rewrites in `next.config.ts` and should be excluded from marketing analytics.

| URL            | Source                                    | Include in GTM page taxonomy? | service_interest                | page_type         | content_category             | Notes                                                                                          |
| -------------- | ----------------------------------------- | ----------------------------- | ------------------------------- | ----------------- | ---------------------------- | ---------------------------------------------------------------------------------------------- |
| `/`            | `app/page.tsx`                            | Yes                           | `nextjs_enterprise_boilerplate` | `home`            | `developer_tooling_overview` | Main landing page with product value proposition, feature grid, and two primary outbound CTAs. |
| `/healthz`     | `next.config.ts` rewrite to `/api/health` | No                            | `not_applicable`                | `system_endpoint` | `health_check`               | Operational health endpoint. Exclude from GA4/GTM marketing reporting.                         |
| `/api/healthz` | `next.config.ts` rewrite to `/api/health` | No                            | `not_applicable`                | `system_endpoint` | `health_check`               | Operational health endpoint. Exclude from GA4/GTM marketing reporting.                         |
| `/health`      | `next.config.ts` rewrite to `/api/health` | No                            | `not_applicable`                | `system_endpoint` | `health_check`               | Operational health endpoint. Exclude from GA4/GTM marketing reporting.                         |
| `/ping`        | `next.config.ts` rewrite to `/api/health` | No                            | `not_applicable`                | `system_endpoint` | `health_check`               | Operational health endpoint. Exclude from GA4/GTM marketing reporting.                         |
| `/api/health`  | `app/api/health/route.ts`                 | No                            | `not_applicable`                | `system_endpoint` | `health_check`               | API route returning service health. Exclude from GA4/GTM marketing reporting.                  |

### Current conversion actions

| Element                 | Current URL                                                                                     | Recommended event                          | service_interest                | page_type | content_category             |
| ----------------------- | ----------------------------------------------------------------------------------------------- | ------------------------------------------ | ------------------------------- | --------- | ---------------------------- |
| `Get started` CTA       | `https://github.com/Blazity/next-enterprise`                                                    | `select_cta`                               | `nextjs_enterprise_boilerplate` | `home`    | `developer_tooling_overview` |
| `Deploy Now` CTA        | `https://vercel.com/new/git/external?repository-url=https://github.com/Blazity/next-enterprise` | `select_cta`                               | `nextjs_enterprise_boilerplate` | `home`    | `developer_tooling_overview` |
| Feature-grid engagement | In-page feature cards                                                                           | `feature_card_click` if cards become links | `nextjs_enterprise_boilerplate` | `home`    | `developer_tooling_overview` |

## Minimum Viable Analytics Scope

Deploy only the events needed to validate traffic quality and CTA intent:

1. GA4 page views with page taxonomy.
2. CTA click events for the two homepage buttons.
3. Outbound click events for GitHub and Vercel destinations.
4. 90% scroll-depth engagement on the homepage.
5. Optional form-submit trigger only if a Nicepage contact form is added later.

## 1. GTM Variables

Create these GTM user-defined variables.

| Variable name                   | Type                | Configuration                                           | Purpose                                       |
| ------------------------------- | ------------------- | ------------------------------------------------------- | --------------------------------------------- |
| `DLV - service_interest`        | Data Layer Variable | Data Layer Variable Name: `service_interest`; Version 2 | Sends page/service taxonomy to GA4.           |
| `DLV - page_type`               | Data Layer Variable | Data Layer Variable Name: `page_type`; Version 2        | Sends page template/category taxonomy to GA4. |
| `DLV - content_category`        | Data Layer Variable | Data Layer Variable Name: `content_category`; Version 2 | Sends content grouping to GA4.                |
| `DLV - cta_text`                | Data Layer Variable | Data Layer Variable Name: `cta_text`; Version 2         | Captures clicked CTA label.                   |
| `DLV - cta_destination`         | Data Layer Variable | Data Layer Variable Name: `cta_destination`; Version 2  | Captures clicked CTA URL.                     |
| `DLV - cta_location`            | Data Layer Variable | Data Layer Variable Name: `cta_location`; Version 2     | Captures CTA placement such as `hero`.        |
| `DLV - feature_name`            | Data Layer Variable | Data Layer Variable Name: `feature_name`; Version 2     | Future-proofs feature-card engagement.        |
| `Constant - GA4 Measurement ID` | Constant            | Value: `G-XXXXXXXXXX`                                   | Replace with the production GA4 stream ID.    |

Enable these built-in GTM variables:

- `Page URL`
- `Page Path`
- `Page Hostname`
- `Click URL`
- `Click Text`
- `Click Classes`
- `Click ID`
- `Click Element`
- `Form ID`
- `Form Classes`
- `Scroll Depth Threshold`

## 2. GTM Triggers

| Trigger name                | Type               | Conditions                                                                                                | Fires on                                            |
| --------------------------- | ------------------ | --------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| `PV - Public Content Pages` | Page View          | `Page Path equals /`                                                                                      | The homepage only. Do not fire on health endpoints. |
| `CE - page_context_ready`   | Custom Event       | Event name equals `page_context_ready`                                                                    | The page taxonomy data layer event.                 |
| `CE - select_cta`           | Custom Event       | Event name equals `select_cta`                                                                            | Nicepage CTA click handler pushes.                  |
| `Click - Outbound Links`    | Click - Just Links | Some Link Clicks: `Click URL does not contain {{Page Hostname}}` AND `Click URL matches RegEx ^https?://` | Outbound GitHub/Vercel clicks.                      |
| `Scroll - 90 Percent`       | Scroll Depth       | Vertical Scroll Depths: `90`; This trigger fires on: `Page Path equals /`                                 | Homepage deep engagement.                           |
| `Form - Lead Submit`        | Form Submission    | Some Forms: `Page Path equals /`                                                                          | Optional only if a Nicepage form is added.          |

## 3. GTM Tags

| Tag name               | Type       | Trigger                                                     | Event name       | Parameters                                                                                                                                                                  |
| ---------------------- | ---------- | ----------------------------------------------------------- | ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `GA4 - Google Tag`     | Google Tag | `Initialization - All Pages` or `PV - Public Content Pages` | n/a              | Tag ID: `{{Constant - GA4 Measurement ID}}`; send automatic page view enabled.                                                                                              |
| `GA4 - page_context`   | GA4 Event  | `CE - page_context_ready`                                   | `page_context`   | `service_interest`, `page_type`, `content_category` from DLV variables.                                                                                                     |
| `GA4 - select_cta`     | GA4 Event  | `CE - select_cta`                                           | `select_cta`     | `cta_text`, `cta_destination`, `cta_location`, `service_interest`, `page_type`, `content_category`. Mark as a key event in GA4 if CTA intent is the primary MVP conversion. |
| `GA4 - outbound_click` | GA4 Event  | `Click - Outbound Links`                                    | `outbound_click` | `link_url={{Click URL}}`, `link_text={{Click Text}}`, `service_interest`, `page_type`, `content_category`.                                                                  |
| `GA4 - scroll_90`      | GA4 Event  | `Scroll - 90 Percent`                                       | `scroll_depth`   | `percent_scrolled={{Scroll Depth Threshold}}`, `service_interest`, `page_type`, `content_category`.                                                                         |
| `GA4 - generate_lead`  | GA4 Event  | `Form - Lead Submit`                                        | `generate_lead`  | Optional. Add only if a Nicepage form exists. Parameters: `form_id`, `service_interest`, `page_type`, `content_category`.                                                   |

## 4. Data Layer Code

### Nicepage site-wide GTM container snippet

Add the standard GTM container script in **Nicepage > Site Settings > HTML > Head** and the noscript iframe immediately after the opening body tag if the Nicepage export/settings area supports it. Replace `GTM-XXXXXXX` with the production GTM container ID.

```html
<!-- Google Tag Manager -->
<script>
  ;(function (w, d, s, l, i) {
    w[l] = w[l] || []
    w[l].push({ "gtm.start": new Date().getTime(), event: "gtm.js" })
    var f = d.getElementsByTagName(s)[0],
      j = d.createElement(s),
      dl = l != "dataLayer" ? "&l=" + l : ""
    j.async = true
    j.src = "https://www.googletagmanager.com/gtm.js?id=" + i + dl
    f.parentNode.insertBefore(j, f)
  })(window, document, "script", "dataLayer", "GTM-XXXXXXX")
</script>
<!-- End Google Tag Manager -->
```

```html
<!-- Google Tag Manager (noscript) -->
<noscript>
  <iframe
    src="https://www.googletagmanager.com/ns.html?id=GTM-XXXXXXX"
    height="0"
    width="0"
    style="display: none; visibility: hidden"
  ></iframe>
</noscript>
<!-- End Google Tag Manager (noscript) -->
```

### Homepage taxonomy push

Add this once on the Nicepage homepage before any CTA click scripts. Use **Page Settings > Additional HTML > Head** or a page-level custom code block.

```html
<script>
  window.dataLayer = window.dataLayer || []
  window.dataLayer.push({
    event: "page_context_ready",
    service_interest: "nextjs_enterprise_boilerplate",
    page_type: "home",
    content_category: "developer_tooling_overview",
  })
</script>
```

### CTA click tracking for the Nicepage homepage

Preferred Nicepage implementation: add stable classes to the two buttons in Nicepage.

- `Get started`: add class `js-gtm-cta-get-started`.
- `Deploy Now`: add class `js-gtm-cta-deploy-now`.

Then add this page-level script after the buttons are rendered, preferably before the closing body tag.

```html
<script>
  window.dataLayer = window.dataLayer || []

  function pushCtaClick(element, ctaText, ctaLocation) {
    if (!element) return

    element.addEventListener("click", function () {
      window.dataLayer.push({
        event: "select_cta",
        service_interest: "nextjs_enterprise_boilerplate",
        page_type: "home",
        content_category: "developer_tooling_overview",
        cta_text: ctaText,
        cta_destination: element.href || "",
        cta_location: ctaLocation,
      })
    })
  }

  document.addEventListener("DOMContentLoaded", function () {
    pushCtaClick(document.querySelector(".js-gtm-cta-get-started"), "Get started", "hero")
    pushCtaClick(document.querySelector(".js-gtm-cta-deploy-now"), "Deploy Now", "hero")
  })
</script>
```

If Nicepage makes class management difficult, use href selectors instead:

```html
<script>
  window.dataLayer = window.dataLayer || []

  document.addEventListener("DOMContentLoaded", function () {
    var ctas = [
      {
        selector: 'a[href="https://github.com/Blazity/next-enterprise"]',
        text: "Get started",
      },
      {
        selector: 'a[href^="https://vercel.com/new/git/external"]',
        text: "Deploy Now",
      },
    ]

    ctas.forEach(function (cta) {
      var element = document.querySelector(cta.selector)
      if (!element) return

      element.addEventListener("click", function () {
        window.dataLayer.push({
          event: "select_cta",
          service_interest: "nextjs_enterprise_boilerplate",
          page_type: "home",
          content_category: "developer_tooling_overview",
          cta_text: cta.text,
          cta_destination: element.href || "",
          cta_location: "hero",
        })
      })
    })
  })
</script>
```

## 5. Two-Hour Deployment Checklist

### Build checklist

1. Create/publish GTM container if it does not already exist.
2. Add the standard GTM snippet to Nicepage global head/body settings.
3. Create the variables listed above.
4. Create the triggers listed above.
5. Create these MVP tags only: `GA4 - Google Tag`, `GA4 - page_context`, `GA4 - select_cta`, `GA4 - outbound_click`, and `GA4 - scroll_90`.
6. Add the homepage taxonomy data layer push to the Nicepage homepage.
7. Add stable CTA classes or use the href-selector fallback script.
8. Preview the GTM workspace.
9. Publish only after all MVP checks pass.

### Testing checklist

| Check                                                                                         | Expected result                                                                                                                            |
| --------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Open `/` in GTM Preview                                                                       | The container loads and `GA4 - Google Tag` fires once.                                                                                     |
| Inspect the `page_context_ready` event                                                        | Data layer contains `service_interest=nextjs_enterprise_boilerplate`, `page_type=home`, and `content_category=developer_tooling_overview`. |
| Confirm `GA4 - page_context`                                                                  | Fires once on `page_context_ready` with all three taxonomy parameters.                                                                     |
| Click `Get started`                                                                           | `select_cta` event fires with `cta_text=Get started`, `cta_location=hero`, and the GitHub destination URL.                                 |
| Click `Deploy Now`                                                                            | `select_cta` event fires with `cta_text=Deploy Now`, `cta_location=hero`, and the Vercel destination URL.                                  |
| Confirm outbound click event                                                                  | `GA4 - outbound_click` fires for GitHub and Vercel links.                                                                                  |
| Scroll homepage to 90%                                                                        | `GA4 - scroll_90` fires once with `percent_scrolled=90`.                                                                                   |
| Open `/health`, `/healthz`, `/api/healthz`, `/ping`, and `/api/health` if publicly accessible | No marketing page taxonomy tag should fire; these endpoints should not appear as content pages in reports.                                 |
| Check GA4 DebugView                                                                           | Events appear with the same parameter names used in GTM.                                                                                   |
| Publish GTM                                                                                   | Workspace version notes include `MVA: homepage page taxonomy, CTA clicks, outbound clicks, scroll depth`.                                  |

## GA4 Reporting Notes

- Register `service_interest`, `page_type`, `content_category`, `cta_text`, and `cta_location` as GA4 custom dimensions if they are needed in standard reports.
- Mark `select_cta` as a GA4 key event for the MVP if GitHub/Vercel click intent is the main conversion.
- Do not mark `scroll_depth` as a key event; use it as an engagement quality signal only.
