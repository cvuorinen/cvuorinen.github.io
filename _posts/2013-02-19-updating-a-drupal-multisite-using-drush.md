---
title: Updating a Drupal multisite using Drush
categories:
  - Web Development
tags:
  - Drupal
  - Drush
---

Recently I had to update a Drupal multisite installation with 62 sites. The whole process felt a little bit daunting at first, but I managed to pull it off quite easily after all thanks to good preparation and research beforehand. So I decided to document my experience for later reference (to myself and others). Please share in the comments if you find it useful.

<!--more-->

## Drush is your friend

I had used Drush previously so I knew it can be a real time saver. For anyone reading this who is not familiar with [Drush](http://drush.ws/), it is a command line tool used to manage a Drupal installation and run all kinds of Drupal commands. Such as: run cron, clear caches, make backups, install modules etc. If you do Drupal development and are not using Drush, you really should. It saves you time and makes working with Drupal more enjoyable by not having to click around the Drupal admin interface that much. Drush is your friend.

In a multisite installation you have to tell Drush which site you want to run the command on (or if you don't, by default the command will be run on the default site). Luckily there is also a **@sites** keyword that executes the command on all sites of a multisite installation. It will list all the sites and ask for a confirmation before executing, so you always know when you are running a command on multiple sites.

## The Drupal update process

According to [Drupal Core update instructions](http://drupal.org/taxonomy/term/34882) the update process involves more or less the following steps:

1. Make a backup
2. Put site in maintenance mode
3. Update core files
4. Run update script to update database
5. Disable maintenance mode

Also, it is always best to do a trial run of the complete update process on a test server before updating production environment. There are some Drush commands that can be used to sync a Drupal database to a remote server that help you in managing a mirrored testing server, but those are out of the scope of this post.

I will now describe the Drush commands I used to accomplish the above steps and update my multisite Drupal. Please note that the commands listed here are for Drupal 7 and Drush 5, so the commands may vary for different versions. Most Drush commands also have a shorthand version but I will be using the full commands in this post to better illustrate what each command is doing. Feel free to explore the Drush help/documentation to learn about the shorthands and all options available for each command.

## 1. Make a backup

To completely backup a Drupal installation, you need to backup the files and the database. Drush has two different commands for backup purposes; **archive-dump** and **sql-dump**. The former backs up everything including files and database as the latter only dumps the database. Since the archive-dump command wraps everything in a single file, including Drupal core and all the files, I did not want to run it on all of the 62 sites (and thus getting 62 tarballs with the same Drupal core code base). So I decided to use the sql-dump command in Drush and create one tar file of the whole Drupal root manually.

Here is the command I used to backup all the databases:

```
drush @sites sql-dump --result-file --gzip
```

This command will dump all the different databases used by the sites into individual files. The *--result-file* switch tells Drush to save the dump into a file (using date based filename under ~/drush-backups directory when no filename is specified) and the *--gzip* switch obviously makes it gzip compressed.

Just for the sake of completeness, here is the command I used to create a tar archive of the Drupal installation:

```
tar -czvf drupal-backup.tar.gz drupal-root/
```

## 2. Put site in maintenance mode

The Drupal maintenance mode (or offline mode as it was called in previous Drupal versions) is saved as a Drupal variable in the database as are any other Drupal options that can be set using the admin interface. To change variable values using Drush we can use the **variable-set** command. You also need to clear caches for the maintenance mode to take effect, so we also need to use the Drush command **cache-clear**. Also make sure you are logged in to the sites (at least one that you can use to test) before putting the site into maintenance mode. As a logged in admin user you can see the site normally instead of only the maintenance mode notification so you can verify everything is working correctly after the update.

Here are the commands used to put all the sites into maintenance mode:

```
drush @sites variable-set maintenance_mode 1
drush @sites cache-clear all
```

## 3. Update core files (and modules)

To update Drupal core and also all installed contrib modules (/sites/all/modules), the **pm-update** command can be used. It downloads the latest version from drupal.org and installs the updated versions. A list of modules to update can also be passed on as argument to this command (and *drupal* can be used as the module name to update only Drupal core). Since I was feeling a little bit adventurous, I decided to update all contrib modules also at the same time.

Because we only have the core and module files in one place in the filesystem even though they are used by all of the sites, we only need to run this command on one site and not all. Thus we use the **default** keyword instead of **@sites** with this command.The pm-update command also runs the database update script, but since we have to run it for all of the sites, we could have also used the **pm-updatecode** command, which is similar to pm-update except it does not run the database update script.

Here is the command to install the updates:

```
drush default pm-update
```

## 4. Run update script to update database

Now we have the updated versions of Drupal core and contrib modules. Drupal has a system for migrating database changes when updating. Normally when you update a module this is run automatically when you visit the modules page in the admin interface. But when using Drush to install the updates, we have to run it manually on all the sites since they all have separate database. This can be accomplished using the **updatedb** command as follows:

```
drush @sites updatedb
```

And now the update is done, time to test everything is working correctly. Now is the time to revert in case something went wrong. And I mean seriously, test EVERYTHING. It's not that uncommon to run into issues, especially when using lots of contrib modules. I will outline some issues we ran into in the last chapter.

## 5. Disable maintenance mode

Ok, so everything seems to be working like a charm (or you have been busy debugging, googling, fixing, pulling out hair and cursing and finally got it working). Time to open the doors to the public again. This is almost identical to step 2, only set the variable value to 0 to disable the maintenance mode. And also remember to clear the cache.

Here are the commands used to disable maintenance mode on all the sites:

```
drush @sites variable-set maintenance_mode 0
drush @sites cache-clear all
```

## All done!

Congratulations, if you have been following this guide to update a real Drupal installation and got this far, you should now have your Drupal multisite installation running a shiny new version of Drupal and also the latest and greatest of contrib modules. Please share any thoughts or issues you ran into in the comments.

## Some issues I encountered when updating

I did not have any issues with Drupal core update, but a few modules caused some problems. Namely Views and ctools.

Our multisite has a custom module where there are views stored in files. These views are used by all of the sites so storing them in files that all of the sites use is convenient so when we update the files all sites get updated without having to click around in the admin interface of every site. Views had changed the way view areas are stored in header and footer so the way they were configured in our files stopped working. I had to reset all the view areas and export the views again and update the files. Nothing too complicated, but it took us a while to figure this out since there were no error messages or anything, the view area configurations were just silently ignored and nothing was showing where they were supposed to be.

The problem with ctools was actually presented by one of our custom modules by spilling out a screen full of error messages. But googling with the error message led me to a post in the ctools issues tracker at drupal.org where some other user had similar error messages and merlinofchaos (ctools creator) had replied that it is a known issue and is already fixed in the dev version. So I updated ctools to the dev version and got no more error messages. Everything seem to have been working fine after that so nothing serious, just a few examples of what kind of issues you can run into when updating contrib modules.
