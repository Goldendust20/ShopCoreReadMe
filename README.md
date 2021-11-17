# Shopcore

# Summary
This website was primarily built using Wordpress. It integrates Part (main function in
project), Part (main function in project), and Part (main function in project).
Click the following links for documentation: [Part](documentationLink.com),
[Part](documentationLink.com), and [Part](documentationLink.com).

## Internal Expert
:sparkles: Mike Le-Reiver

## URLs

* Staging: https://shopcore.soliddigital.com

Pushing to develop deploys to staging.

## Email

This site uses WP Mail SMTP plugin integrated with SendGrid for sending emails.

## Testing
[Shopcore Test Plan](https://docs.google.com/spreadsheets/d/1Zfleh_0MDfbbWyIP2HPz10Fgk_1QlVAbN-C2JBod_uQ/edit#gid=0)


## Branching Strategy

* [Solid's Gitflow](https://docs.google.com/document/d/1EW5v-iI6q5QnmA38_z3W6xtS7X9UCKoGuOsNVkeY_CY/edit)
* [Git flow](http://nvie.com/posts/a-successful-git-branching-model/)

* main is on production
* develop is on staging
    * feature/X comes off of develop

## Code Review Guidelines
New code is accepted into the main branch using GitLab's merge request feature. After a branch is complete, use GitLab
to create a merge request for the branch. See [here](https://docs.gitlab.com/ee/gitlab-basics/add-merge-request.html) for instructions.

Make sure to assign the merge request to someone on the same project or, if you're the only one on the project, try to find an assignee with
the same skill set the project requires.

Here are some high level code guidelines for reviewing a merge request:

- Code syntax should not be part of a code review. That will be taken care of by the linter (WIP for this repo)
- Code is broken out to small, reusable parts.
- Unnecessary code duplication is eliminated (balance effort / reward)
- All reusable settings (api credentials/developer keys/db connection credentials) are stored in a central location.
 If they're environment dependent they should be stored in a configuration file specific to that environment (ie staging or production)
 The config files live @ `httpdocs/config.<env>.php`
- Variable names should be `camelCased` (instead of `snake_cased`)
- Model names should be `UpperCamelCase`
- All `try` statements include a `catch` that correctly and meaningfully handles the error.

## Local Development Environment

1. Install Ansible - http://docs.ansible.com/ansible/latest/intro_installation.html#latest-releases-on-mac-osx
    1. Make sure you are current - pre 2.3 will cause issues - `sudo pip install ansible -U`
1. Install Virtual Box - https://www.virtualbox.org/wiki/Downloads
1. Install the Virtual Box Extensdion Pack - https://www.virtualbox.org/wiki/Downloads
1. Install Vagrant - https://www.vagrantup.com/downloads.html
1. Install `mkcert`
    1. `brew install mkcert`
    1. `mkcert -install`

Local IP:

```
192.168.13.226  www.shopcore.test
```

To get started, run the following commands:

```
bin/mkcertDev
bin/addToHostsFile
npm i
vagrant up
bin/downloadStagingData
bin/downloadStagingUploads
bin/open
```

And now open https://www.www.shopcore.test

Note that if your password contains a `$` or `&`, you will have to escape it in `bin/syncWithStagingServer`. e.g:
`ssh ubuntu@wordpress.soliddigital.com mysqldump -usolid_landing --password='\$blahblahblah\$' solid_landing > staging.sql

### Modifying the Child Themes CSS
The child theme in use is called `elementor-child-theme`. To make modifications to the CSS please follow these steps

1. The CSS will live in the child theme's `style.css` file located in `/httpdocs/wp-content/themes/elementor-child-theme/style.css`
1. This can be updated one of two ways
    1. Via your favorite code editor and pushing changes to repo
    1. Using GitLab GUI followingthe steps outlined here [Solid Docs - How To Add CSS Using GitLab GUI ](https://docs.soliddigital.com/docs/css-best-practices/#1-toc-title)

### Using a MySQL GUI with Vagrant

Using a GUI with Vagrant, is just like using a GUI with any remote server. Use the IP and SSH details to connect.

Vagrant boxes all have a username of `vagrant` and a password of `vagrant`.

For example to connect to this box with Sequal Pro you would click the SSH tab and enter the following:

* MySQL Host: `127.0.0.1`
* Username: `shopcore_user`
* Password: `1q2w3e4r` - The password used for all local dev DBs
* Database: `shopcore`
* SSH Host: `192.168.13.226`
* SSH User: `vagrant`
* SSH Password: `vagrant`

## Rollback

The quickest way to rollback after an automated deploy is to ssh onto the server, cd to the project root and reset git to the previous commit:

```sh
ssh production-shopcore
cd /var/www/vhosts/www.shopcore.com/current
git reset --hard HEAD~1
```

Depending on the nature of what you're trying to roll back, you can also use a DB backup to reset the data.

This project uses the Updraft Plus plugin to create and manage backups to the site and database.

To rollback via data from a backup, navigate to Settings -> UpdraftPlus Backups in the wordpress admin menu, and go to the "Existing Backups" tab. Select the backup data from the relevant date that you want to restore, then click "Restore" to the right of it. You can also download a copy of the backed up database from the relevant date if only this is needed.

Backups generally are only kept for 90 days.

## WP Updates

* Follow the standard wp updates process.
* Then update the third party pages described below if needed (usually major/minor elementor update, or changes to header/footer.)

## Third-Party Integrations

### iCIMS API

* We use the icims api to sync the shopcore jobs to our database on a daily basis using the wp cron.
* We then display these job posts using normal elementor listing grids, etc. and link to the icims details page.
* The staging and prod server ips are whitelisted on icims side, so if the server changes, send the new ip to icims to whitelist.

### Property Capsule

[Property Capsule](https://www.propertycapsule.com/) - A SaaS real estate technology platform.

#### Overview

* Integration with this platform relies on supplying Property Capsule with head, navigation, and footer files, in which they scrape from a URL we provide to them
* The files in `/nav_footer/` are PHP, however, they can be accessed as `.html` files
* There are several Property Capsule form pages that have unique content which use thier own head/nav and footer files
* The Property Capsule files can be found here `httpdocs/nav_footer/`
* The head and footer files includes links to the direct paths of Elementor CSS and JS files
  * **NOTE:** the direct paths will need to be updated using `src='https://<?php echo $domain; ?>/wp-content/...` to handle linking to the proper environment, utilizing `domain.php`
* CSS for these pages and content included within Property Capsule can be found here `httpdocs/nav_footer/application.css`
* JavaScript for these pages and content included within Property Capsule can be found here `httpdocs/nav_footer/application.js`
* There is a Property Search dropdown form in the navigation which interacts with the Property Capsule pages. JS file for this form can be seen here `httpdocs/wp-content/themes/elementor-child-theme/js/property-search.js`
* Due to JavaScript conflicts with Elementor and Property Capsule, JS click handlers for the navigation functionality (Property Search dropdown form & mobile menu) have been handled in `httpdocs/nav_footer/clickhandlers.php`
* If any updates are made, [Saving the files in the Property Capsule Dashboard must me done](https://git.soliddigital.com/the-blackstone-group/www.shopcore.com#how-to-save-to-property-capsule)

#### When To Update

If pages are added to the navigation, `/nav_footer/nav.php` will need to be updated

If footer content or dependencies are updated, the following will need to be updated
* `/nav_footer/nav.php`
* `/nav_footer/footer.php`
* `/nav_footer/contact_us_form_footer.php`
* `/nav_footer/tennant_form_footer.php`

If CSS or JS needs to be added or modified, the following will need to be updated
* `/nav_footer/application.css`
* `/nav_footer/application.js`

#### How To Update Files

`nav.php`
1. Login to admin
2. Deactivate Autoptimize
3. Logout or open incognito/private browser window
4. View Page Source
5. Copy from top - `<!doctype html>` to `<div id="solid-wrapper-div">`
6. Paste into `nav.php` at top of file, right below `<?php require_once "domain.php"; ?>`
7. Add `<link rel="stylesheet" media="all" href="https://<?php echo $domain; ?>/nav_footer/application.css" data-turbolinks-track="true" />` as the **first** stylesheet link in `<head>`
8. Add the follwing links to JS files as the **first** links to JS files in `<head>` in same order
```js
<script type='text/javascript' src='https://properties.shopcore.com/property/3rd_party/underscore/underscore.js'></script>
<script type='text/javascript' src='https://properties.shopcore.com/property/3rd_party/swfobject/swfobject.js'></script>
<script src="https://<?php echo $domain; ?>/nav_footer/application.js" data-turbolinks-track="true"></script>
<script type='text/javascript' src='https://<?php echo $domain; ?>/wp-content/themes/elementor-child-theme/js/property-search.js?ver=1.0.1-build.5' id='property-search-js'></script>
```
9. Login to admin
10. Activate Autoptimize

`footer.php`
1. Follow steps 1-4 above for `nav.php`
2. Copy from `<div data-elementor-type="footer" ... ` to the closing `</html>` tag
3. Paste into `footer.php` at top of file, right below
```php
<?php
require_once "domain.php";
?>
</div>
```
4. Require `clickhandlers.php` just above the closing `</body>` tag at bottom of file like so
```php
</script>
<?php require('clickhandlers.php'); ?>
</body>
</html>
```
5. Follow steps 9-10 above for `nav.php`

`contact_us_form_footer.php`
1. Follow steps 1-4 above for `nav.php`
2. Copy from `<div data-elementor-type="footer" ... ` to the closing `</html>` tag
3. Paste into `footer.php` at around line 42, right below
```php
        </section>
      </main>
    </div>
```
4. Follow Step 4 above for `footer.php`
5. Follow steps 9-10 above for `nav.php`

`tennant_form_footer.php`
1. Follow steps 1-4 above for `nav.php`
2. Copy from `<div data-elementor-type="footer" ... ` to the closing `</html>` tag
3. Paste into `footer.php` at top of file, right below the following, replacing `<div data-elementor-type="footer" ... ` on down
```php
<?php
require_once "domain.php";
?>
            </div>
          </div>
        </div>
      </main>
    </div>
```
4. Follow Step 4 above for `footer.php`
5. Follow steps 9-10 above for `nav.php`

#### How To Save to Property Capsule

There is a sandbox portal for staging and a production portal. Credentials for both can be found in vault under Shopcore.
Testing in the sandbox can be difficult to troubleshoot errors and the form pages do not work.
Only test in sandbox if neccessary.

Staging (credentials in Vault)
- https://exceltrust.staging.propertycapsule.com/web/property/search?view=search
- https://exceltrust.staging.propertycapsule.com/property/admin/host/main2/host_id:1292/

Production - https://propertycapsule.com/property/admin/login/form
Use `michael.lereiver@soliddigital.com` to sign in

1. Go to [https://exceltrust.propertycapsule.com/property/admin/host/main2/host_id:1292/](https://exceltrust.propertycapsule.com/property/admin/host/main2/host_id:1292/)
and click Save button

2. Go to [https://exceltrust.propertycapsule.com/property/admin/host/main2/host_id:1356/](https://exceltrust.propertycapsule.com/property/admin/host/main2/host_id:1356/) and click Save button

Sandbox - https://staging.propertycapsule.com/property/admin/login/form
Use `matt+shopcore@propertycapsule.com` to sign in

1. Go to [https://exceltrust.staging.propertycapsule.com/property/admin/host/main2/host_id:1292/](https://exceltrust.staging.propertycapsule.com/property/admin/host/main2/host_id:1292/)
and click Save button

### iCIMS

[iCIMS](https://www.icims.com/) - a SaaS job recruiting platform

#### Overview

* Integration with this platform relies on supplying iCIMS a URL we provide to them to scrape
* The files can be found in `httpdocs/icims-template/` for production and `httpdocs/icims-template_staging/`
* The head and footer files includes links to the direct paths of Elementor CSS and JS files
  * **NOTE:** each directory has corresponding direct paths for CSS and JS files
* `footer.html` includes a video just above `<div data-elementor-type="footer" data-elementor-id="137" class="elementor elementor-137 elementor-location-footer" data-elementor-settings="[]"><div class="elementor-section-wrap">`

#### When To Update

If pages are added to the navigation, `nav.html` will need to be updated

If footer content or dependencies are updated, the following will need to be updated
* `nav.html`
* `footer.html`

#### How To Update Files
pro tips before you start:
- Before you start copying page source, make sure Autoptimize is deactivated.
- Use an incognito window to copy page source.

`nav.html`
1. View Page Source
2. Copy from top - `<!doctype html>` to just above `<div data-elementor-type="footer" data-elementor-id="137" class="elementor elementor-137 elementor-location-footer" data-elementor-settings="[]"><div class="elementor-section-wrap">`
3. Replace all content in `nav.html` with what you just copied.
4. Remove the popup contact form script. it starts immediately after the `End Signal Tag Script` line, and is about \~118 lines long (currently).
5. Add a link to the theme stylesheet somethere around line 112.

Theme stylesheet link tag:
`<link rel='stylesheet' id='shopcore-style-css' href='https://<?php echo $domain; ?>/wp-content/themes/elementor-child-theme/style.css?ver=1.0.6-build.2' type='text/css' media='all' />`


`footer.php`
1. Follow steps 1-4 above for `nav.html`
2. Copy from `<div data-elementor-type="footer" ... ` to the closing `</html>` tag
3. replace the contents of `footer.html` with what you have copied.

Now, that `footer.html` and `nav.html` are updated, one last thing: use your text editor's find/replace functionality to get the URLs in those files right. So if you copied your markup from staging, you would do something like this:
**find:** `shopcore.soliddigital.com`
**replace:** `<?php echo $domain; ?>`


#### If Updated
* iCIMS no longer has to be updated when the header changes, since the iCIMS header is simplified
* The iCIMS footer shouldn't change, but if it does, it'd have to be rescrapted
* The PM should be aware of who to contact
