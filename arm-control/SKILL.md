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
git clone --recurse-submodules https://github.com/cvte-robotics/carm-openclaw.git
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
- 灵巧手位置：列表（具体单位取决于灵巧手型号）
- 笛卡尔位姿：`[x, y, z, qx, qy, qz, qw]`

## 关节索引

机械臂关节从 J1 到 J6，对应数组索引 `0` 到 `5`。

```python
arm.move_joint([0, 1.0, -1.5, 0, 0, 0])
# 对应 J1=0, J2=1.0, J3=-1.5, J4=0, J5=0, J6=0
```

## 末端执行器类型判断

当前机械臂的末端可能是**夹爪**或**灵巧手**，可通过 `hand_pos` 的返回值判断：

```python
pos = arm.hand_pos
if isinstance(pos, list) and len(pos) == 1:
    end_effector_type = "gripper"  # 夹爪（单自由度）
elif isinstance(pos, list) and len(pos) > 1:
    end_effector_type = "hand"     # 灵巧手（多自由度）
else:
    end_effector_type = "unknown"  # 未连接或无末端执行器
```

> **说明：** `hand_pos` 返回的列表长度为 1 时表示夹爪，大于 1 时表示灵巧手。未连接时返回空列表 `[]`。

## 状态监控

### 关节状态

```python
arm.joint_pos          # 实际关节位置 (rad)
arm.joint_vel          # 实际关节速度 (rad/s)
arm.joint_tau          # 实际关节力矩 (N·m)
arm.plan_joint_pos     # 规划关节位置
arm.plan_joint_vel     # 规划关节速度
arm.plan_joint_tau     # 规划关节力矩
```

### 笛卡尔状态

```python
arm.cart_pose          # 实际笛卡尔位姿 [x, y, z, qx, qy, qz, qw]
arm.plan_cart_pose     # 规划笛卡尔位姿
arm.cart_external_force  # 笛卡尔外力 [fx, fy, fz, tx, ty, tz]
arm.joint_external_tau   # 关节外力矩 [tau1~tauN]
```

### 末端执行器状态

```python
# 通用属性（自动识别夹爪/灵巧手）
arm.end_effector_state       # 状态 (-1未连接, 0未使能, 1正常)
arm.end_effector_pos         # 实际位置（列表）
arm.end_effector_vel         # 实际速度（列表）
arm.end_effector_tau         # 实际力矩（列表）
arm.plan_end_effector_pos    # 规划位置
arm.plan_end_effector_vel    # 规划速度
arm.plan_end_effector_tau    # 规划力矩

# 夹爪属性（eeff_type == gripper 时有效）
arm.gripper_state    # 状态
arm.gripper_pos      # 实际位置 (m)
arm.gripper_tau      # 实际力矩 (N)
arm.plan_gripper_pos # 规划位置

# 灵巧手属性（eeff_type == hand 时有效）
arm.hand_state       # 状态
arm.hand_pos         # 实际位置（列表）
arm.hand_vel         # 实际速度（列表）
arm.hand_tau         # 实际力矩（列表）
arm.plan_hand_pos    # 规划位置
arm.plan_hand_vel    # 规划速度
arm.plan_hand_tau    # 规划力矩
```

### 系统信息

```python
arm.version          # 控制器软件版本
arm.get_limits()     # 关节限位、最大速度、加速度
arm.tool_index       # 当前工具号
```

## 运动控制

```python
# 关节运动
arm.move_joint([0.5, 0.3, -0.5, 0.2, 0.1, 0])   # 点到点（推荐，快）
arm.move_line_joint([0.1, 0.2, 0.1, 0, 0, 0])    # 直线
arm.track_joint([...])                              # 实时跟踪

# 笛卡尔运动
arm.move_pose([0.3, 0, 0.4, 1, 0, 0, 0])          # 点到点
arm.move_line_pose([0.3, 0, 0.4, 1, 0, 0, 0])     # 直线插值
arm.move_flow_pose(                                  # 高精度雅可比
    target_pos=[0.3, 0, 0.4, 1, 0, 0, 0],
    line_theta_weight=0.5,
    accuracy=0.0001,
)
arm.track_pose([...])                                # 实时跟踪
```

## 末端执行器控制

### 夹爪

```python
arm.set_gripper(0.04)          # 开到4cm，力矩上限10N
arm.set_gripper(0.0)           # 关闭
arm.set_gripper(0.06, tau=20)  # 开到6cm，力矩上限20N
```

### 灵巧手

```python
# 灵巧手（多自由度，pos/tau/vel 均为列表）
arm.set_hand(pos=[0.5, 0.5, 0.5], tau=[5, 5, 5], vel=[1.0, 1.0, 1.0])
```

> **说明：** pos/tau/vel 三个参数必须是等长列表，长度由灵巧手自由度决定。

### 通用末端控制

```python
# 通用接口，dof 为自由度数
arm.set_end_effector(dof=1, pos=[0.04], vel=[0.0], tau=[10.0])  # 等同于 set_gripper
arm.set_end_effector(dof=6, pos=[...], vel=[...], tau=[...])    # 灵巧手
```

