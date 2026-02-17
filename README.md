# Isaac Lab Arena: Complete Beginner's Guide

A step-by-step tutorial for setting up Isaac Lab Arena using Docker containers, designed for developers new to Docker.

---

## Table of Contents

1. [What is Docker? (Explained Simply)](#1-what-is-docker-explained-simply)
2. [Prerequisites Checklist](#2-prerequisites-checklist)
3. [Step 1: Install Docker](#3-step-1-install-docker)
4. [Step 2: Install NVIDIA Container Toolkit](#4-step-2-install-nvidia-container-toolkit)
5. [Step 3: Clone Isaac Lab Arena](#5-step-3-clone-isaac-lab-arena)
6. [Step 4: Launch the Docker Container](#6-step-4-launch-the-docker-container)
7. [Step 5: Verify Installation](#7-step-5-verify-installation)
8. [Step 6: Run Your First Arena Environment](#8-step-6-run-your-first-arena-environment)
9. [Step 7: Create Your Own Environment](#9-step-7-create-your-own-environment)
10. [Docker Cheat Sheet](#10-docker-cheat-sheet)
11. [Troubleshooting](#11-troubleshooting)
12. [Reference Links](#12-reference-links)

---

## 1. What is Docker? (Explained Simply)

### The Problem Docker Solves

Imagine you want to run Isaac Lab Arena. It requires:
- Isaac Sim 5.1.0 (specific version)
- Isaac Lab 2.3.0 (specific version)
- NVIDIA drivers configured correctly
- Python 3.11 with dozens of packages
- Specific system libraries

Installing all of this manually is error-prone. One wrong version breaks everything.

### Docker's Solution

Docker creates a "container" — a self-contained box that has everything pre-installed and configured. Think of it like:

```
┌─────────────────────────────────────────────────────────────┐
│  YOUR COMPUTER (Host)                                       │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │  DOCKER CONTAINER (Isolated Environment)            │   │
│   │                                                     │   │
│   │   ✓ Isaac Sim 5.1.0        (pre-installed)         │   │
│   │   ✓ Isaac Lab 2.3.0        (pre-installed)         │   │
│   │   ✓ Isaac Lab Arena        (pre-installed)         │   │
│   │   ✓ Python 3.11 + packages (pre-installed)         │   │
│   │   ✓ All dependencies       (pre-configured)        │   │
│   │                                                     │   │
│   │   You just write your code here!                    │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                             │
│   Your GPU ←── NVIDIA Container Toolkit ──→ Container       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Key Docker Concepts

| Term | What It Means | Analogy |
|------|---------------|---------|
| **Image** | A template/blueprint with everything installed | A recipe |
| **Container** | A running instance of an image | A dish made from the recipe |
| **Volume** | Shared folder between your computer and container | A shared USB drive |
| **Host** | Your actual computer | Your kitchen |


NVIDIA provides a pre-built Docker image with Isaac Sim. Arena's Docker setup builds on top of it, ensuring everything works together correctly.

---



## 3. Step 1: Install Docker

### Check if Docker is Already Installed

```bash
docker --version
```

If you see a version number (e.g., `Docker version 26.0.0`), skip to Step 2.

### Install Docker (Ubuntu 22.04)

Run these commands one at a time:

```bash
# 1. Download Docker's install script
curl -fsSL https://get.docker.com -o get-docker.sh

# 2. Run the install script
sudo sh get-docker.sh

# 3. Start Docker service
sudo systemctl start docker
sudo systemctl enable docker
```

### Configure Docker to Run Without `sudo`

By default, Docker requires `sudo`. This fixes that:

```bash
# 1. Create docker group (may already exist)
sudo groupadd docker

# 2. Add your user to the docker group
sudo usermod -aG docker $USER

# 3. Apply the group change (or log out and back in)
newgrp docker
```

### Verify Docker Installation

```bash
# Test without sudo
docker run hello-world
```

You should see:
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

### Install Docker Compose

```bash
# Check if already installed
docker compose version

# If not installed:
sudo apt install docker-compose-plugin
```

---

## 4. Step 2: Install NVIDIA Container Toolkit (OPTIONAL IF YOU DON'T HAVE A DOCKER CONTAINER WITH ISAACSIM AND ISAACLAB INSTALLED YET)

This toolkit allows Docker containers to access your NVIDIA GPU.

### Check if Already Installed

```bash
nvidia-ctk --version
```

If you see a version, skip to Step 3.

### Install NVIDIA Container Toolkit (Ubuntu)

```bash
# 1. Add NVIDIA's package repository
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# 2. Update package list
sudo apt update

# 3. Install the toolkit
sudo apt install -y nvidia-container-toolkit

# 4. Configure Docker to use NVIDIA runtime
sudo nvidia-ctk runtime configure --runtime=docker

# 5. Restart Docker to apply changes
sudo systemctl restart docker
```

### Verify GPU Access in Docker

```bash
docker run --rm --gpus all nvidia/cuda:12.2.0-base-ubuntu22.04 nvidia-smi
```

You should see your GPU information (same as running `nvidia-smi` on your host).

If this fails with "could not select device driver", restart your computer and try again.

---

## 5. Step 3: Clone Isaac Lab Arena

### Clone the Repository

```bash
# Navigate to where you want the project
cd ~

# Clone Isaac Lab Arena
git clone https://github.com/isaac-sim/IsaacLab-Arena.git

# Enter the directory
cd IsaacLab-Arena
```

### Initialize Submodules (CRITICAL!)

This downloads Isaac Lab and other dependencies:

```bash
git config submodule.submodules/IsaacLab.url https://github.com/isaac-sim/IsaacLab.git
git config submodule.submodules/Isaac-GR00T.url https://github.com/NVIDIA/Isaac-GR00T.git

git submodule update --init --recursive
```

This may take 5-10 minutes. You'll see it downloading Isaac Lab into `submodules/IsaacLab/`.

```
pip install -e .
```

Test
```
python -c "import isaaclab_arena; print('Success!')"
```
Sould print "Success!"

Check available commands
```
python -m isaaclab_arena.examples.example_environments.cli --help
```

---

## 6. Step 4: Launch the Docker Container

### First Launch (Downloads ~20GB)

From the `IsaacLab-Arena` directory:

```bash
./docker/run_docker.sh
```

**What happens:**
1. Docker downloads the NVIDIA Isaac Sim base image (~15GB)
2. Builds the Arena layer on top (~5GB)
3. Starts the container
4. Drops you into a shell inside the container

The first run takes 15-30 minutes depending on your internet speed. Subsequent runs are instant.

### You're Now Inside the Container!

Your terminal prompt changes to something like:
```
root@abc123def456:/workspace/isaaclab-arena#
```

You are now:
- Inside an isolated Ubuntu environment
- With Isaac Sim, Isaac Lab, and Arena pre-installed
- With access to your GPU
- In the `/workspace/isaaclab-arena` directory

### Understanding the Workspace

```
INSIDE CONTAINER:
/workspace/
├── isaaclab-arena/        ← Arena code (mounted from your host)
│   ├── isaaclab_arena/
│   └── submodules/IsaacLab/
└── (Isaac Sim is installed elsewhere in the container)
```

**Key insight:** The `isaaclab-arena` folder is "mounted" from your host computer. This means:
- Files you edit on your host appear inside the container
- Files created in the container appear on your host
- You can use VS Code on your host while running code in the container

---

## 7. Step 5: Verify Installation

### Inside the container, run these checks:

```bash
# 1. Check Python (should show Isaac Sim's Python)
python --version
# Expected: Python 3.11.x

# 2. Check Isaac Lab is importable
python -c "import isaaclab; print('Isaac Lab OK')"

# 3. Check Arena is importable
python -c "import isaaclab_arena; print('Arena OK')"

# 4. Check GPU is accessible
python -c "import torch; print(f'CUDA available: {torch.cuda.is_available()}')"
# Expected: CUDA available: True
```

### Run Arena's Built-in Tests (Optional)

```bash
# Quick test (no cameras)
pytest -sv -m "not with_cameras" isaaclab_arena/tests/

# Full test (with cameras, slower)
pytest -sv -m with_cameras isaaclab_arena/tests/
```

---

## 8. Step 6: Run Your First Arena Environment

### List Available Example Environments

```bash
ls isaaclab_arena/examples/
```

You should see files like:
- `franka_lift_env.py` — Franka robot lifting objects
- `gr1_open_microwave_env.py` — GR1 humanoid opening microwave

### Run the Franka Lift Example

```bash
python isaaclab_arena/scripts/run_env.py \
    --enable_cameras \
    franka_lift
```

**What happens:**
- Isaac Sim launches
- A Franka robot appears in a scene
- Objects appear for the robot to interact with
- The simulation runs

Press `Ctrl+C` to stop.

### Run with Different Options

```bash
# Headless mode (no GUI, faster for training)
python isaaclab_arena/scripts/run_env.py \
    --headless \
    franka_lift

# More parallel environments
python isaaclab_arena/scripts/run_env.py \
    --num_envs 16 \
    --headless \
    franka_lift
```

---

## 9. Step 7: Create Your Own Environment

Now let's create a simple custom environment using **only pre-built assets**.

### What's Available in the Registry?

Arena provides these pre-built assets:

| Category | Asset Name | Description |
|----------|------------|-------------|
| **Robots** | `franka` | Franka Panda 7-DOF arm |
| **Robots** | `gr1`, `gr1_pink` | GR1 humanoid robot |
| **Robots** | `g1` | G1 humanoid robot |
| **Scenes** | `kitchen` | Full kitchen environment |
| **Scenes** | `table` | Simple table |
| **Objects** | `tomato_soup_can` | YCB dataset soup can |
| **Objects** | `microwave` | Openable microwave |
| **Tasks** | `PickAndPlaceTask` | Pick up and place objects |
| **Tasks** | `OpenDoorTask` | Open doors/cabinets |
| **Teleop** | `keyboard` | Keyboard control |

### Create Your Environment File

Still inside the container, create a new file:

```bash
# Create a directory for your environments
mkdir -p /workspace/isaaclab-arena/my_environments

# Create the file (using nano, vim, or edit from host)
nano /workspace/isaaclab-arena/my_environments/soup_pickup_env.py
```

Paste this code:

```python
"""
My First Arena Environment
--------------------------
Franka robot picks up a tomato soup can from a table.
Uses ONLY pre-built Arena assets.
"""
import argparse

from isaaclab_arena.assets.asset_registry import asset_registry
from isaaclab_arena.assets.device_registry import device_registry
from isaaclab_arena.environments.isaaclab_arena_environment import IsaacLabArenaEnvironment
from isaaclab_arena.scene.scene import Scene
from isaaclab_arena.tasks.pick_and_place_task import PickAndPlaceTask
from isaaclab_arena.assets.object_reference import ObjectReference
from isaaclab_arena.assets.object_base import ObjectType
from isaaclab_arena.utils.pose import Pose
from isaaclab_arena.examples.base_env import ExampleEnvironmentBase


class SoupPickupEnvironment(ExampleEnvironmentBase):
    """
    Simple pick-and-place environment.
    
    Components (all from Arena's pre-built registry):
    - Robot: Franka Panda arm
    - Scene: Kitchen background
    - Object: Tomato soup can (YCB dataset)
    - Task: Pick and place
    """
    
    name: str = "soup_pickup"
    
    def get_env(self, args_cli: argparse.Namespace) -> IsaacLabArenaEnvironment:
        """Build and return the environment."""
        
        # ─────────────────────────────────────────────────────────────
        # 1. GET ROBOT FROM REGISTRY
        # ─────────────────────────────────────────────────────────────
        # The robot (called "embodiment" in Arena) is loaded by name.
        # Available: "franka", "gr1", "gr1_pink", "g1"
        
        embodiment = self.asset_registry.get_asset_by_name("franka")(
            enable_cameras=args_cli.enable_cameras
        )
        
        # ─────────────────────────────────────────────────────────────
        # 2. GET SCENE/BACKGROUND FROM REGISTRY
        # ─────────────────────────────────────────────────────────────
        # The background scene provides the environment.
        # Available: "kitchen", "table", etc.
        
        background = self.asset_registry.get_asset_by_name("kitchen")()
        
        # ─────────────────────────────────────────────────────────────
        # 3. GET OBJECT FROM REGISTRY
        # ─────────────────────────────────────────────────────────────
        # Objects the robot will interact with.
        # Available: "tomato_soup_can", "microwave", etc.
        
        soup_can = self.asset_registry.get_asset_by_name("tomato_soup_can")()
        
        # Set where the object starts
        soup_can.set_initial_pose(
            Pose(
                position_xyz=(0.5, 0.0, 0.8),  # x=forward, y=left, z=up
                rotation_wxyz=(1.0, 0.0, 0.0, 0.0)  # w,x,y,z quaternion (upright)
            )
        )
        
        # ─────────────────────────────────────────────────────────────
        # 4. DEFINE DESTINATION (where to place the object)
        # ─────────────────────────────────────────────────────────────
        # ObjectReference points to a location in the scene
        
        destination = ObjectReference(
            name="destination_location",
            prim_path="{ENV_REGEX_NS}/kitchen/Counter_01",  # A counter in the kitchen
            parent_asset=background,
            object_type=ObjectType.RIGID,
        )
        
        # ─────────────────────────────────────────────────────────────
        # 5. GET TELEOP DEVICE FROM REGISTRY
        # ─────────────────────────────────────────────────────────────
        # How you control the robot interactively.
        # Available: "keyboard", "spacemouse", "avp_handtracking"
        
        teleop_device = self.device_registry.get_device_by_name("keyboard")()
        
        # ─────────────────────────────────────────────────────────────
        # 6. COMPOSE THE SCENE
        # ─────────────────────────────────────────────────────────────
        # Combine background and objects into a scene
        
        scene = Scene(assets=[background, soup_can])
        
        # ─────────────────────────────────────────────────────────────
        # 7. CREATE THE TASK
        # ─────────────────────────────────────────────────────────────
        # The task defines what the robot should do.
        # Available: PickAndPlaceTask, OpenDoorTask, etc.
        
        task = PickAndPlaceTask(
            pick_object=soup_can,
            place_location=destination,
            background=background
        )
        
        # ─────────────────────────────────────────────────────────────
        # 8. RETURN THE COMPOSED ENVIRONMENT
        # ─────────────────────────────────────────────────────────────
        
        return IsaacLabArenaEnvironment(
            name=self.name,
            embodiment=embodiment,
            scene=scene,
            task=task,
            teleop_device=teleop_device,
        )


# This allows Arena's run_env.py to find your environment
def register_environment():
    return SoupPickupEnvironment()
```

### Register Your Environment

Create an `__init__.py` to make it a Python package:

```bash
echo "from .soup_pickup_env import SoupPickupEnvironment" > /workspace/isaaclab-arena/my_environments/__init__.py
```

### Run Your Custom Environment

```bash
# From inside the container
cd /workspace/isaaclab-arena

# Run your environment
python -c "
from my_environments.soup_pickup_env import SoupPickupEnvironment
import argparse

# Create minimal args
args = argparse.Namespace(
    enable_cameras=True,
    teleop_device='keyboard',
    headless=False
)

# Create environment
env_def = SoupPickupEnvironment()
arena_env = env_def.get_env(args)

print(f'Environment created: {arena_env.name}')
print('Success!')
"
```

---

## 10. Docker Cheat Sheet

### Essential Commands

| Command | What It Does |
|---------|--------------|
| `./docker/run_docker.sh` | Start container and enter it |
| `exit` | Leave container (keeps it running) |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers (including stopped) |
| `docker stop <name>` | Stop a container |
| `docker start <name>` | Start a stopped container |
| `docker exec -it <name> bash` | Enter a running container |
| `docker images` | List downloaded images |
| `docker system prune` | Clean up unused data |

### Working with the Arena Container

```bash
# Check if Arena container is running
docker ps | grep isaac

# Enter an already-running Arena container
docker exec -it <container_name> bash

# Stop the Arena container
docker stop <container_name>

# Remove the container (image remains)
docker rm <container_name>

# Remove everything and start fresh
docker system prune -a  # WARNING: Deletes all images!
```

### Understanding Mounted Volumes

When you run `./docker/run_docker.sh`, certain folders are "mounted" (shared):

```
YOUR COMPUTER (Host)              INSIDE CONTAINER
─────────────────────────────────────────────────────────────
~/IsaacLab-Arena/          ←→    /workspace/isaaclab-arena/
  (your code edits here)           (appears here too)

Changes sync instantly in both directions!
```

This means you can:
1. Edit code in VS Code on your host
2. Run it inside the container
3. See results immediately

### Copying Files In/Out

```bash
# Copy from host to container
docker cp myfile.py <container_name>:/workspace/isaaclab-arena/

# Copy from container to host
docker cp <container_name>:/workspace/isaaclab-arena/logs ./logs
```

---

## 11. Troubleshooting

### "permission denied" when running docker

```bash
# Add yourself to docker group
sudo usermod -aG docker $USER

# Log out and back in, or run:
newgrp docker
```

### "could not select device driver" or GPU not found

```bash
# 1. Verify NVIDIA driver works on host
nvidia-smi

# 2. Restart Docker
sudo systemctl restart docker

# 3. Test GPU access
docker run --rm --gpus all nvidia/cuda:12.2.0-base-ubuntu22.04 nvidia-smi

# 4. If still failing, reinstall NVIDIA Container Toolkit
sudo apt remove nvidia-container-toolkit
sudo apt install nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

### Container starts but Isaac Sim crashes

Common causes:
- Insufficient VRAM (need 8GB+)
- Driver version too old (need 535+)

```bash
# Check VRAM usage
nvidia-smi

# Update driver if needed
sudo apt install nvidia-driver-545
sudo reboot
```

### "No module named isaaclab_arena"

Make sure you initialized submodules:
```bash
cd ~/IsaacLab-Arena
git submodule update --init --recursive
```

### Changes to code not appearing

If you edit files on your host but don't see changes in the container:
1. Make sure you're editing files in the mounted directory
2. The mount is `/workspace/isaaclab-arena/` inside container
3. Only certain directories are mounted — check `docker-compose.yaml`

### Running out of disk space

Docker images are large (~20GB for Arena). Free space:
```bash
# Remove unused containers and images
docker system prune

# Nuclear option: remove everything
docker system prune -a
```

---

## 12. Reference Links

### Official Documentation

| Resource | URL |
|----------|-----|
| **Isaac Lab Arena Docs** | https://isaac-sim.github.io/IsaacLab-Arena/main/index.html |
| **Arena Installation** | https://isaac-sim.github.io/IsaacLab-Arena/main/pages/quickstart/installation.html |
| **Arena GitHub** | https://github.com/isaac-sim/IsaacLab-Arena |
| **Isaac Lab Docs** | https://isaac-sim.github.io/IsaacLab/main/index.html |
| **Isaac Lab Docker Guide** | https://isaac-sim.github.io/IsaacLab/main/source/deployment/docker.html |
| **Isaac Sim Docs** | https://docs.isaacsim.omniverse.nvidia.com/latest/index.html |

### Docker & NVIDIA

| Resource | URL |
|----------|-----|
| **Docker Install (Ubuntu)** | https://docs.docker.com/engine/install/ubuntu/ |
| **Docker Post-Install** | https://docs.docker.com/engine/install/linux-postinstall/ |
| **NVIDIA Container Toolkit** | https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html |

### Arena Example Workflows

| Workflow | Description | URL |
|----------|-------------|-----|
| **Franka Lift (RL)** | Train Franka to lift objects | https://isaac-sim.github.io/IsaacLab-Arena/main/pages/example_workflows/reinforcement_learning/index.html |
| **GR1 Microwave** | Open microwave door | https://isaac-sim.github.io/IsaacLab-Arena/main/pages/example_workflows/static_manipulation/index.html |
| **G1 Box Transport** | Humanoid carries box | https://isaac-sim.github.io/IsaacLab-Arena/main/pages/example_workflows/locomanipulation/index.html |

### Concepts Documentation

| Concept | URL |
|---------|-----|
| **Environment Design** | https://isaac-sim.github.io/IsaacLab-Arena/main/pages/concepts/concept_environment_design.html |
| **Embodiment Design** | https://isaac-sim.github.io/IsaacLab-Arena/main/pages/concepts/concept_embodiment_design.html |
| **Tasks Design** | https://isaac-sim.github.io/IsaacLab-Arena/main/pages/concepts/concept_tasks_design.html |
| **Scene Design** | https://isaac-sim.github.io/IsaacLab-Arena/main/pages/concepts/concept_scene_design.html |
| **Assets Design** | https://isaac-sim.github.io/IsaacLab-Arena/main/pages/concepts/concept_assets_design.html |

---

## Quick Start Summary

```bash
# 1. Install Docker
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER && newgrp docker

# 2. Install NVIDIA Container Toolkit
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt update && sudo apt install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker

# 3. Clone Arena
git clone https://github.com/isaac-sim/IsaacLab-Arena.git
cd IsaacLab-Arena
git submodule update --init --recursive

# 4. Launch container
./docker/run_docker.sh

# 5. (Inside container) Run example
python isaaclab_arena/scripts/run_env.py --enable_cameras franka_lift
```

---

*Document created: February 2025*
*Isaac Lab Arena version: 0.1.1*
*Isaac Lab version: 2.3.0*
*Isaac Sim version: 5.1.0*
