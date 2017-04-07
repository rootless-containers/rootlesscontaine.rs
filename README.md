## `rootlesscontaine.rs` ##

[`rootlesscontaine.rs`][rc] is a single-purpose website for tracking the
progress of rootless container support in various projects. I'm not good at
this whole "web development" gig, so I apologise for everything.

[rc]: https://rootlesscontaine.rs/

### Usage ###

This site is "built" using [Hugo][hugo], so just run the following:

```
% hugo server --source=site
Started building sites ...
Built site for language en:
0 draft content
0 future content
0 expired content
0 regular pages created
1 other pages created
0 non-page files copied
0 paginator pages created
0 tags created
0 categories created
total in 22 ms
Watching for changes in /<path>/rootlesscontaine.rs/site/{data,content,layouts,static,themes}
Serving pages from memory
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```

If you want to actually compile the site for publishing, just run `hugo
--source=site`. The output will be in `docs`.

[hugo]: https://github.com/spf13/hugo

### License ###

`rootlesscontaine.rs` (the code) is licensed under the terms of the GNU AGPLv3
(or later).

```
Copyright (C) 2017 Aleksa Sarai <cyphar@cyphar.com>

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU Affero General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU Affero General Public License for more details.

You should have received a copy of the GNU Affero General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.
```

The contents of the website are licensed under the terms of the [CC-BY-SA
4.0][cc] license.

[cc]: https://creativecommons.org/licenses/by-sa/4.0
