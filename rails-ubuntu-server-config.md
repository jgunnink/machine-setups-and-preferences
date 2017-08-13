# Setup requirements for Ubuntu server for remote hosting rails

## During setup, select:

- Open SSH
- Postegres

## Log in

- `sudo -i` to act as root
- `apt-get update && apt-get upgrade` to update system
- `reboot now`

## Setup ssh access & disable password auth

1. `ssh-copy-id jk@serverIP-or-hostname` 
1. `sudo nano /etc/ssh/sshd_config`
1. Ensure the following are set:
	```
	PasswordAuthentication no
	PubkeyAuthentication yes
	ChallengeResponseAuthentication no
	```
1. Save and exit
1. Restart SSH daemon `sudo systemctl reload sshd`

## Firewalling (basic)

```
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow OpenSSH
sudo ufw enable
```

## Nginix setup
```
sudo apt-get install nginx
sudo ufw allow 'Nginx HTTP'
systemctl status nginx
```

## Setting up the Rails environment

### Installing ruby

1. Install the dependencies required for rbenv and Ruby: `sudo apt-get install autoconf bison build-essential libssl-dev libyaml-dev libreadline6-dev zlib1g-dev libncurses5-dev libffi-dev libgdbm3 libgdbm-dev`
1. Install rbenv: 
	```bash
	git clone https://github.com/rbenv/rbenv.git ~/.rbenv
	echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
	echo 'eval "$(rbenv init -)"' >> ~/.bashrc
	```
1. Reload bash source `source ~/.bashrc`
1. Make sure it's setup properly using type: `type rbenv`
1. Install ruby build plugin `git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build`
1. Install ruby with required version, eg: `rbenv install 2.3.4`.
You can list available versions by typing: `rbenv install -l`
1. Set global version to use: `rbenv global 2.3.4`
1. Check ruby was installed and is selected by typing `ruby -v`

### Installing rails

1. `gem install rails` for most recent or specify a version: `gem install rails -v 4.2.7`
1. Verify the correct version of rails is installed `rails -v`

### Install Javascript runtime - optional

```
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### Setup Postgres

1. Create postgres user that rails will use to access DB: `sudo -u postgres createuser -s pgprod`
1. Setup a password `sudo -u postgres psql` to enter the postgres prompt
1. `\password pgprod` Enter a password at the prompt and again to confirm
1. Update permissions (still at postgres prompt):
	```SQL
	ALTER USER pgprod CREATEDB;
	ALTER USER pgprod WITH SUPERUSER;
	```
1. Close postgres prompt with `\q`

## nginx setup
Assuming you're just running one app on this server, an example config might be:

1. `sudo nano /etc/nginx/sites-available/default`
	```
	upstream app {
		server localhost:3000;
	}

	server {
		listen 80;
		server_name APPNAME;

		root /railsapp/public;

		try_files $uri/index.html $uri @app;

		location @app {
			proxy_pass http://app;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_set_header Host $http_host;
			proxy_redirect off;
		}

		error_page 500 502 503 504 /500.html;
		client_max_body_size 4G;
		keepalive_timeout 10;
	}
	```
1. `sudo systemctl restart nginx`

## Prepare Rails for production startup

1. `rake db:setup RAILS_ENV=production`
1. Generate production secret:
`RAILS_ENV=production rake secret`
2. Copy output string and insert to the same profile the server will be started on. Eg: `nano ~/.bashrc`
Add to file: `export SECRET_KEY_BASE=(aboveoutput)

Note: The above is not an ideal permanent solution. It will just get you going. Ideally there's a system service which is always running which runs a script at startup that has environment vars in it, rather than just adding it to the users' profile.

## Start

Finally, start the application in production:
`rails s -e production --binding=0.0.0.0`
