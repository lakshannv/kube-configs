# Mount and Configure a SMB Share in Ubuntu

# **On the Host Machine**

## 1. Installing Samba
```
sudo apt update
sudo apt install samba
```

## 2. Setting up Samba
### Folder to be mounted
Now that Samba is installed, we need to create a directory for it to share:
```
mkdir shared
```

### Edit Samba Config
The configuration file for Samba is located at `/etc/samba/smb.conf`. To add the new directory as a share, we edit the file by running:
```
sudo vim /etc/samba/smb.conf
```
Append the following lines to `/etc/samba/smb.conf` :
```
[shared]
   comment = Samba on Ubuntu
   path = /root/shared
   read only = no
   browsable = yes
```

**What we Just added**
* `[shared]` - Name of the SMB endpoint.
* path - The directory of our share.
* `/root/shared` - Path of the folder we created.
* comment - A brief description of the share.
* read only - Permission to modify the contents of the share folder is only granted when the value of this directive is no.
* browsable - When set to yes, file managers such as Ubuntu’s default file manager will list this share under “Network” (it could also appear as browseable).

### Restart Service
Now that we have our new share configured, save it and restart Samba for it to take effect.
```
sudo service smbd restart
```

### Update Firewal Rules
Update the firewall rules to allow Samba traffic.
```
sudo ufw allow samba
```

### Setting up User Accounts
Username used here must belong to a system account. In this case it's `root`.
```
sudo smbpasswd -a root
```

---
# **On the Client Machine(s)**

### **Install & Prepare Mount Directory**

**1. Install cifs-utils**
```
sudo apt install cifs-utils
```

**2. Create folder to be mounted**

This will be the name of the mountpoint. In this case `/shared` is the mountpoint for our SMB share.
```
mkdir /shared
```

### **Mounting the directory**
Here we're setting the password '`123`' (which we have set earlier when Setting up User Accounts) for the user `root`. Local IP Address of the host in this example is `10.122.0.2` and we're connecting to its `/shared` endpoint through the `/shared` mountpoint.

**Method 1 : Mount temporarily**

Mount point will not persist after a reboot.
```
sudo mount -t cifs -o username=root,password=123,uid=$(id -u),gid=$(id -g),forceuid,forcegid,rw,file_mode=0777,dir_mode=0777 //10.122.0.2/shared /shared
```

**Method 2 : Mount permanently**

Edit `/etc/fstab`
```
sudo vim /etc/fstab
```

Append the following line.
```
//10.122.0.2/shared  /shared  cifs  username=root,password=123,uid=0,gid=0,forceuid,forcegid,rw,file_mode=0777,dir_mode=0777  0  0
```
(Re)mount all entries listed in `/etc/fstab`.
```
sudo mount -a
```

---
</br>

Or if you don't want to store passwords there, you can store them in a seperate file.

Create a new file.
```
vim ~/.smbcredentials
```
Add Credentials.
```
username=root
password=123
```
Protect the File if necessary.
```
chmod 600 ~/.smbcredentials
```
Append the following to `/etc/fstab`.
```
//10.122.0.2/shared  /shared  cifs  credentials=/root/.smbcredentials,uid=0,gid=0,forceuid,forcegid,rw,file_mode=0777,dir_mode=0777 0 0
```
(Re)mount all entries listed in `/etc/fstab`.
```
sudo mount -a
```
