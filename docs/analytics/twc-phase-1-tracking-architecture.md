# Thomas Walker Consulting Phase 1 Engagement Intelligence Tracking Architecture

## Purpose and implementation stance

This Phase 1 architecture is designed for Thomas Walker Consulting (TWC), an Australian workplace mental health, coaching psychology, counselling, leadership development, and MHFA training business using a Nicepage website with GA4, GTM, Google Search Console, Google Business Profile, Notion CRM, MailerLite, Stripe, Xero, and LinkedIn. The system prioritises engagement quality over traffic volume, so success is measured by trust-building behaviour, service interest, enquiry quality, pipeline progression, and revenue attribution rather than raw lead count alone.

Phase 1 should be implemented in GTM on the Nicepage website, with GA4 used as the primary behavioural collection layer and Looker Studio or GA4 Explorations used for weekly reporting. Notion should become the manual system of record for qualified leads until Phase 2 automations connect form, booking, payment, and accounting data.

## Phase 1 measurement questions

| Funnel stage         | Business question                                                               | Primary evidence                                                                          |
| -------------------- | ------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| Awareness            | Which channels and search queries introduce qualified visitors to TWC?          | Sessions, users, landing pages, Search Console queries, GBP searches, UTM campaigns       |
| Engagement           | Which traffic sources drive meaningful attention rather than accidental visits? | `scroll_50`, `scroll_75`, `time_60_sec`, `time_120_sec`, `return_visit`, engaged sessions |
| Interest             | Which pages and content assets create trust and service intent?                 | About, contact, service page views, brochure downloads, calendar clicks, pricing clicks   |
| Lead generation      | Which channels and services create enquiries?                                   | Forms, course enquiries, phone clicks, email clicks, booking clicks                       |
| Pipeline progression | Which enquiries become qualified conversations or opportunities?                | Notion lead status, estimated value, service interest, first-touch and last-touch source  |
| Revenue attribution  | Which sources and services ultimately produce income?                           | Stripe payments, Xero invoices, Notion won/lost status, revenue by source and service     |

## Service taxonomy

Use the following `service_interest` values consistently in GTM, GA4, Notion, MailerLite tags, and future Stripe/Xero reporting:

| Service                          | Canonical value                    | Typical page or interaction signals                                                    |
| -------------------------------- | ---------------------------------- | -------------------------------------------------------------------------------------- |
| Mental Health First Aid          | `mhfa`                             | MHFA page view, course enquiry form, MHFA brochure, MHFA booking link                  |
| Executive Coaching               | `executive_coaching`               | Coaching page view, calendar click, leadership profile content                         |
| Leadership Development           | `leadership_development`           | Leadership page view, workshop enquiry, organisational training content                |
| Counselling                      | `counselling`                      | Counselling page view, counselling enquiry, phone or email click from counselling page |
| Burnout Programs                 | `burnout_programs`                 | Burnout page view, burnout brochure, burnout assessment/resource view                  |
| Workplace Mental Health Training | `workplace_mental_health_training` | Workplace training page view, corporate enquiry form, training brochure                |
| Unknown or mixed                 | `unknown`                          | Generic contact interactions when no page/service signal is available                  |

## Event naming conventions

Use lower snake_case for all GA4 event names. Keep event names action-oriented, stable, and platform-agnostic. Use parameters for context instead of creating many near-duplicate event names.

| Rule             | Standard                                                                                               |
| ---------------- | ------------------------------------------------------------------------------------------------------ |
| Event names      | Lower snake_case, for example `calendar_click`, `brochure_download`, `course_enquiry_submit`           |
| Service names    | Use `service_interest` canonical values from the service taxonomy                                      |
| Content grouping | Use `page_type` values such as `home`, `about`, `service`, `contact`, `resource`, `pricing`, `booking` |
| Source metadata  | Preserve GA4 default acquisition fields and add clean UTM discipline for campaigns                     |
| Avoid            | Event names containing page URLs, dates, ad hoc campaign names, or personally identifiable information |

## GA4 event map

