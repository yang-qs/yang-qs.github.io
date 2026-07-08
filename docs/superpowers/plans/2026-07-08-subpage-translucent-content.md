# Subpage Translucent Content Background Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the flat opaque `#f7f2e9` content background on the 5 nav subpages (About / Research / Talks & Teaching / Blog / CV) with a fixed, full-viewport cinematic photo backdrop and a translucent frosted-glass content panel, per [docs/superpowers/specs/2026-07-08-subpage-translucent-content-design.md](../specs/2026-07-08-subpage-translucent-content-design.md).

**Architecture:** A new `position: fixed; z-index: -1` layer (reusing the existing `cinematic-layers` Sass mixin) renders behind all normal page content on any page with `header.cinematic_banner: true`. The existing 200px in-flow `.cinematic-banner` is replaced by a transparent spacer div of the same rhythm, and the shared `#main` container (used by both `single.html` and `archive.html` layouts) gets a `cinematic-content` class that applies a translucent frosted panel (`rgba(cream, .8)` + `backdrop-filter: blur(8px)`) to its `.page`/`.archive` child.

**Tech Stack:** Jekyll (Liquid templates) + Sass (`_sass/theme/_cinematic.scss`). No JS.

**Verification approach:** This repo's local Ruby (4.0.5) cannot run a full `jekyll build`/`jekyll serve` — the pinned `jekyll 3.9.0` (via the `github-pages` gem) depends on `liquid 4.0.3`, which calls `String#tainted?`, a method Ruby fully removed. This is a pre-existing environment issue unrelated to this feature and out of scope to fix (would require Docker or a different local Ruby version, neither available). Two verification methods stand in for it in this plan:
1. **Sass compile check** — `assets/css/main.scss` has a 2-line Jekyll front matter block that trips up the plain `sass` CLI. Strip it first: `tail -n +3 assets/css/main.scss > /tmp/main-clean.scss`, then compile with `bundle exec sass --sourcemap=none -I _sass /tmp/main-clean.scss /tmp/main-test.css`. A clean exit with no `Error:` line means the Sass is syntactically valid. `bundle install` must be run once first (installs the `sass` 3.7.4 gem already in the `Gemfile.lock` dependency graph — no Gemfile changes needed for this).
2. **Static HTML mockup** — since Liquid can't render, build a plain HTML/CSS file that mirrors the real compiled output for one representative page, and check it visually with the `mcp__Claude_Preview__*` tools (screenshot + inspect). This is the same technique already used earlier in this project's session for the homepage font-size change.

---

### Task 1: Replace `.cinematic-banner` with fixed backdrop + spacer, add frosted panel styles

**Files:**
- Modify: `_sass/theme/_cinematic.scss:152-159`

- [ ] **Step 1: Confirm current compiled output has the old `.cinematic-banner` rule**

```bash
cd /Users/yangqiaoshi/Desktop/git_doc/yang-qs.github.io
bundle install
tail -n +3 assets/css/main.scss > /tmp/main-clean.scss
bundle exec sass --sourcemap=none -I _sass /tmp/main-clean.scss /tmp/main-before.css
grep -n "cinematic-banner\|cinematic-page-backdrop" /tmp/main-before.css
```

Expected: a `.cinematic-banner{` block is present; `.cinematic-page-backdrop` is not found (grep prints nothing for it).

- [ ] **Step 2: Replace the `.cinematic-banner` rule**

Replace this block in `_sass/theme/_cinematic.scss`:

```scss
.cinematic-banner {
  position: relative;
  width: 100%;
  height: 200px;
  overflow: hidden;
  margin-bottom: 2em;
  @include cinematic-layers;
}
```

with:

```scss
.cinematic-page-backdrop {
  position: fixed;
  inset: 0;
  z-index: -1;
  overflow: hidden;
  @include cinematic-layers;
}

.cinematic-banner-spacer {
  height: 240px;
  margin-bottom: 2em;
}

#main.cinematic-content {
  .page,
  .archive {
    background: rgba($cinematic-cream, .8);
    -webkit-backdrop-filter: blur(8px);
    backdrop-filter: blur(8px);
    border-radius: 4px;
    padding: 2em;
  }
}
```

- [ ] **Step 3: Recompile and verify the new rules are present, old one is gone**

```bash
tail -n +3 assets/css/main.scss > /tmp/main-clean.scss
bundle exec sass --sourcemap=none -I _sass /tmp/main-clean.scss /tmp/main-after.css
grep -n "cinematic-banner {" /tmp/main-after.css
grep -n "cinematic-page-backdrop" /tmp/main-after.css
grep -n "cinematic-banner-spacer" /tmp/main-after.css
grep -n "main.cinematic-content" /tmp/main-after.css
```

Expected: first grep (old `.cinematic-banner {` selector) prints nothing. The other three each print at least one match.

- [ ] **Step 4: Commit**

```bash
git add _sass/theme/_cinematic.scss
git commit -m "feat: replace subpage cinematic banner with fixed backdrop and frosted content panel styles"
```

---

### Task 2: Render the fixed backdrop + spacer markup in `page__hero.html`

