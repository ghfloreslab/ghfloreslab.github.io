# Personal Portfolio webpage

This website was designed using the template from `github.com:alshedivat/al-folio`. 
Originally, **al-folio** was based on the [\*folio theme](https://github.com/bogoli/-folio) (published by [Lia Bogoev](https://liabogoev.com) and under the MIT license).


## Installation

### Requirements
- [Ruby](https://www.ruby-lang.org/en/downloads/)
- [Bundler](https://bundler.io/)
- [rbenv](https://github.com/rbenv/rbenv)

### Commands to Execute
```bash
$ git clone git@github.com:<your-username>/<your-repo-name>.git
$ cd <your-repo-name>
$ bundle install
$ bundle exec jekyll serve --lsi
```

**Commit** changes before deployment

### Deployment

Deploying your website to [GitHub Pages](https://pages.github.com/) is the most popular option. You need to do the following:

1. Rename your repository to `<your-github-username>.github.io` or `<your-github-orgname>.github.io`.
2. In `_config.yml`, set `url` to `https://<your-github-username>.github.io` and leave `baseurl` empty.
3. Set up automatic deployment of your webpage
4. Make changes, commit, and push
5. After deployment, the webpage will become available at `<your-github-username>.github.io`.

If you decide to not use GitHub Pages and host your page elsewhere, simply run the following command which will generate the static webpage in `_site` directory. Then copy the contents in this folder to your hosting server
```bash
$ bundle exec jekyll build --lsi
```

**Note:** Make sure to correctly set the `url` and `baseurl` fields in `_config.yml` before building the webpage. If you are deploying your webpage to `your-domain.com/your-project/`, you must set `url: your-domain.com` and `baseurl: /your-project/`. If you are deploing directly to `your-domain.com`, leave `baseurl` blank.

## Features

### Publications
- The publications page is generated automatically from your BibTex bibliography. Simply edit `_bibliography/papers.bib`. You can also add new `*.bib` files and customize the look of your publications however you like by editing `_pages/publications.md`.

In publications, the author entry for yourself is identified by string array `scholar:last_name` and string array `scholar:first_name` in `_config.yml`:
```
scholar:
  last_name: [Einstein]
  first_name: [Albert, A.]
```
If the entry matches one form of the last names and the first names, it will be underlined. You can enter multiple version of your name so it is highlighted according how is formatted in your publications or patents

There are several custom bibtex keywords that you can use to affect how the entries are displayed on the webpage:

- `abbr`: Adds an abbreviation to the left of the entry. You can add links to these by creating a venue.yaml-file in the _data folder and adding entries that match.
- `abstract`: Adds an "Abs" button that expands a hidden text field when clicked to show the abstract text
- `arxiv`: Adds a link to the Arxiv website (Note: only add the arxiv identifier here - the link is generated automatically)
- `bibtex_show`: Adds a "Bib" button that expands a hidden text field with the full bibliography entry
- `html`: Inserts a "HTML" button redirecting to the user-specified link
- `pdf`: Adds a "PDF" button redirecting to a specified file (if a full link is not specified, the file will be assumed to be placed in the /assets/pdf/ directory)
- `supp`: Adds a "Supp" button to a specified file (if a full link is not specified, the file will be assumed to be placed in the /assets/pdf/ directory)
- `blog`: Adds a "Blog" button redirecting to the specified link
- `code`: Adds a "Code" button redirecting to the specified link
- `poster`: Adds a "Poster" button redirecting to a specified file (if a full link is not specified, the file will be assumed to be placed in the /assets/pdf/ directory)
- `slides`: Adds a "Slides" button redirecting to a specified file (if a full link is not specified, the file will be assumed to be placed in the /assets/pdf/ directory)
- `website`: Adds a "Website" button redirecting to the specified link
- `altmetric`: Adds an [Altmetric](https://www.altmetric.com/) badge (Note: if DOI is provided just use `true`, otherwise only add the altmetric identifier here - the link is generated automatically)
- `dimensions`: Adds an [Dimensions](https://www.dimensions.ai/) badge (Note: if DOI or PMID is provided just use `true`, otherwise only add the dimensions identifier here - the link is generated automatically)

You can implement your own buttons by editing the bib.html file.

## math & code

- Fast math typesetting is through [MathJax](https://www.mathjax.org/)
- Code syntax highlighting using [GitHub style](https://github.com/jwarby/jekyll-pygments-themes)

## Updates Log

Changed to Boostrap 5.x. It required some minor modifications to make the About page look similar to the original using Bootstrap 4.x. 
The changes are as follows:

  1. In layouts/about.html: Modified this line with new syntax so image aligns right
        <div class="profile float-{%- if page.profile.align == 'left' -%}left{%- else -%}right{%- endif -%}">
        changed to
        <div class="profile float-{%- if page.profile.align == 'left' -%}start{%- else -%}end{%- endif -%}">

  2. In _includes/header.html, added keyword flex-row-reverse so that the entire navigation bar is aligned right
        <div class="collapse navbar-collapse flex-row-reverse" id="navbarNav">

  3. Bootstrap accordion background color does not change when theme changes light/dark. To fix this, I added CSS code in _sass/_base.scss
    so that the accordion element gets the background color specified by the sass variable --global-bg-color

        .accordion-item {
          background-color: var(--global-bg-color);
        }



## License

The theme is available as open source under the terms of the [MIT License](https://github.com/alshedivat/al-folio/blob/master/LICENSE).

