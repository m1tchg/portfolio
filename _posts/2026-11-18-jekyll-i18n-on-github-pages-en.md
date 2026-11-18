---
categories:
- article
lang: en
title: "My i18n portfolio: Jekyll and Polyglot on GitHub Pages"
permalink: "/posts/jekyll-i18n-on-github-pages"
---

Check out my GitHub Pages portfolio. It has views for English and Japanese. 

I stumbled constantly. Use my site as a template. Internationalize your own Jekyll site with Polyglot. Or start from scratch on GitHub Pages.

It's tempting to overthink your portfolio site. I tried Hugo's built-in internationalization. Ultimately, though, Polyglot's "don't overthink it" personality won me back from Hugo.


## Set up a new site

If you're starting your site from scratch, find your shortest path:

1.  Pick a theme, or not. Will you have to rewrite all of the layout files anyway so you can localize UI strings?

    The Chirpy theme ships with translations. But you only get one site. You need Polyglot to build a version of the site for each language.

    To follow along with Chirpy, create your repo with Chirpy's [starter template](https://github.com/cotes2020/chirpy-starter). Click Use This Template at the top, then click Create New Repository.

2.  Configure how to publish your site. You can publish with a GitHub Actions workflow or when you push to a branch.

    Your starter template, like Chirpy's, might include a GitHub Actions workflow. To use the starter template's GitHub Action, configure your repo's publishing settings:

    1.  From your repo on GitHub, click Settings, at the top. You might need to click More \> Settings.
    2.  In the sidebar, go to Pages, then under Build and Deployment, select GitHub Actions in the Source menu.
    3.  Trigger the workflow. With Chirpy, push a commit to the main branch.

    If you're not using a starter template, consider publishing from a branch. See [Tips and troubleshooting](#tips-and-troubleshooting).

