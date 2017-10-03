[![Build Status](https://travis-ci.org/inertia186/drotto.svg?branch=master)](https://travis-ci.org/inertia186/radiator)
[![Code Climate](https://codeclimate.com/github/inertia186/drotto/badges/gpa.svg)](https://codeclimate.com/github/inertia186/drotto)
[![Test Coverage](https://codeclimate.com/github/inertia186/drotto/badges/coverage.svg)](https://codeclimate.com/github/inertia186/drotto)

bakerbob
======

[BakerBob](https://github.com/finleyexp/BakerBob) is a voting bot that accepts payments for votes on baking related posts on the STEEM blockchain.

<center>
  <img src="https://cdn.pixabay.com/photo/2017/01/14/13/17/kermit-1979495_640.jpg" />
</center>

---

The default is that Baker Bob will only vote in 10 batches a day.  Multiple users can bid in a voting batch.  If only one person bids, they get the entire upvote.  If two people bid an equal amount, they share the vote 50/50.  The higher the bid, the higher percentage for the upcoming vote batch.

The bot operator can set any vote weight for the batch, which will affect the number of daily votes to bid on.  Therefore, each day per batch has:

| Votes | Percentage |      Timeframe     | Blocks |
|:-----:|:----------:|:------------------:|:------:|
|  10   |    100 %   |  every 2.4 hours   |  2,880 |
|  20   |     50 %   |  every 1.2 hours   |  1,440 |
|  40   |     25 %   |  every 36 minutes  |    720 |
|  80   |   12.5 %   |  every 18 minutes  |    360 |
|  160  |   6.25 %   |  every 9 minutes   |    180 |
|  320  |   3.13 %   |  every 270 seconds |     90 |

**Example A**

If you set the bot to vote at 100.00%, bids open every 2.4 hours.  Alice and Bob both bid for in the same voting batch.  If Alice bids 4 SBD and Bob bids 2 SBD, Alice will get a 66.66% upvote and Bob will get a 33.33% upvote.

**Example B**

If you set the bot to vote at 3.13%, bids open every 270 seconds.  Alice and Bob both bid for in the same voting batch.  If Alice bids 4 SBD and Bob bids 2 SBD, Alice will get a 2.09% upvote and Bob will get a 1.04% upvote.

<div class="pull-right">
  <img src="http://i.imgur.com/lh40Ab0.png" />
</div>

**Usage Rules:**

1. If there are multiple bids with the same post, only one vote will be cast and the remaining bids will not be returned.
2. If the bot has already voted for a post, additional bids will not be returned.
3. The URL must be correctly expressed in the memo alone.  Malformed memos will not be returned.

---

#### Install

To use this [Radiator](https://steemit.com/steem/@inertia/radiator-steem-ruby-api-client) bot:

##### Linux

```bash
$ sudo apt-get update
$ sudo apt-get install ruby-full git openssl libssl1.0.0 libssl-dev
$ sudo apt-get upgrade
$ gem install bundler
```

##### macOS

```bash
$ gem install bundler
```

[inertia186](https://github.com/inertia186/drotto) tested it on various versions of ruby.  The oldest one that work was:

`ruby 2.0.0p645 (2015-04-13 revision 50299) [x86_64-darwin14.4.0]`

First, clone this git and install the dependencies:

```bash
$ git clone https://github.com/inertia186/drotto.git
$ cd drotto
$ bundle install
```

##### Configure

Edit the `config.yml` file.

```yaml
:bakerbob:
  :block_mode: irreversible
  :account_name: <voting account name here>
  :posting_wif: <posting wif here>
  :active_wif: <active wif here>
  :batch_vote_weight: 100.00 %
  :min_effective_weight: 0.25 %
  :reserve_vote_weight: 0.00 %
  :minimum_bid: 0.100 SBD
  :maximum_bid: 0.500 SBD
  :blacklist: mikethemug
:chain_options:
  :chain: steem
  :url: https://steemd.steemit.com
```

Edit the `support/confirm.md` template, used to reply to the post when voting.

```markdown
This ${content_type} has received a ${vote_weight_percent} % ${vote_type} from @${account_name} thanks to @${from}.
```

##### Run Mode

Then run it:

```bash
$ rake run
```

Baker Bob will now do it's thing.  Check here to see an updated version of this bot:

https://github.com/inertia186/drotto

##### Bounce Mode

Baker Bob is designed to only vote in such a way that it will never run out of voting power.  Ideally, you should never need to shut down for breaks in order to recharge.  However, if it's ever time for Dr. Otto to take a break for some other reason, `bounce` will return transfers to accounts instead of voting.

For your own safety, it is recommended that you transfer your funds out of your wallet before running this mode.

Baker Bob will now return the funds as they arrive in the wallet.  You can also just use `bounce_once` to have Baker Bob make a single pass rather than loop forever until signaled (`^C`).

```bash
$ rake bounce_once
```

Both `bounce` modes accept a limit value as argument, which is especially useful for `bounce_once`.  The default limit is to go back `200` transactions in the history.  You can set up to `10000`, if you need to go back further for some reason.

```bash
$ rake bounce_once[10000]
```

You can also use `bounce_stream` to immediately bounce transfers that are invalid.  The advantage of this is that bounced transactions are returned right when they happen if they cannot be processed rather than wait for the batch.

```bash
$ rake bounce_stream
```

##### Report Mode

Same as `bounce_once` but only for reporting without doing the transfers.

```bash
$ rake report
```

Also accepts a limit argument.

```bash
$ rake report[10000]
```

---

#### Upgrade

Typically, you can upgrade to the latest version by this command, from the original directory you cloned into:

```bash
$ git pull
```

Usually, this works fine as long as you haven't modified anything.  If you get an error, try this:

```
$ git stash --all
$ git pull --rebase
$ git stash pop
```

If you're still having problems, I suggest starting a new clone.

---

#### Troubleshooting

##### Problem: Everything looks ok, but every time Dr. Otto tries to vote, I get this error:

```
Unable to vote with <account>.  Invalid version
```

##### Solution: You're trying to vote with an invalid key.

Make sure the `.yml` file contains the correct voting key and account name (`social` is just for testing).

##### Problem: The node I'm using is down.

Is there a list of nodes?

##### Solution: Yes, special thanks to @ripplerm.

https://ripplerm.github.io/steem-servers/

---

## Tests

* Clone the client repository into a directory of your choice:
  * `git clone https://github.com/finleyexp/bakerbob.git`
* Navigate into the new folder
  * `cd bakerbob`
* Basic tests can be invoked as follows:
  * `rake`
* To run tests with parallelization and local code coverage:
  * `HELL_ENABLED=true rake`

## Get in touch!

If you're using BakerBob, I'd love to hear from you.  Drop me a line and tell me what you think!  I'm @finleyexp on STEEM and Discord. BakerBob is based off of Dr. Otto by @inertia186 on STEEM and Discord.
  
## License

@inertia186 doesn't believe in intellectual "property," but I do. That said, if you also do, then consider Dr. Otto as licensed under a Creative Commons [![CC0](http://i.creativecommons.org/p/zero/1.0/80x15.png)](http://creativecommons.org/publicdomain/zero/1.0/) License. 

Meanwhile, BakerBob is licensed under a Creative Commons Attribution-ShareAlike 4.0 International License: ![CC BY-SA](https://i.creativecommons.org/l/by-sa/4.0/80x15.png)

---

Also see:

* [Dr. Otto Technical Update](https://steemit.com/radiator/@inertia/dr-otto-technical-update)
* [Dr. Otto: Vote Bidding Bot](https://steemit.com/radiator/@inertia/dr-otto-vote-bidding-bot)
