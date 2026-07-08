# Homepage Redesign (Cinematic Film) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rebuild the site's homepage as a pure cinematic-film landing page (full-bleed cool-toned photo, orange/teal grading, grain, vignette, letterbox), switch all content subpages to a warm editorial theme, and simplify navigation to 5 sections (About / Research / Blog / Talks & Teaching / CV).

**Architecture:** This is a Jekyll static site (academicpages theme). No JS framework, no test runner — "tests" in this plan mean `bundle exec jekyll build` succeeding and manual visual verification via `bundle exec jekyll serve`. All new visual effects (film grain, vignette, letterbox, duotone) are pure CSS/SCSS, no JS dependency. The homepage gets a brand-new standalone layout (bypasses the existing masthead/sidebar). Subpages keep the existing `default` → `single`/`archive` layout chain but get new color/typography variables and have their sidebar disabled.

**Tech Stack:** Jekyll, SCSS (existing `_sass` pipeline compiled via `assets/css/main.scss`), Liquid templates, `jekyll-redirect-from` plugin (already installed).

---

## Before you start

This repo's Ruby/Jekyll toolchain may not be installed yet. Run this once, from the repo root, before Task 1:

```bash
bundle install
```

Expected: gems install without errors (uses `github-pages` gem per `Gemfile`). If `bundle install` fails because of Ruby version mismatches, stop and report it — do not work around it by skipping build verification for later tasks.

Every task below ends with:

```bash
bundle exec jekyll build
```

Expected: exits 0, no Liquid/SCSS errors printed. Treat any error output as a failing test — fix before moving on.

---

### Task 1: Add the cinematic SCSS partial (grain / vignette / letterbox / duotone)

**Files:**
- Create: `_sass/theme/_cinematic.scss`
- Modify: `assets/css/main.scss`

- [ ] **Step 1: Create the partial**

```scss
/* ==========================================================================
   CINEMATIC HERO / BANNER
   Film-toned full-bleed hero (homepage) and short banner (subpage headers).
   ========================================================================== */

$cinematic-cream : #f7f2e9;
$cinematic-ink    : #4a4433;
$cinematic-gold   : #c2a15c;

@mixin cinematic-layers {
  .cinematic-photo {
    position: absolute;
    inset: 0;
    background-size: cover;
    background-position: center;
    filter: brightness(.62) contrast(1.15) saturate(.75);
  }

  .cinematic-teal-tint {
    position: absolute;
    inset: 0;
    background:
      linear-gradient(180deg, rgba(20, 40, 55, 0) 35%, rgba(10, 28, 40, .55) 100%),
      linear-gradient(90deg, rgba(10, 35, 50, .35), rgba(10, 35, 50, 0) 55%);
    mix-blend-mode: multiply;
  }

  .cinematic-grain {
    position: absolute;
    inset: 0;
    opacity: .28;
    mix-blend-mode: overlay;
    background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='120' height='120'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='2' stitchTiles='stitch'/></filter><rect width='100%25' height='100%25' filter='url(%23n)'/></svg>");
  }

  .cinematic-vignette {
    position: absolute;
    inset: 0;
    box-shadow: inset 0 0 130px 40px rgba(0, 0, 0, .7);
  }

  .cinematic-letterbox {
    position: absolute;
    left: 0;
    right: 0;
    height: 20px;
    background: #000;
    z-index: 2;

    &--top { top: 0; }
    &--bottom { bottom: 0; }
  }
}

.cinematic-hero {
  position: relative;
  width: 100%;
  height: 100vh;
  overflow: hidden;
  @include cinematic-layers;

  .cinematic-nav {
    position: absolute;
    top: 34px;
    left: 0;
    right: 0;
    padding: 0 34px;
    display: flex;
    justify-content: flex-end;
    z-index: 3;

    a {
      font-family: $serif;
      font-size: 11px;
      letter-spacing: .18em;
      text-transform: uppercase;
      color: $cinematic-cream;
      margin-left: 28px;
      text-decoration: none;

      &:hover { color: $cinematic-gold; }
    }
  }

  .cinematic-hero__text {
    position: absolute;
    bottom: 60px;
    left: 34px;
    z-index: 3;
    max-width: 90%;

    .kicker {
      font-family: $serif;
      font-size: 11px;
      letter-spacing: .2em;
      text-transform: uppercase;
      color: $cinematic-gold;
      margin-bottom: 12px;
    }

    h1 {
      font-family: $serif;
      font-weight: 400;
      font-size: 48px;
      letter-spacing: .01em;
      margin: 0;
      color: #f8f3e6;
      text-shadow: 0 2px 30px rgba(0, 0, 0, .75);
    }

    .gold-rule {
      width: 52px;
      height: 2px;
      background: $cinematic-gold;
      margin: 16px 0;
    }

    p {
      font-family: $serif;
      font-size: 13px;
      color: #d6cfbd;
      letter-spacing: .03em;
      margin: 0;
    }
  }
}

.cinematic-banner {
  position: relative;
  width: 100%;
  height: 200px;
  overflow: hidden;
  margin-bottom: 2em;
  @include cinematic-layers;
}

/* ==========================================================================
   EDITORIAL SUBPAGE ACCENTS
   ========================================================================== */

.editorial-rule {
  width: 32px;
  height: 2px;
  background: $cinematic-gold;
  margin: 1em 0;
}

.about-flex {
  display: flex;
  gap: 2em;
  align-items: flex-start;
  flex-wrap: wrap;

  img.about-portrait {
    width: 220px;
    height: 260px;
    object-fit: cover;
    border-radius: 3px;
    flex-shrink: 0;
  }
}
```

