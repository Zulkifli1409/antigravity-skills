---
name: platform-aware-ui-mobile-web
description: Helps adapt interfaces to the platform they run on, covering desktop to touchscreen porting problems, Android style GUI behaviour such as Back button, dialogs, toasts and hardware keys, plus web and browser realities like web app benefits, security cues in the address bar, cookies and privacy controls, page speed, old browser support and where a web designer fits in the market. Use this when the user asks about mobile UI, Android UI, porting a desktop app to phone or tablet, touch vs mouse, hover and right click on mobile, keyboard input on small screens, responsive vs adaptive, mobile-first, progressive enhancement, web app vs native, login or payment trust cues, website performance for UX, atau karier dan positioning web designer, even if they only say my app is hard to use on a phone.
author: Zulkifli
github: https://github.com/Zulkifli1409
linkedin: https://www.linkedin.com/in/zulkifli1409/
source: GUI Design for Android Apps by Ryan Cohen and Tao Wang (Apress, 2014), chapter 1 fully and chapters 2-4 partly ; 20 Things I Learned About Browsers and the Web by the Google Chrome Team (2010), things 1-20 ; Web Design Is Dead by Ben Hunt (version 1, publisher not stated), opening argument, commoditization and the three designer paths ; UI/UX Design Guide by Joel Parker Henderson (2024), mobile-first, progressive enhancement and graceful degradation topics listed in contents only
---

# Platform-Aware UI for Mobile and Web

Reply in the language the user writes in, go straight to the point, and give concrete changes or a short plan instead of platform history. Most source material here is old (2010 to 2014), so always separate timeless ideas from outdated details, and mark modern additions with (+).

## 1. Choose the track

| Situation | Track |
|---|---|
| Desktop app or game must run on phone or tablet | Section 2 |
| Designing or reviewing an Android-style app flow | Section 3 |
| Website or web app trust, privacy, speed, browser support | Section 4 |
| Freelancer or agency asking where web design work is going | Section 5 |
| Responsive vs adaptive, mobile-first question | Section 2 rules 1 to 3 plus section 4 |

## 2. Porting from desktop to touch devices (timeless core)

The central lesson is that a desktop UI copied as-is becomes unusable on a small touchscreen, because the input device, screen shape and keyboard all change. Redesign for the device, do not shrink.

1. **Screen size and density.** Shrinking proportionally makes text and icons unreadable, and keeping original sizes leaves room for few elements. Re-layout for the smaller space, use scrollable or zoomable views, and pick assets per density bucket.
2. **Text and icon size.** Use less text, replace text with icons or sound only where meaning stays clear, and let users enlarge text.
3. **Clickable targets.** Make buttons big enough for a finger or stylus with enough gap to avoid wrong hits, and make the label part of the tappable area.
4. **Fixed-size windows.** Aspect ratios vary, so a fixed window leaves parts unreachable or hidden. Scale the layout proportionally, rearrange to use wide screens, or make the view scrollable.
5. **Tap-only screens lack mouse behaviour.** There is no hover, no right click and no cursor movement without touching. Replace them:
   - Hover tooltips become tap-to-reveal, long-press, or always-visible hints.
   - Right click becomes a visible on-screen control or a menu button. A long press can work but accidental early release causes wrong actions, so never make it the only path.
   - "Move cursor without clicking" becomes point-and-tap, or a two-tap point A then point B flow. Sensors such as accelerometer and gyroscope are extra input options.
6. **Screen mapping.** If a fixed canvas does not fill the screen, the touch position can mismatch the drawn position. Map touches through the same scaling and offset that was used to draw.
7. **Keyboard input.** Typing is slow on small devices. Reduce typed commands, use context buttons that appear only when needed, offer defaults and pick-lists, auto-generate names (for example from date and time), and disable chat-like text features unless they are core. Show the on-screen keyboard only when needed and make sure it does not cover the field.
8. **Distribution and ownership checks.** Physical media is gone, so use app stores, accounts, licences and cloud content instead of disc checks.

Example (invented). Desktop photo editor shows a 3-pane layout with hover tooltips and a right-click context menu. Mobile version: one main canvas, bottom toolbar with large icons and labels, a "more" button that holds what right click used to hold, tooltips shown on long-press, and a text-free default file name.

## 3. Android-style app behaviours worth knowing

- An Android app is a set of **screens (activities)** that can start each other, with or without data. Plan navigation as a flow of screens and decide what data passes between them before drawing.
- **Back button.** In the 2014 material, Back sends a screen to the background and it stays in the stack, it does not necessarily close it. Decide intentionally what Back does at every point, and avoid forcibly killing the app process. (+) Current guidance is to let the system manage process life and follow the platform's back navigation model.
- **Layouts as the interface.** The interface can be built from layout files with views, and edited visually or in code. Give each interactive component a clear identity so events can be attached cleanly.
- **Hardware keys.** Phones have volume, power, home, menu, back and sometimes a keyboard. Apps can react to them, but respond only to what users expect, and never trap Back.
- **Dialogs.** Three styles appear in the book: an activity styled as a dialog, a dedicated dialog class (alert, progress, and so on), and a **toast**, a short, non-modal message that fades by itself. Use a modal dialog only when a decision is needed (for example confirm exit), and a toast for passive confirmation.
- **Multi-touch and drawing.** Views can draw custom graphics and respond to touch, and multi-touch supports pinch and zoom. Design gestures with a visible alternative (button) for accessibility (+).
- **Settings screens** let users change app properties, keep them predictable.

