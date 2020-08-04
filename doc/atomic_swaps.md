# Overview #

As of mwc-wallet 3.2.2, atomic swaps are supported by mwc-wallet. This feature will allow to exchange MWC coins to 
any Secondary currency from supported currency list. Swap trade workflow is controlled by the wallet and can be run in 
manual or automatic mode. 

This document explain how Atomic Swaps can be done in automatic mode. Please note, there is also manual mode that allow 
to run swap trade step by step and use files for message exchanges. 

We strongly recommend use automatic mode for the normal usage.

# Configuration #

The configuration include some parameter in the mwc-wallet.toml.  Every secondary supported currency will require 
it's own node to monitor the blockchain and process transactions with a secondary currency. Default values point to the 
community node, you can install your ouwn to get more security. 

```
# ElectrumX BTC Node URI needed for atomic swap that include with BTC.
electrum_node_addr = "52.23.248.83:8000"
```

# Atomic swap workflow #

First Seller (person who sell MWC coins) and Buyer (person who buy MWC coins) need to contact with each other, define the
exchange rate, amounts of the coins to exchange. Exchange with wallet addresses (atomic swaps can use mwcmqs or tor for 
communications). 

* Atomic swap is started by the Seller (person who want to sell MWC coins and buy some other coins). Seller should specify 
the swap trade parameters and Buyer destination address. Seller can use `swap_start` command to create a swap trade.
Please note, this command will not start atomic swap trade.

This example creates the atomic swap trade where 5.6 MWC tarded to 0.087 BTC. MWC transactions will require 500 confirmations,
BTC transactions will require 6 confirmations. Time interval required for emessage exchange and redeem are 1 hour (60 minutes).
BTC redeem address is n4GUrta1qhA1Zgy4DUkmDgxULtJKjDhEc6. Seller will lock the funds first.

```
$ mwc-wallet cli
mwc-wallet> help swap_start
    ...
mwc-wallet> open
Password:
Command 'open' completed

mwc-wallet> swap_start --mwc_amount 5.6 --secondary_currency btc --secondary_amount 0.087  --mwc_lock_confirmations 500 --secondary_lock_confirmations 6 --message_exchange_time 60 --redeem_time 60 --secondary_address n4GUrta1qhA1Zgy4DUkmDgxULtJKjDhEc6 --who_lock_first seller
20200804 12:19:27.863 WARN grin_wallet_controller::command - Seller Swap trade is created: 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5
Command 'swap_start' completed
```
This command successfully created trade 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5

* Buyer should have listener up and running.

```
$ mwc-wallet cli
mwc-wallet> open
Password:
Command 'open' completed
mwc-wallet> listen -m mwcmqs
20200804 12:25:55.310 WARN grin_wallet_controller::controller - Starting MWCMQS Listener
20200804 12:25:56.207 WARN grin_wallet_impls::adapters::mwcmq -
mwcmqs listener started for [xmgHFXM1ryJ1ug7kGPsjmDj8Gd7XC18cfhQ8n8uyjxL3JzAq9r73] tid=[tr3MClzoqB1ecFnH0kHBH]
```

* Seller starting atomic swap trade in atomatic mode with a `swap --autoswap` command.  Please specify enough transaction fee 
for redeem transaction, so it will not stuck in the memory pool. Please keep in mind, **if your redeem transaction stuck in the 
memory pool, the Buyer will be able to get all the coins**.

