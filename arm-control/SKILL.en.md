---
name: arm-control
description: Robot arm control through the skill-local pycarm over WebSocket, including connection, motion control, gripper control, and trajectory teaching. Activate when the user mentions robot arm, control arm, arm control, or 机械臂.
emoji: 🤖
metadata:
  openclaw:
    requires:
      bins: ["python3.8", "git"]
---

# Robot Arm Control

## Activation

Activate this skill when the user mentions:

- robot arm
- control arm
- arm control
- 机械臂

## Goal

Use the `pycarm` directory bundled with the skill to control the robot arm.

In the source repository:

```text
arm-control/pycarm
```

is managed as a Git submodule.

In the installed skill directory:

```text
pycarm
```

is the local SDK directory that should be used at runtime.

## Behavior Rules

1. Ask the user for the robot arm IP address before any control action.
2. Always prefer the `pycarm` directory inside the installed skill instead of looking for an external copy in the user's Home directory.
3. If the installed skill's `pycarm` is missing or unusable, repair or update that directory before controlling the arm.
4. If `pycarm` still has git metadata, it is acceptable to initialize it, update it, or switch versions. If it does not, re-clone that directory instead.
5. Use one import model only: add the skill-local `pycarm` repository root to `sys.path`, then run `from carm import Carm`.
6. Do not use any machine-specific absolute development paths.
7. Do not point `sys.path` at the `carm.py` file itself.
8. Keep examples limited to stable, basic public APIs. Do not dynamically inspect `carm.py` on every run to decide how to call it.

## Environment Setup

```bash
pip install websocket-client
```

## Submodule Maintenance

If you are maintaining this skill from source, prefer:

```bash
git clone --recurse-submodules https://github.com/cvte-robotics/openclaw.git
```

If the repository already exists but the submodule is not ready:

```bash
git submodule update --init --recursive
```

## `pycarm` Inside the Installed Skill

If OpenClaw installs the skill but does not fully prepare `pycarm`, and that directory still has git metadata, check and update it from the installed skill directory:

```bash
git -C pycarm pull
```

If `pycarm` does not have git metadata, re-clone it directly:

```bash
git clone https://github.com/cvte-robotics/pycarm.git pycarm
```

If you need to pin a version:

```bash
git -C pycarm fetch --tags
git -C pycarm checkout <tag-or-commit>
```

## Quick Start

```python
import sys
from pathlib import Path

skill_root = Path.home() / ".openclaw" / "workspace" / "skills" / "arm-control"
pycarm_root = skill_root / "pycarm"
sys.path.insert(0, str(pycarm_root))

from carm import Carm

arm = Carm(addr="<USER_PROVIDED_IP>")
arm.set_ready()
arm.move_joint([0, 0, 0, 0, 0, 0])
arm.set_gripper(0.04)
arm.set_gripper(0.0)
```

## Connection Management

```python
arm = Carm(addr="<USER_PROVIDED_IP>", arm_index=0)
arm.connect()
arm.disconnect()
arm.is_connected()
```

## Units

- Joint positions: radians
- Gripper position: meters, where `0` is closed and `0.08` is fully open
- Cartesian pose: `[x, y, z, qw, qx, qy, qz]`

## Joint Indexing

The robot arm joints are J1 through J6, mapped to array indexes `0` through `5`.

```python
arm.move_joint([0, 1.0, -1.5, 0, 0, 0])
# J1=0, J2=1.0, J3=-1.5, J4=0, J5=0, J6=0
```

## Status Monitoring

```python
arm.joint_pos
arm.joint_vel
arm.joint_tau

arm.cart_pose
arm.plan_cart_pose

arm.gripper_pos
arm.gripper_state
arm.gripper_tau
arm.plan_gripper_pos

arm.version
arm.get_limits()
```

## Motion Control

```python
arm.move_joint([0.5, 0.3, -0.5, 0.2, 0.1, 0])
arm.move_line_joint([0.1, 0.2, 0.1, 0, 0, 0])
arm.track_joint([0.5, 0.3, -0.5, 0.2, 0.1, 0])

arm.move_pose([0.3, 0, 0.4, 1, 0, 0, 0])
arm.move_line_pose([0.3, 0, 0.4, 1, 0, 0, 0])
arm.move_flow_pose(
    target_pos=[0.3, 0, 0.4, 1, 0, 0, 0],
    line_theta_weight=0.5,
    accuracy=0.0001,
)
arm.track_pose([0.3, 0, 0.4, 1, 0, 0, 0])
```

## Gripper Control

```python
arm.set_gripper(0.04)
arm.set_gripper(0.0)
arm.set_gripper(0.06, tau=20)
```

## Ready and Safety

```python
arm.set_ready()
arm.set_ready(timeout_ms=5000)
arm.set_servo_enable(True)
arm.set_control_mode(1)
arm.set_collision_config(True, 10)
arm.set_speed_level(5.0)
```

## Stop and Recover

```python
arm.stop()
arm.stop(1)
arm.stop(2)
arm.stop(3)
arm.stop_task(True)
arm.recover()
arm.clean_carm_error()
```

## Teaching and Trajectory Playback

```python
arm.trajectory_teach(True, "test.my_traj")
arm.trajectory_teach(False)
arm.trajectory_recorder("test.my_traj")
arm.check_teach()
```

## Kinematics and Callbacks

```python
pose = arm.forward_kine([0.1, 0.2, 0.3, 0, 0, 0])

joints = arm.inverse_kine(
    cart_pose=[0.3, 0, 0.4, 1, 0, 0, 0],
    ref_joints=[0, 0, 0, 0, 0, 0],
)

def on_error(err):
    print(f"Error {err['error']}: {err['errMsg']}")

def on_finish(task_key):
    print(f"Task {task_key} finished")

arm.on_error(on_error)
arm.on_task_finish(on_finish)
```

## Full Example

```python
import sys
from pathlib import Path

skill_root = Path.home() / ".openclaw" / "workspace" / "skills" / "arm-control"
pycarm_root = skill_root / "pycarm"
sys.path.insert(0, str(pycarm_root))

from carm import Carm

arm = Carm(addr="<USER_PROVIDED_IP>")

if not arm.set_ready(timeout_ms=5000):
    raise RuntimeError("Robot arm failed to enter ready state")

arm.set_gripper(0.06)
arm.move_joint([0.5, 0.3, -0.5, 0.2, 0.1, 0])
arm.set_gripper(0.0)
arm.move_line_pose([0.4, 0.2, 0.3, 1, 0, 0, 0])
arm.set_gripper(0.06)
arm.move_joint([0, 0, 0, 0, 0, 0])
```

## Execute via exec

```bash
python3.8 -c "
import sys
from pathlib import Path
skill_root = Path.home() / '.openclaw' / 'workspace' / 'skills' / 'arm-control'
pycarm_root = skill_root / 'pycarm'
sys.path.insert(0, str(pycarm_root))
from carm import Carm
arm = Carm(addr='<USER_PROVIDED_IP>')
arm.set_ready()
arm.set_gripper(0.04)
arm.move_joint([0.5, 0.3, -0.5, 0.2, 0.1, 0])
arm.set_gripper(0.0)
print('Done')
"
```

## Notes

- Ask for the robot arm IP before connecting
- Use the skill-local `pycarm` by default
- If `pycarm` does not keep git metadata after installation, update it by re-cloning that directory instead of using `git pull`
- If you upgrade `pycarm`, re-check whether the examples still match the current API
- No board SDK is required for WebSocket mode
- Units: joints in radians, gripper in meters
- After execution, output the generated Python script content for the user to review
