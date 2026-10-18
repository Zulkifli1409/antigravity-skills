---
name: ui-visual-accessibility-handoff
description: Helps design and review the visual layer of an interface, covering typography numbers, hierarchy, alignment, colour use, affordance, copy and microcopy, touch targets, forms and error handling, feedback, accessibility and WCAG basics, plus a clean designer-to-developer handoff. Use this when the user asks to critique a UI, fix a layout, choose font size or spacing, check contrast, make an app accessible, aksesibilitas, desain inklusif, design system, style guide, design tokens, microcopy, error message, form design, button states, handoff ke developer, spesifikasi desain, pixel perfect, or says a screen looks off or ugly without naming any rule.
author: Zulkifli
github: https://github.com/Zulkifli1409
linkedin: https://www.linkedin.com/in/zulkifli1409/
source: Pixel Perfect Precision Handbook version 3 by ustwo (PDF dated 2014), sections on principles, details, accessibility, design and development ; UI/UX Design Guide by Joel Parker Henderson (2024), topics on typography, microcopy, iconography, WCAG, colour blindness, dark patterns, design system, style guide, affordance, wizard, progress indicator ; User Interface and User Experience Design - dari Konsep ke Prototipe by Rizki Prasetyo Tulodo, Wresti Andriani, Nur Tulus Ujianto (Badan Penerbit Universitas Pancasakti Tegal, 2025), contents and glossary only
---

# UI Visual Craft, Accessibility and Handoff

Reply in the language the user writes in, get to the point, and return something they can apply at once (a fix list with concrete values, a spec table, a checklist) instead of general theory.

## 1. Decide what the user needs

| Request | Go to |
|---|---|
| "Review my screen" or "why does this look wrong" | Section 2 (principles) then section 9 checklist, output as fix list |
| Text looks hard to read | Section 3 (typography) |
| Forms, errors, confirmations | Section 5 |
| Accessibility or inclusive design | Section 6 |
| Handing a design to developers | Section 7 |
| Starting a style guide or design system | Section 8 |

## 2. Principles that drive every decision

1. **Users first.** Structure the product around what users want to do and who they are. A flat, grid-and-type driven look can suit young audiences, while less tech-savvy users may need friendlier, more obvious cues.
2. **Environment.** Platform is only part of it. A TV viewed from far away with a remote needs bigger text and stronger contrast than a phone used up close.
3. **Test the worst case, not just the pretty case.** Design with very long titles, long translations (German labels run much longer than English, and (+) Indonesian often does too), missing images, huge user-generated text, and empty states. A quick trick for width is to fill a box with capital W, since if it fits, other text will too. Keep ugly-case versions of a screen in the file and check them often.
4. **Affordance.** Make things look like what they do (raised button, content cut at the edge that hints at scrolling, a handle that invites dragging). Two kinds exist: visible cues, and learned conventions such as underlined text meaning link.
5. **Colour and shape carry learned meaning.** Warning tends to be yellow plus triangle, information blue plus circle, success green plus check, error red plus cross. Do not mix them up, and never rely on colour alone (+ see section 6).
6. **Visual hierarchy.** Decide what the eye should see first, second, third, then use size, weight, contrast and position to enforce it. Reduce the weight of secondary items instead of making everything loud.
7. **Consistency.** Same item, same place, same look, same behaviour on every screen. Follow the host platform's conventions where possible, so users can predict behaviour from experience.
8. **Motion with purpose.** Subtle animation can hint at function and add character. Do not use it only for decoration, and keep it optional for sensitive users (+ reduced motion setting).
9. **Test on the real device.** Screens, viewing angles and input methods differ from a desktop monitor, so preview live on the target device.
10. **Proximity, alignment, repetition and contrast** are the four basic layout levers (named in the Indonesian book contents, details unread). Use them to group related controls and separate unrelated ones.

## 3. Typography and layout numbers (starting values, adjust by context)