## 灵巧手实战示例

> 灵巧手 6 自由度索引：`[0]拇指关节1 [1]拇指关节2 [2]食指 [3]中指 [4]无名指 [5]小指`，pos/vel/tau 范围 0~255。

### 握手

```python
import time
DOF = 6; TAU = [128]*DOF; VEL = [128]*DOF

arm.set_hand(pos=[255]*DOF, tau=TAU, vel=VEL)                      # 张开
time.sleep(1)
arm.move_joint([-0.0017, 1.1161, -0.7147, -0.0528, -0.3523, 1.6752])  # 伸向前方
time.sleep(2)
arm.set_hand(pos=[127, 127, 64, 64, 64, 64], tau=TAU, vel=VEL)    # 握住
time.sleep(1)
for _ in range(3):                                                  # 晃动 3 次
    arm.move_joint([-0.0017, 1.1161, -0.5147, -0.0528, -0.3523, 1.6752]); time.sleep(0.5)  # 抬
    arm.move_joint([-0.0017, 1.1161, -0.9147, -0.0528, -0.3523, 1.6752]); time.sleep(0.5)  # 放
arm.set_hand(pos=[255]*DOF, tau=TAU, vel=VEL)                      # 松开
arm.move_joint([0, 0.998, -0.7155, -0.038, -0.3504, -0.0025])     # 回安全点
```

### 石头剪刀布

```python
import time, random
DOF = 6; TAU = [128]*DOF; VEL = [128]*DOF

ROCK     = [0,   0,   0,   0,   0,   0  ]  # 握拳
SCISSORS = [0,   0,   255, 255, 0,   0  ]  # 食指中指伸直
PAPER    = [255, 255, 255, 255, 255, 255]  # 全张开

arm.set_hand(pos=ROCK, tau=TAU, vel=VEL)
time.sleep(1)
choice = random.choice([("石头", ROCK), ("剪刀", SCISSORS), ("布", PAPER)])
print(f"我出：{choice[0]}")
arm.set_hand(pos=choice[1], tau=TAU, vel=VEL)
```

## 工具管理

```python
arm.set_tool_index(1)      # 切换到工具1
arm.tool_index              # 获取当前工具号
tool_coord = arm.get_tool_coordinate(0)  # 获取工具0的坐标系
```

## 就绪与安全

```python
arm.set_ready()                     # 就绪（清除错误+伺服使能+位置模式）
arm.set_ready(timeout_ms=5000)      # 带超时
arm.set_servo_enable(True)          # 伺服使能
arm.set_control_mode(1)             # 0-IDLE, 1-位置, 2-MIT, 3-拖动, 4-力位混合
arm.set_collision_config(True, 10)  # 碰撞检测（灵敏度0~2）
arm.set_speed_level(5.0)            # 速度等级 0~10
```

## 停止与恢复

```python
arm.stop()           # 暂停
arm.stop(1)          # 停止
arm.stop(2)          # 禁用
arm.stop(3)          # 急停
arm.stop_task(True)  # 立即停止当前任务
arm.recover()        # 恢复
arm.clean_carm_error()  # 清除控制器错误
```

## 轨迹与示教

```python
arm.trajectory_teach(True, "test.my_traj")  # 开始录制
arm.trajectory_teach(False)                  # 停止录制
arm.trajectory_recorder("test.my_traj")      # 复现轨迹
arm.check_teach()                            # 查看已录制列表
```

## 运动学与回调

```python
# 正运动学：关节角 → 笛卡尔位姿
pose = arm.forward_kine([0.1, 0.2, 0.3, 0, 0, 0])

# 逆运动学：笛卡尔位姿 → 关节角
joints = arm.inverse_kine(
    cart_pose=[0.3, 0, 0.4, 1, 0, 0, 0],
    ref_joints=[0, 0, 0, 0, 0, 0],
)

# 回调
arm.on_error(lambda err: print(f"Error {err['error']}: {err['errMsg']}"))
arm.on_task_finish(lambda key: print(f"Task {key} finished"))
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

# 判断末端执行器类型
pos = arm.hand_pos
if isinstance(pos, list) and len(pos) == 1:
    print("末端类型：夹爪")
    arm.set_gripper(0.06)
    arm.move_joint([0.5, 0.3, -0.5, 0.2, 0.1, 0])
    arm.set_gripper(0.0)
elif isinstance(pos, list) and len(pos) > 1:
    print(f"末端类型：灵巧手（{len(pos)}自由度）")
    arm.set_hand(pos=[0.5]*len(pos), tau=[5]*len(pos), vel=[1.0]*len(pos))
    arm.move_joint([0.5, 0.3, -0.5, 0.2, 0.1, 0])
    arm.set_hand(pos=[0.0]*len(pos), tau=[5]*len(pos), vel=[1.0]*len(pos))
else:
    print("末端类型：未知或未连接")

arm.move_line_pose([0.4, 0.2, 0.3, 1, 0, 0, 0])
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
