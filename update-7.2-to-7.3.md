# From 7.2 to 7.3

### 0. Backup

```bash
cd /home/[user]/gitlab
bundle exec rake gitlab:backup:create RAILS_ENV=production
```

### 1. Stop server

[assuming you use my scripts](https://blog.kanedo.net/1925,gitlab-7-0-auf-einem-uberspace-installieren.html)
```bash
svc -d ~/service/sidekiq/
svc -d ~/service/gitlab/
```

### 2. Get latest code

```bash
cd /home/[user]/gitlab
git fetch --all
```

For GitLab Community Edition:

```bash
git checkout 7-3-stable
```

### 3. Update gitlab-shell

```bash
cd /home/[user]/gitlab-shell
git fetch
git checkout v2.0.0
```

### 4. Install libs, migrations, etc.

```bash
cd /home/[user]/gitlab

# MySQL installations (note: the line below states '--without ... postgres')
bundle install --without development test postgres --deployment


# Run database migrations
bundle exec rake db:migrate RAILS_ENV=production

# Clean up assets and cache
bundle exec rake assets:clean assets:precompile cache:clear RAILS_ENV=production REDIS_URL=unix:/home/[user]/.redis/sock

# Update init.d script
# sudo cp lib/support/init.d/gitlab /etc/init.d/gitlab
```


### 5. Configure Redis to use sockets

YAY we're already using sockets because it's the only way to use redis on uberspace. Nothing to do here

### 6. Update config files

#### New configuration options for gitlab.yml

There are new configuration options available for gitlab.yml. View them with the command below and apply them to your current gitlab.yml.

```
git diff origin/7-2-stable:config/gitlab.yml.example origin/7-3-stable:config/gitlab.yml.example
```

```
# Use the default Unicorn socket backlog value of 1024
sed -i 's|^:backlog => 64|:backlog => 1024|' config/unicorn.rb
```

### 7. Start application

	svc -u ~/service/sidekiq/
	svc -u ~/service/gitlab/

### 8. Check application status

Check if GitLab and its environment are configured correctly:

	bundle exec rake gitlab:env:info RAILS_ENV=production

To make sure you didn't miss anything run a more thorough check with:

	bundle exec rake gitlab:check RAILS_ENV=production

If all items are green, then congratulations upgrade is complete!

### 9. Update OmniAuth configuration

When using Google omniauth login, changes of the Google account required.
Ensure that `Contacts API` and the `Google+ API` are enabled in the [Google Developers Console](https://console.developers.google.com/).
More details can be found at the [integration documentation](../integration/google.md).

## Things went south? Revert to previous version (7.2)

### 1. Revert the code to the previous version
Follow the [upgrade guide from 7.1 to 7.2](7.1-to-7.2.md), except for the database migration
(The backup is already migrated to the previous version)

### 2. Restore from the backup:

```bash
cd /home/git/gitlab
sudo -u git -H bundle exec rake gitlab:backup:restore RAILS_ENV=production
```
If you have more than one backup *.tar file(s) please add `BACKUP=timestamp_of_backup` to the command above.
