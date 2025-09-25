
### Claude Code Prompt

You are updating the service page at /services/rails_support_desk/

I want to **add new content sections** for:

1. **Rails Emergency Help Pricing (flat-rate, one-off incidents)**
2. **How It Works (3 steps)**
3. **Why Rails Fever (Trust Builder)**
4. **FAQ (Rails Emergency Help)**


The design should **match the existing page styling** — headings, text hierarchy, and spacing. Use the same HTML/CSS classes already in use for sections, headings, pricing tables, and accordions (if FAQs are accordion style).

---

### Content to Add

#### 🚨 Pricing Section (One-Off Incidents)

**Heading:** Rails Emergency Help — Flat Rates

**Pricing Table:** Three simple tiers, side by side, styled like the site’s existing pricing/service cards.

* **Minor Incident — \$3,000**
  *Clear root cause, resolved same day.*
  Examples: expired SSL, background jobs stuck, blocked deployment.

* **Major Incident — \$6,000**
  *Multi-layer failures, up to 2 days to stabilize.*
  Examples: database issues, broken pipeline, severe performance collapse.

* **Critical Incident — \$12,000**
  *Catastrophic failures requiring full recovery.*
  Examples: app offline, corrupted data, security breach.


** After the table **
* Add a small note that customers should expect a surcharge of $1000 - $2000 (tier based) if they need nights or weekend response fix.

---

#### Add a section titled: How It Works

Display three steps side by side (stacked on mobile), styled like numbered cards:

- Tell us what’s happening,
  Send us an email at hello@railsfever.com or schedule a call (include calender scheduling link)

  Confirm flat-rate tier
  We diagnose the issue immediately and confirm whether it’s Minor, Major, or Critical.

  We jump in immediately
  Once confirmed, we start within 2 hours and work until your app is stable.

---

#### Add a section titled: Why Rails Fever?

This should be a trust-building block with a short intro sentence and bullet points.

Intro text:

When your business is at risk, you need a team you can rely on. Rails Fever brings deep Rails expertise and clear communication in critical moments.

Bullets (use checkmark icons if available in the design system):

12+ years of Rails consulting and DevOps experience

Transparent flat-rate pricing — no hourly surprises

Immediate response, typically within 2 hours

Proven track record of rescuing SaaS apps in production emergencies

Add a short testimonial or client win (“Rails Fever brought our app back online in hours when we thought it was lost”).

---

#### ❓ FAQ Section

Add a section titled: **FAQ — Rails Emergency Help**

Include these Q\&A items, styled as accordions or expandable items (depending on the site’s convention):

1. **How do I know which tier my incident falls into?**
   After a quick intake call, we’ll tell you whether it’s Minor (\$3K), Major (\$6K), or Critical (\$12K). Most emergencies fall into the Major tier. You’ll know the cost upfront — no surprises.

2. **What if my issue takes longer than expected?**
   Flat-rate pricing means we stay on it until the system is stable, even if it takes longer than usual. We don’t bill by the hour.

3. **Do you guarantee a fix?**
   We guarantee **dedicated emergency response** and will get your Rails app back online or stable. In rare cases where an underlying third-party service is at fault (e.g. AWS outage), we’ll stabilize your system as much as possible and provide workarounds until the provider resolves the issue.

4. **How fast will you respond?**
   We respond immediately. Typically within **2 hours or less** once payment and intake are complete.

5. **What if my incident happens overnight or on a weekend?**
   We handle off-hours incidents too. Expect a surcharge of $1k - $2k (tier based) for nights/weekends since it pulls us out of bed.

6. **Do you need access to my code and servers?**
   Yes. We’ll walk you through granting secure, temporary access (GitHub, hosting, database). Everything is kept confidential and removed once the incident is resolved.

7. **What if my app is in really bad shape?**
   That’s what we’re here for. Even if your app has years of tech debt, we’ll stabilize it first. Then, if you want, we’ll recommend next steps to prevent another meltdown.

---

### Instructions for Claude

* Keep the markup consistent with the **existing Rails Fever site’s design system**.
* Use the same **section wrappers, heading sizes, and card/accordion components** already in place.
* The pricing should appear as a **three-column layout** on desktop, stacked on mobile.
* Make the “How It Works” steps visual cards or numbered items.
* The FAQ should be **click-to-expand accordions** (or plain Q\&A blocks if the site doesn’t use accordions).

---

