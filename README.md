# Instant Rails Stack

## What?

Using [Vagrant](http://vagrantup.com) and a Debian Lenny machine preconfigured
with the components to run Rails applications, the instant rails stack provides
a Rake task that helps get applications up and running with only one command.

## How?

The instant rails stack rake task uses Vagrant to load up the Rails machine. It
then looks for applications in apps/ and generates an Apache VirtualHost for each.
It then symlinks these files into /etc/apache2/sites-enabled and restarts apache
so that they take effect. Finally, it adds host aliases using the Ghost gem so you
can access your applications via urls like http://your_application:8080

## Install

    [sudo] gem install vagrant ghost
    vagrant box add debian_lenny_rails_platform http://file.vagrantup.com/contrib/debian_lenny_rails_platform.box
    git clone git://github.com/KieranP/Instant-Rails-Stack.git
    cd Instant-Rails-Stack
    rails apps/a_quick_test
    rake
    http://a_quick_test:8080

## Other features

* Will install gems using 'bundle install' if a Gemfile is present
* Will install gems using 'rake gems:install' if using Rails 2
* Will install stack requirements if a file called stack_requirements.yml exists in the app root

# Stack Requirements

If you need a system package for the application to run (perhaps a gem you use needs one),
then put make a stack_requirements.yml in your project root containing a YAML array of package names
that you can install via apt-get. Example:

    # this file has requirements for Nokogiri
    - libxml2-dev
    - libxml2
    - libxslt1-dev

## Credits

Rake task written by Kieran Pilkington.
