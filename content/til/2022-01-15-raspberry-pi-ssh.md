---
title: 'Fixing the Raspberry Pi SSH server'
tags: [til, raspberry pi, linux]
published: true
comments: false
---

After enabling the SSH server on my Raspberry Pi, I was still unable to login remotely with the login and password I had set up. I kept seeing this message:

```bash
Permission denied (publickey,password)
```

I finally managed to fix this by editing the Raspberri Pi `sshd_config`

```bash
sudo nano /etc/ssh/sshd_config
```

And change the settings in there to

```
PasswordAuthentication yes
ChallengeResponseAuthentication yes
```

Then restart the ssh service

```
sudo service ssh restart
```

This let me finally login with the password prompt.

The next thing I did was to only allow login with my public key.

To quickly put my public key on the Raspberry Pi:

```
ssh-copy-id pi@<raspberry-pi-ip-address>
```

Then on the Raspberry Pi, I disabled the password login option, and enabled only the public key option:

```
PubkeyAuthentication yes
PasswordAuthentication no
ChallengeResponseAuthentication no
```

And again restart the ssh server.

```
sudo service ssh restart
```

Now I'm able to login with just my public key!
