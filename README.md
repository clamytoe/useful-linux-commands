# Useful Linux/Unix Commands
> Code examples of some useful Linux/Unix commands.

Some of the multi-line commands can be condensed into one-liners by simply separating the commands with an `;`. The ones that I have left in multi-line format are ones that I've created into scripts and are in their own files.

## Check file change/download status
If you are downloading a bunch of files or generating new ones this command will show you its progress in bytes.

```bash
#!/bin/bash
while true
    do
        clear
        ls -alt | awk '{print $5}' | head -n 2
        sleep 1
    done
```

## Check if your password has been pwned
```bash
#!/bin/bash
sha1=$(echo -n "$1" | sha1sum | awk '{print toupper($1)}' | tr -d '\n');
curl -s -H $'Referer: https://haveibeenpwned.com/' https://api.pwnedpasswords.com/range/$(echo -n $sha1 | cut -c1-5) | grep -i $(echo -n $sha1 | cut -c6-40);
```

## Clear history
```bash
#!/bin/bash
cat /dev/null > ~/.local/share/recently-used.xbel
```

## Creat a favicon.ico from a png file
The following requires that the latest version of *imagemagick* be installed on your system. The `-background transparent` portion is optional.

```bash
#!/bin/bash
convert -background transparent $1 -define icon:auto-resize=16,32,48,64,128 favicon.ico
```

## Converting from .m4b to .mp3

```bash
faad --stdio INPUT.m4b | lame --preset standard - OUTPUT.mp3
```

## Generate an ascii table
I've never needed this, but if I ever do, well I have it :)

```bash
#!/bin/bash
START=33    # Range of printable ASCII characters (decimal).
END=126     # Will not work for unprintable characters (> 126).

echo " Decimal  Hex     Character"  # Header.
echo " -------  ---     ---------"

for ((i=START; i<=END; i++))
do
  echo $i | awk '{printf("   %3d     %2x         %c\n", $1, $1, $1)}'
# The Bash printf builtin will not work in this context:
#   printf "%c" "$i"
done

exit 0
```

## Generate a random password
The following is a python script that uses *f-strings* so requires 3.6 and above.

```python
#!/usr/bin/env python3.6
from secrets import choice
import string

alphabet = string.ascii_letters + string.digits
while True:
    password = ''.join(choice(alphabet) for i in range(10))
    if (any(c.islower() for c in password)
            and any(c.isupper() for c in password)
            and sum(c.isdigit() for c in password) >= 3):
        break

# On standard Linux systems, use a convenient dictionary file.
# Other platforms may need to provide their own word-list.
with open('/usr/share/dict/words') as f:
    words = [word.strip() for word in f]
    xkcd = ' '.join(choice(words) for i in range(4))

print(f'RANDOM PASSWORD: {password}')
print(f'  XKCD PASSWORD: {xkcd}')
```

## Extracting audio track from a video file

```bash
ffmpeg -i INPUT.mp4 -vn -acodec copy OUTPUT.m4a
```

## Fix brightness
My Linux Mint machine dims the screen if I unplug the power, which is a normal behavior for power management. The problem arises later when I plug the power back it, it doesn't restore the brightness to what it normally is. Hence, this hack:

```bash
#!/bin/bash
me=`whoami`
max=`cat /sys/class/backlight/radeon_bl0/max_brightness`
sudo chown ${me}:${me} /sys/class/backlight/radeon_bl0/brightness
sudo chmod o+x /sys/class/backlight/radeon_bl0/brightness
ls -al /sys/class/backlight/radeon_bl0/brightness
echo ${max} > /sys/class/backlight/radeon_bl0/brightness
sudo chmod 444 /sys/class/backlight/radeon_bl0/brightness
sudo chown root:root /sys/class/backlight/radeon_bl0/brightness
exit
```

## Fix Steam
If you start getting all kinds of library errors while trying to start a Steam game, this little command will most likely fix you up.

```bash
#!/bin/bash
find ~/.steam/steam/ubuntu12_32/steam-runtime \( -name "libgcc_s.so*" -o -name "libstdc++.so*" -o -name "libxcb.so*" -o -name "libgpg-error.so*" \) -print -delete
```

## Fix system keys
I don't know why, but there are times when updates just start failing because of bad keys. Here's a quick way of restoring your system back to normal.

```bash
#!/bin/bash
sudo apt-get clean
cd /var/lib/apt
sudo mv lists lists.old
sudo mkdir -p lists/partial
sudo apt-get clean
sudo apt-get update
```

## Fix updates
If your updates stop working, try this before you panic:

```bash
#!/bin/bash
sudo rm -r /var/cache/apt/var/lib/apt/lists
sudo apt-get update
```

## Format USB stick
Make sure you know what device your USB stick is before attempting this.
It's useful to run a `dmesg` right after connecting it, to see what was
detected.

```bash
#!/bin/bash
umount /dev/sdc
mkfs.vfat -I /dev/sdc
mount /dev/sdc
```

## Remove old kernels and headers
I ran into an issue trying to update to a new kernel. It turns out that the old kernels aren't removed by default so my /boot/ partition ran out of space! This little script will list all of the kernels that you are currently installed and display the command that you have to use to remove them.

> NOTE: Keep the most recent kernel and at least one more to fall back to in case of any issues. I just keep the two most recent ones.

```bash
#!/bin/bash
clear
echo "##################"
echo "# CURRENT KERNEL #"
echo "##################"
echo ""
uname -r
echo ""
echo "######################"
echo "# INSTRALLED KERNELS #"
echo "######################"
echo ""
dpkg --list | grep linux-image | awk '{print $2}'
echo ""
echo "######################"
echo "# INSTRALLED HEADERS #"
echo "######################"
echo ""
dpkg --list | grep linux-headers | awk '{print $2}'
echo ""
echo "Remove unwated kernels with:"
echo "  sudo apt autoremove"
echo "  sudo apt-get purge linux-image-x..."
```

## Remove old SSH Key
I keep forgetting this command, so I put it into a script. Although I think the system tells you how to remove it when it finds that the key has changed...

```bash
#!/bin/bash
ssh-keygen -f "/home/`whoami`/.ssh/known_hosts" -R $1
```

## Removing spaces from file names
For some tasks, it's easier to deal with files that don't have any spaces. Used the following to remove spaces and then put them back.

despace.sh
```bash
#!/bin/bash
j=`echo $1 | sed 's# #_#g' - `
mv "$1" "$j"
```

respace.sh
```bash
#!/bin/bash
j=`echo $1 | sed 's#_# #g' - `
mv "$1" "$j"
```

## Setup your git key-ring
```bash
#!/bin/bash

sudo apt-get install libgnome-keyring-dev
cd /usr/share/doc/git/contrib/credential/gnome-keyring
sudo make
git config --global credential.helper /usr/share/doc/git/contrib/credential/gnome-keyring/git-credential-gnome-keyring
```

## Show installed memory modules
```bash
#!/bin/bash
sudo dmidecode | awk '/^Memory\ Device$/,/Size/{if ($0~/Size/)print}'
```

## Show your IP address
```bash
#!/bin/bash
echo $(ifconfig | awk '/inet /{print $2}' | grep -v ^127)
```

## Show system serial number
Need your machine's serial number but the label is damaged or you're just too lazy to look, don't worry, I've got your back!

```bash
#!/bin/bash
sudo /usr/sbin/dmidecode -s system-serial-number
```

## Splitting up a large file
The `-segment_time` value is the amount of *seconds* that the audio segment should be.

```bash
ffmpeg -i INPUT.mp3 -f segment -segment_time 3600 -c copy %03d-OUTPUT.mp3
```

