### Goals

- A site learns which of the age groups it names a person is in. The site gets no birthdate, no name, and no ID, and the answer does not say how the age was set. A site that asks again and again can learn a little more; see [[[#the-day-an-answer-changes]]].
- A site gets the least it needs. The site requests every group it can use, and the browser shares at most one of them, never anything finer.
- A site that only needs to know someone is an adult asks for `18+` and learns nothing more.
- A site can't use the answer to track people across sites, except by matching the day the answer changes; see [[[#the-day-an-answer-changes]]].
- A user finds it as easy to use as today's "I'm over 18" button.
- A site adds one element. The site doesn't need an age-check service.
- The person doesn't need a government ID.
- It works only on secure pages and only in the top-level page, in regular and private browsing alike.
- Users using a screen reader or a keyboard can use the [^agecheck^] element and the browser's prompt.
- The element works without JavaScript.

### Non-goals

- Not an identity system. Sites that need a name or a birthdate should use the [Digital Credentials API](https://www.w3.org/TR/digital-credentials/).
- Not a way to sign in. Sites still use passkeys or passwords to recognize people who come back.
