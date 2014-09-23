Prerequisite: 

sudo apt-get install terminator
or
sudo yum install terminator


termssh is a script to create and maintain gnome terminator layouts for ssh access either via:

 -g    | Auto groups servers per apptype conventions,

 -fs   | Full screen options, 

 -w 8  | window per tab definition of 2 4 or 8 windows per tab 
 
 -n "my layout" | this will set the layout name to a familiar name of your choice for future connections
 
 -vp 122 | This is for vpn users and this is what you define as your vpn port
 
 -v hostname | this is the vpn hostname used to connect through 

 -s "gateway-(lon|gla)[01-02] myql[01-03] apache01"    | in speech marks space seperated list
 
 -f filename  |  to connect to servers in a filename, can contain wild card naming like above

 -l {env} {apptypes} {service_type}   | to TEST autodiscovery of servers and show what servers are being generated
 
 -c to reconnect to existing layout | existing layouts listed and you can numerically choose which layout
 


## Testing termssh 

To understand the power of grouping -g simply place entire code in DEBUG mode, at the top of termssh script find DEBUG which is set to 0 change this to 1.
Save script then on command line type in:


## termssh.bc
Copy this file to /etc/bash_completion.d/termssh, the next shell session to your host will now process auto completion on termssh, this returns valid switches and 
allows you to connect to existing saved layouts, type in termssh press tab and a list of options are shown. Type in termssh conn press tab then tab again and it will list all saved profiles. 
You can start typing in some and pressing tab for it to complete the connection for you



## Normal pattern match using -s to define hosts - must be wrapped - Auto Discovers

# termssh -r -w 8 -g -n "agm1" -x 2 -s "gw-(lon|gla)[01-02] mql[01-03] apache01"

This will open 8 window terminator session with apaches in group 
The layout name will be called agm1 for future connections
The naming convention matches the scripts defined naming convention i.e. gw mql apa which then the -g grouping kicks
in and groups servers per app type
It will connect twice per server -x 2
Finally pattern match servers called gw-lon-01 gw-gla-01 gw-lon-02 gw-gla-02 mql01 mql02 mql03 and finally apache01
and then check if their available if so 2 terminator connections per host will be launched with 8 windows per tab 

In this instance my server names matched that naming convention which then got processed by group function below it.

Whilst DEBUG=1 at top of termssh

# termssh -r -w 4 -x 2 -g -s "apa[01-02]"

This will launch -x 2 i.e. connections to apa01 and apa02 twice for each host.
The grouping works out you have two connections to the host so assigns group-1 to apa01 apa02 first time around
and group apache-2 to apa01 and apa02 for 2nd set of connections.

This means you can now type in for example tail -f /var/log/messages on one of the apaches which sends it to one set of the servers
and then restart a service by typing it to 2nd instance of apa01 which gets sent to apa02 as part of group-2

hope it makes sense.



Enable debug mode at top of the script and try out:

# termssh -r -n "apa-gw-mql" -w 8 -x 2 -fs -g -s "apa[01-04] mql[01-02] gw[01-02]" space seperated list

then type in a command into any of the apache-1 sessions.

This will -r remove layout -w 8 try for 8 windows  and because -x = 2 this means 4 servers twice which equals the 8 windows defined full screen and will autogroup (-g)


 apache01 connection 1 will be part of apache-1 group

 apache01 connection 2 will be part of apache-2 group|

 apache02 connection 1 will be part of apache-1 group

 apache02 connection 2 will be part of apache-2 group

layout name for future connection:  apa-gw-mql


## -s Example with extra verbosity:

 # termssh.sh -n "bad" -r  -s "local(h|i)os[s-t] local(h|i)os[s-t]"
 
 [[mylayer]]

user you have issued -r : Layout bad has now been removed !!

Will recreate bad 

___ Blanked out tmp file /tmp/terminator-hosts-1000.collect.tmp

___ localhoss

___ localhost

___ localioss

___ localiost

___ localhoss

___ localhost

___ localioss

___ localiost

Total servers 2 | Tabs required 1 | LAYOUT= bad ./termssh1.sh -c to reconnect




##  File method - pattern matches

# termssh -r -w 8 -n "my mail servers" -f ./mailservers.txt

-f option to call the file and create layouts layout names will be the filenames so ensure you use different file names each time a new group is created.

-w 8 is windows and you can define 2 4 or 8 servers per tab

-r is to remove existing layout, required if you wish to refresh a layout in cases where new servers have been added to either text file or as part of network with auto discover. if -r is not given and layout already exists - the layout will be loaded instead of discovery or reading file

This will read each value in the filename and create a layout called mailservers which contains each server - 8 windows per tab
to reconnect in the future either run termssh -c or rerun above which locates existing layouts and auto connects|

layout name for future connection:  my mail servers

# termssh -r -w 8 -x 2  -fs  -f ./webservers.txt

-fs full screen mode - if fullscreen option at top is set to 0 then use this to make it full screen use -r in conjunction for existing layouts

-x 2 Amount of times to reconnect per host this also auto groups each server per run so two apache servers  with -x 2 = 4 servers group apache-1 for instance 1 of each server and group-2 for instance 2 of each server 


This will read each value in the webservers.txt file  and create a layout called webservers, it will then run through -x value and connect that many times to each host
Servers can be a combination of specific names or pattern match :
localhost

