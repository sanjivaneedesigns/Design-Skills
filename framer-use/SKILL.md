---
name: framer-use
description: "Enforces style-guide discipline in an already-set-up Framer project: every text element and every color must belong to a named Style (text style or color style), never a raw hardcoded value. Trigger whenever working in a Framer project via the Framer external agent connection (the `/framer` bridge in Claude Code — not an MCP server) and the project already has existing text styles, color styles, or breakpoints. Covers: applying existing styles instead of hardcoding, creating a new style (matching the project's naming convention) only when no match exists, propagating breakpoint overrides to text styles when breakpoints are added, and recreating a design from a reference image in Framer (requires an iterative screenshot-vs-reference comparison loop, up to 3 rounds, before considering the build done). Also trigger on 'style guide', 'design tokens', 'design system', 'breakpoints', or 'recreate/build this in Framer' from an uploaded image."
---

# framer-use — Style Guide Discipline

Core rule: **in a project that already has a design system set up, nothing gets a raw value.** Every text layer uses a Text Style. Every color (fill, stroke, background) uses a Color Style. If a breakpoint is added, every Text Style used at that breakpoint gets an override for it, not just the layout.

This skill is a *policy* layer on top of however you're driving Framer in this session — the Framer external agent connection (`/framer` skill, installed via `npx @framer/agent setup`), which gives Claude Code direct access to the canvas, components, and CMS. This is **not** an MCP server — Framer's docs explicitly position it as a replacement for one. Changes happen on a branch you review and merge yourself; they don't touch the live published site until you say so. This skill doesn't replace knowledge of how that bridge actually works — it tells you the *rules to follow* every time you touch text or color in an existing project, on top of whatever the bridge's own tools let you do.

## 0. Is this project "already set up"?

Before applying these rules, check: does the project already have named text styles and/or color styles in its style panel/assets? If yes, all of section 1–4 below is a hard requirement for everything you build in it.

If the project has **no** styles yet (greenfield, first screen), these rules don't apply yet — that's a separate "set up the design system" task, not this skill. Tell Sanjivanee if you land in a project with no existing styles and she asked you to "build something" — ask whether she wants you to establish the style guide first or proceed unstyled for now.

## 1. Before creating anything: inventory existing styles

Every time you start a build/edit task in the project, first pull the full list of:
- Color styles (name + hex/value)
- Text styles (name + font, size, weight, line-height, letter-spacing, and any existing breakpoint overrides)

Do this once per session/task, not once per element — you need the full picture before placing a single layer. Use whatever read/list capability the Framer external agent connection exposes for styles (check available commands/tools in the current session if unsure). If there's no direct list call, inspect the style panel via whatever read capability is available.

Keep this inventory in mind (or written to a scratch note) for the rest of the task so you're not re-querying for every single layer.

## 2. Applying styles — the matching rule

For every text layer and every color use (fill, background, stroke, border):

1. **Look for a match first.** Does an existing style already fit what this element needs (visually and semantically — e.g. don't reuse "Body" for what's functionally a "Caption" just because the size happens to be close)?
2. **If a match exists, apply it.** Don't recreate the values manually even if it's faster to type a hex code — assign the actual style/token so it stays linked.
3. **If no match exists, create one, then apply it.** Never apply a one-off raw value and move on.

### Naming new styles

Match the project's existing naming convention — don't impose a new pattern. Before naming a new style:
- Look at how current styles are named (e.g. `Heading/H1`, `heading-xl`, `Primary/600`, `Body Regular`) — case, separators, hierarchy (slashes/folders vs flat), and any prefix/category structure.
- Name the new style consistently with that pattern.
- If the existing styles have no discernible convention (inconsistent or only one or two exist), ask Sanjivanee how she wants the new one named rather than guessing.

## 3. The hardcoding exception — flag, don't block

Sometimes a true one-off is unavoidable (e.g. a single decorative element that will never repeat, an inherited color from an embed). In that case:
- It's fine to use a raw value rather than manufacturing a fake "style" for something that isn't reusable.
- But **always flag it explicitly to Sanjivanee** in your response — name the element, the value used, and why you didn't make it a style. Don't silently leave hardcoded values in the file.
- Default assumption should be "this should be a style" — only skip if you have a real reason, not convenience.

## 4. Breakpoints — styles travel with layout

If you're adding or adjusting a responsive breakpoint:

1. **Don't just resize/reflow the layout.** Any Text Style used by elements at that breakpoint needs its own override added *to the style itself* (size, line-height, letter-spacing as needed for that viewport) — not a one-off manual override on the individual layer.
2. Check whether the text style already has overrides for other breakpoints already set up in the project, and follow the same scale/logic (e.g. if Desktop→Tablet drops size by ~15%, apply a similarly reasoned drop for the new breakpoint rather than an arbitrary number).
3. After adding a breakpoint override to a style, double check every element using that style at that breakpoint actually reflects it — don't leave it half-applied.
4. Color styles generally don't need per-breakpoint values (color rarely changes by viewport) — this rule is specific to text styles. Flag to Sanjivanee if a genuine color-per-breakpoint need comes up, since that's unusual enough to be worth a sanity check.

## 5. Recreating from a reference image

If Sanjivanee gives an image and asks to recreate it in Framer, everything in sections 1–4 still applies — matching/creating styles, naming convention, breakpoint propagation. On top of that, run a review loop before calling it done:

1. **Build first pass** using existing/new styles per the rules above.
2. **Compare against the reference, up to 3 rounds.** After building, take a screenshot of what you've made in Framer and compare it side-by-side against the original reference image. Check spacing, alignment, font sizing/weight, color accuracy, proportions, and overall layout fidelity.
3. **Fix what's off, then re-compare.** If there's a meaningful gap, adjust and screenshot again. Repeat for a maximum of 3 review rounds total (initial build + up to 3 comparison passes).
4. **Stop at 3 rounds regardless of remaining gaps.** If it's still not a close match after 3 rounds, stop, show Sanjivanee the current state next to the reference, and call out specifically what's still off and why (e.g. a font isn't available in Framer, an effect can't be replicated, an asset is missing) rather than silently continuing to iterate.
5. **Fixes still have to go through styles, not one-off patches.** If round 2 review shows text is too small, fix the Text Style (or apply a better-matching existing one) — don't slap a manual override on the layer to make the screenshot look right while leaving the underlying style wrong.

Each comparison round counts even if the fix is small — don't skip straight to "close enough" without actually doing the visual diff.

## 6. End-of-task self-check

Before considering a build/edit task done, verify:
- [ ] Every text layer touched uses a named Text Style (no raw font-size/weight/line-height overrides left on individual layers, unless flagged per section 3)
- [ ] Every color used (fill/background/stroke) uses a named Color Style (no raw hex left applied, unless flagged per section 3)
- [ ] Any newly created styles follow the project's existing naming convention
- [ ] Any breakpoints touched have corresponding Text Style overrides, not just layout changes
- [ ] Any hardcoded exceptions are explicitly called out in your response to Sanjivanee, not buried
- [ ] If built from a reference image: at least one screenshot-vs-reference comparison was actually done (not skipped), fixes went through styles not manual overrides, and remaining gaps after 3 rounds are called out explicitly

If any box fails, fix it before calling the task done — this is the actual point of the skill, not an afterthought checklist.
