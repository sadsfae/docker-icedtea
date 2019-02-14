# docker-icedtea
This pulls a docker firefox image with an older java-1.8.0-openjdk and icedtea-web to work around issues with older hardware OEM vendors out-of-band interfaces.

## Usage
Build image from Dockerfile
 ```bash
docker build -t firefox-java .
```
 # Running with Xephyr
 In order to run the container and redirect display to xserver we need to run via Xephyr
 * Run Xephyr with disabled shared memory (X extension MIT-SHM) and disabled X extension XTEST.
 * Set DISPLAY and share access to new unix socket /tmp/.X11-unix/.
 * Drop all capabilities with --cap-drop=ALL --security-opt=no-new-privileges to improve container security.
 * Pass the intended screen resolution via the `myscreen` variables

 ```bash
myscreen=1920x1080
export Displaynumber=1 &
Xephyr :$Displaynumber -extension MIT-SHM -extension XTEST -host-cursor -screen $myscreen &
docker run --rm -it \
    -e DISPLAY=:$Displaynumber \
    -v /tmp/.X11-unix/:/tmp/.X11-unix/:rw \
    --cap-drop=ALL \
    --security-opt=no-new-privileges \
    firefox-java

``` 
## Running with xhost
If Xephyr is not available xhost can be used for sharing host X display for single applications.

_Note: This nice short solution has the disadvantage of breaking container isolation. X security leaks like keylogging and remote host control can be abused by container applications._

* Allow access with xhost for current local user and create a similar container user.
* Share access to host X server with environment variable DISPLAY and X unix socket in /tmp/.X11-unix.
* Allow shared memory with --ipc=host to avoid RAM access failures and rendering glitches due to X extension MIT-SHM.
* Drop all capabilities with --cap-drop=ALL --security-opt=no-new-privileges to improve container security.
```bash
xhost + &
docker run --rm -it \
    -e DISPLAY=unix:0.0 \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    --ipc=host \
    --cap-drop=ALL \
    firefox-java
```
