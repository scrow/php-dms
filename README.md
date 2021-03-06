# PHP Dead Man's Script (PHP-DMS) #

version 1.1  
Released 04-Jan-2014  
<http://scrow.sdf.org/php-dms/>  
<http://github.com/scrow/php-dms/>

---

## Description ##

PHP-DMS is a free (as in beer) Dead Man's Switch primarily targeted for use in a *nix environment.  It requires a minimal Apache and PHP installation, along with basic e-mail sending capabilities, and is designed to be called as a daily cron job.

PHP-DMS sends a "check-in" e-mail to a configured e-mail address at a specified interval, with a link to click to reset a counter.  The counter is incremented by 1 with each call of the script (which should mean once daily).  When clicked, the counter resets to 0.

Once a specified number of days have passed without a check-in, the check-in e-mails are suppressed and the system begins sending messages to designated recipients.

E-mails are in plain text format.  No special databases are required.

---

## License ##

See the separate `LICENSE` file for important license and warranty information.

---

## Installation ##

To install PHP-DMS, unzip the distribution archive to a folder on an Apache web server and run the `initialize.sh`.  Ensure the `data/` subdirectory exists and has the appropriate permissions to be readable by your web server.

The latest bleeding-edge code is available via Git:

    git clone git://github.com/scrow/php-dms
    
Configuration instructions are provided below.

---

## Configuration ##

All configuration variables reside in the `globals.php` file.  This is the only file most users will need to edit.  To start, copy or rename the `globals.php.SAMPLE` file to `globals.php` and edit the following configuration values:

`baseFolder`:     The full path to the installation directory.

`dataFile`:       The full path and filename of the `daynum.dat` file.

`footerFile`:     The full path and filename of the `footer.txt` file.

`tokenFile`:      The full path and filename of the `token.dat` file.

`checkInterval`:  The interval at which check-in e-mails should be sent.  Most users will want to set this at around 5 to 7 days.

`sendAfter`:      The number of days to wait for check-in before releasing messages to recipients.  Can be anything greater than `checkInterval`, but ideally should be 3 to 4 times the `checkInterval` value to avoid unintentional misfires.
                  
`webPath`:        The full web address that corresponds to the location where PHP-DMS is installed.

`ownerMail`:      The e-mail address to which check-in messages should go.

`mailFrom`:       The `From:` address for all PHP-DMS e-mails.

`subjectPrefix`:  The prefix for the `Subject:` line of all messages.

---

## Message Folder Structure ##

Messages are organized into folders by recipient e-mail address, and then by sending day.  The sending day is the number of days beyond the sendAfter value in `globals.php`.

A file called `data/scrow@sdf.org/0/message.txt` will be sent to scrow@sdf.org on the `sendAfter` date.

A file called `data/scrow@sdf.org/7/message.txt` will be sent to scrow@sdf.org 7 days after the `sendAfter` date.

Each numbered folder can contain an unlimited number of text files.  It is not necessary to include a `.txt` extension.  The file name will be used as the subject line of the e-mail, so it is a good idea to give the file a descriptive name.

Numbered folders should be entirely numeric -- no special characters, no letters.  Leading zeroes are okay.  PHP-DMS looks at the base 10 integer value equivalent of the folder name.

---

## Calling the Script ##

The script should be called from cron.  A possible crontab entry would be:

    @daily /usr/bin/php -f /path/to/php-dms/run.php

If calling from a remote system, possible syntax might be:

    @daily /usr/bin/wget -q -O /dev/null \
        http://hostname.domainname/path/to/php-dms/run.php > /dev/null 2>&1

It is important to note that PHP-DMS is not date-aware.  In other words, it works entirely off the counter file `daynum.dat` and not the system clock.  Thus, the run.php script should be called exactly once per day, every day.

---

## Testing PHP-DMS ##

You can test PHP-DMS by creating a recipient folder under `data/` using your own e-mail address and manually calling the PHP-DMS `run.php` script using the PHP CLI or Apache.  When testing is complete, remember to delete your test folder, and reset both the `daynum.dat` and `token.dat` files to their initial values of `0` by running the `initialize.sh` script.

---

## Security Considerations ##

A Dead Man's Switch may be used to send sensitive information, such as login credentials, bank account numbers, insurance policy details, etc.  It is important to ensure this type of sensitive data is as secure as possible on the server hosting PHP-DMS.  File permissions are of particular importance.  The web server needs to be able to read the files, but pay attention to group and world permissions.

There is an `.htaccess` file in the `data/` directory which is intended to restrict remote access to the directory contents.  Be sure to rename this file if your Apache installation is configured to look for a different file name.  The presence of an `.htaccess` file does **not** restrict access to the `data/` directory by other scripts running on the Apache server, so all other scripts on the server must also be secure.

It may be smart to encrypt the message contents as an ASCII-armored GnuPG file, using a command such as:

    /usr/bin/gpg -r recipient@domain.com --armor --encrypt message.txt

It may also be desirable to secure the `checkin.php` file using `.htaccess` restrictions which require authentication against an `.htpasswd` file.  A sample `.htaccess` file is included in the distribution as `htaccess-sample` - simply rename it to `.htaccess` to use this one.  Then, create a `.htpassword` file with the username and password to use when checking in to PHP-DMS:

    /usr/bin/htpasswd -c .htpasswd username

If required by your server, update the .htaccess file to provide the full absolute path to the `AuthUserFile`.  Some Apache installations require this, and will throw an Error 500 Internal Server Error if it is not provided.  

---

## Other Considerations ##

Obviously system uptime is of paramount importance when using PHP-DMS.  It is recommended that this script be installed on a high-availability system.  For example, the [SDF Public Access Unix System][1].  One of the first messages send out might be instructions on how to access and renew membership or pay hosting dues for the server hosting your PHP-DMS installation, to ensure it doesn't go dark before all messages are sent out.

---

## Feedback and Suggestions ##

Feedback and suggestions are welcome via e-mail to <scrow@sdf.org>.

[1]: http://www.sdf.org/      "SDF Public Access Unix System"
