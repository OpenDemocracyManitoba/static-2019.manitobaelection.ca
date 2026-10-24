# 2019.manitobaelection.ca (static archive)

Static HTML copy of **2019.manitobaelection.ca**, Open Democracy Manitoba's guide to the 2019 Manitoba provincial election. It ran on WordPress 5.9.7, with the ElectionData-V2 theme supplied by the ElectionData plugin. It was crawled from olddroplet (`odm-nginx-ruby-php-2018`, 159.203.24.194) on 2026-10-24, before the server was retired.

* `site/` holds the crawl: 2,021 files in the raw crawl commit.
* `crawl.log.gz` holds the wget log.

The WordPress source, the DB dump (`2020_manitobaelection`) and the nginx vhost are in the private repo `OpenDemocracyManitoba/archive-2019.manitobaelection.ca`.

## How it was crawled

The crawl ran from a workstation against the origin directly (an `/etc/hosts` entry, bypassing Cloudflare). wp-cron was disabled and the site was frozen first.

```
wget --mirror --page-requisites --adjust-extension --restrict-file-names=windows \
     --no-host-directories --directory-prefix=site \
     -e robots=off --wait=0.1 --user-agent='ODM-archiver/1.0' \
     --reject-regex='(/wp-json|[?&]rest_route=|xmlrpc\.php|wp-login\.php|/wp-admin|[?&]replytocom=|/feed/?$|/trackback/?$|[?&]s=|/candidates/[^/]+/[0-9]+/|[?&]p=[0-9]+)' \
     -o crawl.log https://2019.manitobaelection.ca/
```

* It ran on 2026-10-24, 12:10–12:19 CDT, and downloaded 2,018 files. wget exited 8 because some URLs returned 404.
* The last two reject patterns skip a crawler trap and WordPress `/?p=N` shortlinks:
  * `/candidates/<slug>/<N>/` is WordPress's path form of a paged candidate page, and each one has its own pager. See the 2016 archive for the details.
  * `/?p=N` shortlinks are duplicates of pages saved at their real paths, so they aren't archived (owner decision, 2026-10-24).
* There is no `--convert-links`, so the saved HTML keeps its original absolute `https://2019.manitobaelection.ca/…` hrefs. The pagers use root-relative ones.
* `filter.php` (a `phpinfo()` demo in the web root) is not linked from anywhere and is not in the crawl.

Three files are referenced only where wget doesn't look, so they were copied in from the source tree:

* `wp-content/uploads/2019/05/cropped-mb_icon-270x270.jpg` (icon `<meta>` tags);
* `wp-includes/js/wp-emoji-release.min.js` (inline JS);
* `wp-content/themes/ElectionData/ElectionData-V2/js/html5.js` (an IE conditional comment).

On the server, `wp-content/themes/ElectionData` is a symlink to `wp-content/plugins/ElectionData/theme/`, so `html5.js` was taken from there.

## Changes made on the live site before the crawl

The DB wasn't modified. The private archive repo has the untouched source and dump.

* **wp-cron disabled** (`DISABLE_WP_CRON`, and nginx returns 403 for `/wp-cron.php`). This stopped the news scraper, which had kept running until 2026 and was pinning the DB server.
* **News cutoff** (owner decision, 2026-10-24). The scraper had collected news long after the election. The mu-plugin `wp-content/mu-plugins/odm-archive-news-cutoff.php` hides `ed_news_articles` published after **2019-11-10 23:59:59**, two months after election day (2019-09-10). The full news table is still in the DB dump.
* **Candidate news paging fixed.** Candidate pages link to `?page=N#news`, but WordPress 5.9 301'd every `?page=N` back to page 1 (`redirect_canonical`, and `WP::handle_404` before it). The mu-plugin `wp-content/mu-plugins/odm-archive-candidate-paging.php` turns both off for `ed_candidates` with `page` > 1.
* **WP-Super-Cache:** its cache was moved aside, so the crawl saw freshly rendered pages. The cache comments in the HTML come from the crawl itself.