- [ ] **Step 2: Import it from `main.scss`**

In `assets/css/main.scss`, add `"theme/cinematic",` to the `@import` list, right after `"theme/dark",`:

```scss
@import
    "vendor/breakpoint/breakpoint",

    "themes",
    "theme/default",
    "theme/dark",
    "theme/cinematic",

    "include/mixins",
```

- [ ] **Step 3: Verify the build**

```bash
bundle exec jekyll build
```

Expected: exits 0, `_site/assets/css/main.css` contains `.cinematic-hero` (check with `grep cinematic-hero _site/assets/css/main.css`).

- [ ] **Step 4: Commit**

```bash
git add _sass/theme/_cinematic.scss assets/css/main.scss
git commit -m "style: add cinematic hero/banner SCSS partial"
```

---

### Task 2: Switch the global theme to warm editorial (serif type, cream background)

**Files:**
- Modify: `_sass/_themes.scss:42-43`
- Modify: `_sass/theme/_default.scss:30-45`

- [ ] **Step 1: Switch body/heading font to serif**

In `_sass/_themes.scss`, change:

```scss
$global-font-family         : $sans-serif;
$header-font-family         : $sans-serif;
```

to:

```scss
$global-font-family         : $serif;
$header-font-family         : $serif;
```

- [ ] **Step 2: Warm up the light theme palette**

In `_sass/theme/_default.scss`, replace the `:root` block:

```scss
/* Default light theme for the site */
:root {
    --global-base-color                 : #{$gray};
    --global-bg-color                   : #fff;
    --global-border-color               : #{$lighter-gray};
    --global-code-background-color      : #fafafa;
    --global-code-text-color            : #{$darker-gray};
    --global-fig-caption-color          : mix(#fff,  #{$dark-gray}, 25%);
    --global-link-color                 : #2f7f93;
    --global-link-color-hover           : mix(#000, #2f7f93, 25%);
    --global-link-color-visited         : mix(#fff, #2f7f93, 25%);  
    --global-masthead-link-color        : #{$dark-gray};
    --global-masthead-link-color-hover  : mix(#000, #{$gray}, 25%);    
    --global-text-color                 : #{$dark-gray};
    --global-text-color-light           : #{$light-gray};
    --global-thead-color                : #{$lighter-gray};
}
```

with:

