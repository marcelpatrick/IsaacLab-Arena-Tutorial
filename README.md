# IsaacLab-Arena-Tutorial


# Complete Walkthrough: Creating an Isaac Lab Arena Project

A step-by-step guide to building an external project with Isaac Lab Arena integration for easy task and object switching — perfect for your burger assembly automation project.

---

## 📚 Reference Documentation & Resources

### Official Documentation
| Resource | Link | Description |
|----------|------|-------------|
| **Isaac Lab Arena Docs** | [Documentation Home](https://isaac-sim.github.io/IsaacLab-Arena/main/index.html) | Main Arena documentation |
| **Arena GitHub Repo** | [GitHub](https://github.com/isaac-sim/IsaacLab-Arena) | Source code and examples |
| **First Arena Environment** | [Tutorial](https://isaac-sim.github.io/IsaacLab-Arena/main/pages/quickstart/first_arena_env.html) | Quick start guide |
| **Assets Design** | [Concepts](https://isaac-sim.github.io/IsaacLab-Arena/release/0.1.1/pages/concepts/concept_assets_design.html) | How to create/register assets |
| **Tasks Design** | [Concepts](https://isaac-sim.github.io/IsaacLab-Arena/main/pages/concepts/concept_tasks_design.html) | How to define tasks |
| **Environment Setup** | [Example Workflow](https://isaac-sim.github.io/IsaacLab-Arena/main/pages/example_workflows/static_manipulation/step_1_environment_setup.html) | GR1 microwave example |

### Isaac Lab Core Documentation
| Resource | Link | Description |
|----------|------|-------------|
| **Isaac Lab Quickstart** | [Quickstart](https://isaac-sim.github.io/IsaacLab/main/source/setup/quickstart.html) | Getting started with Isaac Lab |
| **Template Generator** | [Create Project](https://isaac-sim.github.io/IsaacLab/main/source/overview/own-project/template.html) | External project creation |
| **Project Structure** | [Structure Guide](https://isaac-sim.github.io/IsaacLab/main/source/overview/own-project/project_structure.html) | Understanding 4-layer hierarchy |
| **Manager-Based Env** | [Tutorial](https://isaac-sim.github.io/IsaacLab/main/source/tutorials/03_envs/create_manager_base_env.html) | Creating manager-based environments |
| **Environment Registration** | [Tutorial](https://isaac-sim.github.io/IsaacLab/main/source/tutorials/03_envs/register_rl_env_gym.html) | Registering with Gymnasium |

### Video Tutorials (LycheeAI)
| Resource | Link | Description |
|----------|------|-------------|
| **LycheeAI Hub** | [Website](https://lycheeai-hub.com/) | Comprehensive Isaac Lab learning platform |
| **Template Generator Tutorial** | [Guide](https://lycheeai-hub.com/isaac-lab/build-your-own-isaac-lab-external-project-template-generator) | Step-by-step external project creation |
| **Isaac Lab Basics Series** | [Tutorials](https://lycheeai-hub.com/isaac-lab) | Beginner to intermediate tutorials |
| **SO-ARM101 Project** | [Project Page](https://lycheeai-hub.com/project-so-arm101-x-isaac-sim-x-isaac-lab-tutorial-series) | Real-world robot project example |

### NVIDIA Official Resources
| Resource | Link | Description |
|----------|------|-------------|
| **Isaac Lab Arena Blog** | [NVIDIA Blog](https://developer.nvidia.com/blog/simplify-generalist-robot-policy-evaluation-in-simulation-with-nvidia-isaac-lab-arena) | Official introduction and workflow |
| **Isaac Lab Arena Product Page** | [NVIDIA Developer](https://developer.nvidia.com/isaac/lab-arena) | Product overview |
| **Isaac Lab Product Page** | [NVIDIA Developer](https://developer.nvidia.com/isaac/lab) | Main Isaac Lab page |
| **LeRobot Integration** | [HuggingFace Blog](https://huggingface.co/blog/nvidia/generalist-robotpolicy-eval-isaaclab-arena-lerobot) | Arena + LeRobot evaluation guide |

---

## Step 1: Prerequisites & Installation

### 1.1 System Requirements

Before starting, ensure you have:
- **GPU**: NVIDIA RTX 2070+ (16GB+ VRAM recommended)
- **RAM**: 32GB+ recommended
- **OS**: Ubuntu 22.04 (recommended) or Windows 11
- **CUDA**: 12.x+
- **Python**: 3.11

### 1.2 Install Isaac Lab + Arena

```bash
# =============================================================================
# STEP 1.2.1: Create conda environment
# =============================================================================
conda create -y -n burger_arena python=3.11
conda activate burger_arena
conda install -y -c conda-forge ffmpeg=7.1.1  # Required for video recording

# =============================================================================
# STEP 1.2.2: Install Isaac Sim 5.1.0
# Reference: https://isaac-sim.github.io/IsaacLab/main/source/setup/installation/pip_installation.html
# =============================================================================
pip install "isaacsim[all,extscache]==5.1.0" --extra-index-url https://pypi.nvidia.com

# IMPORTANT: Accept NVIDIA EULA (required for first run)
export ACCEPT_EULA=Y
export PRIVACY_CONSENT=Y

# Verify Isaac Sim installation
isaacsim --help

# =============================================================================
# STEP 1.2.3: Install Isaac Lab 2.3.0
# Reference: https://isaac-sim.github.io/IsaacLab/main/source/setup/installation/binaries_installation.html
# =============================================================================
git clone https://github.com/isaac-sim/IsaacLab.git
cd IsaacLab
git checkout v2.3.0

# Install Isaac Lab and all RL libraries
./isaaclab.sh -i  # or on Windows: isaaclab.bat -i

# Verify installation
./isaaclab.sh -p scripts/tutorials/00_sim/create_empty.py

cd ..  # Return to parent directory

# =============================================================================
# STEP 1.2.4: Install Isaac Lab Arena
# Reference: https://github.com/isaac-sim/IsaacLab-Arena
# =============================================================================
git clone https://github.com/isaac-sim/IsaacLab-Arena.git
cd IsaacLab-Arena
git checkout release/0.1.1  # Use stable release

# Install Arena in editable mode
pip install -e .

# Install additional dependencies
pip install onnxruntime==1.23.2 numpy==1.26.0

cd ..  # Return to parent directory
```

> 📖 **Reference**: [Isaac Lab Arena Installation](https://isaac-sim.github.io/IsaacLab-Arena/main/pages/quickstart/installation.html)

---

## Step 2: Generate Template Project

### 2.1 Run the Template Generator

```bash
# Navigate to Isaac Lab directory
cd IsaacLab

# Run the template generator wizard
./isaaclab.sh --new   # Linux
# OR
isaaclab.bat --new    # Windows
```

### 2.2 Wizard Options to Select

When the wizard prompts you, choose these options for Arena compatibility:

```
? Select project type:
  > External  ← SELECT THIS (required for Arena)

? Select workflow:
  > manager   ← SELECT THIS (Arena uses manager-based workflow)

? Select RL libraries (space to select, enter to confirm):
  [x] skrl    ← Recommended for manipulation
  [ ] rsl_rl
  [ ] rl_games
  [ ] sb3

? Select algorithm:
  > ppo       ← Good default for manipulation

? Enter project name:
  > burger_assembly_arena

? Enter project path:
  > ../burger_assembly_arena  ← Creates outside Isaac Lab folder
```

### 2.3 What Gets Generated

```
burger_assembly_arena/           # PROJECT ROOT
├── scripts/
│   ├── skrl/
│   │   ├── train.py            # Training script
│   │   └── play.py             # Evaluation script
│   ├── list_envs.py            # List registered environments
│   ├── random_agent.py         # Test with random actions
│   └── zero_agent.py           # Test with zero actions
├── source/
│   └── burger_assembly_arena/   # EXTENSION
│       ├── config/
│       │   └── extension.toml   # Extension metadata
│       ├── pyproject.toml       # Package metadata
│       ├── setup.py             # Installation script
│       └── burger_assembly_arena/  # MODULE
│           ├── __init__.py
│           └── tasks/
│               └── direct/
│                   └── burger_assembly_arena/  # TASK
│                       ├── __init__.py         # gym.register() here!
│                       ├── burger_assembly_arena_env_cfg.py
│                       └── burger_assembly_arena_env.py
├── README.md
└── LICENSE
```

> 📖 **Reference**: [Project Structure](https://isaac-sim.github.io/IsaacLab/main/source/overview/own-project/project_structure.html)  
> 📺 **Video**: [LycheeAI Template Generator Tutorial](https://lycheeai-hub.com/isaac-lab/build-your-own-isaac-lab-external-project-template-generator)

---

## Step 3: Add Arena Integration Files

Now we modify the template to use Arena's composable architecture.

### 3.1 Create Arena Environment Directory Structure

```bash
cd burger_assembly_arena

# Create Arena-specific directories
mkdir -p source/burger_assembly_arena/burger_assembly_arena/assets
mkdir -p source/burger_assembly_arena/burger_assembly_arena/embodiments
mkdir -p source/burger_assembly_arena/burger_assembly_arena/environments
mkdir -p source/burger_assembly_arena/burger_assembly_arena/tasks_arena

# Create __init__.py files
touch source/burger_assembly_arena/burger_assembly_arena/assets/__init__.py
touch source/burger_assembly_arena/burger_assembly_arena/embodiments/__init__.py
touch source/burger_assembly_arena/burger_assembly_arena/environments/__init__.py
touch source/burger_assembly_arena/burger_assembly_arena/tasks_arena/__init__.py
```

### 3.2 Create Custom Asset: Burger Patty

Create `source/burger_assembly_arena/burger_assembly_arena/assets/patty.py`:

```python
# =============================================================================
# PATTY ASSET DEFINITION
# This file defines a burger patty as a registerable Arena asset.
# Reference: https://isaac-sim.github.io/IsaacLab-Arena/release/0.1.1/pages/concepts/concept_assets_design.html
# =============================================================================

from isaaclab_arena.assets.object_library import LibraryObject, register_asset
from isaaclab_arena.assets.object_base import ObjectType


@register_asset  # This decorator adds the asset to Arena's registry
class Patty(LibraryObject):
    """Burger patty for stacking task.
    
    This asset can be retrieved using:
        patty = asset_registry.get_asset_by_name("patty")()
    
    Or find all food items:
        foods = asset_registry.get_assets_by_tag("food")
    """
    
    # ==========================================================================
    # REQUIRED: Asset identification
    # ==========================================================================
    name = "patty"  # Used in: asset_registry.get_asset_by_name("patty")
    
    # Tags for grouping/filtering assets
    tags = ["object", "food", "burger", "stackable"]
    
    # ==========================================================================
    # REQUIRED: Asset source
    # ==========================================================================
    # Option 1: Use NVIDIA Nucleus asset (if available)
    # usd_path = f"{ISAAC_NUCLEUS_DIR}/Props/Food/Patty/patty.usd"
    
    # Option 2: Use local USD file in your project
    usd_path = "source/burger_assembly_arena/assets/usd/patty.usd"
    
    # Option 3: Use a simple primitive for prototyping
    # (We'll use this for now - replace with real USD later)
    
    # ==========================================================================
    # REQUIRED: Physics type
    # ==========================================================================
    object_type = ObjectType.RIGID  # Options: RIGID, DEFORMABLE, ARTICULATED
    
    # ==========================================================================
    # OPTIONAL: Physics properties
    # ==========================================================================
    mass = 0.15  # kg (realistic patty weight)
    
    # Collision properties
    collision_enabled = True
    contact_offset = 0.001  # m
    rest_offset = 0.0  # m
    
    # Friction for realistic food handling
    static_friction = 0.6
    dynamic_friction = 0.5
    restitution = 0.1  # Low bounce
    
    # ==========================================================================
    # OPTIONAL: Visual properties
    # ==========================================================================
    # These are set in the USD file, but can be overridden
    scale = (1.0, 1.0, 1.0)


@register_asset
class CheeseSlice(LibraryObject):
    """Cheese slice for burger assembly.
    
    Similar to patty but with different physics properties.
    """
    
    name = "cheese_slice"
    tags = ["object", "food", "burger", "stackable"]
    
    usd_path = "source/burger_assembly_arena/assets/usd/cheese.usd"
    object_type = ObjectType.RIGID
    
    mass = 0.03  # kg (lighter than patty)
    static_friction = 0.4
    dynamic_friction = 0.3
    restitution = 0.05


@register_asset
class BunBottom(LibraryObject):
    """Bottom bun - the base for stacking."""
    
    name = "bun_bottom"
    tags = ["object", "food", "burger", "base"]
    
    usd_path = "source/burger_assembly_arena/assets/usd/bun_bottom.usd"
    object_type = ObjectType.RIGID
    
    mass = 0.05
    static_friction = 0.5
    dynamic_friction = 0.4


@register_asset
class BunTop(LibraryObject):
    """Top bun - placed last on the burger."""
    
    name = "bun_top"
    tags = ["object", "food", "burger"]
    
    usd_path = "source/burger_assembly_arena/assets/usd/bun_top.usd"
    object_type = ObjectType.RIGID
    
    mass = 0.04
    static_friction = 0.5
    dynamic_friction = 0.4
```

> 📖 **Reference**: [Assets Design Concepts](https://isaac-sim.github.io/IsaacLab-Arena/release/0.1.1/pages/concepts/concept_assets_design.html)

### 3.3 Create Arena Environment Definition

Create `source/burger_assembly_arena/burger_assembly_arena/environments/burger_stack_env.py`:

```python
# =============================================================================
# BURGER STACK ARENA ENVIRONMENT
# This is the core file where we compose Scene + Embodiment + Task
# Reference: https://isaac-sim.github.io/IsaacLab-Arena/main/pages/example_workflows/static_manipulation/step_1_environment_setup.html
# =============================================================================

import argparse
from typing import Optional

# =============================================================================
# Arena Imports
# Reference: https://isaac-sim.github.io/IsaacLab-Arena/main/index.html
# =============================================================================
from isaaclab_arena.environments.isaaclab_arena_environment import IsaacLabArenaEnvironment
from isaaclab_arena.scene.scene import Scene
from isaaclab_arena.tasks.pick_and_place_task import PickAndPlaceTask
from isaaclab_arena.assets.asset_registry import asset_registry
from isaaclab_arena.teleop.device_registry import device_registry
from isaaclab_arena.utils.pose import Pose
from isaaclab_arena.assets.object_reference import ObjectReference, ObjectType

# Import our custom assets (registers them when imported)
from ..assets import patty  # This import triggers @register_asset


class BurgerStackEnvironment:
    """Arena environment for burger patty stacking.
    
    This environment demonstrates Arena's composable architecture:
    - Embodiment (robot) can be switched via CLI argument
    - Scene objects can be easily added/removed
    - Task logic is reusable across different setups
    
    Usage:
        env_def = BurgerStackEnvironment()
        arena_env = env_def.get_env(args_cli)
        env_builder = ArenaEnvBuilder(arena_env, args_cli)
        env = env_builder.make_registered()
    
    Reference:
        https://isaac-sim.github.io/IsaacLab-Arena/main/pages/example_workflows/static_manipulation/step_1_environment_setup.html
    """
    
    # Environment name (used for gym registration)
    name: str = "burger_patty_stack"
    
    def get_env(self, args_cli: argparse.Namespace) -> IsaacLabArenaEnvironment:
        """Compose and return the Arena environment.
        
        Args:
            args_cli: Command line arguments containing:
                - embodiment: Robot name (e.g., "franka", "xarm6")
                - enable_cameras: Whether to enable camera sensors
                - teleop_device: Input device name (e.g., "keyboard")
        
        Returns:
            IsaacLabArenaEnvironment: Composed environment ready for building
        """
        
        # =====================================================================
        # STEP 1: Get Embodiment (Robot) from Registry
        # This is SWAPPABLE via CLI: --embodiment franka OR --embodiment xarm6
        # Reference: https://isaac-sim.github.io/IsaacLab-Arena/main/pages/concepts/concept_embodiment_design.html
        # =====================================================================
        embodiment = asset_registry.get_asset_by_name(
            args_cli.embodiment  # e.g., "franka", "xarm6", "ur10"
        )(
            enable_cameras=getattr(args_cli, 'enable_cameras', False)
        )
        
        # Set robot's initial pose in the world
        embodiment.set_initial_pose(Pose(
            position_xyz=(0.0, 0.0, 0.0),  # Robot base at origin
            rotation_wxyz=(1.0, 0.0, 0.0, 0.0)  # No rotation (quaternion)
        ))
        
        # =====================================================================
        # STEP 2: Get Scene Objects from Registry
        # EASILY SWAPPABLE: Change "patty" to "cheese_slice" for different task
        # Reference: https://isaac-sim.github.io/IsaacLab-Arena/release/0.1.1/pages/concepts/concept_assets_design.html
        # =====================================================================
        
        # Background/table (where everything sits)
        background = asset_registry.get_asset_by_name("table")()
        # Alternative: kitchen = asset_registry.get_asset_by_name("kitchen")()
        
        # Object to manipulate - THIS IS THE KEY SWAPPABLE COMPONENT
        pick_object = asset_registry.get_asset_by_name("patty")()
        # To test with different object, just change to:
        # pick_object = asset_registry.get_asset_by_name("cheese_slice")()
        # pick_object = asset_registry.get_asset_by_name("tomato_soup_can")()  # Pre-built Arena asset!
        
        # =====================================================================
        # STEP 3: Set Object Initial Poses
        # Reference: https://isaac-sim.github.io/IsaacLab-Arena/main/pages/concepts/concept_scene_design.html
        # =====================================================================
        pick_object.set_initial_pose(Pose(
            position_xyz=(0.5, 0.0, 0.1),  # In front of robot, slightly elevated
            rotation_wxyz=(1.0, 0.0, 0.0, 0.0)
        ))
        
        # =====================================================================
        # STEP 4: Define Destination Location
        # This is where the robot should place the object
        # =====================================================================
        destination = ObjectReference(
            name="stack_position",
            # Use ENV_REGEX_NS for per-environment paths
            prim_path="{ENV_REGEX_NS}/destination_marker",
            parent_asset=background,
            object_type=ObjectType.RIGID,
        )
        # Alternative: Reference a location on the table/kitchen
        # destination = ObjectReference(
        #     name="assembly_station",
        #     prim_path="{ENV_REGEX_NS}/table/assembly_zone",
        #     parent_asset=background,
        #     object_type=ObjectType.RIGID,
        # )
        
        # =====================================================================
        # STEP 5: Compose Scene
        # The Scene is just a list of assets - very composable!
        # Reference: https://isaac-sim.github.io/IsaacLab-Arena/main/pages/concepts/concept_scene_design.html
        # =====================================================================
        scene = Scene(assets=[
            background,
            pick_object,
            # Add more objects easily:
            # asset_registry.get_asset_by_name("cheese_slice")(),
            # asset_registry.get_asset_by_name("bun_bottom")(),
        ])
        
        # =====================================================================
        # STEP 6: Define Task
        # Tasks are REUSABLE - PickAndPlaceTask works with ANY object
        # Reference: https://isaac-sim.github.io/IsaacLab-Arena/main/pages/concepts/concept_tasks_design.html
        # =====================================================================
        task = PickAndPlaceTask(
            pick_up_object=pick_object,
            destination_location=destination,
            background_scene=background,
            # Optional parameters:
            # success_threshold=0.05,  # Distance tolerance for success
            # grasp_height_offset=0.02,  # Approach height above object
        )
        
        # =====================================================================
        # STEP 7: Get Teleoperation Device
        # For data collection and testing
        # =====================================================================
        teleop_device = device_registry.get_device_by_name(
            getattr(args_cli, 'teleop_device', 'keyboard')
        )()
        
        # =====================================================================
        # STEP 8: Compose Final Environment
        # This brings everything together into an IsaacLabArenaEnvironment
        # =====================================================================
        return IsaacLabArenaEnvironment(
            name=self.name,
            embodiment=embodiment,
            scene=scene,
            task=task,
            teleop_device=teleop_device,
        )


class BurgerCheeseStackEnvironment(BurgerStackEnvironment):
    """Variant: Stack cheese instead of patty.
    
    This demonstrates how easy it is to create variants with Arena!
    """
    
    name: str = "burger_cheese_stack"
    
    def get_env(self, args_cli: argparse.Namespace) -> IsaacLabArenaEnvironment:
        # Get base environment
        base_env = super().get_env(args_cli)
        
        # Just swap the object
        cheese = asset_registry.get_asset_by_name("cheese_slice")()
        cheese.set_initial_pose(Pose(
            position_xyz=(0.5, 0.0, 0.1),
            rotation_wxyz=(1.0, 0.0, 0.0, 0.0)
        ))
        
        # Update scene and task
        base_env.scene = Scene(assets=[
            asset_registry.get_asset_by_name("table")(),
            cheese,
        ])
        base_env.task = PickAndPlaceTask(
            pick_up_object=cheese,
            destination_location=base_env.task.destination_location,
            background_scene=asset_registry.get_asset_by_name("table")(),
        )
        base_env.name = self.name
        
        return base_env
```

> 📖 **Reference**: [Environment Setup Example](https://isaac-sim.github.io/IsaacLab-Arena/main/pages/example_workflows/static_manipulation/step_1_environment_setup.html)

### 3.4 Create Entry Point Script

Create `scripts/run_arena_env.py`:

```python
#!/usr/bin/env python3
# =============================================================================
# ARENA ENVIRONMENT ENTRY POINT
# This script demonstrates how to run an Arena environment.
# 
# Usage examples:
#   # Run with Franka robot
#   python scripts/run_arena_env.py --embodiment franka
#   
#   # Run with different robot (if available)
#   python scripts/run_arena_env.py --embodiment xarm6
#   
#   # Run with cameras enabled
#   python scripts/run_arena_env.py --embodiment franka --enable_cameras
#
# Reference: https://isaac-sim.github.io/IsaacLab-Arena/main/index.html
# =============================================================================

"""Launch Isaac Sim Simulator first."""

import argparse
from isaaclab.app import AppLauncher

# =============================================================================
# STEP 1: Parse Command Line Arguments
# These arguments control which robot, device, and settings to use
# =============================================================================
parser = argparse.ArgumentParser(
    description="Run burger assembly Arena environment"
)

# Arena-specific arguments
parser.add_argument(
    "--embodiment", 
    type=str, 
    default="franka",
    help="Robot embodiment name from registry (e.g., franka, ur10, xarm6)"
)
parser.add_argument(
    "--teleop_device", 
    type=str, 
    default="keyboard",
    help="Teleop device name (keyboard, spacemouse, gamepad)"
)
parser.add_argument(
    "--enable_cameras", 
    action="store_true",
    help="Enable camera sensors on the robot"
)

# Standard Isaac Lab arguments
parser.add_argument(
    "--num_envs", 
    type=int, 
    default=1,
    help="Number of parallel environments"
)
parser.add_argument(
    "--headless", 
    action="store_true",
    help="Run without GUI (for training)"
)

# Add Isaac Lab/Sim arguments (device, etc.)
AppLauncher.add_app_launcher_args(parser)

args_cli = parser.parse_args()

# =============================================================================
# STEP 2: Launch Isaac Sim Application
# IMPORTANT: This must happen BEFORE any other Isaac imports!
# Reference: https://isaac-sim.github.io/IsaacLab/main/source/tutorials/00_sim/launch_app.html
# =============================================================================
app_launcher = AppLauncher(args_cli)
simulation_app = app_launcher.app

# =============================================================================
# STEP 3: Import Arena and Environment (AFTER app launch)
# =============================================================================
import torch

from isaaclab_arena.env_builder.arena_env_builder import ArenaEnvBuilder

# Import our custom environment (this also imports/registers our assets)
from burger_assembly_arena.environments.burger_stack_env import BurgerStackEnvironment


def main():
    """Main function to run the Arena environment."""
    
    # =========================================================================
    # STEP 4: Create Environment Definition
    # =========================================================================
    print(f"\n{'='*60}")
    print(f"Creating Burger Stack Environment")
    print(f"  Embodiment: {args_cli.embodiment}")
    print(f"  Teleop Device: {args_cli.teleop_device}")
    print(f"  Cameras: {args_cli.enable_cameras}")
    print(f"  Num Envs: {args_cli.num_envs}")
    print(f"{'='*60}\n")
    
    # Get environment definition
    env_definition = BurgerStackEnvironment()
    arena_env = env_definition.get_env(args_cli)
    
    # =========================================================================
    # STEP 5: Build and Register Environment
    # ArenaEnvBuilder converts Arena components into Isaac Lab ManagerBasedRLEnvCfg
    # Reference: https://deepwiki.com/isaac-sim/IsaacLab-Arena/7.1-creating-custom-environments
    # =========================================================================
    env_builder = ArenaEnvBuilder(arena_env, args_cli)
    
    # This does two things:
    # 1. Registers environment with Gymnasium
    # 2. Returns the environment configuration
    env = env_builder.make_registered()
    
    print(f"Environment registered as: {env_definition.name}")
    print(f"Observation space: {env.observation_space}")
    print(f"Action space: {env.action_space}")
    
    # =========================================================================
    # STEP 6: Run Simulation Loop
    # =========================================================================
    print("\nStarting simulation...")
    print("Press Ctrl+C to stop\n")
    
    obs, _ = env.reset()
    
    step_count = 0
    try:
        while simulation_app.is_running():
            # Random actions for testing
            # Replace with your policy for training/evaluation
            action = env.action_space.sample()
            
            # Environment step
            obs, reward, terminated, truncated, info = env.step(action)
            
            step_count += 1
            
            # Print progress every 100 steps
            if step_count % 100 == 0:
                print(f"Step {step_count}, Reward: {reward.mean().item():.4f}")
            
            # Reset if episode ended
            if terminated.any() or truncated.any():
                print(f"Episode ended at step {step_count}, resetting...")
                obs, _ = env.reset()
                
    except KeyboardInterrupt:
        print("\nSimulation stopped by user")
    
    # =========================================================================
    # STEP 7: Cleanup
    # =========================================================================
    env.close()
    simulation_app.close()


if __name__ == "__main__":
    main()
```

> 📖 **Reference**: [Isaac Lab AppLauncher Tutorial](https://isaac-sim.github.io/IsaacLab/main/source/tutorials/00_sim/launch_app.html)

---

## Step 4: Register Assets and Environment

### 4.1 Update Module `__init__.py`

Update `source/burger_assembly_arena/burger_assembly_arena/__init__.py`:

```python
# =============================================================================
# BURGER ASSEMBLY ARENA MODULE
# This __init__.py imports and exposes all components of the module.
# =============================================================================

# Import assets to register them with Arena
from .assets import patty  # Triggers @register_asset decorators

# Import environments
from .environments.burger_stack_env import (
    BurgerStackEnvironment,
    BurgerCheeseStackEnvironment,
)

# Module version
__version__ = "0.1.0"

# Expose for easy import
__all__ = [
    "BurgerStackEnvironment",
    "BurgerCheeseStackEnvironment",
]
```

### 4.2 Update Assets `__init__.py`

Update `source/burger_assembly_arena/burger_assembly_arena/assets/__init__.py`:

```python
# =============================================================================
# ASSETS MODULE
# Importing this module registers all custom assets with Arena's registry.
# =============================================================================

from . import patty

# You can also explicitly import classes if needed
from .patty import Patty, CheeseSlice, BunBottom, BunTop

__all__ = [
    "Patty",
    "CheeseSlice", 
    "BunBottom",
    "BunTop",
]
```

---

## Step 5: Install and Test

### 5.1 Install the Project

```bash
# Navigate to project root
cd burger_assembly_arena

# Install in editable mode
pip install -e source/burger_assembly_arena

# Verify installation
python -c "import burger_assembly_arena; print('Success!')"
```

### 5.2 List Registered Environments

```bash
# Use the built-in list script
python scripts/list_envs.py

# You should see your environment listed:
# burger_patty_stack
```

### 5.3 Test with Random Agent

```bash
# Test environment loads correctly
python scripts/run_arena_env.py --embodiment franka --num_envs 1

# Test with headless mode (faster, no GUI)
python scripts/run_arena_env.py --embodiment franka --num_envs 16 --headless
```

---

## Step 6: Train with RL

### 6.1 Create Training Configuration

Create `source/burger_assembly_arena/burger_assembly_arena/agents/skrl_ppo_cfg.py`:

```python
# =============================================================================
# SKRL PPO CONFIGURATION FOR BURGER STACKING
# Reference: https://skrl.readthedocs.io/en/latest/
# =============================================================================

from skrl.agents.torch.ppo import PPO_DEFAULT_CONFIG

SKRL_PPO_CFG = PPO_DEFAULT_CONFIG.copy()

# Update with task-specific hyperparameters
SKRL_PPO_CFG.update({
    # Learning parameters
    "learning_rate": 3e-4,
    "learning_rate_scheduler": "KLAdaptiveLR",
    "learning_rate_scheduler_kwargs": {
        "kl_threshold": 0.01,
    },
    
    # PPO parameters
    "rollouts": 16,
    "learning_epochs": 8,
    "mini_batches": 4,
    "discount_factor": 0.99,
    "lambda": 0.95,
    "clip_ratio": 0.2,
    "value_clip_ratio": 0.2,
    
    # Entropy for exploration
    "entropy_loss_scale": 0.01,
    
    # Network
    "state_preprocessor": None,
    "state_preprocessor_kwargs": {},
    "value_preprocessor": None,
    "value_preprocessor_kwargs": {},
})
```

### 6.2 Run Training

```bash
# Train with SKRL
python scripts/skrl/train.py \
    --task=burger_patty_stack \
    --num_envs=1024 \
    --headless \
    --max_iterations=5000

# Monitor with TensorBoard
tensorboard --logdir=logs/burger_patty_stack
```

> 📖 **Reference**: [Isaac Lab RL Training](https://isaac-sim.github.io/IsaacLab/main/source/tutorials/03_envs/run_rl_training.html)

---

## Step 7: Easy Switching Demo

### 7.1 Switch Robots

```bash
# Train with Franka
python scripts/run_arena_env.py --embodiment franka

# Train with different robot (if registered in Arena)
python scripts/run_arena_env.py --embodiment ur10

# The ONLY change is the --embodiment argument!
```

### 7.2 Switch Objects

In your environment file, simply change one line:

```python
# Original: Pick up patty
pick_object = asset_registry.get_asset_by_name("patty")()

# Variant 1: Pick up cheese
pick_object = asset_registry.get_asset_by_name("cheese_slice")()

# Variant 2: Use Arena's pre-built objects
pick_object = asset_registry.get_asset_by_name("tomato_soup_can")()
pick_object = asset_registry.get_asset_by_name("cracker_box")()
pick_object = asset_registry.get_asset_by_name("mustard_bottle")()
```

### 7.3 Create Multiple Environment Variants

```python
# In your environments/__init__.py, register multiple variants:

ENVIRONMENTS = {
    "burger_patty_stack": BurgerStackEnvironment,
    "burger_cheese_stack": BurgerCheeseStackEnvironment,
    # Easy to add more:
    "burger_lettuce_stack": BurgerLettuceStackEnvironment,
    "burger_full_assembly": BurgerFullAssemblyEnvironment,
}

# Now you can train on ANY variant with:
# python scripts/skrl/train.py --task=burger_cheese_stack
```

---

## 📋 Quick Reference Summary

### File Locations

| File | Purpose |
|------|---------|
| `environments/burger_stack_env.py` | Arena environment composition |
| `assets/patty.py` | Custom asset definitions |
| `scripts/run_arena_env.py` | Entry point for running |
| `scripts/skrl/train.py` | RL training script |

### Key Arena APIs

```python
# Get robot from registry
embodiment = asset_registry.get_asset_by_name("franka")(enable_cameras=True)

# Get objects from registry  
obj = asset_registry.get_asset_by_name("patty")()
obj.set_initial_pose(Pose(position_xyz=(x, y, z)))

# Compose scene
scene = Scene(assets=[background, obj1, obj2])

# Define task
task = PickAndPlaceTask(pick_obj, destination, background)

# Create environment
env = IsaacLabArenaEnvironment(name, embodiment, scene, task, teleop)

# Build and register
builder = ArenaEnvBuilder(env, args_cli)
gym_env = builder.make_registered()
```

### CLI Commands

```bash
# Generate new project
./isaaclab.sh --new

# Install project
pip install -e source/burger_assembly_arena

# Run environment
python scripts/run_arena_env.py --embodiment franka

# Train policy
python scripts/skrl/train.py --task=burger_patty_stack --headless

# Evaluate policy
python scripts/skrl/play.py --task=burger_patty_stack --checkpoint=path/to/model
```

---

## 🔗 Additional Resources

### Documentation
- [Isaac Lab Arena Main Docs](https://isaac-sim.github.io/IsaacLab-Arena/main/index.html)
- [Isaac Lab Tutorials](https://isaac-sim.github.io/IsaacLab/main/source/tutorials/index.html)
- [NVIDIA Developer Blog on Arena](https://developer.nvidia.com/blog/simplify-generalist-robot-policy-evaluation-in-simulation-with-nvidia-isaac-lab-arena)

### Video Tutorials
- [LycheeAI Hub - Isaac Lab](https://lycheeai-hub.com/isaac-lab)
- [LycheeAI YouTube Channel](https://www.youtube.com/@LycheeAI)

### Example Code
- [Arena Example Environments](https://github.com/isaac-sim/IsaacLab-Arena/tree/main/isaaclab_arena/examples/example_environments)
- [GR1 Open Microwave Example](https://isaac-sim.github.io/IsaacLab-Arena/main/pages/example_workflows/static_manipulation/index.html)
- [G1 Locomanipulation Example](https://isaac-sim.github.io/IsaacLab-Arena/main/pages/example_workflows/locomanipulation/index.html)

### Community
- [Isaac Lab GitHub Discussions](https://github.com/isaac-sim/IsaacLab/discussions)
- [NVIDIA Developer Forum - Robotics](https://forums.developer.nvidia.com/c/robotics/)

---

*Created for burger assembly automation project - easy task and object switching for sim-to-real robotics development.*
