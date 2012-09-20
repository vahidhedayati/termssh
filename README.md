termssh is a script to create and maintain gnome terminator layouts for ssh access either via:

 -g    | Auto groups servers per apptype conventions,

 -fs   | Full screen options, 

 -w 8  | window per tab definition of 2 4 or 8 windows per tab 

 -n "my layout" | this will set the layout name to a familiar name of your choice for future connections
 
 -s "gateway-(lon|gla)[01-02] myql[01-03] apache01"    | in speech marks space seperated list
 
  -f filename  |  to connect to servers in a filename, can contain wild card naming like above

 -a {env} {apptypes} {service_type}   | to autodiscover servers and make layouts. 
 
 
 -l {env} {apptypes} {service_type}   | to TEST autodiscovery of servers and show what servers are being generated
 
 -c to reconnect to existing layout | existing layouts listed and you can numerically choose which layout
 


## Testing termssh 

To understand the power of grouping -g simply place entire code in DEBUG mode, at the top of termssh script find DEBUG which is set to 0 change this to 1.
Save script then on command line type in:


# termssh -r -w 8 -g -n "agm1" -x 2 -s "gw-(lon|gla)[01-02] mql[01-03] apache01"

This will open 8 window terminator session with apaches in group 
The layout name will be called agm1 for future connections
The naming convention matches the scripts defined naming convention i.e. gw mql apa which then the -g grouping kicks
in and groups servers per app type
It will connect twice per server -x 2
Finally pattern match servers called gw-lon-01 gw-gla-01 gw-lon-02 gw-gla-02 mql01 mql02 mql03 and finally apache01
and then check if their available if so 2 terminator connections per host will be launched with 8 windows per tab 

Once you have seen this working you can look further in the script and see I have defined group names as per application type i.e. 2nd input of auto discovery.
In this instance my server names matched that naming convention which then got processed by group function below it.

Whilst DEBUG=1 at top of termssh

# termssh -r -w 4 -x 2 -g -s "apa[01-02]"

This will launch -x 2 i.e. connections to apa01 and apa02 twice for each host.
The grouping works out you have two connections to the host so assigns group-1 to apa01 apa02 first time around
and group apache-2 to apa01 and apa02 for 2nd set of connections.

This means you can now type in for example tail -f /var/log/messages on one of the apaches which sends it to one set of the servers
and then restart a service by typing it to 2nd instance of apa01 which gets sent to apa02 as part of group-2

hope it makes sense.

use: 
# termssh -l prod t gw 
# termssh -l prod ta ab
# termssh -l prod tajmo ab

with our without DEBUG mode being enabled to understand how the auto discovery works and then feel free to hack the prod definitions the tajmo which stands for tomcat,apache,jboss,mysql,oracle and then finally ending convention which gets mapped with a-z of characters before it.





## 1. INPUT SERVERS
Enable debug mode at top of the script and try out:

# termssh -r -n "apa-gw-mql" -w 8 -x 2 -fs -g -s "apa[01-04] mql[01-02] gw[01-02]" space seperated list

then type in a command into any of the apache-1 sessions.

This will -r remove layout -w 8 try for 8 windows  and because -x = 2 this means 4 servers twice which equals the 8 windows defined full screen and will autogroup (-g)


 apache01 connection 1 will be part of apache-1 group

 apache01 connection 2 will be part of apache-2 group|

 apache02 connection 1 will be part of apache-1 group

 apache02 connection 2 will be part of apache-2 group

layout name for future connection:  apa-gw-mql
## 2. File method:

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



## 3. Auto discovery method:

Auto discover method 1 - a :

There is a large segment at the top of the code to allow you to configure this section, to be honest I have worked in quite a few places and naming conventions vary hugely so I am afraid you may need to get hands dirty and hack the code around a bit to auto discover for you, my advice is to tweak sections that deals with auto discovery:

-a {environment} {app_types_first_chars} {app_end_naming}

so

-a

[ prod uat stage ] - this is then configured in script to define locations according to environment and set up numeric convention ---> londons 01 | 02 || londong 04

[app_type ] tamoj which stands for Tomcat Apache Mysql Oracle and Jboss - each of which have been defined in the top part of script as variables ---> apa | tct | jbs | mql {londons}{apa}{02} you can define one or all of the tamjo combinations which validate servers uses to find servers matching patterns

