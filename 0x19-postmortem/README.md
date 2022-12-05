# Postmortem

Approximately 03:47 West African Time (WAT), an outage occurred on an isolated
Ubuntu 14.04 container running an Apache web server. GET requests on the server led to
`500 Internal Server Error`'s, when the expected response was an HTML file defining a
simple GcHub project I was working on for ALX foundation project.

## Debugging Process

Upon encoutering the issue while I was working on the project and made some querries. I was forced to solve the problem before I will be able to continue the project. As at 04:20 WAT I was still struggling to overcome the issue.

1. Checked running processes using `ps aux`. Two `apache2` processes - `root` and `www-data` -
   were properly running.

2. Looked in the `sites-available` folder of the `/etc/apache2/` directory. Determined that
   the web server was serving content located in `/var/www/html/`.

3. In one terminal, ran `strace` on the PID of the `root` Apache process. In another, curled
   the server. Expected great things... only to be disappointed. `strace` gave no useful
   information.

4. Repeated step 3, except on the PID of the `www-data` process. Kept expectations lower this
   time... but was rewarded! `strace` revelead an `-1 ENOENT (No such file or directory)` error
   occurring upon an attempt to access the file `/var/www/html/wp-includes/class-wp-locale.hp`.

5. Looked through files in the `/var/www/html/` directory one-by-one, using Vim pattern
   matching to try and locate the erroneous `.hp` file extension. Located it in the
   `wp-settings.php` file. (Line 137, `require_once( ABSPATH . WPINC . '/class-wp-locale.php' );`).

6. Added the trailing `p` from the line.

7. Tested another `curl` on the server. 200 A-ok!

## Summation

A typo made my blood pressure rose like something. The GcHub app was encountering a critical
error in `wp-settings.php` when tyring to load the file `class-wp-locale.hp`. The correct
file name, located in the `wp-content` directory of the application folder, was
`class-wp-locale.php`.

Patch involved a simple fix on the typo, adding the trailing `p`.

## Prevention

This outage was not a web server error, but an application error. To prevent such outages
moving forward, please keep the following in mind.

- Test! Always test your application before deploying. This error would have arisen
  and could have been addressed earlier had the app been tested.

- Status monitoring. Enable some uptime-monitoring service such as
  [UptimeRobot](./https://uptimerobot.com/) to alert instantly upon outage of the website.

In response to this error, I wasted a precious time which could have been used to add further functionalities to the application I was developing.

I have learnt my lesson and this will never happen again.
