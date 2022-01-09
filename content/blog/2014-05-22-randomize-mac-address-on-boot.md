---
title: "How to Spoof a (Valid) Random MAC Address on Bootup"
tags: [scripts, snippets]
categories: blog
published: true
comments: true
---

**Updated for Yosemite 10.10.4**

I spend a lot of time at coffee shops, using their wi-fi networks. I'm not sure who is operating these things, or if they're hacked, so I wanted to add a layer anonymity to my connection. Tons of routers record and store the MAC address of anyone connected. That makes me uneasy. Here's how I randomized a new (and valid) MAC address on every reboot of my MacBook pro, running OS X 10.10.4 (Yosemite).

### Create a Bash script

Open your terminal. Let's create a folder and an empty script file in one swoop.

```bash
sudo mkdir /Scripts && sudo nano /Scripts/mac-random.sh
```

Nano will open up a new `mac-random.sh` file with blank contents. Paste the following in there:

```bash
#!/bin/bash

# disconnect the airport from any network

sudo /System/Library/PrivateFrameworks/Apple80211.framework/Resources/airport -z

# use OpenSSL to generate a random MAC address

mac=$(openssl rand -hex 1 | tr '[:lower:]' '[:upper:]' | xargs echo "obase=2;ibase=16;" | bc | cut -c1-6 | sed 's/$/00/' | xargs echo "obase=16;ibase=2;" | bc | sed "s/$/:$(openssl rand -hex 5 | sed 's/\(..\)/\1:/g; s/.$//' | tr '[:lower:]' '[:upper:]')/" )

# reassign the wifi adapter with the saved $mac variable above

# the wifi adapter in my case is "en0". yours may be different. run "ifconfig" to find yours.

sudo ifconfig en0 ether $mac
networksetup -detectnewhardware
networksetup -setairportpower airport off
networksetup -setairportpower airport on

```

Notice the elaborate chain of commands that we used with OpenSSL? Props to commenter [osmium](http://osxdaily.com/2012/03/01/change-mac-address-os-x/#comment-384258) for that. This ensures that the [least significant bit of the first byte is 0](https://en.wikipedia.org/wiki/MAC_address#Address_details).

Save `mac-random.sh` with the new contents. Now we get need to reference this file on startup!

### Load the Bash script on every bootup

But first let's make sure it has all the right permissions.

```bash
sudo chown -R root:admin /Scripts/mac-random.sh
sudo chmod u=rwx /Scripts/mac-random.sh
```

The kosher way OS X loads boot items is with a launchd `.plist` file. Let's create one which references the script.

```bash
cd /Library/LaunchDaemons && sudo nano com.superuser.macrandom.plist
```

Paste the following XML-style contents into the blank file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.superuser.macrandom</string>
  <key>ProgramArguments</key>
  <array>
    <string>/bin/sh</string>
    <string>/Users/ericschaefer/Scripts/mac-random.sh</string>
  </array>
  <key>UserName</key>
  <string>root</string>
  <key>GroupName</key>
  <string>wheel</string>
  <key>RunAtLoad</key>
  <true/>
  <key>Debug</key>
  <true/>
  <key>KeepAlive</key>
  <false/>
</dict>
</plist>
```

Save and exit nano. Then change the permissions on this file that you just saved:

```bash
sudo chown -R root:wheel com.superuser.macrandom.plist
```

Then load the file into launchd:

```bash
sudo launchctl load com.superuser.macrandom.plist
```

Check if you're good to go...

```bash
sudo launchctl list | grep macrandom
```

If you see the launch item, then this will load on boot!

Once you've rebooted, run `ifconfig` to check if your MAC address has changed for your wi-fi adapter (named `en1` in my case).
