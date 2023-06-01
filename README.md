## Description ##
A Minecraft server resource repository for running it on RockyLinux 8.x/9.x

## Installation ##
<details>
<summary>To setup your Minecraft server start out with a fresh RockyLinux 8.x/9.x server/vps/LXC -  i use a ProxMox (https://www.proxmox.com/en/proxmox-ve) LXC,
and will assumes that your server has:</summary>
  
    - (atleast) 4Gb of memory available.
    - Default minecraft port 25565 (TCP/UDP) open in firewall
    - SSH acccess to the box.
</details>

Install required packages: (some packages like zstd are work in progress for changing backup to use them in the future)
```
dnf install wget tmux tar zstd java-17-openjdk-headless -y
```

Create folderstructure for Minecraft.
```
mkdir -p /opt/minecraft/backups/{hourly,daily,weekly,monthly}
```

Create minecraft user.
```
useradd -r -m -U -d /opt/minecraft -s /bin/bash minecraft
```

Set ownership to minecraft user on created directory-structure.
```
chown -R minecraft.minecraft /opt/minecraft
```

Change user to `minecraft`.
```
su - minecraft
```

As minecraft user download the server-jar from [Minecraft.net](https://www.minecraft.net/en-us/download/server) When writing this the version was `1.18.1`.
```
wget -O minecraft_server.1.18.1.jar https://launcher.mojang.com/v1/objects/125e5adf40c659fd3bce3e66e67a16bb49ecc1b9/server.jar
```

Create a symbolic link to the current version of the server-jar you downloaded (this will make it easy to just replace the symbolic link to a new/future version when updating)
```
ln -s minecraft_server.1.18.1.jar server.jar
```

Start the minecraft server to create `server.properties` and `eula.txt`, it will shutdown again afterwards (ignore errors/warnings on the output).
```
java -Xmx1024M -Xms512M -jar server.jar nogui
```

Overwrite the content of the previously created `eula.txt` so the content reads `eula=true`.
```
echo "eula=true" > eula.txt
```

Place/create either the (in this repository) provided tar [backup script](https://raw.githubusercontent.com/Glowsome/Minecraft/main/backup.sh) or the zst with better compression [backup script](https://raw.githubusercontent.com/Glowsome/Minecraft/main/backup-zst.sh) and make it executable.

```
wget https://raw.githubusercontent.com/Glowsome/Minecraft/main/backup.sh
chmod +x backup.sh
```
OR
```
wget https://raw.githubusercontent.com/Glowsome/Minecraft/main/backup-zst.sh
chmod +x backup-zst.sh
```

Install a crontab job to run the backup job each hour (the `-s` switch specifies to the script to run on a schedule)
```
echo "0 * * * * /opt/minecraft/backup.sh -s" | crontab -
```
OR
```
echo "0 * * * * /opt/minecraft/backup-zst.sh -s" | crontab -
```

<details>
<summary>The default backup-script settings are for a hourly/daily/weekly rotation, if you like to change this, then edit the script:</summary>
  - Find the line in the script `# Increments of time on which to back up`.
  - Adapt the parameters to your own needs for HOURLY,DAILY,WEEKLY,MONTHLY by setting true/false.
</details>

Switch back to user root by pressing `CRTL-D` 

Place/create the also (in this repository) provided [systemd file](https://raw.githubusercontent.com/Glowsome/Minecraft/main/minecraft.service) in `/etc/systemd/system/minecraft.service`
```
wget -O /etc/systemd/system/minecraft.service https://raw.githubusercontent.com/Glowsome/Minecraft/main/minecraft.service
```

Reload Systemd daemon.
```
systemctl daemon-reload
```

Start your Minecraft server via the systemd service.
```
systemctl start minecraft
```

Verify a successfull start of the server.
```
systemctl status minecraft
```
The output should look something like this:
```
● minecraft.service - Minecraft Server
   Loaded: loaded (/etc/systemd/system/minecraft.service; enabled; vendor preset: disabled)
  Drop-In: /run/systemd/system/minecraft.service.d
           └─zzz-lxc-service.conf
   Active: active (running) since Tue 2022-02-08 01:29:58 CET; 6s ago
  Process: 304926 ExecStop=/bin/sleep 2 (code=exited, status=0/SUCCESS)
  Process: 304925 ExecStop=/usr/bin/tmux send-keys -t minecraft:0.0 say SERVER SHUTTING DOWN. Saving map... C-m save-all C-m stop C-m (code
=exited, status=0/SUCCESS)
  Process: 304935 ExecStart=/usr/bin/tmux new -s minecraft -d /usr/bin/java -Xmx3096M -Xms2048M -XX:+UseG1GC -jar server.jar --nogui (code=
exited, status=0/SUCCESS)
 Main PID: 304937 (tmux: server)
    Tasks: 25 (limit: 927876)
   Memory: 330.1M
   CGroup: /system.slice/minecraft.service
           ├─304937 /usr/bin/tmux new -s minecraft -d /usr/bin/java -Xmx3096M -Xms2048M -XX:+UseG1GC -jar server.jar --nogui
           └─304938 /usr/bin/java -Xmx3096M -Xms2048M -XX:+UseG1GC -jar server.jar --nogui

Feb 08 01:29:58 minecraft.myservername.tld systemd[1]: Starting Minecraft Server...
Feb 08 01:29:58 minecraft.myservername.tld systemd[1]: Started Minecraft Server.
```
When the start was corect enable the minecraft service.
```
systemctl enable minecraft
```

## Updating ##
<summary>
To update the server to a newer version of minecraft perform the follwing procedure:
</summary>

Stop the minecraft server (when running / as root)
```
systemctl stop minecraft
```

Change to the `minecraft` user.
```
su minecraft
```

Download the new minecraft server file ( example , the exact link should be otained from mojang itself).
```
wget -O minecraft_server.1.19.jar https://launcher.mojang.com/v1/objects/e00c4052dac1d59a1188b2aa9d5a87113aaf1122/server.jar
```

Remove old symbolic link to the old version.
```
rm -f server.jar
```

Recreate symbolic link to the corect version of the server jarfile ( again the example is based on a specific version, so adapt where needed)
```
ln -s minecraft_server.1.19.jar server.jar
```

Exit minecraft user (assuming you logged in originally as root).
```
exit
```

Start minecraft Server.
```
systemctl start minecraft.service
```

Enjoy your Minecraft Server !
