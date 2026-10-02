# Voltizone: conversation recap (2026-10-02)

A summary of everything discussed, in order, plus talking points for the client meeting.

**How to come back to this**
- **Easiest:** reopen the same session at claude.ai/code, or from the Code tab in the Claude app. The whole conversation is still there: https://claude.ai/code/session_01QW91euETdaqd1bLTqFA8JS
- **Or start a new session** on the Clientsolution repo. Claude reads `CLAUDE.md` automatically, so just say "let's continue the Voltizone project."

---

## 1. The original question

- **Client:** Voltizone, a Ninja Warrior / parkour gym with two locations in Quebec (Mascouche and Laval).
- **Setup:** the website is built on **Wix**. Bookings and payments go through **Bookeo** (bookeo.com), which the client called "Book.io."
- **Note from the visit:** "Wix build. Not confirming if purchases confirm. They have a linked site to both, it doesn't work."
- **The problem:** they run Meta (Facebook/Instagram) ads. They can see people clicking the ads and visiting the site, but **not whether those people booked and paid**. Wix and Bookeo each blame the other.
- **The question:** is it fixable, and can we offer them a solution?

## 2. What's causing it

- Their booking calendar isn't really part of the Wix page. It's Bookeo's site shown through a frame, and Wix puts every embedded widget inside a second frame of its own (on a Wix domain, `filesusr.com`).
- The Meta pixel on the Wix page can't see inside those frames. Bookeo's pixel ends up firing from inside Wix's frame, cut off from the ad-click information stored on voltizone.com.
- That's why putting the pixel on both Wix and Bookeo didn't work.
- Bookeo's own help article says its Meta tracking needs the calendar placed directly on the page, "without intermediate frames," and that it may not work on Wix for this reason. Yet Bookeo's own Wix guide says to use Wix's "Embed HTML," which creates that frame.
- Each company's part works as designed; the break is between them, which is why they blame each other.

```
Meta ad → voltizone.com (Wix pixel ✅ sees the visit)
           └─ Wix frame (filesusr.com)
                └─ Bookeo frame (bookeo.com) → customer pays ✅
                                               → Meta never links it to the ad ❌
```

*Not yet checked on the live site:* the Claude workspace couldn't reach voltizone.com (network setting, see section 7), so this comes from Bookeo's and Wix's documentation.

## 3. First solution offered: send bookings straight from Bookeo to Meta (server-side)

- When a booking is confirmed in Bookeo, it's sent directly to Meta's Conversions API as a "Purchase," with the amount and the customer's hashed email and phone.
- **Ways to build it:** with Zapier (no-code), or as a small custom connector.
- **Your reaction:** too complex for the client, and not 100% foolproof. It's parked as an optional add-on for later.

## 4. Your idea: build them a new site and connect Bookeo properly

- **Keep Bookeo exactly as is.** It's closer to Shopify's checkout than to Stripe: the calendar, checkout page and confirmation emails. Card payments run through a processor connected inside Bookeo (Stripe, Square or PayPal). The website never touches payments.
- **A new site fixes it**, as long as Bookeo's calendar is placed directly on the page. Linking out to a bookeo.com page brings the same problem back.
- **The cookie banner** is required by Quebec's Law 25: tracking must stay off until the visitor accepts. It asks permission; it doesn't make tracking always work. People who decline can't be tracked.
- **Smaller option raised:** you don't need a whole new site to fix the tracking. Booking pages alone are enough.

## 5. What a full rebuild would involve

- **Rankings** stay with the domain and the page addresses. Keep voltizone.com, keep the same addresses or add permanent (301) redirects, copy page titles and descriptions, and submit the sitemap in Google Search Console. Expect a few weeks of small ups and downs.
- **Email traps:** they use ninja@voltizone.com and ninjalaval@voltizone.com, so the email settings (MX records) must be kept when switching the domain.
- **Other work:** the client loses the Wix editor, and forms, contacts and Wix apps all need replacing. Check where the domain is managed, and don't cancel Wix until the new site is live. Keep it French-first.
- **Your conclusion:** too many changes, so not the best option.

## 6. Decision: booking pages only

**What they are**
- A mini website, one page per location (Mascouche and Laval), at `book.voltizone.com`.
- Each page has the Bookeo calendar placed directly on it, plus their logo, colours and menu.
- It must be on their own domain. That's what lets the ad-click information carry over, and on a different domain tracking breaks again.

**How people reach them**
- From the "Book now" buttons on their Wix site (same tab recommended), or straight from the Meta ads.
- Customers still pick a time and pay inside Bookeo, exactly like today.

```
Meta ad ───────────┐
                   ├──► book.voltizone.com → pick a time → pay → "Purchase" shows in Meta ✅
Wix "Book now" ────┘    (Bookeo calendar)
```

