---
name: verify
description: Record proof that the change works. Use after /test, before /ship, when a reviewer needs to see the behavior rather than take your word for it — security gates, auth flows, error states, anything a diff cannot show. Drives a real browser with Playwright MCP and records a narrated video to attach to the PR. Requires the playwright MCP server with --caps=devtools.
---

# Verify

> Key principle: A video shows. Assertions prove. Ship both.
>
> **Role in the workflow — step 6 of 7, between `/test` and `/ship`.** See [docs/SPECS.md](../../../docs/SPECS.md).
> - **Input:** a built, tested change and the acceptance criteria from its `spec.md`.
> - **Output:** a recorded walkthrough + a pass/fail assertion log, both attached to the PR.

`/test` proves the change works. `/verify` makes that proof *legible to a reviewer* — it records the
running app doing the thing, with each acceptance criterion called out as a chapter in the video.

A recording alone is weak evidence: it is easy to record a flow that looks right and proves nothing.
**Every claim in the video must be backed by an assertion that fails loudly.** The video is how a
human sees it; the assertion log is why they can believe it.

---

## When to Use

- **Access control and auth flows** — allowlists, permission gates, login/logout, session expiry.
  A reviewer cannot tell from a diff whether the gate actually holds.
- **Security-sensitive paths** — redaction, consent gates, anything where "it silently didn't apply"
  is the failure mode.
- **Error and edge states** — 4xx/5xx handling, rate limits, empty states, malformed data.
- **Anything a reviewer would otherwise have to run locally to believe.**
- **Bug fixes where the bug was visual or interactive** — record before *and* after.

**When NOT to use:**

- Backend-only or pure-logic changes — `/test` is the right proof; a browser adds nothing.
- Routine refactors, dependency bumps, copy changes.
- As a substitute for tests. A video is not a regression test: it never runs again.
- When Playwright MCP is not configured — say so rather than quietly falling back to screenshots.

---

## Setup

Video tools are **opt-in behind `--caps=devtools`**. Without it the server exposes no video tools at
all and the recording step will fail with "tool not found".

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest", "--caps=devtools", "--output-dir=.playwright-artifacts"]
    }
  }
}
```

Add the artifact directory to `.gitignore` — recordings are build output, not source.

### The video tools

| Tool | Parameters | Notes |
|---|---|---|
| `browser_start_video` | `filename`, `size: {width, height}` | Start recording. |
| `browser_stop_video` | — | Returns the path(s) written. |
| `browser_video_chapter` | `title`, `description`, `duration` (ms) | Full-screen card with blurred backdrop. |
| `browser_video_show_actions` | `duration`, `position`, `cursor` | Annotates each action and animates a cursor. |
| `browser_video_hide_actions` | — | Stop annotating. |

---

## Three gotchas that will cost you a re-record

These are the difference between one take and three. All three were confirmed against
`@playwright/mcp` 0.0.80.

**1. Navigate before you start recording.** Calling `browser_start_video` before a page exists fails
with `Error: Screencast is already started` and leaves a stray extra file (`name-1.webm`) alongside
a truncated recording. Open the page first, *then* start the video.

**2. `filename` resolves against the server's working directory, not `--output-dir`.** A relative
`filename` lands in whatever directory the MCP server was started from. **Always pass an absolute
path** — then the file is exactly where you said, and `browser_stop_video` echoes it back to confirm.

**3. Nothing is flushed until recording stops.** Read the path from `browser_stop_video` and confirm
the file exists and is non-zero before you rely on it. Do not assume a file appeared.

---

## The workflow

### 1. Write the claim list first

Before recording, state what the video must demonstrate — one line per acceptance criterion, each
with the observable signal that settles it. If you cannot name the signal, you are not ready to record.

```markdown
## Claims — invite-only access

1. Uninvited address is refused        → generic error toast, stays on the email step, HTTP 403
2. Look-alike address is refused       → same, HTTP 403 (suffix must not match)
3. Allowlisted address is let through  → advances to the OTP step, HTTP 200
```

Include the **negative and adversarial cases**, not just the happy path. A video of the happy path
alone is the single most common way this step produces false confidence. For a gate, the
interesting frames are the ones where it *refuses* — and a near-miss (`user@allowed.com.evil.com`)
is worth more than another obvious reject.

### 2. Get the app into a real state

Run it the way it actually runs — real dev server, real backing services where safe. A recording
against a mock proves the mock works.

> **Stop here if the flow has outward-facing side effects.** Sending email or SMS, charging a card,
> writing to shared data, or touching anyone's account but the user's own — **ask before recording,
> and say exactly what will happen and to whom.** Offer a narrower alternative: record the paths
> that are side-effect-free live, and cover the rest at the assertion level. If the user approves a
> limited scope, respect it — and state in the PR which cases were *not* exercised live and why.
> Re-records multiply the side effects: one extra take is one extra real email.

### 3. Record

```
browser_navigate(app_url)                    ← page must exist first (gotcha 1)
browser_start_video({ filename: "<abs>/verify.webm", size: {width: 960, height: 720} })
browser_video_show_actions({ position: "bottom", cursor: "pointer" })

