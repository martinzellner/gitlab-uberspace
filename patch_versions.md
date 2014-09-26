# Universal update guide for patch versions

For example from 6.2.0 to 6.2.1, also see the [semantic versioning specification](http://semver.org/).

### 0. Backup

It's useful to make a backup just in case things go south:
(With MySQL, this may require granting "LOCK TABLES" privileges to the GitLab user on the database version)

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

### 2. Get latest code for the stable branch

```bash
cd /home/[user]/gitlab
git fetch --all
git checkout LATEST_TAG
```

Replace LATEST_TAG with the latest GitLab tag you want to upgrade to, for example `v6.6.3`.

### 3. Update gitlab-shell if it is not the latest version

```bash
cd /home/[user]/gitlab-shell
git fetch
git checkout LATEST_TAG
```

Replace LATEST_TAG with the latest GitLab Shell tag you want to upgrade to, for example `v1.7.9`.

### 4. Install libs, migrations, etc.

```bash
cd /home/[user]/gitlab

# MySQL
bundle install --without development test postgres --deployment

bundle exec rake db:migrate RAILS_ENV=production
bundle exec rake assets:clean RAILS_ENV=production
bundle exec rake assets:precompile RAILS_ENV=production
bundle exec rake cache:clear RAILS_ENV=production
```

### 5. Start application

    svc -u ~/service/sidekiq/
	svc -u ~/service/gitlab/

### 6. Check application status

Check if GitLab and its environment are configured correctly:

    bundle exec rake gitlab:env:info RAILS_ENV=production

To make sure you didn't miss anything run a more thorough check with:

    bundle exec rake gitlab:check RAILS_ENV=production

If all items are green, then congratulations upgrade complete!
