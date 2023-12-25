# Ssh to localhost
Yesterday, i tried few ways
I did install Ubuntu via WSL and it worked fine
Today, i found these solution
[Medium Blog](https://medium.com/ci-cd-devops/ssh-receive-packet-type-51-154288e46609)
[SuperUser thread](https://superuser.com/questions/1137438/ssh-key-authentication-fails)
So the answer is, i was messing with read permission of the authorized_keys file and it fucked
BTW, the ultimate resolution is
```
YOURUSER=$(whoami)
sudo chown $YOURUSER:$YOURUSER /home/$YOURUSER/{.,.ssh/,.ssh/authorized_keys}
sudo chmod u+rwX,go-rwX,-t /home/$YOURUSER/{.ssh/,.ssh/authorized_keys}
sudo chmod go-w /home/$YOURUSER/
```
Perfecto!

# Remove snapd from Ubuntu
Funny thing is firefox is default installed in Ubuntu
And run 
```
sudo apt install firefox 
```
will eventually install snapd as well
If you want remove snapd, u have to remove firefox first
But it got stucked by 

> rm: cannot remove '/var/snap/firefox/common/host-hunspell/en_US.dic': Read-only file system
> rm: cannot remove '/var/snap/firefox/common/host-hunspell/en_US.aff': Read-only file system

I found this [Thread](https://forum.snapcraft.io/t/unlinkat-var-snap-firefox-common-host-hunspell-en-us-aff-read-only-file-system/34022/2)
So I can run this
```
umount /var/snap/firefox/common/host-hunspell
```
Boom! you can run
```
sudo apt autoremove --purge snapd
```
Back to this [Thread](https://askubuntu.com/questions/1035915/how-to-remove-snap-from-ubuntu)