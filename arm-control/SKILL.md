---
name: arm-control
description: 机械臂控制。通过 skill 目录内的 pycarm 使用 WebSocket 直连控制机械臂，包括连接、运动控制、夹爪控制和轨迹示教。激活条件：用户提到机械臂、robot arm、控制机械臂、arm control。
emoji: 🤖
metadata:
  openclaw:
    requires:
      bins: ["python3.8", "git"]
---

# 机械臂控制

## 激活条件

当用户提到以下关键词时激活：

- 机械臂
- robot arm
- 控制机械臂
- arm control

## 目标

使用 skill 目录中的 `pycarm` 控制机械臂。

在源码仓库中：

```text
arm-control/pycarm
```

是由 Git submodule 管理的 SDK 目录。

在已安装的 skill 目录中：

```text
pycarm
```

是运行时应使用的本地 SDK 目录。

## 行为规则

1. 控制机械臂前，必须先询问用户机械臂 IP 地址。
2. 默认使用当前 skill 目录里的 `pycarm`，不要再去用户 Home 目录寻找外部副本。
3. 如果已安装 skill 目录中的 `pycarm` 缺失或不可用，先补齐或更新该目录，再执行控制。
4. 如果 `pycarm` 保留了 git 元数据，允许执行初始化、更新或切换版本；如果没有 git 元数据，则重新拉取该目录。
5. Python 导入方式统一为：把 skill 目录下的 `pycarm` 根目录加入 `sys.path`，再执行 `from carm import Carm`。
6. 不要使用任何开发机私有绝对路径。
7. 不要把 `sys.path` 指向 `carm.py` 文件本身。
8. 示例只使用稳定、基础的公共接口，不要在每次执行时动态分析 `carm.py` 再决定怎么调用。

## 环境准备

```bash
pip install websocket-client
```

## submodule 维护

如果你在源码仓库中维护这个 skill，建议使用：

```bash
git clone --recurse-submodules https://github.com/cvte-robotics/openclaw.git
```

如果仓库已存在但 submodule 尚未就绪：

```bash
git submodule update --init --recursive
```

## 已安装 Skill 中的 pycarm

如果 OpenClaw 安装 skill 后没有自动准备好 `pycarm`，且该目录保留了 git 元数据，可在已安装的 skill 目录中检查并更新：

```bash
git -C pycarm pull
```

如果 `pycarm` 没有 git 元数据，直接重新拉取：

```bash
git clone https://github.com/cvte-robotics/pycarm.git pycarm
```

如果需要固定版本：

```bash
git -C pycarm fetch --tags
git -C pycarm checkout <tag-or-commit>
```

## 快速开始

```python
import sys
from pathlib import Path

skill_root = Path.home() / ".openclaw" / "workspace" / "skills" / "arm-control"
pycarm_root = skill_root / "pycarm"
sys.path.insert(0, str(pycarm_root))

from carm import Carm

arm = Carm(addr="<用户提供的IP>")
arm.set_ready()
arm.move_joint([0, 0, 0, 0, 0, 0])
arm.set_gripper(0.04)
arm.set_gripper(0.0)
```

## 连接管理

```python
arm = Carm(addr="<用户提供的IP>", arm_index=0)
arm.connect()
arm.disconnect()
arm.is_connected()
```

## 单位说明

- 关节位置：弧度
- 夹爪位置：米，`0` 为闭合，`0.08` 为全开
- 笛卡尔位姿：`[x, y, z, qw, qx, qy, qz]`

## 关节索引

机械臂关节从 J1 到 J6，对应数组索引 `0` 到 `5`。

```python
arm.move_joint([0, 1.0, -1.5, 0, 0, 0])
# 对应 J1=0, J2=1.0, J3=-1.5, J4=0, J5=0, J6=0
```

## 状态监控

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

## 运动控制

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

## 夹爪控制

```python
arm.set_gripper(0.04)
arm.set_gripper(0.0)
arm.set_gripper(0.06, tau=20)
```

## 就绪与安全

```python
arm.set_ready()
arm.set_ready(timeout_ms=5000)
arm.set_servo_enable(True)
arm.set_control_mode(1)
arm.set_collision_config(True, 10)
arm.set_speed_level(5.0)
```

## 停止与恢复

```python
arm.stop()
arm.stop(1)
arm.stop(2)
arm.stop(3)
arm.stop_task(True)
arm.recover()
arm.clean_carm_error()
```

## 轨迹与示教

```python
arm.trajectory_teach(True, "test.my_traj")
arm.trajectory_teach(False)
arm.trajectory_recorder("test.my_traj")
arm.check_teach()
```

## 运动学与回调

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

## 完整控制流程示例

```python
import sys
from pathlib import Path

skill_root = Path.home() / ".openclaw" / "workspace" / "skills" / "arm-control"
pycarm_root = skill_root / "pycarm"
sys.path.insert(0, str(pycarm_root))

from carm import Carm

arm = Carm(addr="<用户提供的IP>")

if not arm.set_ready(timeout_ms=5000):
    raise RuntimeError("机械臂未进入就绪状态")

arm.set_gripper(0.06)
arm.move_joint([0.5, 0.3, -0.5, 0.2, 0.1, 0])
arm.set_gripper(0.0)
arm.move_line_pose([0.4, 0.2, 0.3, 1, 0, 0, 0])
arm.set_gripper(0.06)
arm.move_joint([0, 0, 0, 0, 0, 0])
```

## 通过 exec 执行

```bash
python3.8 -c "
import sys
from pathlib import Path
skill_root = Path.home() / '.openclaw' / 'workspace' / 'skills' / 'arm-control'
pycarm_root = skill_root / 'pycarm'
sys.path.insert(0, str(pycarm_root))
from carm import Carm
arm = Carm(addr='<用户提供的IP>')
arm.set_ready()
arm.set_gripper(0.04)
arm.move_joint([0.5, 0.3, -0.5, 0.2, 0.1, 0])
arm.set_gripper(0.0)
print('Done')
"
```

## 注意事项

- 机械臂 IP：连接前必须询问用户
- 默认以当前 skill 自带的 `pycarm` 为准
- 如果 `pycarm` 没有 git 元数据，升级方式应改为重新拉取该目录
- 如果升级了 `pycarm`，要同步确认示例仍然适用
- WebSocket 方式无需板卡 SDK
- 运动命令单位：关节为弧度，夹爪为米
- 执行完成后，输出编写的 Python 脚本内容供用户查看
