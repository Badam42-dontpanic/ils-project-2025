# Wordpress Fix
## Defining the Problem
Whenever a VM instance is started using gCloud, a new external IP address is generated. This causes a problem when trying to connect to a VM server's Wordpress site.  Wordpress is configured in a way that the Wordpress site will always try to connect to the IP address used when it was installed using the Wordpress web installer.  So the problem is, how do we change what IP address that Wordpress is looking for?

#### Important Note:
This solution was not found by me. I helped with theorizing what the eventual solution would be, but Jackson M. was the knight in shining armor that actually got this solution across the finish line. I'm writing this documentation by myself, but I would be remiss to not mention Jackson's essential contributions.

---
## Solution 1:
### Useful if: You don't plan on changing your IP address anymore / You don't want to change Wordpress config file.
Solution 1 involves going into your MySQL as the root user and looking at the database for Wordpress to change the data in the "wp_options" table.

First, log into MySQL as the root user and USE the wordpress database.

`sudo mysql -u root`

`USE "name_of_wordpress_database";`

You can see that Wordpress is looking for the wrong IP address by selecting the data in the wp_options table.

`SELECT * FROM wp_options WHERE option_name IN ('siteurl', 'home');`

By editing the option value, we can change the IP address that Wordpress is expecting.

`UPDATE wp_options SET option_value='http://<correct_IP>' WHERE option_name = 'siteurl' OR option_name = 'home';`

Confirm that the IP address is correct by selecting the data again.

`SELECT * FROM wp_options WHERE option_name IN ('siteurl', 'home');`

Exit MySQL

`\q`

This solution works best if you do not plan on changing the external IP address again, or if you don't want to go into the Wordpress configurations. 

---
## Solution 2:
### Useful if: Your external IP address is going to change again.
Solution 2 involves editing the wp-config.php file in your Wordpress directory to expect a dynamic IP address.

First, open the wp-config.php file.

`sudo nano /var/www/html/wordpress/wp-config.php`

Note: instead of nano, you can use your editor of your choice, like vim. 

At the bottom of the file, paste these two lines of code. They will change what site URL and HOME that Wordpress is looking for.

`define('WP_Home', '/wordpress/');`

`define('WP_SITEURL', '/wordpress/');`

Now, wordpress will not be looking for one specfic IP.

Save and exit the file.

This solution works best if you expect your external IP address to change frequently (or at all). For VM instances that generate a new IP every time they start, solution 2 is a much stronger fix than solution 1. 
