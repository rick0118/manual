# devtool modify recipe
[Yocto documentation](https://docs.yoctoproject.org/ref-manual/system-requirements.html#required-packages-for-the-build-host)
## Clone and Build a Repo

Today we'll be extracting the [phosphor-state-manager](https://github.com/openbmc/phosphor-state-manager.git) repository using devtool and modifying it.

This assumes you have followed the first tutorial and are in the bitbake
environment, right after doing the ". setup" command.
* Pre-pre
  ```sh
  . setup <machine>  
  ```

1. Use devtool to extract source code

   ```
   devtool modify phosphor-state-manager
   ```

   The above command will output the directory the source code is contained in.

2. Modify a source file and add a cout

   ```
   vi workspace/sources/phosphor-state-manager/bmc_state_manager_main.cpp
   ```

   Your diff should look something like this:

   ```
   +#include <iostream>

   int main(int argc, char**)
   {
        bus.request_name(BMC_BUSNAME);

   +    std::cout<<"Hello World" <<std::endl;
   
   ```

3. Rebuild the flash image which will now include your change

   This will be a much faster build as bitbake will utilize all of the cache
   from your previous build, only building what is new.

   ```
   bitbake obmc-phosphor-image
   ```

   Load your new image into a QEMU session and boot it up.

4. Confirm your "Hello World" made it into the new image

   After you login to your QEMU session, verify the message is in the journal

   ```
   journalctl | grep "Hello World"
   ```

   You should see something like this:

   ```
   <date> romulus phosphor-bmc-state-manager[1089]: Hello World
   ```



In this section we're going to modify the same source file, but instead of fully
re-generating the flash image and booting QEMU again, we're going to just build
the required binary and copy it into the running QEMU session and launch it.

1. Modify your hello world

   ```
   vi workspace/sources/phosphor-state-manager/bmc_state_manager_main.cpp
   ```

   Change your cout to "Hello World Again"

2. Bitbake only the phosphor-state-manager repository

   Within bitbake, you can build only the repository (and it's dependencies)
   that you are interested in. In this case, we'll just rebuild the
   phosphor-state-manager repo to pick up your new hello world change.

   ```
   bitbake phosphor-state-manager
   ```

   Your new binary will be located at the following location relative to your
   bitbake directory:
   `./workspace/sources/phosphor-state-manager/oe-workdir/package/usr/bin/phosphor-bmc-state-manager`



4. scp this binary onto your QEMU instance

   If you used the default ports when starting QEMU then here is the scp command
   to run from your phosphor-state-manager directory. If you chose your own port
   then substitute that here for the 2222.

   ```
   scp -P 2222 ./workspace/sources/phosphor-state-manager/oe-workdir/package/usr/bin/phosphor-bmc-state-manager root@127.0.0.1:/usr/bin/
   ```

## Run the Application in QEMU

 ```
 qemu-system-arm -m 256 -M romulus-bmc -nographic -drive file=./obmc-phosphor-image-romulus.static.mtd,format=raw,if=mtd -net nic -net user,hostfwd=:10.32.49.218:2222-:22,hostfwd=:10.32.49.218:2443-:443,hostfwd=udp:127.0.0.1:2623-:623,hostname=qemu 
 ```
1. Stop the BMC state manager service

   ```
   systemctl stop xyz.openbmc_project.State.BMC.service
   ```

2. Run the application in your QEMU session:

   ```
   phosphor-bmc-state-manager
   ```


3. Start application via systemd service

   
   ```
   systemctl restart xyz.openbmc_project.State.BMC.service
   ```

  

   ```
   journalctl | tail
   ```

#### Use overlay to test New Applications (second)
   3. Create a safe file system for your application

   Now is time to load your Hello World application in to QEMU virtual hardware.
   The OpenBMC overrides the PATH variable to always look in /usr/local/bin/
   first so you could simply scp your application in to /usr/local/bin, run it
   and be done. That works fine for command line tests, but when you start
   launching your applications via systemd services you will hit problems since
   application paths are hardcoded in the service files. Let's start our journey
   doing what the professionals do and create an overlay file system. Overlays
   will save you time and grief. No more renaming and recovering original files,
   no more forgetting which files you messed with during debug, and of course,
   no more bricking the system. Log in to the QEMU instance and run these
   commands.

   ```
   mkdir -p /tmp/persist/usr
   mkdir -p /tmp/persist/work/usr
   mount -t overlay -o lowerdir=/usr,upperdir=/tmp/persist/usr,workdir=/tmp/persist/work/usr overlay /usr
   ```

   ## workflow
  O**n the development machine, build a base image on which we’ll test our work**

 ```
cd ~/src/openbmc/openbmc
. setup p10bmc
bitbake obmc-phosphor-image
```

**On the development machine, set up a repository to work on**
 ```
cd ~/src/openbmc
git clone https://github.com/openbmc/dbus-sensors.git
cd dbus-sensors
git checkout -b my-hacks
vi src/NVMeSensorMain.cpp
git commit -a -m "my hacks"
 ```

**On the development machine, configure and build our modified application**

```
devtool modify dbus-sensors
cd ~/src/openbmc/openbmc/build/p10bmc/workspace/sources/dbus-sensors
git remote add local ~/src/openbmc/dbus-sensors
git fetch local my-hacks && git merge FETCH_HEAD
devtool build dbus-sensors
```
[workflow](https://amboar.github.io/notes/2022/01/13/openbmc-development-workflow.html)