+++
date = '2024-12-12T23:31:30Z'
draft = true
title = 'How to selfhost a public FiveM server with txAdmin and QBcore'
author = 'Bumzag'
categories = ["FiveM", "txAdmin", "QBCore"]
tags = ["fivem", "txadmin", "qbcore", "ufw", "mariadb"]
description = "Description of this page."
keywords = "how to install fivem, how to setup public fivem, selfhost fivem, fivem txadmin qbcore"
+++

### Requirements
- MariaDB
- ufw
- FiveM
- txadmin
- QBox

This guide assumes a fresh debian install, starting as `root`. This is on VPS so there's no need to portforward. Home-hosting is not recommended.

Shell into your server and start by updating your packages. Then create your new `sudo` user and switch to it. We'll use this new user going forward.

```bash
ssh default@server
```

```bash
apt update; apt upgrade -y;
adduser user #replace 'user' with your desired username
usermod -aG sudo user
su - user
```

This would be a good time to set up this new user's ssh keys 

### Install MariaDB

QBCore requires a database, so we'll need to install and configure it, as well as create a new database user.

```bash
sudo apt install mariadb-server -y
sudo mysql_secure_installation
```
{{< details summary="mysql_secure_installation output" >}}
```bash
# mariadb-secure-installation
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] n
 ... skipping.

You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] y
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```
{{< /details >}}

To create the user, open the MariaDB prompt

```bash
sudo mariadb
# change 'db_user' to your preferred database username
MariaDB [(none)]> GRANT ALL ON *.* TO 'db_user'@'localhost' IDENTIFIED BY 'db_password' WITH GRANT OPTION;
```

That's it, your database is a now installed and configured. Make sure you securely store the password for `root` and `db_user`.

### Install ufw

The `ufw` package stands for uncomplicated firewall, it's a super easy to configure firewall for debian. It's optional, but recommended. Install it and allow your ports.

```bash
sudo apt install ufw -y
sudo ufw enable # if remote, do no disconnect until reconfiguring ssh
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
sudo ufw allow 22/tcp # for SSH
sudo ufw allow 30120 # allow FiveM connections through
```

You may notice we didn't allow port 40120 for txAdmin, more on that later.

### Download and install FiveM

First create a directory where your server will live. We're using `/opt/fivem-server`

```bash
sudo mkdir /opt/fivem-server # name your server directory whatever you'd like
sudo chown user:user /opt/fivem-server # take ownership
cd /opt/fivem-server
```

