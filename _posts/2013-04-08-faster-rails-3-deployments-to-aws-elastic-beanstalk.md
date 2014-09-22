---
layout: post
title: Faster Rails 3 deployments to AWS Elastic Beanstalk
---

Web applications deployments cannot be slow. The definition of _slow_,
of course, is relative. I like to think _slow_ is anything that breaks
the flow, that takes long enough to make me lose focus of what I am
doing.

Recently I moved a Rails 3 app from Heroku to Amazon Elastic Beanstalk
(EB). Deploying to Heroku was not showing the best performance results,
but to EB was taking enough time to make me lose my patience.

After understanding how EB works, identifying bottlenecks,
and caching things; I was able to reduce the time to deploy to near one
minute.

### EB: Behind the scenes

EB allows deployments in a Heroku-esque way simply executing `git
aws.push`. This command is actually an alias to a ruby script located
at `.git/AWSDevTools/aws.elasticbeanstalk.push`. It just pushes yours
latest commit to the EB servers.

When EB receives the latest version of your app, it packs the source
files in a zip archive, uploads to a S3 bucket under your account, and
deploys to the EB environment.

### Inspecting the EC2 plumbing

Analyzing the file `/var/log/eb-tools.log`, we can see the deployment
has hooks and scripts associated with it. The program at
`/usr/bin/directoryHooksExecutor.py` executes those scripts
sequentially in alphabetical order by name.

```
2013-04-07 14:03:59,414 [INFO] (3056 MainThread) [directoryHooksExecutor.py-29] [root directoryHooksExecutor info] Executing directory: /opt/elasticbeanstalk/hooks/appdeploy/enact/
2013-04-07 14:03:59,511 [INFO] (3056 MainThread) [directoryHooksExecutor.py-29] [root directoryHooksExecutor info] Executing script: /opt/elasticbeanstalk/hooks/appdeploy/enact/01_flip.sh
2013-04-07 14:04:00,420 [INFO] (3056 MainThread) [directoryHooksExecutor.py-29] [root directoryHooksExecutor info] Executing script: /opt/elasticbeanstalk/hooks/appdeploy/enact/09clean.sh
2013-04-07 14:04:00,600 [INFO] (3056 MainThread) [directoryHooksExecutor.py-29] [root directoryHooksExecutor info] Executing script: /opt/elasticbeanstalk/hooks/appdeploy/enact/99_reload_app_server.sh
```

Given EC2 (non-EBS) does not persist data, and EB auto-scale our EC2 instances,
we cannot change these deploys scripts. However, we can [customize the
EC2](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers-ec2.html)
with `commands` and `container_commands`.

### Finding the bottlenecks

To customize the EC2 correctly, I first had to understand how exactly
the commands specified in my `.ebextensions/*.config` files act in the
whole deployment.

I created the two commands below, and added to the top of each
`/opt/elasticbeanstalk/hooks/appdeploy/**/*.sh` script an instruction to
create a file with the timestamp of its execution.

```yaml
# .ebextensions/app.config
commands:
  01_first_command:
    command: "touch /tmp/$(date +'%T.%N').command"
container_commands:
  01_first_container_command:
    command: "touch /tmp/$(date +'%T.%N').container_command"
```

```sh
# /opt/elasticbeanstalk/hooks/appdeploy/pre/01_unzip.sh
#!/usr/bin/env bash
touch /tmp/$(date +"%T.%N").$(basename $0)

. /opt/elasticbeanstalk/support/envvars

mkdir -p $EB_CONFIG_APP_BASE && chown $EB_CONFIG_APP_USER:$EB_CONFIG_APP_USER $EB_CONFIG_APP_BASE
[ -d $EB_CONFIG_APP_ONDECK ] && rm -rf $EB_CONFIG_APP_ONDECK
su -c "/usr/bin/unzip -d $EB_CONFIG_APP_ONDECK $EB_CONFIG_SOURCE_BUNDLE" $EB_CONFIG_APP_USER
chmod 775 $EB_CONFIG_APP_ONDECK
```

After a deploy, I was able to see the exact sequence of the
commands executed, and how long each one was taking. The whole process
was taking about unconceivable __8 minutes__ to complete. The culprits
for such slowness were, as suspected, gems installation and assets
compilation.

```
$ ls -1 /tmp
19:56:22.524828173.command
19:56:23.188093045.01_unzip.sh
19:56:23.524226323.02_setup_envvars.sh
19:56:23.575868688.10_bundle_install.sh
19:59:32.932137842.11_asset_compilation.sh
20:04:29.588351276.12_db_migration.sh
20:04:30.618231911.container_command
20:04:38.656611666.01_flip.sh
20:04:39.683074030.09clean.sh
20:04:39.706735596.99_reload_app_server.sh
```

### Caching gems and assets

