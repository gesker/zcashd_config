# Setup Zcash on Development Workstation


This document is a guide for installing zcashd, both mainnet and testnet, on a development workstation.


## Preamble


This guide has been generated using notes I have collected from various sources that have been useful to me for setting up full node zcashd server. It is useful to have a full node server installed if one is building an application using the zcash blockchain. It is helpful when the zcashd software is communicating and syncronizing on BOTH the main (mainnet) and test (testnet) networks and the start/stop lifecycle is controlled by the operating system. While there is a great deal of good information resources available for zcashd I hope this recipe/guide can bring much of that together, along with a touch of sysadmin, to lower the barrier of entry for developers wishing to build applications utilizing the zcash blockchain.


## License


    Copyright (c) 2023 Dennis R. Gesker.
    Permission is granted to copy, distribute and/or modify this document
    under the terms of the GNU Free Documentation License, Version 1.3
    or any later version published by the Free Software Foundation;
    with no Invariant Sections, no Front-Cover Texts, and no Back-Cover Texts.
    A copy of the license is included in the section entitled "GNU


## Goals

- Install zcashd (zcashd)[https://github.com/zcash/zcash] 
  - Using the apt.z.cash repository
- Configure zcashd to automatically start using systemd
  - Mainnet (mainnet)
  - Testnet (testnet)
- Configure zcashd
  - Mainnet (mainnet)
    - RPC Credentials
    - Backup Directory
        - Backup Wallet
        - Generate Pass Phrase
    - Mining Address
    - Connect to Specific upstream zcashd server (optional)
  - Testnet (testnet)
    - RPC Credentials
    - Backup Directory
        - Backup Wallet
        - Generate Pass Phrase
    - Mining Address
    - Connect to Specific upstream zcashd server (optional)
- Configure logrotated
 - Symlink logs to /var/log directory


## Non-Goals


- Guarantee Security of the configuration
- This is for a personal workstation only!


## Miscellaneous 


- I am not part of the (zcash)[https://github.com/zcash/zcash] project
- I do use this configuration on my own workstation
  - Use at your own risk!
- Pull requests to improve these notes are welcome
- **DO NOT USE ANY OF THE GENERATED CREDENTIALS OR ADDRESSES FOUND IN THIS DOCUMENT** 
  - I will show you how to generate **your own** credentials using the script provided by the zcash project.


## Requirements


- Debian 11 (Bullseye)
- Ability to sudo on the system
- Ability to use a text editor and navigate directories
- About 400 GB of disk space in your home ~/ directory


## Step 1 - Install some prerequisites


```
sudo apt update
sudo apt install git python logrotate curl 
```


## Step 2 - Install Zcashd (Using apt)


### Add an entry for zcash to your apt sources


```bash
sudo vi /etc/apt/sources.list

```

Append the following line:

```
deb [arch=amd64 signed-by=/usr/share/keyrings/zcash.gpg] https://apt.z.cash/ bullseye main
```

A reference copy of a sources.list file can be found [here](./sources.list).


### Add the zcash project apt repository gpg keys:


```bash
sudo wget -qO - https://apt.z.cash/zcash.asc | sudo gpg --import
sudo gpg --output /usr/share/keyrings/zcash.gpg --export 6DEF3BAF272766C0
```

*Tip*: Always verify any gpg keys provided to you in guides and scripts. Compare the signature 6DEF3BAF272766C0 to the signature published in the [keyserver](http://pgp.mit.edu/pks/lookup?search=signingkey%40z.cash&op=index) and in the [official documentation](https://zcash.readthedocs.io/en/latest/rtd_pages/install_debian_bin_packages.html) provided by the zcash project.
 

### Update apt and install zcashd


```
sudo apt update
sudo apt install zcashd
```

The zcashd software is now installed on your system. However, we are going to complete its configuration before we attempt to start the daemon.


## Step 3 - Download Source Code and Parameters


We have installed the zcash software using the apt repository. Why are we downloading the source code?

Good question! There is a command, a python 2 script, that we require that does not seem to be included with the Debian package provided by the zcash project. We will use that script when we generate credentials in a step of this guide below. There are other tools available that will also let us generate these credentials but since this guide is about installing zcashd we will use the scripts made available by the zcash project.


```
mkdir ~/Development/
cd ~/Development/
./zcutil/fetch-params.sh
git clone git@github.com:zcash/zcash.git
```

We will also install the zcash blockchain parameters which will be installed to ~/.zcash-params.


```bash
zcash-fetch-params
```


## Step 3 - Add Zcashd configuration(s)

Create ~/.zcash directory parameters and copy both [zcash.conf.mainnet](zcash.conf.mainnet) and [zcash.conf.testnet](zcash.conf.testnet) into the ~/.zcash folder.

Also, create a symlink for one of these configuration files to ~/.zcash/zcash.conf. This zcash.conf file is not needed specifically needed for this recipe but zcash will complain and fail to start should you launch zcashd manually and do not specify a configuration file on the command line. So, it is good to have a default zcash.conf file in the expected place.

```bash
mkdir ~/.zcash
cp zcash.conf.mainnet ~/.zcash
cp zcash.conf.testnet ~/.zcash
ln -s ~/.zcash/zcash.conf.mainnet ~/.zcash/zcash.conf
```

### RPC Credentials


There are a few lines in [zcash.conf.mainnet](zcash.conf.mainnet) and [zcash.conf.testnet](zcash.conf.testnet) that will need your SPECIAL attention.

- rpcauth=mySecureUsername:mySecureCredential
- rpcuser=mySecureUsername
- rpcpassword=mySecurePassword

Even though this is a setup for your development workstation it is still wise to take security seriously and use good secure credentials.

**Note**: *This guide does NOT cover TLS configuration!*

'rpcuser' and 'rpcpassword' are being deprecated. Zcashd will issue a log entry confirming this. Still, it seems that if you want to make sue of another package from the zcash project [lightwalletd][https://github.com/zcash/lightwalletd] these configuration entries need to be present.

'rpcauth' will correspond to the credentials one would put into the header of the REST calls that your application makes to the zcashd server. 

Please use a much better username than **mySecureUsername**.

We'll use a utility included with the zcash source code to generate the values you will need to replace the **mySecurePassword** and **mySecureCredential** values in **your** [zcash.conf.mainnet](zcash.conf.mainnet) and [zcash.conf.testnet](zcash.conf.testnet) files.


```bash
~/Development/zcash/share/rpcuser/rpcuser.py mySecureUsername
```


The output will look something like:


```
String to be appended to zcash.conf:
rpcauth=mySecureUsername:1228c4e1b813a679f35ea18cfe81b8d$70b7a9a81583cd612dfd68d1a6a074feef90dff106eabed799f253762d9ac2f6
Your password:
SQ-VFegQi-IJFQYvtXvzhaP1HQOcaGfhskZ2aPa_M1M=
```

So, In this example you would update the lines:

```
rpcauth=mySecureUsername:mySecureCredential
rpcuser=mySecureUsername
rpcpassword=mySecurePassword
```

to


```
rpcauth=mySecureUsername:1228c4e1b813a679f35ea18cfe81b8d$70b7a9a81583cd612dfd68d1a6a074feef90dff106eabed799f253762d9ac2f6
rpcuser=mySecureUsername
rpcpassword=SQ-VFegQi-IJFQYvtXvzhaP1HQOcaGfhskZ2aPa_M1M=
```


Again, use the values you generated for 'rpcauth' and 'rpcpassword' in **both** the [zcash.conf.mainnet](zcash.conf.mainnet) and [zcash.conf.testnet](zcash.conf.testnet) files in your own ~/.zcash folder and choose a better 'rpcuser' value/name.



### Backup Directory


Currently 'backupdir' is set to /tmp which is not at all secure. It is important that you update this value in both files.

Make sure 'backupdir' in both the [zcash.conf.mainnet](zcash.conf.mainnet) and [zcash.conf.testnet](zcash.conf.testnet) are set to VALID and SAFE locations. 


### Mining address


If you set 'gen=1' in the [zcash.conf.mainnet](zcash.conf.mainnet) and [zcash.conf.testnet](zcash.conf.testnet) update the 'mineraddress' to your **OWN** zcash addresses! And, it is best if you have both an mainnet and testnet address NOT associated with this wallet.dat file. The reason being that sometimes in development things go wrong. Even though it is hard to do you may have a scenario where the wallet.dat file gets corrupted and, wouldn't you know it, you don't have a way to restore the wallet.dat file from backup some reason. **Hey! Thing happen and go wrong.** In this guide we are going to backup both our mainnet and testnet wallet.dat files but still consider having a second address for any mining activity performed by your zcashd nodes and use those address(es) in the nodes that you control. 

You will see that generating a new wallet or address is pretty trivial after you setup up your first zcash node.


### Connect


Both [zcash.conf.mainnet](zcash.conf.mainnet) and [zcash.conf.testnet](zcash.conf.testnet) have a commented out 'connect' entry. If you have the resources to run a full node on your LAN/WAN that can safely be connected to the Internet; NOT YOUR WORKSTATION AS WE ARE SETTING UP HERE. Having more nodes in a blockchain network is always helpful to the blockchain network you are using. And, if this **ISOLATED** and **PROPERLY FIRE WALLED** dedicated full node is also accessible on your LAN then you can use the IP address of that dedicated node as your connect entry. Then your workstation will connect ONLY to the dedicated node on your LAN instead of random nodes on the Internet.


## Step 3 - Add Zcashd Systemd configuration(s)


In this repository there are two systemd configuration files; [zcashd_mainnet.service](zcashd_mainnet.service) and [zcashd_testnet.service](zcashd_testnet.service). Systemd is the part of your Debian system that, among other things, controls which services and daemons run on your system. 

In both of these files update the **'ExecStart'** lines to be sure they are pointing at YOUR home directory and more specifically the corresponding **zcash.conf.????net** file under in the **~/.zcash** directory.


In [zcashd_mainnet.service](zcashd_mainnet.service):


ExecStart=/usr/bin/zcashd -conf=/home/***yourUsername***/.zcash/zcash.conf.mainnet


In [zcashd_testnet.service](zcashd_testnet.service):


ExecStart=/usr/bin/zcashd -conf=/home/***yourUsername***/.zcash/zcash.conf.testnet


and copy these files to your systemd folder and enable these as systemd services.


```bash
sudo cp zcashd_mainnet.service /etc/systemd/system/
sudo cp zcashd_testnet.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable zcashd_mainnet
sudo systemctl enable zcashd_testnet
```


The zcashd server is now configured so that it can be started or stopped on either the mainnet or testnet just like any other service on your Debian workstation. The 'sudo systemctl enable' commands we issue instruct systemd to start zcashd when you reboot. 


## Step 3 - Add ZcashD logrotate setup


Having the log files generated by zcashd stored and rotated along with the rest of your system logs is useful especially if you leave your workstation running for a long time.


### Prepare a location for the logs


We will prepare a location for the logs and create symlinks to the default locations:


```bash
sudo mkdir /var/log/zcashd
sudo chown -R yourUsername:yourUsername /var/log/zcashd
touch /var/log/zcashd/db.mainnet.log
touch /var/log/zcashd/debug.mainnet.log
touch /var/log/zcashd/db.testnet.log
touch /var/log/zcashd/debug.testnet.log
ln -s /var/log/zcashd/db.mainet.log ~/.zcash/db.log
ln -s /var/log/zcashd/debug.mainet.log ~/.zcash/debug.log
mkdir ~/.zcash/testnet3
ln -s /var/log/zcashd/debug.testnet.log ~/.zcash/testnet3/debug.log
ln -s /var/log/zcashd/db.testnet.log ~/.zcash/testnet3/db.log
```


*Note*: We could have added a 'debuglogfile' entry to our [zcash.conf.mainnet](zcash.conf.mainnet) and [zcash.conf.testnet](zcash.conf.testnet) files but I prefer to use symlinks on my workstation.


### Add a logrotate configuration

Copy over the provided logrotate files.


```bash
sudo cp zcashd.logrotate /etc/logrotate.d/zcashd
```


## Step 4 - Start Zcashd


Earlier we enabled zcashd as a systemd services for mainnet and testnet. We are going to start those services now!


### Mainnet

Let's start zcashd on mainnet first:


```bash
sudo systemctl start zcashd_mainnet
tail -f ~/.zcash/debug.log
```


You will now see the log files being generated by the zcashd server for mainnet. 'Ctrl-c' to exit logging.

Now what is required is patience! It could takes a very long time, days even, to fully syncronize the mainnet blockchain. Most commands, including RPC commands, will not be available until the mainnet blockchain is fully syncronized.


### Testnet


Repeat for testnet. Notice that testnet files are in a ~/.zcash/testnet3; a subdirectory of ~/.zcash

**Note**: You CAN start zcashd on testnet BEFORE mainnet has finished syncronizing.


```bash
sudo systemctl start zcashd_testnet
tail -f ~/.zcash/testnet3/debug.log
```


Even though the testnet blockchain is much smaller than the mainnet blockchain it can still take quite a while to syncronize the testnet blockchain. Most commands, including RPC commands, will not be available until the testnet blockchain is fully syncronized. Again, patience.


## Step 4 - Zcashd Files


With the blockchain(s) fully syncronized one will see a number of new files and folders created under ~/.zcash. Let's take a look!


### Log Files


With both zcashd configurations running your ~./zcash/debug.log and ~./zcash/testnet3/debug.log files should be filling. 

You can watch these log files being appended in real time which comes in handy when you use more verbose debug settings in your configuration, e.g. 'debug=all'.



### File Structure


Your directory tree will look something like the tree structure below except that you will have MANY MANY *dat* and *ldb* files.


```
/home/yourUsername/.zcash
├── banlist.dat
├── blocks
│   ├── blk00000.dat
│   ├── ...
│   ├── index
│   │   ├── 000123.ldb
│   │   ├── ...
│   │   ├── CURRENT
│   │   ├── LOCK
│   │   ├── LOG
│   │   ├── LOG.old
│   │   └── MANIFEST-405383
│   ├── rev00000.dat
│   ├── ...
├── chainstate
│   ├── 000123.ldb
│   ├── ...
│   ├── CURRENT
│   ├── LOCK
│   ├── LOG
│   ├── LOG.old
│   └── MANIFEST-1093374
├── database
│   └── log.0000000123
├── db.log -> /var/log/zcashd/db.mainnet.log
├── debug.log -> /var/log/zcashd/debug.mainnet.log
├── fee_estimates.dat
├── peers.dat
├── testnet3
│   ├── banlist.dat
│   ├── blocks
│   │   ├── blk00000.dat
│   │   ├── ...
│   │   ├── index
│   │   │   ├── 000123.ldb
│   │   │   ├── ...
│   │   │   ├── CURRENT
│   │   │   ├── LOCK
│   │   │   ├── LOG
│   │   │   ├── LOG.old
│   │   │   └── MANIFEST-041811
│   │   ├── rev00000.dat
│   │   ├── ...
│   ├── chainstate
│   │   ├── 000123.ldb
│   │   ├── ...
│   │   ├── CURRENT
│   │   ├── LOCK
│   │   ├── LOG
│   │   ├── LOG.old
│   │   └── MANIFEST-005528
│   ├── database
│   │   └── log.0000012345
│   │   └── ...
│   ├── db.log -> /var/log/zcashd/db.testnet.log
│   ├── debug.log -> /var/log/zcashd/debug.testnet.log
│   ├── fee_estimates.dat
│   ├── peers.dat
│   ├── testnet3
│   │   ├── banlist.dat
│   │   ├── blocks
│   │   │   ├── blk00000.dat
│   │   │   ├── ...
│   │   │   ├── index
│   │   │   │   ├── 000123.ldb
│   │   │   │   ├── ...
│   │   │   │   ├── CURRENT
│   │   │   │   ├── LOCK
│   │   │   │   ├── LOG
│   │   │   │   ├── LOG.old
│   │   │   │   └── MANIFEST-032489
│   │   │   ├── rev00000.dat
│   │   │   ├── ...
│   │   ├── chainstate
│   │   │   ├── 000123.ldb
│   │   │   ├── ...
│   │   │   ├── CURRENT
│   │   │   ├── LOCK
│   │   │   ├── LOG
│   │   │   ├── LOG.old
│   │   │   └── MANIFEST-004291
│   │   ├── database
│   │   │   └── log.0000000123
│   │   ├── fee_estimates.dat
│   │   ├── peers.dat
│   │   └── zcashd.pid
│   ├── wallet.dat
│   └── zcashd_testnet.pid
├── wallet.dat
├── zcash -> .zcash
├── zcash.conf -> zcash.conf.mainnet
├── zcash.conf.mainnet
├── zcash.conf.testnet
└── zcash_mainnet.pid
```


## Wallet Files

Of all the files found under ~./zcash the **MOST** important are your wallet.dat files. These files, and their contents, must be protected!


```
/home/yourUsername/.zcash
├── testnet3
│   ├── wallet.dat
├── wallet.dat
```


### Wallet File Discussion


These data can files can be protected in a number of ways and it is best to do them all!


 - Backup these files themselves
  - Copy to a safe external media
  - Copy to a SECURE network location
- Issue the backup command to create a zcashd backup file
  - Copy that generated HUMAN READABLE file to external media
  - Copy that generated HUMAN READABLE file to a SECURE network location
  - Issue a backup command EVERY TIME you create a new address
    - The more often the better!
    - The backup file is put into the directory associated with the 'backupdir' setting. 
        - *Double check that 'backupdir' points to a secure directory!!* 
- Create a 'Recovery Phrase'
  - Store this recovery phrase someplace safe


We're going to address some of these safety/security items in a subsequent section.

**Note**: If you move your wallet.dat files OUT of their current locations and restart the zcashd_mainnet and zcashd_testnet services the zcashd server will create NEW wallet.dat files!


### Look at the Wallet


Look at the mainnet wallet first


```bash
zcash-cli getwalletinfo 
zcash-cli listaddresses
```


The output of these commands will look something like


```
{
  "walletversion": 60000,
  "balance": 0.00000000,
  "unconfirmed_balance": 0.00000000,
  "immature_balance": 0.00000000,
  "shielded_balance": "0.00",
  "shielded_unconfirmed_balance": "0.00",
  "txcount": 0,
  "keypoololdest": 1675120114,
  "keypoolsize": 101,
  "paytxfee": 0.00000000,
  "mnemonic_seedfp": "****************************************************************" **REDACTED**
}
[
  {
    "source": "mnemonic_seed",
    "transparent": {
      "addresses": [
        "t1*********************************" **REDACTED**
      ]
    }
  }
]
```


You will see some information about the wallet including the "mnemonic seed" and the single TRANSPARENT address in the wallet.

Try to avoid using your 'transparent' or 'T' address as transactions are visible. As a matter of fact avoid send OR receiving to 'transparent' addresses. Still, there might be a circumstance where you need a transparent address so you have one.


## Wallet Addresses


We are going to add TWO new additional addresses.

The first will be a 'Z' address which is an encrypted address meaning to/from other encrypted 'Z' addresses are private.


```bash
zcash-cli z_getnewaddress
```


The second will be a 'U' address which is a 'unifed' address.

'U' addresses are future proof. As the zcash blockchain evolves and 'unifies' transparent and subsequent types of encrypted transactions the 'U' addresses should prove to be resilient as new technologies are introduced. These new 'U' addresses should prove very handy in simplifying the transparent/encrypted complexity form the end user of your application as the zcash blockchain ecosystem goes through inevitable evolution. It is a very activy project after all. 

'U' addresses appear to be the future of the zcash addressing scheme and if you are doing development against the zcash blockchain I recommend getting familiar with the ins and outs of sending transactions using a unified address. Especially user feedback on when transactions could be attempting to utilize one or more TRANSPARENT [utxos](https://www.investopedia.com/terms/u/utxo.asp) which could result in a transaction being public! 

**Note**: The 'z_sendmany' RPC method has a 'privacyPolicy' parameter that seems to work well with 'U' addresses to help prevent mistakes.

[Unified Addresses](https://electriccoin.co/blog/unified-addresses-in-zcash-explained) are explained [here](https://electriccoin.co/blog/unified-addresses-in-zcash-explained) and [here](https://github.com/zcash/zips/issues/470) and [here](https://zips.z.cash/protocol/nu5.pdf#unifiedpaymentaddrencoding) and are beyond the scope of this guide.


```bash
zcash-cli z_getnewaccount
zcash-cli z_getaddressforaccount 0
```


Let's view the wallet AFTER we have added a new 'Z' and 'U' address.


```bash
zcash-cli listaddresses
```


The output of the addresses stored in the wallet now looks more like


```
[
  {
    "source": "mnemonic_seed",
    "transparent": {
      "addresses": [
        "t1RBPsGfbyBEpBJLsjha1ryPbh5swo1h7YU"
        "t**********************************" **REDACTED**
      ]
    },
    "sapling": [
      {
        "zip32KeyPath": "m/32'/133'/2147483647'/0'",
        "addresses": [
          "zs****************************************************************************" **REDACTED**
        ]
      }
    ],
    "unified": [
      {
        "account": 0,
        "seedfp": "****************************************************************", **REDACTED**
        "addresses": [
          {
            "diversifier_index": 0,
            "receiver_types": [
              "p2pkh",
              "sapling",
              "orchard"
            ],
            "address": "u1*******************************************************************************************************************************************************************************************************************" **REDACTED**
          }
        ]
      }
    ]
  }
]

```

You will see you now have a **T (transparent)**, **Z (encrypted)**, and **U (unifed)** address. Or, put another way... our wallet.dat files has an address that represents the past, present and future proof addresses of the zcash blockchain.

Now repeat the above for testnet using port 18232 and send some RPC commands using curl with a json payload. This should also provide some hints for when you integrate zcash with your application using REST calls.


```bash
curl --user mySecureUsername --data-binary '{"jsonrpc": "1.0", "id":"curltest1", "method": "z_getnewaddress", "params": [] }' -H 'content-type: text/plain;' http://127.0.0.1:18232/;
curl --user mySecureUsername --data-binary '{"jsonrpc": "1.0", "id":"curltest2", "method": "z_getnewaccount", "params": [] }' -H 'content-type: text/plain;' http://127.0.0.1:18232/;
curl --user mySecureUsername --data-binary '{"jsonrpc": "1.0", "id":"curltest3", "method": "z_getaddressforaccount", "params": [0] }' -H 'content-type: text/plain;' http://127.0.0.1:18232/;
curl --user mySecureUsername --data-binary '{"jsonrpc": "1.0", "id":"curltest1", "method": "z_getwalletinfo", "params": [] }' -H 'content-type: text/plain;' http://127.0.0.1:18232/;
curl --user mySecureUsername --data-binary '{"jsonrpc": "1.0", "id":"curltest1", "method": "listaddresses", "params": [] }' -H 'content-type: text/plain;' http://127.0.0.1:18232/;
```

You can see that 'method' section of the json contains exactly the same command/options that we used with the **zcash-cli** tool.


## Step 5 - Wallet Safety


As discussed above it is important to protect the wallet.dat files. Backup up these files as you would backup any other important file on your system. If your wallet.dat file is on mainnet and contains currency BE EXTRA diligent in taking precautions. 

I'd recommend backing up using the RPC command (or zcash-cli) anytime you add a new address - AT THE VERY LEAST.


### Zcashd Backup Command


The 'walletrequirebackup' option is set to FALSE in our [zcash.conf.mainnet](zcash.conf.mainnet) and [zcash.conf.testnet](zcash.conf.testnet) this option should be set to TRUE. Let's backup our wallet.dat file, set this option to TRUE and restart our mainnet service.

On your development machine I recommend the naming convention wallet[Network][YYMMDD] so for 2021-Jan-01 I would enter:


```bash
zcash-cli backupwallet walletMainnet20210101; 
```


which is the same as 


```bash
curl --user myusername --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "z_exportwallet", "params": ["walletMainnet20210101"] }' -H 'content-type: text/plain;' http://127.0.0.1:8232/
```


we can adapt this for testnet by changing the port (and changing the backup file name using Testnet)


```bash
url --user myusername --data-binary '{"jsonrpc": "1.0", "id":"curltest", "method": "z_exportwallet", "params": ["walletTestnet20210101"] }' -H 'content-type: text/plain;' http://127.0.0.1:18232/
```


### Zcashd Recover Phrase


To obtain a recovery phrase we will use a tool that was installed when along with the zcashd server; zcashd-wallet-tool.

Simply issue the command and follow the instructions:


```bash
zcashd-wallet-tool
```
**COPY THIS PASS PHRASE TO A SAFE PLACE!**


## Final Thoughts and Discussion


### Dedicated Full Node

Above I mention that is useful to also have a dedicated full node on your LAN. Most of this guide could be helpful but there could/should be some differences I would consider.

- Apply a really good harding to the operating system even BEFORE attempting to configure zcash!!!
  - [Securing Debian](https://www.debian.org/doc/manuals/securing-debian-manual/index.en.html)

- The firewall configuration would BLOCK the rpc ports facing the INTERNET; 8232 and 18232
- The firewall configuration would ALLOW the rpc ports facing the LAN; 8232 and 18232
- The rpcuser, rpcpassword and rpcauth would be unique to that system

**Even better... NO RPC AT ALL!!!**

- The wallet.dat files on BOTH mainnet and testnet would contain NO coins and the files/addresses would be considered disposable
  - Still backed up just in case!
  - zcashd-wallet-tool run against both wallets just in case!
- All of this would be setup using a regular group also called zcash
  -  groupadd -g 2001 zcash
- All of this would be setup using a regular user called zcash; no login allowed!
  - useradd zcash -u 2001 -g 2001 --no-create-home -s /usr/sbin/nologin
- [TOR](https://www.torproject.org/) would also be used following the directions posted [here](https://zcash.readthedocs.io/en/latest/rtd_pages/tor.html) once version 3 was available
- Perhaps the configuration files would reside in a subdirectory of /etc
    - /etc/zcashd
  - 'debuglogfile' options would point directly to files under /var/log/zcashd/
  - 'datadir' would point to a subdirectory of /var
    - /var/zcashd
  - debug off

- On the workstation(s) on my LAN would use the 'connect' option using LAN IP address for this dedicated machine
  
The biggest benefits would be

- If/when I blew out or damaged my ~./zcash blockchain data - which I have done - during the course of development activities one could copy down the relevant files/folders to again bootstrap the blockchain syncronization activity on the workstation 
  - This cuts the time down from Days to Minutes on a slow connection
- If you work on team, as I do, more than one workstation can connect to this dedicated server.
  - This can ease the burden on a slow WAN connection quite a bit
- Supplying a node to the blockchain is always useful if you have the resources


## Additional Information Sources


- [Zcash Documentation](https://zcash.readthedocs.io/en/latest/)
- [Electric Coin Company](https://electriccoin.co/)
- [Zcash Github](https://github.com/zcash/zcash)
- [Zcash Forums](https://forum.zcashcommunity.com/)
- [Debian Adminsitrator's Handbook](https://debian-handbook.info/)


## Other Development Environment Alternatives


- Docker is always nice
  - [Zcash Docker](https://hub.docker.com/r/electriccoinco/zcashd)
  - A great alternative BUT one still has to deal with blockchain syncronization so a local install was better for my needs.
  - YMMV


## This Document

- Feedback welcome
- Pull requests welcome
- Original Located at [Github](https://github.com/gesker/zcashd_config)

I hope you found these notes useful, getting zcashd setup relatively quickly and can back to coding your zcash application!