```scss
/* Default light theme for the site (warm editorial) */
:root {
    --global-base-color                 : #8a7a5f;
    --global-bg-color                   : #f7f2e9;
    --global-border-color               : #e4dbc8;
    --global-code-background-color      : #fbf8f1;
    --global-code-text-color            : #4a4433;
    --global-fig-caption-color          : #8a7a5f;
    --global-link-color                 : #a3803f;
    --global-link-color-hover           : #7d6330;
    --global-link-color-visited         : #b89459;
    --global-masthead-link-color        : #4a4433;
    --global-masthead-link-color-hover  : #a3803f;
    --global-text-color                 : #4a4433;
    --global-text-color-light           : #8a7a5f;
    --global-thead-color                : #e4dbc8;
}
```

- [ ] **Step 3: Verify the build**

```bash
bundle exec jekyll build
```

Expected: exits 0.

- [ ] **Step 4: Manual visual check**

```bash
bundle exec jekyll serve
```

Open `http://localhost:4000/publications/` (any existing subpage) — background should now be warm cream, body text serif. Stop the server after checking (Ctrl-C).

- [ ] **Step 5: Commit**

```bash
git add _sass/_themes.scss _sass/theme/_default.scss
git commit -m "style: switch light theme to warm editorial serif palette"
```

---

### Task 3: Extend `page__hero.html` to support a cinematic banner

**Files:**
- Modify: `_includes/page__hero.html`
- Modify: `_layouts/single.html:7-9`
- Modify: `_layouts/archive.html:5-7`

Subpages opt into a short cinematic banner (grain/vignette/letterbox, same as the homepage hero but shorter) by setting `header.cinematic_image` (+ `header.cinematic_banner: true`) in front matter. This is deliberately a **separate** key from the theme's existing `header.overlay_image` — `single.html`/`archive.html` suppress their own `<h1>` page title whenever `header.overlay_image` is set (they assume the old hero already renders the title). We still want the normal `<h1>` title to render below our banner, so the banner must not use `overlay_image`.

- [ ] **Step 1: Add the cinematic branch to `page__hero.html`**

Replace the whole file content with:

```html
{% include base_path %}

{% if page.header.image contains "://" %}
  {% capture img_path %}{{ page.header.image }}{% endcapture %}
{% else %}
  {% capture img_path %}{{ page.header.image | prepend: "/images/" | prepend: base_path }}{% endcapture %}
{% endif %}

{% if page.header.cta_url contains "://" %}
  {% capture cta_path %}{{ page.header.cta_url }}{% endcapture %}
{% else %}
  {% capture cta_path %}{{ page.header.cta_url | prepend: base_path }}{% endcapture %}
{% endif %}

{% if page.header.overlay_image contains "://" %}
  {% capture overlay_img_path %}{{ page.header.overlay_image }}{% endcapture %}
{% elsif page.header.overlay_image %}
  {% capture overlay_img_path %}{{ page.header.overlay_image | prepend: "/images/" | prepend: base_path }}{% endcapture %}
{% endif %}

{% if page.header.overlay_filter contains "rgba" %}
  {% capture overlay_filter %}{{ page.header.overlay_filter }}{% endcapture %}
{% elsif page.header.overlay_filter %}
  {% capture overlay_filter %}rgba(0, 0, 0, {{ page.header.overlay_filter }}){% endcapture %}
{% endif %}

{% if page.header.cinematic_image contains "://" %}
  {% capture cinematic_img_path %}{{ page.header.cinematic_image }}{% endcapture %}
{% elsif page.header.cinematic_image %}
  {% capture cinematic_img_path %}{{ page.header.cinematic_image | prepend: "/images/" | prepend: base_path }}{% endcapture %}
{% endif %}

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
  <div class="page__hero{% if page.header.overlay_color or page.header.overlay_image %}--overlay{% endif %}"
    style="{% if page.header.overlay_color %}background-color: {{ page.header.overlay_color | default: 'transparent' }};{% endif %} {% if overlay_img_path %}background-image: {% if overlay_filter %}linear-gradient({{ overlay_filter }}, {{ overlay_filter }}), {% endif %}url('{{ overlay_img_path }}');{% endif %}"
  >
    {% if page.header.overlay_color or page.header.overlay_image %}
      <div class="wrapper">
        <h1 class="page__title" itemprop="headline">
          {% if paginator %}
            {{ site.title }}{% unless paginator.page == 1 %} {{ site.data.ui-text[site.locale].page | default: "Page" }} {{ paginator.page }}{% endunless %}
          {% else %}
            {{ page.title | default: site.title | markdownify | remove: "<p>" | remove: "</p>" }}
          {% endif %}
        </h1>
        {% if page.excerpt %}
          <p class="page__lead">{{ page.excerpt | markdownify | remove: "<p>" | remove: "</p>" }}</p>
        {% endif %}
        {% if site.read_time and page.read_time %}
          <p class="page__meta"><i class="fa fa-clock" aria-hidden="true"></i> {% include read-time.html %}</p>
        {% endif %}
        {% if page.header.cta_url %}
          <p><a href="{{ cta_path }}" class="btn btn--light-outline btn--large">{{ page.header.cta_label | default: site.data.ui-text[site.locale].more_label | default: "Learn More" }}</a></p>
        {% endif %}
      </div>
    {% else %}
      <img src="{{ img_path }}" alt="{{ page.title }}" class="page__hero-image">
    {% endif %}
    {% if page.header.caption %}
      <span class="page__hero-caption">{{ page.header.caption | markdownify | remove: "<p>" | remove: "</p>" }}</span>
    {% endif %}
  </div>
{% endif %}
```

