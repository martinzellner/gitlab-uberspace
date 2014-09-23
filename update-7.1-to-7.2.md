# From 7.1 to 7.2

## Editable labels

In GitLab 7.2 we replace Issue and Merge Request tags with labels, making it
possible to edit the label text and color. The characters `?`, `&` and `,` are
no longer allowed however so those will be removed from your tags during the
database migrations for GitLab 7.2.

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
git checkout 7-2-stable
```

### 3. Update gitlab-shell

```bash
cd /home/[user]/gitlab-shell
git fetch
git checkout v1.9.8
```

### 4. Install new system dependencies

drop the uberspace support a line and ask for cmake (please try section 5 first to see if the dependencies are really missing!)

### 5. Install libs, migrations, etc.

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
please use the new version of the gitlab service script in this repo under `services/gitlab`
```
if you see an error like
```
Gem::Ext::BuildError: ERROR: Failed to build gem native extension.

    /package/host/localhost/ruby-2.1.1/bin/ruby extconf.rb
checking for cmake... no
ERROR: CMake is required to build Rugged.
*** extconf.rb failed ***
```

please ask the uberspace support for cmake

### 6. Update config files

#### New configuration options for `gitlab.yml`

Update rack attack middleware config

```
cp config/initializers/rack_attack.rb.example config/initializers/rack_attack.rb
```

### 7. Start application

	svc -u ~/service/sidekiq
	svc -u ~/service/gitlab

### 8. Check application status

Check if GitLab and its environment are configured correctly:

    bundle exec rake gitlab:env:info RAILS_ENV=production

To make sure you didn't miss anything run a more thorough check with:

    bundle exec rake gitlab:check RAILS_ENV=production

If all items are green, then congratulations upgrade is complete!

## Things went south? Revert to previous version (7.1)

### 1. Revert the code to the previous version
Follow the [upgrade guide from 7.0 to 7.1](7.0-to-7.1.md), except for the database migration
(The backup is already migrated to the previous version)

### 2. Restore from the backup:

```bash
cd /home/git/gitlab
sudo -u git -H bundle exec rake gitlab:backup:restore RAILS_ENV=production
```
If you have more than one backup *.tar file(s) please add `BACKUP=timestamp_of_backup` to the command above.