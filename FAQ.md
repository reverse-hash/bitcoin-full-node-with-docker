# Frequently Asked Questions

## Do you plan to add support for Lightning Network (LND)?

The short answer is no.

Adding LND increases considerably the complexity of the project and its maintenance. Keep in mind that it requires periodic backups of the channels, watchtowers (your own or from third parties) in case your node goes down, loses internet connection or suffers some kind of attack, and in the meantime someone closes a channel fraudulently... 

So for the time being, it's not on the roadmap.

## Why debian-slim and not alpine?

The main problem is that Tor, Bitcoind and Electrs are dynamically linked to glibc. Alpine is based on musl so it's necessary to install additional packages which cause the image size to increase considerably.

More info here: https://wiki.alpinelinux.org/wiki/Running_glibc_programs
