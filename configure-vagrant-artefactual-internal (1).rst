.. _configure-vm-artefactual:

================================================
Configuring your Vagrant box for Artefactual use
================================================

.. _PuTTY: http://www.chiark.greenend.org.uk/~sgtatham/putty/
.. _GitHub: https://github.com/artefactual/atom/
.. _User Forum: https://groups.google.com/forum/#!forum/ica-atom-users

This article will help you set up your local testing environment for use at
Artefactual. First, we'll follow the instructions in the AtoM documentation to
set up a local Vagrant box for testing and development. The following
instructions will help you make a few necessary changes to begin using this
internally.

.. _install-atom-vagrant:

Install the AtoM Vagrant box
============================

Follow the instructions in the AtoM documentation to install and configure the
AtoM Vagrant box. They can be found at:

* https://www.accesstomemory.org/docs/latest/dev-manual/env/vagrant/

If you prefer something visual, you can also check out the first 3 videos of
the AtoM CLI tutorial series, available on YouTube. The videos were made using
the AtoM 2.3 vagrant box, but the general principles will apply regardless of
the version.

* https://www.youtube.com/playlist?list=PLZiwlG5eSMeyeETe15EsEBSu5htPLK-wm

When everything is installed, boot up the VM!

.. _vm-ssh-key:

Add your SSH key
================

By default, the AtoM Vagrant box points to our public `GitHub`_ repository.
However, this is in fact merely a public mirror of our internal gitolite
repository. If you try to commit changes directly to GitHub, they are not
added to the repository of record (gitolite) - meaning they will be lost the
next time someone pushes changes internally, and will only exist on GitHub.

.. IMPORTANT::

   If you push changes to `GitHub`_ by accident, talk to one of the
   developers; they can help cherry-pick your changes and apply them to the
   internal gitolite repo.

Accessing the internal gitolite repository requires SSH access - which means
you'll need an SSH key

SEVEIN ADD SOME INSTRUCTIONS HERE

Keep a password-protected copy of your key somewhere secure on your local
computer. Make sure the password is unique and secure! If you want, you can
store a note in LastPass with the password - do not share this with other
staff.

It's also possible to ask for a copy of the key to be kept on tamias, one of
the Artefactual servers, as a back-up. **This should only be done if your SSH
key is password protected**.

.. _vm-ssh-key-windows:

Windows users - using Pageant for SSH forwarding
------------------------------------------------

If you're a Windows user, you might already be using `PuTTY`_ as a remote
terminal emulator. Pageant is an SSH authentication agent designed for use
with PuTTY. It holds your private keys in memory, already decoded, so that you
can use them often without needing to type a passphrase. You can download it
as part of a package with PuTTY, or as a separate download on the same page:

* http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html

**Pageant documentation:**

* http://the.earth.li/~sgtatham/putty/0.67/htmldoc/Chapter9.html#pageant


[DO I NEED TO ADD MORE HERE???]

Once you have Pageant configured, let's fire up our Vagrant box, and open our
terminal. Then we'll begin making a few changes to the default configuration
of the public Vagrant box. Make sure you're running Pageant with your SSH key
loaded as well for the next steps.

.. TIP::

   Once you have a password-protected key stored locally for use with Vagrant
   via Pageant, you can re-use the key for multiple boxes. Your SSH key will
   remain the same when accessing Artefactual, so the same key will work.

.. _vm-set-remote-origin:

Set the remote origin
=====================

As mentioned above, the public Vagrant box is set to track the GitHub
repository by default - but we want to be pushing any changes we make to the
internal gitolite server. First, let's change the remote. If you're not
already in the ``atom`` directory, change there - then run:

.. code-block:: bash

   git remote set-url origin git@git.artefactual.com:atom.git

Remember, you'll want to make the same changes for ``atom-docs`` as well:

.. code-block:: bash

   cd ~/atom-docs
   git remote set-url origin git@git.artefactual.com:atom-docs.git
   cd ~/atom

Now you're tracking against the internal gitolite repository! You can always
check to see if this has been successful  with the following command:

.. code-block:: bash

   git remote -v

.. _vm-add-branches:

Enable access to all development branches (2.3 box only)
========================================================

The 2.3 AtoM Vagrant box did not allow the user to see and check out all
available branches - so to be able to checkout other branches for testing,
we'll first have to remove this. **This should be fixed** in the next Vagrant
box, but in case it's needed again in the future, here's how you can solve it.
From the root AtoM directory, enter:

.. code-block:: bash

   git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"
   git fetch --unshallow

You will be asked to confirm the authenticity of host ``git.artefactual.com``
the first time you do this. Enter "yes" to continue.

Now we should be able to see all the available development branches. If you
want to double-check, or get a full list of available dev branches, try:

.. code-block:: bash

   git branch -avv

.. _vm-config-git-user:

Configure your global git identity
==================================

