# csz.academy [![License: AGPL v3](https://img.shields.io/badge/License-AGPL%20v3-blue.svg)](https://www.gnu.org/licenses/agpl-3.0.html) [![License: CC BY-NC 4.0](https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc/4.0/)
> Official CSZ Blog.

This is the repository for [csz.academy](https://csz.academy).

## Licensing

Two licenses apply:

- [AGPL-3.0 or later](./LICENSE.AGPL-3.0-or-later) applies to
  - Code used to generate the webpage
  - Code which is part of the content if no other license is specified within the same post
- [CC-BY-NC-4.0](./LICENSE.CC-BY-NC-4.0) applies to any non-code content if not specified otherwise.

## How to create your author profile

Create a file under the `content/authors` directory with your acronym as filename. Then add a YAML content with your personal information like this:
```yaml
name: <name>
acronym: <acronym>
bio: |
  Your profile description.
social:
  - github: <github username>
web: <your homepage>
```

## Directory Structure

### Posts
Every post is a single unit which contains all markup- and asset files.
At a minimum you create the following structure
```
content/
└── <Year>-<Month>-<Day>-<post-title>/
    └── index.md
```

#### Using Assets

Drop the assets next to your post:
```
<Year>-<Month>-<Day>-<post-title>/
├── archive.zip
├── image.png
└── index.md
```
And you can link them in your post like this:
~~~
![My Image](image.png)
[Download Archive here](archive.zip)
~~~
