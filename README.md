# The `<agecheck>` element - Explainer

Adult content sites that check age today use either a simple link or button, or send people to third-party services that ask them to upload a copy of a physical document, such as a driver's license. Uploading physical documents risks leaking personal data and collects far more than the site needs. Sites need a way to learn a person's age group and nothing else, without needing a third-party age service, in regular and private browsing alike.

Parents can already set a child's age on the platform, in the parental controls on the child's device. A website has no way to ask for that age. So a game that limits voice chat and spending must take the child's word. So must a social media site that limits who can message a younger user. On adult content sites, the child picks an age or clicks past an "I'm over 18" button.

`<agecheck>` is an HTML element that lets a site ask which age group a person is in. The site lists the groups it needs from: `13+`, `16+`, `18+`, `13-15`, `13-17`, and `16-17`. The answer is declared, not cryptographically verified. The browser takes the answer from an age the person or a parent already gave the platform. Where the platform offers no age API, the browser can get the answer from an age the user self-declares. The browser prompts the user before sharing it, and gives the site one of the groups the site requests, or an error.

When the person clicks the element, the browser asks the platform for the person's age range, or uses an age that the browser stores itself. The platform may ask whether to share the age with the browser, as it does for any app, usually only the first time. Then the browser shows its own prompt, naming the site.

If the person agrees and the browser has a group to share, the page learns that group, such as `13-17`. Otherwise the page gets the same error as for any failure, so the browser never tells the page why. The element works inside a form, like an input element.

To the server, the answer is an honor system, like today's "I'm over 18" button. Anyone can send the same form value, so the answer proves nothing to a regulator that requires verification.

### Using `<agecheck>` on a gaming site

An online game wants to turn on features by age. Teens get filtered text chat and a monthly spending cap. Adults get open voice chat. The game requests the two groups it cares about and reads the answer:

```html
<agecheck id="age" groups="13-17 18+">Set up my account</agecheck>
<script>
  age.addEventListener("change", () => {
    if (age.group === "13-17") useTeenSettings();
    else useAdultSettings();
  });
  age.addEventListener("error", useSafestSettings);
</script>
```

A person under 13, a person with no age set, and a person who declines give the page the same `error`. The game then uses its safest settings and offers other ways to share an age.

### Using `<agecheck>` with an adult content site

A site that only blocks people under 18 names no group. With no `groups` attribute, the element defaults to asking for `18+`:

```html
<form method="post" action="/enter">
  <agecheck name="age" required>I'm 18 or older</agecheck>
  <button>Enter</button>
</form>
```

### Using `<agecheck>` streaming services

A streaming site with titles rated 13, 16 and 18 can ask for all three bands and learn which band the person is in:

```html
<agecheck groups="13-15 16-17 18+">
  Show titles for my age
</agecheck>
```

A site with one minimum age names that group and learns nothing finer:

```html
<agecheck groups="13+">I'm 13 or older</agecheck>
```

## Goals

- A site learns the age group it requests, and no personally identifiable information.
- A site gets the least it needs. The site requests every group it can use, and the browser shares at most one of them, never anything finer. 
- A site that only needs to know someone is an adult asks for `18+` and learns nothing more.
- A site can't use the answer to track people across sites, except by matching the day the answer changes; see "[Ages](#ages)".
- A user finds it as easy to use as today's "I'm over 18" button or link.
- The site doesn't need an age-check service.
- The person doesn't need a government ID.
- It works only on secure pages and only in the top-level page, in regular and private browsing alike.
- Users using a screen reader or a keyboard can use the `<agecheck>` element and the browser's prompt.
- The element works without JavaScript.

## Non-goals

