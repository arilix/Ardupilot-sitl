# Tutorial Install SITL ArduPilot, MAVROS, dan ArduPilot Gazebo

Tutorial lengkap untuk menginstal **ArduPilot SITL (Software In The Loop)**, **MAVROS**, dan **ArduPilot Gazebo Plugin** pada Ubuntu 20.04/22.04 dengan ROS Noetic / ROS 2 Foxy / ROS 2 Humble.

## Prasyarat

- **OS**: Ubuntu 22.04 LTS (disarankan), atau Ubuntu 20.04 LTS 
- **RAM**: Minimal 8 GB
- **Disk**: Minimal 30 GB ruang kosong
- **ROS**: ROS Noetic (Ubuntu 20.04), ROS 2 Foxy (Ubuntu 20.04), atau ROS 2 Humble (Ubuntu 22.04)

## Komponen yang Diinstal

| Komponen | Deskripsi |
|---|---|
| **ArduPilot SITL** | Simulator untuk menjalankan firmware ArduPilot tanpa hardware fisik |
| **MAVROS** | ROS package untuk komunikasi MAVLink antara ROS dan autopilot |
| **ArduPilot Gazebo Plugin** | Plugin Gazebo untuk simulasi kendaraan ArduPilot di lingkungan 3D |

## Quick Start (Instalasi Otomatis)

Jalankan script instalasi otomatis:

```bash
# Clone repository ini
git clone https://github.com/arilix/Ardupilot-sitl.git
cd ardupilot-sitl-mavros-gazebo-tutorial

# Beri izin eksekusi
chmod +x install.sh

# Jalankan installer
./install.sh
```

## Panduan Instalasi Manual

### 1. Update Sistem

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl wget python3-pip python3-dev build-essential
```

### 2. Install ArduPilot SITL

```bash
# Clone ArduPilot
cd ~
git clone --recurse-submodules https://github.com/ArduPilot/ardupilot.git
cd ardupilot

# Install dependensi
Tools/environment_install/install-prereqs-ubuntu.sh -y

# Reload PATH
. ~/.profile

# Build SITL untuk Copter
cd ~/ardupilot
./waf configure --board sitl
./waf copter
```

**Test SITL:**

```bash
cd ~/ardupilot/ArduCopter
sim_vehicle.py -v ArduCopter --console --map
```

### 3a. Install ROS Noetic (Ubuntu 20.04)

```bash
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
sudo apt update
sudo apt install -y ros-noetic-desktop-full
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
sudo apt install -y python3-rosdep python3-rosinstall python3-rosinstall-generator python3-wstool
sudo rosdep init
rosdep update
```

### 3b. Install ROS 2 Foxy (Ubuntu 20.04)

> **Catatan:** ROS 2 Foxy sudah End-of-Life (EOL) sejak Juni 2023 namun masih banyak digunakan. Pertimbangkan untuk menggunakan ROS 2 Humble jika memungkinkan.

```bash
# Set locale
sudo apt update && sudo apt install -y locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

# Setup sources
sudo apt install -y software-properties-common
sudo add-apt-repository universe
sudo apt update && sudo apt install -y curl
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

# Install ROS 2 Foxy
sudo apt update
sudo apt install -y ros-foxy-desktop

# Setup environment
echo "source /opt/ros/foxy/setup.bash" >> ~/.bashrc
source ~/.bashrc

# Install colcon build tools
sudo apt install -y python3-colcon-common-extensions python3-rosdep
sudo rosdep init
rosdep update
```

### 4a. Install MAVROS (ROS Noetic)

```bash
# Install MAVROS dan MAVROS extras
sudo apt install -y ros-noetic-mavros ros-noetic-mavros-extras

# Install GeographicLib datasets (WAJIB)
sudo /opt/ros/noetic/lib/mavros/install_geographiclib_datasets.sh
```

**Test MAVROS (Noetic):**

```bash
# Terminal 1: Jalankan SITL
cd ~/ardupilot/ArduCopter
sim_vehicle.py -v ArduCopter --console --map

