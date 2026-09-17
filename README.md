# RedNeck Lo Mechanics — NFC business card page

A single-page, static site for **RedNeck Lo Mechanics** (James & Misty Combs).
It is the landing page written to the shop's NFC business card tags: tap the
tag, the phone opens this page, and the visitor can call either number or save
the contact.

**Live URL:** https://redneck-lo-mechanics.github.io/redneck-lo-mechanics/

## What's in the repo

| File | Purpose |
| --- | --- |
| `index.html` | The whole page. Self-contained: the card artwork is embedded as a base64 JPEG, with invisible tappable `tel:` links positioned over the printed phone numbers, two call buttons, a **Save Contact** button, and a one-line hint shown only on Android. |
| `contact.vcf` | vCard 3.0 with an embedded photo. CRLF line endings, which iOS requires. |
| `card.jpg` | The same card artwork as a plain file. Only used by the `og:image` / structured-data links in `<head>`, so link previews and Google have an absolute image URL. The page itself still uses the embedded copy. |
| `robots.txt` | Lets every crawler in and points at the sitemap. |
| `sitemap.xml` | One-entry sitemap for the page and its image. Bump `<lastmod>` when the page changes. |
| `.gitattributes` | Marks `*.vcf` as binary so Git never rewrites those CRLF endings. |
| `.nojekyll` | Tells GitHub Pages to serve the files as-is instead of running Jekyll. |
| `LICENSE` | AGPL-3.0. |

Three details are deliberate and should not be "fixed":

- The **Save Contact** link has **no `download` attribute**. Without it, iPhone
  Safari opens the native *Add Contact* sheet. With it, Safari saves the file
  to Files instead, which is worse.
- `contact.vcf` uses **CRLF** line endings, per the vCard spec. LF-only files
  are rejected or mangled by some phones.
- The **tiny script at the bottom of `index.html`** only reveals the Android
  hint under the button. It does not, and cannot, stop Android from downloading
  the vCard. See *Save Contact on Android* below before "improving" it.

## Save Contact on iPhone and Android

The same link behaves differently on the two platforms, and both behaviours are
the best each platform allows.

**iPhone and iPad (Safari).** Tapping **Save Contact** opens the vCard directly
in the native contact sheet: name, both numbers, company, title, note and photo
are shown, with **Create New Contact** and **Add to Existing Contact** buttons.
Nothing is downloaded. This depends on the link having no `download` attribute
and on `contact.vcf` keeping its CRLF line endings. Chrome, Firefox and Edge on
iOS are all Safari underneath and do the same. Opened from the NFC tag, iOS
shows a banner first; tapping it lands on this page in Safari.

**Android (Chrome, Samsung Internet, etc.).** Chrome cannot display a `.vcf`, so
tapping **Save Contact** downloads the file. The phone then shows a download
notice; tapping **Open** on it brings up the "open with" chooser (or goes
straight to Contacts if it is the only handler), and Contacts offers to import
the card, photo included. The page shows a hint under the button on Android
explaining those two taps.

That download step cannot be skipped from a web page. Two workarounds were
tried and removed because neither can work on a stock phone:

- **The Web Share API with the `.vcf` file.** Chrome keeps a fixed allow-list of
  shareable file types (images, audio, video, PDF, CSV, HTML, plain text).
  `.vcf` / `text/vcard` is not on it, so `navigator.canShare()` says no.
- **An `intent:` URL to the Contacts app's "new contact" screen.** Chrome adds
  the `BROWSABLE` category to every intent a web page fires, and the Contacts
  app's insert activity does not declare it, so the intent never resolves and
  Chrome falls back to the plain link, i.e. the same download.

**Calling from Firefox on Android.** Firefox asks "open in app?" before it
hands a `tel:` link to the dialer. If the visitor declines, Firefox stores that
answer for **one hour, per tab, keyed on the `tel:` scheme**, and then silently
ignores every phone link on the page in that tab. Reloading does not clear it;
opening the page in a new tab does. This is Firefox's own do-not-ask cache
(`AppLinksInterceptor.addUserDoNotIntercept` in the Firefox for Android
source), not a bug in the page, and nothing the page does can reset it. Varying
the link does not help because the number is not part of the key. Opening each
call in a new tab would dodge the cache but leaves an empty tab behind on every
call, so it is not done. The page instead shows a one-line hint under the call
buttons on Firefox for Android only. Chrome and Samsung Internet dial without
asking and are unaffected. Both hints key off the browser's user agent, and
"Desktop site" mode replaces the Android token with a generic Linux one, so
the page also treats a touch-capable Linux browser as Android.

The one route that skips the browser entirely on Android is writing a vCard
record to the NFC tag itself, which Android's Contacts app opens directly.
That breaks iPhone tap-to-open (iOS only auto-opens URL records) and the
17 KB card with its photo does not fit on common tags, so it is not done here.

## Local search (SEO)

The page is the business's only page, so it doubles as the contact page and is
what should come up for "mechanic" / "mobile mechanic" searches in the service
area. Everything search engines see lives in `index.html`:

- **`<head>`**: the `<title>`, `description`, `keywords`, `geo.*`, canonical
  link, Open Graph / Twitter tags, and two JSON-LD blocks (`AutoRepair` +
  `LocalBusiness` for the business, `ContactPage` for the page).
- **Visible text**: the `<h1>` / tagline above the card and the service-area
  `<footer>` under the buttons. Google weighs on-page words more than meta tags,
  so those must stay visible.

