# csz.academy
> Official CSZ Blog.

This is the repository for [csz.academy](https://csz.academy).

## Licensing
Three (yes, three) licenses apply:

- [AGPL-3.0 or later](./LICENSE.AGPL-3.0-or-later) applies to code used to generate the blog.
- [MIT](./LICENSE.MIT) applies to all code samples in blog posts that are publicly available at https://csz.academy (if not specified otherwise).
- [CC-BY-NC-4.0](./LICENSE.CC-BY-NC-4.0) applies to any non-code content (if not specified otherwise).

## Writing own Posts

### Prerequisites
You need to install [Hugo](https://gohugo.io).

After installation:
```sh
git clone git@github.com:csz-academy/csz.academy.git
cd csz.academy

# start site builder with live-reload
hugo serve
```

### Creating an Author Profile
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

### Writing a Post

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
