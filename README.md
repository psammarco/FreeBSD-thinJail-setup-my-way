# FreeBSD-thinJail-setup-my-way
The idea behind this setup is to separate the thinjail's base structure and system configuration into different paths,  where system image template goes into /jails/structure/thinjail, jail specific RW skeleton folder to /jails/thinjails,  fstab in /jails/fstab, and ultimately base + skeleton are mounted in /jails/mnt.
