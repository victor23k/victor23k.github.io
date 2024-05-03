+++
title = 'WSL port forwarding'
date = 2024-01-14T07:07:07+01:00
draft = true
tags = ["wsl"]
+++

# Access server in WSL through Windows network


1. Open a hotspot in Windows


2. Add a proxy in Windows to access WSL deployed apps through Windows localhost

   In powershell execute the following command:

   ```cmd
   netsh interface portproxy add v4tov4 listenport=4000 listenaddress=0.0.0.0 connectport=8080 connectaddress=192.168.1.4
   ```

   - listenport: the port external devices will connect to
   - connectport: the port where our application server is configured to listen inside WSL
   - connectaddress: IP address given to WSL. To get this IP address get `inet` of `eht0` through `ifconfig`


3. Open port (listenport) in Windows firewall


4. Get the access url to use in devices connected to Windows hotspot ipconfig and get ip address. Example:

   ```url
   http://192.168.137.1:4000/test
   ```
