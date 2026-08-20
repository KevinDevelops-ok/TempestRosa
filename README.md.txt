# Tempest Rosa

Independent digital studio website. Static HTML/CSS/JS, no framework, no build step — deployable directly on GitHub Pages.

## Structure

```
/
├── index.html
├── style.css
├── script.js
├── assets/
│   ├── images/               (empty — add project imagery here as it's produced)
│   ├── videos/                (empty — reserved; avoid large background video per brief)
│   └── logo/
│       ├── mainlogo.png            — colour T·R·rose emblem, transparent, square-cropped
│       │                              (nav, entry screen, hero signature element)
│       ├── mainlogo-nav@2x.png     — small retina copy of the emblem for the nav bar
│       ├── tempest.png             — full "TEMPEST ROSA" wordmark lockup, transparent,
│       │                              dark-background version (footer)
│       └── favicon.png / favicon.ico
└── README.md
```

Both logo files are pre-processed (black background removed → real transparency, cropped square/tight, optimized) so they drop straight in without any editing.

**If a logo ever looks stretched or shows a solid black box behind it:** the file being served isn't the one from this folder — a re-exported screenshot or a copy taken from somewhere other than `assets/logo/` in this project will have the wrong aspect ratio and a solid (non-transparent) background. Re-copy `mainlogo.png` / `tempest.png` from here rather than re-saving them yourself, and the display will correct itself. The CSS also now pins the correct aspect ratio on every logo placement as a safety net, so a wrong file will show clearly wrong (letterboxed) rather than silently stretching.

## Deploying to GitHub Pages

1. Push this folder to a repo (e.g. `tempest-rosa`).
2. In the repo's Settings → Pages, set the source to the `main` branch, root folder.
3. Site will be live at `https://<username>.github.io/tempest-rosa/`.

## Connecting the project-intake form (5 minutes, no code) — write your own email

The "JOIN THE VERSE" form sends a **real email you compose yourself** — it's wired for [EmailJS](https://emailjs.com), a free service built for static sites, chosen specifically because it lets you write the actual email (subject, greeting, layout, sign-off) instead of relaying a fixed default format.

Two things people usually ask about first:

- **"Which email receives it?"** → that's the **To Email** field on the Template (step 3 below) — type your own address there. It's separate from the account you connect in step 2, so you can send *from* one inbox and deliver *to* a different one if you want.
- **"Where do I pick the draft?"** → when you click **Create New Template** (step 3), EmailJS shows you a gallery of starter drafts to pick from (or a blank one) — pick one, then the whole thing is just a text editor where you rewrite it into your own words.

Full steps:

1. Go to **emailjs.com** and create a free account.
2. **Connect the inbox you'll send from:** sidebar → **Email Services** → **Add New Service** → pick Gmail (or Outlook, Yahoo, custom SMTP) → **Connect Account** and sign into that inbox → **Create Service**. This gives you a **Service ID**. (This is just the "from" account — not where submissions land.)
3. **Write the email itself:** sidebar → **Email Templates** → **Create New Template**. EmailJS shows you a few starter drafts — pick one close to what you want, or start blank. You'll land on an editor with these fields:
   - **Subject** and the **body** — write these yourself, in your own words. Drop the visitor's answers in wherever you like using these merge fields:
     ```
     {{name}}         {{contact}}      {{what}}
     {{description}}  {{references}}  {{budget}}  {{category}}
     ```
   - **To Email** — type **your own email address** here. This is the actual answer to "which email should the website use" — whatever you put here is where every submission arrives.
   - **Reply To** (optional but recommended) — set this to `{{contact}}` so that when you hit "Reply" on the notification, it goes straight to the visitor, not back to yourself.

   A simple starting draft, if you'd rather not use one of theirs:

   ```
   Subject: New inquiry — {{what}}

   You've got a new project inquiry through Tempest Rosa.

   From: {{name}} ({{contact}})
   Wants to create: {{what}}  [category: {{category}}]

   Description:
   {{description}}

   References/inspiration: {{references}}
   Budget: {{budget}}
   ```

   Rewrite that however you like — it's your email now. Save the template to get a **Template ID**.
4. Go to **Account → API Keys** and copy your **Public Key**.
5. Open `index.html`, find `<form id="intakeForm" ... data-emailjs-service-id="YOUR_SERVICE_ID" data-emailjs-template-id="YOUR_TEMPLATE_ID" data-emailjs-public-key="YOUR_PUBLIC_KEY">`, and replace all three placeholder values with what you copied.
6. Push the change and submit the form once yourself to test. Every future submission arrives at the address you put in **To Email**, formatted exactly the way you wrote the template.

Until step 5 is done, the form still works end-to-end in the browser (loading state, success message) but logs a warning to the browser console instead of sending anywhere, so it's obvious setup isn't finished rather than silently losing submissions.

A basic honeypot field (`_gotcha`) filters out most bot spam automatically — no action needed there. EmailJS's free plan covers 200 emails/month, which is generous for an intake form.

If you'd rather use something else (Formspree, a custom API), the submit logic is isolated in the `initIntake()` function in `script.js` — replace the `emailjs.send()` call with your own.

## Locking EmailJS down to your domain only

Your EmailJS Public Key, Service ID, and Template ID are visible in `index.html`'s page source — that's normal and by design for EmailJS (the key is meant to be public, unlike a real secret). But **by default**, anyone who copies those three values could send email through your template from their own site, using your quota. One dashboard setting closes that off completely:

1. Go to your EmailJS dashboard → **Account → Security**.
2. Find **Allowed Origins** (sometimes labelled **Allowed Domains** or **API Restrictions** depending on the current dashboard version).
3. Add your real site's domain — e.g. `https://<your-username>.github.io` (and your custom domain too, if you add one later). Include `http://localhost:*` temporarily if you want to keep testing locally.
4. Save.

From that point on, EmailJS checks where each request actually came from and silently rejects anything not on that list — so even if someone copies your key, service ID, and template ID directly out of your page source, it does nothing for them anywhere else. This takes about a minute and is worth doing as soon as your form is otherwise working.

## Notes

- Colors, type (Fraunces / Manrope / Space Mono), and motion are all defined as CSS custom properties at the top of `style.css` — change tokens there rather than hunting through selectors.
- Reduced-motion, keyboard focus states, and IntersectionObserver-based scroll reveals are already in place.
