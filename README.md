termssh is a script to create and maintain gnome terminator layouts for ssh access either via -s s1,s2,s3,s4 .. ie given list of servers, -f filename to connect o servers in a filename or via -a {env} {apptypes} {service_type} to autodiscover servers and make layouts. Auto groups servers per apptype conventions, Full screen options, window per tab definitio of 2 4 or 8 windows per tab 

It opens terminator with 2,4 or 8 windows per tab.

Will need configuration and tweaking if you wish for auto discovery to work in your work place, For now you can either put one host per ine into a text file and call it something like web.txt mail.txt or define servers comma seperated after -s argument

## 1. INPUT SERVERS

# termssh -r -w 8 -x 2 -fs -s apache01,apache02,mysql01,gateway01   
{comma seperated list of servers}


This will -r remove layout -w 8 try for 8 windows  and because -x = 2 this means 4 servers twice which equals the 8 windows defined full screen 


 apache01 connection 1 will be part of apache-1 group

 apache01 connection 2 will be part of apache-2 group|

 apache02 connection 1 will be part of apache-1 group

 apache02 connection 2 will be part of apache-2 group


## 2. File method:

# termssh -r -w 8 -f ./mailservers.txt

-f option to call the file and create layouts layout names will be the filenames so ensure you use different file names each time a new group is created.

-w 8 is windows and you can define 2 4 or 8 servers per tab

-r is to remove existing layout, required if you wish to refresh a layout in cases where new servers have been added to either text file or as part of network with auto discover. if -r is not given and layout already exists - the layout will be loaded instead of discovery or reading file

This will read each value in the filename and create a layout called mailservers which contains each server - 8 windows per tab
to reconnect in the future either run termssh -c or rerun above which locates existing layouts and auto connects|


# termssh -r -w 8 -x 2  -fs  -f ./webservers.txt

-fs full screen mode - if fullscreen option at top is set to 0 then use this to make it full screen use -r in conjunction for existing layouts

-x 2 Amount of times to reconnect per host this also auto groups each server per run so two apache servers  with -x 2 = 4 servers group apache-1 for instance 1 of each server and group-2 for instance 2 of each server 


This will read each value in the webservers.txt file  and create a layout called webservers, it will then run through -x value and connect that many times to each host



## 3. Auto discovery method:

There is a large segment at the top of the code to allow you to configure this section, to be honest I have worked in quite a few places and naming conventions vary hugely so I am afraid you may need to get hands dirty and hack the code around a bit to auto discover for you, my advice is to tweak sections that deals with auto discovery:

-a {environment} {app_types_first_chars} {app_end_naming}

so

-a

[ prod uat stage ] - this is then configured in script to define locations according to environment and set up numeric convention ---> londons 01 | 02 || londong 04

[app_type ] tamoj which stands for Tomcat Apache Mysql Oracle and Jboss - each of which have been defined in the top part of script as variables ---> apa | tct | jbs | mql {londons}{apa}{02} you can define one or all of the tamjo combinations which validate servers uses to find servers matching patterns

[app_ending_name] if your servers are set up to end with ml for mail gw for gateway it would end up as | {londons}{apa}{02}[a-z]{ml} or {londons}{apa}{02}[a-z]{gw} the a-z is what validate servers does goes from londonsapa01aml all the way to londonsapa01zml trying to execute check_method to see if it exists

As I have said this segment is specific to current environment and you will need to really have a go if you want to sit back and auto discover stuff

AUTO DISCOVER

# termssh -r -w 4 -a prod ta ml 

{which will rediscover 4 windows per tab and load londonstct01{a-z}ml and londonsapa01{a-z}ml mail servers so long as it found them

Connection / Removal of existing layouts

# termssh -c

 Connect to existing layout - this will list your layouts and you can connect to what is already saved

# termssh -d 

To remove existing layout - again it will list items and numerically select layout to be removed

If all auto discovery found or file contains 1,2,3 or 4 servers it will still create the layout and will be useable, 3 windows is flakey since it opens a spare one. Windows can be moved within tabs.





## Help 
# termssh -h 


options: 

-r  | --removelayout                            |  remove layout and refresh it

-fs | --fullscreen                              |  start session in full screen mode

-w  | --windows  [2/4/8 ]                       | -w followed by 2 or 4 or 8 windows per tab

-x  | --times  [1-10 ]                          | -x followed by 1 to any number above - reconnect value per server

Followed by either:

-s  | --servers |[s1,s2,s3]                     | where s1,s2,s3 is server name seperated by commas

-f  | --file [./mail.txt]                       | where mail.txt contains list of mail servers

-a  | --autodiscover [val1] [val2] [val3]       | explained in detail below


Usage: termssh [-h {for help}] | [-c {connect to existing layouts} ] | [-d {delete existing layout} ]

Usage: termssh [-l {for list servers} followed by prod/stage/uat atjmo app1 app2 app3 app4 app5 app6 ] 

Usage: termssh [-r remove layout and recreate ] [-fs {fullscreen mode} ] [-w {windows} 2/4/8] [-f {for file} filename ]

Usage: termssh [-r remove layout and recreate ] [-fs {fullscreen mode} ] [-w {windows} 2/4/8] [-a {for autodiscovery} prod/stage/uat atjmo app1..app6 ] 

Usage: termssh [-r remove layout and recreate ] [-fs {fullscreen mode} ] [-w {windows} 2/4/8] [-s {server1,server2,server3} 


Command Line Input CLI connect example:

EXAMPLE 1: termssh -fs -x 2 -w 8 -s server1,server2,server4..server8
 {Fullscreen mode, 8 windows per tab connect twice per server to comma seperated list of servers given = 16 windows 2 tabs } 

EXAMPLE 2: termssh -r -w 2 -x 3 -s montct01,monapa01 
{Remove layout 2 windows per tab X 3 comma seperated list i.e. 3 times to monstct01 and 3 times to monsapa01 2 per tab = 3 tabs } 



File connect example:

EXAMPLE 3: termssh -w 8 -f ./servers.txt 
{Go through servers.txt and connect to all with 8 windows per tab } 

EXAMPLE 4: termssh -r -w 8 -f ./servers.txt 
{Remove layout go through file and connect 8 windows per tab} 



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

EXAMPLE 9: termssh -c 
{List existing layouts and give u numeric option to connect to them} 



Removal of existing layout:

EXAMPLE 10: termssh -d
 {List existing layouts and give u numeric option to remove 1} 