- Body text: not smaller than 12 pt, comfortable reading around 16 pt (1 em).
- Line length: keep under 80 characters, aim for 45 to 75, with about 66 as the sweet spot.
- Line spacing about 1.5 times type size, paragraph spacing also around 1.5 times line spacing so breaks are obvious.
- Use a clear scale of a few sizes and weights. Do not leave tool defaults untouched.
- Keep numbered lists once there are more than three or four points, since numbers aid navigation.
- Do not underline text that is not a link.
- Straight edges should land on whole pixels so they look sharp. Align and space with a grid, and reuse the same margins for title bars, back buttons and footers across screens.
- Building palettes in HSB is efficient, since you keep hue and vary saturation and brightness to get related shades.
- For digital work, skip heavy colour-profile management and verify colours on the device itself.
- Triangles drawn by tools may not be equilateral. Scale height to about 86.6 percent of width to fix it.

## 4. Copy and microcopy

- Write like a person, not a machine. Tone is part of the experience.
- Labels, button text, tooltips and error messages must be short, specific, and in the user's vocabulary.
- Spell out abbreviations on first use, or avoid them.
- Link text should describe the destination (and file size if it is a download). Never use "click here", especially for touch users and screen readers.
- (+) Button text should name the action ("Save changes") rather than describe the button.
- Example. Weak error: "Error 04, invalid input." Strong: "Phone number needs 10 to 13 digits. Remove spaces and try again."

## 5. Forms, errors, feedback

**Prevent errors by design**
- Offer defaults, pickers and selections instead of free text where possible.
- Put the risky action away from frequent actions, and make people pause on destructive steps with a clear confirmation that states the consequence.
- Validate while the user types or on leaving a field, so mistakes surface early, especially on mobile.
- For purchases or bulk actions use a review, confirm, correct step.
- Wizards help by chunking long tasks into steps with a progress indicator, validation per step, free movement back and forth, and optional steps skipped.

**Labels and fields**
- Put labels above fields, not as placeholder-only text, because once a value is typed a screen reader may announce only the value.
- Place the error message next to the field it concerns.

**Feedback**
- Always show that something is happening (spinner, progress, pressed state), so users do not wonder whether the app froze.
- Give feedback in more than one channel (visual plus sound or vibration), so deaf users or users in noisy places are not left out.
- Error messages must say what happened in plain words, how to fix it, and give a way back to a useful place.

**Navigation**
- Title every screen so users know where they are. Keep navigation in the same place and with the same labels everywhere.
- Aim to reach any information within about four screens, avoid cramming everything onto one screen, and avoid very long pages. Scroll in one direction only.

**Dark patterns to refuse** (and flag in reviews): misdirection, fake urgency, forced continuity, roach motel (easy in, hard out), hidden costs, sneaking items into the basket, friend spam.

## 6. Accessibility

Accessible design is good design for everyone, and needs little extra time when built in from the start. Main impairment groups to consider are sight, hearing, touch or motor, and cognitive. Offer more than one way to perceive and operate things.

WCAG organises requirements under four ideas, perceivable, operable, understandable and robust, with conformance levels A, AA and AAA. Practical rules:
- Text alternatives for images and captions for video. Write alt text that conveys purpose, not decoration.
- Do not use colour alone to show state. Add icon, text or pattern. Check palettes for colour blindness and test with simulations and real users.
- Keep strong contrast between text and background. (+) Common target is 4.5:1 for normal text and 3:1 for large text and UI parts (WCAG AA), verify with a contrast checker.
- Everything must work with a keyboard, a screen reader and assistive technology. Use semantic markup and ARIA attributes only where native elements are not enough.
- Touch targets: the book uses a minimum of about 7 mm by 7 mm with at least 2 mm gap, and makes thumb-driven controls wider. (+) Platform equivalents are about 44 pt on iOS and 48 dp on Android.
- Labels for text-to-speech should be short and clear ("Go to next screen"), not explanatory paragraphs.
- Plain language, logical order, help available, clear recovery from mistakes.
- Keep images light for slow mobile connections.
- Test with automated tools, keyboard-only runs, a screen reader, and real users with disabilities (+ do all four, since automated tools catch only part).

