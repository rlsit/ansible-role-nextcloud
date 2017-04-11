# Ansible role to install and maintain Nextcloud setups

The purpose is to make installing and upgrading Nextcloud instances
straightforward.

# How it works

* The role installs the requested Nextcloud version into
  `{{ nextcloud_work_dir }}/nextcloud-{{ nextcloud_version }}`.
* A maintenance page (to be displayed during upgrades) is installed
  to `{{ nextcloud_work_dir }}/nextcloud-maintenance`.
* A symlink `{{ nextcloud_work_dir }}/nextcloud-current` is installed that
  points to the current and active Nextcloud installation (or to the
  maintenance page during the upgrade).

# Preliminaries

The role takes care of installing and upgrading Nextcloud and its apps. It
doesn't install or configure MariaDB/MySQL, mail or web service. This is
done better in separate roles.

On the other hand, PHP dependencies particularly required for Nextcloud are
installed on the target system.

Some modules used in this role are new to Ansible 2.2 and it's tested with
Ansible 2.2 only.

In other words, the following preliminaries need to be met:

* Ansible 2.2 or newer
* A MariaDB/MySQL server with admin permissions (may be remote)
* A webserver (e.g. Apache2/Nginx) with basic PHP support (needs to be local
  to the target system)
* A mail server if you want the Nextcloud instance to send out mails (may be
  remote)
* The Nextcloud `data` directory needs to be located outside
  `nextcloud_work_dir`

# Usage example

* Integrate the role in your playbook:  
    
  ```
- hosts: cloud.example.org
  roles:
    - nextcloud
  tags:
    - nextcloud
```

* Configure role variables in `host_vars` for the target system (see
  `defaults/main.yml` for a list of supported config options):  
    
  ```
# Nextcloud settings

nextcloud_version: 11.0.2

nextcloud_apps:
  admin_audit: ""
  calendar: "https://github.com/nextcloud/calendar/releases/download/v1.5.2/calendar.tar.gz"
  contacts: "https://github.com/nextcloud/contacts/releases/download/v1.5.3/contacts.tar.gz"
nextcloud_src_type: tar
nextcloud_workdir: "/var/www/cloud.example.org/nextcloud"
nextcloud_data_dir: "/srv/nextcloud/data"
nextcloud_trusted_domains:
  - cloud.example.org
nextcloud_mysql_password: "******"
nextcloud_admin_password: "******"
```

* Deploy Nextcloud to the target system:  
    
  `ansible-playbook site.yml -t nextcloud -l cloud.example.org --diff`

* Configure the VirtualHost of your webserver. The following is an example snippet for
  a jinja2 template:  
    
  ```
{% set nextcloud_webroot = nextcloud_webroot|d(nextcloud_workdir|d() + '/nextcloud-current') %}
{% if nextcloud_www_alias|d(true) %}
Alias /{{ nextcloud_www_alias_name|d('nextcloud') }} {{ nextcloud_webroot }}
{% endif %}
	<Directory {{ nextcloud_webroot }}>
		Options FollowSymlinks
		AllowOverride All

		<IfModule mod_dav.c>
			Dav off
		</IfModule>

		SetEnv HOME {{ nextcloud_webroot }}
		SetEnv HTTP_HOME {{ nextcloud_webroot }}
	</Directory>
```

# License

This Ansible role is licensed under the GNU GPLv2 or later.

# Author

Copyright 2017 Jonas Meurer <jonas@freesources.org>
