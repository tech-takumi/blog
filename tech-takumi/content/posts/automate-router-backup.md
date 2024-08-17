---
title: How to Automate Your Router's Backup and Keep Your Sanity Intact
date: 2024-08-17T23:47:55+02:00
---
## Automating My Fritz!Box Backup: The Quest for Router Zen

Every techie knows that there’s nothing quite like the feeling of having your entire setup just... work. But let’s be honest—this nirvana is often more myth than reality. I’ve always envisioned automating the configuration of all my devices, making it reproducible, and yes, even bulletproof. The router, of course, had to be part of this vision.

### Why Bother with Router Backups?

Let’s face it—routers are the unsung heroes of our homes. They work tirelessly, beaming Wi-Fi signals through walls, surviving firmware updates, and sometimes even a curious toddler’s “experiments.” Yet, when it comes to backup, they get no love. I decided it was time to change that.

Now, before you ask—yes, I had this romantic notion of achieving a point-in-time recovery of my [FRITZ!Box 7530 AX](https://en.avm.de/products/fritzbox/fritzbox-7530-ax/) configurations. It’s less about fearing hardware failure (though that’s still a thing) and more about maintaining a snapshot of my configurations. After all, how interchangeable are these backups between devices? (Spoiler: I still don’t know, but Linux’s `file` command seems optimistic.)

```bash
[rodrigo@ws1:~/Downloads]$ file FRITZ.Box_7530_AX_256.07.80i_10.08.24_1840.export 
FRITZ.Box_7530_AX_256.07.80i_10.08.24_1840.export: FRITZ!Box configuration backup of 7530, firmware version 256.07.80, oem avme, language en, Sat Aug 10 18:40:03 2024
```

### The Automation Journey: A Road Paved with Obstacles

**Surviving Firmware Updates:**

It’s a real conundrum. While I can manually back up my configurations, the sticky issue is secrets. Apparently, the backup includes this crucial bit, and decrypting it with my password isn’t as straightforward as it should be. But hey, there’s a tool for that! (Thanks, PeterPawn’s [decoder](https://github.com/PeterPawn/decoder)).

**Automation Inspiration:**

Enter [Solence's blog](https://solence.de/2017/03/28/scripting-an-automated-backup-for-an-avm-fritzbox-router/). A goldmine, albeit one that doesn’t completely solve the modern-day conundrum of Multi-Factor Authentication (MFA). And no, disabling MFA isn’t an option—I’m not that reckless.

Instead, I had the brilliant idea of using the [1Password CLI](https://developer.1password.com/docs/cli/secrets-scripts/) to get MFA One-Time Passwords (OTP). This way, I could automate the entire process, including those pesky OTPs. Setting up MFA for a user specifically designed for backups? Check.

**Packaging It All Up:**

The next logical step was to package this into a Docker image, because who doesn’t love a good containerized solution? I dove into [Nix](https://nix.dev/tutorials/nixos/building-and-running-docker-images.html), spent some quality time with [Flakes](https://nixos.wiki/wiki/Flakes), and battled an existential crisis when the build process complained that my new package was already defined somewhere in Nix stores. Seriously, Nix, make up your mind!

With some help from ChatGPT, I managed to structure the project properly, integrating `op` and `curl` into the Docker image. A few iterations later—boom! The first image was born. It wasn’t pretty, but it worked (well, sort of).

### The Final Hurdle: MFA Shenanigans and the Push Service Surprise

**MFA Madness:**

No automation project is complete without a final, face-palm-worthy challenge. For me, it was the realization that my FRITZ!Box has a push service that can send backups of the configuration. That’s right—I was a hairsbreadth away from victory when I discovered I could have simply received the backup via email. But where’s the fun in that? Besides it only sends information when firmware is about to be updated or someone triggers a factory reset. I want more peace of mind than that!

So, I moved into capturing all the traffic between my browser and the router (good old `dev tool` in Chrome), parsing through the noise until I found the relevant calls to `data.lua` and `two_factor.lua`. And just like that, I could finally automate the backup download without needing to physically click the router’s buttons. Victory never tasted so sweet.

**Publishing the Docker Image:**

Of course, I had to publish my creation. Initially, I planned to put everything on GitHub and host the image on Docker Hub. But after squashing some embarrassing history from my repo (I may or may not have pushed a token to Gitea), I decided to skip GitHub and push directly to Docker Hub.

Here’s where you can find it: [rcouto/fritzbox-backup-docker](https://hub.docker.com/repository/docker/rcouto/fritzbox-backup-docker). Enjoy!
### Scheduling and Storing Backups

The final piece of the puzzle was figuring out where to store the backups and how to schedule the runs. The answer? My trusty [Lincstation N1 NAS](https://www.lincplustech.com/products/lincstation-n1-network-attached-storage), where backups could live in peace, gzip-compressed and taking up a mere 30KB a day. 

```bash
#!/bin/bash

BACKUP_DIR=/mnt/user/backup/fritzbox
TOKEN=/root/.1p_token

docker run --rm -v ${TOKEN}:/etc/token:ro -v ${BACKUP_DIR}:/mnt/backup:rw rcouto/fritzbox-backup-docker
gzip -9 ${BACKUP_DIR}/*.export
```

Scheduled via the “User Scripts” plugin on Unraid, this script now runs daily, backing up my Fritz!Box configurations like clockwork.

### The Final Frontier: Vulnerabilities and Backup Rotation

No good deed goes unpunished. My image was flagged for vulnerabilities in Docker Hubs, but that’s a battle for another day. For now, I’m content with the knowledge that my backups are automated, my configurations are safe, and my router will live to see another firmware update—without breaking a sweat.

In conclusion, the journey to automate my Fritz!Box backups was riddled with challenges, from MFA to Docker quirks, but in the end, it was all worth it. Who knows? Maybe one day, I’ll automate my entire life. But for now, I’ll settle for router zen.

---

**References:**

- [Scripting an Automated Backup for an AVM FRITZ!Box Router](https://solence.de/2017/03/28/scripting-an-automated-backup-for-an-avm-fritzbox-router/)
- [1Password CLI Documentation](https://developer.1password.com/docs/cli/secrets-scripts/)
- [Nix Docker Tutorial](https://nix.dev/tutorials/nixos/building-and-running-docker-images.html)