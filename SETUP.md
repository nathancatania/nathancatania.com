# Setup and Configuration Notes

## New Clone
1. Install Homebrew
2. `brew install ruby`
3. `brew install npm`
4. Edit .bash_profile as per the below (this ensures that Homebrew's packages are used):
```
$ nano ~/.bash_profile

# homebrews should always take precedence
export PATH=/usr/local/bin:$PATH
```
5. Navigate to the project/repo directory.
6. `bundle install`
7. `npm run setup`
8. `npm install`

## Run & Test Locally:
`npm run local`

## Deploying to GitHub Pages:
Before you deploy, commit your changes to any working branch __except__ `gh-pages`

Run the following to build and publish the site:
`npm run publish`

As the Chalk theme uses modules not supported by GitHub pages, it must be built first, committed, and then published (which is different to the normal GH Pages process).

## Misc
- The GH Pages source must be set to `gh-pages branch`.
- Ensure A records are added.
- Directory will be nathancatania.github.io/nathancatania.com (redirect www here).


## Fonts
To change fonts:
1. Select a font from Google Fonts.
2. Generate compatible files to serve [here](https://google-webfonts-helper.herokuapp.com).
3. Create a folder for the font in the `_assets` directory and move the files from #2 here. Eg: `_assets/open-sans/`
4. Copy the CSS from #2 into `_assets/stylesheets/fonts.scss`
5. Change `_assets/stylesheets/_variables.scss` to use the new font family.
6. Change the font family (and weights) in `_assets/javascripts/webfonts.js`.

## Page Width
* Change `$max-width: 800px;` at the bottom of `_assets/stylesheets/_variables.scss`.
