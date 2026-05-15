# tutorial for libreboot install
This is the tutorial I found to install libreboot on the thinkpad t480, and is the basis for these instructions:
- https://youtu.be/8c-ODWXg6F8
- https://github.com/radleylewis/t480_libreboot/

I really liked his idea to use docker, but I wanted to run the setup scripts myself in the docker container (instead of using a dockerfile, like this author does). I'm pretty sure these setup steps work for any supported hardware!

## useful links
- https://libreboot.org/docs/install/ - guides for all supported motherboards
- https://libreboot.org/download.html#https - mirrors for ROM downloads

## dependencies
- this will all be containerized with docker, for easy dependency management without messin up your host environment.

## instructions

### pre-reqs
MAC Address is needed! the libreboot docs say a random MAC address is fine, but I wanted to use the same MAC address that is already on my laptop. to do that, you just needed to run:
```
$ ifconfig
```
the NIC will be listed as `enp1s0` or `eth0`, or something similar to that. the MAC address for that interface is listed under `ether`, and will look something like 11:22:33:aa:bb:cc

### create new ROM with lbmk
```shell
# 1. start debian container
docker run -it --name libreboot_env debian:bookworm

# 2. regular docker set up
apt-get update && apt-get install -y \
git \
wget \
curl \
xz-utils \
ca-certificates \
build-essential \
python3 \
sudo \
gnupg \
software-properties-common \
&& rm -rf /var/lib/apt/lists/*

# 3. clone lbmk library in /opt/
cd /opt
git clone https://codeberg.org/libreboot/lbmk.git
cd lbmk

# 4. install dependencies for lbmk
apt-get update
yes | ./mk dependencies debian

# 5. get your desired ROM from a mirror for your specified motherboard. The list of mirrors are linked above. 
# For example, I have a t440p, and I want to use the latest stable libreboot image (26.01rev1). Using the MIT mirror, that would be https://mirrors.mit.edu/libreboot/stable/26.01rev1/roms/libreboot-26.01rev1_t440plibremrc_12mb.tar.xz

curl -O https://mirrors.mit.edu/libreboot/stable/26.01rev1/roms/libreboot-26.01rev1_t440plibremrc_12mb.tar.xz

# 6. a non-root user is needed for lbmk at this point
useradd -m builder
chown -R builder:builder /opt/lbmk
su - bulder

# 7. configure git with dummy values
git config --global user.name "John Doe"
git config --global user.email "johndoe@example.com"

# 8. run ./mk, replace the tar file with whichever you downloaded for your hardware
# IMPORTANT: MAC address will be set to random if you don't use the "setmac" argument, but I like to use the MAC address already assigned to my NIC, which will be fed manually here (instructions are in pre-reqs).
./mk inject libreboot-26.01rev1_t440plibremrc_12mb.tar.xz setmac 11:22:33:aa:bb:cc

# 9. copy the edited tar.xz file from the docker container onto your host file system
# exit the docker container with ctrl+P, ctrl+Q, then run the following on the host to copy the file (of course, your filename may be different)
docker container cp libreboot_env:/opt/lbmk/libreboot-26.01rev1_t440plibremrc_12mb.tar.xz .

# you are done with creating the ROM!
```
### next steps
gotta flash your ROM! TBD