[app_ending_name] if your servers are set up to end with ml for mail gw for gateway it would end up as | {londons}{apa}{02}[a-z]{ml} or {londons}{apa}{02}[a-z]{gw} the a-z is what validate servers does goes from londonsapa01aml all the way to londonsapa01zml trying to execute check_method to see if it exists

As I have said this segment is specific to current environment and you will need to really have a go if you want to sit back and auto discover stuff

Auto discover method 1 -a

# termssh -r -w 4 -n "prod_mail" -a prod ta ml 

{which will rediscover 4 windows per tab and load londonstct01{a-z}ml and londonsapa01{a-z}ml mail servers so long as it found them

This will now be called layout prod_mail


Auto discover method 2 -ad

# termssh -r -w 4 -ad mx[1-5]xml - this will do a lookup against mx1xml to mx5xml if existant it will connect to them


## Connection / Removal of existing layouts

# termssh -c

 Connect to existing layout - this will list your layouts and you can connect to what is already saved

# termssh -d 

To remove existing layout - again it will list items and numerically select layout to be removed




If all auto discovery found or file contains 1,2,3 or 4 servers it will still create the layout and will be useable, 3 windows is flakey since it opens a spare one. Windows can be moved within tabs.





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

-a  | --autodiscover [val1] [val2] [val3]       | explained in detail below

Usage: termssh [-h {for help}] | [-c {connect to existing layouts} ] | [-d {delete existing layout} ]
Usage: termssh [-l {for list servers} followed by prod/stage/uat atjmo app1 app2 app3 app4 app5 app6 ] 
Usage: termssh [-r remove layout and recreate ] [-fs {fullscreen mode} ] [-w {windows} 2/4/8] [-n "my file_layout name"] [-f {for file} filename ]
Usage: termssh [-r remove layout and recreate ] [-fs {fullscreen mode} ] [-w {windows} 2/4/8] [-n "my ad_layout name"] [-a {for autodiscovery} prod/stage/uat atjmo app1..app6 ] 
Usage: termssh [-r remove layout and recreate ] [-fs {fullscreen mode} ] [-w {windows} 2/4/8] [-n "my input_layout name"] [-s {"server1 server[2-3] (web|gateway)[01-03]asd" } 

Command Line Input CLI connect example:

EXAMPLE 1: termssh -fs -x 2 -w 8 -n custom_server1 -s "(apache|mysql)-[01-03][a-b] gateway[01-03]" {Fullscreen mode, 8 windows per tab connect twice per server brace expand patterns and connect to
 apache01a apache01b mysql01a mysql01b ... mysql01-03a/b servers as well as gateway01 gateway02 gateway03 set the layout name to custom_server1 for reconnection
EXAMPLE 2: termssh -r -w 2 -x 3 -s montct01,monapa01 {Remove layout 2 windows per tab X 3 comma seperated list i.e. 3 times to monstct01 and 3 times to monsapa01 2 per tab = 3 tabs } 

File connect example:

EXAMPLE 3: termssh -w 8 -f ./servers.txt {Go through servers.txt and connect to all with 8 windows per tab } 
EXAMPLE 4: termssh -r -w 8 -f ./servers.txt {Remove layout go through file and connect 8 windows per tab} 

Auto discovery connect examples:

termssh -a {environment} {apptype} {applications}
{environment} can be: prod/stage/uat
{apptype} can be: matjo includes msyql/tomcat/apache/jboss/oracle use 1 or all or a combination
{applications} are seperated by spaces i.e. gw td iw and so forth up to 6 environements accepted

EXAMPLE 5: termssh -r -w 8 -fs -x 3 -a prod at td {Remove layout, rediscover and connect to londons(tct/apa)01[a-z]{td|at} 3 times, 8 windows per tab, fullscreen}
EXAMPLE 6: termssh -w 8 -a prod gw {Connect to londons(apa/jbs)01[a-z]gw and try for 8 windows per tab}
EXAMPLE 7: termssh -x 2 -a prod at bh gw {Connect to londons(apa/tct)01[a-z]bh + londons(apa/tct)01[a-z]gw twice per server}

Auto discovery list example:

EXAMPLE 8: termssh -l prod at sd fg {List all servers connectable on londons(apa/tct)01[a-z]sd + londons(apa/tct)01[a-z]fg} 

Existing Layout connection:

EXAMPLE 9: termssh -c {List existing layouts and give u numeric option to connect to them} 

Removal of existing layout:

EXAMPLE 10: termssh -d {List existing layouts and give u numeric option to remove 1} 
