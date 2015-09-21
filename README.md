#modify-ip-zone

Version: *1.0*
<br />
Author: *William Stearns - @william-stearns - wstearns@pobox.com*


This Ruby script adds or removes IP addresses from an IP zone that is used in a Halo firewall policy. 

You can use this tool with an ip blacklisting or whitelisting tool, or simply to automate operations across a large cloud deployment.



##List of Files

* modify-ip-zone.rb: The Ruby script file.
* README.md: This ReadMe file.



##Requirements and Dependencies

* A CloudPassage Halo account with API access
* Ruby library wlslib.rb (available here: LINK)
* The Ruby interpreter must be installed on the machine that executes the script.
* The following Ruby gems are required: oauth2, rest-client, json, date, and optparse. You can use this command to install them:

        sudo gem install oauth2 rest-client json public_suffix ip


##Installation 

1. Copy modify-ip-zone.rb and wlslib.rb to the same directory in your path.  
1. In the Halo portal, create an IP zone called fail2ban-ssh .  Give it a dummy address of 169.254.255.255.
1. Configure firewall policy:  create rules for authorized SSH users first, to be sure they can get in even if fail2ban mistakenly adds one of their addresses.  Below those rules, add a rule with a source of the "fail2ban-ssh" IP zone, a service of"ssh" and Action DROP.
1. Create a full access Halo API key in the Halo Portal, then edit /etc/halo-api-keys on the system that will run this script and add your key on its own line, with the key ID and secret separated by a vertical pipe, like this:

        aabb3344|0123456789abcdeffedcba9876543210
1. Check that the /etc/halo-api-keys  file is owned by the user running the script, and mode is 600.
1. Modify the configuration of fail2ban so that it stops making changes to the firewall directly, and instead calls out to external programs to add and remove addresses.  Use these commands:
    modify-ip-zone.rb -i aabb3344 -z fail2ban-ssh -a <ip>
    modify-ip-zone.rb -i aabb3344 -z fail2ban-ssh -r <ip>


##Usage

You can run this script manually to add or remove addresses.

The zone you create must have between 1 and 375 addresses.

###*Example Commands*
* To see the command line help
```
modify-ip-zone.rb -h
```

* To add an address to a zone:
```
modify-ip-zone.rb -i aabb3344 -z fail2ban-ssh -a 6.7.8.9
```

* To remove an address from a zone:
```
modify-ip-zone.rb -i aabb3344 -z fail2ban-ssh -r 6.7.8.9
```

* You can add and remove multiple addresses with one invocation:
```
modify-ip-zone.rb -i aabb3344 -z fail2ban-ssh -a 9.8.7.6/28 -a 1.1.2.2/30 -r 3.5.7.9
```

* To ignore the current contents of the zone and start off with an empty list:
```
modify-ip-zone.rb -i aabb3344 -z fail2ban-ssh --empty -a 9.8.7.6/28
```

* To add a list of addresses, one per line, from stdin:
```
echo -e '169.254.0.1\n169.254.1.0/24' | ./modify-ip-zone.rb -i aabb3344 -z fail2ban-ssh --empty --add-stdin
```

* To remove a list of addresses supplied on stdin:
```
echo -e '169.254.0.1' | ./modify-ip-zone.rb -i aabb3344 -z fail2ban-ssh --remove-stdin
```

* If your zone has spaces or other characters that might be processed by the shell, surround it with quotes:
```
modify-ip-zone.rb -i aabb3344 -z '|My Very odd zone|!!' -r 6.7.8.9
```

* Example of loading a public blacklist into an IP zone
```
curl -s -o - https://isc.sans.edu/block.txt | sed -e 's/#.*//' | egrep -v '(^$|^Start)' | awk '{print $1 "/" $3}' | ./modify-ip-zone.rb -i aabb3344 -z isc-blocklist --empty --add-stdin
```


###*Troubleshooting*

* To see addresses as they're added, you can refresh the following URL: https://portal.cloudpassage.com/firewall_zones

* If you're ever concerned that this zone is interfering with normal ssh, edit any firewall policies using this zone and uncheck the "Active" box to the left of any firewall rules using this zone.  When you save your changes, new firewalls will be pushed out to these servers. Remember to re-activate after troubleshooting is complete.

## License

Copyright (c) 2013, CloudPassage, Inc.
All rights reserved.

Redistribution and use in source and binary forms, with or without modification,
are permitted provided that the following conditions are met:
    * Redistributions of source code must retain the above copyright
      notice, this list of conditions and the following disclaimer.
    * Redistributions in binary form must reproduce the above copyright
      notice, this list of conditions and the following disclaimer in the
      documentation and/or other materials provided with the distribution.
    * Neither the name of the CloudPassage, Inc. nor the
      names of its contributors may be used to endorse or promote products
      derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL CLOUDPASSAGE, INC. BE LIABLE FOR ANY DIRECT,
INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING,
BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED ANDON ANY THEORY OF
LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE
OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF
ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.


<!---
#CPTAGS:community-unsupported integration automation
#TBICON:images/ruby_icon.png
-->
