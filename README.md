# Example repo for HSPH Self-Hosted Websites

This repository is an example of how to create a Self-Hosted Website on Pantheon, set up according to the guidelines in the [HSPH Self-Hosted Websites documentation](#).

## Repository structure

This repository is a monorepo, containing the WordPress core files, as well as all plugin and theme files. 

The structure is inherited from the upstream [Self-Hosted Upstream](https://github.com/HarvardChanSchool/self-hosted-upstream) repository. When creating your Self-Hosted Website repository, be sure to populate it using the Self-Hosted Upstream as a remote:

```bash
git remote add upstream https://github.com/HarvardChanSchool/self-hosted-upstream
git fetch upstream
git merge upstream/build
```

The Self-Hosted Upstream repo includes WordPress core files, as well as a set of plugins and themes that are approved and recommended for use on Self-Hosted Websites. To receive updates for any of these items, merge from the `build` branch of the upstream repository.

## Local development and a Git-based deployment workflow

All code, including WordPress core files, third-party plugins and themes, and plugins and themes build specifically for your Self-Hosted Website, must be hosted in your Git repository. This allows for a Git-based deployment workflow, where pushing to development branches triggers a deployment to the Pantheon development environment, and tagging a release triggers a deployment to the Pantheon live environment. For more on this workflow, see the [HSPH Self-Hosted Websites documentation](#).

Because of the need to host all code in the repository, all maintenance and feature development must take place in a local development environment. It is not possible to update themes and plugins directly on the Pantheon development or live environments.

## Managing plugins and themes

As in a typical WordPress installation, plugins and themes are stored in the `wp-content/plugins` and `wp-content/themes` directories, respectively. For the purposes of managing your Self-Hosted Website, there are several different workflows for installing and updating plugins and themes, depending on where those tools are sourced from:

### Plugins and themes from the WordPress.org repository

For plugins and themes that are available in the WordPress.org repository, you can install and update them as you would in a typical WordPress installation. This typically means either (a) installing from the WordPress Dashboard (Dashboard > Plugins), or, ideally, (b) using WP-CLI. Then, you will use `git add` and `git commit` to track the tools in the repo.

By default, all subdirectories of `wp-content/themes/` and `wp-content/plugins` are ignored in the `.gitignore` file. When adding a new plugin, be sure to add a line to the `.gitignore` file overriding this global ignore rule.

Here's a complete example of how to add a new plugin from the WordPress.org repository. For this example, we'll use the popular [Yoast Duplicate Post](https://wordpress.org/plugins/duplicate-post/) plugin:

First, install the plugin using WP-CLI:

```bash
wp plugin install duplicate-post
```

Next, tell `.gitignore` that you would like to track this plugin:

`.gitignore`
```
# Single-site-specific plugins and themes #
###########################################
!/wp-content/plugins/duplicate-post/
```

```bash
git add .gitignore
git commit -m "Don't ignore Yoast Duplicate Post plugin in .gitignore"
```

Finally, add the plugin to the repository, commit, and push to GitHub:

```bash
git add wp-content/plugins/duplicate-post
git commit -m "Add Yoast Duplicate Post plugin"
git push origin main
```

### Custom plugins and themes

Tools that are built specifically for your Self-Hosted Website, such as a custom theme, are handled in a similar fashion. They will be tracked in the appropriate `wp-content` subdirectory, and tracked as part of the monorepo. So, for a theme called `my-custom-theme`:

First, tell `.gitignore` that you would like to track this theme:

`.gitignore`
```
# Single-site-specific plugins and themes #
###########################################
!/wp-content/plugins/duplicate-post/
!/wp-content/themes/my-custom-theme/
```

```bash
git add .gitignore
git commit -m "Don't ignore my-custom-theme in .gitignore"
```

Next, add the theme to the repository, commit, and push to GitHub:

```bash
git add wp-content/themes/my-custom-theme
git commit -m "Add my custom theme"
git push origin main
```

### Plugins and themes loaded from private GitHub repositories

In certain cases, you may need to load a plugin or theme that is not hosted on wordpress.org, but instead is hosted in a private GitHub repository. This will happen most frequently with tools developed by HSPH. In this case, the tools will be defined as Composer dependencies. Unlike publicly-available tools, these plugins and themes are *not* stored directly in the repository, and they should continue to be ignored by `.gitignore`. Instead, they will be added to the composer.json manifest. For local development, use `composer install` to install the tools. For deployment, the custom GitHub Actions found in this repository will handle the necessary build steps.

As an example, we will add the legacy `plugin-widget-visibility` plugin build by HSPH. First, add the plugin as a Composer dependency:

`composer.json`
```
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/HarvardChanSchool/plugin-widget-visibility"
        }
    ],
```

Then, tell Composer to require the plugin. For our example, we'll require the `dev-main` branch:

```bash
composer require harvardchanschool/plugin-widget-visibility:dev-main
```

Finally, commit the changes to `composer.json` and `composer.lock`, and push to GitHub:

```bash
git add composer.json composer.lock
git commit -m "Add plugin-widget-visibility as a Composer dependency"
git push origin main
```
