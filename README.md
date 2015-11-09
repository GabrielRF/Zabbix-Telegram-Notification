# How to integrate Telegram and Zabbix

###### Pull requests are more than welcome!

* [Introduction](#introduction)
  * [Zabbix](#zabbix)
  * [Telegram](#telegram)
  * [Telegram bot](#telegram-bot)
* [Getting started](#getting-started)
  * [Dependencies](#dependencies)
  * [Installation](#installation)
  * [Creating a bot](#creating-a-bot)
* [Configuration](#configuration)
  * [Bot configuration](#bot-configuration)
  * [Zabbix integration](#zabbix-integration)
    * [Python script](#python-script)
    * [Actions](#actions)
    * [Media type](#media-type)
    * [Users](#users)
* [Telegram ID](#telegram-id)
  * [UserID](#userid)
  * [GroupID](#groupid)
  * [ChannelID](#channelid)
* [More info](#more-info)

## Introduction

### Zabbix

Zabbix is the ultimate enterprise-level software designed for real-time monitoring of millions of metrics collected from tens of thousands of servers, virtual machines and network devices.

Zabbix is Open Source and comes at no cost.

http://zabbix.com

### Telegram

Telegram is a cloud-based mobile and desktop messaging app with a focus on security and speed.

http://telegram.org

### Telegram bot

Bots are special Telegram accounts designed to handle messages automatically. Users can interact with bots by sending them command messages in private or group chats. You control your bots using HTTPS requests available on the [bot API](https://core.telegram.org/bots/api).

To everything described here I'll use the following bot token:
`158700146:AAHOPReqqTR8V7FXysa8mJCbQACUWSTBog8`.

> Please note that this token is only an ___example___. You must generate your own by following the steps on [Creating a bot](#creating-a-bot).

## Getting started

### Dependencies

The only thing you must have before following there instructions is a Zabbix Server running.

### Installation

The only installation required is pyTelegramBotAPI.

#### Using pip
```
$ pip install pyTelegramBotAPI
```
#### From source

```
$ git clone https://github.com/eternnoir/pyTelegramBotAPI.git
$ cd pyTelegramBotAPI
$ python setup.py install
```

### Creating a bot

To create a bot, talk to [@BotFather](http://telegram.me/BotFather) on Telegram.

Send ```/newbot``` and it will ask the bot's name. Choose any name.

Then it will ask the bot's username. The username must end with ```bot``` and is unique.

When everything is done, the [@BotFather](http://telegram.me/BotFather) will display the bot's _token_. Save it and keep it secret. Here is an example of a token:
`158700146:AAHOPReqqTR8V7FXysa8mJCbQACUWSTBog8`

In order to receive messages, the recipient user ___MUST___ send a message to the bot at least once.

If a group will be used, add the bot on the group before testing.

For channels, add the bot to the channel as an _administrator_.

## Configuration

### Bot configuration

No bot configuration is needed, but there are optional settings.

Commands available on [@BotFather](http://telegram.me/BotFather):

- `/token` Shows your bot's token.
- `/revoke` Revoke the token and give you a new one.
- `/setname` Change the bot's name.
- `/setdescription` Set the bot's description, a window displayed when a person opens a chat with the bot.
- `/setabouttext` Set the bot's about message, displayed on the bot's profile page.
- `/setuserpic` The bot's photo.
- `/setcommands` The commands that will be listed as bot's options.
- `/setjoingroups` Defines if the bot may or may not join groups.
- `/setprivacy` Set if the bot reads all the messages on a group or only messages that it is cited.
- `/deletebot` Used to destroy the bot.

### Zabbix integration

#### Python script

Create a file named `telegram_notification.py` on the folder `/usr/src/zabbixbot`:

```
$ vi telegram_notification.py
```
Then paste the code:
```
#!/usr/bin/env python

import telebot,sys

BOT_TOKEN='158700146:AAHOPReqqTR8V7FXysa8mJCbQACUWSTBog8'
DESTINATION=sys.argv[1]
SUBJECT=sys.argv[2]
MESSAGE=sys.argv[3]

MESSAGE = MESSAGE.replace('/n','\n')

tb = telebot.TeleBot(BOT_TOKEN)
tb.send_message(DESTINATION,SUBJECT + '\n' + MESSAGE)
```

#### Zabbix_server.conf
Go to file `/etc/zabbix/zabbix_server.conf`
```
vi /etc/zabbix/server.conf
```
find the line `AlertScriptsPath=`
```
/AlertScriptsPath
```
and include a new line
```
AlertScriptsPath=/usr/src/zabbixbot
```

#### Media type

On the Zabbix interface, go to _Adminstration_, _Media types_, and click on _Create media type_.

- Name: `telegram_notification.py`
- Type: `Script`
- Script name: `telegram_notification.py`

#### Actions

Open Zabbix web interface, go to _Configuration_, _Actions_ and click on _Create Action_.

- Name: `Telegram notification`
- Subject: `{HOSTNAME}: {TRIGGER.NAME} Status: {TRIGGER.STATUS} Value: {ITEM.VALUE}`
- Message:
```
notification = {
  'hostname' : {HOSTNAME},
  'ipaddress' : {IPADDRESS},
  'trigger' : {
    'id' : {TRIGGER.ID},
    'name' : {TRIGGER.NAME},
    'status' : {TRIGGER.STATUS},
    'severity' : {TRIGGER.SEVERITY},
    'nseverity' : {TRIGGER.NSEVERITY},
    'expression' : {TRIGGER.EXPRESSION},
    'url' : {TRIGGER.URL},
  },
  'item' : {
    'id' : {
      '1' : {ITEM.ID1},
      '2' : {ITEM.ID2},
      '3' : {ITEM.ID3},
      '4' : {ITEM.ID4},
      '5' : {ITEM.ID5},
      '6' : {ITEM.ID6},
      '7' : {ITEM.ID7},
      '8' : {ITEM.ID8},
      '9' : {ITEM.ID9},
    },
    'description' : {
      '1' : {ITEM.DESCRIPTION1},
      '2' : {ITEM.DESCRIPTION2},
      '3' : {ITEM.DESCRIPTION3},
      '4' : {ITEM.DESCRIPTION4},
      '5' : {ITEM.DESCRIPTION5},
      '6' : {ITEM.DESCRIPTION6},
      '7' : {ITEM.DESCRIPTION7},
      '8' : {ITEM.DESCRIPTION8},
      '9' : {ITEM.DESCRIPTION9},
    },
    'key' : {
      '1' : {ITEM.KEY1},
      '2' : {ITEM.KEY2},
      '3' : {ITEM.KEY3},
      '4' : {ITEM.KEY4},
      '5' : {ITEM.KEY5},
      '6' : {ITEM.KEY6},
      '7' : {ITEM.KEY7},
      '8' : {ITEM.KEY8},
      '9' : {ITEM.KEY9},
    },
    'value' : {ITEM.VALUE},
  },
  'event' : {
    'id' : {EVENT.ID},
    'date': {EVENT.DATE},
    'time': {EVENT.TIME},
    'age' : {EVENT.AGE},
  }
};
```
The fields _Subject_ and _Message_ are supposed to be customized as your needs.

Go to tab _Conditions_ and add settings as your needs.

Go to tab _Actions_ and add settings as your needs.

#### Users

The last step is to set who will receive the alerts.

Go to _Administration_, _Users_ and choose who will receive the notifications. Then, go to _Media_ and click on _Add_.

- Type: `telegram_notification.py`
- Send to: `ID` | Refer to [Telegram ID](telegram-id)

## Telegram ID

Telegram needs `ids` to send messages. The easiest way to get this id is using the bot you have just created.

Go to `https://api.telegram.org/bot158700146:AAHOPReqqTR8V7FXysa8mJCbQACUWSTBog8/getUpdates` using your browser.

### UserID

If you want to get a user id, send a message from this user to the bot. Reload the page and the user id will be shown.

Example:

```
"message":{"message_id":59,"from":{"id":9083329,"first_name":"Gabriel","last_name":"R F","username":"GabrielRF"},"chat":{"id":9083329,"first_name":"Gabriel","last_name":"R F","username":"GabrielRF","type":"private"},"date":1446911853,"text":"\/start"}}]}
```
In this case, the user id is `9083329`. So, on the step [Users](#users), the field `Send to` would be `9083329`.

### GroupID

If you prefer to have your bot working on a group, then create the group and add the bot to it. Reload the page and you will see a message like:

```
"message":{"message_id":60,"from":{"id":9083329,"first_name":"Gabriel","last_name":"R F","username":"GabrielRF"},"chat":{"id":-57169325,"title":"q31231","type":"group"},"date":1446912067,"group_chat_created":true}},{"update_id":727527785,
```
In this case, the group id is `-57169325`. So, on the step [Users](#users), the field `Send to` would be `-57169325`.

Note that a group id is always ___negative___. Only user's id are positive.

### ChannelID

The last alternative is the easiest one. There's no need to get a channel id. Simply set the `Send to` field on the step [Users](#users) to `@ChannelID`. The `@` symbol is ___obligatory___.

## More info

### Inspiration

Mo@st of this method is based on a post from a Brazilian dev named Tobias. [Zabbix com notificações pelo Telegram (Portuguese only)](http://tobias.ws/blog/zabbix-com-notificacoes-pelo-telegram/)

### pyTelegramBotAPI

A simple, but extensible Python implementation for the Telegram Bot API. [eternoir/pyTelegramBotAPI](https://github.com/eternnoir/pyTelegramBotAPI)