- Not an identity system. Sites that need a name or a birthdate should use the [Digital Credentials API](https://www.w3.org/TR/digital-credentials/).
- Not a way to sign in. Sites still use passkeys or passwords to recognize people who come back.

## Why an element

The user clicks the element, the browser shows its own prompt, and the page gets only the result or an error. It works in an HTML `<form>` with no script, and the browser knows what the control is, so it can give it the right role and accessible name. Only the user's own action on it or its label opens the prompt.

By default, the element looks and lays out like a `<button>`. Authors can restyle it or hide it behind a label, as they do other form controls. A site could label the element misleadingly, as with any button. So the browser's prompt, not the element's look, is what protects people.

## Embedded content

In this first version, embedded third-party content can only get the age range from the top-level page. Embedded content includes a third-party `<iframe>`, such as a video player. The embedded content gets the age range by its own means, such as `postMessage()`. So the embedded content must trust the top-level page. A third party can run the check only as a top-level page of its own, such as a popup. In that case, the prompt names the third party.

## Where the age comes from

The platform here means the operating system. When the platform has an age API, the browser uses an age the person or a parent already gave the platform, such as Apple's Declared Age Range or Google's Play Age Signals. Both give apps an age that the person declared, that a parent set, or that the platform verified. How the platform sets the age is up to the platform.

What the browser does:

- If the platform offers an age API, the browser uses only that, so a stored age can't get around a parent's setting. The rule holds when this browser can't call the API, or when the API has no age for the person. In those cases the check fails.
- If the platform offers no age API, a browser could offer to store the user's age, the way the browser stores payment instruments for autofill. Alternatively, it could derive the age range from the user's contacts information (with their consent), which often contains their birthdate.
- A browser should not store a birth date. A browser should keep only the age group (under 13, 13 to 15, 16 to 17, or 18 and over) and the date the person entered it.
- A browser that stores its own age lets the person change that age.
- Private browsing works the same way as regular browsing and still uses the platform API. A private window can use an age that regular browsing stored. The browser forgets any age that a private window stored once the private browsing session ends.
- The browser never tells the platform which site asked.
- The browser could have a setting to turn the feature off, and a control that clears every remembered age choice at once. The browser should not list which sites have asked; see "People in danger" under "Open risks".

For children, the platform's parental controls decide whether the browser gets an age at all. On Apple platforms a parent can choose to always share, never share, or have the child asked each time. If a parent's setting or the platform blocks sharing, the page gets the same error as for any other failure, for children and adults alike.

## Ages

A site names the age groups it can use. The six groups are built from three cut-offs: 13, 16, and 18. Every platform age API we know of supports those cut-offs, though some regions use their own (see "Jurisdiction"). They also match the default groups of Google's Play Age Signals (0 to 12, 13 to 15, 16 to 17, and 18 and over), so a browser on Android needs no special setup.

A site can ask more than once, with a different list of groups in each check. For example, a movie site with films rated 13, 16, and 18 asks for `13+`, `16+`, and `18+` in three checks.

A site that asks again and again can see the day the answer changes. That day is the anniversary of when the age was set, not the birthday. Where the browser keeps a copy or stores the age itself, a higher answer appears on the anniversary of when the browser got it. If the person updates the age early, the change may hint at a birthday. The cost is that someone who has just had a birthday may fail until that anniversary, unless they or a parent update the age. A lower range from the platform applies at once, so a site can see a group drop to a lower group, or to no answer. The site can't tell why, and a drop doesn't point to a birthday.

A site can name groups that do not overlap, such as `13-15 16-17 18+`, and learn the person's exact age range in one check. A site that puts the groups in several elements can learn more than a site that lists the same groups in one element, because each element gets its own answer. Neither way learns more than the exact age range. The browser can't stop a site from requesting more groups than the site needs; see "[Asking for the least](#asking-for-the-least)". A refusal looks the same as a person who is too young, so the site can't be sure why an answer is missing. Sites that already share data about a person can also pool what they each learned. The answer carries little that links a person across sites, except the day it changes. That day is the same on every site, so sites that pool can match it. A site that checks every day sees the exact day, which narrows the person to one day of the year. Apart from a correction, only people under 18, who still have an age to reach, have such a day.

## Asking for the least

The browser shares the least a site asks for. These rules keep it that way:

- The site names every group it can use. The browser never adds a group the site did not name. A site that names no group the browser knows counts as naming `18+`.
- The browser shares at most one group. It picks a group that contains the person's whole age bracket. If the site names groups that overlap, the browser shares the one that tells the site least. A group with no upper end tells the site less than one with an upper end. Otherwise the group that starts lowest wins. If two groups start at the same age, the one that ends higher wins.
- If no group the site named contains the person's age bracket, the person shares nothing, and the site gets the same error as for any other failure.
- The browser never shares "under 13".
- The browser never shares a group finer than the one the site named. If a person's age setting is more exact than the group, the site still gets only the wider group.

A site that needs only a floor, such as 18 or over, asks for that floor and learns nothing about anyone younger.

## Jurisdiction

The platform, not the browser, knows which rules apply to the person's location. Declared Age Range uses the person's location and account settings. The platform answers with an age range. The browser shares a group only if the group contains the whole range. Declared Age Range gives someone under 13 an age range with no bottom, and the browser never shares a group for that range. The browser never shares a group finer than the one the site named.

When the platform returns no age range, the browser treats the missing age range as not sharing instead of having no age set.

Some regions replace the cut-off ages that the site's groups use with their own ages. An age range can then start below and end above a group the site named. For example, a site names `16+`. The platform gives an age range of 14 to 17. The range is not inside `16+`, so no group fits and the check fails. In those regions, some people who meet the age fail the check.

## Using the element

The introduction shows the element in use. These are its attributes and events:

- `groups`: the age groups the site can use, as space-separated tokens. The allowed tokens are `13+`, `16+`, `18+`, `13-15`, `13-17`, and `16-17`. The browser ignores any other token. With no valid token, the default is `18+`.
- `group`: the group that a successful check shared, such as `13-17`. The value is empty until then. Every token starts with its lowest age, so `parseInt(element.group, 10)` gives the lowest age.
- `name`, `required`, `disabled`, `form`: The browser treats these the same as for an input.
- Events: The browser fires `change` on success. The browser fires `error` on any failure after the check starts. A failure includes when the person closes the prompt. A click that does not start a check fires no event. `change` bubbles and `error` does not, so a failed check never looks like a script error to the page. The page cannot tell "declined" from "didn't pass".

**Advice for sites.** This section is advice that the browser does not enforce.

- Sites should use `method="post"` on the form. This choice keeps the answer out of URLs.
- Sites should name only the groups they need. A site that names groups that do not overlap learns the person's exact age range.
- With `required`, a site should offer another way to show age on the same page. Use a link. A submit button in the same form would not work until the check passes.
- Sites should treat `error` as "not now". Sites should offer to try again or offer another way.
- Sites should show that message in a live region, a part of the page that screen readers announce when it changes, so screen reader users hear the message.
- Sites should put any new options right after the element or the element's label. The next Tab reaches the options.
- Sites should never treat `error` as "too young".
- A site that needs a different path for younger visitors can offer an ordinary "I'm under 18" link next to the element. The person clicks the link only if the person chooses.

This code example shows a site following the advice. People who use a screen reader or only the keyboard can use the page this way.

```html
<form method="post" action="/enter">
  <agecheck id="check" name="age" required>Share that I'm 18 or older</agecheck>
  <a href="/other-ways">Share my age some other way</a>
  <span id="message" role="status"></span>
  <button>Enter</button>
</form>

<script>
  const check = document.getElementById("check");
  const message = document.getElementById("message");

  check.addEventListener("change", () => {
    message.textContent = "Thanks. You can go in now.";
  });

  check.addEventListener("error", () => {
    message.textContent = "We couldn't use your age this time. You can try again, or share your age some other way.";
  });
</script>
```

The live region starts empty, so screen readers are already watching the live region when the text changes. The example page never moves focus on its own. The message says what happened.

## Older browsers and repeat checks

A site can check for `HTMLAgeCheckElement` in `window` and show its old fallback if it's missing. The interface is also missing on a page that isn't a secure context, so the check fails there too. In a browser that doesn't know the element, the text inside it shows as plain text. A site can't learn in advance whether the person has an age set.

The first time a site uses the element, the browser shows its own prompt. If the person agrees, the browser remembers that choice for that site and that age, and later checks on the same site skip the browser's prompt. A site that asks about a different age gets a new prompt. A site should still remember a successful check in its own session rather than asking again. A private window starts with no remembered choices and never reads the choices from regular browsing. The browser forgets the choices made in a private window when the private browsing session ends. People can clear all of these choices at once in the browser's site settings. The browser also clears these choices whenever it clears the site's data.

## Processing model

The element starts from a click or tap on it or its label, a key press, or an assistive technology's press action. Which keys depends on its role, which is an open question. The browser requires a trusted event. A trusted event is one the browser creates from the person's own input, not from script. The event must aim at the element or its label. The page must also have transient activation, which is HTML's name for a recent click, tap, or key press that no other feature has used yet. The check uses the activation up, so one click starts at most one check. So `click()` and events that script dispatches do nothing, even inside the person's click on something else. HTML lets each platform decide what a click on a label does. This proposal sets a new rule. A click on the element's label starts a check only if that click is a trusted event.

When the person activates the element, the browser takes a snapshot of `groups` and uses it for the steps below, so the person answers the question they saw:

1. If the element is disabled, if `group` is already set, or if a check from this element is already in progress, stop without firing an event.
2. If the element isn't in a top-level document, for example inside an iframe or an embedded object, fire `error` and stop.
3. If `groups` is missing, or has no token the browser knows, default to `18+`. If the page has no transient activation, fire `error` and stop. Otherwise the check uses the activation up. For this step and step 2, the browser also writes the reason to the developer console, which scripts can't read.
4. On every check, the browser asks the platform for the person's age range. The browser lets the platform show a prompt if the platform chooses. The browser passes the ages 13, 16, and 18 to the platform.
   - Some platforms have an anniversary rule. For these platforms, the browser keeps no copy, so the platform's own rules apply every time. These rules include a parent's choice to ask the child each time.
   - A platform might lack an anniversary rule. For this platform, the browser keeps one copy of the answer for a user-agent specific time period (e.g., one year). The browser uses that copy for every site. If the platform reports a higher range, the browser keeps using the copy, so a higher answer appears only on an anniversary. If the platform reports a lower range, the browser uses the lower range and replaces the copy at once. A parent's correction then applies at once.
   - The platform might stop sharing. The browser drops its copy, so a parent's change applies at once.
   - For an age the browser stores itself, the rules in "Where the age comes from" apply.
   - If the person declines in the platform's prompt, the browser fires `error`. The browser then stops.
   - For any other result, the browser goes to the next step. A failure counts as another result.
5. The browser picks the group to share. First it keeps the groups in `groups` that contain the person's whole age range. A person under 13 never shares a group. If more than one group is left, the browser picks the one that tells the site least: a group with no upper end wins over one with an upper end, and then the group that starts lowest wins. If two groups start at the same age, the one that ends higher wins. If no group is left, the browser has no group to share, and still goes on to the prompt.
6. The browser shows its own prompt, naming the site (for example, the origin). The browser skips the prompt only if the person already agreed to share this group with this site.
   - If the browser stores the age itself and has no age yet, the prompt lets the person select the age group the person is in. The browser continues the steps with that declared age range.
   - The prompt gives the same choices whether or not the browser has a group to share. If the person agrees and the browser has no group to share, the check fails with the same `error`.
   - The prompt is modal. The person cannot use the page until the person answers the prompt.
   - If the person cancels the prompt, the browser fires `error` and stops.
7. The browser fires `error`, writes the reason to the developer console, and stops, if any of these is true:
   - the element is no longer in the document;
   - the document is no longer fully active;
   - `groups` no longer holds the same tokens as the snapshot;
   - the element is now disabled;
   - the form reset since the check started.

   Otherwise the browser sets `group` to the group it picked and fires `change`.

The browser adds no timing difference of its own, but how long the person takes can still differ (see "Human timing" under "Open risks").

With `required`, the element counts as missing a value until `group` is set.

When the form submits, if `name` is set and `group` is set, add one entry named `name`. Its value is the group, such as `13-17`. If `group` is empty, add nothing, the same as an unticked checkbox.

When the form resets, or when `groups` changes to a different set of tokens, clear `group`. Order and repeats don't count as a change. The browser never restores `group` after navigation. When a page comes back from the back/forward cache, the browser clears `group` before the `pageshow` event, so the page can update. The browser never copies `group` when the element is cloned. When the person answers the prompt after a reset, the browser drops the answer and fires `error`.

The steps after the browser's prompt appears run in parallel, so scripts on the page keep running while the person decides.

## Automated testing

Like the Digital Credentials API's WebDriver module, a WebDriver extension, available only in automation sessions, could set a test age and drive the prompt, so sites and browsers can test the element automatically.

To do: define that extension. It needs a command to set the test age, and a way to agree to or cancel the browser's prompt.

## Web IDL

```webidl
enum AgeGroup { "", "13+", "16+", "18+", "13-15", "13-17", "16-17" };

[Exposed=Window, SecureContext]
interface HTMLAgeCheckElement : HTMLElement {
  [HTMLConstructor] constructor();

  [SameObject, PutForwards=value] readonly attribute DOMTokenList groups;
  readonly attribute AgeGroup group;

  readonly attribute HTMLFormElement? form;
  [CEReactions] attribute DOMString name;
  [CEReactions] attribute boolean disabled;
  [CEReactions] attribute boolean required;
  readonly attribute NodeList labels;
  readonly attribute boolean willValidate;
  readonly attribute ValidityState validity;
  readonly attribute DOMString validationMessage;
  boolean checkValidity();
  boolean reportValidity();
  undefined setCustomValidity(DOMString error);
};
```

## Accessibility and internationalization

The browser gives the element its role, keyboard handling, and focus return. The element's name comes from its visible text or its label, so voice control users can say what they see. When the prompt closes, focus returns to the element if it can take focus, and otherwise to where it was before.

The prompt belongs to the browser or platform, not the page, but its accessibility is in scope. It should follow WCAG 2.2, and where it isn't web content, it should follow WCAG 2.2 as [WCAG2ICT](https://www.w3.org/TR/wcag2ict-22/) describes, the W3C guidance on applying WCAG to software that isn't a web page. Its content is exposed to assistive technology with the right names, roles, and values. It doesn't rely on a single sense or way of interacting, and it gives people enough time.

The prompts use the language of the browser's or the platform's own interface. They must show site names in any script, including right-to-left scripts, without letting a name pretend to be another site.

## Privacy and security

- The site learns at most one group per check.
- Every failure looks the same to the site, whether the person is too young, has no age set, or declined.
- The browser's prompt should ignore input for a short time after it appears, for example as long as the system's double-click time, and while a window from the page or from another app covers it. A window that the person's assistive technology draws, such as an on-screen keyboard or a magnifier, doesn't count as covering the prompt. A page can't use a fast second click to answer a prompt the person hasn't seen yet. A key that the person was already holding when the prompt appeared must not answer the prompt either. For example, the prompt can have no default button.

## Stakeholder feedback

No public positions yet.

## Related approaches

The `<agecheck>` element doesn't replace these. Several solve different problems, and they can work alongside it.

**Mozilla's content labelling** ([whatwg/html#11722](https://github.com/whatwg/html/issues/11722)). Sites label adult pages, and the browser blocks them for people whose platform restricts adult content. The site learns nothing. It blocks whole pages, while the `<agecheck>` element lets a site check before showing something specific.

**Mozilla's `Prefer: safe` header** ([whatwg/fetch#1891](https://github.com/whatwg/fetch/issues/1891)). The browser tells every site the person prefers safe content, without the site asking. The `<agecheck>` element answers only a site that asks, after the person agrees.

**An Age Range API**, [proposed by Nick Doty](https://github.com/npdoty/age-range) (Center for Democracy & Technology). A page calls `navigator.ageRange()` and, once the person gives permission, gets an age range such as 13 to 17. The site must also publish a declaration at a well-known address, which the browser checks first. The `<agecheck>` element answers with one of the age groups that the page names, such as 13 to 17. Only the person's own action starts the element, and the element needs no declaration file.

**The Digital Credentials API**, with or without passkeys. It can carry a cryptographic proof, but it needs a digital ID, which many people don't have. It doesn't guarantee that only the age is revealed, and it lets a site ask for more than age. Presenting an ID is also heavy for a simple age check. The `<agecheck>` element uses the age the platform already has, or an age the person enters in the browser, and asks only about age.

**A JavaScript method** could open the same prompt. But it would still need a user action to start it, and wouldn't work in a form without script. For a canvas, the [HTML-in-Canvas proposal](https://github.com/WICG/html-in-canvas) would let the element itself be drawn inside the canvas, if a click on a drawn element counts as the person's own click.

## Open risks

- **Proof for regulators.** Sites may need to show a regulator that they checked age, and a declared answer can't show that.
- **People left out.** Some people have no account, no supported device, or no age set. Sites may fall back to ID uploads, and nothing here can stop them. Adults whose account is still set up as a child's, for example one a carer manages, are treated as children until whoever manages the account changes it.
- **Old buttons.** A site can keep today's "I'm 18" button next to the element, and then the element protects no one. Nothing here makes a site remove the button, so the element helps children only on sites that do. Those same sites shut out people who can't or won't answer.
- **Ages the browser stores.** Where the platform has no age API, the browser keeps an age the person enters, and the person can change it. A child can enter 18 and over, and the element then protects that child no more than today's "I'm 18" button does.
- **Shared devices.** A minor can use an adult's unlocked device, or control it remotely. Child accounts on the platform are the main defense. On a shared computer or browser profile, a stored age applies to whoever is using it. Anyone looking at the screen can also tell from the browser's prompt whether the person passed.
- **A third party as the top-level page.** A company can run the check as the top-level page, in a popup or after a redirect, and pass the answer back, as "Embedded content" describes. Nothing stops this.
- **Tricked approvals.** A page can push someone to approve without thinking. A page can make a label for the element cover the whole page, so a click meant for something else opens the prompt. The page can open the prompt again after every refusal. After the person agrees for a site and age, a click on that label starts a new check that runs with no prompt. See "Why an element".
- **Pushy platforms.** A platform could use the feature to push people into creating or signing in to its own account before it shares an age.
- **An age on every device.** To pass the check on sites that rely on the element, people must declare an age on every device they use. Over time, declaring an age could become a condition for using parts of the web.
- **Friction and refusal.** People may find the prompt annoying and refuse it. We have no research yet on how many.
- **Scripts on the page.** Like any form field, the answer is visible to every script the site chooses to run, including analytics. A script can also add its own `<agecheck>` element and read the result of a check that a click on it starts. The prompt names the site, not the script, so the person can't tell who asked. After the person agrees for a site and age, a click on that script's element or label starts a new check that runs with no prompt.
- **Trusting the platform.** A platform age API may contact its vendor's servers. Declared Age Range, for example, can fail with a network error. Because the browser never names the site, the platform can't report which sites a person visits. Where the browser asks the platform on every check, the platform can learn how often and when checks happen.
- **Human timing.** A prompt that only informs takes a different time than one that asks, and a platform that has the person set up an age first takes longer still. So how long the person took, or the page noticing focus leave for the system settings, may hint that no age is set.
- **A failed try.** A site sees that the person started a check and did not pass. The site can't tell why, but it can treat a failed try as a hint that a child tried, and block or flag that person. The advice says sites should never treat `error` as "too young", and nothing here makes them follow it.
- **Traces of the person's choice.** The page can't see the prompt, but the prompt can leave traces. For example, the pointer stays where the person pressed, and the page may be able to read that position. A trace could tell a person who declined from a person who couldn't share.
- **People in danger.** For some people, an adult age on their device is risky. If sharing is turned off, for example because the person declined the platform's prompt or turned off a browser setting, a site must see it as an ordinary refusal. The browser's record of which sites asked about age is also sensitive, so it should be cleared with the site's data and kept out of view.

## User research

None of our own yet.
