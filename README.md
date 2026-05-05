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

```bash
sudo add-apt-repository ppa:open5gs/latest
sudo apt update
sudo apt install open5gs
```