**Files:**
- Modify: `_includes/page__hero.html:33-41`

- [ ] **Step 1: Replace the `cinematic_banner` branch markup**

Replace this block in `_includes/page__hero.html`:

```liquid
{% if page.header.cinematic_banner and cinematic_img_path %}
  <div class="cinematic-banner">
    <div class="cinematic-photo" style="background-image: url('{{ cinematic_img_path }}');"></div>
    <div class="cinematic-teal-tint"></div>
    <div class="cinematic-grain"></div>
    <div class="cinematic-vignette"></div>
    <div class="cinematic-letterbox cinematic-letterbox--top"></div>
    <div class="cinematic-letterbox cinematic-letterbox--bottom"></div>
  </div>
{% else %}
```

with:

```liquid
{% if page.header.cinematic_banner and cinematic_img_path %}
  <div class="cinematic-page-backdrop">
    <div class="cinematic-photo" style="background-image: url('{{ cinematic_img_path }}');"></div>
    <div class="cinematic-teal-tint"></div>
    <div class="cinematic-grain"></div>
    <div class="cinematic-vignette"></div>
    <div class="cinematic-letterbox cinematic-letterbox--top"></div>
    <div class="cinematic-letterbox cinematic-letterbox--bottom"></div>
  </div>
  <div class="cinematic-banner-spacer"></div>
{% else %}
```

(Only the `<div class="cinematic-banner">` wrapper class and the added spacer div change — the 6 inner layer divs are untouched, and the `{% else %}` branch below is untouched.)

- [ ] **Step 2: Confirm the file has no leftover reference to the old class**

```bash
grep -n "cinematic-banner\"" /Users/yangqiaoshi/Desktop/git_doc/yang-qs.github.io/_includes/page__hero.html
```

Expected: no output (the only `class="cinematic-banner"` was the one just replaced; `cinematic-banner-spacer` and `cinematic-page-backdrop` don't match this exact quoted string so they won't show).

- [ ] **Step 3: Commit**

```bash
git add _includes/page__hero.html
git commit -m "feat: render subpage cinematic image as a fixed page backdrop instead of a scrolling banner"
```

---

### Task 3: Add `cinematic-content` class to `#main` in both content layouts

**Files:**
- Modify: `_layouts/single.html:17`
- Modify: `_layouts/archive.html:15`

- [ ] **Step 1: Update `_layouts/single.html`**

Replace:

```liquid
<div id="main" role="main"{% unless page.author_profile or layout.author_profile or page.sidebar %} class="main--no-sidebar"{% endunless %}>
```

with:

```liquid
{% capture main_classes %}{% unless page.author_profile or layout.author_profile or page.sidebar %}main--no-sidebar {% endunless %}{% if page.header.cinematic_banner %}cinematic-content{% endif %}{% endcapture %}{% assign main_classes = main_classes | strip %}
<div id="main" role="main"{% if main_classes != "" %} class="{{ main_classes }}"{% endif %}>
```

- [ ] **Step 2: Update `_layouts/archive.html`**

Replace:

```liquid
<div id="main" role="main"{% unless page.author_profile or layout.author_profile or page.sidebar %} class="main--no-sidebar"{% endunless %}>
```

with:

```liquid
{% capture main_classes %}{% unless page.author_profile or layout.author_profile or page.sidebar %}main--no-sidebar {% endunless %}{% if page.header.cinematic_banner %}cinematic-content{% endif %}{% endcapture %}{% assign main_classes = main_classes | strip %}
<div id="main" role="main"{% if main_classes != "" %} class="{{ main_classes }}"{% endif %}>
```

- [ ] **Step 3: Confirm both files changed identically**

```bash
grep -n "cinematic-content" /Users/yangqiaoshi/Desktop/git_doc/yang-qs.github.io/_layouts/single.html /Users/yangqiaoshi/Desktop/git_doc/yang-qs.github.io/_layouts/archive.html
```

Expected: one matching line in each file.

- [ ] **Step 4: Commit**

```bash
git add _layouts/single.html _layouts/archive.html
git commit -m "feat: apply cinematic-content class to #main on pages with a cinematic banner"
```

---

### Task 4: Visually verify with a static mockup

Since Liquid can't render locally (see Verification approach above), this task builds a standalone HTML file that mirrors the real compiled output for a representative archive-style page (Talks & Teaching), using the actual new CSS rules, and checks it in the browser.

**Files:**
- Create: `/private/tmp/claude-501/-Users-yangqiaoshi-Desktop-git-doc-yang-qs-github-io/050cc604-87f6-47da-968c-68b5a818b777/scratchpad/subpage-translucent-mockup.html` (scratch file, not part of the repo)

- [ ] **Step 1: Write the mockup**

