# From 7.3 to 7.4

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
git checkout -- db/schema.rb # local changes will be restored automatically
```

```bash
git checkout 7-4-stable
```

### 3. Install libs, migrations, etc.

```bash
cd /home/[user]/gitlab

# MySQL installations (note: the line below states '--without ... postgres')
bundle install --without development test postgres --deployment

# Run database migrations
bundle exec rake db:migrate RAILS_ENV=production

# Clean up assets and cache
bundle exec rake assets:clean assets:precompile cache:clear RAILS_ENV=production
```

### 5. Update config files

#### New configuration options for gitlab.yml

There are new configuration options available for gitlab.yml. View them with the command below and apply them to your current gitlab.yml.

```
git diff origin/7-3-stable:config/gitlab.yml.example origin/7-4-stable:config/gitlab.yml.example
```

#### Change timeout for unicorn

```
# config/unicorn.rb
timeout 60
```

#### Update database.yml config file(for mysql only) if needed (basically it is required for old gitlab installations)

* Add `collation: utf8_general_ci` to config/database.yml as seen in [config/database.yml.mysql](config/database.yml.mysql)


### 6. Start application

```
svc -u ~/service/sidekiq/
svc -u ~/service/gitlab/
```

### 7. Check application status

Check if GitLab and its environment are configured correctly:

    bundle exec rake gitlab:env:info RAILS_ENV=production

To make sure you didn't miss anything run a more thorough check with:

    bundle exec rake gitlab:check RAILS_ENV=production

If all items are green, then congratulations upgrade is complete!

## Things went south? Revert to previous version (7.3)

### 1. Revert the code to the previous version
Follow the [upgrade guide from 7.2 to 7.3](7.2-to-7.3.md), except for the database migration
(The backup is already migrated to the previous version)

### 2. Restore from the backup:

```bash
cd /home/[user]/gitlab
bundle exec rake gitlab:backup:restore RAILS_ENV=production
```
If you have more than one backup *.tar file(s) please add `BACKUP=timestamp_of_backup` to the command above.
