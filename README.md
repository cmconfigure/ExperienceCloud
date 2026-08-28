# cmconfigure.com

Static marketing site for CM | Configure, plus a click-through prototype of the Salesforce
Experience Cloud demo portal it links to.

Both HTML files are **fully self-contained** — no build step, no CDN, no dependencies. Fonts,
stylesheets, logo assets and the JS runtime are inlined. Commit and serve.

```
deploy/
├── index.html     cmconfigure.com — marketing site (hero, services, case studies, bio, access form)
├── portal.html    demo portal prototype (intake flow, fee calculator, defendant merge, CMO tracker,
│                  reports, data model explorer)
├── assets/        logo lockups loaded at runtime by the footer (keep alongside the HTML)
├── CNAME          custom domain for GitHub Pages
└── .nojekyll      stops Jekyll from touching the files
```

---

## 1. Deploy to GitHub Pages

```bash
gh repo create cmconfigure --public --clone
cd cmconfigure
cp -r /path/to/deploy/. .
git add -A && git commit -m "CM Configure site + demo portal prototype"
git push -u origin main
gh api -X POST repos/:owner/cmconfigure/pages -f source[branch]=main -f source[path]=/
```

Or in the UI: **Settings → Pages → Source: main / (root)**.

## 2. Point cmconfigure.com at it

At your registrar, create these records:

| Type  | Name | Value                    |
|-------|------|--------------------------|
| A     | @    | 185.199.108.153          |
| A     | @    | 185.199.109.153          |
| A     | @    | 185.199.110.153          |
| A     | @    | 185.199.111.153          |
| CNAME | www  | `<user>.github.io`       |

Then **Settings → Pages → Custom domain** → `cmconfigure.com` → wait for the cert → check
**Enforce HTTPS**. The `CNAME` file in this repo already declares the domain, so Pages will
pick it up on first deploy.

---

## 3. Swap the prototype for the real Experience Cloud site

`portal.html` is a **prototype** — it demonstrates the six artifacts with placeholder data and no
Salesforce behind it. Once the Experience Cloud site is live in your Dev Edition org, replace the
prototype link with the real thing.

**Link out** (safest, works immediately) — in `index.html`, change the hero and portal-section
hrefs from `portal.html` to your site URL and add `target="_blank"`:

```html
<a href="https://cmconfigure-demo.my.site.com/portal" target="_blank" rel="noopener">
  Enter the Demo Portal →
</a>
```

**Embed** (the "both" approach — key demos framed in the page, the rest linked):

```html
<iframe
  src="https://cmconfigure-demo.my.site.com/portal/s/intake"
  title="Litify intake questionnaire"
  style="width:100%;height:900px;border:1px solid #E3E7ED;border-radius:4px"
  loading="lazy"></iframe>
```

> **Experience Cloud blocks framing by default.** Before the iframe will render you must, in Setup:
> - **Digital Experiences → All Sites → Builder → Settings → Security & Privacy**: set clickjack
>   protection to *Allow framing by any third party* (or, better, *Allow framing of site pages on
>   external domains* and whitelist `https://cmconfigure.com`).
> - Add `https://cmconfigure.com` to **CSP Trusted Sites** if any embedded component calls out.
> - Keep **Guest user access** read-only and enable **Guest user sharing rules** only for the
>   curated demo dataset.
>
> Also note Experience Cloud iframes and Salesforce session cookies do not always survive
> third-party-cookie blocking in Safari. If reviewers hit a blank frame, fall back to link-out for
> authenticated pages and embed only guest-accessible ones.

---

## 4. What the portal prototype specifies

Treat `portal.html` as the **functional spec** for the org build. Each view carries an *Under the
hood* panel naming the objects, automation and components involved. Summary:

| View | Salesforce artifact | Notes |
|---|---|---|
| Intake questionnaire | Screen Flow `Litify_Intake_Screening` | 4 practice-area branches; question sets in custom metadata; decision element stores an eligibility **reason string**, not just a verdict |
| Fee Split Calculator | LWC + Apex | Referral fee comes out of the attorney fee, not gross; one `Fee_Split__c` record written per line on approval |
| Defendant directory | LWC + Apex | Normalized-name formula + fuzzy match; merge keeps all aliases on the survivor and self-lookups retired records |
| CMO deadline tracker | LWC + scheduled Flow | Deadlines modeled per product and joined to matters, not duplicated per matter |
| Reports & dashboards | Reports + dashboard components | Settlement by practice area, retained cases by intake source, matter velocity by stage |
| Data model explorer | — | Documents 7 objects across grouping / detail / calculation layers |

Objects referenced: `Primary_Matter__c`, `Property_Location__c`, `Defendant__c`,
`Defendant_Settlement__c` (junction), `Fee_Split__c`, `CMO_Deadline__c`, plus standard Litify
`Matter` left unmodified so package upgrades stay safe.

---

## 5. Editing the site

These files are compiled output. The editable sources are the Design Components in the parent
project (`CM Configure Site.dc.html`, `Demo Portal.dc.html`) — edit there and re-export rather than
hand-patching a 1.5 MB bundle.

If you'd rather own the source here, ask Claude Code to extract the markup into a normal
`index.html` + `styles.css` pair. The design tokens to preserve:

- Navy `#0F2749` · Teal `#2A7D7A` · Gold `#C8A66B` (accent only, never a large fill) · Charcoal `#333`
- Backgrounds alternate white → `#F2F4F7` → `#FAF8F5` → navy; hairline `#E3E7ED`
- Playfair Display 600 for headings, Fira Sans Extra Condensed for everything else
- Radii 2–4px; 1200px container; 96px section rhythm; transitions 120/200ms on `cubic-bezier(.2,.6,.2,1)`

---

## Before this goes public

- [ ] Replace `[LinkedIn]` placeholders with the real URL
- [ ] Confirm the access-form copy matches how you actually configure sharing (it currently promises
      read-only access, 30-day expiry, no client data)
- [ ] Wire the request-access form to a real endpoint — it currently only shows a success state.
      Formspree, a Salesforce Web-to-Lead endpoint, or a Litify intake Screen Flow all work
- [ ] Confirm the dashboard figures are labelled as illustrative wherever they appear