| Event                   | Stage      | Trigger logic                                                                       | Key event? | Required parameters                                                                                  |
| ----------------------- | ---------- | ----------------------------------------------------------------------------------- | ---------- | ---------------------------------------------------------------------------------------------------- |
| `page_view`             | Awareness  | GA4 enhanced measurement or GTM configuration tag on all pages                      | No         | `page_type`, `service_interest`, `content_group`, `page_path`                                        |
| `scroll_50`             | Engagement | User reaches 50% vertical scroll once per page                                      | No         | `scroll_depth`, `page_type`, `service_interest`, `engagement_score_delta` = 0                        |
| `scroll_75`             | Engagement | User reaches 75% vertical scroll once per page                                      | No         | `scroll_depth`, `page_type`, `service_interest`, `engagement_score_delta` = 5                        |
| `time_60_sec`           | Engagement | User remains active for 60 seconds once per session                                 | No         | `time_threshold`, `page_type`, `service_interest`, `engagement_score_delta` = 3                      |
| `time_120_sec`          | Engagement | User remains active for 120 seconds once per session                                | No         | `time_threshold`, `page_type`, `service_interest`, `engagement_score_delta` = 0                      |
| `return_visit`          | Engagement | Visitor has previous visit marker in first-party storage                            | No         | `visit_count`, `days_since_last_visit`, `engagement_score_delta` = 5                                 |
| `video_play`            | Engagement | Embedded video begins playing                                                       | No         | `video_title`, `video_provider`, `page_type`, `service_interest`                                     |
| `resource_view`         | Engagement | User opens a resource page, PDF, guide, checklist, article, or embedded resource    | No         | `resource_name`, `resource_type`, `service_interest`, `content_group`                                |
| `about_page_view`       | Interest   | Page path or title maps to About page                                               | No         | `page_type` = `about`, `engagement_score_delta` = 5                                                  |
| `contact_page_view`     | Interest   | Page path or title maps to Contact page                                             | Yes        | `page_type` = `contact`, `engagement_score_delta` = 10                                               |
| `mhfa_page_view`        | Interest   | Page path or title maps to MHFA page                                                | No         | `service_interest` = `mhfa`, `page_type` = `service`, `engagement_score_delta` = 7                   |
| `coaching_page_view`    | Interest   | Page path or title maps to coaching page                                            | No         | `service_interest` = `executive_coaching`, `page_type` = `service`, `engagement_score_delta` = 7     |
| `counselling_page_view` | Interest   | Page path or title maps to counselling page                                         | No         | `service_interest` = `counselling`, `page_type` = `service`, `engagement_score_delta` = 7            |
| `burnout_page_view`     | Interest   | Page path or title maps to burnout page                                             | No         | `service_interest` = `burnout_programs`, `page_type` = `service`, `engagement_score_delta` = 7       |
| `leadership_page_view`  | Interest   | Page path or title maps to leadership page                                          | No         | `service_interest` = `leadership_development`, `page_type` = `service`, `engagement_score_delta` = 7 |
| `pricing_click`         | Interest   | Click on pricing, fees, package, quote, or investment CTA                           | Yes        | `link_text`, `link_url`, `service_interest`                                                          |
| `calendar_click`        | Interest   | Click to Calendly, calendar, booking page, discovery call, or appointment scheduler | Yes        | `link_text`, `link_url`, `service_interest`, `engagement_score_delta` = 20                           |
| `brochure_download`     | Interest   | Click to brochure PDF or downloadable guide                                         | Yes        | `file_name`, `file_extension`, `service_interest`, `engagement_score_delta` = 15                     |
| `form_submit`           | Lead       | Successful generic contact form submission only                                     | Yes        | `form_id`, `form_name`, `service_interest`, `lead_type`, `engagement_score_delta` = 30               |
| `phone_click`           | Lead       | Click on `tel:` link                                                                | Yes        | `phone_location`, `page_type`, `service_interest`                                                    |
| `email_click`           | Lead       | Click on `mailto:` link                                                             | Yes        | `email_location`, `page_type`, `service_interest`                                                    |
| `booking_click`         | Lead       | Click on primary booking/discovery call CTA                                         | Yes        | `link_text`, `link_url`, `service_interest`                                                          |
| `course_enquiry_submit` | Lead       | Successful MHFA or course-specific enquiry submission                               | Yes        | `form_id`, `course_name`, `service_interest` = `mhfa`, `engagement_score_delta` = 30                 |

## Recommended GA4 custom dimensions and metrics

