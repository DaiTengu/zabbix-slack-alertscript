# Zabbix-Slack AlertScript

## About

This Readme.md is a work in progress, Images need to be converted from the old to the new. It SHOULD be easy enough to follow, however.


This script will push events from [Zabbix](http://www.zabbix.com/) and display them in [Slack](https://slack.com/). This is an updated version of 
[ssplat's](https://github.com/DaiTengu) which is in turn an updated version of [ericoc's](https://github.com/ericoc) 
[zabbix-slack-alertscript](https://github.com/ericoc/zabbix-slack-alertscript). It is written in python and tested on Zabbix 3.2 and CentOS 7.2.

---

![slack](https://raw.githubusercontent.com/DaiTengu/zabbix-slack-alertscript/master/web_assets/slack_ss.png)

---

## Installation

### Slack

Log in to your Slack account and create a new incoming webhook: https://my.slack.com/services/new/incoming-webhook

1. Copy the Webhook Url. You will need to enter this into Zabbix later.
2. Set the Channel where you'd like the alerts to go (i.e. #alerts)
3. Set the bot username you'd like to use (i.e. Zabbix-bot)
4. Upload an icon for the service
	* i.e. https://raw.githubusercontent.com/DaiTengu/zabbix-slack-alertscript/master/web_assets/zabbix_logo.png

example config:

![slack_config](https://raw.githubusercontent.com/DaiTengu/zabbix-slack-alertscript/master/web_assets/slack_webhook_setup.png)

### Zabbix
#### Install the script
Log into the command line and place `slack_zabbix.py` in the alertscripts folder, i.e. `/usr/lib/zabbix/alertscripts`
```
# cd /usr/lib/zabbix/alertscripts
# wget https://raw.githubusercontent.com/DaiTengu/zabbix-slack-alertscript/master/slack_zabbix.py
# chmod 755 slack_zabbix.py
```

You may need to install extra python modules, you can do this with pip. 

CentOS:
```
# yum install python-pip
# pip install httplib2 json sys
```

Debian/Ubuntu:
```
# apt-get install python-pip
# pip install httplib2 json sys
```
#### Copy images to /img (Optional)
In the /img directory of this repository are some images scraped from a Google Image Search that will be displayed with your notifications. Copy them to the /img directory 
wherever your Zabbix web front-end is installed.  For instance, `/usr/share/zabbix/img` on CentOS if you installed via the Zabbix Repo. You can replace these images with your 
own, or skip this part if you do not want to use them.
 
#### Configure Zabbix
##### Media Type Configuration
In the WebUI, create a new Media Type for Slack. You can either clone an existing media type or click `Create Media Type`.

![zabbix_admin_media](https://raw.githubusercontent.com/DaiTengu/zabbix-slack-alertscript/master/web_assets/zabbix_admin_mediatypes.png)

Set the name to `Slack`, type to `Script`, Script Name to `slack_zabbix.py`, and mark it enabled. 
Under `Script parameters` click `Add` **7** times, and fill in the following:
1. {ALERT.SENDTO}
2. {ALERT.SUBJECT}
3. {ALERT.MESSAGE}
4. *https://my.website.com/zabbix* - The URL to your zabbix front-end
5. *Zabbix* - The Username you want to send notifications as (can be anything)
6. https://hooks.slack.com/services/ABC123/ABC123/ABCDEF123456 - the Slack hook URL you saved earlier
7. https://my.website.com/zabbix/img/zabbix-icon.png - URL to an icon that will be displayed next to the slack username in notifications.

![zabbix_admin_media_config](https://raw.githubusercontent.com/DaiTengu/zabbix-slack-alertscript/master/web_assets/zabbix_media_config.png)


##### User configuration
Configure users to have the Slack media type. I chose to modify the default admin user to only have the Slack media type.

![zabbix users config](https://raw.githubusercontent.com/DaiTengu/zabbix-slack-alertscript/master/web_assets/zabbix_user_admin.png)

![zabbix users media config](https://raw.githubusercontent.com/DaiTengu/zabbix-slack-alertscript/master/web_assets/zabbix_user_media.png)

##### Action configuration
Create a new action to send the alerts. You can clone an existing action or click `Create Action`.

![zabbix trigger actions](https://raw.githubusercontent.com/DaiTengu/zabbix-slack-alertscript/master/web_assets/zabbix_config_actions.png)

Configure the action. Set the name to anything you'd like. The subject should just be `{TRIGGER.STATUS}` which will be substituted by `PROBLEM` or `OK`. For the Message, set it to:
```
{TRIGGER.SEVERITY} ;; {HOST.NAME1} ;; {TRIGGER.NAME} ;; {TRIGGER.ID} ;; {EVENT.ID} ;; {ITEM.ID1} ;; {ITEM.NAME1}
```
' ;; ' is used as a delimiter but also keeps the string readable in case the fallback text is used instead of the fully formatted version. If you want to change the delimiter, you'll need to update `slack_zabbix.py` to match. If you change the order of the macros, you'll also need to update `slack_zabbix.py`, as well.

Configure the `Operations` section to send a message to a single user or a group..

![zabbix action config](https://raw.githubusercontent.com/DaiTengu/zabbix-slack-alertscript/master/web_assets/zabbix_action.png)


The `Recovery operations` tab should look the same, with the `Operations` section being set to `Send recovery message`. 


## Testing

You can run the `slack_zabbix.py` script manually from the command line with the format `slack_zabbix.py <to> <subject> <message>`.

```
$  ./slack_zabbix.py '#alerts' 'PROBLEM' 'Average ;; Zabbix server ;; Zabbix discoverer processes more than 75% busy ;; 83170 ;; 6480932 ;; 106552 ;; SNMP' 
'https://my.website.com/zabbix' 'Zabbix' 'https://hooks.slack.com/services/ABC123/ABCD1234/ABCDEFGHI123456789' 'https://my.website.com/zabbix/img/zabbix-icon.png''
```

## More Information
 * [Slack incoming web-hook functionality](https://my.slack.com/services/new/incoming-webhook)
 * [Zabbix (2.4) custom alertscripts documentation](https://www.zabbix.com/documentation/2.4/manual/config/notifications/media/script)