**How accurate it is**
- **Counted:** most online bookings from people who accept cookies.
- **Not counted:** people who reject cookies or use ad blockers, and bookings made by phone or in person.
- **Sometimes missed:**
  - Someone clicks the ad on their phone but books on a computer.
  - Someone books more than 7 days after clicking (Meta's default window).
- Today they see none, so this is a big jump.

**There's no single line of code.** The fix is the pages plus four settings:
1. **Domain:** add one record so `book.voltizone.com` opens the new pages. Email and the main site aren't touched.
2. **Bookeo, each location:** point the website setting at the new page and switch on the Meta pixel. This is also where we get the calendar code.
3. **Wix:** change the "Book now" buttons to link to the new pages.
4. **Meta:** confirm a test booking shows up as a "Purchase."

**Who does what**
- Claude builds the pages.
- The agency hosts them, cheaply or for free (Cloudflare Pages or Netlify).
- Ask the client for access invites, not passwords.

**How it meets your must-haves**
- **Safe:**
  - Payments stay in Bookeo's secure checkout, and card details never touch our page.
  - The page stores nothing (no logins, customer data or database) and runs on a padlocked https address.
  - Their main site, email and Bookeo account stay as they are, and access is by invite and can be removed any time.
- **Lives inside their site:** it's on their own domain with their branding, opening in the same tab.
- **Easy to use:**
  - Customers get the same Bookeo booking they use now. It may work better on phones, because Bookeo no longer has to squeeze inside Wix's frame.
  - Staff have nothing new to learn, since schedules and prices still live in Bookeo.
- **Tracks most sales:** this is the setup Bookeo itself says Meta tracking needs. The only limit is people who decline cookies or block ads.

**Prove it before going live**
- Build a test page first.
- Make one real booking, then refund it, and check that "Purchase" shows up in Meta.
- Only then switch the "Book now" buttons. Their live site doesn't change until it works.

## 7. Claude workspace housekeeping

- **Network access:** the Claude workspace couldn't reach voltizone.com or bookeo.com. To fix it, on a computer:
  1. Go to claude.ai/code and click the cloud icon showing the environment name, just above the message box.
  2. Go to **Cloud**, hover over the environment and click the gear icon.
  3. Set **Network access** to **Full**, or to **Custom** with these domains, and tick "Also include default list of common package managers":
     `voltizone.com`, `*.voltizone.com`, `*.wixstatic.com`, `*.filesusr.com`, `bookeo.com`, `*.bookeo.com`
  - The change reaches the running session within about a minute, so no new session is needed.
  - The phone app may not show these settings; Claude's docs only describe doing it in a browser or the Desktop app.
  - **Still blocked at the end of this session.**
- **Notes:** saved to this repo. `CLAUDE.md` has the key facts and is read automatically by new sessions; this file is the full recap.

---

## For your client meeting

### What to tell them, in plain words

- **Why it's broken:** Wix puts the Bookeo calendar inside its own frame, and Bookeo's tracking only works when the calendar sits directly on the page. Each company's part works; the gap is between them, which is why they keep blaming each other.
- **The fix:** a booking page for each location on `book.voltizone.com`, with the Bookeo calendar placed directly on it. Bookeo, payments and their Wix site stay the same; only the "Book now" buttons change.
- **What they get:** Meta shows which ads led to bookings and how much revenue they brought in. That covers most online bookings from people who accept cookies.
- **Proven first:** we test with a real booking before anything changes on their live site.
- **Privacy:** we add a cookie banner, which Quebec's Law 25 requires.

### Questions to ask them

- **Bookeo accounts:** is there one Bookeo account per location, or one for both?
- **Payments:** which payment processor is connected in Bookeo (Stripe, Square, PayPal)?
- **Domain:** where was voltizone.com bought (Wix, GoDaddy, other), and who manages it?
- **Meta:** who manages their Meta Business account and pixel?
- **Cookie banner:** do they have one on the Wix site today?
- **Other systems:** do they sell anything outside Bookeo, for example through WellnessLiving (listed for Laval)?
- **Ads:** which offers do their ads push most (birthday parties, free time, classes)? That decides where each ad should point.

### Access to request (invites, not passwords)

- **Wix:** invite you as a collaborator (Settings → Roles & Permissions).
- **Bookeo:** add you as a user on each location's account.
- **Meta:** give your business partner access to the pixel and ad account (Business Settings → Partners).
- **Domain:** access to the DNS settings, or they add the one `book` record themselves using instructions we provide.

---

## Sources

- [Bookeo: Meta pixel tracking](https://support.bookeo.com/hc/en-us/articles/360017923772-How-can-I-create-a-Facebook-conversion-tracking-pixel-on-Bookeo)
- [Bookeo: Wix integration](https://support.bookeo.com/hc/en-us/articles/360018197571-Integration-in-a-Wix-com-website)
- [Bookeo: tracking problems from Wix's filesusr.com frame](https://support.bookeo.com/hc/en-us/articles/360018201631-Google-Analytics-is-not-tracking-traffic-sources-conversions-correctly)
- [Bookeo: payment gateways](https://support.bookeo.com/hc/en-us/articles/360023559032-Payment-gateways-supported-by-Bookeo-online-payments)
- [Bookeo: booking source tracking (`?source=`)](https://support.bookeo.com/hc/en-us/articles/360017919212-Can-I-track-the-source-of-bookings-in-Bookeo)
- [Bookeo: API webhooks](https://www.bookeo.com/api/webhooks/)
- [IRL Experience Design: Bookeo links vs. embedded widget](https://irlxd.com/bookeo-and-google-analytics-tracking-take-2/)
- [Meta: Conversions API with Zapier](https://developers.facebook.com/documentation/ads-commerce/conversions-api/guides/zapier-integration)
- [McCarthy Tétrault: Quebec's Law 25 and cookies](https://www.mccarthy.ca/en/insights/blogs/techlex/quebecs-law-25-and-cookies-not-so-cookie-cutter)
- [Claude Code docs: cloud environments and network access](https://code.claude.com/docs/en/cloud-environments)