Create event-scoped custom dimensions for `service_interest`, `page_type`, `content_group`, `resource_name`, `resource_type`, `lead_type`, `form_name`, `course_name`, `link_text`, `link_url`, `engagement_temperature`, and `traffic_quality_segment`. Create user-scoped dimensions for `first_service_interest`, `returning_user_flag`, and `highest_intent_service` if consent settings and data retention policies allow them.

Create an event-scoped custom metric called `engagement_score_delta` and use it to sum event-level score changes in Explorations or Looker Studio. In Phase 1, calculate total engagement score in reports rather than relying on client-side persistent scoring as the source of truth.

## Engagement scoring model

| Behaviour         | Score delta | Implementation note                                                                        |
| ----------------- | ----------: | ------------------------------------------------------------------------------------------ |
| Page view         |           1 | Report-calculated from `page_view`, not sent as score unless using a custom event pipeline |
| 60 seconds        |           3 | Send on `time_60_sec`                                                                      |
| 75% scroll        |           5 | Send on `scroll_75`                                                                        |
| Return visit      |           5 | Send on `return_visit`                                                                     |
| About page        |           5 | Send on `about_page_view`                                                                  |
| Service page      |           7 | Send on service page view events                                                           |
| Contact page      |          10 | Send on `contact_page_view`                                                                |
| Brochure download |          15 | Send on `brochure_download`                                                                |
| Calendar click    |          20 | Send on `calendar_click`                                                                   |
| Form submission   |          30 | Send on `form_submit` and `course_enquiry_submit`                                          |

| Segment | Score range | Suggested interpretation                                                   |
| ------- | ----------- | -------------------------------------------------------------------------- |
| Cold    | 0-10        | Low-intent or early awareness traffic; evaluate source quality cautiously  |
| Warm    | 11-30       | Demonstrated trust or service interest; retargeting and nurture candidates |
| Hot     | 31+         | High-intent prospect; prioritise fast response and CRM follow-up           |

## GTM implementation architecture

### Container naming conventions

| Asset type    | Naming convention                       | Example                                                  |
| ------------- | --------------------------------------- | -------------------------------------------------------- |
| Tags          | `GA4 - Event - <event_name>`            | `GA4 - Event - calendar_click`                           |
| Triggers      | `<Trigger Type> - <condition>`          | `Click - Calendar Booking Links`                         |
| Variables     | `DLV - <parameter>` or `JS - <purpose>` | `DLV - service_interest`, `JS - Engagement Temperature`  |
| Lookup tables | `LUT - <mapping purpose>`               | `LUT - Page Path to Service Interest`                    |
| Folders       | `TWC - Engagement Intelligence`         | Store all Phase 1 tags, triggers, and variables together |

### Core GTM variables

| Variable                              | Type                | Purpose                                                     |
| ------------------------------------- | ------------------- | ----------------------------------------------------------- |
| `DLV - event`                         | Data Layer Variable | Reads custom data layer event names                         |
| `DLV - service_interest`              | Data Layer Variable | Reads service taxonomy value                                |
| `DLV - page_type`                     | Data Layer Variable | Reads page category                                         |
| `DLV - content_group`                 | Data Layer Variable | Reads broader content grouping                              |
| `DLV - engagement_score_delta`        | Data Layer Variable | Sends scoring delta to GA4                                  |
| `DLV - resource_name`                 | Data Layer Variable | Sends resource name                                         |
| `DLV - lead_type`                     | Data Layer Variable | Distinguishes contact, booking, course, phone, email        |
| `LUT - Page Path to Service Interest` | Lookup Table        | Maps Nicepage URLs to canonical service values              |
| `LUT - Page Path to Page Type`        | Lookup Table        | Maps URLs to home/about/service/contact/resource            |
| `JS - Returning Visitor`              | Custom JavaScript   | Checks localStorage or cookie visit marker                  |
| `JS - Engagement Temperature`         | Custom JavaScript   | Optional client-side label from current score for debugging |

### Nicepage data layer snippet

Add this before the GTM container code in Nicepage site settings or the global header HTML area. Replace example paths with the live TWC URL structure before publishing.

