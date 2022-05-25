# USE AT OWN RISK, NOT ALL SCRIPTS ARE VERIFIED TO WORK 100% WITH LTO
# 
# LTO_LPoSDistributor
A revenue distribution tool for LTO.network nodes

Welcome to LTO.network version of the LPoSdistribution script,
which queues up multiple sessions and automates next session info :-)

Massive thank you to Rob / G1zm0 (https://lto.tools/generators/ and https://ltonode.com) and Sven Stam from LTO team for helping out and fixing bugs.

Many thanks to the version of Plukkie (https://github.com/plukkie/WavesLPoSDistributer), and the original version of Marc Jansen (https://github.com/jansenmarc/WavesLPoSDistributer) and the fork of W0utje (https://github.com/w0utje/WavesLPoSDistributer)!

You can lease to '3JqGGBMvkMtQQqNhGVD6knEzhncb55Y7JJ5' for Liquid Leasing Network or to Rob/G1zm0's ltonode.com : '3JnN8psLjuEyiPbH2bYcEFKUFpcamxzwFiv'

## Installation steps: prerequisits
First of all, you need to install Node.js (https://nodejs.org/en/) and npm.\
This version is succesfully tested with versions;
 - node v10.12.0 (allthough lower should work probably)
 - npm 6.4.1 (allthough lower should work probably)
 - tested on Ubuntu 14.0 with kernel 4.4.0-116-generic (allthough of minor importance)
 - get the latest version from github: git clone https://github.com/justmvg/LTO_LPoSDistributor.git

To install node.js and npm, do following steps;
 - add repository: curl -sL https://deb.nodesource.com/setup_10.x | sudo bash -
 - install both packages: sudo apt-get install -y nodejs

Now you can proceed with the lto LPOS scripts installation, select the variant which applies to you.

## Installation steps: first time users
These steps are for users that do not use an older version of the LTO_LPoSDistributor package yet.
1. CD into the LPoS package directory : LTO_LPoSDistributor
2. install the package independencies:
```sh
mkdir node_modules
npm install
```
3. configure for one time only the initial settings of the relevant blocks in the batchinfo.json file:
```sh
EDIT file batchinfo.json with vim or nano;

{
    "batchdata": {
        "paymentid": "1",				<== Leave as is
        "paystartblock": "1044012",			<== Put here same value as 'scanstartblock'. It's when payouts should start
        "paystopblock": "1050000",			<== Put here a value when payouts should stop (i.e. paystartblock+5000)
							    It doesn't really matter, as long as it is higher than paystartblock.
							    It only counts for the first run, and if no blocks were forged yet, that
							    is no problem. Follow up session results are just queued up in line :-))
        "scanstartblock": "1044012"			<== Put here the blockheight of the first ACTIVE lease
    }
}
```
NOTE\
This file is updated automatically after the collector session finishes.\
The size of the next batch (paystart / paystop blocks), is used from the\
'blockwindowsize' configuration value in the appng.js file.

4. EDIT file appng.js with vim or nano;
   This file is the collector that checks all blocks for leases and fees;
```sh
const myleasewallet = '<your node wallet>';		<== Put here the address of the wallet that your node uses
const myquerynode = "http://localhost:6869";		<== The node and API port that you use (defaults to localhost)
const feedistributionpercentage = 90;			<== How many % do you want to share with your leasers (defaults to 90%)
const specialfeedistributionpercentage = 95;    <== How many % do you want to share with specific addresses?
const blockwindowsize = 10000;				<== How many blocks to process for every subsequent paymentcycle.

var nofeearray = [ ];					<== Put here wallet addresses that you want to exclude from payments,
							    Default empty, so everyone get\'s payouts

var specialfeearray = [ ];					<== Put here wallet addresses that you want to process with special fees
```


5. EDIT file checkPaymentsFile.js with vim or nano;
```sh
var config = {
    <SNIP>,
    node: 'http://localhost:6869',			<== Change this value to your blockchain node/API port (defaults to localhost)
    <SNIP>
};
```
6. EDIT file **massPayment.js** and **masstx.js**
```sh
var config = {
    <SNIP>,
    node: 'http://localhost:6869',			<== Change this value to your blockchain node/API port (defaults to localhost)
    apiKey: 'your api key'				<== Put here the API key of your lto node
};
```
NOTE\
For security reasons, remove 'rwx' worldrights from massPayment.js and masstx.js -> ```chmod o-rwx massPayment.js``` \
Repeat for masstx.js. Now you can jump to chapter "Running the collector sessions".

## Running the collector sessions
After a successful configuration of the tool, start with:
```sh
node appng.js OR start_collector.sh
```
In Windows it's easier to just run:
```sh
node --stack-size=65565 --max-old-space-size=8192 appng.js
```
NOTE0\
If you can't start 'start_collector', check if the script has execute 'x' on it.\
If not add with: ```chmod u+x start_collector.sh```

NOTE1\
The script can consume a serious amount of memory and exits with errors during it's run.\
Therefore I've put 'start_collector.sh' script as starter which runs 'node appng.js' with some memory optimized settings.\
For me it works with tweaks to 65KB of stack memory and 8GB of available RAM. So use 'start_collector.sh' if you run into problems\
and tweak to your available RAM. If it keeps on exitting, then shrink the block batchsize that are collected in one batch.\
This way multiple smaller batchsizes will be collected and consume less memory.\
To decrease the initial batchsize, edit file 'batchinfo.json' and set 'paystopblock' smaller (closer to 'paystartblock').
To have all subsequent runs also changed, edit file 'appng.js' and set 'blockwindowsize' smaller.

NOTE2\
To run the collector tool every night @1 AM, edit /etc/crontab and put in following line;\
```sh
00 01 * * * root cd /home/myuser/LTO_LPoSDistributor/ && ./start_collector.sh
```
After the tool ran, it finishes up by writing the actual payments to be done into the file which is configured in the script by:
```sh
var config = {
	filename: 'ltoleaserpayouts',
```
The name is constructed together with the paymentid (or batchID) of every batch session.\
So, for the first run, the following three files will be created;
- ltoleaserpayouts1.json
- ltoleaserpayouts1.html
- ltoleaserpayouts1.log

The batchID is added to the payqueue.dat file. When there are already pending payments, it's just added.
For the next session, the batchid is incremented by 1 and the batchdata.json file is updated with the new blockheights and batchID.

## Checking pending payments
After the collector ran (or ran multiple times as you wish), you can check the payments that are stored in the payment queue.
The script for checking is checkPaymentsFile.js. After you configured some settings (see above), you can start with;
```sh
node checkPaymentsFile.js
```
The script reads all all batchIDs from the payqueue.dat file and the corresponding leaser files that were constructed by the collector tool. It does only checking, nothing else. The results for all pending payments is printed on the screen.\
The checker also calculates the cost for single transactions (payment tool massPayment.js) and the cost for masstransfers (masstx.js).
Often the number of transactions are high enough to benefit from masstransfers, which are often cheapest :-)
See more about both payment psossibilities in next chapter (doing the payments).
After checking this information, you have a good overview what tokens and the amounts are planned for payout and which transaction type is best to use!

## Doing the payments
For the actual payout, you can choose the massPayment.js tool or the masstx.js tool. They can be started with:
```sh
node massPayment.js or node masstx.js
```
The massPayment.js tool does a single transaction for every payment to be done.
The masstx.js tool makes use of masstransfers and pushes multiple payments for one one and the same asset
into one masstransfer transaction. This optimizes blockchain storage and and transaction costs.
If you run the checker first (checkPaymentsfile.js), you'll get a nice overview which method is cheapest for
for your payment batches. Both tools can just be used interchangelly.
All batchIDs are sequencially read from the payment queue and the transactions are executed.\
When a job finishes, the batchID is removed from the payqueue.dat and the three ltoleaserpayoutX.* files\
are moved to the archival directory (paymentsDone/).

NOTE massPayment.js\
If there would be a crash of the system, script or other transaction breaking interruption,\
make note of the last succesfull transaction counter and the batchID. Then edit the massPayment.js\
file and change these values for:
```sh
const crashconfig = {
        batchidstart: '0',		<== batchID here
        transactionstart: '0' }		<== last succesfull transaction +1
```
Then start the 'node massPayment.js'.\
The values you can leave in or you can put it back to 0 / 0 if you like.

## Why three seperate tools?
We decided to use seperate tools since this allows for additional tests of the payments before the payments are actually executed.
On the other hand, it does not provide any drawback since both scripts could also be called directly one after the other with:
```sh
node appng.js && node masstx.js or ./start_collector && node massPayment.js
```
However, it is strongly recommended to check the payments before the actual payments are done.\
So what you could do for example, run from crontab;
- run the collector session every saterday evening
- run the checkPaymentFile every sunday, mail the output, so you have visibility
- run the massPayment job every tuesday

With this scheme, you have a nice automated schema and works as follows;
- Every wednesday & saterday, if the collector batchsize is still too large (because the mainnet blockheight is to low),
  the cronjob exits and waits till next collector day. If the blockrange fits, the payments are collected and
  the job is logged and queued
- Every Sunday, the checkPayment job checks the payqueue. If empty, it exits. If there is (are) job(s) in the queue,
  the paymentdata for all batchIDs are shown and your output was send by email. You have time to check the results
- Every tuesday, the payment is done for all batchIDs in the queue.

NOTE\
It safe to schedule the collector and check jobs. However, regarding payments,\
it's always possible that something disrupts the transaction process, in which payments\
could fail and leasers don't receive what they should. It's up to you, if you feel confident\
with automated payments. If not, you can just execute massPayment.js by hand.\
MassPayment has forseen in the event that crashes or failed transactions (due to whatever reason) happen,\
by which you can add the batchID and the number of the last succesfull transactionnr.+1, to the file\
and then start massPayment.js again. Transactions will be executed from where the failures started.\
The nice thing is that the three tools are decoupled. So, if you run the collector three times a week\
and the the checks every week and the payout just once a month or whenever you feel it's a good moment,\
that's all fine. It also depends on the frequency of blockhits for your node and the blockwindows size\
you configure. It's all up to you and it doesn't bite one another.

## Disclaimer
Please always test your resulting payment scripts, e.g., with the _checkPaymentsFile.js_ script!
