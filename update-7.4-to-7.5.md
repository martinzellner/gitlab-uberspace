# From 7.4 to 7.5

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

### 2. Get latest code

```bash
git fetch --all
git checkout -- db/schema.rb # local changes will be restored automatically
git checkout 7-5-stable
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

### 4. Update config files

#### New configuration options for gitlab.yml

There are new configuration options available for gitlab.yml. View them with the command below and apply them to your current gitlab.yml.

```
git diff origin/7-4-stable:config/gitlab.yml.example origin/7-5-stable:config/gitlab.yml.example
```

#### Change timeout for unicorn

```
# set timeout to 60
editor config/unicorn.rb
```

### 5. Start application

```
svc -u ~/service/sidekiq/
svc -u ~/service/gitlab/
```

### 6. Check application status

Check if GitLab and its environment are configured correctly:

    bundle exec rake gitlab:env:info RAILS_ENV=production

To make sure you didn't miss anything run a more thorough check with:

    bundle exec rake gitlab:check RAILS_ENV=production

If all items are green, then congratulations upgrade is complete!


### 7. Optional optimizations for GitLab setups with MySQL databases

Only applies if running MySQL database created with GitLab 6.7 or earlier. If you are not experiencing any issues you may not need the following instructions however following them will bring your database in line with the latest recommended installation configuration and help avoid future issues. Be sure to follow these directions exactly. These directions should be safe for any MySQL instance but to be sure make a current MySQL database backup beforehand.

```
# Stop GitLab
```bash
svc -d ~/service/sidekiq/
svc -d ~/service/gitlab/
```

# Login to MySQL
mysql -u [user] -p

# do not type the 'mysql>', this is part of the prompt

# Convert all tables to use the InnoDB storage engine (added in GitLab 6.8)
SELECT CONCAT('ALTER TABLE gitlabhq_production.', table_name, ' ENGINE=InnoDB;') AS 'Copy & run these SQL statements:' FROM information_schema.tables WHERE table_schema = 'gitlabhq_production' AND `ENGINE` <> 'InnoDB' AND `TABLE_TYPE` = 'BASE TABLE';

# If previous query returned results, copy & run all outputed SQL statements

# Convert all tables to correct character set
SET foreign_key_checks = 0;
SELECT CONCAT('ALTER TABLE gitlabhq_production.', table_name, ' CONVERT TO CHARACTER SET utf8 COLLATE utf8_general_ci;') AS 'Copy & run these SQL statements:' FROM information_schema.tables WHERE table_schema = 'gitlabhq_production' AND `TABLE_COLLATION` <> 'utf8_unicode_ci' AND `TABLE_TYPE` = 'BASE TABLE';

# If previous query returned results, copy & run all outputed SQL statements

# turn foreign key checks back on
SET foreign_key_checks = 1;

# Find MySQL users
mysql> SELECT user FROM mysql.user WHERE user LIKE '%git%';

# If git user exists and gitlab user does not exist
# you are done with the database cleanup tasks
mysql> \q

# If both users exist skip to Delete gitlab user

# Create new user for GitLab (changed in GitLab 6.4)
# change $password in the command below to a real password you pick
mysql> CREATE USER 'git'@'localhost' IDENTIFIED BY '$password';

# Grant the git user necessary permissions on the database
mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER, LOCK TABLES ON `gitlabhq_production`.* TO 'git'@'localhost';

# Delete the old gitlab user
mysql> DELETE FROM mysql.user WHERE user='gitlab';

# Quit the database session
mysql> \q

# Try connecting to the new database with the new user
mysql -u git -p -D gitlabhq_production

# Type the password you replaced $password with earlier

# You should now see a 'mysql>' prompt

# Quit the database session
mysql> \q

# Update database configuration details
# See config/database.yml.mysql for latest recommended configuration details
#   Remove the reaping_frequency setting line if it exists (removed in GitLab 6.8)
#   Set production -> pool: 10 (updated in GitLab 5.3)
#   Set production -> username: git
#   Set production -> password: the password your replaced $password with earlier
editor /home/git/gitlab/config/database.yml

# Run thorough check
bundle exec rake gitlab:check RAILS_ENV=production
```


## Things went south? Revert to previous version (7.4)

### 1. Revert the code to the previous version
Follow the [upgrade guide from 7.3 to 7.4](7.3-to-7.4.md), except for the database migration
(The backup is already migrated to the previous version)

### 2. Restore from the backup:

```bash
cd /home/[user]/gitlab
bundle exec rake gitlab:backup:restore RAILS_ENV=production
```
If you have more than one backup *.tar file(s) please add `BACKUP=timestamp_of_backup` to the command above.
