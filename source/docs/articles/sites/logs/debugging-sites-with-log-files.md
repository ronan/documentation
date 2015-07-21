---
title: Debugging Sites with Log Files
description: Learn to debug your Pantheon Drupal or WordPress sites using database log files.
category:
  - developing
keywords: debug, debugging sites, debug sites, debugging site, debugging mysql, debug sql, troubleshoot mysql, troubleshoot sql, database logs, db logs, where are db logs stored, where are database logs
---
One of the key ways to find issues on your website is to check your database logs to isolate current or potential problems.

### Drupal

Drupal, by default, logs events using the Database Logging module (dblog). Sometimes, PHP fatal errors can be found in these logs, depending on how much Drupal bootstrapped. These event logs can be accessed in a couple ways:  

1. Visit `/admin/reports/dblog` once logged in as administrator.
2. Using [Terminus](https://github.com/pantheon-systems/cli):  

```bash
terminus drush --site=<site> --env=<env> watchdog-show
```
<div class="alert alert-info" role="alert">
<strong>Note</strong>: Replace <code>&lt;site&gt;</code> with your site name, and <code>&lt;env&gt;</code> with the environment (Dev, Test, or Live). You can see a list of all your sites by running <code>terminus sites list</code></div>

Terminus can invoke Drush commands to "watch" events in real-time; tail can be used to continuously show new watchdog messages until interrupted (Control+C).  

```bash
terminus drush --site=<site> --env=<env> watchdog-show --tail
```

### WordPress

Set the WP_DEBUG variable to "true" within your wp-settings.php file to display all PHP errors, notices, and warnings. Reference the [WordPress codex](http://codex.wordpress.org/Debugging_in_WordPress) for additional information on debugging in WordPress.

```php
define('WP_DEBUG', true);
```

## Raw Webserver Log Files

When developing a site, it can be useful to directly access the server logs for the site environment.  

From your Dashboard for a given site environment, click **Connection Info** for SFTP access credentials and take note of the non-standard port.

Once connected, you'll see several directories:

- **`code/`** - All executable code; stored in Git version-controlled repository. Cannot write (upload) to `code` unless site Connection Mode is set to [SFTP mode](/docs/articles/sites/code/developing-directly-with-sftp-mode/), on the Development environment only.
- **`files/`** - Pantheon File System (Valhalla) mount. `code/sites/default/files` and `code/wp-content/uploads` are symbolically linked to this directory. Read and write (upload), all environments.
- **`logs/`** - Environment-specific access and error logs. Read only.
 - **`newrelic.log`** - New Relic log; check if an environment is not logging.
 - **`nginx-access.log`** - Webserver access log. Do not consider canonical, as this will be wiped if the application server is reset or rebuilt.
 - **`nginx-error.log`** - Webserver error log.
 - **`php-error.log`** - PHP [fatal error log](http://php.net/manual/en/book.errorfunc.php); will not contain stack overflows. Errors from this log are also shown in the Dashboard.
 - **`php-slow.log`** - PHP-FPM generated collection of stack traces of slow executions, similar to MySQL's slow query log. See [http://php-fpm.org/wiki/Features#request\_slowlog\_timeout](http://php-fpm.org/wiki/Features#request_slowlog_timeout).
 - **`watcher.log`** - Log of service that checks for files changed in `code` directory while in SFTP Connection Mode.

### See Also
- [Automate Downloading Logs from the Live Environment](/docs/articles/sites/downloading-live-error-logs)

## Frequently Asked Questions

#### How can I parse my Nginx access logs?

You can use a free utility like [goaccess](http://goaccess.io/) to parse your Pantheon Nginx access logs. The Pantheon log format can be stored in the <tt>.goaccessrc</tt> configuration file as follows:

```nginx
time_format %H:%M:%S %z
date_format %d/%b/%Y
log_format %^ %^ %^ [%d:%t]  "%r" %s %b "%R" "%u" %T "%h"
```

#### Can I log to the system logger and access syslog?

The short answer is no, syslog is not available. Technically, you can log Drupal events using the syslog module, but you won't be able to read or access them.

#### Can I access Apache Solr logs?

No, access to Apache Solr logs is not available. For more information on debugging Solr, see [Apache Solr on Pantheon](/docs/articles/sites/apache-solr).

#### Can I download Varnish logs?

No, Varnish logs are not available for download.

#### My Drupal database logs are huge. Should I disable dblog?

The best recommended practice is to find and resolve the problems. PHP notices, warnings, and errors mean more work for PHP, the database, and your site. If your logs are filling up with PHP messages, find and eliminate the root cause of the problems. The end result will be a faster site.  

#### How do I access logs in environments with multiple containers?

Business and Enterprise plans have more than a single container in the Live environment. In order to download the logs from each application container, use the following shell script:

```bash
# Site UUID from Dashboard URL
SITE_UUID=UUID
for app_server in `dig +short appserver.live.$SITE_UUID.drush.in`;
do
mkdir $app_server
sftp -o Port=2222 live.$SITE_UUID@$app_server << !
  cd logs
  lcd $app_server
  mget *.log
!
done
```
