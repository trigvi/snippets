# Kill vscode processes on Linux Debian

* Create an empty bash script called *vscodekill.sh*, make it executable.
    ```
    touch vscodekill.sh
    chmod 700 vscodekill.sh
    ```

* Open the above script and insert the following contents:
    ```
    #!/bin/bash
    ps --sort -rss -eo pid,lstart,ruser,rss,command | grep vscodei | grep -v grep | awk '{print $1}' | xargs sudo kill -9
   ```

* Execute the bash script
    ```
    ./vscodekill.sh
    ```