The business has **no street address yet**, so no address appears anywhere. The
JSON-LD uses `areaServed` instead, which is what Google expects from a
service-area business. When an address exists, add a `PostalAddress` under
`address` in the first JSON-LD block and a `geo.position` / `ICBM` meta pair.

### Service area

Current list, in priority order:

1. Traverse City
2. Grand Traverse County
3. Garfield Township
4. Blair Township
5. Acme Township
6. East Bay Township

### Adding a township

Add the new place in all four spots so they agree:

1. `<meta name="description">` and `<meta name="keywords">` in `<head>`.
2. The `og:description` meta tag.
3. The `areaServed` array in the first JSON-LD block (copy an existing
   `AdministrativeArea` entry and change the name).
4. The visible `<footer class="area">` paragraph near the bottom of the page.

Then bump `<lastmod>` in `sitemap.xml`. After a change, paste the live URL into
Google's Rich Results Test to confirm the structured data still parses.

If the repository is transferred (see below), every absolute URL that starts
with `https://warstorm548.github.io/redneck-lo-mechanics/` in `index.html`,
`robots.txt` and `sitemap.xml` must change to the new address, and the site
should be re-submitted in Google Search Console.

## Changing a phone number

The numbers live in two files and in the card artwork. All three must agree.

### 1. `index.html`

Current numbers: 1st Phone `+1 (231) 383-3786`, 2nd Phone `+1 (231) 383-5325`.
Each appears in `tel:` form (`+12313833786`, digits only, no punctuation) and in
display form (`(231) 383-3786`). Lines as of this writing:

| Line | What it holds |
| --- | --- |
| 118 | `alt` text on the card image — both numbers, display form |
| 119 | 1st Phone hotspot over the artwork — `tel:` link and `aria-label` |
| 120 | 2nd Phone hotspot over the artwork — `tel:` link and `aria-label` |
| 124 | 1st Phone call button — `tel:` link |
| 126 | 1st Phone call button — visible display number |
| 128 | 2nd Phone call button — `tel:` link |
| 130 | 2nd Phone call button — visible display number |

If the line numbers have drifted, search the file for `383-3786` and
`12313833786` (and the same pair for the second number). Every hit must be
updated, including the `description` meta tag and both JSON-LD blocks in
`<head>` (there the numbers are written `+1-231-383-3786`). Do not change
anything else in `index.html`.

### 2. `contact.vcf`

| Line | Content |
| --- | --- |
| 7 | `item1.TEL;TYPE=CELL,VOICE,pref:+12313833786` — 1st Phone |
| 9 | `item2.TEL;TYPE=CELL,VOICE:+12313835325` — 2nd Phone |

Edit only the number after the final colon, in `+1XXXXXXXXXX` form. Leave the
`item1.` / `item2.` prefixes and the `X-ABLabel` lines on 8 and 10 alone — those
are what make the phone show "1st Phone" and "2nd Phone".

**Use an editor that preserves CRLF line endings** (VS Code: the status bar
shows `CRLF`; keep it there). Do not reflow or re-wrap the long `PHOTO:` line
starting on line 12 — its continuation lines begin with a single space and that
folding is part of the format.

### 3. The card artwork

The printed numbers are baked into the base64 JPEG on line 117 of `index.html`.
Changing a number means re-exporting the card image and replacing that data URI,
otherwise the picture and the links disagree.

After any change, open the page on an actual iPhone and an actual Android phone.
Tap both numbers, both call buttons, and Save Contact.

## Turning on GitHub Pages

1. Go to the repository's **Settings** tab.
2. Pick **Pages** in the left sidebar.
3. Under **Build and deployment**, set **Source** to **Deploy from a branch**.
4. Set the branch to **main** and the folder to **/ (root)**.
5. Click **Save**.

The first build takes a minute or two. The Pages panel then shows the live URL.

## Transferring ownership

The repo is expected to move to a different GitHub account. **This changes the
live URL, and NFC tags do not update themselves.** Plan for it.

### Transferring the repository

1. In the repository's **Settings**, scroll to the **Danger Zone** at the bottom.
2. Click **Transfer** (Transfer ownership).
3. Enter the new owner's GitHub username or organization, type the repository
   name to confirm, and complete the transfer.
4. The new owner accepts the transfer from their email or notifications.

### The Pages URL changes and is not redirected

GitHub **does not redirect the old GitHub Pages URL after a transfer.**
`https://warstorm548.github.io/redneck-lo-mechanics/` will stop working. The
site's new address becomes:

```
https://NEWUSER.github.io/redneck-lo-mechanics/
```

where `NEWUSER` is the new owner's GitHub username, lowercased. If the new owner
also renames the repository, the last path segment changes to match.

### New owner's checklist

1. **Confirm Pages is still on.** Open **Settings > Pages** and check that the
   source is still **Deploy from a branch**, branch **main**, folder
   **/ (root)**. Re-enable it if the transfer turned it off.
2. **Load the new URL on a phone** and test both call links and Save Contact.
3. **Rewrite every NFC tag with the new URL.** Any tag still holding the old
   address is dead. Use NFC Tools (iOS/Android) or a similar writer, write the
   new URL as a URI/URL record, and re-test each tag by tapping it. Include any
   printed QR codes that point at the old address.

Rewriting the tags is the step people forget. Do it before handing cards out.