```html
<script>
  window.dataLayer = window.dataLayer || []
  ;(function () {
    var path = window.location.pathname.toLowerCase()
    var service = "unknown"
    var pageType = "standard"

    if (path === "/" || path === "/index.html") pageType = "home"
    if (path.indexOf("about") > -1) pageType = "about"
    if (path.indexOf("contact") > -1) pageType = "contact"
    if (path.indexOf("mhfa") > -1 || path.indexOf("mental-health-first-aid") > -1) {
      service = "mhfa"
      pageType = "service"
    }
    if (path.indexOf("coaching") > -1 || path.indexOf("executive-coaching") > -1) {
      service = "executive_coaching"
      pageType = "service"
    }
    if (path.indexOf("counselling") > -1 || path.indexOf("counseling") > -1) {
      service = "counselling"
      pageType = "service"
    }
    if (path.indexOf("burnout") > -1) {
      service = "burnout_programs"
      pageType = "service"
    }
    if (path.indexOf("leadership") > -1) {
      service = "leadership_development"
      pageType = "service"
    }
    if (path.indexOf("workplace-mental-health") > -1 || path.indexOf("workplace-training") > -1) {
      service = "workplace_mental_health_training"
      pageType = "service"
    }

    window.dataLayer.push({
      event: "twc_page_context",
      service_interest: service,
      page_type: pageType,
      content_group: pageType === "service" ? "service_pages" : pageType,
    })
  })()
</script>
```

### Trigger architecture

| Trigger                           | Type                                    | Configuration                                                                                   |
| --------------------------------- | --------------------------------------- | ----------------------------------------------------------------------------------------------- |
| `Page View - All Pages`           | Page View                               | Fires GA4 configuration tag on all pages                                                        |
| `Custom Event - twc_page_context` | Custom Event                            | Fires context-dependent page interest events after data layer context exists                    |
| `Scroll - 50 Percent`             | Scroll Depth                            | Vertical depth 50%, once per page                                                               |
| `Scroll - 75 Percent`             | Scroll Depth                            | Vertical depth 75%, once per page                                                               |
| `Timer - 60 Seconds`              | Timer                                   | 60,000 ms interval, limit 1, active on all content pages                                        |
| `Timer - 120 Seconds`             | Timer                                   | 120,000 ms interval, limit 1, active on all content pages                                       |
| `Page View - Returning Visitor`   | Page View or Custom HTML                | Fires when returning visitor variable is true                                                   |
| `Click - Calendar Booking Links`  | Just Links                              | Link URL contains booking provider, calendar URL, Calendly, appointment, or discovery-call path |
| `Click - Brochure Downloads`      | Just Links                              | Link URL ends in `.pdf` or contains brochure/guide/download                                     |
| `Click - Pricing CTAs`            | Just Links                              | Click Text contains pricing, fees, investment, packages, quote, proposal                        |
| `Click - Phone Links`             | Just Links                              | Link URL starts with `tel:`                                                                     |
| `Click - Email Links`             | Just Links                              | Link URL starts with `mailto:`                                                                  |
| `Form - Contact Success`          | Form Submission or custom success event | Fire only on successful submission confirmation, not on button click                            |
| `Form - Course Enquiry Success`   | Form Submission or custom success event | Fire only on MHFA/course form success confirmation                                              |
| `Click - Resource Views`          | Just Links                              | Resource URLs, PDF opens, guide/checklist links, or resource page paths                         |
| `Video - Starts`                  | YouTube Video or custom listener        | Start/progress trigger for embedded videos                                                      |

### Event tag build pattern

Each GA4 event tag should use the GA4 configuration tag or measurement ID variable, set the exact event name, and pass common parameters: `service_interest`, `page_type`, `content_group`, `engagement_score_delta`, `link_text`, `link_url`, `form_id`, `form_name`, and `resource_name` where applicable. Use consent mode and avoid sending names, email addresses, phone numbers, counselling notes, or free-text message contents to GA4.

## GA4 audit checklist

Because this repository does not include live GA4 or GTM container exports, the implementation audit must be performed in GA4 Admin, GTM Preview, Tag Assistant, and DebugView against the live Nicepage site. Use this checklist before publishing the Phase 1 container.