We need to cache the gems to optimize `bundle install`, and to boost
the asset compilation we install [Turbo
Sprockets](https://github.com/ndbroadbent/turbo-sprockets-rails3) and
cache the compiled assets as well.

Unfortunately, we cannot use `container_commands`
because we need to set up our cached files between `01_unzip.sh` and
`10_bundle_install.sh`. Nor can we use `commands`, because
`01_unzip.sh` cleans up the target directory before unpacking your
source files. We can use `files` option to inject a script to set up
the cache.

Since the hook scripts are executed in alphabetical order, we need to
name the injected script correctly to be executed just after
`01_unzip.sh`. We will name it `01a_bootstrap.sh`.

```yaml
files:
  /opt/elasticbeanstalk/hooks/appdeploy/pre/01a_bootstrap.sh:
    mode: 00755
    owner: root
    group: root
    source: http://s3.amazonaws.com/mybucket/bootstrap.sh
```

The cached files were previously moved to `/var/app/support`. The
bootstrap script will create symbolic links to the directory being
deployed.

```sh
# /opt/elasticbeanstalk/hooks/appdeploy/pre/01a_bootstrap.sh
#!/usr/bin/env bash

mkdir /var/app/ondeck/vendor /var/app/ondeck/public /var/app/support/bundle /var/app/support/assets

ln -s /var/app/support/bundle /var/app/ondeck/vendor
ln -s /var/app/support/assets /var/app/ondeck/public
```

Ultimately, we could drop the deployment time near to __1 minute and 4
seconds__---the time to download the packages is not counted here, but
since it is in S3 in the same region, it takes no longer than 3 seconds
to get 30MB. This is the kind of slowness I can withstand.

```
$ ls -1 /tmp
01:49:05.696544004.01_unzip.sh
01:49:06.029004848.01a_bootstrap.sh
01:49:06.055676774.02_setup_envvars.sh
01:49:06.120822495.10_bundle_install.sh
01:49:07.156151174.11_asset_compilation.sh
01:50:07.243165374.12_db_migration.sh
01:50:08.454668833.01_flip.sh
01:50:09.648283462.09clean.sh
01:50:09.889387126.99_reload_app_server.sh
```

### Cleaning up and updating the cache

Given the deployment is no more an issue, we need to make sure our
cached files correspond to the latest version of our application. We
can use a `post` hook to execute another script to update the cache.

```yaml
files:
  /opt/elasticbeanstalk/hooks/appdeploy/post/01_update_cache.sh:
    mode: 00755
    owner: root
    group: root
    source: http://s3.amazonaws.com/mybucket/update_cache.sh
```

```sh
# /opt/elasticbeanstalk/hooks/appdeploy/post/01_update_cache.sh
#!/usr/bin/env bash

. /opt/elasticbeanstalk/support/envvars

if [ "$RAILS_ENV" != "staging" ]; then
  exit
fi

cd /var/app/support

tar zcf bundle.tar.gz bundle
s3put -b mybucket -p /var/app/support -g public-read bundle.tar.gz

tar zcf assets.tar.gz assets
s3put -b mybucket -p /var/app/support -g public-read assets.tar.gz
```

Thanks to staging-production parity, we can safely use our
staging server to keep the cache updated, and leave the production
server exclusively to our application.

We send the cache packages to our S3 bucket, so it will be available to
any new EC2 instance EB starts. The only thing left is to configure
the EC2 to download and unpack those packages. We can easily accomplish
this using the `sources` key.

```yaml
sources:
  /var/app/support: http://s3.amazonaws.com/mybucket/bundle.tar.gz
  /var/app/support: http://s3.amazonaws.com/mybucket/assets.tar.gz
```

We have made great changes to our deploy time. Nevertheless, there is
room for improvements. For example, the `update_cache.sh` script could
check for changes in the cached packages and upload a new version only
when necessary.

### Update Apr, 09

I had some issues with the asset compilation. The problem is that the
script
`/opt/elasticbeanstalk/hooks/appdeploy/pre/11_asset_compilation.sh`
executes `rake` directly, therefore it was not using the bundled gems.
I set my bootstrap script to change it to `bundle exec rake`.

```sh
$ sed -i 's/"rake/"bundle exec rake/' /opt/elasticbeanstalk/hooks/appdeploy/pre/11_asset_compilation.sh
```

Another issue happened with passenger and git backed libraries.
Thanks to [these](http://stackoverflow.com/a/13657473)
[answers](http://stackoverflow.com/a/13656775), I was able to fix the
problem injecting another script right after `10_bundle_install.sh` to
pack all the gems.

```yaml
files:
  /opt/elasticbeanstalk/hooks/appdeploy/pre/10a_bundle_pack.sh:
    mode: "00755"
    owner: root
    group: root
    source: https://s3.amazonaws.com/mybucket/bundle_pack.sh
```

```sh
# /opt/elasticbeanstalk/hooks/appdeploy/pre/10a_bundle_pack.sh
#!/usr/bin/env bash

. /opt/elasticbeanstalk/support/envvars

cd /var/app/ondeck

bundle pack --all
```

I also added the `vendor/cache` directory to the `bundle.tar.gz` to
avoid any delays in the deployment.

I have put all my configuration files and scripts in this [gist](https://gist.github.com/wicz/5345688).