Before you can push any changes, you need to configure git. You should have
already created an account on `GitHub`_ already - if not, do so now (there
should be a login / sign up option in the top right corner of the page.

Generally, The first thing you should do when you install Git is to set your
user name and email address. This is important because every Git commit uses
this information, and it’s immutably baked into the commits you start
creating. We can configure this globally in our Vagrant environment with the
following commands:

.. code-block:: bash

   git config --global user.name "your-username"
   git config --global user.email your-email@example.com

You need to do this only once if you pass the ``--global`` option, because
then Git will always use that information for anything you do on that system.
If you want to override this with a different name or email address for
specific projects, you can run the command without the ``--global`` option
when you’re in that project.

There are other things you can configure if you want, such as your default
text editor and other preferences. For more information, see:

* https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup

.. _vm-checkout-pull:

Check out the branch you want and pull in the most recent changes
=================================================================

We're almost ready! Now you can check out whatever branch you want to work on.
Remember, ``git branch`` will tell you which branch you're currently on, and
``git branch --avv`` will list all available remote branches (it's actually a
combination of the ``-a`` option to list all local and remote-tracking
branches, and the ``-vv`` verbose option to show the SHA1 and commit sublect
line for each head).

Don't forget to pull in the latest changes and rebase when switching branches:

.. code-block:: bash

   git checkout qa/2.4.x
   git pull --rebase

Remember, if you are making a major branch change, you should probably also
clear the cache, restart services, and do a ``tools:purge`` to ensure the
database schema is in the right state. The purge command will flush any data
you have - we're going to use the ``--demo`` option with it, which will skip
confirmation, and create a demo@example.com admin account in AtoM for you.

.. code-block:: bash

   php symfony cc
   sudo service php5-fpm restart
   sudo /etc/init.d/memcached restart
   php symfony tools:purge --demo
   php symfony search:populate

You should be ready to go!

.. _vm-troubleshooting:

Troubleshooting tips
====================

SEVEIN ANY USEFUL TIPS TO ADD HERE??

.. _vm-troubleshooting-permissions:

Filesystem permissions in the Vagrant box
-----------------------------------------

If for some reason, the filesystem permissions get screwed up in your local
VM, you might be tempted to try the suggestions we mention in the AtoM upgrade
documentation, or often in the AtoM `User Forum`_, to reset the permissions.
That command, **which you should not use in the Vagrant box**, looks like
this:

.. code-block:: bash

   sudo chown -R www-data:www-data /usr/share/nginx/atom

In a regular AtoM deployment, the PHP-FPM server runs as the ``www-data`` user
- hence the AtoM directory and its contents have to be owned by the same user.
Our Vagrant box is a bit different in that the PHP-FPM server runs as the
``vagrant``user and so the AtoM directory and its contents must be owned by
the ``vagrant`` user.

So, if you run into filesystem permissions issues and intend to try to resolve
them by reapplying the permissions, use the following command:

.. code-block:: bash

   sudo chown -R vagrant.vagrant /usr/share/nginx/atom

.. _vm-troubleshooting-vagrant-version:

Installing multiple Vagrant boxes
---------------------------------

You can run into problems downloading a newer version of the Vagrant box if
you've already or previously installed an older version. For example, let's
say you've got one box running stable/2.3.x, and you want to Artefactualize a
second box for qa/2.4.x testing, so you can have 2 separate environments and
don't have to flush your data every time, or manage different dependencies. In
July, Jesus created a new 2.4 version of the box - so for first-time users,
this is what *should* download.

However, if you've previously installed an earlier Vagrant box, you might find
that in creating a second box, you've ended up with another version of the
earlier box instead of the expected updated version. You could just change
branches - but between 2.3 and 2.4 for example, the dependencies have changed,
so it will always be more reliable to use the correct box - otherwise you
might end up having to manually update some of the dependencies in your box,
or face errors. Below is the best way to fix that. We'll use 2.4 as the
example box you are trying to install.

1. In the host system's command line (e.g. windows, mac, linux, etc),
   navigate to the directory where you intended to install the 2.4 box.
2. Run ``vagrant destroy``
3. Use your filebrowser (e.g. Windows Explorer, Mac Finder, etc) to open the
   2.4 Vagrant directory, and open the Vagrantfile with a text editor
4. You'll see that most of the contents are commented out, but there will be
   one line mid-way down with the following:

   .. code-block:: none

      config.vm.box = "artefactual/atom"

   Underneath it, add the following line:

   .. code-block:: none

      config.vm.box_version = "2.4.0.0"

5. Save the Vagrant file. From the terminal, you can now run ``vagrant up``
   again. Note, you'll have to repeat the steps above if you'd already done
   them, since you've destroyed the previous version of the box.

.. _vm-aliases:

Optional: add aliases for common tasks
======================================

It is possible to add aliases for commonly used commands to your terminal,
which act as a kind of short-cut to typing out longer commands. This is **not
recommended** for a production instance, but can be useful in the development
environment for testing, where we are often having to purge and rebuild our
environments.

Aliases can be added to the ``.bashrc`` file in your root directory. the dot
before the filename means it is a hidden file - you can see all files in a
directory including hidden ones with ``ls -a``

.. code-block:: bash

   cd
   ls -a

You can open the file with nano:

.. code-block:: bash

   nano .bashrc

You'll see other default aliases used in bash in there, where you can
get a sense of the syntax. Essentially, the syntax to add an alias is:
``alias XXX="[command]"``. Scroll to the very bottom of the file and add your
aliases there, to keep them separate and easy to find. If you do make
yourself some aliases, **it is critical they are unique** so they will not
interfere with existing commands! I often add a capital S before mine as
this does not seem to be used in existing commands. Some useful examples:

.. code-block:: bash

   alias Supgrade="php symfony tools:upgrade-sql"
   alias Spurge="php symfony tools:purge --demo"
   alias Srestart="sudo service php5-fpm restart && sudo /etc/init.d/memcached restart && sudo service nginx restart && sudo restart atom-worker"
   alias Scc="php symfony cc"
   alias Spop="php symfony search:populate"
   alias Smake="make -C plugins/arDominionPlugin"

If using bash, use ``CTRL+X`` to exit - you'll be asked to save your changes.
The first time you do this, you might need to reload the ``.bashrc`` file to
make the aliases available. You can do this by entering:

.. code-block:: bash

   . /home/vagrant/.bashrc



:ref:`Back to top <configure-vm-artefactual>`