localhost

loca(l|h)[h-s]ost




## VPN Connections / SSH REVERSE TUNNEL Connections

This allows you to define a vpn host and port - the problem may come into effect for pattern match 
since if current connection requires a vpn to connect the script using nc will not be able to catch port 22
thus pattern match will fail

 termssh.sh -n "vpncon" -r -vp 222 -v windows_host -s "local(h|i)os[s-t] local(h|i)os[s-t]"
 
The above would probably fail - check_method at the top of script can be changed to ping so long as it can resolve short names locally
I would suggest for vpn to either use -f and have all servers listed in a text file or do something like:

# termssh.sh -n "vpncon" -r -vp 222 -v windows_host -s "server1 server2 server3 server4"
or

# termssh.sh -n "vpncon" -r -vp 222 -v windows_host -s "server[1-4]" 

So long as the pattern match matches exact server convention start/end. This would result in the expanded list matching initial input above it

This way it will do ssh -tt -p $VPN_PORT $VPN_SERVER -c "ssh $current_server" for each server in the list

 querymethod is automatically disabled for vpn connections which in short disable server validation 
 
 

## SSH Reverse tunnel connections:

remot_server issue: 
# ssh -R1999:localhost:22 vahid@my_local_host

To connect through my reverse tunnel same as vpn:

where -n is layout name

 -v is vpn host (included my ldap username)

 -vp is vpn port

 -s is server input within speech marks


#termssh -r -n "test2" -v "remote_username@localhost" -vp 1999 -s "apache01 apache02"

or

#termssh -r -n "test2" -v "remote_username@localhost" -vp 1999 -s "apache[01-10]"

Which will connect from apache01 to apache10 servers, the remote_username@localhost can seem confusing but you would define the username used on remote network that you are reversing from
the initial 



## Help 
# termssh -h 


options: 

-r  | --removelayout                            |  remove layout and refresh it

-fs | --fullscreen                              |  start session in full screen mode, Override FULL_SCREEN_MODE=0

-g | --group                            |  groups server as per naming set_apptype, Override AUTO_GROUPING_ENABLED=0

-w  | --windows  [2/4/8 ]                       | -w followed by 2 or 4 or 8 windows per tab

-x  | --times  [1-10 ]                          | -x followed by 1 to any number above - reconnect value per server

-n  | --layoutname  "something or another"      | -n followed by a layout name in speech marks if it has spaces

Followed by either:

-s  | --servers | "apache01 mysql[02-05][a-j] gateway(a|b)[01-02] oracle02"     | must be in speech marks if regex input.
  This will connect to initial server of apache01 then also connect to mysql02 to 05 a j so mysql02a - mysql05-j every combination
 followed by gatewaya01 gatewaya01 gatewaya02 gatewayb01 gatewayb02 finally oracle02 


You may use round brackets to look for either pattern and square brackets for wild card such as d-h or 5-10 which it will then go through this loop


-f  | --file [./mail.txt]                       | where mail.txt contains list of mail servers

Typical file could be single server names or like above brace expansion / regex : 
one server per line it can be either something like:

mail01

mysql-(lon|gla)-[01-03]


where it will connect to simple host as well as match all hosts of mysql-lon-01 mysql-gla-01 mysql-lon-02 mysql-gla-02 mysql-lon-03 mysql-gla-03


Usage: termssh [-h {for help}] | [-c {connect to existing layouts} ] | [-d {delete existing layout} ]

Usage: termssh [-l {for list servers} followed by prod/stage/uat atjmo app1 app2 app3 app4 app5 app6 ] 

Usage: termssh [-r remove layout and recreate ] [-fs {fullscreen mode} ] [-w {windows} 2/4/8] [-n "my file_layout name"] [-f {for file} filename ]

Usage: termssh [-r remove layout and recreate ] [-fs {fullscreen mode} ] [-w {windows} 2/4/8] [-n "my ad_layout name"]

Usage: termssh [-r remove layout and recreate ] [-fs {fullscreen mode} ] [-w {windows} 2/4/8] [-n "my input_layout name"] [-s {"server1 server[2-3] (web|gateway)[01-03]asd" } 



Command Line Input CLI connect example:



EXAMPLE 1: termssh -fs -x 2 -w 8 -n custom_server1 -s "(apache|mysql)-[01-03][a-b] gateway[01-03]" {Fullscreen mode, 8 windows per tab connect twice per server brace expand patterns and connect to

apache01a apache01b mysql01a mysql01b ... mysql01-03a/b servers as well as gateway01 gateway02 gateway03 set the layout name to custom_server1 for reconnection

EXAMPLE 2: termssh -r -w 2 -x 3 -s montct01,monapa01 {Remove layout 2 windows per tab X 3 comma seperated list i.e. 3 times to monstct01 and 3 times to monsapa01 2 per tab = 3 tabs } 



File connect example:



EXAMPLE 3: termssh -w 8 -f ./servers.txt {Go through servers.txt and connect to all with 8 windows per tab } 

EXAMPLE 4: termssh -r -w 8 -f ./servers.txt {Remove layout go through file and connect 8 windows per tab} 



Existing Layout connection:



EXAMPLE 9: termssh -c {List existing layouts and give u numeric option to connect to them} 


Removal of existing layout:



EXAMPLE 10: termssh -d {List existing layouts and give u numeric option to remove 1} 

