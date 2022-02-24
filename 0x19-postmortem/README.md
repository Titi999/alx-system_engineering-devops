# Postmortem

Shortly after the introduction of ALX System Engineering & DevOps project 0x19, an outage occurred on an isolated Ubuntu 14.04 container running an Apache web server (Postmortem). GET queries on the server returned 500 Internal Server Errors when the intended response was an HTML file specifying a simple ALX WordPress site.

## Debugging Process

Abubakari Bilal, a bug debugger, discovered the problem when he opened the project and was told to fix it. He got right to work on resolving the situation.

1. 'ps aux' was used to check the status of ongoing processes. 'root' and 'www-data' were the only two 'apache2' processes that were up and running.

2. In the '/etc/apache2/' directory, I looked in the 'sites-available' folder. The web server was found to be providing material from the directory '/var/www/html/'.

3. Run 'strace' on the PID of the 'root' Apache process in one terminal. The server coiled in another. I had high hopes, just to be disappointed. 'strace' returned no meaningful results.

4. Step 3 was repeated, only this time on the PID of the 'www-data' process. This time, I kept my expectations lower... and was rewarded! When trying to access the file '/var/www/html/wp-includes/class-wp-locale.phpp','strace' revealed a '-1 ENOENT (No such file or directory)' error.

5. I went through each file in the '/var/www/html/' directory one by one, looking for the erroneous '.phpp' file extension using Vim pattern matching. It was found in the wp-settings.php file. ('require once(ABSPATH. WPINC. '/class-wp-locale.php');', line 137).

6. The trailing 'p' has been removed from the line.

7. On the server, I ran another 'curl' command. 200 A-ok!

8.I created a Puppet manifest to automate the mistake correction.

## Summation

In short, a typo. Gotta love'em. In full, the WordPress app was encountering a critical
error in `wp-settings.php` when tyring to load the file `class-wp-locale.phpp`. The correct
file name, located in the `wp-content` directory of the application folder, was
`class-wp-locale.php`.

Patch involved a simple fix on the typo, removing the trailing `p`.

## Prevention

This outage was not a web server error, but an application error. To prevent such outages
moving forward, please keep the following in mind.

* Test! Test test test. Test the application before deploying. This error would have arisen
and could have been addressed earlier had the app been tested.

* Status monitoring. Enable some uptime-monitoring service such as
[UptimeRobot](./https://uptimerobot.com/) to alert instantly upon outage of the website.

Note that in response to this error, I wrote a Puppet manifest
[0-strace_is_your_friend.pp](https://github.com/bdbaraban/holberton-system_engineering-devops/blob/master/0x17-web_stack_debugging_3/0-strace_is_your_friend.pp)
to automate fixing of any such identitical errors should they occur in the future. The manifest
replaces any `phpp` extensions in the file `/var/www/html/wp-settings.php` with `php`.

But of course, it will never occur again, because we're programmers, and we never make
errors! :wink:
