---
name: arm-control
description: Robot arm control through the skill-local pycarm over WebSocket, including connection, motion control, gripper/dexterous hand control, and trajectory teaching. Activate when the user mentions robot arm, control arm, arm control, or 机械臂.
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
git clone --recurse-submodules https://github.com/cvte-robotics/carm-openclaw.git
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
- Gripper position: meters, `0` is closed, `0.08` is fully open
- Dexterous hand position: list (unit depends on hand model)
- Cartesian pose: `[x, y, z, qx, qy, qz, qw]`

## Joint Indexing

The robot arm joints are J1 through J6, mapped to array indexes `0` through `5`.

```python
arm.move_joint([0, 1.0, -1.5, 0, 0, 0])
# J1=0, J2=1.0, J3=-1.5, J4=0, J5=0, J6=0
```

## End Effector Type Detection

The end effector may be a **gripper** (single DOF) or a **dexterous hand** (multi-DOF). Detect via `hand_pos`:

```python
pos = arm.hand_pos
if isinstance(pos, list) and len(pos) == 1:
    end_effector_type = "gripper"   # single DOF
elif isinstance(pos, list) and len(pos) > 1:
    end_effector_type = "hand"      # multi DOF
else:
    end_effector_type = "unknown"   # not connected
```

> `hand_pos` returns a list of length 1 for gripper, >1 for dexterous hand. Returns `[]` when disconnected.

## Status Monitoring

### Joint Status

```python
arm.joint_pos          # actual joint positions (rad)
arm.joint_vel          # actual joint velocities (rad/s)
arm.joint_tau          # actual joint torques (N·m)
arm.plan_joint_pos     # planned joint positions
arm.plan_joint_vel     # planned joint velocities
arm.plan_joint_tau     # planned joint torques
```

### Cartesian Status

```python
arm.cart_pose          # actual Cartesian pose [x, y, z, qx, qy, qz, qw]
arm.plan_cart_pose     # planned Cartesian pose
arm.cart_external_force  # Cartesian external force [fx, fy, fz, tx, ty, tz]
arm.joint_external_tau   # joint external torques [tau1~tauN]
```

### End Effector Status

```python
# Generic (auto-detects gripper vs hand)
arm.end_effector_state       # state (-1=disconnected, 0=disabled, 1=normal)
arm.end_effector_pos / vel / tau
arm.plan_end_effector_pos / vel / tau

# Gripper attributes (valid when ee_type == "gripper")
arm.gripper_state / pos / tau
arm.plan_gripper_pos

# Dexterous hand attributes (valid when ee_type == "hand")
arm.hand_state / pos / vel / tau
arm.plan_hand_pos / vel / tau
```

### System Info

```python
arm.version          # controller software version
arm.get_limits()     # joint limits, max velocity/acceleration
arm.tool_index       # current tool index
```

## Motion Control

```python
# Joint motion
arm.move_joint([0.5, 0.3, -0.5, 0.2, 0.1, 0])   # point-to-point (recommended, fast)
arm.move_line_joint([0.1, 0.2, 0.1, 0, 0, 0])    # linear
arm.track_joint([...])                              # real-time tracking

# Cartesian motion
arm.move_pose([0.3, 0, 0.4, 1, 0, 0, 0])          # point-to-point
arm.move_line_pose([0.3, 0, 0.4, 1, 0, 0, 0])     # linear interpolation
arm.move_flow_pose(                                  # high-precision Jacobian
    target_pos=[0.3, 0, 0.4, 1, 0, 0, 0],
    line_theta_weight=0.5,
    accuracy=0.0001,
)
arm.track_pose([...])                                # real-time tracking
```

## End Effector Control

### Gripper

```python
arm.set_gripper(0.04)          # open to 4cm
arm.set_gripper(0.0)           # close
arm.set_gripper(0.06, tau=20)  # open to 6cm, torque limit 20N
```

### Dexterous Hand

```python
# pos/tau/vel must be equal-length lists
arm.set_hand(pos=[0.5, 0.5, 0.5], tau=[5, 5, 5], vel=[1.0, 1.0, 1.0])
```

> 6-DOF index: `[0]thumb joint1 [1]thumb joint2 [2]index [3]middle [4]ring [5]pinky`, range 0~255.

### Generic End Effector

```python
arm.set_end_effector(dof=1, pos=[0.04], vel=[0.0], tau=[10.0])  # same as set_gripper
arm.set_end_effector(dof=6, pos=[...], vel=[...], tau=[...])    # dexterous hand
```

## Dexterous Hand Examples

### Handshake

