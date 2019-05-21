# Bitcoin RM - Node Open Mining Portal

**Setting up your own Pool is very complex. Please contact us for help**

This is a Equihash mining pool based off of Node Open Mining Portal.

#### Production Usage Notice
This is beta software. All of the following are things that can change and break an existing Z-NOMP setup: functionality of any feature, structure of configuration files and structure of redis data. If you use this software in production then *DO NOT* pull new code straight into production usage because it can and often will break your setup and require you to tweak things like config files or redis data. *Only tagged releases are considered stable.*

#### WARNING
Usage of this software requires abilities with sysadmin, database admin, coin daemons, and sometimes a bit of programming. Running a production pool can literally be more work than a full-time job. 


### Community / Support

Please contact us via Discord. [Discord Invite](https://discord.gg/aj2QVc9)

### Some pools using BCRM Pool:

https://pool.bitcoinrm.org - The Official Bitcoin RM Mining Pool

Usage
=====

#### Requirements
* Coin daemon(s) (find the coin's repo and build latest version from source)
* [Node.js](http://nodejs.org/) v7+ ([follow these installation instructions](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager))
* [Redis](http://redis.io/) key-value store v2.6+ ([follow these instructions](http://redis.io/topics/quickstart))

[**Redis security warning**](http://redis.io/topics/security): be sure firewall access to redis - an easy way is to
include `bind 127.0.0.1` in your `redis.conf` file. Also it's a good idea to learn about and understand software that
you are using - a good place to start with redis is [data persistence](http://redis.io/topics/persistence).


#### 1) Setting up coin daemon
Follow the build/install instructions for your coin daemon. Your bcrm.conf file should end up looking something like this:
```
daemon=1
rpcuser=rpcuser
rpcpassword=rpcpasswd
rpcport=2095
```
For redundancy, its recommended to have at least two daemon instances running in case one drops out-of-sync or offline,
all instances will be polled for block/transaction updates and be used for submitting blocks. Creating a backup daemon
involves spawning a daemon using the `-datadir=/backup` command-line argument which creates a new daemon instance with
it's own config directory and coin.conf file. Learn about the daemon, how to use it and how it works if you want to be
a good pool operator. For starters be sure to read:
   * https://en.bitcoin.it/wiki/Running_bitcoind
   * https://en.bitcoin.it/wiki/Data_directory
   * https://en.bitcoin.it/wiki/Original_Bitcoin_client/API_Calls_list
   * https://en.bitcoin.it/wiki/Difficulty


#### 2) Downloading & Installing

Clone the repository and run `npm update` for all the dependencies to be installed:

Make sure your machine is running Fedora 27 with at least 2 GB RAM and 4GB swap.

CentOS Linux is too behind in package versions and not officially supported :(

```bash

  sysctl -w vm.overcommit_memory=1
  echo never > /sys/kernel/mm/transparent_hugepage/enabled
  sudo dnf install nodejs redis boost-devel libsodium-devel g++ make
  sudo systemctl enable redis
  sudo systemctl start redis

  git clone https://github.com/BitcoinRM/bcrm-pool
  cd bcrm-pool

  Edit hex string on Line 60 of node_modules/stratum-pool/lib/transactions.js for your pool name!
  (If you don't, the existing hex string reads: The Bitcoin RM Mining Pool pool.bitcoinrm.org)

  Edit files: pool_configs/bcrm.json pool_configs/bcrm_testnet.json
  Make sure you use correct username and password in place of rpcuser and rpcpasswd in two places
  Make sure you use correct addresses for: address tAddress invalidAddress rewardRecipients

  Edit website/pages/getting_started.html to replace pool.bitcoinrm.org by your stratum server name in two places

  Rebuild equihashverify as follows:
  sudo dnf install python2
  cd node_modules/equihashverify
  ../node-gyp/bin/node-gyp.js install    (This will install development files in ~/.node-gyp) 
  ../node-gyp/bin/node-gyp.js configure
  ../node-gyp/bin/node-gyp.js build
  strip build/Release/equihashverify.node build/Release/obj.target/equihashverify.node

  Rebuild bignum as follows:
  cd ../bignum
  ../node-gyp/bin/node-gyp.js configure
  ../node-gyp/bin/node-gyp.js build 
  strip build/Release/bignum.node build/Release/obj.target/bignum.node

  Clean up unwanted stuff
  sudo dnf erase make g++ python2
  \rm -rf ~/.node-gyp

  cd ../..
  npm start

If it works, automate using pm2 as explained in the next step.
```
###### Use pm2 to run it as a daemon:

```bash
cd ~/bcrm-pool
pm2 start npm --name bcrm-pool -- start

pm2 save
sudo env PATH=$PATH:/usr/bin /usr/local/lib/node_modules/pm2/bin/pm2 startup systemd -u $LOGNAME --hp $HOME
sudo systemctl enable pm2-$LOGNAME
pm2 save
```

##### Pool config

```
Please Note that: 1 Difficulty is actually 8192, 0.125 Difficulty is actually 1024.

Whenever a miner submits a share, the pool counts the difficulty and keeps adding them as the shares. 

ie: Miner 1 mines at 0.1 difficulty and finds 10 shares, the pool sees it as 1 share. Miner 2 mines at 0.5 difficulty and finds 5 shares, the pool sees it as 2.5 shares. 
```


##### [Optional] Setting up blocknotify
1. In files pool_configs/*.json, set enabled to false for "p2p" section
2. In your bcrm.conf file, set listen=0
3. In your bcrm.conf file, set the `blocknotify` command to use:
```
node [path to cli.js] [coin name in config] [block hash symbol]
```
Example: Add the following to `bcrm.conf`
```
blocknotify=node /home/user/bcrm-pool/scripts/cli.js blocknotify bcrm %s
```

Alternatively, you can use a more efficient block notify script written in pure C as follows:
```
cd /home/user/bcrm-pool/scripts
gcc blocknotify.c -o blocknotify
strip blocknotify
```

Example: Add the following to `bcrm.conf`
```
blocknotify=/home/user/bcrm-pool/scripts/blocknotify bcrm %s
```
4. Set blockRefreshInterval to 0 in config.json to disable RPC polling

##### [Optional] Setting up peer notification
1. In files pool_configs/*.json, set enabled to true for "p2p" section
2. In your bcrm.conf file, set listen=1
3. In your bcrm.conf file, delete blocknotify line
4. Set blockRefreshInterval to 0 in config.json to disable RPC polling

Credits
-------
### Z-NOMP
* [Joshua Yabut / movrcx](https://github.com/joshuayabut)
* [Aayan L / anarch3](https://github.com/aayanl)
* [hellcatz](https://github.com/hellcatz)

### NOMP
* [Matthew Little / zone117x](https://github.com/zone117x) - developer of NOMP
* [Jerry Brady / mintyfresh68](https://github.com/bluecircle) - got coin-switching fully working and developed proxy-per-algo feature
* [Tony Dobbs](http://anthonydobbs.com) - designs for front-end and created the NOMP logo
* [LucasJones](//github.com/LucasJones) - got p2p block notify working and implemented additional hashing algos
* [vekexasia](//github.com/vekexasia) - co-developer & great tester
* [TheSeven](//github.com/TheSeven) - answering an absurd amount of my questions and being a very helpful gentleman
* [UdjinM6](//github.com/UdjinM6) - helped implement fee withdrawal in payment processing
* [Alex Petrov / sysmanalex](https://github.com/sysmanalex) - contributed the pure C block notify script
* [svirusxxx](//github.com/svirusxxx) - sponsored development of MPOS mode
* [icecube45](//github.com/icecube45) - helping out with the repo wiki
* [Fcases](//github.com/Fcases) - ordered me a pizza <3
* Those that contributed to [node-stratum-pool](//github.com/zone117x/node-stratum-pool#credits)


License
-------
Released under the MIT License. See LICENSE file.