3.  To preview your changes, open a GitHub codespace or clone the repo to your machine and [install Jekyll](https://jekyllrb.com/docs/installation/).

    In a terminal, run `bundle install` to install the gems. Run `bundle exec jekyll serve` to build the site and start the development server.

## Install Polyglot

In your Gemfile, add the jekyll-polyglot gem.

``` ruby
group :jekyll_plugins do
  gem "jekyll-polyglot"
end
```

In \_config.yml, you can mostly paste the following YAML block.

``` yaml
languages: ["en", "ja-JP"]
default_lang: "en"
url: https://<your-username>.github.io/<your-repo>
exclude_from_localization: ["javascript", "images", "css", "public", "sitemap", "CNAME"]
parallel_localization: true
serial_default_lang: true
```

If you run into trouble, make sure you set the prefs you need:

- Set `languages` to the languages you'll support. Specify the [locale code](https://developer.chrome.com/docs/extensions/reference/api/i18n#locales).

- Set `default_lang` to the fallback language used for other languages.

- Make sure `url` is set to the production URL of your Jekyll website.

  The starter theme might not set `url` for you. Or it might unset this for you, if `url` is set later in the file.

- On Windows, set `parallel_localization` to false.

See the [Polyglot README](https://github.com/untra/polyglot/blob/1.14.0/README.md#configuration) for more options.

## Localize your posts

In the YAML frontmatter of your post, set `lang`. You're ready to write and translate.

``` yaml
---
lang: "ja-JP"
title: GitHub Pagesのポートフォリオをローカライズしてみた 
# I tried localizing my GitHub Pages portfolio
---
```

Before you start your next post, check out my highlights from Polyglot's [README](https://github.com/untra/polyglot):

- Polyglot localizes the URLs in your posts. Links like `[See my other posts](/menu)` become `[他の投稿もご覧ください](/ja/menu)`.
- Get the current language site's URL with `{% raw %}{{ site.url }}{% endraw %}`. A link like `[ここ]({% raw %}{{ site.url }}{% endraw %})` renders to `<a href="https://mywebsite.com/ja/">ここ</a>`.

When you add a post and its translation, use consistent file naming. Jekyll posts start with the date.

``` txt
_posts/2026-11-09-localized-gh-pages-portfolio-en.md
_posts/2026-11-09-localized-gh-pages-portfolio-ja.md
```

If you organize your posts, Polyglot generates consistent [permalinks](https://jekyllrb.com/docs/permalinks/). To localize pages other than posts, set permalinks in the frontmatter. Point all the language versions of a page to the same permalink.

``` yaml
---
lang: "ja-JP"
permalink: /about
---
```

I needed to set `permalink` in my posts since I needed to match `lang` with the `ja-JP` locale that Chirpy uses. Polyglot's permalink generator seemed to choke on the hyphen.

You can localize the permalink string as well. For example, on a Spanish site, you might want URLs to end with `/acerca-de` instead of `/about`. See the [Polyglot README](https://github.com/untra/polyglot#using-different-permalinks-per-language).

## Internationalize strings

Before launch, translate all of your navbars, dropdowns, and other strings that make up your rich content. Rich content keeps visitors on your site.

If your theme doesn't support multiple languages, you need to internationalize the theme's templates. That involves replacing all the strings with `site.data` variables. The strings move out of the template HTML files in the `_includes` and `_layouts` folders. Save them in [data files](https://jekyllrb.com/docs/datafiles/) in the `_data` directory.

Chirpy keeps translated strings in `_data/locales/:lang:.yml`. Here's how Chirpy localizes the title of the Table of Contents in `_layouts/post.html`.

``` liquid
{% raw %}{{- site.data.locales[lang].panel.toc -}}{% endraw %}
```

Chirpy defines a `lang` variable that you set in `_config.yml`. It's the language for the site.

But `lang` only helps you localize one site. Polyglot's Liquid tools help you stay DRY when you're building separate sites for each language.

### Localizing the sidebar and dynamic pages

When you're creating dynamic pages, like all the articles with a tag, it's a pain to set `lang` in the YAML frontmatter, like you do with posts. You end up overriding all of your theme's files.

You can use Polyglot's `site.active_lang` variable to change the language of dynamic content with one line, in Chirpy at least. That sidebar stuck in the default language is an example.

**Working across sites**

Polyglot sets `site.active_lang` on each pass as it builds each language site. So you can rebuild the sidebar for each language site.

**Localizing the sidebar**

On dynamic pages, like the home page, the `lang` variable isn't set in the page's YAML frontmatter. It falls back to the default site language.

In Chirpy's `_includes/lang.html`, you can insert Polyglot's `site.active_lang` into the chain of fallbacks.

``` liquid
{% raw %}{% if site.data.locales[page.lang] %}
  {% assign lang = page.lang %}

<!-- Add a check for Polyglot's site.active_lang -->
{% elsif site.active_lang] %}
  {% assign lang = site.active_lang %}

{% elsif site.data.locales[site.lang] %}
  {% assign lang = site.lang %}
{% else %}
  {% assign lang = 'en' %}
{% endif %}{% endraw %}
```


See [Tips and Troubleshooting](#tips-and-troubleshooting) to override theme files.

## Create a language switcher

You can loop over Polyglot's `site.languages` variable to do things like output links to each supported language. That's the approach of the Polyglot site.

I added my supported language list to Chirpy's `_includes/sidebar.html`.

``` liquid
{% raw %}<!-- .sidebar-bottom -->
<div class="sidebar-bottom d-flex flex-wrap align-items-center w-100">
  <p>
    {% for tongue in site.languages %}
    <a{% if tongue == site.active_lang %} style="font-weight: bold;"{% endif %} href="{% if tongue == site.default_lang %}{{site.baseurl}}{{page.url}}{% else %}{{site.baseurl}}/{{ tongue }}{{page.url}}{% endif %}" >{{ tongue }}</a>{%- if forloop.last == false -%}{{" "}}&bull;{%- endif -%}
    {% endfor %}
  </p>{% endraw %}
</div>
```

## Technical SEO localization

Without the right technical SEO, translations can depress your page rank. For example, you need to tell Google that translations aren't duplicates.

Polyglot provides SEO recipes that help you complete your [SEO localization checklist](https://crowdin.com/blog/seo-localization-guide).

### SEO headers

Add Polyglot's SEO headers to Chirpy's `_includes/head.html`.

``` liquid
{% raw %}{%- capture seo_tags -%}
  {% seo title=false canonical=false %}
  <!-- Add the Polyglot headers -->
  {% I18n_Headers %}
{%- endcapture -%}

...
{% seo_tags %}{% endraw %}
```

Polyglot's [SEO guide](https://polyglot.untra.io/seo/) explains more about how Polyglot layers on top of the `jekyll-seo-tag` plugin.

### Localizing sitemap.xml and robots.txt

Polyglot also provides Liquid templates for sitemap.xml and robots.txt. Get them from the [Polyglot README](https://github.com/untra/polyglot#sitemap-generation). Drop sitemap.xml and robots.txt into the root folder of your repo. That's enough to replace the files Chirpy generates.

You want only one sitemap.xml that links to all of the language sites, not a sitemap.xml for each language site. The sitemap.xml file lives under the default language site.

But each language site needs its own [robots.txt](https://developers.google.com/crawling/docs/robots-txt/robots-txt-spec#examples-of-valid-robotstxt-urls), according to Google. Each robots.txt file points back to the sitemap.xml at the root of the default language site.

Submit your sitemap.xml in the Google Search Console to get your portfolio crawled.

## Tips and troubleshooting

### Publish from a branch

If you're not using a starter template that ships a GitHub Action, you need a strategy to publish your site. Consider Polyglot's [recommended steps](https://polyglot.untra.io/about#github-pages-support) to build your site locally, then publish the static files to a branch, like `gh-pages`. You can find a guide to [publish to a branch](https://polyglot.untra.io/about/) in the GitHub Docs.

When you're building locally, make sure you generate URLs for your production site. You can set an environment variable.

``` bash
JEKYLL_ENV=production bundle exec jekyll build
```

If all the links are to http://localhost and the site's published, you'll only see text. The other resources won't load.

### Overriding theme files

To override a theme file, copy the file into same file path in your repository.

Chirpy's a gem-based theme, so all the theme data lives in your local gem directory. To find the gem directory on your machine, run the following command.

``` bash
bundle info --path jekyll-theme-chirpy
```

On my machine, the gem directory is /home/mitch/gems/gems/jekyll-theme-chirpy-7.6.0/.