```python
import time
DOF = 6; TAU = [128]*DOF; VEL = [128]*DOF

arm.set_hand(pos=[255]*DOF, tau=TAU, vel=VEL)                      # open
time.sleep(1)
arm.move_joint([-0.0017, 1.1161, -0.7147, -0.0528, -0.3523, 1.6752])  # reach forward
time.sleep(2)
arm.set_hand(pos=[127, 127, 64, 64, 64, 64], tau=TAU, vel=VEL)    # grip
time.sleep(1)
for _ in range(3):                                                  # shake 3 times
    arm.move_joint([-0.0017, 1.1161, -0.5147, -0.0528, -0.3523, 1.6752]); time.sleep(0.5)  # up
    arm.move_joint([-0.0017, 1.1161, -0.9147, -0.0528, -0.3523, 1.6752]); time.sleep(0.5)  # down
arm.set_hand(pos=[255]*DOF, tau=TAU, vel=VEL)                      # release
arm.move_joint([0, 0.998, -0.7155, -0.038, -0.3504, -0.0025])     # safe position
```

### Rock Paper Scissors

```python
import time, random
DOF = 6; TAU = [128]*DOF; VEL = [128]*DOF

ROCK     = [0,   0,   0,   0,   0,   0  ]  # fist
SCISSORS = [0,   0,   255, 255, 0,   0  ]  # index + middle extended
PAPER    = [255, 255, 255, 255, 255, 255]  # all open

arm.set_hand(pos=ROCK, tau=TAU, vel=VEL)
time.sleep(1)
choice = random.choice([("Rock", ROCK), ("Scissors", SCISSORS), ("Paper", PAPER)])
print(f"I choose: {choice[0]}")
arm.set_hand(pos=choice[1], tau=TAU, vel=VEL)
```

## Tool Management

```python
arm.set_tool_index(1)                 # switch to tool 1
arm.tool_index                         # current tool index
arm.get_tool_coordinate(0)            # get tool 0 coordinate frame
```

## Ready and Safety

```python
arm.set_ready()                     # ready (clear error + enable servo + position mode)
arm.set_ready(timeout_ms=5000)      # with timeout
arm.set_servo_enable(True)          # enable servo
arm.set_control_mode(1)             # 0-IDLE, 1-position, 2-MIT, 3-drag, 4-force-position
arm.set_collision_config(True, 10)  # collision detection (sensitivity 0~2)
arm.set_speed_level(5.0)            # speed level 0~10
```

## Stop and Recover

```python
arm.stop()           # pause
arm.stop(1)          # stop
arm.stop(2)          # disable
arm.stop(3)          # emergency stop
arm.stop_task(True)  # immediately stop current task
arm.recover()        # recover from pause/e-stop
arm.clean_carm_error()  # clear controller errors
```

## Teaching and Trajectory Playback

```python
arm.trajectory_teach(True, "test.my_traj")  # start recording
arm.trajectory_teach(False)                  # stop recording
arm.trajectory_recorder("test.my_traj")      # replay trajectory
arm.check_teach()                            # list recorded trajectories
```

## Kinematics and Callbacks

```python
# Forward kinematics: joint angles -> Cartesian pose
pose = arm.forward_kine([0.1, 0.2, 0.3, 0, 0, 0])

# Inverse kinematics: Cartesian pose -> joint angles
joints = arm.inverse_kine(
    cart_pose=[0.3, 0, 0.4, 1, 0, 0, 0],
    ref_joints=[0, 0, 0, 0, 0, 0],
)

# Callbacks
arm.on_error(lambda err: print(f"Error {err['error']}: {err['errMsg']}"))
arm.on_task_finish(lambda key: print(f"Task {key} finished"))
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

# Detect end effector type
pos = arm.hand_pos
if isinstance(pos, list) and len(pos) == 1:
    print("End effector: gripper")
    arm.set_gripper(0.06)
    arm.move_joint([0.5, 0.3, -0.5, 0.2, 0.1, 0])
    arm.set_gripper(0.0)
elif isinstance(pos, list) and len(pos) > 1:
    print(f"End effector: dexterous hand ({len(pos)} DOF)")
    arm.set_hand(pos=[0.5]*len(pos), tau=[5]*len(pos), vel=[1.0]*len(pos))
    arm.move_joint([0.5, 0.3, -0.5, 0.2, 0.1, 0])
    arm.set_hand(pos=[0.0]*len(pos), tau=[5]*len(pos), vel=[1.0]*len(pos))
else:
    print("End effector: unknown or disconnected")

arm.move_line_pose([0.4, 0.2, 0.3, 1, 0, 0, 0])
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