## 7. Designer to developer handoff

**Before designing**
- Ask for platform, screen sizes, pixel density, colour depth, available fonts, rendering limits (shadows, gradients, animation frame rates) and what styling can be recreated faithfully.
- Agree on deliverables and formats (images, vectors, scalable stretchable assets, specs) and on the workflow (waterfall or agile, build cadence, review sessions).
- Set up a shared tracker for asset requests, bugs and visual change requests, instead of scattering them through email.

**During design**
- Involve developers in ideation. Each side learning the other's pain points leads to better decisions and fewer arguments.
- Use realistic content, not perfect sample text.
- Keep source files editable, with named and grouped layers, sensible folder structure and file names, and a dated archive of old versions instead of "final_final" names.
- Design and export every state of a component (default, hover or focus, pressed, selected, disabled) at identical sizes.
- Be realistic. Matching mockups to the last pixel is worth less than fixing bugs or improving usability, so decide where exactness really matters.

**Spec table example** (invented values)

| Element | Token | Value | Notes |
|---|---|---|---|
| Body text | type.body | 16 / 24, regular, #333333 | line length 66 chars max |
| Primary button | btn.primary | height 48, radius 8, #0A7E8C | states default, pressed, disabled |
| Divider | line.default | 1 px, #E0E0E0 | |

## 8. Style guide and design system

A style guide fixes visual elements (colour, type, imagery, icons, grid and spacing, voice and tone, writing rules, examples). A design system adds principles, a component library, interaction patterns, accessibility rules and documentation. Start small: tokens for colour, type and spacing, then buttons, inputs, cards and navigation, each with states and do and do-not examples. Version it and let it grow with the product (+ design tokens exported to code keep design and development in sync).

## 9. Review checklist

- [ ] One clear focal point and a visible reading order.
- [ ] Type size, line length and spacing within the ranges above.
- [ ] Alignment and spacing follow a grid, edges are crisp.
- [ ] Same patterns for same functions across screens.
- [ ] Meaning is never carried by colour alone, contrast was measured.
- [ ] All interactive items look interactive and are big enough with gaps.
- [ ] Loading, empty, error and success states are designed.
- [ ] Forms have labels above fields, defaults, inline validation, errors near fields.
- [ ] Worst-case text (long, translated, missing image) still fits.
- [ ] Keyboard and screen reader paths were checked.
- [ ] Handoff package has specs, all states, realistic content, named layers, tracker.
- [ ] No dark patterns.

## 10. Coverage notes and outdated parts

- The ustwo handbook is tool-centred in its last third (Photoshop and Illustrator workflows, colour profiles, exporting, tips, pages roughly 110 to 209). Those parts were not read and are not used here. Its handoff advice about @2x slicing and fixed-size PSD exports is dated. (+) Use vector and component-based tools (Figma, Sketch), auto-layout, and exportable tokens.
- The accessibility numbers (such as the share of people with a disability) come from a UK-focused statement in the handbook, so treat them as an example, not a global figure.
- The Indonesian book text is not in the provided file, only its contents and glossary, so no chapter details are used.
- (+) Modern additions marked above: contrast ratios, platform touch target sizes, reduced motion, dark mode, design tokens.

## Credits
- **Source material**: *Pixel Perfect Precision Handbook, version 3* by ustwo (PDF dated 2014); *UI/UX Design Guide* by Joel Parker Henderson (2024); *User Interface & User Experience Design: dari Konsep ke Prototipe* by Rizki Prasetyo Tulodo, Wresti Andriani, Nur Tulus Ujianto (Badan Penerbit Universitas Pancasakti Tegal, 2025), contents and glossary only. This skill is an original summary and application of the books' ideas, not a reproduction of their text.
- **Skill author**: Zulkifli | GitHub: https://github.com/Zulkifli1409 | LinkedIn: https://www.linkedin.com/in/zulkifli1409/
