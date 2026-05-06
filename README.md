A minimal O-RAN + srsRAN setup

        +---------------------+
        |     Near-RT RIC     |   ← xApps (control, ML, slicing)
        |  (OSC / FlexRIC)    |
        +----------+----------+
                   |
                   | E2 interface
                   |
        +----------v----------+
        |     srsRAN gNB      |  ← CU/DU (software base station)
        +----------+----------+
                   |
                   | RF
                   |
            +------v------+
            |  USRP X310  |
            +-------------+
                   |
                Over-the-air
                   |
                 UE (phone / COTS UE)
                

Step 1 — USRP X310 bring-up (foundation)


Step 2 - Install srsRAN (5G stack)
[srsRAN 5G Stask](https://docs.srsran.com/projects/project/en/latest/user_manuals/source/installation.html)

2.1 Build Tools and Dependencies: (Ubuntu 22.04)
`sudo apt-get install cmake make gcc g++ pkg-config libfftw3-dev libmbedtls-dev libsctp-dev libyaml-cpp-dev libgtest-dev`

2.2 Main srsRAN repo: Clone and Build

`git clone https://github.com/srsRAN/srsRAN_Project.git`

``` bash
cd srsRAN_Project
mkdir build
cd build
cmake ../
make -j $(nproc)
make test -j $(nproc)

sudo make install
```

Step 3 - Add 5G Core (Open5GS)
[Open5GS - Github](https://github.com/open5gs/open5gs)
[Open5GS Website](https://open5gs.org/open5gs/docs/)
3.1 - Installation

```bash
sudo add-apt-repository ppa:open5gs/latest
sudo apt update
sudo apt install open5gs
```

3.2 Configure Open5GS
Out of the box, the default configurations see all of the Open5GS components fully configured for use on a single computer. They are set to communicate with each other using the local loopback address space (127.0.0.X). The default addresses for each of the bind interfaces for these components and functions are as follows:

```bash
MongoDB   = 127.0.0.1 (subscriber data) - http://localhost:9999

MME-s1ap  = 127.0.0.2 :36412 for S1-MME
MME-gtpc  = 127.0.0.2 :2123 for S11
MME-frDi  = 127.0.0.2 :3868 for S6a

SGWC-gtpc = 127.0.0.3 :2123 for S11
SGWC-pfcp = 127.0.0.3 :8805 for Sxa

SMF-gtpc  = 127.0.0.4 :2123 for S5c
SMF-gtpu  = 127.0.0.4 :2152 for N4u (Sxu)
SMF-pfcp  = 127.0.0.4 :8805 for N4 (Sxb)
SMF-frDi  = 127.0.0.4 :3868 for Gx auth
SMF-sbi   = 127.0.0.4 :7777 for 5G SBI (N7,N10,N11)

AMF-ngap  = 127.0.0.5 :38412 for N2
AMF-sbi   = 127.0.0.5 :7777 for 5G SBI (N8,N12,N11)

SGWU-pfcp = 127.0.0.6 :8805 for Sxa
SGWU-gtpu = 127.0.0.6 :2152 for S1-U, S5u

UPF-pfcp  = 127.0.0.7 :8805 for N4 (Sxb)
UPF-gtpu  = 127.0.0.7 :2152 for S5u, N3, N4u (Sxu)

HSS-frDi  = 127.0.0.8 :3868 for S6a, Cx

PCRF-frDi = 127.0.0.9 :3868 for Gx

NRF-sbi   = 127.0.0.10:7777 for 5G SBI
SCP-sbi   = 127.0.0.200:7777 for 5G SBI
SEPP-sbi  = 127.0.0.250:7777 for 5G SBI
SEPP-n32  = 127.0.0.251:7777 for 5G N32
SEPP-n32f = 127.0.0.252:7777 for 5G N32-f
AUSF-sbi  = 127.0.0.11:7777 for 5G SBI
UDM-sbi   = 127.0.0.12:7777 for 5G SBI
PCF-sbi   = 127.0.0.13:7777 for 5G SBI
NSSF-sbi  = 127.0.0.14:7777 for 5G SBI
BSF-sbi   = 127.0.0.15:7777 for 5G SBI
UDR-sbi   = 127.0.0.20:7777 for 5G SBI
```

3.2.1 - Setup a 4G/5G NSA Core
Modify `/etc/open5gs/mme.yaml` to set the S1AP IP address, PLMN ID, and TAC  
Modify `/etc/open5gs/sgwu.yaml` to set the GTP-U IP address.  

After changing config files, please restart Open5GS daemons.
```bash
$ sudo systemctl restart open5gs-mmed
$ sudo systemctl restart open5gs-sgwud
```

3.2.2 - Setup a 5G Core
Modify `/etc/open5gs/nrf.yaml` to set the Serving PLMN ID.  
Modify `/etc/open5gs/amf.yaml` to set the NGAP IP address, PLMN ID, TAC and NSSAI.  
Modify `/etc/open5gs/upf.yaml` to set the GTP-U address  
After changing config files, please restart Open5GS daemons.  
```bash
$ sudo systemctl restart open5gs-nrfd
$ sudo systemctl restart open5gs-amfd
$ sudo systemctl restart open5gs-upfd
```

3.2.3 - Configure logging
The Open5GS components log to `/var/log/open5gs/*.log` and to stderr by default.

3.2.4 - Refister Subscriber Information
Connect to `http://localhost:9999` and loging with **admin** account.
```
Username: admin
Password: 1423
```

3.2.5- Turn on eNB/gNB and UE

First, connect your eNB/gNB to the Open5GS core:  
- Make sure the PLMN and TAC of the eNB/gNB matches the settings in your MME/AMF  
- Connect your eNB/gNB to the IP of your server via the standard S1AP/NGAP SCTP port 36412/38412 (for MME/AMF)  
- Your eNB/gNB should report a successful S1/NG connection - congrats, your core is fully working!  
- You can see actual traffic through wireshark – [srsenb.pcapng].  
- You can view the log at `/var/log/open5gs/*.log`, eg:  

``` bash
### Watch the live MME log
tail -f /var/log/open5gs/mme.log
```

Next, try to attach a UE to the basestation: 
- Insert your SIM card to the UE  
- Set the UE’s APN to match the APN you configured in the Open5GS WebUI  
- Toggle the UE in and out of flight mode  
- If it doesn’t automatically connect, try manually searching for a network  
- If the PLMN set on the SIM card does not match the PLMN being used by the radio, you will need to ensure ‘data roaming’ on the UE is switched on  

3.2.6 - Starting and Stopping Open5GS

```bash
$ sudo systemctl stop open5gs-mmed
$ sudo systemctl stop open5gs-sgwcd
$ sudo systemctl stop open5gs-smfd
$ sudo systemctl stop open5gs-amfd
$ sudo systemctl stop open5gs-sgwud
$ sudo systemctl stop open5gs-upfd
$ sudo systemctl stop open5gs-hssd
$ sudo systemctl stop open5gs-pcrfd
$ sudo systemctl stop open5gs-nrfd
$ sudo systemctl stop open5gs-scpd
$ sudo systemctl stop open5gs-seppd
$ sudo systemctl stop open5gs-ausfd
$ sudo systemctl stop open5gs-udmd
$ sudo systemctl stop open5gs-pcfd
$ sudo systemctl stop open5gs-nssfd
$ sudo systemctl stop open5gs-bsfd
$ sudo systemctl stop open5gs-udrd
$ sudo systemctl stop open5gs-webui
```

```bash
$ sudo systemctl restart open5gs-mmed
$ sudo systemctl restart open5gs-sgwcd
$ sudo systemctl restart open5gs-smfd
$ sudo systemctl restart open5gs-amfd
$ sudo systemctl restart open5gs-sgwud
$ sudo systemctl restart open5gs-upfd
$ sudo systemctl restart open5gs-hssd
$ sudo systemctl restart open5gs-pcrfd
$ sudo systemctl restart open5gs-nrfd
$ sudo systemctl restart open5gs-scpd
$ sudo systemctl restart open5gs-seppd
$ sudo systemctl restart open5gs-ausfd
$ sudo systemctl restart open5gs-udmd
$ sudo systemctl restart open5gs-pcfd
$ sudo systemctl restart open5gs-nssfd
$ sudo systemctl restart open5gs-bsfd
$ sudo systemctl restart open5gs-udrd
$ sudo systemctl restart open5gs-webui
```