Now download the latest FiveM artifacts from [here](https://runtime.fivem.net/artifacts/fivem/build_proot_linux/). As of writing this, it's `11895`.

```bash
wget https://runtime.fivem.net/artifacts/fivem/build_proot_linux/master/11895-91bd18e26c425d8316e98f0f7abbc7630b54e092/fx.tar.xz
tar -xf fx.tar.xz # extract the contents
rm -R fx.tar.xz   # delete the tarball
```

If you `ls`, you'll see a directory named `alpine` and a file named `run.sh`. Start the server with that bash file

```bash
bash run.sh
```

{{< details summary="server output" open="true" >}}
```
[    c-server-monitor]   _______  ______
[    c-server-monitor]  |  ___\ \/ / ___|  ___ _ ____   _____ _ __
[    c-server-monitor]  | |_   \  /\___ \ / _ \ '__\ \ / / _ \ '__|
[    c-server-monitor]  |  _|  /  \ ___) |  __/ |   \ V /  __/ |
[    c-server-monitor]  |_|   /_/\_\____/ \___|_|    \_/ \___|_|
[    c-server-monitor] -------------------------------- monitor ---
[    c-server-monitor]
[    c-scripting-core] Creating script environments for monitor
[23:13:26][tx:v6.0.2] Profile 'default' starting...
[23:13:26][tx:SetupProfile] =========================================================
[23:13:26][tx:SetupProfile] Creating new profile folder...
[23:13:26][tx:SetupProfile] Server profile was saved in '/opt/illium-server/txData/default'
[23:13:26][tx:SetupProfile] =========================================================
[23:13:26][tx:WebServer] Listening on 0.0.0.0.
[23:13:27][tx:UpdateChecker] This version of txAdmin is outdated.
[23:13:27][tx:UpdateChecker] Please update as soon as possible.
[23:13:27][tx:UpdateChecker] For more information: https://discord.gg/uAmsGa2
[23:13:36][tx]
[23:13:36][tx]    ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
[23:13:36][tx]    ┃                                    ┃
[23:13:36][tx]    ┃     All ready! Please access:      ┃
[23:13:36][tx]    ┃    http://your-public-ip:40120/    ┃
[23:13:36][tx]    ┃     http://your_ip:40120/          ┃
[23:13:36][tx]    ┃                                    ┃
[23:13:36][tx]    ┃   Use the PIN below to register:   ┃
[23:13:36][tx]    ┃                1111                ┃
[23:13:36][tx]    ┃                                    ┃
[23:13:36][tx]    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
[23:13:36][tx]
[23:13:36][tx] To be able to access txAdmin from the internet open port 40120
[23:13:36][tx] on your OS Firewall as well as in the hosting company.
[23:13:36][tx:FXRunner] Please open txAdmin on the browser to configure your server.
[23:13:55][tx:PlayerDatabase] Internal Database optimized. This applies only for the txAdmin internal database, and does not affect your MySQL or framework (ESX/QBCore/etc) databases.
[23:13:55][tx:PlayerDatabase] - 0 players that haven't connected in the past 16 days and had less than 2 hours of playtime.
[23:13:55][tx:PlayerDatabase] - 0 whitelist requests older than a week.
[23:13:55][tx:PlayerDatabase] - 0 whitelist approvals older than a week.
```
{{< /details >}}

At this point you'll see that message about opening port 40120 to access txAdmin remotely. You can if you'd like, but I don't recommend it. It's not necessary, and you can avoid it with an ssh tunnel:

```bash
ssh -L 40120:localhost:40120 user@server
```

If you'd prefer to just open port 40120, allow it in `ufw`:
```
sudo ufw allow 40120
```

Now on your personal PC, open your browser and go to `localhost:40120`. You should be prompted for the 4 digit code shown in the output after starting your server above, `1111` in our case. Type that in and then link your `cfx.re` account.

When that's done, it should have brought you back to set up your server profile. Go through the 6 steps:
  1. Click 'next'
  2. Give your server a name
  3. Select your deployment type. For QBCore, select Popular recipes 
  4. Select the QBCore framework recipe
  5. Data location. 
      You can change this if you'd like, but txAdmin will autofill a path for you. It's the recommended path, so change with caution.
  6. Click "Go to Recipe Deployer"

You'll now set up the server recipe. There will be a large textfield with the recipe/resource info that will get loaded into the database, no need to make changes.

  1. Click 'Next'
  2. Enter your server license key. More info on that [here](https://support.cfx.re/hc/en-us/articles/8014850328348-How-to-create-a-server-license-key)

  **DO NOT CLICK RUN RECIPE**

  The reason why is because we changed our database information earlier. If you had run the recipe now, it will throw an error because it attempts to connect to the database with the default info, i.e `root` with no password. 

  Instead, click `Show/Hide Database options (advanced)`. A form should now be visible, this is where our database information goes.

  - `Database Host` and `Database Port` 
  You can leave these alone. You wouldn't need to change host unless you were installing the server/database on different hosts, or the port unless you specifically need to run MariaDB on a different port (3306 is MySQL default)
  - `Database Username`
  put your `db_user` from the MariaDB section.
  - `Database Password`
  put your `db_password` from the MariaDB section.
  - `Database Name`
  This should be pre-filled with the deployment ID. It will also be the deployment ID if left blank.
  - `Delete Database`
  This will delete the database with the name from above if it already exists.

  3. Now click `Run Recipe`, and you should see output below the message: 
  ```
  Your recipe is being executed, the server will be deployed to: 
  /opt/fivem-server/txData/QboxStable_XXXXXX.base/
  ```

  4. Edit your server config if you'd like, and then click `Save & Run Server` at the bottom

  You will now be logged into the txAdmin interface on the `Live Console` page where you'll see the server startup output. You may see your CPU usage, in the bottom left, spike while the server sets up, but it should go down after.

#### if you get the following error
`[              STDERR] Could not bind on 0.0.0.0:30120 - is this address valid and not already in use?`

Find the process ID of whatever is running on 30120 and kill it
```
sudo lsof -i :30120
COMMAND    PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
example    100  user   77u  IPv6  17394      0t0  UDP *:30120
sudo kill 100
```

### Done

That should be it, you should now be able to access your server through the FiveM app and connect to it.