# Qwen + Ollama LAN Setup for Neovim / OpenCode

## Goal

Run the Qwen coding model on an HP Victus laptop with an RTX 4050, then use that model from another computer over the local network.

```text
Other PC
  │
  │ LAN / Wi-Fi
  ▼
HP Victus
  │
  ├── Ollama
  │
  └── Qwen3 8B
       │
       ▼
    RTX 4050


The second PC does not need to download the Qwen model. It connects to Ollama running on the Victus.

1. Install Ollama on the Victus

Install Ollama for Windows.

Verify the installation from CMD or PowerShell:

ollama --version

2. Download Qwen

Example:

ollama pull qwen3:8b


Check installed models:

ollama list


You should see:

qwen3:8b


Test the model:

ollama run qwen3:8b


For example:

Write a Python function to check if a number is prime.


If Qwen responds, the local setup is working.

Exit with:

/bye

3. Configure Ollama for LAN access

By default, Ollama may only accept connections from the local computer.

On the Victus, open CMD and run:

setx OLLAMA_HOST "0.0.0.0:11434"


Important: after running setx, completely quit Ollama from the Windows system tray and start it again.

Verify the environment variable:

echo %OLLAMA_HOST%


Expected:

0.0.0.0:11434

4. Find the Victus IP address

On the Victus:

ipconfig


Find the IPv4 address for the active Wi-Fi/Ethernet connection.

Example:

IPv4 Address. . . . . . : 192.168.1.50


Your actual IP will probably be different.

This address is what the other PC will use.

5. Allow Ollama through Windows Firewall

Open PowerShell as Administrator on the Victus:

New-NetFirewallRule `
  -DisplayName "Ollama LAN" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 11434 `
  -Action Allow `
  -Profile Private


This allows computers on your Windows Private network to reach Ollama's port.

6. Test Ollama from the second PC

Make sure both computers are connected to the same LAN/Wi-Fi.

On the second PC, replace the IP below with the Victus's actual IP:

curl http://192.168.1.50:11434/api/tags


If successful, you should receive JSON containing the installed models.

For example, you should see something containing:

qwen3:8b


This confirms:

Second PC → Network → Victus → Ollama → Qwen

7. Test an actual Qwen request remotely

From the second PC:

curl http://192.168.1.50:11434/api/generate -d "{\"model\":\"qwen3:8b\",\"prompt\":\"Write a hello world program in Python\",\"stream\":false}"


If Qwen returns an answer, the remote setup is working.

OpenCode Setup

The important concept is that OpenCode must connect to the Victus's IP, not localhost.

Wrong
http://localhost:11434


This means:

OpenCode PC → itself

Correct
http://192.168.1.50:11434


This means:

OpenCode PC
     │
     ▼
192.168.1.50
     │
     ▼
Victus
     │
     ▼
Ollama
     │
     ▼
Qwen


Use the actual IP address of the Victus.

Neovim Setup

The final setup can be used in two ways.

OpenCode / AI Agent
Neovim
   │
   ▼
OpenCode
   │
   │ LAN
   ▼
Victus
   │
   ▼
Ollama
   │
   ▼
Qwen3 8B


This allows Qwen to perform coding-agent tasks such as:

Explain code
Modify files
Refactor code
Find bugs
Write tests
Generate code
Inline Code Completion

For a more GitHub Copilot-like experience, use a Neovim completion plugin that supports an Ollama/OpenAI-compatible backend.

The architecture becomes:

Neovim
 ├── Inline completion
 │       │
 │       ▼
 │    Ollama/Qwen
 │       │
 │       ▼
 │     Victus
 │
 └── OpenCode
         │
         ▼
      Ollama/Qwen
         │
         ▼
       Victus

Troubleshooting
curl cannot connect

First check that Ollama is running on the Victus:

ollama list


Then check the Victus IP:

ipconfig


From the second PC:

curl http://VICTUS_IP:11434/api/tags


Example:

curl http://192.168.1.50:11434/api/tags

localhost:11434 works on Victus but IP doesn't work

Check:

OLLAMA_HOST was set correctly.
Ollama was completely restarted after running setx.
Windows Firewall allows TCP port 11434.
Both computers are on the same LAN.
You are using the Victus's current IPv4 address.

Check:

echo %OLLAMA_HOST%


Expected:

0.0.0.0:11434

Victus IP changes

Your router may assign the Victus a different IP after reconnecting to Wi-Fi.

Check again with:

ipconfig


If the IP changes frequently, consider giving the Victus a DHCP reservation in your router.

Useful Commands
Check Ollama version
ollama --version

List models
ollama list

Run Qwen
ollama run qwen3:8b

Check running models
ollama ps

Download Qwen
ollama pull qwen3:8b

Set LAN address
setx OLLAMA_HOST "0.0.0.0:11434"

Check LAN address setting
echo %OLLAMA_HOST%

Find Victus IP
ipconfig

Test Ollama from another PC
curl http://VICTUS_IP:11434/api/tags


Example:

curl http://192.168.1.50:11434/api/tags

Security Note

This setup is intended for a trusted home/local network.

Do not expose port 11434 directly to the public internet.

If remote access from outside your home network is needed later, use a private VPN such as Tailscale rather than port-forwarding Ollama directly.

Final Setup
                 LOCAL NETWORK
                       │
                       │
        ┌──────────────▼──────────────┐
        │          HP VICTUS           │
        │                              │
        │       RTX 4050 6GB           │
        │             │                │
        │             ▼                │
        │          Ollama               │
        │             │                │
        │             ▼                │
        │        Qwen3 8B               │
        │             │                │
        │       :11434 API              │
        └──────────────┬───────────────┘
                       │
                       │ Wi-Fi / Ethernet
                       │
        ┌──────────────▼───────────────┐
        │          OTHER PC             │
        │                              │
        │          Neovim               │
        │             │                │
        │         OpenCode              │
        │             │                │
        │             ▼                │
        │   http://VICTUS_IP:11434     │
        └──────────────────────────────┘

Key Command

The main command that enables the LAN connection is:

setx OLLAMA_HOST "0.0.0.0:11434"


After setting it:

Completely quit Ollama.
Start Ollama again.
Run ipconfig to find the Victus IP.
From the other PC, test with:
curl http://VICTUS_IP:11434/api/tags
