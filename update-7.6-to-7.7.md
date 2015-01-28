# From 7.6 to 7.7

### 0. Stop server

    svc -d ~/service/gitlab 
    svc -d ~/service/sidekiq

### 1. Backup

```bash
cd /home/[user]/gitlab
bundle exec rake gitlab:backup:create RAILS_ENV=production
```

### 2. Get latest code

```bash
git fetch --all
git checkout -- db/schema.rb # local changes will be restored automatically
```

For GitLab Community Edition:

```bash
git checkout 7-7-stable
```

### 3. Update gitlab-shell

```bash
cd /home/[user]/gitlab-shell
git fetch
git checkout v2.4.1
```

### 4. Install libs, migrations, etc.

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

#### New configuration options for `gitlab.yml`

There are new configuration options available for [`gitlab.yml`](config/gitlab.yml.example). View them with the command below and apply them to your current `gitlab.yml`.

```
git diff origin/7-6-stable:config/gitlab.yml.example origin/7-7-stable:config/gitlab.yml.example
```

#### Change Nginx settings

* HTTP setups: Make `/etc/nginx/sites-available/gitlab` the same as [`lib/support/nginx/gitlab`](/lib/support/nginx/gitlab) but with your settings
* HTTPS setups: Make `/etc/nginx/sites-available/gitlab-ssl` the same as [`lib/support/nginx/gitlab-ssl`](/lib/support/nginx/gitlab-ssl) but with your setting

#### Setup time zone (optional)

Consider setting the time zone in `gitlab.yml` otherwise GitLab will default to UTC. If you set a time zone previously in [`application.rb`](config/application.rb) (unlikely), unset it.

### 6. Start application

    svc -u ~/service/sidekiq
    svc -u ~/service/gitlab

### 7. Check application status

Check if GitLab and its environment are configured correctly:

    bundle exec rake gitlab:env:info RAILS_ENV=production

To make sure you didn't miss anything run a more thorough check with:

    bundle exec rake gitlab:check RAILS_ENV=production

If all items are green, then congratulations upgrade is complete!

### 8. GitHub settings (if applicable)

If you are using GitHub as an OAuth provider for authentication, you should change the callback url so that it 
only contains a root url (ex. `https://gitlab.example.com/`)

## Things went south? Revert to previous version (7.6)

### 1. Revert the code to the previous version
Follow the [upgrade guide from 7.5 to 7.6](update-7.5-to-7.6.md), except for the database migration
(The backup is already migrated to the previous version)

### 2. Restore from the backup:

```bash
cd /home/[user]/gitlab
bundle exec rake gitlab:backup:restore RAILS_ENV=production
```
If you have more than one backup *.tar file(s) please add `BACKUP=timestamp_of_backup` to the command above.