| Audit area           | What to check                                                   | Desired result                                                                                          |
| -------------------- | --------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Existing GA4 tags    | Whether GA4 is installed directly in Nicepage and through GTM   | Keep one implementation path; prefer GTM and remove duplicate direct tags if duplicate page views occur |
| Enhanced measurement | Scrolls, outbound clicks, file downloads, and form interactions | Disable overlapping enhanced events if GTM sends custom equivalents with better parameters              |
| Duplicate page views | Compare DebugView, Realtime, and Tag Assistant hits             | One `page_view` per page load unless SPA routing requires virtual page views                            |
| Lead duplicates      | Form button click and form success events                       | Fire lead events only on success confirmation                                                           |
| Parameters           | Service, page type, source, campaign, link text, form name      | Every interest and lead event carries enough context to analyse service quality                         |
| Key events           | Contact, pricing, brochure, booking, lead events                | Key event list matches the event map and does not include low-intent engagement events                  |
| Consent and PII      | Free-text form fields, user names, emails, phone numbers        | No PII or counselling-sensitive content sent to GA4                                                     |
| Cross-domain         | Booking, Stripe, MailerLite, or calendar domains                | Add referral exclusions or cross-domain settings where journeys should remain connected                 |

## Search Console connection to engagement analysis

Connect Search Console to GA4, then review organic search landing pages in GA4 and Search Console side by side. Use Search Console for query-level visibility and GA4 for engagement and lead outcomes because GA4 does not expose every query directly at the same granularity.

| Analysis                       | Method                                                                                                                | Output                                                                                                |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Queries generating engagement  | Search Console query + landing page, then GA4 landing-page engagement events                                          | Queries whose landing pages produce `scroll_75`, `time_60_sec`, `time_120_sec`, or service page views |
| Queries generating high intent | Search Console landing pages matched to GA4 `contact_page_view`, `calendar_click`, `brochure_download`, `form_submit` | Organic search terms and topics that move beyond reading into action                                  |
| Top-performing landing pages   | Search Console clicks/impressions + GA4 engagement score by landing page                                              | Pages that should be expanded, internally linked, or turned into downloadable resources               |
| Content gaps                   | High impressions with low CTR or engagement                                                                           | Rewrite title/meta, add trust proof, improve above-the-fold CTA, or create better matching content    |

## Google Business Profile reporting design

Track Google Business Profile activity weekly and add UTM parameters to the profile website URL, for example `utm_source=google&utm_medium=organic&utm_campaign=google_business_profile`. If booking links are available in the profile, tag them separately with `utm_campaign=gbp_booking`.

| Metric                     | Report use                                                                 |
| -------------------------- | -------------------------------------------------------------------------- |
| Profile views              | Local awareness and brand discovery                                        |
| Search queries             | Local intent language and service demand                                   |
| Calls                      | High-intent local lead signal; reconcile with `phone_click` where possible |
| Website clicks             | GBP-to-site traffic quality; evaluate engagement score by UTM campaign     |
| Direction requests         | Local demand signal, especially for counselling and in-person services     |
| Photo or post interactions | Content resonance and local credibility                                    |

## Notion CRM database: TWC Engagement & Lead Intelligence

Create a Notion database named **TWC Engagement & Lead Intelligence** with the following fields:

| Field                    | Type         | Notes                                                                                                             |
| ------------------------ | ------------ | ----------------------------------------------------------------------------------------------------------------- |
| Lead Name                | Title        | Person or organisation name; store only with appropriate consent                                                  |
| Organisation             | Text         | Company, school, government agency, or individual                                                                 |
| Lead Source              | Select       | Organic Search, Google Business Profile, LinkedIn, Direct, Referral, Email, Paid Search, Paid Social, Other       |
| Campaign                 | Text         | UTM campaign or manual campaign label                                                                             |
| First Touch              | Text         | First known source/medium/campaign or landing page                                                                |
| Last Touch               | Text         | Most recent source/medium/campaign before enquiry                                                                 |
| Landing Page             | URL          | First known landing page                                                                                          |
| Last Page Before Enquiry | URL          | Page that created the lead action                                                                                 |
| Service Interest         | Multi-select | MHFA, Executive Coaching, Leadership Development, Counselling, Burnout Programs, Workplace Mental Health Training |
| Engagement Score         | Number       | Total score from GA4/Looker/manual calculation at lead creation                                                   |
| Engagement Temperature   | Select       | Cold, Warm, Hot                                                                                                   |
| Lead Status              | Select       | New, Contacted, Discovery Booked, Qualified, Proposal Sent, Won, Lost, Nurture                                    |
| Estimated Value          | Number       | Expected revenue, AUD                                                                                             |
| Revenue Won              | Number       | Actual Stripe/Xero revenue, AUD                                                                                   |
| Probability              | Number       | Pipeline probability percentage                                                                                   |
| Next Action              | Text         | Follow-up task                                                                                                    |
| Next Action Date         | Date         | Follow-up due date                                                                                                |
| Enquiry Type             | Select       | Form, Phone, Email, Booking, Course Enquiry, Referral                                                             |
| GA4 Client ID            | Text         | Optional; only if collected lawfully and not used as PII                                                          |
| Notes                    | Text         | Qualitative lead context; do not sync sensitive counselling details into analytics                                |

