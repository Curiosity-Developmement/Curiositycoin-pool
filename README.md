


Curiositycoincoin-pool (for NodeJS 10)
====================
Formerly known as turtle-pool, which in turn was known as cryptonote-forknote-pool, forked from Forknote Project.

High performance Node.js (with native C addons) mining pool for Cryptonote based coins, created with the Forknote software such as Bytecoin, Dashcoin, etc..

Comes with lightweight example front-end script which uses the pool's AJAX API.

#### Table of Contents
* [Features](#features)
* [Community Support](#community--support)
* [Pools Using This Software](#pools-using-this-software)
* [Usage](#usage)
  * [Requirements](#requirements)
  * [Downloading & Installing](#1-downloading--installing)
  * [Configuration](#2-configuration)
  * [Configure Easyminer](#3-optional-configure-cryptonote-easy-miner-for-your-pool)
  * [Starting the Pool](#4-start-the-pool)
  * [Host the front-end](#5-host-the-front-end)
  * [Customizing your website](#6-customize-your-website)
  * [Upgrading](#upgrading)
* [Setting up Testnet](#setting-up-testnet)
* [JSON-RPC Commands from CLI](#json-rpc-commands-from-cli)
* [Monitoring Your Pool](#monitoring-your-pool)
* [Configuring Blockchain Explorer](#configuring-blockchain-explorer)
* [Credits](#credits)
* [License](#license)


#### Basic features

* TCP (stratum-like) protocol for server-push based jobs
  * Compared to old HTTP protocol, this has a higher hash rate, lower network/CPU server load, lower orphan
    block percent, and less error prone
* IP banning to prevent low-diff share attacks
* Socket flooding detection
* Payment processing
  * Splintered transactions to deal with max transaction size
  * Minimum payment threshold before balance will be paid out
  * Minimum denomination for truncating payment amount precision to reduce size/complexity of block transactions
* Detailed logging
* Ability to configure multiple ports - each with their own difficulty
* Variable difficulty / share limiter
* Share trust algorithm to reduce share validation hashing CPU load
* Clustering for vertical scaling
* Modular components for horizontal scaling (pool server, database, stats/API, payment processing, front-end)
* Live stats API (using AJAX long polling with CORS)
  * Currency network/block difficulty
  * Current block height
  * Network hashrate
  * Pool hashrate
  * Each miners' individual stats (hashrate, shares submitted, pending balance, total paid, etc)
  * Blocks found (pending, confirmed, and orphaned)
* An easily extendable, responsive, light-weight front-end using API to display data

#### Extra features

* Admin panel
  * Aggregated pool statistics
  * Coin daemon & wallet RPC services stability monitoring
  * Log files data access
  * Users list with detailed statistics
* Historic charts of pool's hashrate and miners count, coin difficulty, rates and coin profitability
* Historic charts of users's hashrate and payments
* Miner login(wallet address) validation
* Five configurable CSS themes
* Universal blocks and transactions explorer based on [chainradar.com](http://chainradar.com)
* FantomCoin & MonetaVerde support
* Set fixed difficulty on miner client by passing "address" param with ".[difficulty]" postfix
* Prevent "transaction is too big" error with "payments.maxTransactionAmount" option


### Community / Support

* [CryptoNote Technology](https://cryptonote.org)
* [CryptoNote Forum](https://forum.cryptonote.org/)
* [CryptoNote Universal Pool Forum](https://bitcointalk.org/index.php?topic=705509)
* [Forknote](https://forknote.net)
* [TurtleCoin](http://chat.turtlecoin.lol)

#### Pools Using This Software

* http://democats.org
* http://cryptonotepool.com/

Usage
===

#### Requirements
* Curiositycoin node daemon
* Curiositycoin-service
* [Node.js](http://nodejs.org/) LTS (6,8,10) ([follow these installation instructions](https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions))
* [Redis](http://redis.io/) key-value store v2.6+ ([follow these instructions](http://redis.io/topics/quickstart))
* libssl required for the node-multi-hashing module
  * For Ubuntu: `sudo apt-get install -y libssl-dev`

##### Windows Support

You will need the windows build tools to install this module (and many more) on windows. Run the following command to set up your environment.

```bash
npm install -g windows-build-tools --vs2015
```

##### Seriously
Those are legitimate requirements. If you use old versions of Node.js or Redis that may come with your system package manager then you will have problems. Follow the linked instructions to get the last stable versions.

[**Redis security warning**](http://redis.io/topics/security): be sure firewall access to redis - an easy way is to
include `bind 127.0.0.1` in your `redis.conf` file. Also it's a good idea to learn about and understand software that
you are using - a good place to start with redis is [data persistence](http://redis.io/topics/persistence).


##### Debian 9/Ubuntu 18 installation
These are the steps taken to install pool on Debian 9.  These steps will also work on Ubuntu 16 & 18:

Always start with:

```bash
sudo apt update
sudo apt upgrade
```

The packages need for the pool software

```bash
sudo apt-get install -y git curl wget screen build-essential redis-server libboost-all-dev cmake libssl-dev
```
I have currently tested this on Node 10.22.0

You can install node here: (https://nodejs.org/en/download/package-manager/)

Or directly from a terminal:

```bash
curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
sudo apt-get install -y nodejs
```

I have found using a screen session to keep everything running on the server works well.

Grab the most recent Curiositycoin release here https://github.com/Curiosity-Developmement/CuriosityCoin/releases then launch your daemon and sync your chain.
Or compile the source: https://github.com/Curiosity-Developmement/CuriosityCoin


Once your daemon is synced with the network start your curiositycoin-service and redis-server.

#### 1) Downloading & Installing

Clone the repository and run `npm install` for all the dependencies to be installed:

```bash
git clone https://github.com/Curiosity-Developmement/Curiositycoin-pool
cd Curiosity-pool
npm install && npm test
```

To fix vulnerabilities and remove some packges not needed run:

```bash
npm audit fix
```

#### 2) Configuration


Explanation for each field:
```javascript
/* Used for storage in redis so multiple coins can share the same redis instance. */
"coin": "Curiositycoin",

/* Used for front-end display */
"symbol": "CURI",

/* Minimum units in a single coin, see COIN constant in DAEMON_CODE/src/cryptonote_config.h */
"coinUnits": 100000,

/* Coin network time to mine one block, see DIFFICULTY_TARGET constant in DAEMON_CODE/src/cryptonote_config.h */
"coinDifficultyTarget": 60,

"logging": {

    "files": {

        /* Specifies the level of log output verbosity. This level and anything
           more severe will be logged. Options are: info, warn, or error. */
        "level": "info",

        /* Directory where to write log files. */
        "directory": "logs",

        /* How often (in seconds) to append/flush data to the log files. */
        "flushInterval": 5
    },

    "console": {
        "level": "info",
        /* Gives console output useful colors. If you direct that output to a log file
           then disable this feature to avoid nasty characters in the file. */
        "colors": true
    }
},

/* Modular Pool Server */
"poolServer": {
    "enabled": true,

    /* Set to "auto" by default which will spawn one process/fork/worker for each CPU
       core in your system. Each of these workers will run a separate instance of your
       pool(s), and the kernel will load balance miners using these forks. Optionally,
       the 'forks' field can be a number for how many forks will be spawned. */
    "clusterForks": "auto",

    /* Address where block rewards go, and miner payments come from. */
    "poolAddress": "<adress>"

    /* Poll RPC daemons for new blocks every this many milliseconds. */
    "blockRefreshInterval": 1000,

    /* How many seconds until we consider a miner disconnected. */
    "minerTimeout": 900,

    "ports": [
        {
            "port": 3333, //Port for mining apps to connect to
            "difficulty": 100, //Initial difficulty miners are set to
            "desc": "Low end hardware" //Description of port
        },
        {
            "port": 5555,
            "difficulty": 2000,
            "desc": "Mid range hardware"
        },
        {
            "port": 7777,
            "difficulty": 10000,
            "desc": "High end hardware"
        }
    ],

    /* Variable difficulty is a feature that will automatically adjust difficulty for
       individual miners based on their hashrate in order to lower networking and CPU
       overhead. */
    "varDiff": {
        "minDiff": 2, //Minimum difficulty
        "maxDiff": 100000,
        "targetTime": 100, //Try to get 1 share per this many seconds
        "retargetTime": 30, //Check to see if we should retarget every this many seconds
        "variancePercent": 30, //Allow time to very this % from target without retargeting
        "maxJump": 100 //Limit diff percent increase/decrease in a single retargeting
    },

    /* Set difficulty on miner client side by passing <address> param with .<difficulty> postfix
       minerd -u D3z2DDWygoZU4NniCNa4oMjjKi45dC2KHUWUyD1RZ1pfgnRgcHdfLVQgh5gmRv4jwEjCX5LoLERAf5PbjLS43Rkd8vFUM1m.5000 */
    "fixedDiff": {
        "enabled": true,
        "separator": ".", // character separator between <address> and <difficulty>
    },

    /* Feature to trust share difficulties from miners which can
       significantly reduce CPU load. */
    "shareTrust": {
      "enabled": false, //enable or disable the shareTrust system. shareTrust can offer significant CPU workload reduction, however does present a risk of being exploited by miners gaming the percentages of the system.
      "maxTrustPercent": 50, //The maximum percent chance a share will be considered trusted (not fully validated) 50 means 1 of 2 shares are fully validated at random, 75 means 1 of 4 are fully validated (or 3 of 4 are trusted).
      "probabilityStepPercent": 1, //The percent the probabality of a share is trusted increases from 0 to maxTrustPercent at a maximum rate of once per probabilityStepWindow seconds in steps of probabilityStepPercent and only on share submission.
      "probabilityStepWindow": 30, //The probability (chance a share is considered trusted) will increase from 0 to maxTrustPercent by steps of probabilityStepPercent at a maximum rate of once every probabilityStepWindow seconds.
      "minUntrustedShares": 50, //The minimum amount of shares that will be fully validated before shareTrust will begin.
      "minUntrustedSeconds": 300, //The minimum amount of time in seconds shares will be fully validated before shareTrust will begin.
      "maxTrustedDifficulty": 100000, //Shares above this difficulty will be fully validated (not trusted).
      "maxPenaltyMultiplier": 100, //The maximum penalty multiplied against minUntrustedShares and minUntrustedSeconds.
      "minPenaltyMultiplier": 2, //The minimum penalty multiplied against minUntrustedShares and minUntrustedSeconds.
      "penaltyMultiplierStep": 1, //The penalty is multiplied against minUntrustedShares and minUntrustedSeconds. The penalty Steps up/down penaltyMultiplierStep a maximum of once per every penaltyStepUpWindow or penaltyStepDownWindow and only on share submission.
      "penaltyStepUpWindow": 30, //The penalty steps up a maximum of penaltyMultiplierStep every penaltyStepUpWindow seconds and only on share submission.
      "penaltyStepDownWindow": 120, //The penalty steps down a maximum of penaltyMultiplierStep every penaltyStepDownWindow seconds and only on share submission.
      "maxShareWindow": 300, //Must Submit within this window or minUntrustedSeconds, minUntrustedShares and Probability are reset.
      "maxIPCRate": 15, //The minimum amount of seconds between sharing a miners shareTrust data between pool threads.
      "maxAge": 604800 //Maximum seconds to retain dissconnected miner shareTrust data in memory.
    },

    /* If under low-diff share attack we can ban their IP to reduce system/network load. */
    "banning": {
        "enabled": true,
        "time": 600, //How many seconds to ban worker for
        "invalidPercent": 25, //What percent of invalid shares triggers ban
        "checkThreshold": 30 //Perform check when this many shares have been submitted
    },
    /* Slush Mining is a reward calculation technique which disincentivizes pool hopping and rewards users to mine with the pool steadily: Values of each share decrease in time – younger shares are valued higher than older shares.
    More about it here: https://mining.bitcoin.cz/help/#!/manual/rewards */
    /* There is some bugs with enabled slushMining. Use with '"enabled": false' only. */

    "slushMining": {
        "enabled": false, // 'true' enables slush mining. Recommended for pools catering to professional miners
        "weight": 120, //defines how fast value assigned to a share declines in time
        "lastBlockCheckRate": 1 //How often the pool checks for the timestamp of the last block. Lower numbers increase load for the Redis db, but make the share value more precise.
    }
},

/* Module that sends payments to miners according to their submitted shares. */
"payments": {
    "enabled": true,
    "interval": 600, //how often to run in seconds
    "mixin": 1, // The mixin count refers to the number of other signatures (aside from yours) in the ring signature that authorizes the transaction
    "maxAddresses": 50, //split up payments if sending to more than this many addresses
    "transferFee": 100, //fee to pay for each transaction
    "minPayment": 1000000, //miner balance required before sending payment
    "maxTransactionAmount": 100000000, //split transactions by this amount(to prevent "too big transaction" error)
    "denomination": 100 //truncate to this precision and store remainder
},

/* Module that monitors the submitted block maturities and manages rounds. Confirmed
   blocks mark the end of a round where workers' balances are increased in proportion
   to their shares. */
"blockUnlocker": {
    "enabled": true,
    "interval": 30, //how often to check block statuses in seconds

    /* Block depth required for a block to unlocked/mature. Found in daemon source as
       the variable CRYPTONOTE_MINED_MONEY_UNLOCK_WINDOW */
    "depth": 60,
    "poolFee": 1.8, //1.8% pool fee (2% total fee total including donations)
    "devDonation": 0.1, //0.1% donation to send to pool dev - only works with Monero
    "coreDevDonation": 0.1 //0.1% donation to send to core devs - works with Bytecoin, Monero, Dashcoin, QuarazCoin, Fantoncoin, AEON and OneEvilCoin
},

/* AJAX API used for front-end website. */
"api": {
    "enabled": true,
    "hashrateWindow": 600, //how many second worth of shares used to estimate hash rate
    "updateInterval": 3, //gather stats and broadcast every this many seconds
    "host": "127.0.0.1", //if api module is running on a different host (i.e, containerized),
    "port": 8117,
    "blocks": 30, //amount of blocks to send at a time
    "payments": 30, //amount of payments to send at a time
    "password": "test" //password required for admin stats
},

/* Coin daemon connection details. */
"daemon": {
    "host": "127.0.0.1",
    "port": 11898
},

/* Wallet daemon connection details. */
"wallet": {
    "host": "127.0.0.1",
    "port": 8070,
    "password": "<replace with rpc password>"
},

/* Redis connection into. */
"redis": {
    "host": "127.0.0.1",
    "port": 6379
}

/* Monitoring RPC services. Statistics will be displayed in Admin panel */
"monitoring": {
    "daemon": {
        "checkInterval": 60, //interval of sending rpcMethod request
        "rpcMethod": "getblockcount" //RPC method name
    },
    "wallet": {
        "checkInterval": 60,
        "rpcMethod": "get_address_count"
    }

/* Collect pool statistics to display in frontend charts  */
"charts": {
    "pool": {
        "hashrate": {
            "enabled": true, //enable data collection and chart displaying in frontend
            "updateInterval": 60, //how often to get current value
            "stepInterval": 1800, //chart step interval calculated as average of all updated values
            "maximumPeriod": 86400 //chart maximum periods (chart points number = maximumPeriod / stepInterval = 48)
        },
        "workers": {
            "enabled": true,
            "updateInterval": 60,
            "stepInterval": 1800, //chart step interval calculated as maximum of all updated values
            "maximumPeriod": 86400
        },
        "difficulty": {
            "enabled": true,
            "updateInterval": 1800,
            "stepInterval": 10800,
            "maximumPeriod": 604800
        },
        "price": { //USD price of one currency coin received from cryptonator.com/api
            "enabled": true,
            "updateInterval": 1800,
            "stepInterval": 10800,
            "maximumPeriod": 604800
        },
        "profit": { //Reward * Rate / Difficulty
            "enabled": true,
            "updateInterval": 1800,
            "stepInterval": 10800,
            "maximumPeriod": 604800
        }
    },
    "user": { //chart data displayed in user stats block
        "hashrate": {
            "enabled": true,
            "updateInterval": 180,
            "stepInterval": 1800,
            "maximumPeriod": 86400
        },
        "payments": { //payment chart uses all user payments data stored in DB
            "enabled": true
        }
    }
```

#### 3) [Optional] Configure cryptonote-easy-miner for your pool
Your miners that are Windows users can use [cryptonote-easy-miner](https://github.com/zone117x/cryptonote-easy-miner)
which will automatically generate their wallet address and start up multiple threads of simpleminer. You can download
it and edit the `config.ini` file to point to your own pool.
Inside the `easyminer` folder, edit `config.init` to point to your pool details
```ini
pool_host=example.com
pool_port=5555
```

Rezip and upload to your server or a file host. Then change the `easyminerDownload` link in your `config.json` file to
point to your zip file.

#### 4) Start the pool

```bash
node init.js
```

The file `config.json` is used by default but a file can be specified using the `-config=file` command argument, for example:

```bash
node init.js -config=config_backup.json
```

This software contains four distinct modules:
* `pool` - Which opens ports for miners to connect and processes shares
* `api` - Used by the website to display network, pool and miners' data
* `unlocker` - Processes block candidates and increases miners' balances when blocks are unlocked
* `payments` - Sends out payments to miners according to their balances stored in redis


By default, running the `init.js` script will start up all four modules. You can optionally have the script start
only start a specific module by using the `-module=name` command argument, for example:

```bash
node init.js -module=api
```

[Example screenshot](http://i.imgur.com/SEgrI3b.png) of running the pool in single module mode with tmux.


#### 5) Host the front-end

Simply host the contents of the `website_example` directory on file server capable of serving simple static files.


Edit the variables in the `website_example/config.js` file to use your pool's specific configuration.
Variable explanations:

```javascript

/* Must point to the API setup in your config.json file. */
var api = "http://poolhost:8117";

/* Pool server host to instruct your miners to point to.  */
var poolHost = "poolhost.com";

/* IRC Server and room used for embedded KiwiIRC chat. */
var irc = "irc.freenode.net/#forknote";

/* Contact email address. */
var email = "support@poolhost.com";

/* Market stat display params from https://www.cryptonator.com/widget */
var cryptonatorWidget = ["DSH-BTC", "DSH-USD", "DSH-EUR"];

/* Download link to cryptonote-easy-miner for Windows users. */
var easyminerDownload = "https://github.com/zone117x/cryptonote-easy-miner/releases/";

/* Used for front-end block links. */
var blockchainExplorer = "http://chainradar.com/{symbol}/block/{id}";

/* Used by front-end transaction links. */
var transactionExplorer = "http://chainradar.com/{symbol}/transaction/{id}";

/* Any custom CSS theme for pool frontend */
var themeCss = "themes/default-theme.css";

```

#### 6) Customize your website

You can customize your website by changing the pool name in pages/home.html at line 113. 
The website includes dark mode / light mode by default that will automatically adjust according to the users OS preference so there is no need to switch css themes anymore. 


Then simply serve the files via nginx, Apache, Google Drive, or anything that can host static content.


#### Upgrading
When updating to the latest code its important to not only `git pull` the latest from this repo, but to also update
the Node.js modules, and any config files that may have been changed.
* Inside your pool directory (where the init.js script is) do `git pull` to get the latest code.
* Remove the dependencies by deleting the `node_modules` directory with `rm -r node_modules`.
* Run `npm update` to force updating/reinstalling of the dependencies.
* Compare your `config.json` to the latest example ones in this repo or the ones in the setup instructions where each config field is explained. You may need to modify or add any new changes.

### Setting up Testnet

No cryptonote based coins have a testnet mode (yet) but you can effectively create a testnet with the following steps:

* Open `/src/p2p/net_node.inl` and remove lines with `ADD_HARDCODED_SEED_NODE` to prevent it from connecting to mainnet (Monero example: http://git.io/0a12_Q)
* Build the coin from source
* You now need to run two instance of the daemon and connect them to each other (without a connection to another instance the daemon will not accept RPC requests)
  * Run first instance with `./forknoted --p2p-bind-port 28080 --allow-local-ip`
  * Run second instance with `./forknoted --p2p-bind-port 5011 --rpc-bind-port 5010 --add-peer 0.0.0.0:28080 --allow-local-ip`
* You should now have a local testnet setup. The ports can be changes as long as the second instance is pointed to the first instance, obviously

*Credit to surfer43 for these instructions*


### JSON-RPC Commands from CLI

Documentation for JSON-RPC commands can be found here:
* Daemon https://wiki.bytecoin.org/wiki/Daemon_JSON_RPC_API
* Wallet https://wiki.bytecoin.org/wiki/Bytecoin_RPC_Wallet_API


Curl can be used to use the JSON-RPC commands from command-line. Here is an example of calling `getblockheaderbyheight` for block 100:

```bash
curl 127.0.0.1:18081/json_rpc -d '{"method":"getblockheaderbyheight","params":{"height":100}}'
```


### Monitoring Your Pool

* To inspect and make changes to redis I suggest using [redis-commander](https://github.com/joeferner/redis-commander)
* To monitor server load for CPU, Network, IO, etc - I suggest using [New Relic](http://newrelic.com/)
* To keep your pool node script running in background, logging to file, and automatically restarting if it crashes - I suggest using [forever](https://github.com/nodejitsu/forever)


### Configuring Blockchain Explorer

You need the latest stable version of Forknote for the blockchain explorer - [forknote releases](https://github.com/forknote/forknote/releases)
* Add the following code to the coin's config file:

```
rpc-bind-ip=0.0.0.0
enable-blockchain-indexes=1
enable-cors=*
```

* Launch forknoted with the corresponding config file
* Change the following line in the pool's frontend config.js:

```
var api_blockexplorer = "http://daemonhost.com:1118";
```

* Finally, edit these variables in the pool's frontend config.js using this syntax:

```
var blockchainExplorer = 'http://poolhost/?hash={id}#blockchain_block'

var transactionExplorer = 'http://poolhost/?hash={id}#blockchain_transaction'
```

Credits
===

* [LucasJones](//github.com/LucasJones) - Co-dev on this project; did tons of debugging for binary structures and fixing them. Pool couldn't have been made without him.
* [surfer43](//github.com/iamasupernova) - Did lots of testing during development to help figure out bugs and get them fixed
* [wallet42](http://moneropool.com) - Funded development of payment denominating and min threshold feature
* [Wolf0](https://bitcointalk.org/index.php?action=profile;u=80740) - Helped try to deobfuscate some of the daemon code for getting a bug fixed
* [Tacotime](https://bitcointalk.org/index.php?action=profile;u=19270) - helping with figuring out certain problems and lead the bounty for this project's creation
* [fancoder](https://github.com/fancoder/) - See his repo for the changes
* [TurtleCoin](https://github.com/turtlecoin/) - For making this great again

License
-------
Released under the GNU General Public License v2

http://www.gnu.org/licenses/gpl-2.0.html