# Terminal 2: Jalankan MAVROS
roslaunch mavros apm.launch fcu_url:=udp://127.0.0.1:14551@14555
```

### 4b. Install MAVROS (ROS 2 Foxy)

```bash
# Install MAVROS2 dan MAVROS2 extras
sudo apt install -y ros-foxy-mavros ros-foxy-mavros-extras

# Install GeographicLib datasets (WAJIB)
sudo /opt/ros/foxy/lib/mavros/install_geographiclib_datasets.sh
```

**Test MAVROS (Foxy):**

```bash
# Terminal 1: Jalankan SITL
cd ~/ardupilot/ArduCopter
sim_vehicle.py -v ArduCopter --console --map

# Terminal 2: Jalankan MAVROS2
ros2 launch mavros apm.launch fcu_url:=udp://127.0.0.1:14551@14555
```

### 5. Install Gazebo dan ArduPilot Gazebo Plugin

```bash
# Install Gazebo 11
sudo apt install -y gazebo11 libgazebo11-dev

# Clone ArduPilot Gazebo Plugin
cd ~
git clone https://github.com/khancyr/ardupilot_gazebo.git
cd ardupilot_gazebo

# Build plugin
mkdir build && cd build
cmake ..
make -j$(nproc)
sudo make install

# Setup environment variables
echo 'source /usr/share/gazebo/setup.sh' >> ~/.bashrc
echo 'export GAZEBO_MODEL_PATH=~/ardupilot_gazebo/models:$GAZEBO_MODEL_PATH' >> ~/.bashrc
echo 'export GAZEBO_RESOURCE_PATH=~/ardupilot_gazebo/worlds:$GAZEBO_RESOURCE_PATH' >> ~/.bashrc
echo 'export GAZEBO_PLUGIN_PATH=/usr/lib/x86_64-linux-gnu/gazebo-11/plugins:$GAZEBO_PLUGIN_PATH' >> ~/.bashrc
source ~/.bashrc
```

**Test Gazebo dengan ArduPilot:**

```bash
# Terminal 1: Jalankan Gazebo
gazebo --verbose ~/ardupilot_gazebo/worlds/iris_ardupilot.world

# Terminal 2: Jalankan SITL
cd ~/ardupilot/ArduCopter
sim_vehicle.py -v ArduCopter -f gazebo-iris --console --map
```

## Test Integrasi Penuh (SITL + MAVROS + Gazebo)

```bash
# Terminal 1: Jalankan Gazebo
gazebo --verbose ~/ardupilot_gazebo/worlds/iris_ardupilot.world

# Terminal 2: Jalankan SITL
cd ~/ardupilot/ArduCopter
sim_vehicle.py -v ArduCopter -f gazebo-iris --console --map

# Terminal 3: Jalankan MAVROS
roslaunch mavros apm.launch fcu_url:=udp://127.0.0.1:14551@14555

# Terminal 4: Cek koneksi
rostopic echo /mavros/state
```

## Troubleshooting

| Masalah | Solusi |
|---|---|
| `sim_vehicle.py` tidak ditemukan | Jalankan `. ~/.profile` atau restart terminal |
| MAVROS tidak konek ke SITL | Pastikan port FCU URL sesuai dengan output SITL |
| Gazebo crash saat startup | Cek GPU driver: `glxinfo \| grep "OpenGL"` |
| GeographicLib error | Jalankan ulang script `install_geographiclib_datasets.sh` |
| Build ArduPilot gagal | Pastikan semua submodule ter-clone: `git submodule update --init --recursive` |

## Referensi

- [ArduPilot SITL Docs](https://ardupilot.org/dev/docs/sitl-simulator-software-in-the-loop.html)
- [MAVROS Wiki](http://wiki.ros.org/mavros)
- [ArduPilot Gazebo Plugin](https://github.com/khancyr/ardupilot_gazebo)
- [ROS Noetic Installation](http://wiki.ros.org/noetic/Installation/Ubuntu)
- [ROS 2 Foxy Installation](https://docs.ros.org/en/foxy/Installation/Ubuntu-Install-Debians.html)
