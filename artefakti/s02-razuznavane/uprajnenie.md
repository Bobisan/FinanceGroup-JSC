
##Setup

The goal of this exercise was to to do basic reconnaissance through nmap and fluff, determine which mitre attack navigator techinques their use falls into and to scan a host running known vulnerabilities, on the virtual machines that were setup in the last lab.

For this purpose I had to setup NAT on the gateway vm, so that the server vm could access the internet. After that I deployed the docker enviornemnt with the cve-2025-3248 vulnerability from vulhub. After that I installed nmap and ffuf on the gateway vm, so that I can use them to find a vulnerability (while pretending that I don't know the cve number from vulhub).

##Vulnerability Search Proccess

When I scanned for services and their versions, running on server, using this nmap command:

`nmap -p1-10000 -sV 192.168.100.3`

which gave me this output:

`Nmap scan report for 192.168.100.3
Host is up (0.00047s latency).
Not shown: 9999 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
7860/tcp open  http    Uvicorn`

I realised that the server has a http service running on port 7680, so I decided to curl that page, which gave me this output:

`<!doctype html>
<html lang="en">
  <head>
    <base href="/" />
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link rel="icon" href="./assets/favicon-new-BaG6bPWd.ico" />
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
    <link
      href="https://fonts.googleapis.com/css2?family=Chivo:ital,wght@0,100..900;1,100..900&family=Inter:ital,opsz,wght@0,14..32,100..900;1,14..32,100..900&family=JetBrains+Mono:ital,wght@0,100..800;1,100..800&display=swap"
      rel="stylesheet"
    />
    <title>Langflow</title>
    <script type="module" crossorigin src="./assets/index-hNpyfBay.js"></script>
    <link rel="stylesheet" crossorigin href="./assets/index-CwAwSIfd.css">
  </head>
  <body id="body" class="dark" style="width: 100%; height: 100%">
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div style="width: 100vw; height: 100vh" id="root"></div>
  </body>
</html>`

(the output also shows that I didn't set up the container correctly, but that's beside the point)

From where I searched for "Langflow" and found out it's some local ai platform, so I searched for "Langflow CVE", which showed me results for CVE-2025-3248 (the very same one from the start), which can be found on langflow instances prior to version 1.3.0.

##Detected Vulnerability

Langflow versions prior to 1.3.0 are susceptible to code injection in the /api/v1/validate/code endpoint. A remote and unauthenticated attacker can send crafted HTTP requests to execute arbitrary code.