```html
<!doctype html>
<html>
<head>
<meta charset="utf-8">
<style>
  body { margin: 0; font-family: Georgia, Times, serif; background: #f7f2e9; }
  .masthead {
    position: fixed; top: 0; left: 0; right: 0; height: 56px; z-index: 20;
    background: #f7f2e9; display:flex; align-items:center; justify-content:flex-end;
    gap: 20px; padding: 0 20px; box-shadow: 0 1px 0 rgba(0,0,0,.08);
  }
  .masthead a { font-size: 12px; letter-spacing: .08em; color: #4a4433; text-decoration:none; }

  .cinematic-page-backdrop {
    position: fixed; inset: 0; z-index: -1; overflow: hidden;
  }
  .cinematic-page-backdrop .cinematic-photo {
    position: absolute; inset: 0; background-size: cover; background-position: center;
    filter: brightness(.62) contrast(1.15) saturate(.75);
    background: linear-gradient(135deg,#2b3a42,#1a2933 60%,#0d1a20);
  }
  .cinematic-page-backdrop .cinematic-teal-tint {
    position: absolute; inset: 0;
    background:
      linear-gradient(180deg, rgba(20,40,55,0) 35%, rgba(10,28,40,.55) 100%),
      linear-gradient(90deg, rgba(10,35,50,.35), rgba(10,35,50,0) 55%);
    mix-blend-mode: multiply;
  }
  .cinematic-page-backdrop .cinematic-vignette {
    position: absolute; inset: 0; box-shadow: inset 0 0 130px 40px rgba(0,0,0,.7);
  }
  .cinematic-page-backdrop .cinematic-letterbox {
    position: absolute; left: 0; right: 0; height: 20px; background: #000; z-index: 2;
  }
  .cinematic-page-backdrop .cinematic-letterbox--top { top: 0; }
  .cinematic-page-backdrop .cinematic-letterbox--bottom { bottom: 0; }

  .cinematic-banner-spacer { height: 240px; margin-bottom: 2em; }

  #main { max-width: 900px; margin: 0 auto; padding: 0 1em; }
  #main.cinematic-content .archive {
    background: rgba(247, 242, 233, .8);
    -webkit-backdrop-filter: blur(8px);
    backdrop-filter: blur(8px);
    border-radius: 4px;
    padding: 2em;
    color: #4a4433;
  }
  .archive h1 { margin-top: 0; font-weight: 400; }
  .archive-item { border-top: 1px solid rgba(74,68,51,.2); padding: 12px 0; }
  .archive-item .title { font-size: 14px; color: #3d3826; }
  .archive-item .meta { font-size: 11px; color: #6b6350; }
</style>
</head>
<body>
  <div class="masthead">
    <a href="#">About</a><a href="#">Research</a><a href="#">Talks</a><a href="#">Blog</a><a href="#">CV</a>
  </div>

  <div class="cinematic-page-backdrop">
    <div class="cinematic-photo"></div>
    <div class="cinematic-teal-tint"></div>
    <div class="cinematic-vignette"></div>
    <div class="cinematic-letterbox cinematic-letterbox--top"></div>
    <div class="cinematic-letterbox cinematic-letterbox--bottom"></div>
  </div>
  <div class="cinematic-banner-spacer"></div>

  <div id="main" class="cinematic-content">
    <div class="archive">
      <h1>Talks &amp; Teaching</h1>
      <div class="archive-item"><div class="title">Seminar on Stochastic Control</div><div class="meta">UPenn AMCS · 2025</div></div>
      <div class="archive-item"><div class="title">Guest Lecture: Applied Finance</div><div class="meta">Wharton · 2024</div></div>
      <div class="archive-item"><div class="title">Conference Talk: Optimal Execution</div><div class="meta">INFORMS · 2024</div></div>
    </div>
  </div>
  <div style="height: 150vh;"></div>
</body>
</html>
```

- [ ] **Step 2: View it with the Preview tools**

Use `mcp__Claude_Preview__preview_screenshot` on the file (open it directly in the preview browser), then `mcp__Claude_Preview__preview_eval` with `window.scrollTo(0, 800)` followed by another screenshot, to confirm:
- The frosted `.archive` panel is legible (text readable against the photo tint showing through)
- The fixed backdrop does not scroll — after scrolling, the photo/letterbox bars stay in place while the archive card moves up
- The masthead stays solid/opaque and above the backdrop

- [ ] **Step 3: Note any tuning needed**

If the panel is too see-through or the text contrast is weak, adjust the `.8` alpha value and/or blur radius in `_sass/theme/_cinematic.scss` from Task 1 and recompile per Task 1 Step 3, then re-screenshot. Do not commit further changes for this task — Task 1's commit already covers the styling; only amend it if a real problem is found here (see plan note below).

---

## Note on the pre-existing local Jekyll breakage

This plan does not attempt to fix the local `jekyll build`/`serve` failure (Ruby 4.0 removed `String#tainted?`, which `liquid 4.0.3` — pinned transitively by the `github-pages` gem — still calls). That is a separate, pre-existing issue unrelated to this feature. If real end-to-end verification against the actual Jekyll/Liquid output becomes necessary later, it needs either Docker (there's already a `Dockerfile`/`docker-compose.yaml` in the repo pinned to `ruby:3.2`, but Docker isn't installed on this machine) or a Ruby version manager (rbenv/rvm/asdf — none installed) to run an older Ruby locally. Surface this to the user as a separate, opt-in task rather than folding it into this plan.
