# AGUILAS #

## A web-based LDAP user management system ##

Aguilas is a centralized registration system that allows users to create and manage their accounts on an LDAP authentication platform and (in some cases) using MYSQL databases to store temporary records. It's mostly written in PHP, and depends heavily on the use of LDAP and MYSQL servers. It has the following features:

  * Creates user accounts based on determined LDAP attributes.
  * Resends confirmation email in case it gets lost on spam folders.
  * Changes user password, if the user knows it.
  * Resets user password, using email as confirmation.
  * Reminds username.
  * Deletes user accounts.
  * Edits user profile (ajax enabled).
  * Lists all registered users.
  * Searches for a term in the user database.


The following topics will certainly help you to know more about Aguilas:



:)

# **CONTENTS** #

## Installation Methods ##

You can install Aguilas in various ways, depending on your requirements. Here you will find all the supported installation methods. Generally, the best and easier way to install Aguilas on Debian based distributions is to use a Debian package, see more on [installing from a debian package](#installing-from-a-debian-package.md). In any case, you will need to read [installing software dependencies](#installing-software-dependencies.md) before you can actually install Aguilas.

### Installing software dependencies ###

Aguilas requires the installation of certain software applications so that specific funcionality is available to it's internal processes. As of this documentation, we will be covering the installation of these dependencies on _Debian based distributions_ like Ubuntu, Canaima, Linux Mint and Debian (of course). If you use another type of GNU/Linux distribution (or Windows), you might need to investigate which packages or executables you need to install in order to satisfy these dependencies. In any case, you can always take a look at [getting help](#getting-help.md).

Basically, You will need:

  * A web server like Apache, Lighttpd or Nginx.
  * A Mail Transport Agent (MTA) like Exim, Pastfix or Sendmail.
  * PHP 5 or higher (with LDAP, MYSQL, MHASH, MCRYPT and CLI support).
  * Imagemagick image conversion utility with SVG support.
  * Python with Sphinx and Docutils support.
  * Icoutils.
  * Gettext.
  * Bash.
  * Make.


Optionally, you will have to install an LDAP and a MYSQL server depending on where is located the computer you are going to use as a server. If you are going to use a local computer as a server (e.g. your personal computer), then you will have to install and configure both on that computer. If you are going to use a remote computer as a server (e.g. a shared hosting server), then you will have to install and configure LDAP and MYSQL on that remote machine.

If you are impatient, you can install all dependencies at once with the following command:

```
aptitude install apache2 php5 php5-gd php5-ldap php5-mcrypt php5-mysql php5-mhash php5-suhosin php5-cli make bash gettext python-sphinx icoutils python-docutils libmagickcore-extra imagemagick apache2 mysql-server slapd postfix
```

If you want to keep reading, We will now explain how to install and configure each one of these dependencies. If you already have them installed in your working environment, please ignore it and proceed to the next step.

#### Installing Apache and PHP ####

Run the following command as superuser:

```
aptitude install apache2 php5 php5-gd php5-ldap php5-mcrypt php5-mhash php5-mysql php5-suhosin php5-cli
```

#### Installing the MYSQL server ####

Run the following command as superuser:

```
aptitude install mysql-server
```

You will be asked for a password for the "root" account for MYSQL. Remember this password as you will be asked for it later.

#### Installing the LDAP server ####

Run the following command as superuser:

```
aptitude install slapd
```

You will be asked for a password for the Distinguished Name (DN) of the administrator entry for LDAP; normally, the admin DN is _cn=admin,dc=nodomain_. Remember this password as you will be asked for it later.

#### Installing Python modules and other applications ####

Run the following command as superuser:

```
aptitude install make bash gettext python-sphinx icoutils python-docutils libmagickcore-extra imagemagick
```

#### Installing the Mail Transport Agent ####

If you already have a working Mail Transport Agent, please proceed to the next step.

Normally you can install a _Mail Transport Agent (MTA)_ like `postfix` or `exim` to serve as a mail server on any computer connected to the internet with a public IP address assigned. However, due to the problem of SPAM, many Internet mail servers block mail from unauthenticated dynamic IP addresses, which are common in domestic connections.

One of the existing solutions is to install a mail server to not send mail directly to the destination server, but to use [Google Mail](http://gmail.com) (or other SMTP) to retransmit the messages.

To send email using GMail SMTP server (`smtp.gmail.com`) the connection must be encrypted with TLS. To do this we need three elements:

  * A certificate authenticated by a valid certificate authority for _GMail_.
  * A _GMail_ email account.
  * A local MTA.


##### Installation #####

First install _Postfix_, a fairly complete and configurable MTA. Open a root terminal and type the following command:

```
aptitude install postfix
```

Note: Postfix conflicts with Exim, but it is safe to remove exim in favor of postfix.

The process will make us some questions:

  * _Configuration type_: you will select "Web Site".
  * _Mail System Name_: you will enter the domain name of your local mail server. For this case, we can enter the same domain name of your PC. e.g. "OKComputer".


Done, the installation is completed.

##### Configuration #####

Then we have to edit the file `/etc/postfix/main.cf` and add the following lines at the end of the file:

```
relayhost = [smtp.gmail.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl/passwd
smtp_sasl_security_options = noanonymous
smtp_use_tls = yes
smtp_tls_CAfile = /etc/postfix/cacert.pem
```

There we are telling `postfix` to use `relayhost` to connect to the Gmail server, which uses `smtp_sasl_password_maps` to extract the SASL data connection and use `smtp_tls_CAfile` as a certificate for the secure connection.

We must create the file `/etc/postfix/sasl/passwd` with the following contents:

```
[smtp.gmail.com]:587    [ACCOUNT]@gmail.com:[PASSWORD]
```

Where `[ACCOUNT]` is the gmail account name and `[PASSWORD]`, the password of `[ACCOUNT]`.

For example, we could use this command:

```
echo "[smtp.gmail.com]:587    luis@gmail.com:123456" > /etc/postfix/sasl/passwd
```

Then we must restrict it's access:

```
chmod 600 /etc/postfix/sasl/passwd
```

Next, we must transform the file into a postfix indexed hash file with the command:

```
postmap /etc/postfix/sasl/passwd
```

That will create the file `/etc/postfix/sasl/passwd.db`

##### Certification #####

We have to install the SSL certificates of certification authorities to perform this step. We can install them like this:

```
aptitude install ca-certificates
```

To add the _Equifax certificate authority_ (which certifies emails from Gmail) to authorized certificates that use postfix, run the following command in a root console:

```
cat /etc/ssl/certs/Equifax_Secure_CA.pem > /etc/postfix/cacert.pem
```

##### Testing #####

Finally, restart postfix to apply the changes, as follows:

```
/etc/init.d/postfix restart
```

You can check it's proper functioning by opening two consoles. In one execute the following command to monitor it's behavior (as root):

```
tail -f /var/log/mail.log
```

And in the other send a mail:

```
echo "This is a test mail" | mail test@gmail.com
```

If you did things right, on the other console you should see something like this:

```
Dec 18 18:33:40 OKComputer postfix/pickup[10945]: 75D4A243BD: uid=0 from=
Dec 18 18:33:40 OKComputer postfix/cleanup[10951]: 75D4A243BD: message-id=
Dec 18 18:33:40 OKComputer postfix/qmgr[10946]: 75D4A243BD: from=, size=403, nrcpt=1 (queue active)
Dec 18 18:33:44 OKComputer postfix/smtp[10953]: 75D4A243BD: to=<test@gmail.com>, relay=smtp.gmail.com[74.125.93.109]:587, delay=3.7, delays=0.15/0.14/1.8/1.6, dsn=2.0.0, status=sent (250 2.0.0 OK 1324249500 eb5sm36008464qab.10)
Dec 18 18:33:44 OKComputer postfix/qmgr[10946]: 75D4A243BD: removed
```

### Installing the stable version ###

  1. Download the source tarball from [Aguilas download page](http://code.google.com/p/aguilas/downloads/list). Select the version of your preference, usually the last one will be more complete.
  1. Decompress the source with your favorite program:
```
e.g.: tar -xvf aguilas-1.0.0.tar.gz
```

  1. Access the uncompressed source:
```
e.g.: cd aguilas-1.0.0/
```

  1. Build the sources:
```
make
```



You will be promted with the following questions to configure AGUILAS:
  * Name of the Application, e.g.: Aguilas for Debian User Management
  * The person or group responsible for managing the application, e.g.: Debian Admins
  * The e-mail address that will appear as sender in all operation e-mails to registered users, e.g.: [aguilas@debian.org](mailto:aguilas@debian.org)
  * The e-mail address you wish to use for sending error reports, e.g.: [admins@debian.org](mailto:admins@debian.org)
  * The language that you wish to see in your application (must be available on _locale_ folder), e.g.: en\_US
  * The theme applied to the application (must be available in _themes_ folder), e.g.: debian
  * The public address of the aplication, e.g.: aguilas.debian.org
  * IP or Domain of the server where the MYSQL database is located, e.g.: localhost
  * MYSQL database name (will be created if it doesn't exist), e.g.: aguilas
  * User with permissions to read and create tables on the database, e.g.: root
  * Password for the MYSQL user, e.g.: 123456
  * IP or Domain of the server where the LDAP service is located, e.g.: localhost
  * DN with read-write priviledges on the LDAP server, e.g.: cn=admin,dc=nodomain
  * Password for the writer DN, e.g.: 123456
  * Base DN to perform searches and include new users, e.g.: dc=nodomain



If you need to modify these parameters, you can always edit `/usr/share/aguilas/setup/config.php` after installation.

  1. Obtain superuser priviledges, and install aguilas:
```
sudo make install
```



### Installing the stable version using a debian package ###

Aguilas is distributed in various Debian derivatives. Grab the latest debian package from the [Aguilas download page](http://code.google.com/p/aguilas/downloads/list) and install it with the following command:

```
dpkg -i [PATH/TO/FILE]
```

### Download the development version ###

If you want to download the development version of Aguilas, which has the lastest changes and new features, you can follow the procedure described below. You should install this version if you are planning to contribute in Aguilas development or try some hard-on new features. You should also know that a development version is likely to be unstable or even broken, as it is being developed actively.

That being said:

  1. Start cloning the development branch from Aguilas (you will need to install the `git-core` package first):
```
git clone --branch development https://github.com/HuntingBears/aguilas.git
```

  1. Access the folder that has just been created:
```
cd aguilas
```

  1. Prepare and update the sources:
```
make prepare
```



That's it. You have the latest code from Aguilas. If you want to install it, you can follow the same procedure described at [installing the stable version](#installing-the-stable-version.md).

## Getting help ##

If you are in trouble, you can alwas ask for help here:

  * [Aguilas IRC Channel](irc://irc.freenode.net/aguilas).
  * [Aguilas Mailing List](http://groups.google.com/group/aguilas-list).


## Contributing to Aguilas ##

We're passionate about helping Aguilas users make the jump to contributing members of the community, so there are many ways you can help aguilas development:

  * Report bugs and request features in our [ticket tracker](https://github.com/HuntingBears/aguilas/issues). Please read [reporting bugs](#reporting-bugs.md), below, for the details on how you sould do this.
  * Submit patches or pull requests for new and/or fixed behavior. Please read [submitting patches](#submitting-patches.md), below, for details on how to submit a patch. If you're looking for an easy way to start contributing to Aguilas have a look at the easy-pickings tickets.
  * Join the [Aguilas Mailing List](http://groups.google.com/group/aguilas-list) and share your ideas for how to improve Aguilas. We're always open to suggestions.
  * Improve Aguilas documentation by reading, writing new articles or correcting existing ones. You can find more information on how we do this at [writing documentation](#writing-documentation.md).
  * Translate Aguilas into your local language, by joining our translation team. Read more on [translating aguilas](#translating-aguilas.md).


That's all you need to know if you'd like to join the Aguilas development community. The rest of this document describes the details of how our community works and how it handles bugs, mailing lists, and all the other details of Aguilas development.

### Reporting bugs ###

Well-written bug reports are incredibly helpful. However, there's a certain amount of overhead involved in working with any bug tracking system so your help in keeping our ticket tracker as useful as possible is appreciated. In particular:

  * Check that someone hasn't already filed the bug or feature request by searching in the [Ticket Tracker](https://github.com/HuntingBears/aguilas/issues).
  * If you are not sure if what you are seeing is a bug, ask first on the [Aguilas Mailing List](http://groups.google.com/group/aguilas-list) or the [Aguilas IRC Channel](irc://irc.freenode.net/aguilas).
  * Don't use the ticket system to ask support questions. Use the [Aguilas Mailing List](http://groups.google.com/group/aguilas-list) or the [Aguilas IRC Channel](irc://irc.freenode.net/aguilas) for that.
  * Write complete, reproducible, specific bug reports. You must include a clear, concise description of the problem, and a set of instructions for replicating it. Add as much debug information as you can: code snippets, test cases, error messages, screenshots, etc. A nice small test case is the best way to report a bug, as it gives us an easy way to confirm the bug quickly.


First of all, [Signup at GitHub](https://github.com/signup/free), is very quick and easy.

Next, gather all the information you have about the bug you are going to report (_everything_ could be useful). Go to the [New Issue Form](https://github.com/HuntingBears/aguilas/issues/new) in our [Ticket Tracker](https://github.com/HuntingBears/aguilas/issues) and fill in a descriptive title for the ticket and the content of the bug you are reporting.

If you need to attach a piece of code, error or patch, please use the [Gist](https://gist.github.com/). The gist is a special service from github to add pieces of code in a tiny repository publicly available. Add the code you need in a [New public Gist](https://gist.github.com/gists/new), and then paste the resultant link in the bug report.

### Submitting patches ###

A patch is a structured file that consists of a list of differences between one set of files and another. All additions and deletions of **contributed code** to Aguilas are done through patches.

A patch can be sent to Aguilas developers through the [Ticket Tracker](https://github.com/HuntingBears/aguilas/issues) and the [Gist](https://gist.github.com/). Add the patch code in a [New public Gist](https://gist.github.com/gists/new) and then [open a new ticket](https://github.com/HuntingBears/aguilas/issues/new) describing the patch you made. Remember to add the link to the public gist so it can be referenced and checked out by developers. Also, you will need to [open an account](https://github.com/signup/free) on github to do all of this.

If you want to know more about creating, applying and submitting patches, see [working with patches](#working-with-patches.md).

If you want to install the development version of Aguilas to start reviewing, understanding and improving the code, read [downloading the development version](#downloading-the-development-version.md).

You might also want to start [understanding aguilas source code](#understanding-aguilas-source-code.md).

### Submitting pull requests ###

A pull request is a special service from code repositories that enable developers to manage contribution activities more easily. It allows a developer to clone or fork a repository of code, make changes to it and then request the addition of this modified code to the upstream developers of Aguilas.

If you want to contribute to Aguilas using this method, you will have to fork Aguilas on github, download the forked code to your computer, make the changes you need to make in order to correct the bug or implement a new feature, push your changes and finally request your code to be pulled to Aguilas main repository. You will need to [open an account](https://github.com/signup/free) on github to do all of this.

If you want detailed information how to make a pull request, please read the [pull request guide](#pull-request-guide.md).

You might also want to start reading [understanding aguilas source code](#understanding-aguilas-source-code.md).

There's also more information about [working with git](#working-with-git.md) available.

### Writing documentation ###

We place a high importance on consistency, readability, simplicity and portability of documentation. After all, great documentation is what makes great projects great.

Documentation changes generally come in two forms:

  * **General improvements:** typo corrections, error fixes and better explanations through clearer writing and more examples.
  * **New features:** documentation of features that have been added to the core since the last release.


Aguilas's documentation uses the [Sphinx Documentation System](http://sphinx.pocoo.org), which in turn is based on reStructuredText (reST or RST). The basic idea is that lightly-formatted plain-text documentation is transformed into HTML, WIKI, and MAN pages with minor efforts. We always modify the documentation in the development stage/branch.

If you want to obtain a copy of the development version of Aguilas to start reviewing, understanding and improving documentation, read [downloading the development version](#downloading-the-development-version.md).

To actually build the documentation locally, you'll currently need to install sphinx and other applications. Read the document on [installing software dependencies](#installing-software-dependencies.md) for the complete list of dependencies and how to install them.

Once downloaded, you will find the documentation sources (rest format) on `documentation/rest`. You will also notice other folders (described below). These are autogenerated documentation formats, **DO NOT** edit them directly, edit the sources instead.

  * `documentation/html`:  Contains the generated HTML using sphinx. These HTML pages are updated using the rest sources located at `documentation/rest` through the command `make gen-html`.
  * `documentation/man`:  Contains the rest source to produce a manual page (man) through the `rst2man` application. `make gen-man` on the top level directory will produce the file `documentation/man/aguilas.1`.
  * `documentation/githubwiki`:  This directory contains a git submodule for the GitHub Wiki. The generated wiki format is copied here and uploaded to GitHub to keep all documentation in sync. `make gen-wiki` will update the files on this folder using the rest sources on `documentation/rest`.
  * `documentation/googlewiki`:  This directory also contains a git submodule but for the Google Code Wiki. The generated wiki format is copied here and uploaded to Google Code to keep all documentation in sync. `make gen-wiki` will update the files on this folder using the rest sources on `documentation/rest`.


If you want to modify or add new documents, feel free to inspect the `documentation/rest` folder. If you add a new rest file in the sources folder, don't worry about it not being referenced on the index or the table of contents, this is done automatically at build time.

To get started contributing, you'll want to read the [reStructuredText basics](http://docutils.sourceforge.net/docs/user/rst/quickref.html). After that, you'll want to read [understanding aguilas source code](#understanding-aguilas-source-code.md) to know how and where you are going to make the changes.

If you want your contributions to be added on official documentation, please read [submitting patches](#submitting-patches.md) and [submitting pull requests](#submitting-pull-requests.md).

### Translating Aguilas ###

Translations are contributed by Aguilas users everywhere. The translation work is coordinated at [Transifex](https://www.transifex.net/projects/p/aguilas/) and the [Aguilas Mailing List](http://groups.google.com/group/aguilas-list).

If you find an incorrect translation or want to discuss specific translations, go to the translation team page for that language. If you would like to help out with translating or add a language that isn't yet translated, here's what to do:

  1. Join the [Aguilas Mailing List](http://groups.google.com/group/aguilas-list) and introduce yourself, explaining what you want to do.
  1. [Signup at Transifex](https://www.transifex.net/plans/signup/free/) and visit the [Aguilas Project Page](https://www.transifex.net/projects/p/aguilas/).
  1. On the [Translation Teams Page](https://www.transifex.net/projects/p/aguilas/teams/), choose the language team you want to work with, or – in case the language team doesn't exist yet – request a new team by clicking on the "Request a new team" button and select the appropriate language.


Then, click the "Join this Team" button to become a member of this team. Every team has at least one coordinator who is responsible to review your membership request. You can of course also contact the team coordinator to clarify procedural problems and handle the actual translation process.

Once you are a member of a team choose the translation resource you want to update on the team page. For example the "core" resource refers to the main translation catalogue.

#### Generating POT templates ####

A POT template is a file which contains all translatable strings contained in the PHP sources; these strings are translated in a PO file for each language. You will need Aguilas POT template (which is located in `locale/pot/aguilas/messages.pot`) to update the PO file of the language you are working on.

To update or generate Aguilas POT template, enter the following command in the root folder of Aguilas development version (see [downloading the development version](#downloading-the-development-version.md)):

```
make gen-pot
```

#### Updating PO files from POT template ####

To update all PO files in the _locale_ folder using Aguilas POT template (`locale/pot/aguilas/messages.pot`), enter the following command in the root folder of Aguilas development version (see [downloading the development version](#downloading-the-development-version.md)):

```
make gen-po
```

#### Generate a new PO with Aguilas translation assistant ####

If you want to create a new PO for a new language, you can use the Aguilas translation assistant which is located at `tools/maint/l10n-newpo.sh`. Enter the following command, passing as argument a valid l10n code according to the language you wish to translate to:

```
./tools/maint/l10n-newpo.sh [L10N CODE]
```

For example:

```
./tools/maint/l10n-newpo.sh en_GB
```

Basically, it will copy the POT template to the proper folder, depending on what l10n code you specify; then, it will start asking you to translate each string to generate the new PO.

## Contributing code through pull requests ##

The Aguilas development model follows the standard _GitHub model_ for contributions: [fork a project](https://github.com/HuntingBears/aguilas/), clone it to your local computer, hack on it there, push your finished changes to your forked repository, and send a _Pull Request_ back upstream to the project. If you're already familiar with this process, then congratulations! You're done here, [get hacking](https://github.com/HuntingBears/aguilas/)!

The GitHub guide for the standard [fork](http://help.github.com/fork-a-repo)/clone/push/[pull request](http://help.github.com/send-pull-requests/) model of contributing is a great place to start, but we'll cover all the basics here, too, as well as various contribution scenarios (fixing bugs, adding features, updating documentation, etc.).

### Getting Started ###

First, you'll need a GitHub account. If you don't have one, go and [Signup at GitHub](https://github.com/signup/free). After that, you'll need to [provide your SSH key](http://help.github.com/linux-set-up-git/) to authorize your computer to make pushes.

You need to download and install Git and then configure it. You might want to take a look at our [working with git](#working-with-git.md).

Now you're ready to grab the Aguilas repository. Hit the [Aguilas Repository Page](https://github.com/HuntingBears/aguilas/) and click the _Fork_ button. This will create a complete copy of the Aguilas repository within your GitHub account.

Now you've got your own fork. This is your own personal copy of the Aguilas source tree, and you can do anything you want with it. Next, let's clone your fork of Aguilas. Just switch to a directory where you want to keep your repository and do this, replacing `[USERNAME]` with your actual GitHub username:

```
git clone --branch development git@github.com:[USERNAME]/aguilas.git
```

Congratulations! You now have your very own Aguilas repository. Now you'll want to make sure that you can always pull down changes from the upstream canonical repository. To do so, do this:

```
cd aguilas
git remote add upstream git://github.com/HuntingBears/aguilas.git
git pull upstream development
```

Anytime you want to merge in the latest changes from the upstream repository, just issue the command `git pull upstream development` to pull the latest from the development branch of the upstream repository and you'll be good to go. You should also push it up to your fork:

```
git push origin development
```

### Submitting the pull request ###

After you have pushed the changes to your fork, go to your fork page on github and hit the _pull request_ button. After that, you are presented with a preview page where you can enter a title and optional description, see exactly what commits will be included when the pull request is sent, and also see who the pull request will be sent to (Aguilas upstream repository).

More information on [pull request](http://help.github.com/send-pull-requests/).

## Working with patches ##

### What's a patch? ###

A patch is a structured file that consists of a list of differences between one set of files and another. All additions and deletions of **contributed code** to Aguilas are done through patches.

**Patches make development easier**, because instead of supplying a replacement file, possibly consisting of thousands of lines of code, the patch includes only the exact changes that were made. In effect, a patch is a list of all the changes made to a file, which can be used to re-create those changes on another copy of that file.

Here is an example of what a patch looks like:

```
diff --git a/token_example/token_example.tokens.inc b/token_example/token_example.tokens.inc
index 585dcea..b06d9d6 100644
--- a/token_example/token_example.tokens.inc
+++ b/token_example/token_example.tokens.inc
@@ -13,8 +13,8 @@ function token_example_token_info() {
   // second is the user's default text format, which is itself a 'format' token
   // type so it can be used directly.

-  // This is a comment in the original file. It will be removed when the patch is applied.
+ // And here are lines we added when we were editing the file.
+ // They will replace the line above when the patch is applied.
$info['types']['format'] = array(
     'name' => t('Text formats'),
     'description' => t('Tokens related to text formats.'),
```

At the top it says which file is being affected. Changes to several files can be included in one patch. Lines added are shown with a '+', lines removed are shown with a '-', lines changed are shown as the old line being removed and the new one added.

Patch files are good because they only show the changed parts of the file. This has two advantages: it's easy to understand the change; and if other parts of the same files change between the patch being made an being used, there is no problem, the patch will still apply.

Note that there are several slight variants of the patch format. The example above is a 'unified' patch, which is now the standard.

If you want to create a patch, the first thing you will have to do is to download the latest snapshot (development stage) of Aguilas to your computer. Use git to clone the project and obtain a copy from github (you only need to do this once):

```
git clone --branch development https://github.com/HuntingBears/aguilas.git
```

Aguilas code is also mirrored on [Google Code](https://code.google.com/p/aguilas) and [Gitorious](https://gitorious.org/huntingbears/aguilas).

If you already have cloned the code, then pull the latest changes down to ensure you're working with the latest snapshot:

```
git checkout development
git pull origin development
```

### Creating a patch using diff ###

  1. Make a copy of the entire folder that contains the code:
```
cp -r aguilas/ aguilas.changes/
```

  1. Start making the necessary changes to implement the feature or bug correction you have in mind:
```
cd aguilas.changes/
# Edit file in your favorite editor:
<your_editor_here> [filename]
# Make more edits in other files and save theme:
<your_editor_here> [other-filename]
```

  1. To create the patch "my\_changes.patch", issue the following command at folder containing the original and the changed folder:
```
diff -Naur aguilas.orig aguilas.changes > my_changes.patch
```



### Creating a patch using git ###

  1. Create and checkout a _local branch_ (with the name of your preference) for the issue/feature/bug you're working on:
```
git branch [NEWBRANCH]
git checkout [NEWBRANCH]
```

  1. Make whatever changes are necessary to complete what you're doing:
```
# Edit file in your favorite editor:
<your_editor_here> [filename]
# After making your edits and saving the file, confirm your changes are present:
git status
# Make more edits in other files and save theme:
<your_editor_here> [other-filename]
# Confirm changes again:
git status
```

  1. When you are satisfied, make one single commit with all your changes using the -a option:
```
git commit -a -m "Descriptive message here."
```

  1. Now, create your patch file. The following command will output the difference between the current branch (where your changes are) and the _development_ branch that you cloned from the code repository to a file named "my\_changes.patch":
```
git diff development > my_changes.patch
```



### Submitting the patch ###

A patch can be sent to Aguilas developers through the [Ticket Tracker](https://github.com/HuntingBears/aguilas/issues) and the [Gist](https://gist.github.com/). Add the patch code in a [New Public Gist](https://gist.github.com/gists/new) and then [open a new ticket](https://github.com/HuntingBears/aguilas/issues/new) describing the patch you made. Remember to add the link to the public gist so it can be referenced and checked out by developers. Also, you will need to [open an account](https://github.com/signup/free) on github to do all of this.

## Describing Aguilas's develpment cycle ##

Aguilas's stable release version numbering works as follows:

Stable versions are numbered in the form `X.Y.Z`.
  * **X** is the major version number, which is only incremented for major changes to Aguilas.
  * **Y** is the minor version number, which is incremented for large yet backwards compatible changes.
  * **Z** is the micro version number, which is incremented for bug and security fixes.
  * In some cases, we'll make alpha, beta, or release candidate releases. These are of the form `X.Y.Z+{alpha,beta,rc}N`, which means the Nth alpha, beta or release candidate of version X.Y.Z.

  * Development versions are numbered in the form `X.Y.Z+[YEAR][MONTH][DAY][HOUR][MINUTE][SECONDS]`.


## Understanding Aguilas's Source Code ##

When deploying Aguilas into a real production environment, you will almost always want to use an official packaged release (see [installation methods](#installation-methods.md)) of Aguilas. However, if you'd like to try out in-development code from an upcoming release or contribute to the development of Aguilas, you'll need to obtain a copy from Aguilas's source code repository. This document covers the way the code repository is laid out and how to work with and find things in it.

### Aguilas repositories ###

The Aguilas development model uses _git_ (see [working with git](#working-with-git.md)) to track changes to the code over time and manage collaboration. You'll need a copy of the git client program on your computer, and you'll want to familiarize yourself with the basics of [working with git](#working-with-git.md).

The Aguilas git repository is located online at various places, all syncronized with the last code from the authors. We host Aguilas on [Google Code](https://code.google.com/p/aguilas), [GitHub](https://github.com/HuntingBears/aguilas), and [Gitorious](https://gitorious.org/huntingbears/aguilas). In terms of source code, all of them have the same information, so, you can choose the one you like the most. We use GitHub for tracking bugs and other project management stuff. If you want to contribute to Aguilas, please read our [contributing to aguilas](#contributing-to-aguilas.md) section.

### Branches and Tags ###

Inside the repository you will find that Aguilas development is divided into four branches: development, release, master and debian.

`development branch`
> This is where the development is actually made, and because of that, the code provided here might be experimental, unstable or broken. If you want to contribute to Aguilas (see [contributing to aguilas](#contributing-to-aguilas.md)), _this is where you should start_.


`release branch`
> This is where stable releases are obtained from. After a development cycle is finished, all changes are merged here, plus other _tactical movements_ (see [development cycle](#development-cycle.md)).


`master branch`
> The purpose of the master branch is to serve as a container to build the Aguilas debian package.


Tags in aguilas are done each time a new (development or release) version is made. The difference is that development (_snapshots_) versions are numbered in the form of `X.Y.Z+[YEAR][MONTH][DAY][HOUR][MINUTE][SECONDS]` and stable (_releases_) versions are numbered in the form `X.Y.Z`.

### Common tasks and procedures ###

Every common task for the Aguilas maintainers, developers, contributors and users are automatized using the Makefile. Here they are listed:

#### Checking dependencies ####

  * `make check-buildep`:  Checking build tasks dependencies. It allows the user/developer to check if all software dependencies requiered to build documentation, themes, tranlations and cofigurations are met.
  * `make check-maintdep`:  Checking maintainer tasks dependencies. It allows the developer to check if all software dependencies requiered to exacute common maintainer (update PO and POT files, make a snapshot, make a release) tasks are met.
  * `make check-instdep`:  Checking install dependencies. It allows the user to check if all software dependencies for installation are met.


#### Maintainer tasks ####

  * `make gen-po`:  Generates PO files from POT templates. It generates the PO files for every language listed in the `locale` folder, using the POT file located at `locale/pot/aguilas/messages.pot`.
  * `make gen-pot`:  Generates POT template from PHP sources. It generates the POT template from every traslatable string in the PHP sources passed through.
  * `make snapshot`:  Generates a development snapshot.
  * `make release`:  Generates a stable realease.


#### Build tasks ####

  * `make gen-img`:  Generates PNG images and ICO icons for each theme, based on the SVG sources.
  * `make gen-mo`:  Generates MO files from PO sources.
  * `make gen-man`:  Generates the MAN page from REST sources.
  * `make gen-wiki`:  Generates the wiki pages for Google Code Wiki and GitHub Wiki from REST sources
  * `make gen-html`:  Generates HTML pages from REST sources.
  * `make gen-doc`:  Executes gen-man, gen-wiki and gen-html.
  * `make gen-conf`:  Asks the user for the configuration data and builds config.php.


#### Clean tasks ####

  * `make clean-img`:  Cleans generated PNG and ICO files.
  * `make clean-mo`:  Cleans generated MO files.
  * `make clean-html`:  Cleans generated HTML files.
  * `make clean-wiki`:  Cleans generated WIKI files.
  * `make clean-man`:  Cleans generated MAN
  * `make clean-conf`:  Cleans generated config.php.


#### Install tasks ####

  * `make copy`:  Copies application files to it's destination.
  * `make config`:  Configures Aguilas using the generated config.php.
  * `make install`:  Does `make copy` and `make config`.
  * `make uninstall`:  Deconfigures and removes Aguilas data.
  * `make reinstall`:  Does `make uninstall` and `make install`.


### Files and Directories ###

Aguilas directory structure is distributed in the following form:

```
.
├── [PHP FILES]
├── [PHP FILES]
├── [...]
├── [DOCUMENTATION FOLDER]
│   ├── [GITHUBWIKI FOLDER]
│   ├── [GOOGLEWIKI FOLDER]
│   ├── [HTML FOLDER]
│   ├── [MAN FOLDER]
│   └── [REST FOLDER]
├── [EVENTS FOLDER]
├── [LIBRARIES FOLDER]
├── [LOCALE FOLDER]
│   ├── [LOCALE A]
│   ├── [LOCALE B]
│   ├── [...]
│   └── [POT FOLDER]
├── [SETUP FOLDER]
├── [THEMES FOLDER]
├── [TOOLS FOLDER]
└── Makefile
```

## Working with git ##

Git is a free and open source, distributed version control system designed to handle everything from small to very large projects with speed and efficiency. If you want a list of common commands, see the [git cheat sheet](#git-cheat-sheet.md) or it's [graphical summary](http://zrusin.blogspot.com/2007/09/git-cheat-sheet.html).

For those too impatient to read, here's what you need to work with someone else's Git project:

```
git clone [URL]
git add .
git commit -a
git push
git pull
```

Before running any command the first time, it's recommended that you skim through its manual page. Many of the commands have very useful and interesting features (that we won't list here) and sometimes there are some extra notes you might want to know. You can also get help on any Git command by doing `git [COMMAND] -h` or `git help [COMMAND]`.

### Installing git ###

If you are using Debian, Ubuntu, Canaima or other Debian derivative, you can install git easily with the following command (as an administrator user):

```
aptitude install git-core
```

### Configuring git ###

Git includes your real name and your e-mail address on every commit you make. You should add these before you start using Git:

```
git config --global user.name "Your Complete Name"
git config --global user.email you@yourdomain.example.com
```

`git config` is the command to set configuration options. While you're configuring, here are some other things you might like to do:

```
# colorized `git diff`, etc.
git config --global color.ui auto

# list all configuration options
git config -l
```

### Creating a New Repository ###

Let's step through how you create and update a new project:

If you are creating a new project, you should:

```
mkdir myproject; cd myproject
git init
```

If you are putting an existing project under git version control, you should:

```
cd myproject
git init
git add .
git commit -a
```

`git init` initializes the repository, `git add .` adds all the files under the current directory and `git commit -a` creates the initial import of files.

If you are downloading someone else's git project, you should do this instead:

```
git clone [URL]
```

Whichever the case, now your tree is officially tracked by git. One nice thing to note - no matter how many subdirectories your project has, there's only one .git directory that contains all the version control information. Do some random changes to your tree now - poke around in a few files or something.

### Making Changes ###

When you've edited some files, next thing you have to do is add them to version control. First you check what you've done:

```
git diff
```

That's it. This is one of the more powerful commands. To get a diff with a specific commit, do:

```
git diff [COMMIT]
```

You can obtain the commit number by listing all commits with:

```
git log
```

Also, there is a more concise representation of changes available:

```
git status
```

This will show the concise changes summary as well as list any files that you haven't either ignored or told Git about. In addition, it will also show at the top which branch you are in.

The status command also shows the "untracked files" that Git doesn't know what to do with. You can either include them in the registry (in the case these are new files):

```
git add .
```

Or delete them (in the case these are unwanted files from a build process, for example). _WARNING_ this will delete all untracked files forever:

```
git clean -fd
```

You could also list all files you want to ignore forever. These files will never be added to version control. What you need to do is create a file named `.gitignore`, containing a list of the files, line by line.

If you made changes to a file that you want to undo, you can get back the last version you committed:

```
git checkout [PATH/TO/FILE]
```

So after you have messed around with the project files, it's about time for us to commit our changes:

```
git commit -a
```

There are two considerations to observe:

First, you have to specify `-a` if you want to commit all your files, or `git commit [PATH/TO/FILE]` to commit file by file.

Second, Git commits are private by default - they aren't pushed to any central server unless you specify it. We'll talk about pushing changes later, but private commits have some important benefits. For example, when you realise you left some debugging iformation in your last commit, or made a typo in the commit message, you can do `git commit --amend` to fix it, or even do `git reset HEAD^` to toss the commit away completely without affecting your files.

A few words about the commit message: it is customary to have a short commit summary as the first line of the message, because many tools just show the first line of the message. You can specify the commit message using the -m parameter (extra -m arguments will create extra paragraphs in the commit message).

### Browsing the repository ###

Now that we have committed some stuff, you might want to review your history:

```
git log
git blame [PATH/TO/FILE]
```

The `log` command is very powerful, it shows you the complete history of the project, commit by commit, and also extra information like authors and dates. For example, `git log --oneline` will show you the first few characters of each commit ID and the first line of each commit message. See the _git-log manual page_ for more stuff git log can do.

The `blame` command is also very useful, as it identifies the author of every line of every file registered by git.

You can see the contents of a file, the listing of a directory or a commit with:

```
git show [COMMIT]:[PATH/TO/FILE]
git show [COMMIT]:[PATH/TO/DIRECTORY]
git show -s [COMMIT]
git show [COMMIT]
```

### Branching and Tagging ###

Git marks checkpoints in history by applying a label to a commit. You can create a branch with the following commands:

git branch [NEW](NEW.md) [OLD](OLD.md) git checkout [NEW](NEW.md)

The first command creates a branch, the second command switches your tree to the new branch. You can pass an extra argument to `git branch` to base your new branch on a different revision than the latest one.

Running `git branch` without arguments lists your branches. The **in the output marks the current branch:**

```
git branch
```

To move your tree to some older revision, use:

```
git checkout [COMMIT]
```

Git tags are fairly similar to Git branches, but with some extra tricks. Git tags can have a date, committer, and message that act just like the equivalents for Git commits. They can also be signed with a PGP key if you really want to stamp them with your seal of approval. This is great if you want to release a public version of your work, because you can have one place to store your release announcement and your guarantee that the code hasn't been tampered with. So, let's do it:

```
git tag -a [NAME]
```

To list tags and to show a tag message:

```
git tag -l
git show [NAME]
```

### Merging branches ###

Let's suppose you are on branch "release", and you want to bring the changes you've made on "development" branch, then you'll have to do:

```
git merge development
```

If changes were made on only one of the branches since the last merge, they are simply replayed on your other branch (a so-called fast-forward merge). If changes were made on both branches, they are merged intelligently (a so-called three-way merge). If the three-way merge doesn't have any merge conflicts, it makes a commit with a convenient log message (the `--no-commit` option disables this). If there are merge conflicts (when one ore more lines of a file being merged have different values in the previous state), `git merge` will report them and let you resolve them.

To resolve a conflict, you will have to look in the file being reported as conflict for the following pattern:

```
<<<<<<< HEAD:file.txt
Hello world
=======
Goodbye
>>>>>>> 77976da35a11db4580b80ae27e8d65caf5208086:file.txt
```

which can be explained like this:

```
<<<<<<<
changes made on my branch
=======
changes made on the branch i'm merging
>>>>>>>
```

You will have to erase manually which part are you going to leave. After editing all conflicts, you have to commit your changes:

```
git commit -a
```

Aside from merging, sometimes you want to just pluck one commit out of a different branch. To apply the changes in revision rev and commit them to the current branch use:

```
git cherry-pick [COMMIT]
```

### Working with remote servers ###

If you created your repository with one of the `clone` commands, Git will have already set up a remote repository for you called _origin_. If you created your repository from scratch, you will have to set it up.

To see which remote servers you have configured, you can run the `git remote` command. It lists the shortnames of each remote handle you've specified. If you've cloned your repository, you should at least see _origin_ — that is the default name Git gives to the server you cloned from:

```
git remote -v
```

To add a new remote Git repository as a shortname you can reference easily, run:

```
git remote add [SHORTNAME] [URL]
```

When you cloned your repository, Git downloaded all the branches and tags in that repository, and created your master branch based on the master branch in that repository. Even though it only used the master branch, it kept copies of all the others in case you needed them. Copies of branches from a remote repository are called remote branches, and don't behave quite like the local branches you've used before.

For starters, remote branches don't show up in a normal git branch. Instead, you list remote branches with `git branch -r`. You can log these branches, diff them and merge them, but you can't commit to them, or they would stop being copies of the branch on the remote repository. If you want to work with a remote branch, you need to create a local branch that "tracks" it, like this:

```
git checkout -t origin/branch
```

Now, how do you download new changes from a remote repository? You fetch them with `git fetch`. But usually you don't just want to fetch, you also want to merge any changes into your local branch:

```
git pull
```

A pull is technically a bit different to a rebase. As always, see the relevant manual pages for details.

### Sharing your Work ###

We saw in the previous section that you can pull other people's work into your repository, but how do your changes get back out? Well, your Git repository is as good as any other repository, so you could just ask people to git pull from you the same way you git pulled from them.

This is fine as far as Git's concerned, but what if you have a slow Internet connection, or you are behind a firewall, or you like to amend your commits before letting people see them? Most people get around this by having two repositories: a private repository they work on, and a public repository for people to pull from.

So how do you get your work onto your public repository? Well, it's the opposite of `git pull`, so you `git push`!

When you have your project at a point that you want to share, you have to push it upstream. The command for this is simple:

```
git push [REMOTE-NAME] [BRANCH-NAME]
```

If you want to push your master branch to your origin server, then you can run this to push your work back up to the server:

```
git push origin master
```

This command works only if you cloned from a server to which you have write access and if nobody has pushed in the meantime. If you and someone else clone at the same time and they push upstream and then you push upstream, your push will rightly be rejected. You’ll have to pull down their work first and incorporate it into yours before you’ll be allowed to push.

### Working with submodules ###

Git submodules allows you to attach or include an external repository inside another repository at a specific path. It basically permits to handle various "subprojects" inside one big project. For example, Aguilas has two subprojects inside: the [Google code Wiki](http://code.google.com/p/aguilas/w/list) (`documentation/googlewiki`) and the [GitHub Wiki](https://github.com/HuntingBears/aguilas/wiki) (`documentation/githubwiki`), which both update through the command `make gen-wiki` on the main project.

There are four main functions you will need to understand in order to work with Git submodules. In order, you will need to know how to add, make use of, remove, and update submodules.

#### Adding Submodules to a Git Repository ####

Adding a submodule to a git repository is actually quite simple. For example, let's suppose we want to add support for another (fictionary) wiki: the mediawiki format wiki, on the `documentation/mediawiki` folder. You can do so with the following command:

```
git submodule add git@github.com:mediawiki/wiki.git documentation/mediawiki
```

There are three main parts to this command:

  * **git submodule add**:  This simply tells Git that we are adding a submodule. This syntax will always remain the same.
  * **git@github.com:mediawiki/wiki.git**:  This is the external repository that is to be added as a submodule. The exact syntax will vary depending on the setup of the Git repository you are connecting to. You need to ensure that you have the ability to clone the given repository.
  * **documentation/mediawiki**:  This is the path where the submodule repository will be added to the main repository.


If you make `git status`, you will notice how the supplied path was created and added to the changes to be committed. In addition, a new file called `.gitmodules` was created. This new file contains the details we supplied about the new submodule. Out of curiosity, if you check out the contents of that new file:

```
cat .gitmodules
```

```
[submodule "documentation/mediawiki"]
path = documentation/mediawiki
url = git@github.com:mediawiki/wiki.git
```

Being able to modify this file later will come in handy later.

All that is left to do now is to commit the changes and then push the commit to a remote system if necessary.

#### Using Submodules ####

Having submodules in a repository is great and all, but if you look inside, all you will have is an empty folder rather than the actual contents of the submodule's repository. In order to fill in the submodule's path with the files from the external repository, you must first initialize the submodules and then update them.

First, you need to initialize the submodule(s). You can do that with the following command on the root folder of the main project:

```
git submodule init
```

Then you need to run the update in order to pull down the files:

```
git submodule update
```

Looking in the `documentation/mediawiki` directory now shows a nice listing of the needed files.

#### Removing Submodules ####

What happens if we need to remove a submodule? Maybe I made a mistake. It could also be that the design of the project has changed, and the submodules need to change with it. Unfortunately, Git does not have a built in way to remove submodules. we have to do it manually.

Sticking with the example, we'll remove the `documentation/mediawiki` module from Aguilas. All the instructions will be run from the working directory of the Aguilas repository. In order, we need to do the following:

  * **Remove the submodule's entry in the .gitmodules file**: Open it up in vim, or your favorite text editor, and remove the three lines relevant to the submodule being removed. In this case, these lines will be removed:
```
[submodule "documentation/mediawiki"]
path = documentation/mediawiki
url = git@github.com:mediawiki/wiki.git
```

  * **Remove the submodule's entry in the .git/config file**: Open it up in vim, or your favorite text editor, and remove the two lines relevant to the submodule being removed. In this case, these lines will be removed:
```
[submodule "documentation/mediawiki"]
url = git@github.com:mediawiki/wiki.git
```

  * **Remove the path created for the submodule**: Run the following command to finish removing the submodule:
```
git rm --cached documentation/mediawiki
```



#### Updating Submodules ####

Unfortunately, like removing submodules, Git does not make it clear how to update a submodule to a later commit. Fortunately though, it's not that tough.

Initialize the repository's submodules by running `git submodule init` followed by `git submodule update`:

```
git submodule init
git submodule update
```

Change into the submodule's directory. In this example, `documentation/mediawiki`:

```
cd documentation/mediawiki
```

The submodule repositories added by `git submodule update` are "headless". This means that they aren't on a current branch.To fix this, we simply need to switch to a branch. In this example, that would be the development branch:

```
git checkout development
```

Next, we simply need to update the repository to ensure that we have the latest updates:

```
git pull
```

Now switch back to the root working directory of the repository:

```
cd ../..
```

Everything is now ready to be committed and pushed back in. If you run `git status`, you'll notice that the path to the submodule is listed as modified. This is what you should expect to see. Simply add the path to be committed and do a commit. When you do the commit, the index will update the commit string for the submodule:

```
git add .
git commit -a
```

## Git Cheat Sheet ##

### Setup ###

git clone 

&lt;repo&gt;


> clone the repository specified by 

&lt;repo&gt;

; this is similar to "checkout" in some other version control systems such as Subversion and CVS


Add colors to your ~/.gitconfig file:

[color](color.md)
> ui = auto


["branch"](color.md)
> current = yellow reverse local = yellow remote = green


["diff"](color.md)
> meta = yellow bold frag = magenta bold old = red bold new = green bold


["status"](color.md)
> added = yellow changed = green untracked = cyan


Highlight whitespace in diffs

[color](color.md)
> ui = true


["diff"](color.md)
> whitespace = red reverse


[core](core.md)
> whitespace=fix,-indent-with-non-tab,trailing-space,cr-at-eol


Add aliases to your ~/.gitconfig file:

[alias](alias.md)
> st = status ci = commit br = branch co = checkout df = diff dc = diff --cached lg = log -p lol = log --graph --decorate --pretty=oneline --abbrev-commit lola = log --graph --decorate --pretty=oneline --abbrev-commit --all ls = ls-files

# Show files ignored by git: ign = ls-files -o -i --exclude-standard


### Configuration ###

git config -e [--global]
> edit the .git/config [~/.gitconfig](or.md) file in your $EDITOR


git config --global user.name 'John Doe' git config --global user.email [johndoe@example.com](mailto:johndoe@example.com) sets your name and email for commit messages

git config branch.autosetupmerge true
> tells git-branch and git-checkout to setup new branches so that git-pull(1) will appropriately merge from that remote branch.  Recommended.  Without this, you will have to add --track to your branch command or manually merge remote tracking branches with "fetch" and then "merge".


git config core.autocrlf true
> This setting tells git to convert the newlines to the system's standard when checking out files, and to LF newlines when committing in


git config --list
> To view all options


git config apply.whitespace nowarn
> To ignore whitespace


You can add "--global" after "git config" to any of these commands to make it apply to all git repos (writes to ~/.gitconfig).

### Info ###

git reflog
> Use this to recover from _major_ mess ups! It's basically a log of the last few actions and you might have luck and find old commits that have been lost by doing a complex merge.


git diff
> show a diff of the changes made since your last commit to diff one file: "git diff -- 

&lt;filename&gt;

" to show a diff between staging area and HEAD: `git diff --cached`


git status
> show files added to the staging area, files with changes, and untracked files


git log
> show recent commits, most recent on top. Useful options: --color       with color --graph       with an ASCII-art commit graph on the left --decorate    with branch and tag names on appropriate commits --stat        with stats (files changed, insertions, and deletions) -p            with full diffs --author=foo  only by a certain author --after="MMM DD YYYY" ex. ("Jun 20 2008") only commits after a certain date --before="MMM DD YYYY" only commits that occur before a certain date --merge       only the commits involved in the current merge conflicts


git log 

&lt;ref&gt;

..

&lt;ref&gt;


> show commits between the specified range. Useful for seeing changes from remotes: git log HEAD..origin/master # after git remote update


git show 

&lt;rev&gt;


> show the changeset (diff) of a commit specified by 

&lt;rev&gt;

, which can be any SHA1 commit ID, branch name, or tag (shows the last commit (HEAD) by default) also to show the contents of a file at a specific revision, use git show 

&lt;rev&gt;

:

&lt;filename&gt;

 this is similar to cat-file but much simpler syntax.


git show --name-only 

&lt;rev&gt;


> show only the names of the files that changed, no diff information.


git blame 

&lt;file&gt;


> show who authored each line in 

&lt;file&gt;




git blame 

&lt;file&gt;

 

&lt;rev&gt;


> show who authored each line in 

&lt;file&gt;

 as of 

&lt;rev&gt;

 (allows blame to go back in time)


git gui blame
> really nice GUI interface to git blame


git whatchanged 

&lt;file&gt;


> show only the commits which affected 

&lt;file&gt;

 listing the most recent first E.g. view all changes made to a file on a branch: git whatchanged 

&lt;branch&gt;

 

&lt;file&gt;

  | grep commit | colrm 1 7 | xargs -I % git show % 

&lt;file&gt;

 this could be combined with git remote show 

&lt;remote&gt;

 to find all changes on all branches to a particular file.


git diff 

&lt;commit&gt;

 head path/to/fubar
> show the diff between a file on the current branch and potentially another branch


git diff --cached [

&lt;file&gt;

]
> shows diff for staged (git-add'ed) files (which includes uncommitted git cherry-pick'ed files)


git ls-files
> list all files in the index and under version control.


git ls-remote 

&lt;remote&gt;

 [HEAD](HEAD.md)
> show the current version on the remote repo. This can be used to check whether a local is required by comparing the local head revision.


### Adding / Deleting ###

git add 

&lt;file1&gt;

 

&lt;file2&gt;

 ...
> add 

&lt;file1&gt;

, 

&lt;file2&gt;

, etc... to the project


git add 

&lt;dir&gt;


> add all files under directory 

&lt;dir&gt;

 to the project, including subdirectories


git add .
> add all files under the current directory to the project _WARNING_: including untracked files.


git rm 

&lt;file1&gt;

 

&lt;file2&gt;

 ...
> remove 

&lt;file1&gt;

, 

&lt;file2&gt;

, etc... from the project


git rm $(git ls-files --deleted)
> remove all deleted files from the project


git rm --cached 

&lt;file1&gt;

 

&lt;file2&gt;

 ...
> commits absence of 

&lt;file1&gt;

, 

&lt;file2&gt;

, etc... from the project


### Ignoring ###

Option 1:

Edit $GIT\_DIR/info/exclude. See Environment Variables below for explanation on $GIT\_DIR.

Option 2:

Add a file .gitignore to the root of your project. This file will be checked in.

Either way you need to add patterns to exclude to these files.

### Staging ###

git add 

&lt;file1&gt;

 

&lt;file2&gt;

 ... git stage 

&lt;file1&gt;

 

&lt;file2&gt;

 ... add changes in 

&lt;file1&gt;

, 

&lt;file2&gt;

 ... to the staging area (to be included in the next commit

git add -p git stage --patch interactively walk through the current changes (hunks) in the working tree, and decide which changes to add to the staging area.

git add -i git stage --interactive interactively add files/changes to the staging area. For a simpler mode (no menu), try `git add --patch` (above)

### Unstaging ###

git reset HEAD 

&lt;file1&gt;

 

&lt;file2&gt;

 ...
> remove the specified files from the next commit


### Committing ###

git commit 

&lt;file1&gt;

 

&lt;file2&gt;

 ... [-m 

&lt;msg&gt;

]
> commit 

&lt;file1&gt;

, 

&lt;file2&gt;

, etc..., optionally using commit message 

&lt;msg&gt;

, otherwise opening your editor to let you type a commit message


git commit -a
> commit all files changed since your last commit (does not include new (untracked) files)


git commit -v
> commit verbosely, i.e. includes the diff of the contents being committed in the commit message screen


git commit --amend
> edit the commit message of the most recent commit


git commit --amend 

&lt;file1&gt;

 

&lt;file2&gt;

 ...
> redo previous commit, including changes made to 

&lt;file1&gt;

, 

&lt;file2&gt;

, etc...


### Branching ###

git branch
> list all local branches


git branch -r
> list all remote branches


git branch -a
> list all local and remote branches


git branch 

&lt;branch&gt;


> create a new branch named 

&lt;branch&gt;

, referencing the same point in history as the current branch


git branch 

&lt;branch&gt;

 

&lt;start-point&gt;


> create a new branch named 

&lt;branch&gt;

, referencing 

&lt;start-point&gt;

, which may be specified any way you like, including using a branch name or a tag name


git push 

&lt;repo&gt;

 

&lt;start-point&gt;

:refs/heads/

&lt;branch&gt;


> create a new remote branch named 

&lt;branch&gt;

, referencing 

&lt;start-point&gt;

 on the remote. Repo is the name of the remote. Example: git push origin origin:refs/heads/branch-1 Example: git push origin origin/branch-1:refs/heads/branch-2 Example: git push origin branch-1 ## shortcut


git branch --track 

&lt;branch&gt;

 

&lt;remote-branch&gt;


> create a tracking branch. Will push/pull changes to/from another repository. Example: git branch --track experimental origin/experimental


git branch --set-upstream 

&lt;branch&gt;

 

&lt;remote-branch&gt;

 (As of Git 1.7.0)
> Make an existing branch track a remote branch Example: git branch --set-upstream foo origin/foo


git branch -d 

&lt;branch&gt;


> delete the branch 

&lt;branch&gt;

; if the branch you are deleting points to a commit which is not reachable from the current branch, this command will fail with a warning.


git branch -r -d 

&lt;remote-branch&gt;


> delete a remote-tracking branch. Example: git branch -r -d wycats/master


git branch -D 

&lt;branch&gt;


> even if the branch points to a commit not reachable from the current branch, you may know that that commit is still reachable from some other branch or tag. In that case it is safe to use this command to force git to delete the branch.


git checkout 

&lt;branch&gt;


> make the current branch 

&lt;branch&gt;

, updating the working directory to reflect the version referenced by 

&lt;branch&gt;




git checkout -b 

&lt;new&gt;

 

&lt;start-point&gt;


> create a new branch 

&lt;new&gt;

 referencing 

&lt;start-point&gt;

, and check it out.


git push 

&lt;repository&gt;

 :

&lt;branch&gt;


> removes a branch from a remote repository. Example: git push origin :old\_branch\_to\_be\_deleted


git co 

&lt;branch&gt;

 <path to new file>
> Checkout a file from another branch and add it to this branch. File will still need to be added to the git branch, but it's present. Eg. git co remote\_at\_origintick702\_antifraud\_blocking ..../...nt\_elements\_for\_iframe\_blocked\_page.rb


git show 

&lt;branch&gt;

 -- <path to file that does not exist>
> Eg. git show remote\_tick702 -- path/to/fubar.txt show the contents of a file that was created on another branch and that does not exist on the current branch.


git show 

&lt;rev&gt;

:<repo path to file>
> Show the contents of a file at the specific revision. Note: path has to be absolute within the repo.


### Merging ###

git merge 

&lt;branch&gt;


> merge branch 

&lt;branch&gt;

 into the current branch; this command is idempotent and can be run as many times as needed to keep the current branch up-to-date with changes in 

&lt;branch&gt;




git merge 

&lt;branch&gt;

 --no-commit
> merge branch 

&lt;branch&gt;

 into the current branch, but do not autocommit the result; allows you to make further tweaks


git merge 

&lt;branch&gt;

 -s ours
> merge branch 

&lt;branch&gt;

 into the current branch, but drops any changes in 

&lt;branch&gt;

, using the current tree as the new tree


### Cherry-Picking ###

git cherry-pick [--edit] [-n] [-m parent-number] [-s] [-x] 

&lt;commit&gt;


> selectively merge a single commit from another local branch Example: git cherry-pick 7300a6130d9447e18a931e898b64eefedea19544


### Squashing ###

WARNING: "git rebase" changes history. Be careful. Google it.

git rebase --interactive HEAD~10
> (then change all but the first "pick" to "squash") squash the last 10 commits into one big commit


### Conflicts ###

git mergetool
> work through conflicted files by opening them in your mergetool (opendiff, kdiff3, etc.) and choosing left/right chunks. The merged result is staged for commit.


For binary files or if mergetool won't do, resolve the conflict(s) manually and then do:

git add 

&lt;file1&gt;

 [

&lt;file2&gt;

 ...]

Once all conflicts are resolved and staged, commit the pending merge with:

git commit

### Sharing ###

git fetch 

&lt;remote&gt;


> update the remote-tracking branches for 

&lt;remote&gt;

 (defaults to "origin"). Does not initiate a merge into the current branch (see "git pull" below).


git pull
> fetch changes from the server, and merge them into the current branch. Note: .git/config must have a ["some\_name"](branch.md) section for the current branch, to know which remote-tracking branch to merge into the current branch.  Git 1.5.3 and above adds this automatically.


git push
> update the server with your commits across all branches that are _COMMON_ between your local copy and the server.  Local branches that were never pushed to the server in the first place are not shared.


git push origin 

&lt;branch&gt;


> update the server with your commits made to 

&lt;branch&gt;

 since your last push. This is always _required_ for new branches that you wish to share. After the first explicit push, "git push" by itself is sufficient.


git push origin 

&lt;branch&gt;

:refs/heads/

&lt;branch&gt;


> E.g. git push origin twitter-experiment:refs/heads/twitter-experiment Which, in fact, is the same as git push origin 

&lt;branch&gt;

 but a little more obvious what is happening.


### Reverting ###

git revert 

&lt;rev&gt;


> reverse commit specified by 

&lt;rev&gt;

 and commit the result.  This does _not_ do the same thing as similarly named commands in other VCS's such as "svn revert" or "bzr revert", see below


git checkout 

&lt;file&gt;


> re-checkout 

&lt;file&gt;

, overwriting any local changes


git checkout .
> re-checkout all files, overwriting any local changes.  This is most similar to "svn revert" if you're used to Subversion commands


### Fix mistakes / Undo ###

git reset --hard
> abandon everything since your last commit; this command can be DANGEROUS. If merging has resulted in conflicts and you'd like to just forget about the merge, this command will do that.


git reset --hard ORIG\_HEAD or git reset --hard origin/master
> undo your most recent _successful_ merge _and_ any changes that occurred after.  Useful for forgetting about the merge you just did.  If there are conflicts (the merge was not successful), use "git reset --hard" (above) instead.


git reset --soft HEAD^
> forgot something in your last commit? That's easy to fix. Undo your last commit, but keep the changes in the staging area for editing.


git commit --amend
> redo previous commit, including changes you've staged in the meantime. Also used to edit commit message of previous commit.


### Plumbing ###

test 

&lt;sha1-A&gt;

 = $(git merge-base 

&lt;sha1-A&gt;

 

&lt;sha1-B&gt;

)
> determine if merging sha1-B into sha1-A is achievable as a fast forward; non-zero exit status is false.


### Stashing ###

git stash git stash save 

&lt;optional-name&gt;

 save your local modifications to a new stash (so you can for example "git svn rebase" or "git pull")

git stash apply
> restore the changes recorded in the stash on top of the current working tree state


git stash pop
> restore the changes from the most recent stash, and remove it from the stack of stashed changes


git stash list
> list all current stashes


git stash show 

&lt;stash-name&gt;

 -p
> show the contents of a stash - accepts all diff args


git stash drop [

&lt;stash-name&gt;

]
> delete the stash


git stash clear
> delete all current stashes


### Remotes ###

git remote add 

&lt;remote&gt;

 

<remote\_URL>


> adds a remote repository to your git config.  Can be then fetched locally. Example: git remote add coreteam git://github.com/wycats/merb-plugins.git git fetch coreteam


git push 

&lt;remote&gt;

 :refs/heads/

&lt;branch&gt;


> delete a branch in a remote repository


git push 

&lt;remote&gt;

 

&lt;remote&gt;

:refs/heads/

<remote\_branch>


> create a branch on a remote repository Example: git push origin origin:refs/heads/new\_feature\_name


git push 

&lt;repository&gt;

 +

&lt;remote&gt;

:

<new\_remote>


> replace a 

&lt;remote&gt;

 branch with 

<new\_remote>

 think twice before do this Example: git push origin +master:my\_branch


git remote prune 

&lt;remote&gt;


> prune deleted remote-tracking branches from "git branch -r" listing


git remote add -t master -m master origin git://example.com/git.git/
> add a remote and track its master


git remote show 

&lt;remote&gt;


> show information about the remote server.


git checkout -b <local branch> 

&lt;remote&gt;

/<remote branch>
> Eg git checkout -b myfeature origin/myfeature Track a remote branch as a local branch.


git pull 

&lt;remote&gt;

 

&lt;branch&gt;

 git push For branches that are remotely tracked (via git push) but that complain about non-fast forward commits when doing a git push. The pull synchronizes local and remote, and if all goes well, the result is pushable.

git fetch 

&lt;remote&gt;


> Retrieves all branches from the remote repository. After this 'git branch --track ...' can be used to track a branch from the new remote.


### Submodules ###

git submodule add 

<remote\_repository>

 <path/to/submodule>
> add the given repository at the given path. The addition will be part of the next commit.


git submodule update [--init]
> Update the registered submodules (clone missing submodules, and checkout the commit specified by the super-repo). --init is needed the first time.


git submodule foreach 

&lt;command&gt;


> Executes the given command within each checked out submodule.


Removing submodules

  1. Delete the relevant line from the .gitmodules file.
  1. Delete the relevant section from .git/config.
  1. Run git rm --cached path\_to\_submodule (no trailing slash).
  1. Commit and delete the now untracked submodule files.


Updating submodules
> To update a submodule to a new commit:
> update submodule:
    1. cd <path to submodule> git pull


commit the new version of submodule:
  1. cd <path to toplevel> git commit -m "update submodule version"


check that the submodule has the correct version
  1. git submodule status





If the update in the submodule is not committed in the main repository, it is lost and doing git submodule update will revert to the previous version.


### Patches ###

git format-patch HEAD^
> Generate the last commit as a patch that can be applied on another clone (or branch) using 'git am'. Format patch can also generate a patch for all commits using 'git format-patch HEAD^ HEAD' All page files will be enumerated with a prefix, e.g. 0001 is the first patch.


git format-patch 

&lt;Revision&gt;

^..

&lt;Revision&gt;


> Generate a patch for a single commit. E.g. git format-patch d8efce43099^..d8efce43099 Revision does not need to be fully specified.


git am <patch file>
> Applies the patch file generated by format-patch.


git diff --no-prefix > patchfile
> Generates a patch file that can be applied using patch: patch -p0 < patchfile Useful for sharing changes without generating a git commit.


### Tags ###

git tag -l
> Will list all tags defined in the repository.


git co 

<tag\_name>


> Will checkout the code for a particular tag. After this you'll probably want to do: 'git co -b <some branch name>' to define a branch. Any changes you now make can be committed to that branch and later merged.


### Archive ###

git archive master | tar -x -C /somewhere/else
> Will export expanded tree as tar archive at given path


git archive master | bzip2 > source-tree.tar.bz2
> Will export archive as bz2


git archive --format zip --output /full/path master
> Will export as zip


### Git Instaweb ###

git instaweb --httpd=webrick [--start | --stop | --restart]

### Environment Variables ###

GIT\_AUTHOR\_NAME, GIT\_COMMITTER\_NAME
> Your full name to be recorded in any newly created commits.  Overrides user.name in .git/config


GIT\_AUTHOR\_EMAIL, GIT\_COMMITTER\_EMAIL
> Your email address to be recorded in any newly created commits.  Overrides user.email in .git/config


GIT\_DIR
> Location of the repository to use (for out of working directory repositories)


GIT\_WORKING\_TREE
> Location of the Working Directory - use with GIT\_DIR to specifiy the working directory root or to work without being in the working directory at all.
