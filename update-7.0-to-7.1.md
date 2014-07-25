# From 7.0 to 7.1

### 0. Backup

```bash
cd /home/git/gitlab
bundle exec rake gitlab:backup:create RAILS_ENV=production
```

### 1. Stop server

```bash
svc -d ~/service/gitlab
svc -d ~/service/sidekiq
```

### 2. Update Ruby
Don't. Unless you know what you're doing.
<!-- Update to ruby 2.1.2. See [uberspace documentation](https://wiki.uberspace.de/development:ruby)
If you're like me and updated ruby before backup, use `gem update bundler` to fix `There was an error in your Gemfile, and Bundler cannot continue.` errors.  
After this, run `bundle install` to update gems.
 -->
### 3. Get latest code

```bash
cd /home/git/gitlab
git fetch --all
```
```bash
git checkout 7-1-stable
```

### 4. Update gitlab-shell (and its config)

```bash
cd /home/git/gitlab-shell
git fetch
git checkout v1.9.6
```

### 5. Install libs, migrations, etc.

```bash
cd /home/git/gitlab

# MySQL installations (note: the line below states '--without ... postgres')
bundle install --without development test postgres --deployment

# Run database migrations
bundle exec rake db:migrate RAILS_ENV=production

# Clean up assets and cache
bundle exec rake assets:clean assets:precompile cache:clear RAILS_ENV=production

# Update init.d script @todo
sudo cp lib/support/init.d/gitlab /etc/init.d/gitlab
sudo chmod +x /etc/init.d/gitlab
```

### 6. Update config files

#### New configuration options for gitlab.yml

There are new configuration options available for gitlab.yml. View them with the command below and apply them to your current gitlab.yml if desired.

```
git diff 7-0-stable:config/gitlab.yml.example 7-1-stable:config/gitlab.yml.example
```

### 7. Start application @todo

    svc -u ~/service/sidekiq/
    svc -u ~/service/gitlab/

### 8. Check application status

Check if GitLab and its environment are configured correctly:

    bundle exec rake gitlab:env:info RAILS_ENV=production

To make sure you didn't miss anything run a more thorough check with:

    bundle exec rake gitlab:check RAILS_ENV=production

If all items are green, then congratulations upgrade is complete!

## Things went south? Revert to previous version (7.0)

### 1. Revert the code to the previous version
Follow the [`upgrade guide from 6.9 to 7.0`](6.9-to-7.0.md), except for the database migration 
(The backup is already migrated to the previous version)

### 2. Restore from the backup:

```bash
cd /home/git/gitlab
sudo -u git -H bundle exec rake gitlab:backup:restore RAILS_ENV=production
```
If you have more than one backup *.tar file(s) please add `BACKUP=timestamp_of_backup` to the command above.