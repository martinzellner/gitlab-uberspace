# From 7.5 to 7.6

### 0. Stop server

[assuming you use my scripts](https://blog.kanedo.net/1925,gitlab-7-0-auf-einem-uberspace-installieren.html)
```bash
svc -d ~/service/sidekiq/
svc -d ~/service/gitlab/
```

### 1. Backup

```bash
cd /home/[user]/gitlab
bundle exec rake gitlab:backup:create RAILS_ENV=production
```

### 2. Upgrade using the upgrade

```bash
cd /home/[user]/gitlab
ruby bin/upgrade.rb
```

### 3. Start application

```
svc -u ~/service/sidekiq/
svc -u ~/service/gitlab/
```

### 4. Check application status

Check if GitLab and its environment are configured correctly:

    bundle exec rake gitlab:env:info RAILS_ENV=production

To make sure you didn't miss anything run a more thorough check with:

    bundle exec rake gitlab:check RAILS_ENV=production

If all items are green, then congratulations upgrade is complete!