```
$ mwc-wallet cli
mwc-wallet> open
Password:
Command 'open' completed
mwc-wallet> listen -m mwcmqs
20200804 12:25:55.310 WARN grin_wallet_controller::controller - Starting MWCMQS Listener
20200804 12:25:56.207 WARN grin_wallet_impls::adapters::mwcmq -
mwcmqs listener started for [xmgHFXM1ryJ1ug7kGPsjmDj8Gd7XC18cfhQ8n8uyjxL3JzAq9r73] tid=[tr3MClzoqB1ecFnH0kHBH]
mwc-wallet> swap --autoswap --method mwcmqs --dest xmggm9xA2ryzDARaRKNEdbw9rmSHxyLTMCqNua8iSPjCQAvsyx6s --secondary_fee_per_byte 30 -i 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5

    Swap ID: 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5
    Selling 5.6 MWC for 0.087 BTC. BTC redeem address: n4GUrta1qhA1Zgy4DUkmDgxULtJKjDhEc6
    Requied lock confirmations: 500 for MWC and 6 for BTC
    Time limits: 60 minutes for messages exchange and 60 minutes for redeem/refund
    Locking order: Seller lock MWC first
    MWC funds locked until block 508701, expected to be mined in 21 hours and 11 minutes
    BTC funds locked for 33 hours and 25 minutes

-------- Execution plan --------
    Offer Created at August  4 12:19:27
--> Sending Offer to Buyer                    required by August  4 13:19:27
        Sending Offer message, expired in 49 minutes
    Waiting For Buyer to accept the offer     required by August  4 13:19:27
    Locking MWC funds                         required by August  4 13:46:57
    Waiting for Lock funds confirmations      required by August  4 23:29:27
    Waiting For Init Redeem message           required by August  4 23:29:27
    Sending back Redeem Message               required by August  4 23:29:27
    Wait For Buyer to redeem MWC              required by August  5 09:41:12
    Post Secondary Redeem Transaction         required by August  5 20:49:27
    Wait For Redeem Tx Confirmations
    Swap completed

-------- Trade Journal --------
    August  4 12:19:27  Swap offer is created

-------- Required Action --------
    Sending Offer message, expired in 49 minutes


WARNING: [xmggm9xA2ryzDARaRKNEdbw9rmSHxyLTMCqNua8iSPjCQAvsyx6s] has not been connected to mwcmqs for 72666 seconds. This user might not receive the swap message.
Swap Trade 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5: Offer message was sent
Swap Trade 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5: Waiting for Accept Offer message
```

* Buyer waiting for the message that swap offer is received. The offer Swap Id will be printed.
```
You get an offer to swap BTC to MWC. SwapID is 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5
```

* Buyer reviews the trade details. If something wrong, just cancel the trade and notify Seller about the problem. 
The seller will need to cancel it's own stap tarde and create a new trade with fixed parameters. 

```
mwc-wallet> swap --check -i 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5

    Swap ID: 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5
    Buying 5.6 MWC for 0.087 BTC
    Requied lock confirmations: 500 for MWC and 6 for BTC
    Time limits: 60 minutes for messages exchange and 60 minutes for redeem/refund
    Locking order: Seller lock MWC first
    MWC funds locked until block 508701, expected to be mined in 21 hours and 9 minutes
    BTC funds locked for 33 hours and 23 minutes

-------- Execution plan --------
    Get an Offer at August  4 12:19:27
--> Send Accept Offer Message                 required by August  4 13:19:27
        Sending Accept Offer message, expired in 47 minutes
    Wait for seller to start locking MWC      required by August  4 13:46:57
    Post BTC to lock account                  required by August  4 13:46:57
    Wait for Locking funds confirmations      required by August  4 23:29:27
    Send Init Redeem Message                  required by August  4 23:29:27
    Wait For Redeem response message          required by August  4 23:29:27
    Redeem MWC                                required by August  5 00:29:27
    Wait For Redeem Tx Confirmations
    Swap is completed

-------- Trade Journal --------
    August  4 12:30:29  Get a Swap offer

-------- Required Action --------
    Sending Accept Offer message, expired in 47 minutes

Command 'swap' completed
```

* Buyer starting atomic swap trade in atomatic mode with a `swap --autoswap` command. Please specify enough transaction fee 
for refund transaction, so it will not stuck in the memory pool.

