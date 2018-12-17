# Lisk-Core on Raspberry pi 3b+

This will be a readme on how to install Lisk-Core on a Raspberry pi. The network that I’ve used to test this is testnet.
The installation will be like the source installation of Lisk-Core.
Make sure that you have enough space available on your Raspberry pi! I would recommend to get a higher micro SD card instead of the one that you receive with your pi.
## Creating a new user
` sudo adduser lisk `

After creating your user, you can change to the lisk user if you have sudo rights.
` su – lisk `
If you don’t have sudo rights use the following command to give lisk those rights:
` sudo usermod -aG sudo lisk ` 
### Tool chain components
Used for compiling dependecies.
```
sudo apt-get update
sudo apt-get install -y python build-essential curl automake autoconf libtool ntp
```
## Git
Git is used for cloning and updating Lisk.

` sudo apt-get install -y git `
## Node-js
Node.js serves as the underlying engine for code execution.

Install System wide via package manager, like so:
```
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt-get install -y nodejs
```
## Installation of the correct node version
Here I’ve followed custom steps that I’ve used while setting up my betanet node.
```
sudo apt-get install npm
sudo npm i -g npm
sudo npm install -g n
sudo n 8.9.0
```

## PM2
It is recommended to install PM2 for an easier use on your node. You can install it like this.

` sudo npm install -g pm2 `
## PostgreSQL
The version that is used on the other linux distros is PostgreSQL 10. But because this version isn’t supported on Raspberry pi you must use PostgreSQL 9.6
```
sudo apt-get purge -y postgres* # remove all already installed postgres versions
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
sudo apt install wget ca-certificates
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt update
sudo apt-get install postgresql-9.6 postgresql-contrib libpq-dev
```
When you installed PostgreSQL, you should see it running with the following command:
` pg_lsclusters `
When you see that your cluster is running you must drop the existing one, and replace it with the locale ` en_US.UTF-8 `
```
sudo pg_dropcluster --stop 9.6 main
sudo pg_createcluster --locale en_US.UTF-8 --start 9.6 main
```
After you’ve created the locale cluster, create a new database user called lisk with the rights to create its own databases:
` sudo -u postgres createuser --createdb lisk `

When you created the new database user you can start setting up your database that will be used for the chain that you will use on your Raspberry pi.

` createdb lisk_test `

If you want to use your pi for mainnet you will need to use: 

` createdb lisk_main `

After the db is created you can continue setting up your database:

` sudo -u postgres psql -d lisk_test -c "alter user lisk with password 'password';" `

## Installing Lisk
When the database setup is done you have to install lisk, you can do it like this:
` git clone https://github.com/LiskHQ/lisk.git `

After lisk is downloaded change to the lisk directory:
` cd lisk `

When you’re in the lisk folder change to the latest release tag (you can check this on https://github.com/LiskHQ/lisk/releases):

`git checkout v1.3.1-rc.0 -b v1.3.1-rc.0`

v1.3.0 is the latest version while creating this readme.

After lisk is installed and you’re on the right the you can choose to sync from a snapshot or sync from genesis block.
Both take some time but syncing from the snapshot is a lot faster.
First you must download the latest snapshot from https://downloads.lisk.io/lisk/test/blockchain.db.gz
```
For example:
wget https://downloads.lisk.io/lisk/test/blockchain.db.gz
```

To sync your database after you’ve downloaded the snapshot you must use: 

` gunzip -fcq blockchain.db.gz | psql -d lisk_test `

Note that this will take some time to sync and I would recommend attaching your pi to an external monitor and use the command there. While doing it over ssh I’ve lost the connection with my pi. 

After the database is fully synced to the latest point from the snapshot use ` node app.js --network testnet ` while you’re in the lisk folder.

For mainnet this will be ` node app.js --network mainnet `.

If you want to sync from the genesis block use:
` node app.js --network testnet `
If you want to use it on mainnet change ` testnet ` to ` mainnet `.

If everything went well you shouldn’t see any error code, only some output like this:
```
[{"code":"UNUSED_DEFINITION","message":"Definition is not used: #/definitions/CommonBlock","path":["definitions","CommonBlock"]},{"code":"UNUSED_DEFINITION","message":"Definition is not used: #/definitions/PeersList","path":["definitions","PeersList"]},{"code":"UNUSED_DEFINITION","message":"Definition is not used: #/definitions/WSPeerHeaders","path":["definitions","WSPeerHeaders"]},{"code":"UNUSED_DEFINITION","message":"Definition is not used: #/definitions/WSPeerUpdateRequest","path":["definitions","WSPeerUpdateRequest"]},{"code":"UNUSED_DEFINITION","message":"Definition is not used: #/definitions/WSSignaturesList","path":["definitions","WSSignaturesList"]},{"code":"UNUSED_DEFINITION","message":"Definition is not used: #/definitions/WSBlocksList","path":["definitions","WSBlocksList"]},{"code":"UNUSED_DEFINITION","message":"Definition is not used: #/definitions/WSBlocksCommonRequest","path":["definitions","WSBlocksCommonRequest"]},{"code":"UNUSED_DEFINITION","message":"Definition is not used: #/definitions/WSTransactionsRequest","path":["definitions","WSTransactionsRequest"]},{"code":"UNUSED_DEFINITION","message":"Definition is not used: #/definitions/WSAccessObject","path":["definitions","WSAccessObject"]},{"code":"UNUSED_DEFINITION","message":"Definition is not used: #/definitions/WSTransactionsResponse","path":["definitions","WSTransactionsResponse"]},{"code":"UNUSED_DEFINITION","message":"Definition is not used: #/definitions/WSSignaturesResponse","path":["definitions","WSSignaturesResponse"]},{"code":"UNUSED_DEFINITION","message":"Definition is not used: #/definitions/WSBlocksBroadcast","path":["definitions","WSBlocksBroadcast"]}] 2
```
You can also check your logs in logs/lisk.log to see if everything went well.

If this is successful you can create a process with the following command:

`  pm2 start --name lisk app.js -- --network testnet `

Or on mainnet:

`  pm2 start --name lisk app.js -- --network mainnet `

Check once again in your logs to see if everything is working.

## Updating Lisk-Core

When you want to update Lisk-Core you must stop the PM2 process and pull the latest tagged release from GitHub. After that you must start the PM2 process again. You can do it by using the following commands:

### Testnet
```
pm2 stop lisk
git fetch --all
git checkout tags/v{version}
pm2 start lisk
```

If you run into problems while upgrading it is recommended to check the git status and there might show up some errors.` git status `

### Mainnet

```
pm2 stop lisk
git pull
git checkout v{version} -b v{version}
pm2 start lisk
```

Links that I've used to make Lisk-Core available on a Raspberry pi:
- https://www.postgresql.org/docs/9.4/backup-dump.html
- http://snapshots.lisk.io.s3-eu-west-1.amazonaws.com/index.html?prefix=lisk/test/
- https://pastebin.com/nmgtawca
- https://lisk.io/documentation/lisk-core/setup/source
- https://lisk.io/documentation/lisk-core/upgrade/source
- https://www.raspberrypi.org/forums/viewtopic.php?t=211528
- https://github.com/S3x0r/Lisk-Core-FreeBSD

If you need more help contact me on [lisk.chat](https://lisk.chat/direct/zOwn3Ds).
Feel free to give a small donation if this was helpfull to make Lisk-Core available on your Raspberry pi on: `12056889420610179211L`.
