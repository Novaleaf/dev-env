- [dev-env](#dev-env)
- [dev environment](#dev-environment)
- [gcp devops](#gcp-devops)
  - [key based auth](#key-based-auth)
  - [setup a bastion host](#setup-a-bastion-host)
    - [- ***use byobu***](#--use-byobu)
    - [- or use SSH](#--or-use-ssh)
    - [- do not use vnc or teamviewer](#--do-not-use-vnc-or-teamviewer)




# dev-env
steps to setup a good typescript dev environment on windows, mac, or linux

# dev environment 

- nvm
  - windows https://github.com/coreybutler/nvm-windows
  - MAC/LINUX https://github.com/creationix/nvm
- vscode https://code.visualstudio.com/
- git https://git-scm.com/   
- keybase https://keybase.io/ *(required only to access secure repos)*
   1. enable file+git capabilities
   2. reboot your computer
   3. join the novaleaf_keybase.ops team to access secstore subrepo
- sourcetree
- gcloud sdk *(required only if interacting with google cloud technologies)*
- install important node packages
	- shelljs to run scripts https://github.com/shelljs/shelljs
		- ```npm install -g shelljs```  
	- typescript
  	- ```npm install -g shelljs typescript mocha node-dev```
- winscp https://winscp.net/eng/index.php
- vnc viewer: http://tightvnc.net/download.php
  - used to connection to bastion instances


# gcp devops

gcp = google cloud platform.  our cloud provider of choice.

## key based auth 

if you want to use 3rd party tools (not gcloud.exe)

- an example using ```winscp``` on ```windows```
   - install winscp, make sure to also choose pageant in the options
   - have your private key you want to use to login.  
   - add your key to pageant
   - in your gcp project, go to ```Compute Engine``` --> ```Metadata```
   - click ```SSH Keys```, then ```Edit```
   - add the key, specifying the gcp user it'll be associated with.   for example:
      ```ssh
      ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAgEA4+fjC1OwrBCjC7zsZtj8fjTZzKnCSo5ygkduYAg7hCO425NTPkjMEoMLLQjyEJuIrrnTBFA8AIaID+8r2uzPNvRxPNj3KGG/xpNuhvD2m2VBVWoVCpzRa8AjdSOgA8Oo4hkUFt2MuMF4bq5/jmVdpuCS6Xpw6n2+yvjOfAm5pn+Cm8LC2As08MLCTfFlFLN9NjIyDu4j0VxnV6LWMBbCRdDK8U1Ju54VbQGAqZJPekdO/K+o+uBrTr3FRjFi0zGMfoj+k7ugGZw2COCtBISe7B1DKqXiYTMQuMJHZldYNsNb9f9klfW54WPbuFLl3zzKx2OVr91+/w1i7mLzB8ervw+F+lHAEMFU885A4eJnxv44jd92EiZqoJN27TrHK+xxYSfpyVKEB99PXwkb9IpEz67Dlx+ml6FJlH9nX8nAWmFUzXVSL8CdhbGiSd/8JaGssQkOU2yjVoSLwiHRuQD61PL+K5/WDjdIArrD1WQUgXGpRwtUbaBQJr1w65w7p1nlowWAOl6HWALGtj/08lgfkiXbDC9A8au0s+pjyvfFo7j391T0+Xocv4+SFtUI2/MpfnIxR9T0dBuo0eNOBYmS49X1LcQeZA76fNKifnRrT4Nwk4fFCjEWcuOSQ0UXrGShV2RDznDNxh7Hs9AYxcPF+SrvA2RVVmPdfFRNWNPii80= jasons@novaleaf.com
      ```
   - connect to the server with ```winscp```, use the short username not full email (in the above example key, you would use ```jasons``` )

## setup a bastion host

interact with vm's without external ip's

### - ***use byobu***
If you can do everything you need via console (no gui), use byobu.  
- http://byobu.co/
- https://www.digitalocean.com/community/tutorials/how-to-install-and-use-byobu-for-terminal-management-on-ubuntu-16-04
- once you figure out byobu, follow the instructions in the SSH subsection below

### - or use SSH
you can SSH to the bastion host and then SSH to the target's internal ip address:
- ```gcloud compute ssh bastion-us-c1```
- Once you are connected: 
   - If you need to persist your session (risk getting disconnected) you can now  ```byobu``` (or ```tmux``` or ```screen```)
  - next you can ```gcloud compute ssh target --internal-ip```  (see https://cloud.google.com/sdk/gcloud/reference/beta/compute/ssh#--internal-ip)
      - or if you are using normal ```ssh``` you can ```ssh -A INTERNAL_IP``` ***within 2 minutes of the first ssh call*** due to key expiration, as described here: https://cloud.google.com/compute/docs/ssh-in-browser#known_issues


### - do not use vnc or teamviewer
- teamviewer+gnome method is stuck at a low res (1300x780 or something)
- vnc+gnome method (ubuntu 1804 default) uses too much ram and seems to have a memory leak (1.4gb on idle after an hour)