```
mwc-wallet> swap --autoswap --method mwcmqs --dest xmgHFXM1ryJ1ug7kGPsjmDj8Gd7XC18cfhQ8n8uyjxL3JzAq9r73 --buyer_refund_address mjdcskZm4Kimq7yzUGLtzwiEwMdBdTa3No --secondary_fee_per_byte 30 -i 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5

    Swap ID: 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5
    Buying 5.6 MWC for 0.087 BTC
    Requied lock confirmations: 500 for MWC and 6 for BTC
    Time limits: 60 minutes for messages exchange and 60 minutes for redeem/refund
    Locking order: Seller lock MWC first
    MWC funds locked until block 508701, expected to be mined in 21 hours and 7 minutes
    BTC funds locked for 33 hours and 20 minutes

-------- Execution plan --------
    Get an Offer at August  4 12:19:27
--> Send Accept Offer Message                 required by August  4 13:19:27
        Sending Accept Offer message, expired in 44 minutes
    Wait for seller to start locking MWC      required by August  4 13:46:57
    Post BTC to lock account                  required by August  4 13:46:57
    Wait for Locking funds confirmations      required by August  4 23:29:27
    Send Init Redeem Message                  required by August  4 23:29:27
    Wait For Redeem response message          required by August  4 23:29:27
    Redeem MWC                                required by August  5 00:29:27
    Wait For Redeem Tx Confirmations
    Swap is completed

-------- Trade Journal --------
    August  4 12:30:29  Get a Swap offer

-------- Required Action --------
    Sending Accept Offer message, expired in 44 minutes

Command 'swap' completed
Swap Trade 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5: Response to offer message was sent back
Swap Trade 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5: Seller locking funds, waiting for 1 MWC lock confirmations, has 0
```

* Seller should get a message that swap is accepted and automactic mode will process to the next steps. At this moment 
Seller need to keep wallet running until trade will be finished. There is no need to be around, swap for Seller will go 
in automatic mode. If Buyer didn't act in reasonable and timely manner, the swap trade will be cancelled and refunded automatically.

```
Processed Offer Accept message
Swap Trade 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5: MWC lock slate is posted
Swap Trade 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5: MWC Lock transaction, waiting for 500 MWC lock confirmations, has 0
```

* Buyer will need to wait until he will need to deposit Secondary (for example BTC) coins to a multisig lock account. 
In a few minutes after starting the trade, Buyer should see a message `Please deposit exactly XXXXXX BTC at <address>`
 
```
Swap Trade 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5: Please deposit exactly 0.087 BTC at 2N4YMbGBzP39WjhJMEFtrMhCxLz8iifcepW
``` 
 
* Buyer should post the funds to that address. Please specify enough transaction fees. Transation must be mined before 
expiration time. 

* At this moment Buyer need to keep wallet running until trade will be finished. There is no need to be around, swap will
 go in automatic mode. If Seller didn't act in reasonable and timely manner, the swap trade will be cancelled and refunded automatically.


# Cancellation #

The swap trade can be cancelled at the starting stage, until Buyer posted a redeem transaction. Depend on the stage of this transaction,
the waiting for refund might be required.

In this example Buyer didn't lock any funds yet, so his trade is cancelled immediately. 
```
mwc-wallet> swap --adjust cancel  -i 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5
Swap trade 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5 was successfully adjusted. New state: Buyer swap was cancelled, nothing was locked, no need to refund
Command 'swap' completed
mwc-wallet> Swap Trade 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5: Swap trade is finished
```

The seller already locked MWC, so he will need to wait about 20 hours and 55 minutes to get the fund.
```
mwc-wallet> swap --adjust cancel -i 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5
Swap trade 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5 was successfully adjusted. New state: Waiting when refund Slate can be posted
Command 'swap' completed successfully

mwc-wallet> swap --check  -i 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5
Password:

    Swap ID: 975ab0c2-27f5-45bd-99f2-2c3b01ce0fa5
    Selling 5.6 MWC for 0.087 BTC. BTC redeem address: n4GUrta1qhA1Zgy4DUkmDgxULtJKjDhEc6
    Requied lock confirmations: 500 for MWC and 6 for BTC
    Time limits: 60 minutes for messages exchange and 60 minutes for redeem/refund
    Locking order: Seller lock MWC first
    MWC funds locked until block 508701, expected to be mined in 20 hours and 55 minutes
    BTC funds locked for 33 hours and 6 minutes

-------- Execution plan --------
--> Wait for MWC refund to unlock             required by August  5 09:43:53
        Waiting when locked MWC can be refunded. About 1255 minutes are left, expired in 20 hours 55 minutes
    Post MWC Refund Slate                     started August  5 09:43:53  required by August  5 10:43:53
    Wait for MWC Refund confirmations
    Swap is cancelled, MWC are refunded

-------- Trade Journal --------
    August  4 12:19:27  Swap offer is created
    August  4 12:30:13  Offer message was sent
    August  4 12:35:24  Processed Offer Accept message
    August  4 12:35:27  MWC lock slate is posted
    August  4 12:47:52  Cancelled by user

Command 'swap' completed successfully
```
 