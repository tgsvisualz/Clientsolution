# Client context: Voltizone (Meta ads purchase tracking)

Notes carried over from an earlier session (2026-10-02). Read this before working on the Voltizone project.

## The client

- **Voltizone**: Ninja Warrior / parkour gym with two locations in Quebec: **Mascouche** and **Laval**.
- Website: https://www.voltizone.com, built on **Wix**, French-language.
- Booking and payments: **Bookeo** (bookeo.com). The user sometimes calls it "Book.io"; it is Bookeo.
  - Bookeo is embedded on two Wix pages: `/page-bookeo` ("Bookeo ninja Mascouche") and `/copie-de-bookeo-ninja-laval` ("Bookeo parkour Laval"). Possibly one Bookeo account per location; not confirmed.
  - Payments run through a processor connected inside Bookeo (Bookeo supports Stripe, Square, PayPal and others).
- Voltizone also has listings on WellnessLiving (Laval) and ClassPass. Still need to ask whether anything is sold through WellnessLiving.
- They run Meta (Facebook/Instagram) ads. They can see ad clicks and site visits, but **cannot see whether people who clicked an ad went on to book and pay**. Wix and Bookeo each blame the other.

## Diagnosis

- Wix's "Embed HTML" element puts the Bookeo widget inside a Wix-owned iframe on `filesusr.com`. The widget then loads its own `bookeo.com` iframe inside that.
- Bookeo's Meta pixel help article says the widget must be inserted directly in the page "without intermediate frames," and that tracking may not work on Wix for exactly this reason. Bookeo's own Wix setup guide still tells you to use Embed HTML.
- Result: the Wix pixel sees visits, but Bookeo's Purchase event is cut off from the ad click, so no purchases are attributed to the ads.
- **Not yet verified on the live site.** The environment's network policy blocked voltizone.com and bookeo.com, so this diagnosis comes from Bookeo and Wix docs plus search results.

## Options discussed

1. **Server-side feed:** Bookeo (Zapier "New booking" trigger, or Bookeo API webhooks) sends a Purchase event to the Meta Conversions API. It's robust, but the user felt it was too complex for the client. Keep it as an optional later add-on.
2. **Chosen direction: build our own page(s) or site with the Bookeo widget code directly in the page (no iframe).**
   - Then turn on Bookeo's built-in Meta pixel (Bookeo: Marketing → Conversion tracking and analytics → Facebook Pixel ID).
   - Use the same pixel ID as the site, for both locations.
   - Bookeo stays as-is for booking and payments; nothing changes for the client's staff.
   - Bookeo fires these events: AddToCart, AddPaymentInfo, InitiateCheckout and Purchase (with service name, fee and SKU).
   - **Smallest version:** booking pages on a subdomain such as `book.voltizone.com`, one per location, with Wix "Book" buttons pointing there. Meta pixel cookies (`_fbp`/`_fbc`) are set on `.voltizone.com`, so the ad-click data carries over.
   - **Bigger version:** a full new site. Redirect the old URLs to keep Google rankings, and keep the content French-first.
   - **Rule:** embed the widget, don't link out to bookeo.com hosted pages. Bookeo only tracks properly when the widget is on your own site.

## Requirements and caveats

- **Quebec Law 25:** tracking must be off by default until the visitor opts in. That means a cookie consent banner that blocks the Meta pixel until accepted. Verify that Bookeo's widget tracking also waits for consent, and update the privacy policy.
- **Not 100%:** people who decline cookies, use ad blockers, or have some iPhone privacy settings won't be counted. The goal is to go from seeing zero bookings to seeing most bookings from people who accept.
- **Testing:** make a real booking (then refund it), confirm "Purchase" appears in Meta Events Manager under Test events, and check with Meta Pixel Helper.
- **Quick reporting trick:** Bookeo supports `?source=` on booking links, and the value shows in Bookeo's Bookings report.

## If we rebuild the full site (migration notes)

- Wix can't export a site. A rebuild means copying the content (text, and photos from `static.wixstatic.com`) and recreating the design.
- **Rankings** belong to the domain and the page addresses. To keep them:
  - Keep `voltizone.com`.
  - Keep the same page addresses where possible, and add 301 redirects for any that change (for example `/copie-de-bookeo-ninja-laval`).
  - Copy each page's title and meta description.
  - Submit the new sitemap in Google Search Console after launch.
  - Expect a few weeks of small ranking fluctuation.
