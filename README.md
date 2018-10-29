# How MIT’s AFS Work
---
# Introduction 
*The following document is a guide targeted at hosting websites on MIT’s AFS system and is for any club organization’s website team.*

Here’s the truth: **You will not be able to effectively run your club’s website if you do not spend a couple of hours reading these documentations.** Even if you avoid it, you will still have a harder time because the time spent learning how this system works below will tremendously reduce the cost you could later spend trying to figure it out on your own. 

Here, I tried to simplify the original documentation as much as possible, and more importantly, I tried to clarify misunderstanding or things that are generally difficult to grasp. I truly hope this helps. 

At the same time, I assume that you are somewhat comfortable with command line interface. If your organization had someone in charge of your organization's website before, I urge you to speak to them and have a 1-2 hours session in which they help you quickly figure out what the heck is going on with this intricate system. 

Anyways, *happy learning*!

_**Last Updated:** October 29, 2018_

# AFS
## AFS Summary

You can find all the details on the [SIBP’s documentation of AFS](https://sipb.mit.edu/doc/afs-and-you/), which is reachable from their [documentation page](https://sipb.mit.edu/doc/) should the link above ever change. Other helps are below.

## Quick Overview & Definitions

- In order to log-in from a personal computer using a terminal, just type ssh `<athena_username>@athena.dialup.mit.edu`. Then, this will prompt you for your password, which you can enter. Then, you can navigate within the directories (or "lockers").
- AFS stands for Andrew File System. 
- **Locker:** "For practical purposes, a folder. Probably what you'll care about most of the time. Technically any directory with a hesiod FILSYS entry, regardless of how its stored. Most lockers, however, are stored in AFS and are usually accessed as directories in `/mit`"
- **Tokens:** Proof to the AFS servers that you are who you say you are, thus allowing you to access files you are supposed to. Technically, a special type of Kerberos ticket.
- **Cells:** AFS concept of an "administrative domain of authority." Each cell has its own set of users, groups, and administrators. Analogous to a Kerberos realm. Each top-level directory in `/afs` generally corresponds to a cell. The cells you are most likely to care about are athena.mit.edu and sipb.mit.edu.

## Lockers

Each user has a locker, which can be found at `/mit/<username>`. This same locker is stored in `/afs/athena.mit.edu/user/<first_letter>/<second_letter>/<username>`. Lockers for projects, software, classes, living groups, and student groups are all mounted at `/mit/<lockername>` and stored in various places in AFS. 

Every locker at `/mit/<lockername>` have four main directories by default:

- **Public:** The content here can be read and listed by everyone. 
  - By "listed", I mean calling the "ls" command to see what files are in the directory. 
- **Private:** This directory can only be listed by the owner of the locker. Every other directories and files can be listed by everyone at MIT, but not read (except Public). This one is the only special one that cannot be listed by other MIT users.
- **OldFiles:** A backup folder. This is not too important. See documentations for more details.
- **www:** this is where one puts his/her static website.

If you are using a locker often, it’s recommended to attach it to `/mit` since they are by default found at `/afs/athena.mit.edu/<<<extra_path>>>`. To do so, simply type add `<lockername>` in the command line. Now, you will be able to find it at `/mit/<lockername>`. (You may need to do this every time you log into ssh. To prevent it from happening, type `echo "add <lockername>" > ~/.environment`. This will write the line add `<lockername>` into the file located in `~/.environment`. At later time, you will need to vim into `~/.environment` to prevent overwriting previous settings. 
Web access only serve static files. To serve more dynamic files, see [script.mit.edu](https://scripts.mit.edu/), which is also summarized towards the end of this document.

## Control Who Has Access
Control is managed by Access Control Lists (ACLs). To view who has control, type: 
```bash
fs listacl
```
A typical user locker would look like:
```bash
> fs listacl
	Access list for . is
	Normal rights:
	system:expunge ld
	system:anyuser l
	user rlidwka
```
That is just a list of users or AFS groups that have access to this locker. The meaning of each letter are shown below (copied from the site):

| letter abbreviation | word abbreviation | meaning |
|:---|:---|:---|
| `r` | _read_ | user or members of group can read files in the directory (i.e. see the contents of files) |
| `l` | _list_ | user or members of group can list files in the directory (i.e. see the names of files), they can also list the permissions of the directory. | 
| `i` | _insert_ | user or members of group can create files (or subdirectories) in the directory | 
| `d` | _delete_ | user or members of group can delete files in the directory |
| `w` | _write_ | user or members of group can modify files in the directory (i.e. change the contents of files) |
| `k` | _lock_ | user or members of group can lock files in the directory (you will likely never use this)
| `a` | _admin_ | user or members of group can change permissions of this directory (though not existing subdirectories, unless they have the a permission in those too. They will have admin powers in newly created subdirectories.) |
| `A,B,…,H` | _site-specific_ | Site-specific permissions that are meaningless to AFS but may be used by other software (or special versions of AFS). |

To add a user or group to the ACL for a given directory simply run `fs setacl` or `fs sa` as follows:
```bash
fs setacl -dir <directory> [<directory>]* -acl <user or group> <permissions> [<user or group> <permissions>]*
```
For example, let’s say your name is "bob" and you want to add writing permission to your friend "alice" into your locker called "awesome_me" (which is in your user locker for example), you would run:
```bash
> cd awesome_me
> fs setacl -dir . -acl alice write
```
Say you want to the same permissions to your friends "camille" and "dan", then you would run:
```bash
> fs setacl -dir . -acl camille write dan write
```
Say you want any user to have write permission into your locker, then you would run (_I recommend not doing this however…_):
```bash
fs setacl -dir . -acl system:anyuser write
```
Note that the "." means "this directory". `<directory>` can be an absolute path. Say you want to grant write permission into your locker for any user, you would run (_I REALLY advise against doing this however…_):
```bash
fs setacl -dir /afs/athena.mit.edu/user/b/o/bob -acl system:anyuser write
```
Also note that `<permissions>` can be any of the shorten strings concatenated or a comma separated value list of words. For example, if you want to grant read and write permission to any user, the following are equivalent:  
```bash
fs setacl -dir . -acl system:anyuser read,write
```
```bash
fs setacl -dir . -acl system:anyuser write,read
```
```bash
fs setacl -dir . -acl system:anyuser rw
```
```bash
fs setacl -dir . -acl system:anyuser wr
```
Note that the above is highlighting that the order doesn’t matter. You can grant all permissions using `all` or `rlidwka`.

There is a section on creating groups that’s worth reading. Two commands are important:

- This helps examine the details about a group: `pts examine <group>`
- This helps examine the members of a group: `pts membership <group>`

Usually, groups have “system” preceding them. For example, The africans group is `system:africans-www-group`.

## Control Who Has Access From the Web

You need to create a directory called `.htaccess.mit`. In the file, you can either:

- Require that the user has valid kerberos certificate by adding the line: `require valid-user`.
- Require access only to a specific user with: `require user alice bob <otherusers>`.
- Require access onto to a specific moira-group with: `require group <groupname>`. 

Finally, you need to add the line: `fs setacl -dir <dir> -acl system:htaccess.mit read`.
Make sure you add yourself if you want access from the web. Now, it can be access via online with `https://web.mit.edu/<locker>/<path to folder>`. It can’t be accessed via “HTTP”, we need the secure connection.

# Scripts.mit.edu (SME)

## Summary

The help in this section was found through [scripts.mit.edu](https://scripts.mit.edu/). Additionally, a lot of this has been both through mentoring from someone who has done it before and through playing around with various things. **SME (short for script.mit.edu)** is definitely the most confusing part. Those who initially wrote the documentations on SIPB assume a proficient technical background from the reader, and that is not always the case. This guide is an attempt at making the guide more accessible.

## Overview

SME is essentially a way to run scripts on MIT’s system. Normally, you are not allowed to do this because it’s insecure and more costly to MIT. So, what SIBP has done in the past is create SME, which essentially gives you the ability to run scripts on MIT’s system by creating a [sandbox](https://en.wikipedia.org/wiki/Sandbox_(software_development)) where to run your scripts for you or your organization’s locker exclusively. A sandbox is an isolated environment where one can run code without having the threat of being attacked due to the environment being isolated. See the wikipedia link for more details. 

## How to “Sign Up”

You will find this help at [scripts.mit.edu/web/](https://scripts.mit.edu/web/).

“Signing up” is very simple. You first run `add scripts`: This will add into your ssh environment the ability to run script specific commands. If you do not run this first, you will not be able to run the following methods.

Then, run `signup-web`. This effectively “signs” you up for being able to run scripts. It does two things:

- Create a directory called `web_scripts` in which you can now run your scripts. Unfortunately, these scripts are [Common Gateway Interface (CGI)](https://en.wikipedia.org/wiki/Common_Gateway_Interface) scripts. Essentially, the only file extensions you can run are `.pl`, `.py`, `.cgi`, and `.php`, which are perl, python, CGI and PHP respectively.
- Create the sandbox that is specific to you on SME’s end. I am actually not sure about this, but this is a safe assumption I made. 

Now that you are signed up, you are essentially done! At this point, if you go on `<lockername>.scripts.mit.edu`, it will point you to that folder similar to how when you go on `www.mit.edu/~<lockername>`, it points you to the `www` folder. In addition, any of these methods will point directly to the `index.<file_extension>` file, which will be “rendered”. To be more clear:

- If you go on `www.mit.edu/~<lockername>`, this will open the index.html file that lives there or give an error (it will likely say “Permission denied” or “directory listing forbidden”).
- If you go on `<lockername>.scripts.mit.edu`, this will run the script `index.[pl|py|cgi|php]` (NOTE I use the bracket and the “|” to signify that it will run one of the given options!). I am not sure what happens if you put two `index.[pl|py|cgi|php]` files, but you can give it a try and see what happens. 
- The script that runs is a CGI script, so from what I understand, it has to “return” or “output” a webpage (i.e. HTML). So, as you run your script, make sure you output the HTML. There are examples at the bottom of the page [scripts.mit.edu/web/](https://scripts.mit.edu/web/).

One thing you should know is that just like on AFS, you have quota with SME.

## Hosting A Website Using SME

Anyway, once that’s done, things can diverge pretty quickly from here depending on how your club decided to host their website. Here are some of the options, ranging from simple to complex:

- A series of simple and static **HTML files** that are just linked to each other. 
- Hosted on Github (the public version)
- Hosted on MIT’s Github (this is private to only MIT students)
- Hosted using Wordpress

I will now talk about each of them and how they work. 

### HTML Files

This is probably the simplest way to host your website on AFS system. As long as all the files are [static](http://www.spiderwriting.co.uk/static-dynamic.php), you probably don’t even need to use SME. 
If you have done basic web programming, this is simply a bunch of HTML files linking to each other (and of course some javascript files).
If you haven’t done basic web programming before, I recommend this [tutorial by MDN](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web). Once you learn some basic web development, you will essentially place the HTML files in the `www` directory, then name the home page `index.html`. Then, when you go on `http://www.mit.edu/~<lockername>`, it will open the file named `index.html` inside the `www` of this locker.

### Github Hosting

[Github](https://github.com/) is a very convenient way of sharing web page. It’s very similar to HTML Files. Essentially, you place your all your HTML files in the www folder. Then, you can host that on Github. 


**Why is this good?** 

You can edit the content of your website without needing to edit with [vim](https://www.engadget.com/2012/07/10/vim-how-to/). Vim is definitely very useful in terms of increasing your productivity, but vim is not beginner’s friend at all. So, it’s very nice to be able to edit the website without needing to navigate with the command line and vim. 

#### How it Works

You can create your project, making sure you include an `index.html` file in the root directory of your project. Then, you push your code into a github repository. Then, you navigate into the the www directory. Within that directory, clone the project from Github. 

#### How to Update the Website

You update your Github project similar to how you would update any Github project. However, the extra step you need to do is `git pull` from the `www` directory every time you make an update (i.e. every time you push from the files in your computer). 

#### Using SME Instead

Note that you can do any of the above in the `web_scripts` directory. Just replace index.html with `index.[pl|py|cgi|php]` (and of course any other `.html` file). Note that with this, you can do more. The only problem is that you are limited to using one of the following file type: `[pl|py|cgi|php]`.

#### Using MIT’s Github

MIT’s Github can be found at [github.mit.edu](https://github.mit.edu/). Using this is essentially the same as using regular Github. However, there is an extra step. You will need to use create an ssh key, which you will use every time you push into or pull from a project. 
In order to generate a key, go into the settings of your account (assuming you have created one), then go into “SSH and GPG keys”. You might need to do extra steps in order to get it work. I recommend looking at this [Github guide to SSH](https://help.github.com/articles/connecting-to-github-with-ssh/).

### Wordpress (WP)

The Wordpress (WP) installer is like the most buried useful thing out there. You can find it on this [link](https://scripts.mit.edu/news/80/update-to-wordpress-233). As a side note, the way I got there was to look up “wordpress” on the search bar on the SME website, then click on “Update to Wordpress 2.3.3”. 
The link says to do two things:
Run `add scripts` again. (you may need to do this each time you ssh).
Then run `scripts-wordpress`.
`scripts-wordpress` will take you to an interactive WP installer. It will ask you which locker you want to add wordpress to, and then it will do the necessary installments into the `web_scripts` directory on that locker. **Note that this will ask you for a username and password: take note of that and remember it**. You will need it to log in. When done, if you navigate to the web_scripts directory, you will find a wp directory. That is the WP directory. If you cd into it, you will find a bunch of mysterious files. Those files are generated by the WP installer. 
Before you start editing them, I’d suggest you go on `<lockername>.scripts.mit.edu/wp`. You will notice a fresh new but generic WP page. This is the newly created WP page, and you can change it. To change it, go into `<lockername>.scripts.mit.edu/wp/admin`. This is where you use the credentials to log in. Once you login, you get into a dashboard where you can easily edit the page. At this point, there is just too much going on, so I suggest just playing around to figure out what’s going on with WP. The one thing I will say is that if you go on the “appearance” section in the menu, then go on “editor”, you will see all those files and can change them manually! So, changing them via this online platform is much easier and better than using vim on the ssh environment because you can see the changes more real time. 

## Re-Pointing `<lockername>.scripts.mit.edu` To `<lockername>.mit.edu`

For this, please consult [scripts.mit.edu/web/](https://scripts.mit.edu/web/). At the very beginning, it talks about re-pointing your site.

## Beware of `.htaccess`

You may find a file named `.htaccess` or `.htaccess.mit` in your `www` repository. This file essentially sets rules that are applied to your repository when it comes to loading its website. 

One thing that you will sometimes want is to repoint `www.mit.edu/~<lockername>` to `<lockername>.scripts.mit.edu`. To do that, you simply need to add the following line to your `.htaccess` in your `www` directory:

```.htaccess
Redirect 301 /~<lockername> http://<lockername>.scripts.mit.edu
```

Here is an [Apache documentation](https://httpd.apache.org/docs/2.4/howto/htaccess.html) that goes more in depth. In addition, the `301` number above is a `redirect` HTTP status code. That is essentially a code that identifies something that happens. For example, you probably are familiar with the code `404`, which indicates that the website was not found. You can find the list of all status codes in this [HTTP RFC (RFC2616)](https://tools.ietf.org/html/rfc2616#section-10) going from section 10.

## Extra

You may still be lingering with questions. You can check out the [FAQ](https://scripts.mit.edu/faq/) on SME.

# How to Use `.htaccess`

If you create a wordpress site, you will notice a bunch of `.htaccess` files in directories and sub-directories. This file is a file that essentially gives directives to the PHP server. The MIT AFS system runs on PHP, so you must learn to deal with it.

This [guide](http://www.htaccess-guide.com/how-to-use-htaccess/) is pretty useful for describing the content of this page in details. Here, I will put common things about this file. The main thing to know is that each line or block on the `.htaccess` tells the server how to do something or what to do when something happens. It has a section on [`.htaccess` resources](http://www.htaccess-guide.com/useful-resources/).

This other [guide](http://httpd.apache.org/docs/1.3/howto/htaccess.html) is useful as well. 

## Rerouting to Error Documents

When someone loads your website with an invalid url, it's custom to point them to an error page (a nice one). To do that, you can create an error page on your site, then add the following line:

```
ErrorDocument 404 /path/to/error/from/root/error.php
```

Here, by root, I mean the path of the url `foo.mit.edu`. So, the root path is the file `/web_scripts`.

The `404` is the error status. You can specify a line like that for each error document. See the list of [HTTP error codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) (error codes are those with `4xx` and `5xx`). 

## Redirecting Urls

To redirect a given path, add the following:

```
Redirect [code] /old/path /new/path
```

For example, adding the line `Redirect 301 /data /web/data` redirects all urls with incoming source path `/data` to the path `/web/data`. So, if you look for `foo.mit.edu/data/some_data.json`, you will actually get `foo.mit.edu/web/data/some_data.json`.

The code `301` means "moved permanently", and the code `302` means food. You can view it from the list of [HTTP codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes). Note that the code for `Redirect` instructions is optional.

## Directory Index

This specifies a default file to open when a directory is accessed. The syntax is the following:

```
DirectoryIndex index.php
```

The above specifies to open `index.php` whenever one accesses the directory that the `.htaccess` lives in without a specific file path. 

## Run Commands For Specific Files or Directories

To run the directive on a specific file, you can add your directives within the block:

```
<Files filename>
# add directives here, will run only when 
# the file requested matches the filename
</Files>
```

You can also match a set of files by extension. If you wanted to have something for all `.js` and `.css` files, you can do the following:
```
<FilesMatch "\.(js|css)$">
# add directives here, will run only when 
# the file requested matches the filename
# that are javascript or css
</FilesMatch>
```

Above, we put a regular expression within quotation marks. If you don't know how that works, visit [RegexOne](https://regexone.com/). 

To match a specific directory, you can have:

```
<Directory "/www">
# add directives here, will run only when 
# the file requested is in the /www directory
</Directory>
```

## Allowing And Denying Access

It's recommended to put these within blocks. Say, for the directory `/www`, you only want to allow incoming sources from the website `www.example.com`. Then, you can add the following:

```
<Directory "/www">
  Order allow,deny
  allow from www.example.com
  deny from all
</Directory>
```

The order tells it to start with allow commands, then approach deny commands. You can also do the following to deny from `www.example.com`:

```
<Directory "/www">
  Order deny,allow
  deny from www.example.com
  allow from all
</Directory>
```

## Symbolic Links

See this [guide on symbolic links](http://www.maxi-pedia.com/FollowSymLinks).

To enable symbolic links, add:
```
Options +FollowSymLinks
```

This must be enabled in order to use `mod_rewrite.c`, which rewrites urls. See below.

## Rewrite Rules

Rewrite rules are a bit complicated. Essentially, given a url, if the url matches the condition given, the url is rewritten according to the rewriting rule. Here is an example:
```
<IfModule mod_rewrite.c>
  RewriteCond %{HTTP_HOST} ^example\.com$ [NC]
  RewriteRule ^(.*)$ http://www.example.com/$1 [L,R=301]
</IfModule>
```
The above redirect all users to access the site WITH the 'www.' prefix, (http://example.com/... will be redirected to http://www.example.com/...). 

If you are interested in rewriting the url rules, you can read more about it [here](https://httpd.apache.org/docs/current/mod/mod_rewrite.html). 

## Other Uses

There is so much to `.htaccess`. Here are links to other things you can do with it:

- [Password Protection](http://www.htaccess-guide.com/password-protection/)


# Troubleshooting

Unexpected things always happen! Here's a set of things you might run into. Some of them will be very specific. Some of them will be general. 

## Understand Your `.htaccess`

This file can cause trouble. A lot of the errors could happen because this file is not set properly. Look at the guide above to ensure you have it set up properly. 

Here, we outline some bugs that can happen because of it.  

### Some Scripts and Files Aren't Loading

Let's pretend you have your website setup in the following way, starting with `/web_scripts` directory:
```
/web_scripts
|_ index.php
|_ .htaccess
|_ /web
   |_ index.php
   |_ /js
      |_ main.js
   |_ /css
      |_ stylesheet.css
```
First, you notice that in the structure above, there is a `index.php` file in the `/web_scripts` directory. This is because this has to be there by default, otherwise the website won't know what to do. 

Let's say your locker is called `foo`. You can have the content of your initial `php` file be something like:
```php
/* 
 * this opens the index.php file inside of the /web directory, which
 * is the main point of entry of the website. 
 */
<?php require( dirname( __FILE__ ) . '/web/index.php' );
``` 
Then, you'd surely hope that if you go into `foo.mit.edu`, it loads all your files properly (the `.js` and `.css` files). 

However, that might not happen. If you go into your developers tool in your web browser and then go into the network tab, you'll notice that the website will load your scripts with `foo.mit.edu/js/main.js` and `foo.mit.edu/css/stylesheet.css`. However, these things are located in `foo.mit.edu/web/js/main.js` and ditto for `css`. 

One way to fix that is rewrite the url to that. However, that's not the proper way to do it. Instead, you would want urls for the sort `foo.mit.edu/js/` to reroute to `foo.mit.edu/web/js/`. To do that, you can call a Redirect in your `.htaccess`. 

It's very simple. Open your `.htaccess`, and add the following line:
```
Redirect 301 /js /web/js
Redirect 301 /css /web/css
```

### I made an Ajax call to some json file, and it doesn't load

The file you are trying to get with your call is probably not accessible. Try this link on [making files readable](https://scripts.mit.edu/faq/48/how-do-i-mark-a-file-as-world-readable-to-the-script-server).

On the other hand, if that doesn't work, you probably want to allow [Cross-Origin-Access](). You can add the following in your `.htaccess`: 
```
<IfModule mod_headers.c>
  Header set Access-Control-Allow-Origin "*"
  Header set Access-Control-Allow-Credentials true
</IfModule>
```
Note that with this, you will need to add `.htaccess` files in the sub directories as well where they are requested. 

**TODO:** This information may not be completely exact. Test it to figure out which of the two solutions above fixes the problem. Likely the first one. 

### I keep getting Emails From "(Cron Deamon)" With My Website

You might have the following line in your `.htaccess`:
```
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.php$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . /index.php [L]
</IfModule>
```

That is not the source of the email. If you go into [script.mit.edu/cron](https://scripts.mit.edu/cron/), you find the following. Cron is sort of like a periodic script that is ran. This script looks for `foo.mit.edu/cron.php`. The problem that is happening is that if you're missing the file `cron.php`, the rules above will redirect `foo.mit.edu/cron.php` to `foo.mit.edu/index.php`.

If you don't have anything periodic to run, you shouldn't run a crop script. To fix that, here's what you want to do. Go into the root directory of your locker, then `cd` into the `/cron_scripts` directory. There, you'll find multiple files and maybe a directory. 

Open `crontab` with `vi crontab`. If you aren't familiar with vim, see this [cheatsheet](https://vim.rtorr.com/). There should have a lot of comments explaining what it does. So, just read the comments. Notably, there is a setting that sets an email to send outputs of scripts to a given email. It looks like `MAILTO="{email}@mit.edu"`. That setting is what sends the email. There is also a setting to run the script `foo.mit.edu/cron.php`. It may look like:
```
0,30 * * * * /usr/bin/wget -O - -q http://foo.mit.edu/cron.php
```
That is basically to run that script every 30 minutes. The cron file should have descriptions on this. So, you can either remove this line or remove the email setting.


If you don't want to deal with command lines, this can also help: [how to list or remove cron script](https://scripts.mit.edu/faq/30/how-do-i-list-my-current-crontab-how-do-i-remove-my-crontab). In the mean time, removing that rule in the `.htaccess` will "solve" the problem. Sometimes, people who have worked on a locker in the past may have added periodic scripts that are not in use anymore. That is why you are getting this error. However, removing the line on the `.htaccess` is the hacky way to do it.

If you do want to run a script, you can also add `cron.php` file and put whatever you need in it. Not that the output will still be emailed to you in case you don't change the edits in your `crontab`.



# Commands Summary

- `add scripts`: Add scripts methods into your ssh environment. This allows you to run script specific commands, such as `signup-web`.
- `fs listquota`: see your quota usage
- `fs listacl` or fs la: list the users and permissions of a locker
- `fs setacl` or fs sa: set the permission for groups or users of a locker
- `pts examine <group>`: examine the details about a group
- `pts membership <group>`: find the members of a group
- `ssh <athena_username>@athena.dialup.mit.edu`: log into mit’s afs.
- `signup-web`: creates a `web_scripts` directory into the locker. In that folder, one can run scripts written in one of the following extensions: `.pl`, `.py`, `.cgi`, and `.php`.
- `scripts-wordpress`: creates a `wp` directory into the `web_scripts` directory within the locker. This then allows the wordpress page to be seen via `<lockername>.scripts.mit.edu/wp` or the link specified by the user on creation. 
- `tokens`: checks when tokens expire

# (Optional) Extra Information

## `chmod`

This is a special command that sets permissions. The [Wikipedia page](https://en.wikipedia.org/wiki/Chmod) about this is very detailed. Here is a summary.

To run a chmod command, the syntax is the following:
```bash
chmod [options] mode[,mode] file1 [file2 ...]
```
For example, 
```bash
chmod arwx index.php
```

Take a look at the wikipedia page. It has examples and describe what the above command does. 

Running 
```bash
chmod 777 FILENAME
```
Sets read, write, and execute rights to that file to all users, group, and owner. 