Outdated parts (do not copy): Intel Atom hardware framing, Dalvik details, the old `showDialog` family and `System.exit` pattern, dp buckets quoted for old devices, and the Menu hardware key. (+) Modern equivalents are Jetpack Compose or current View system, Material Design 3, window size classes for phones, tablets and foldables, DialogFragment or Compose dialogs, snackbars, edge-to-edge and predictive back, 48 dp touch targets. Chapters 2 to 4 were read for headings and sections on dialogs, layouts, exit and key events only, and code listings are not reused.

## 4. Web and browser realities

**Why web apps win, still valid in spirit.** Data reachable from any device, always the latest version, works across devices through the browser, and sits behind the browser's security boundary.

**Technology layers.** HTML structures content, CSS styles it, JavaScript adds behaviour, and background requests let parts of a page update without a full reload, which makes pages feel faster. (+) Today this means modern JS frameworks, fetch, and service workers for offline use.

**Trust and security cues users rely on**
- Teach users to read the **host name** in the address bar, not the path or subdomain tricks. A fake site can sit at a look-alike subdomain or path, so "bank" appearing in the URL is not enough.
- A secure scheme (https) and a padlock show that traffic is encrypted, but do not prove the site is the right one.
- Design your own sensitive pages so they are easy to verify: stable domain, https everywhere, no surprise redirects, no login forms inside embedded frames from other domains (+).
- Never copy-cat a bank-style email asking to update details, and design real emails so they never ask for passwords (+).
- (+) The old green extended-validation box shown in the book has been removed from major browsers, so do not rely on it. Passkeys and two-factor sign-in are now preferred.

**Privacy and cookies.** Cookies remember language, cart and preferences, and are also used for ads. Give users clear control, explain what you store, and offer sensible defaults. There is real tension between personalisation and privacy, so choose the minimum data needed. (+) Follow current consent laws such as GDPR or local rules.

**Speed as UX.** Images, video and scripts dominate page weight, so compress images, avoid giant scripts, lazy-load below the fold, and preload what is likely next. The book quoted that images made up about 65 percent of page bytes in 2010, treat that number as dated. (+) Use modern image formats (WebP, AVIF), a CDN, and measure with Core Web Vitals.

**Old browsers.** Supporting very old browsers pushes teams to the lowest common denominator. Decide a support baseline using real analytics, use **progressive enhancement** (core works everywhere, extras layer on top) or **graceful degradation** (full design that falls back), and say clearly which browsers are supported. The browser names and versions in the 2010 text (IE8, Firefox 3.6, Chrome Frame, Flash plug-ins) are obsolete. (+) Flash is gone, use native video and standard web APIs.

**Responsive vs adaptive.** Responsive layouts flow continuously with screen size, adaptive layouts switch between fixed variants. Prefer mobile-first, start with the smallest screen's essentials and enhance upward (terms from the Indonesian and Henderson contents lists, details unread).

## 5. Web design as a profession (one author's view, dated)

Ben Hunt argues that building ordinary marketing websites has become a commodity product, because themes and platforms make a decent site cheap and fast, and that this follows the usual economy-of-scale cycle (new technology, wider use, standardisation, productisation). His advice is that designers who sell only hand-built generic sites will struggle, and that three paths remain:
1. **Custom high-end niche** (large brands, complex web apps, often in-house).
2. **Product development** (themes, plugins, apps, resources), ideally with a specialised and more strategic skill.
3. **Specialist services for clients**, such as marketing strategy, campaign design and delivery, and live operation of campaigns, rather than just page production.

Treat this as an opinionated business view, not a research result. It was read only in the opening argument, commoditization history and the three-path section. Later phases (strategy, campaign design, delivery, live operation) were read by headings only. (+) Today also consider design systems work, product design inside software teams, and UX research as additional paths.

## 6. Output formats this skill can produce

1. Porting plan: table of desktop behaviour, problem on touch, replacement.
2. Screen flow list for a mobile app with Back behaviour and dialog choices.
3. Web trust and performance audit: cue, current state, fix.
4. Browser support decision memo: baseline, enhancement strategy, test matrix.

## 7. Review checklist

- [ ] No hover-only, right-click-only or drag-only functions without an alternative.
- [ ] Targets are large, spaced, and labels are tappable.
- [ ] Layout survives other screen shapes, rotation, and text enlargement.
- [ ] Typing is minimised through defaults, pickers and smart names.
- [ ] Back and exit behave as the platform's users expect.
- [ ] Dialogs are used only for decisions, passive messages use toast or snackbar style.
- [ ] Sensitive web pages are verifiable (domain, https) and never ask for secrets through email links.
- [ ] Privacy choices are explained and controllable.
- [ ] Page weight and load time were measured on a slow connection.
- [ ] Browser support baseline is documented.

## Credits
- **Source material**: *GUI Design for Android Apps* by Ryan Cohen and Tao Wang (Apress, 2014), chapter 1 fully, chapters 2-4 partly; *20 Things I Learned About Browsers and the Web* by the Google Chrome Team (2010); *Web Design Is Dead* by Ben Hunt (v1); *UI/UX Design Guide* by Joel Parker Henderson (2024), contents only for mobile-first topics. This skill is an original summary and application of the books' ideas, not a reproduction of their text.
- **Skill author**: Zulkifli | GitHub: https://github.com/Zulkifli1409 | LinkedIn: https://www.linkedin.com/in/zulkifli1409/
