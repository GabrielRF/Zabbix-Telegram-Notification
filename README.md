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
  * [Inspiration](#inspiration)
  * [pyTelegramBotAPI](#pytelegrambotapi)
  * [Thanks](#thanks)

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

The only thing you must have before following these instructions is a Zabbix Server running.

### Installation

The only installation required is pyTelegramBotAPI. Choose one of the following methods:

#### Using pip
```
$ pip install pyTelegramBotAPI
```
It is also recommended to install security packages. To do this, run:
```
$ pip install pyopenssl ndg-httpsclient pyasn1
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
tb.send_message(DESTINATION,SUBJECT + '\n' + MESSAGE, disable_web_page_preview=True, parse_mode='HTML')
```
Change the file permission and allow it to be executed
```
# chown -R zabbix: /usr/src/zabbixbot/
# chmod +x telegram_notification.py
```
#### Zabbix_server.conf
Go to file `/etc/zabbix/zabbix_server.conf`
```
vi /etc/zabbix/server.conf
```
find the line `AlertScriptsPath=`. And include:
```
AlertScriptsPath=/usr/src/zabbixbot/
```
_The easiest way to find is typing_ `/AlertScriptsPath=`.

Restart Zabbix-server service.
```
service zabbix-server restart
```

#### Media type

On the Zabbix interface, go to _Adminstration_, _Media types_, and click on _Create media type_.

- Name: `telegram_notification.py`
- Type: `Script`
- Script name: `telegram_notification.py`

##### If using Zabbix 3.0.1:

 - Script Parameters
  - {ALERT.SENDTO}
  - {ALERT.SUBJECT}
  - {ALERT.MESSAGE}

#### Actions

Open Zabbix web interface, go to _Configuration_, _Actions_ and click on _Create Action_.

- Name: `Telegram notification`
- Subject: `#{HOSTNAME}: {TRIGGER.NAME} {TRIGGER.STATUS}`
- Message:
```
Value: {ITEM.VALUE} {TRIGGER.STATUS}
Date: {EVENT.DATE} Time: {EVENT.TIME}
```
The fields _Subject_ and _Message_ are supposed to be customized as your needs. HTML tags supported:
```
<b>bold</b>, <strong>bold</strong>
<i>italic</i>, <em>italic</em>
<a href="URL">inline URL</a>
<code>inline fixed-width code</code>
<pre>pre-formatted fixed-width code block</pre>
```

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

__If this method doesn't work or if you want to get an id from a private channel, please, visit https://github.com/GabrielRF/telegram-id__

## More info

### Inspiration

Most of this method is based on a post from a Brazilian dev named Tobias. [Zabbix com notificações pelo Telegram (Portuguese only)](http://tobias.ws/blog/zabbix-com-notificacoes-pelo-telegram/)

### pyTelegramBotAPI

A simple, but extensible Python implementation for the Telegram Bot API. [eternoir/pyTelegramBotAPI](https://github.com/eternnoir/pyTelegramBotAPI)

### Thanks

Finally, I would like to thank [João Paulo Nascimento](https://github.com/JoaoPNascimento), from Sergipe, who was the first person to test this method, helping me to find bugs.
