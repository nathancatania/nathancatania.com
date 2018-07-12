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
