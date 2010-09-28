Chef-Solo Capistrano Bootstrap
============
by Mike Smullin <mike@smullindesign.com>

**Utilize Capistrano to automatically bootstrap any remote server for Chef-Solo via SSH using a single command.**

Installation
------------

  - Git clone the blank Opscode Chef Cookbook Repository to your local machine

    `git clone http://github.com/opscode/chef-repo`

  - Reset the git history

    `rm -rf .git && git init .`

  - Make a special directory for containing all your dna.json files

    `mkdir ./dna/ && touch ./dna/example-node.json`

  - Install Capistrano gem on the local machine either as a system gem or using bundler

    `gem install capistrano`

  - Git clone or wget the Capfile into your cookbook repo root dir

    `wget http://github.com/mikesmullin/Chef-Solo-Capistrano-Bootstrap/raw/master/Capfile`

  - Write your cookbook recipes and roles

Usage
------------

  - Execute the Capistrano Chef Bootstrap task

    `cap chef:bootstrap <dna> <remote_host>`

  - Enjoy!


Example
------------

    cap chef:bootstrap <dna> <remote_host>

There are other commands you can find by `cap -vT`.

For eaxmple, during development, it is handy to use:

     cap chef:resolo <dna> <remote_host>

Which will push out your latest cookbook plus the dna.json and execute chef-solo on it.

When you are done, you can remove all traces of Chef with:

     cap chef:clean <dna> <remote_host>

Or if you want to nuke and pave to be sure old recipes are not causing problems:

     cap chef:clean_solo <dna> <remote_host>

If you are like me you enjoy provisioning a new Rackspace VPS and issuing this command
to automatically create my users, groups, and copy ssh keys over for a passwordless ssh session
going forward:

     cap chef:init_server <new_server_remote_ip>


Future goals
------------

Is it possible that with the power of Ruby Expect, either Capistrano or Chef could be made better
so that only one or the other is ultimately necessary?

Credits
------------

Opscode Chef is originally by Opscode, Inc. see http://www.opscode.com/chef/

Capistrano is originally by Jamis Buck <jamis@37signals.com> see http://www.capify.org/