## Weekly dashboard design

| Section             | Core widgets                                                                                            | Decision it supports                                             |
| ------------------- | ------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| Awareness           | Users, sessions, source/medium, campaign, Search Console clicks/impressions, GBP views/searches         | Which channels and topics are creating discoverability           |
| Engagement          | Engagement score by source, `scroll_75`, `time_60_sec`, `time_120_sec`, return visits, engaged sessions | Which sources bring attentive visitors                           |
| Interest            | Service page views, about/contact views, brochure downloads, pricing clicks, calendar clicks by service | Which services are building trust and high-intent behaviour      |
| Lead Generation     | Form submits, course enquiries, phone clicks, email clicks, booking clicks by source and service        | Which channels and pages generate enquiries                      |
| Pipeline            | Notion lead status counts, estimated value by service, discovery booked rate, proposal rate             | Which enquiries are commercially meaningful                      |
| Revenue Attribution | Won revenue by first touch, last touch, service, campaign, and lead source                              | Which channels and services produce revenue rather than activity |

## Testing framework

| Test                     | Tool                            | Pass condition                                                                                             |
| ------------------------ | ------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| GTM Preview smoke test   | GTM Preview and Tag Assistant   | Each event fires once with correct parameters                                                              |
| GA4 DebugView test       | GA4 DebugView                   | Events appear within seconds with service and page context                                                 |
| Duplicate page view test | Tag Assistant                   | One page view per load and no parallel hard-coded GA4 tag duplicates                                       |
| Form success test        | Live or staging form submission | `form_submit` fires only after confirmed success                                                           |
| Course enquiry test      | Live or staging MHFA form       | `course_enquiry_submit` fires only after confirmed success and carries `service_interest=mhfa`             |
| Phone/email click test   | Browser + DebugView             | `phone_click` and `email_click` fire on `tel:` and `mailto:` links                                         |
| Brochure/calendar test   | Browser + DebugView             | Download and booking interactions are captured with URL and service context                                |
| Search/GBP UTM test      | URL Builder and Realtime        | UTM-tagged visits appear under intended source, medium, and campaign                                       |
| CRM reconciliation test  | Notion manual entry             | A hot lead can be traced from GA4 event path to Notion source, status, estimated value, and revenue fields |

## Future automation opportunities

1. Push form submissions from Nicepage or form provider into Notion with UTM, landing page, last page, and service interest.
2. Sync MailerLite subscriber tags from `service_interest`, `lead_type`, and engagement temperature.
3. Import Stripe and Xero revenue into the Notion database to connect won revenue to first touch and last touch.
4. Export GA4 BigQuery data for robust user journey and engagement scoring beyond GA4 interface limits.
5. Build LinkedIn campaign UTM governance and compare LinkedIn engagement to organic search and GBP quality.
6. Create automated weekly alerts for hot leads, high-intent organic landing pages, and pages with declining engagement.
7. Add server-side GTM or Measurement Protocol only after Phase 1 browser-side data quality is proven.

## Recommended implementation priorities

1. Finalise URL-to-service taxonomy and clean UTM naming before changing GTM.
2. Audit the live GA4/GTM/Nicepage installation for duplicate tags, enhanced measurement overlap, and missing parameters.
3. Add the Nicepage page context data layer snippet and validate that every page has `service_interest`, `page_type`, and `content_group`.
4. Build GTM variables, triggers, and GA4 event tags in the `TWC - Engagement Intelligence` folder.
5. Mark the recommended contact, pricing, brochure, booking, and lead events as GA4 Key Events.
6. Create GA4 custom dimensions and the `engagement_score_delta` custom metric.
7. Test all events in GTM Preview, Tag Assistant, and GA4 DebugView before publishing.
8. Build the Notion **TWC Engagement & Lead Intelligence** database and start manual weekly lead reconciliation.
9. Connect Search Console and GBP reporting into the weekly dashboard.
10. Review the first four weeks of data, then decide which automations should move into Phase 2.