This is a pure addition — the pre-existing `overlay_color`/`overlay_image` behavior (used elsewhere in the theme) is untouched inside the `{% else %}` branch.

- [ ] **Step 2: Make `single.html` and `archive.html` trigger the include for `cinematic_image` too**

Both layouts only call `{% include page__hero.html %}` when `page.header.overlay_color or page.header.overlay_image or page.header.image` is set — our new pages use `cinematic_image` instead, so without this change the banner would never render.

In `_layouts/single.html`, change:

```liquid
{% if page.header.overlay_color or page.header.overlay_image or page.header.image %}
  {% include page__hero.html %}
{% endif %}
```

to:

```liquid
{% if page.header.overlay_color or page.header.overlay_image or page.header.image or page.header.cinematic_image %}
  {% include page__hero.html %}
{% endif %}
```

Make the identical change in `_layouts/archive.html` (same 3 lines, near the top of the file, right after the `layout: default` front matter).

Note this only affects whether `page__hero.html` is *invoked* — it does not touch the `unless page.header.overlay_color or page.header.overlay_image` checks further down in each layout that suppress the normal `<h1>` title block. Since our pages set `cinematic_image` and not `overlay_image`, that check still evaluates false, so the normal title still renders below the banner. That's the whole point of using a separate key.

- [ ] **Step 3: Verify the build**

```bash
bundle exec jekyll build
```

Expected: exits 0.

- [ ] **Step 4: Commit**

```bash
git add _includes/page__hero.html _layouts/single.html _layouts/archive.html
git commit -m "feat: add cinematic banner mode to page__hero include"
```

---

### Task 4: Build the standalone cinematic home layout

**Files:**
- Create: `_layouts/home-cinematic.html`

- [ ] **Step 1: Create the layout**

```html
---
layout: compress
---

{% include base_path %}

<!doctype html>
<html lang="{{ site.locale | slice: 0,2 }}" class="no-js">
  <head>
    {% include head.html %}
    {% include head/custom.html %}
  </head>

  <body class="layout--home-cinematic">

    {% if page.header.cinematic_image contains "://" %}
      {% capture cinematic_img %}{{ page.header.cinematic_image }}{% endcapture %}
    {% else %}
      {% capture cinematic_img %}{{ page.header.cinematic_image | prepend: "/images/" | prepend: base_path }}{% endcapture %}
    {% endif %}

    <div class="cinematic-hero">
      <div class="cinematic-photo" style="background-image: url('{{ cinematic_img }}');"></div>
      <div class="cinematic-teal-tint"></div>
      <div class="cinematic-grain"></div>
      <div class="cinematic-vignette"></div>
      <div class="cinematic-letterbox cinematic-letterbox--top"></div>
      <div class="cinematic-letterbox cinematic-letterbox--bottom"></div>

      <nav class="cinematic-nav">
        {% for link in site.data.navigation.main %}
          {% if link.url contains 'http' %}
            {% assign nav_domain = '' %}
          {% else %}
            {% assign nav_domain = base_path %}
          {% endif %}
          <a href="{{ nav_domain }}{{ link.url }}">{{ link.title }}</a>
        {% endfor %}
      </nav>

      <div class="cinematic-hero__text">
        {% if page.kicker %}<div class="kicker">{{ page.kicker }}</div>{% endif %}
        <h1>{{ page.title | default: site.author.name }}</h1>
        <div class="gold-rule"></div>
        {% if page.subtitle %}<p>{{ page.subtitle }}</p>{% endif %}
      </div>
    </div>

    {% include scripts.html %}

  </body>
</html>
```

