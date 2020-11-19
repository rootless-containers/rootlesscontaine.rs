## `rootlesscontaine.rs` ##

[`rootlesscontaine.rs`][rc] is a single-purpose website for tracking the
progress of rootless container support in various projects. I'm not good at
this whole "web development" gig, so I apologise for everything.

[rc]: https://rootlesscontaine.rs/

### Usage ###

This site is "built" using [Hugo][hugo], so just run the following:

```
% hugo server
Start building sites …

                   | EN
-------------------+-----
  Pages            | 40
  Paginator pages  |  0
  Non-page files   |  0
  Static files     | 38
  Processed images |  0
  Aliases          |  0
  Sitemaps         |  1
  Cleaned          |  0

Built in 69 ms
Watching for changes in /home/cyphar/.local/src/rootlesscontaine.rs/{content,themes}
Watching for config changes in /home/cyphar/.local/src/rootlesscontaine.rs/config.toml
Environment: "development"
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```

If you want to actually compile the site for publishing, just run `hugo`. The
output will be in `public`.

[hugo]: https://github.com/spf13/hugo

### Theme ###

This site uses [Geekdoc][hugo-geekdoc] as the theme. Unfortunately it depends
on build-time postprocessing (using Gulp) and so we cannot use it as a
submodule and instead must vendor the release artefacts into this repository.

In order to update to a newer version of Geekdoc, download a [newer release
archive][hugo-geekdoc-releases] and extract it into `themes/hugo-geekdoc/`.

```console
% rm -r themes/hugo-geekdoc/
% curl -L https://github.com/thegeeklab/hugo-geekdoc/releases/latest/download/hugo-geekdoc.tar.gz | tar -xz -C themes/hugo-geekdoc/ --strip-components=1
```

See also https://geekdocs.de/usage/getting-started/.

[hugo-geekdoc]: https://github.com/thegeeklab/hugo-geekdoc
[hugo-geekdoc-releases]: https://github.com/thegeeklab/hugo-geekdoc/releases

### License ###

The text contents of the website are licensed under the terms of the [CC-BY-SA
4.0][cc] license. Any scripts used to build or otherwise generate this website
are licensed under the [Apache 2.0 license][apache2]. They are copyrighted by
their respective authors.

The [`hugo-geekdoc` theme][hugo-geekdoc] is licensed under the terms of the
[MIT][mit-hugo-geekdoc] license.

[apache2]: http://www.apache.org/licenses/LICENSE-2.0
[cc]: https://creativecommons.org/licenses/by-sa/4.0
[mit-hugo-geekdoc]: https://github.com/thegeeklab/hugo-geekdoc/blob/master/LICENSE
