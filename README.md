# ArduPilot + Gazebo SITL Setup Guide
## Ubuntu 24.04

### Step 1 — Clone ArduPilot
```bash
cd ~
git clone --recurse-submodules https://github.com/ArduPilot/ardupilot.git
cd ardupilot
Tools/environment_install/install-prereqs-ubuntu.sh -y
. ~/.profile
```

### Step 2 — Install Gazebo Harmonic
```bash
sudo curl https://packages.osrfoundation.org/gazebo.gpg \
  --output /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] \
  http://packages.osrfoundation.org/gazebo/ubuntu-stable \
  $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/gazebo-stable.list

sudo apt-get update
sudo apt-get install gz-harmonic -y
```

### Step 3 — Install Plugin Dependencies
```bash
sudo apt-get install rapidjson-dev libopencv-dev \
  libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev -y
```

### Step 4 — Build ArduPilot-Gazebo Plugin
```bash
cd ~
git clone https://github.com/ArduPilot/ardupilot_gazebo.git
cd ardupilot_gazebo
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=RelWithDebInfo
make -j4
```

### Step 5 — Add Paths to .bashrc
```bash
echo 'export GZ_SIM_SYSTEM_PLUGIN_PATH=\
$HOME/ardupilot_gazebo/build:\
$GZ_SIM_SYSTEM_PLUGIN_PATH' >> ~/.bashrc

echo 'export GZ_SIM_RESOURCE_PATH=\
$HOME/ardupilot_gazebo/models:\
$HOME/ardupilot_gazebo/worlds:\
$GZ_SIM_RESOURCE_PATH' >> ~/.bashrc

source ~/.bashrc
```

### Step 6 — Run Simulation

**Terminal 1 — Gazebo:**
```bash
gz sim -v4 -r iris_runway.sdf
```

**Terminal 2 — SITL:**
```bash
cd ~/ardupilot
sim_vehicle.py -v ArduCopter -f gazebo-iris \
  --model JSON --map --console \
  --out=udp:127.0.0.1:14550
```

**Mission Planner:**
- Protocol: UDP
- Port: 14550
- Click CONNECT

### Common Issues & Fixes

| Error | Fix |
|-------|-----|
| `RapidJSON not found` | `sudo apt-get install rapidjson-dev` |
| `OpenCV not found` | `sudo apt-get install libopencv-dev` |
| `gstreamer not found` | `sudo apt-get install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev` |
| `No JSON sensor message` | Start Gazebo before SITL |
| `No Heartbeat` | Use UDP port 14550 not TCP 5760 |
# ardupilot-gazebo-setup1
