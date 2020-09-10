# 2020-09-10 Setting Up

* 🎵 Playlist: https://www.youtube.com/watch?v=pkIrKAZNRCU&ab_channel=MarcRebillet

* Digital Ocean login through google
* Setting up basic $5 a month shared vcpu droplet, will use until it's not big enough
  * Would probably move up the basic plan until it's $40, then move to CPU Optimised $40 plan
* Not setting up VPC, if I find I need a dedicated database box, can snapshot and move
* Using `Ubuntu 20.04.1`
* Added `Lukes Laptop` ssh key
* IP: `188.166.222.208`
* Added `lbat.ch` domain to droplet
* Need to [update nameservers to point to DO](https://www.digitalocean.com/community/tutorials/how-to-point-to-digitalocean-nameservers-from-common-domain-registrars)
  * Has guide specifically for namecheap 👌
* in namecheap, update namesevers to:
  * `ns1.digitalocean.com`
  * `ns2.digitalocean.com`
  * `ns3.digitalocean.com`
* Subdomains can now be set up in Digital Ocean
* Able to ssh in `ssh root@188.166.222.208` using ssh keys
* Installing zsh `sudo apt-get install zsh`
* Created new user `adduser lbatch` (password in lastpass)
  * Confirmed does not give ssh access, not sure if will use this yet
* Installed ohmzsh `sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`
* Modified prompt to add username and host machine. Started with PROMPT from `$ZSH/themes/robbyrussell.zsh-theme`
```bash
PROMPT="%(?:%{$fg_bold[green]%}➜ :%{$fg_bold[red]%}➜ )"
PROMPT+=' %{$fg[cyan]%}%c%{$reset_color%} $(git_prompt_info)'
```
* echo -e "\e[32mRed Text\e[0m"
* Created custom motd in `/etc/update-motd/99-ascii-art`

```
#!/bin/sh
  
echo "\e[32m"
echo "██╗     ██████╗  █████╗ ████████╗ ██████╗██╗  ██╗"
echo "██║     ██╔══██╗██╔══██╗╚══██╔══╝██╔════╝██║  ██║"
echo "██║     ██████╔╝███████║   ██║   ██║     ███████║"
echo "██║     ██╔══██╗██╔══██║   ██║   ██║     ██╔══██║"
echo "███████╗██████╔╝██║  ██║   ██║██╗╚██████╗██║  ██║"
echo "╚══════╝╚═════╝ ╚═╝  ╚═╝   ╚═╝╚═╝ ╚═════╝╚═╝  ╚═╝"
echo "\e[0m"
```
* font from [http://patorjk.com(http://patorjk.com/software/taag/#p=display&h=1&v=0&f=ANSI%20Shadow&t=lbat.ch%0A%0A)
* Set up rmate for remote file editing in vscode
* Install  [Remote VScode extension](https://marketplace.visualstudio.com/items?itemName=rafaelmaiolla.remote-vscode)
* Install [rmate](https://github.com/aurora/rmate) on server

```
sudo wget -O /usr/local/bin/rmate https://raw.githubusercontent.com/aurora/rmate/master/rmate
sudo chmod a+x /usr/local/bin/rmate
```
* Now can start ssh session with remote forwarding `ssh -R 52698:localhost:52698 root@188.166.222.208`
* Can set it up in local `~/.ssh/config`:

```
Host lbat.ch
    RemoteForward 52698 127.0.0.1:52698
```
* Looked at serveral other reverse proxies and did not like them...
  * [lighttpd](https://redmine.lighttpd.net/projects/lighttpd/wiki/TutorialConfiguration) was the best but didn't standout. Would be the only other one I'd consider
  * [haproxy](http://www.haproxy.org/) - literally has perf data from 2007 on their hompage
  * [traefik](https://github.com/containous/traefik) - looks super cool but needs something like kubernetes in the background
  * [varnish](https://varnish-cache.org/docs/index.html) - actually looks okay, has a webfrontend that might be worth looking at. Took a while to actually get to the docs...

* Set up nginx, follow [this guide](https://www.hostinger.com/tutorials/how-to-set-up-nginx-reverse-proxy/)

```
sudo apt-get install nginx
sudo unlink /etc/nginx/sites-enabled/default
cd /etc/nginx/sites-available/
mkdir -p /home/lbatch/sites/lbat.ch
touch /home/lbatch/sites/lbat.ch/index.html
sudo rmate /etc/nginx/sites-available/lbat.ch
```

* Config based on `/etc/nginx/sites-available/default`
```
server {
      listen 80;
      listen [::]:80;

      server_name lbat.ch;

      root /home/lbatch/sites/lbat.ch;
      index index.html;

      location / {
              try_files $uri $uri;
      }
}
```

* test config using `service nginx configtest` - seems to fail...
* restart nginx using `service nginx restart`
* site is up! [http://lbat.ch/](http://lbat.ch/)
* nginx docs to read:
  * https://www.nginx.com/resources/wiki/start/
  * https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
  * https://wiki.debian.org/Nginx/DirectoryStructure
* set up ssl with certbot using [this guide](https://certbot.eff.org/lets-encrypt/ubuntuxenial-nginx.html)

```
sudo certbot --nginx
sudo certbot certonly --nginx
# cert saved to /etc/letsencrypt/live/lbat.ch/fullchain.pem
# key saved to /etc/letsencrypt/live/lbat.ch/privkey.pem
# expires 2020-12-09
# says to backup /etc/letsencrypt "Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now"

# testing auto-renewal
sudo certbot renew --dry-run
```

* Deploying an app, staging generozity
* Creating an ssh key for deployments

```
ssh-keygen -t rsa -b 4096 -C "batchelor.luke@gmail.com"
# wouldn't let me write to /home/lbatch/.ssh/deploy-key so had to run as sudo
# and change owner
```
* Copied public key into [github deploy keys](https://github.com/lukebatchelor/generozity/settings/keys)
* Need a `.ssh/config` to point to that key (copying from generozity-staging)

```
Host github.com
    IdentityFile ~/.ssh/deploy-key
    IdentitiesOnly yes
```
* cloning repo `cd ~/sites && git clone git@github.com:lukebatchelor/generozity.git`
* installing [nvm](https://github.com/nvm-sh/nvm) `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash`
* Added completions to `~/.zshrc` manually instead of bash_profile
* Installing node `nvm install 12.16.1`
* Installing [yarn](https://classic.yarnpkg.com/en/docs/install/) `curl -o- -L https://yarnpkg.com/install.sh | bash`
* Will set the rest up another time