## Completeness

|Check|Result|
|-|-|
|Candidate pages|235 of 235 published `ed_candidates`|
|Paginated candidates|69, with 454 extra news pages. None missing, none identical to page 1.|
|News|2,361 of the 2,381 approved news articles up to the cutoff appear on some page (details below). None after the cutoff appear.|
|Questionnaire answers|266 of 266 question/answer pairs, on 69 candidate pages. The DB has 268 non-empty `ed_answers` out of 976; the other 2 are the Green Party's answers, shown on `parties/green/`.|
|Uploads|Every referenced `wp-content/uploads/` file on this host is in the crawl. The crawl holds 1,145 of the 1,574 files in the source uploads tree; the rest aren't linked from any page.|
|Asset CDNs|no `i0.wp.com` / `c0.wp.com` references|

**News.** There is no paginated news archive on this site: `news_articles/` shows the latest 90 items, and older news is reachable only through the candidate pages.

The plugin moderates news, and only `approved` articles were ever shown. Of the 54,373 published articles, 14,684 are approved; 39,645 are `new` and 44 are `rejected`. 2,382 approved articles fall on or before the cutoff, and 2,381 of those mention a candidate.

2,361 of them appear in the crawl. The other 20 couldn't be reached on the live site either. The cause is in the theme's `display_news_summaries()`, and **it's archived as is** (owner decision, 2026-10-24):

* The pager computes `total = round(count / 10)`, so a candidate's last page is never linked when it would hold fewer than 5 articles. 16 articles are lost this way; for example, Wab Kinew has 722 articles but 72 pages, and Heather Stefanson has 73 but 7.
* 2 mention only Jeff Wharton, who has no candidate page.
* 2 have an empty summary for every candidate they mention, and an article with no summary for that candidate is skipped. That's why some pages show fewer than 10 items.

The candidate pages carry the theme's notice "Showing news articles published after June 1st 2019". That's only text: `get_news()` has no date filter.

**Party pages.** The party template links its news pager as `?page=N`, but it reads WordPress's `paged` variable, so on the live site every `?page=N` showed page 1's news again. wget saved the two pages each pager linked (`?page=2` and the last page) for the 4 parties. Their news lists are the same as page 1. No news is lost, since every article is also on a candidate page.

**404s:** only `wp-content/themes/ElectionData/ElectionData-V2/ipad4/arrow.gif` and `arrowl.gif`, which are referenced by the theme CSS and missing on the live site too.

## Links to other hosts

* 2 files (`frequently-asked-questions/` and its slashless copy) link to `http://www.manitobaelection.ca/frequently-asked-questions/#news`, the "news gathering process" explanation on the provincial site. That's a real cross-site link, left as it is.
* 4 files (`about-manitobaelection-ca/`, `about-open-democracy-manitoba/`, `press-releases/` and its slashless copy) link to 9 URLs on `2016.manitobaelection.ca`: older press releases, their PDFs and images, and the 2016 FAQ and press pages. All of them are in the 2016 static archive.

There were no references to `upcoming.manitobaelection.ca` (the DB's old `home`) or `159.203.24.194`.

## wget quirk, left as is

`frequently-asked-questions.html`, `press-mentions.html` and `press-releases.html` at the root are wget's saves of slashless links (e.g. `/press-releases`). The server 301'd each one to the slash form, and wget saved the redirect target under the slashless name. Static hosting with `auto-trailing-slash` serves `/press-releases` from `press-releases.html`.

## Commits and URL mapping

1. **Raw crawl.** `--restrict-file-names=windows` saved each URL's `?` as `@`.

## Secret scan

The Phase A §7.2 scan was run over `site/` and found nothing. It checks for:

* no `.git`, `.env`, `.ftpconfig`, `.htpasswd`, `debug.log`, `*.php` or `*.sql*` files;
* no `DB_PASSWORD`, `AUTH_KEY`, `SECURE_AUTH`, private keys, `phpinfo()` or `mysql(i)_connect` in any text file;
* no `*.php.html` pages.