for each claim:
    browser_video_chapter({ title: "1 / 3  Uninvited address",
                            description: "expected: blocked", duration: 1500 })
    ... drive the flow ...
    ... assert the signal ...            ← the part that can actually fail
    pause so a human can read the result  ← ~2.5s after a toast or state change

browser_stop_video()                         ← returns the path; confirm the file exists
```

**Use chapters rather than hand-rolled overlays.** A chapter card is centered, temporary, and clears
the corners of the viewport — so it never hides the UI you are documenting. An injected banner
pinned to an edge will eventually cover a toast, a badge, or a focus ring, and you will not notice
until you watch the file back.

**Pace it for a human.** Type at a visible speed, and hold for ~2.5s after each result. A toast that
appears and vanishes in four frames documents nothing. Total runtime under ~45s.

### 4. Assert — this is the part that can fail

Capture the machine-readable signal alongside the visual one, and **make failure loud**:

- `browser_network_requests` → status codes and payloads for the calls the flow fired
- `browser_snapshot` → the accessibility tree actually contains the expected text/role
- `browser_console_messages` → no unexpected errors

Report a pass/fail line per claim and a final `N/M checks passed`. **If a check fails, the change is
not verified — fix it and re-record.** Never narrate a failure as a success, and never quietly drop
a claim that turned out to be inconvenient to demonstrate.

### 5. Watch the recording before you attach it

Non-negotiable, and the step most often skipped. Pull 3–4 frames at the moments that matter and
look at them:

```bash
ffmpeg -ss 7.5 -i verify.webm -frames:v 1 -vf scale=620:-1 frame.png
```

Check the evidence is actually *visible*: the error text is legible and unobscured, the right field
holds the right value, the state change is on screen long enough to read. A recording that proves
nothing is worse than none — it looks like diligence.

---

## Attaching it to the PR

Playwright MCP writes **VP8 `.webm`**, which GitHub will not render inline. Convert before attaching:

```bash
# MP4 — for a human to drag into the PR (plays inline, small)
ffmpeg -i verify.webm -c:v libx264 -pix_fmt yuv420p -crf 23 -preset slow -movflags +faststart verify.mp4

# GIF — renders inline in markdown; larger, so cap the width and frame rate
ffmpeg -i verify.webm -vf "fps=9,scale=680:-1:flags=lanczos,split[a][b];[a]palettegen=max_colors=96:stats_mode=diff[p];[b][p]paletteuse=dither=bayer:bayer_scale=4:diff_mode=rectangle" -loop 0 verify.gif
```

Then pick the delivery route — **check repo visibility first, it changes the answer**:

| Situation | What works |
|---|---|
| **Public repo** | Commit the GIF, embed it with a raw URL. Renders inline for everyone. |
| **Private repo** | An embedded raw URL renders **broken** — GitHub's image proxy fetches anonymously and gets a 404. Commit the file and *link* it instead (the blob view plays a GIF for anyone with access). |
| **Inline video** | Only a human can do this: GitHub accepts video attachments through the web UI only, never the API. Hand them the MP4 and say so. |

Whatever route you take, **verify the link actually resolves** rather than assuming — a broken image
in a PR description is worse than a plain link.

Always include the assertion log in the PR body next to the recording. A table of claim → result →
status code is what a reviewer can actually check; the video is what makes them look.

State plainly what was *not* covered: cases skipped for side effects, environments not exercised,
anything stubbed. A reviewer who discovers a gap you did not disclose stops trusting the rest.

---

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "The video shows it working" | A video shows what you chose to record. Without assertions it is a screencast of your intentions. |
| "I'll just record the happy path" | The happy path is what already worked. The gate, the reject, the error state are why anyone is reviewing. |
| "The recording came out fine, I don't need to watch it" | Overlays cover toasts, states flash by, the wrong field gets focus. You will not know until you look. |
| "It's only a test email" | It is a real email to a real person. Ask first — and remember a re-record sends another. |
| "I'll embed the GIF with a raw URL" | On a private repo that renders broken for every reviewer. Check visibility first. |
| "Screenshots are basically the same" | Stills cannot show a transition, a race, or a state that appears and disappears. |
| "Playwright MCP isn't set up, I'll approximate it" | Say it is not configured. A substituted method presented as the requested one is a false report. |

## Red Flags

- A recording attached with no assertions next to it
- Only passing cases on screen; no reject, no error state
- The video never watched back before being attached
- A claim in the PR body that no frame in the video and no assertion supports
- Outward-facing side effects (email, SMS, payments, third-party accounts) triggered without asking
- Cases skipped for side effects, but the PR does not say so
- A broken embed left in the PR description
- The recording treated as a regression test — it never runs again
- Captions or overlays covering the very UI being demonstrated

## Verification Checklist

- [ ] Claim list written *before* recording, including negative and adversarial cases
- [ ] Side effects identified, and consent obtained for anything outward-facing
- [ ] App running in a realistic state, not against a mock that proves only the mock
- [ ] Recording is paced to be readable — results held long enough to see
- [ ] Every claim backed by an assertion that would fail loudly
- [ ] Assertion log shows `N/M checks passed`, with no claim quietly dropped
- [ ] Frames pulled and inspected — evidence visible and unobscured
- [ ] Converted to a format the destination renders; link verified to resolve
- [ ] PR states what was not covered and why