- **Email runs on the domain:** `ninja@voltizone.com` (Mascouche) and `ninjalaval@voltizone.com` (Laval). Keep the MX records when changing DNS.
- **Domain:** check where it's registered (Wix or elsewhere). Don't cancel Wix until the new site is live and verified.
- **Wix extras to replace:** forms, contacts (export them first), newsletters and any Wix apps.
- **Editing:** the client edits the site themselves on Wix today. A custom site needs a simple editor, or a maintenance plan where we make the changes.
- **Language:** keep the site French-first (Quebec language law).
- **Next step once network access works:** crawl `sitemap.xml` and produce a page-by-page plan (keep / redirect / replace) to show the client.

## Network access (this environment)

- `voltizone.com`, `bookeo.com` and `support.bookeo.com` were blocked by the environment's network policy.
- **Fix:**
  - At claude.ai/code, click the cloud icon showing the environment name, just above the message box.
  - Go to **Cloud**, hover over the environment, then click the gear icon.
  - Set **Network access** to **Full**, or to **Custom** with these allowed domains: `voltizone.com`, `*.voltizone.com`, `*.wixstatic.com`, `*.filesusr.com`, `bookeo.com`, `*.bookeo.com`. With Custom, also tick "Also include default list of common package managers".
- Changes reach running sessions within about a minute.

## Open questions / next steps

- **Current direction: booking pages only.** The user stepped back from a full rebuild because it involves too many changes.
  - Build two small booking pages (Mascouche and Laval) on `book.voltizone.com`, with the Bookeo widget directly on each page.
  - Keep the Wix site as-is, and point its "Book now" buttons (and optionally the ads) to the new pages.
  - Host the pages on the agency's account (for example Cloudflare Pages or Netlify).
- **User's must-haves:** safe; lives inside their site (their domain and branding); easy for customers and staff; tracks at least most sales.
- **Plan:** prove it before going live. Build a test page, make a real test booking and refund it, confirm "Purchase" in Meta Events Manager, and only then switch the live buttons.
- **Access:** ask the client for invites, not passwords:
  - Wix collaborator, via Settings → Roles & Permissions.
  - Bookeo user, for both locations.
  - Meta partner access to the pixel and ad account, via Business Settings → Partners.
  - Domain/DNS access to add the `book` record. The client can enter that one record themselves if they prefer.
- Get each location's Bookeo widget code from Bookeo: Settings → Theme and Layout → Website integration.
- **Status at the end of the 2026-10-02 session:** network access was still blocked (voltizone.com not reachable from this environment). The user is meeting the client next.
- **Full conversation recap and client-meeting talking points:** `notes/voltizone-conversation-2026-10-02.md`.

## Sources

- Bookeo, Meta pixel: https://support.bookeo.com/hc/en-us/articles/360017923772-How-can-I-create-a-Facebook-conversion-tracking-pixel-on-Bookeo
- Bookeo, Wix integration: https://support.bookeo.com/hc/en-us/articles/360018197571-Integration-in-a-Wix-com-website
- Bookeo, GA not tracking (filesusr.com frame): https://support.bookeo.com/hc/en-us/articles/360018201631-Google-Analytics-is-not-tracking-traffic-sources-conversions-correctly
- Bookeo, booking source tracking: https://support.bookeo.com/hc/en-us/articles/360017919212-Can-I-track-the-source-of-bookings-in-Bookeo
- Bookeo, payment gateways: https://support.bookeo.com/hc/en-us/articles/360023559032-Payment-gateways-supported-by-Bookeo-online-payments
- Bookeo, API webhooks: https://www.bookeo.com/api/webhooks/
- Meta, Conversions API with Zapier: https://developers.facebook.com/documentation/ads-commerce/conversions-api/guides/zapier-integration
- Quebec Law 25 and cookies: https://www.mccarthy.ca/en/insights/blogs/techlex/quebecs-law-25-and-cookies-not-so-cookie-cutter
