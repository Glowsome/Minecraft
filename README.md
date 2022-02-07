## Description ##
A Minecraft server resource repository for running it on RockyLinux 8.x

## Installation ##
<details>
<summary>To setup your Minecraft server start out with a fresh RockyLinux 8.x server/vps/LXC -  i use a ProxMox (https://www.proxmox.com/en/proxmox-ve) LXC,
and will assumes that your server has:</summary>
  
    - (atleast) 4Gb of memory available.
    - default minecraft port 25565 (TCP/UDP) open in firewall
</details>

Install required packages: (some packages like zstd are work in progress for changing backup to use them in the future)
```
dnf install openssh-server wget tmux tar zstd java-17-openjdk-headless -y
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

Start the minecraft server to create `server.properties` and `eula.txt`, it will shutdown again afterwards (ignore errors/warnings on the output).
```
java -Xmx1024M -Xms512M -jar minecraft_server.1.18.1.jar nogui
```

Overwrite the content of the previously created `eula.txt` so the content reads `eula=true`.
```
echo "eula=true" > eula.txt
```

Place/create the also (in this repository) provided [backup script](https://raw.githubusercontent.com/Glowsome/Minecraft/main/backup.sh) and make it executable.
```
wget https://raw.githubusercontent.com/Glowsome/Minecraft/main/backup.sh
chmod +x backup.sh
```

Install a crontab job to run the backup job each hour (the `-s` switch specifies to the script to run on a schedule)
```
echo "0 * * * * /opt/minecraft/backup.sh -s" | crontab -
```

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

When the start was corect enable the minecraft service.
```
systemctl enable minecraft
```

Enjoy your Minecraft Server !
