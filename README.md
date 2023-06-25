Enjoy this easy tunable idea.<br>
It's a simple remote fact collector. <br>
No stress due to it's relative simplicity and endless tuning possibilities.<br>
The probe doesn't write anything to the disks of your hosts, requires only a minimal level of authorization required for your probes and it cleany show up in the log files.<br>

## Instruction. 
Clone this repo or put the code below in a file named <i>host_probe.sh</i> and make sure to run
```
chmod u+x host_probe.sh
```
Create a file (or edit) hosts.txt with you targets of investigation. <br>
Should look like:
```
Mycoolserver1.test.com
Mycoolserver2.test.com
```
Make sure you can ssh into your hosts using the names in hosts.txt, preferably with a key.<br>  
With this set up the code should work out of the box.<br>
See the inline examples to tune your probe.<br>
Watch out for \ and '" pitfall.<br> 
A small warning should be mentioned: security was not my focus when writing this article.<br>
<br>
Happy probing!

```
#!/bin/bash
# Name: Host_probe
# What it does: Host_probe is a relative simple script to collect host data for any purpose 
# Written by: Abby Eeninkwinkel
# Version 2.4 Juni 2023 2.4 (release for community)
# Todo: needs to collect possible error values from remote
# Ideas: lots :)

OLD_IFS="$IFS"
version="2.4"

# define your hosts here
# host rules.
# - name should resolve 
# - you should have ssh access with a key
# specify them inline
#    hosts=(server1, server2)
# or alternatively read them from a file as for now the default
hosts=$(cat hosts.txt)

# initialize variables
probe=""
probe_key=""
probe_value_action=""

add_to_probe () 
  {
   probe=$probe"printf '$probe_key '; $probe_value_action; printf '\n';"
  }

# construct your probe
# probe rules:
# - a probe line consist of pair of key name and return value(s)
# - both key value and return value may not contain any spaces

# Example 1: adding a probe timestamp when run on the remote host
probe_key="date"
probe_value_action="date +'%Y-%m-%dT%H:%M:%S,%3N'"
add_to_probe

# Example 2: probe a remote environment variable. 
# You must have "PermitUserEnvironment yes" in your remote ssh config file. Security issue!
# notice the escape \
probe_key="remote_pwd"
probe_value_action="printf \$PWD"
add_to_probe

# Example 3: probe the hostname\ip as found on the host.
probe_key="remote_host_name"
probe_value_action="cat /etc/hostname"
add_to_probe

# Example 4: get the size of a remote directory
# notice a return value should not have spaces 
# awk should be available on the remote system
probe_key="size_of_my_remote_home"
probe_value_action="du -hs | awk '{print \$1}'"
add_to_probe

# Example 5: get the size of multiple remote directories
for item in /disk1 /disk2 /disk3; do
   probe_key="size_of_$item"
   probe_value_action="du -hs $item | awk '{print \$1}'"   
   add_to_probe 
 done

echo "This how your probe looks like:"
echo $probe
echo " " 

# Probing and working with the probed data
IFS=$'\n'
for host_nm in ${hosts[@]}; do
    printf "======= Running Probe for host $host_nm =======\n"   
    result=$(ssh $host_nm "$probe")
    echo Result for host nost_nm
    for element in ${result[@]};  do
      str="${element}"
      # print each return line    
      printf $str
      # test/search for a field
      # notice that we use cut because of the current IFS 
      if [[ "$str" == "date "* ]]; then
         printf "  <= Found a date field!"
         value=$(echo "${str}" | cut -d ' ' -f2)
         printf " with value: %s" "$value"
      fi
      printf "\n"
    done
done

IFS="$OLD_IFS"   
echo "Ready"
exit
```
<HR>
Abby Eeninkwinkel 2023 
<hr>