Note: no `masthead.html`, no `sidebar.html`, no `footer.html` — this is intentionally a single immersive screen, per the approved design (pure landing page).

- [ ] **Step 2: Verify the build**

```bash
bundle exec jekyll build
```

Expected: exits 0 (layout isn't used by any page yet, so this just checks Liquid syntax is valid).

- [ ] **Step 3: Commit**

```bash
git add _layouts/home-cinematic.html
git commit -m "feat: add standalone home-cinematic layout"
```

---

### Task 5: Wire up the homepage content and move About off `/`

**Files:**
- Create: `_pages/home.md`
- Modify: `_pages/about.md`
- Add: `images/hero-boya-tower.jpg` (already present on disk, currently untracked)

`_pages/about.md` currently owns `permalink: /`. It needs to move to `/about/` so the new cinematic layout can take over `/`.

- [ ] **Step 1: Repoint About's permalink**

In `_pages/about.md`, replace the front matter:

```yaml
---
permalink: /
title: "About me"
author_profile: true
redirect_from: 
  - /about/
  - /about.html
---
```

with:

```yaml
---
permalink: /about/
title: "About"
author_profile: false
header:
  cinematic_image: hero-boya-tower.jpg
  cinematic_banner: true
---
```

(`author_profile: false` disables the old sidebar — Task 6 covers About's new two-column layout.)

- [ ] **Step 2: Create the home page**

```yaml
---
permalink: /
layout: home-cinematic
title: "Qiaoshi Yang"
kicker: "Finance × Applied Math"
subtitle: "PhD Candidate, UPenn AMCS"
header:
  cinematic_image: hero-boya-tower.jpg
---
```

Save as `_pages/home.md`. No body content is needed — `home-cinematic.html` doesn't render `{{ content }}` at all.

- [ ] **Step 3: Track the hero image**

```bash
git add images/hero-boya-tower.jpg
```

- [ ] **Step 4: Verify the build**

```bash
bundle exec jekyll build
```

Expected: exits 0. Check `_site/index.html` contains `class="cinematic-hero"` (`grep cinematic-hero _site/index.html`).

- [ ] **Step 5: Manual visual check**

```bash
bundle exec jekyll serve
```

Open `http://localhost:4000/` — full-bleed graded photo, right-aligned nav (About · Research · Blog · Talks & Teaching · CV — links will 404 until later tasks land, that's expected), name/kicker/subtitle bottom-left, letterbox bars top/bottom. Stop the server after checking.

- [ ] **Step 6: Commit**

```bash
git add _pages/home.md _pages/about.md
git commit -m "feat: add cinematic homepage, move About to /about/"
```

---

### Task 6: Rebuild the About page (portrait + bio, editorial layout)

**Files:**
- Modify: `_pages/about.md`

- [ ] **Step 1: Replace the body content**

`_pages/about.md` currently has the bio as a flat block of paragraphs. Replace everything below the front matter (keep the front matter from Task 5 Step 1 unchanged) with:

```markdown
<div class="about-flex">
  <img src="/images/profile.png" alt="Qiaoshi Yang" class="about-portrait">
  <div>

My name is Qiaoshi Yang (杨 巧诗). I'm a final year undergraduate student from [Guanghua School of Management](https://www.gsm.pku.edu.cn/), [Peking University](https://www.pku.edu.cn/) and will be a Ph.D. candidate in Upenn AMCS. My research interest includes network analysis, adversarial training, transformer and medical AI.

I am very fortunate to be advised by [Prof. Song Xi Chen](https://songxichen.com/), [Prof. Yumou Qiu](https://yumou.org/) and [Prof. Cong Wang](https://www.gsm.pku.edu.cn/faculty/wangcong/). I learned a lot in the process of studying with these professors, and felt the fun of scientific research!😄

In my free time, I enjoy reading detective novels (Atticus Pünd, Poirot, and Ellery Queen), long-distance running, and painting. I also love watching Japanese anime. Additionally, I'm a huge fan of the Persona series!🎮

<div class="editorial-rule"></div>

[Email](mailto:sy413102@outlook.com) / [Github](https://github.com/yang-qs)

  </div>
</div>
```

- [ ] **Step 2: Verify the build**

```bash
bundle exec jekyll build
```

Expected: exits 0.

- [ ] **Step 3: Manual visual check**

```bash
bundle exec jekyll serve
```

Open `http://localhost:4000/about/` — short graded banner at top, no sidebar, portrait photo to the left of the bio text, warm cream background. Stop the server after checking.

- [ ] **Step 4: Commit**

```bash
git add _pages/about.md
git commit -m "feat: rebuild About page with portrait + editorial layout"
```

---

### Task 7: Update Research (Publications), Blog, and CV pages

**Files:**
- Modify: `_pages/publications.html`
- Modify: `_pages/blog.html`
- Modify: `_pages/cv.md`

- [ ] **Step 1: Publications → Research**

In `_pages/publications.html`, replace the front matter:

```yaml
---
layout: archive
title: "Publications"
permalink: /publications/
author_profile: true
---
```

with:

```yaml
---
layout: archive
title: "Research"
permalink: /publications/
author_profile: false
header:
  cinematic_image: hero-boya-tower.jpg
  cinematic_banner: true
---
```

(URL stays `/publications/` — only the nav label changes to "Research", per the approved design. No need to touch the Liquid loop below the front matter.)

- [ ] **Step 2: Blog**

In `_pages/blog.html`, replace the front matter:

```yaml
---
layout: archive
title: "Blog"
permalink: /blog/
author_profile: true
---
```

with:

```yaml
---
layout: archive
title: "Blog"
permalink: /blog/
author_profile: false
header:
  cinematic_image: hero-boya-tower.jpg
  cinematic_banner: true
---
```

- [ ] **Step 3: CV — add contact links, disable sidebar, add banner**

In `_pages/cv.md`, replace the front matter:

```yaml
---
layout: archive
title: "CV"
permalink: /cv/
redirect_from:
  - /resume
---
```

with:

```yaml
---
layout: archive
title: "CV"
permalink: /cv/
author_profile: false
redirect_from:
  - /resume
header:
  cinematic_image: hero-boya-tower.jpg
  cinematic_banner: true
---
```

Then, immediately below the front matter (before the `Education` section), add a contact line — since the sidebar (which used to show email/GitHub icons) is now disabled:

```markdown
[Email](mailto:sy413102@outlook.com) / [Github](https://github.com/yang-qs)

<div class="editorial-rule"></div>

Education
======
```

(replace the existing bare `Education\n======` line with the block above, which now includes it).

- [ ] **Step 4: Verify the build**

```bash
bundle exec jekyll build
```

Expected: exits 0.

- [ ] **Step 5: Manual visual check**

```bash
bundle exec jekyll serve
```

Open `http://localhost:4000/publications/`, `http://localhost:4000/blog/`, `http://localhost:4000/cv/` — each should show the graded banner, no sidebar, warm cream background, and the CV page should show the Email/Github links above "Education". Stop the server after checking.

- [ ] **Step 6: Commit**

```bash
git add _pages/publications.html _pages/blog.html _pages/cv.md
git commit -m "feat: apply cinematic banner + editorial layout to Research, Blog, CV"
```

---

### Task 8: Merge Talks and Teaching into one page

**Files:**
- Modify: `_pages/talks.html`
- Delete: `_pages/teaching.html`

- [ ] **Step 1: Rewrite talks.html to include both collections**

Replace the entire contents of `_pages/talks.html` with:

```html
---
layout: archive
title: "Talks & Teaching"
permalink: /talks/
author_profile: false
redirect_from:
  - /teaching/
header:
  cinematic_image: hero-boya-tower.jpg
  cinematic_banner: true
---

{% include base_path %}

{% if site.talkmap_link == true %}
  <p style="text-decoration:underline;"><a href="/talkmap.html">See a map of all the places I've given a talk!</a></p>
{% endif %}

<h2>Talks</h2>

{% for post in site.talks reversed %}
  {% include archive-single-talk.html %}
{% endfor %}

<div class="editorial-rule"></div>

<h2>Teaching</h2>

{% for post in site.teaching reversed %}
  {% include archive-single.html %}
{% endfor %}
```

- [ ] **Step 2: Remove the standalone Teaching page**

```bash
git rm _pages/teaching.html
```

`jekyll-redirect-from` (already in `Gemfile`/`_config.yml` plugins) will generate a static redirect at `/teaching/` → `/talks/` from the `redirect_from` front matter added in Step 1.

- [ ] **Step 3: Verify the build**

```bash
bundle exec jekyll build
```

Expected: exits 0. Confirm the redirect stub exists: `ls _site/teaching/index.html`.

- [ ] **Step 4: Manual visual check**

```bash
bundle exec jekyll serve
```

Open `http://localhost:4000/talks/` — should show both Talks and Teaching sections under the graded banner. Open `http://localhost:4000/teaching/` — should redirect to `/talks/`. Stop the server after checking.

- [ ] **Step 5: Commit**

```bash
git add _pages/talks.html
git commit -m "feat: merge Teaching into Talks page with redirect"
```

---

### Task 9: Simplify navigation to 5 items

**Files:**
- Modify: `_data/navigation.yml`

- [ ] **Step 1: Replace the nav list**

Replace the entire contents of `_data/navigation.yml` with:

```yaml
# The following is the order of the links in the header of the website.
#
# Changing the order here will adjust the order and you can also add additional
# links. Removing a link prevents it from showing in the header, but does not
# prevent it from being included in the site.

main:
  - title: "About"
    url: /about/

  - title: "Research"
    url: /publications/

  - title: "Blog"
    url: /blog/

  - title: "Talks & Teaching"
    url: /talks/

  - title: "CV"
    url: /cv/
```

- [ ] **Step 2: Verify the build**

```bash
bundle exec jekyll build
```

Expected: exits 0.

- [ ] **Step 3: Manual visual check**

```bash
bundle exec jekyll serve
```

Open `http://localhost:4000/` and click through every nav link (both the cinematic homepage nav and the regular masthead nav on subpages) — all 5 should resolve without 404s. Stop the server after checking.

- [ ] **Step 4: Commit**

```bash
git add _data/navigation.yml
git commit -m "feat: simplify site navigation to 5 sections"
```

---

### Task 10: Full-site QA pass

**Files:** none (verification only)

- [ ] **Step 1: Clean build**

```bash
rm -rf _site
bundle exec jekyll build
```

Expected: exits 0, no warnings about missing layouts/includes.

- [ ] **Step 2: Full manual walkthrough**

```bash
bundle exec jekyll serve
```

Check, in order:
1. `/` — cinematic hero fills the viewport, no scrollbar-inducing extra content, nav top-right, name/kicker/subtitle bottom-left, letterbox bars visible top and bottom.
2. `/about/` — banner + portrait + bio, no sidebar.
3. `/publications/` — banner, title reads "Research" in both the `<title>` tag and on-page heading, publication list still renders.
4. `/blog/` — banner, post list still renders.
5. `/talks/` — banner, Talks section then Teaching section, both non-empty if source data exists.
6. `/teaching/` — redirects to `/talks/`.
7. `/cv/` — banner, Email/Github links above Education, rest of CV content intact.
8. Masthead nav (visible on every subpage) shows exactly: About, Research, Blog, Talks & Teaching, CV — no leftover "Publications"/"Teaching" labels.
9. Resize the browser to mobile width on `/` and one subpage — hero text and nav should not overflow horizontally.

Stop the server after checking (Ctrl-C).

- [ ] **Step 3: Fix anything broken, then re-run Step 1**

Do not proceed to marking this task done until Step 1 and Step 2 both pass clean.
