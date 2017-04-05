---
layout: page
title: "Upgrading very old installation"
category: install
date: 2017-03-23 00:00:00
order: 4
---

## Upgrading from pre 1.2.0 versions

Several files have changed their name and location in MailWatch 1.2.0, particularly the PHP files that are launched in cron and the Init script files.

Please read the documentation carefully to remove old files during the upgrade.

### Upgrading from 1.0.3 to 1.2.0

follow [1.2.0 upgrading]({% post_url 2017-03-23-upgrading %}) instructions

### Upgrading from 1.0.x to 1.0.3

- Add grant for GeoIP updates: as mysql 'root' users run the following SQL:
  
  ```sql
    GRANT FILE ON *.* TO mailwatch@localhost IDENTIFIED BY '<password>';
    FLUSH PRIVILEGES;
  ```
- Run update.php to update schema

- Move /var/www/html/mailscanner to /var/www/html/mailscanner.old and move the mailscanner directory from the tarball to /var/www/html

- Copy conf.php.example to conf.php and edit the file to suit.

- Replace MailWatch.pm with new version and set database connection settings, restart MailScanner for this to take effect.

### Upgrading from 0.5.1 to 1.0

- Create conf.php
   In the mailscanner directory - copy conf.php.example to conf.php, and 
   edit it to suit.

   Note that MailWatch 1.0 can use the quarantine more effectively when used
   with MailScanner version 4.43 or later as Julian added some code for me
   to keep track of messages quarantined by using a flag in the maillog table.
   This means that MailWatch 1.0 is *much* faster when you have a large 
   quarantine directory.  The new quarantine report requires the use of the
   new functionality - so you must upgrade if you want to run this.
  
   The new quarantine flag is not used by default - if you have MailScanner
   verions 4.43 or later, you can activate the new functionality by setting
   QUARANTINE_USE_FLAG to true in conf.php - if you do this, you must disable
   the clean.quarantine script supplied by MailScanner and use the new 
   quarantine_maint.php script in the tools directory instead.

   The once the new flag is enabled - you will need to populate the database
   with a flag for each of the message currently existing in your quarantine.
   You do this by running './quarantine_maint.php --reconcile'.

   To clean the quarantine - set 'QUARANTINE_DAYS_TO_KEEP' in conf.php and 
   run './quarantine_maint.php --clean'.  This should then be run daily from cron.
  
- Stop MailScanner/Apache from using the database while the schema is updated.
   
  ```shell
  $ service httpd stop
  ```

   Either stop MailScanner and start the inbound sendmail to queue the messages
   or change 'Always Looked Up Last = none' in MailScanner.conf and restart.
   
  ```shell
  $ service MailScanner stop
  $ service MailScanner start
  ``` 

- Run the upgrade script to update the database schema.

  ```shell
  $ ./upgrade.php        # IMPORTANT - do not run this more than once!!!
  $ rm upgrade.php
  ```

- Install the new MailWatch files
  
  ```shell
  $ mv /var/www/html/mailscanner /var/www/html/mailscanner.old
  $ mv mailscanner /var/www/html/mailscanner
  ```
  
   Check the permissions of images/cache - they should be ug+rwx and owned by
   the same group as your web server run as.

- Install new MailWatch.pm
   Edit MailWatch.pm and change the database connection string to 
   match your existing.
   
   Move MailWatch.pm to /usr/share/MailScanner/MailScanner/CustomFunctions
   
   Remove the old MailWatch.pm if it was installed in 
   /usr/share/MailScanner/MailScanner and remove the
   'require MailScanner/MailWatch.pm' from CustomConfig.pm if necessary.

- Install SQLBlackWhiteList.pm if required
   If you wish to use the new integrated Blacklist/Whitelist functionality, 
   then edit the file and change the connection string in the CreateList
   subroutine to match MailWatch.pm. 

   Copy SQLBlackWhiteList.pm to /usr/share/MailScanner/MailScanner/CustomFunctions
   and in MailScanner.conf set:
  
  ```
  Is Definitely Not Spam = &SQLWhitelist
  Is Definitely Spam = &SQLBlacklist
  ```


- Restart MailScanner and check /var/log/maillog for errors.
   # service MailScanner start
   
- Test the new web interface.

  Run the SpamAssassin Rules Update and the MCP Rules Update (is you use MCP) 
  and run the GeoIP database update.

- If used, replace sendmail_relay.php and mailq.php with the latest versions.

If you encounter any problems with the procedure, then please contact us
by the mailing-list at Sourceforge.