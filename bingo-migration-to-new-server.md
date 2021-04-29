# When I tried to migrate the backend for BINGO
from an AWS server to an Oracle Cloud Compute Instance

Everything was done perfectly!

The code was cloned and npm install done neatly

supervisor was installed and configured as process manager for the backend

nginx was installed and configured as the reverse proxy

The real issue happened when I tried to run `certbot` for TLS connection

The acme-challenge was not working

### WTF is an acme-challenge ?
I far as I know, certbot generates some kind of token and stores it in some folder. Then it sends a request using the domain which we requested for TLS.

This request should return the token. 

*But it wasn't :|*

### So who is the culprit ?
During the first few hours of my debugging, My intution was, it was certbot, who isn't generating such a file. That's why it isn't returning the token file.

I tried accessing that URL via browser, the issue persist.


The folder in which this token is being generated was unknown to me. So I started searching for that.

But I thought better of finding that folder and found a way to specify that folder when I run `certbot`.

I created a folder  `/home/ubuntu/www/letsencrypt` and ran this: 

`sudo certbot --nginx --webroot -w /home/ubuntu/www/letsencrypt`


Still it was failing!


Certbot isn't generating the token file?


To confirm this I placed an html file inside that folder and tried accessing that. You know what? No html was in the response. Still `refused to connect`

So it aint certbot. That folder is not accessible via http.

I updated the nginx config, so that if the url was like what certbot is requesting for, then it would return that file.

***Still certbot was raising the acme-challenge issue!***

### As usual, Sid with the clue
I contacted one of my friends and he asked if any packet was reaching the machine.
I did a tcpdump on port 80. Gladly packets were reaching my machine. So it is Nginx who isn't getting any. (I checked the nginx access.log and error.log, I didn't find any relevant trace)

After a long time, when my browser was running out of space to show tabs, Some random stackoverflow post showed up. 

The guy who posted the answer claimed 2 commands will do the magic.

***And they did***

The commands were these:
```
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 80 -j ACCEPT
sudo netfilter-persistent save
```
I was running out of hope. So I didn't think updating the iptables might work. Anyway I tried these and BOOOOM !

Nginx got the request and it responded with the html file I had configured.


### Epilogue? 
After that boom, I did the `certbot` command and the certificate was created. 

***PEACEFUL was the atmosphere***

Then I remembered the real cause I was there for. BINGO.

I tried creating a ROOM for online match. Unfortunately still it was not working :(

I thought this might take some time, but to my surprise, it didn't. 

I wrote a small TS script for connecting to this server and creating a room. The error I got was `ECONNREFUSED`

Why is it refusing the connection?

For a moment I looked at the line below that and that's when I was relieved.

I did the iptables `spell` for port 80. Not 443.
