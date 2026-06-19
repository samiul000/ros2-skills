---
name: ros2-engineering-skills
description: >
  TRIGGER when the user: writes or reviews ROS 2 nodes (rclcpp/rclpy), creates packages
  (colcon/ament), edits launch files (.launch.py), configures QoS or DDS, writes URDF/xacro,
  implements ros2_control hardware interfaces or controllers, sets up Nav2/MoveIt 2 pipelines,
  processes sensor data (camera/LiDAR/PCL), works with Gazebo/Isaac Sim, configures SROS2
  security, develops micro-ROS firmware, manages multi-robot fleets (Open-RMF), debugs with
  ros2 doctor/rosbag2, deploys via Docker/cross-compilation, or migrates from ROS 1.
  DO NOT TRIGGER for general C++/Python questions unrelated to ROS 2, non-robotics middleware,
  or web/mobile development tasks.
context: fork
classification: capability
category: api-reference
version: 1.1.0
deprecation-risk: medium
evals: SKILL.evals.yaml
hooks:
  PreToolUse:
    - matcher: "Edit|Write|MultiEdit|Bash"
      hooks:
        - type: command
          command: "python3 ${CLAUDE_PLUGIN_ROOT}/scripts/skill_validate_hook.py"
          timeout: 10000
  Stop:
    - hooks:
        - type: command
          command: "python3 ${CLAUDE_PLUGIN_ROOT}/scripts/skill_stop_hook.py"
          timeout: 15000
---

# ROS 2 Engineering Skills

> **Single responsibility:** This skill is an **API reference & code template guide**
> for ROS 2 development. It tells you _how to use ROS 2 APIs correctly_ and
> _what mistakes to avoid_. It does NOT do CI/CD orchestration, incident response,
> data analysis, or deployment automation — those are separate skill categories.

A progressive-disclosure skill for ROS 2 development — from first workspace to
production fleet deployment. Each section below gives you the essential decision
framework; detailed patterns, code templates, and anti-patterns live in the
`references/` directory. Read the relevant reference file before writing code.

## How to use this skill

**Progressive disclosure — do NOT read everything at once.**
This skill is structured in layers. Only load what you need for the current task:

1. **This file (SKILL.md)** — always loaded. Contains decision routing, core
   principles, pitfalls, and anti-patterns. Sufficient for answering quick
   questions and making architectural decisions.
2. **`references/*.md`** — load on demand. Use the Decision Router below to
   pick the 1–2 files relevant to the user's current task. Do NOT read all 20
   reference files — that wastes context and causes confusion.
3. **`scripts/`** — run only when the user needs code generation, QoS checking,
   or launch validation. These are tools, not reading material.

**Steps:**

1. If `.skill-runs.log` exists in the workspace, read the last few lines to
   understand what was done and what issues occurred in previous sessions.
2. Identify what the user is building (see Decision Router below).
3. Read **only** the matching `references/*.md` file(s) for detailed guidance.
4. Check the **AI pitfalls** table before generating any code.
5. Apply the Core Engineering Principles in every artifact you produce.
6. When multiple domains intersect (e.g. Nav2 + ros2_control), read both files
   but favor safety > determinism > simplicity when recommendations conflict.

**Execution log:** The Stop hook automatically appends a session summary to
`.skill-runs.log` in the workspace. This lets you see what was validated last
time and what issues were found — check it to avoid repeating past mistakes.

## Decision router

| User is doing...                                                                                                                                                            | Read                                 | Notes                                                                                                                                                                   |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Creating a workspace, package, or build config — `colcon build`, `ament_cmake`, `ament_python`, `package.xml`, `rosdep`, overlay                                            | `references/workspace-build.md`      | Skip if question is only about node code. Combine with deployment.md if Dockerizing the workspace.                                                                      |
| Writing nodes, executors, callback groups — `rclcpp::Node`, `rclpy.node`, `spin`, `MultiThreadedExecutor`, `CallbackGroup`, `create_timer`, `create_subscription`           | `references/nodes-executors.md`      | Skip if question is purely about QoS mismatches (→ communication.md) or lifecycle state machine (→ lifecycle-components.md).                                            |
| Topics, services, actions, custom interfaces, QoS — `create_publisher`, `create_subscription`, `QoSProfile`, `rclcpp::QoS`, `qos_profile_sensor_data`, `send_goal`          | `references/communication.md`        | Skip if question is only about executor threading (→ nodes-executors.md) or bag replay QoS (→ debugging.md). Combine with security.md if data crosses a trust boundary. |
| Lifecycle nodes, component loading, composition — `LifecycleNode`, `on_configure`, `on_activate`, `ComponentManager`, `load_component`, `ComposableNode`                    | `references/lifecycle-components.md` | Always combine with hardware-interface.md when writing hardware drivers. Combine with launch-system.md for composition launch syntax.                                   |
| Launch files, conditional logic, event handlers — `generate_launch_description`, `IncludeLaunchDescription`, `DeclareLaunchArgument`, `IfCondition`, `RegisterEventHandler` | `references/launch-system.md`        | Skip if question is about package build config (→ workspace-build.md). Combine with lifecycle-components.md for component container launch patterns.                    |
| tf2, URDF, xacro, robot_state_publisher — `TransformBroadcaster`, `lookup_transform`, `xacro:macro`, `joint`, `link`, `robot_state_publisher`                               | `references/tf2-urdf.md`             | Skip if question is about the `<ros2_control>` URDF tag specifically (→ hardware-interface.md). Combine with navigation.md for map→odom→base_link chain.                |
| ros2_control, hardware interfaces, controllers — `SystemInterface`, `ControllerInterface`, `on_configure`, `on_activate`, `controller_manager`, EtherCAT                    | `references/hardware-interface.md`   | Always combine with lifecycle-components.md. Combine with realtime.md if control loop is ≥100 Hz. Combine with tf2-urdf.md if editing the URDF ros2_control tag.        |
| Real-time constraints, PREEMPT_RT, memory, jitter — `mlockall`, `SCHED_FIFO`, CPU isolation, `realtime_tools`, `RealtimeBuffer`, latency measurement                        | `references/realtime.md`             | Combine with hardware-interface.md when hardening a control loop. Skip if question is only about executor threading without RT requirements (→ nodes-executors.md).     |
| Nav2, SLAM, costmaps, behavior trees — `nav2_params.yaml`, `BehaviorTree`, `bt_navigator`, `AMCL`, `costmap_2d`, `DWBLocalPlanner`, recovery behaviors                      | `references/navigation.md`           | Combine with tf2-urdf.md for map→odom→base_link frame chain. Combine with simulation.md if testing in Gazebo Sim.                                                       |
| MoveIt 2, planning scene, grasp pipelines — `MoveGroupInterface`, `MTC`, `PlanningScene`, `CollisionObject`, `servo`, `MoveItVisualTools`                                   | `references/manipulation.md`         | Combine with hardware-interface.md for ros2_control integration. Combine with tf2-urdf.md for planning frame setup.                                                     |
| Camera, LiDAR, PCL, cv_bridge, depth processing — `image_transport`, `cv_bridge`, `pcl_ros`, `PointCloud2`, `CameraInfo`, Isaac ROS                                         | `references/perception.md`           | Combine with tf2-urdf.md for sensor frame transforms. Combine with simulation.md if sensor data comes from Gazebo Sim.                                                  |
| Unit tests, integration tests, launch_testing, CI — `gtest`, `pytest`, `launch_testing`, `industrial_ci`, parameter testing, GitHub Actions                                 | `references/testing.md`              | Combine with workspace-build.md for colcon test configuration. Skip if question is only about runtime debugging (→ debugging.md).                                       |
| ros2 doctor, tracing, profiling, rosbag2, CLI cheat sheet — `ros2 topic echo`, `ros2 bag`, `MCAP`, `ros2 doctor`, Foxglove, `lttng`                                         | `references/debugging.md`            | Skip if question is about QoS profile design (→ communication.md). Combine with realtime.md for RT jitter profiling.                                                    |
| Docker, cross-compile, fleet deployment, OTA — `Dockerfile`, `qemu`, `colcon bundle`, Zenoh router, over-the-air update                                                     | `references/deployment.md`           | Combine with security.md for production hardening. Combine with testing.md for CI/CD pipeline setup.                                                                    |
| Gazebo Sim (Ionic/Harmonic), Isaac Sim, sim-to-real, use_sim_time — `gz_ros2_control`, `ros_gz_bridge`, `gz sim`, `use_sim_time`, sensor plugins, sim-to-real gap           | `references/simulation.md`           | Combine with navigation.md if testing a Nav2 stack. Combine with perception.md if working with simulated sensors.                                                       |
| SROS2, DDS security, certificates, supply chain — `ros2 security`, keystore, enclave, `ROS_SECURITY_STRATEGY`, DDS security plugins                                         | `references/security.md`             | Combine with deployment.md for production fleet hardening. Combine with multi-robot.md if securing a multi-robot system.                                                |
| micro-ROS, MCU/RTOS, XRCE-DDS, rclc — `rclc_node_init`, `rclc_executor`, XRCE-DDS agent, `micro_ros_arduino`, ESP32/STM32/RP2040                                            | `references/micro-ros.md`            | Do not combine with nodes-executors.md — rclc API is entirely separate from rclcpp/rclpy. Combine with realtime.md if running on an RTOS with RT constraints.           |
| Multi-robot fleet, Open-RMF, DDS discovery scale — `ROS_DOMAIN_ID`, namespace strategy, `frame_prefix`, Open-RMF, fleet manager                                             | `references/multi-robot.md`          | Combine with security.md for cross-robot trust boundaries. Combine with deployment.md for fleet-scale infrastructure.                                                   |
| Message types, units, covariance, frame conventions — `std_msgs`, `sensor_msgs`, `geometry_msgs`, covariance layout, REP-103, REP-105                                       | `references/message-types.md`        | Combine with communication.md if also designing the transport (QoS, topic names). Combine with tf2-urdf.md for frame convention questions.                              |
| ROS 1 migration, ros1_bridge, hybrid operation — `ros1_bridge`, `rospy`→`rclpy`, `roscpp`→`rclcpp`, `.msg` format changes                                                   | `references/migration-ros1.md`       | After migration planning, read the target-domain reference file (e.g. nodes-executors.md for node rewrites, communication.md for topic/service changes).                |

**Cross-cutting concerns:** Apply security, error handling, and QoS whenever data crosses a trust boundary, a node owns hardware, or reliability matters.

## Core engineering principles

These apply to every ROS 2 artifact you produce, regardless of domain.

### 1. Distro awareness

<!-- LAST_UPDATED: 2026-03-30 | NEXT_REVIEW: 2026-09-30 | Verify at https://docs.ros.org/en/rolling/Releases.html -->

> **Staleness warning:** The table below was last verified on **2026-03-30**.
> If the current date is more than 6 months past that, re-verify EOL dates and
> feature support against https://docs.ros.org/en/rolling/Releases.html before
> relying on this table. When you update it, change both `LAST_UPDATED` and
> `NEXT_REVIEW` comments above.

Always ask which ROS 2 distribution the user targets. Key differences:

| Feature                  | Foxy (**EOL**)       | Humble (LTS)      | Jazzy (LTS)  | Kilted (non-LTS) | Rolling          |
| ------------------------ | -------------------- | ----------------- | ------------ | ---------------- | ---------------- |
| EOL                      | Jun 2023 (**ended**) | May 2027          | May 2029     | Nov 2025         | Rolling          |
| Ubuntu                   | 20.04                | 22.04             | 24.04        | 24.04            | Latest           |
| Default DDS              | Fast DDS             | Fast DDS          | Fast DDS     | Fast DDS         | Fast DDS         |
| Zenoh support            | —                    | —                 | —            | Tier 1           | Tier 1           |
| Type description support | No                   | No                | Yes          | Yes              | Yes              |
| Service introspection    | No                   | No                | Yes          | Yes              | Yes              |
| EventsExecutor           | No                   | No                | Experimental | Stable (+ rclpy) | Stable (+ rclpy) |
| Default bag format       | sqlite3              | sqlite3           | MCAP         | MCAP             | MCAP             |
| ros2_control interface   | N/A (separate)       | 2.x               | 4.x          | 4.x              | Latest           |
| CMake recommendation     | ament_target_deps    | ament_target_deps | either       | target_link_libs | target_link_libs |

When the user does not specify, default to the latest LTS (Jazzy).
Pin the exact distro in Dockerfile, CI, and documentation so builds are reproducible.

### 2. C++ vs Python decision

Choose the language based on the node's role, not personal preference.

**Use rclcpp (C++) when:**

- The node sits in a control loop running ≥100 Hz
- Deterministic memory allocation matters (real-time path)
- The node is a hardware driver or controller plugin
- Intra-process zero-copy communication is required

**Use rclpy (Python) when:**

- The node is orchestration, monitoring, or parameter management
- Rapid prototyping with frequent iteration
- Heavy use of ML frameworks (PyTorch, TensorFlow) that are Python-native
- The node does not sit in a latency-critical path

**Mixed stacks are normal.** A typical robot has C++ drivers/controllers and Python
orchestration/monitoring. Note: `component_container` (composition) only loads
C++ components via pluginlib. Python nodes run as separate processes, but can
share a launch file and communicate via zero-overhead intra-host DDS.

**Intra-process communication** works for any nodes sharing a process — not only
composable components. Any nodes instantiated in the same process with
`use_intra_process_comms(true)` can use zero-copy transfer.

### 3. Package structure conventions

Every package should follow this layout. Consistency across a workspace reduces
onboarding time and makes CI scripts portable.

```text
my_package/
├── CMakeLists.txt
├── package.xml
├── config/
│   └── params.yaml
├── launch/
│   └── bringup.launch.py
├── include/my_package/     # public headers (C++)
├── src/                    # C++ source files
├── my_package/             # Python modules (if ament_python or mixed)
├── test/                   # gtest, pytest, launch_testing
├── urdf/                   # URDF/xacro (if applicable)
├── msg/ srv/ action/       # custom interfaces (dedicated _interfaces package preferred)
└── README.md
```

Separate interface definitions into a `*_interfaces` package so downstream
packages can depend on interfaces without pulling in implementation.

### 4. Parameter discipline

- Declare every parameter with a type, description, range, and default
  in the node constructor — never use undeclared parameters.
- Use `ParameterDescriptor` with `FloatingPointRange` or `IntegerRange`
  for numeric bounds. The parameter server rejects out-of-range values at set time.
- Group related parameters under a namespace prefix:
  `controller.kp`, `controller.ki`, `controller.kd`.
- Load defaults from a `config/params.yaml`; allow launch-time overrides.
- For dynamic reconfiguration, register a `set_parameters_callback` and
  validate new values atomically before accepting.

### 5. Error handling philosophy

- Nodes must not silently swallow errors. Log at the appropriate severity,
  then take a safe action (stop motion, request help, transition to error state).
- Prefer lifecycle node error transitions over ad-hoc boolean flags.
- When calling a service, always handle the "service not available" and
  "future timed out" cases explicitly.
- For hardware drivers, distinguish transient errors (retry with backoff)
  from fatal errors (transition to `FINALIZED` and alert the operator).

### 6. Quality of Service defaults

Start from these profiles and adjust per use case:

| Use case         | Reliability | Durability      | History   | Depth | Deadline | Lifespan |
| ---------------- | ----------- | --------------- | --------- | ----- | -------- | -------- |
| Sensor stream    | BEST_EFFORT | VOLATILE        | KEEP_LAST | 5     | —        | —        |
| Command velocity | RELIABLE    | VOLATILE        | KEEP_LAST | 1     | 100 ms   | 200 ms   |
| Map (latched)    | RELIABLE    | TRANSIENT_LOCAL | KEEP_LAST | 1     | —        | —        |
| Diagnostics      | RELIABLE    | VOLATILE        | KEEP_LAST | 10    | —        | —        |
| Parameter events | RELIABLE    | VOLATILE        | KEEP_LAST | 1000  | —        | —        |
| Action feedback  | RELIABLE    | VOLATILE        | KEEP_LAST | 1     | —        | —        |
| Safety heartbeat | RELIABLE    | VOLATILE        | KEEP_LAST | 1     | 500 ms   | 1 s      |

QoS mismatches are the #1 cause of "I published but nobody receives."
Always check compatibility with `ros2 topic info -v` when debugging.

**DEADLINE and LIFESPAN** are critical for safety-critical systems. DEADLINE fires an
event when no message arrives within the specified period (detect stale data). LIFESPAN
discards messages older than the specified duration before delivery (prevent acting on
stale data). See `references/communication.md` section 9 for full API and examples.

### 7. Naming conventions

| Entity    | Convention                  | Example                        |
| --------- | --------------------------- | ------------------------------ |
| Package   | `snake_case`                | `arm_controller`               |
| Node      | `snake_case`                | `joint_state_broadcaster`      |
| Topic     | `/snake_case` with ns       | `/arm/joint_states`            |
| Service   | `/snake_case`               | `/arm/set_mode`                |
| Action    | `/snake_case`               | `/arm/follow_joint_trajectory` |
| Parameter | `snake_case` with dot ns    | `controller.publish_rate`      |
| Frame     | `snake_case`                | `base_link`, `camera_optical`  |
| Interface | `PascalCase.msg/srv/action` | `JointState.msg`               |

### 8. Thread safety and callbacks

- A `MutuallyExclusiveCallbackGroup` serializes its callbacks — safe for
  shared state without locks, but limits throughput.
- A `ReentrantCallbackGroup` allows parallel execution — you must protect
  shared state with `std::mutex` (C++) or `threading.Lock` (Python).
- **Calling a service from a callback:** The service client **must** be in a
  separate `MutuallyExclusiveCallbackGroup` from the calling callback. Otherwise
  the executor deadlocks — the callback waits for the response while the executor
  cannot deliver it. Always use `async_send_request` with a response callback;
  never use `spin_until_future_complete` inside an executor callback.
- Never do blocking work (file I/O, long computation, `sleep`) inside a
  timer or subscription callback on the default executor. Offload to a
  dedicated thread or use a `MultiThreadedExecutor` with a reentrant group.
- In rclcpp, prefer `std::shared_ptr<const MessageT>` in subscription
  callbacks to avoid unnecessary copies and enable zero-copy intra-process.

### 9. Lifecycle-first design

Default to lifecycle (managed) nodes for anything that owns resources:
hardware drivers, sensor pipelines, planners, controllers.

```text

States: Unconfigured → (on_configure) → Inactive → (on_activate) → Active → (on_deactivate) → Inactive → (on_cleanup) → Unconfigured → (on_shutdown) → Finalized

```

This gives the system manager (launch file, orchestrator, or operator) explicit
control over when resources are allocated, when the node starts processing,
and how it shuts down. It also makes error recovery predictable.

### 10. Build and CI hygiene

- Use `colcon build --cmake-args -DCMAKE_BUILD_TYPE=RelWithDebInfo` for
  development; `Release` for deployment.
- Enable `-Wall -Wextra -Wpedantic` and treat warnings as errors in CI.
- Run `colcon test` with `--event-handlers console_cohesion+` so test
  output groups by package.
- Pin rosdep keys in `rosdep.yaml` for reproducible dependency resolution.
- Cache `/opt/ros/`, `.ccache/`, and `build/`/`install/` in CI to cut build
  times by 60–80%.

## Common anti-patterns

| Anti-pattern                                              | Why it hurts                                         | Fix                                                                                           |
| --------------------------------------------------------- | ---------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| Global variables for node state                           | Breaks composition, untestable                       | Store state as class members                                                                  |
| `spin()` in `main()` for multi-node processes             | Starves other nodes                                  | Use `MultiThreadedExecutor` or component composition                                          |
| Hardcoded topic names                                     | Breaks reuse across robots                           | Use relative names + namespace remapping                                                      |
| `KEEP_ALL` history with no bound                          | Memory grows unbounded on slow subscribers           | Use `KEEP_LAST` with explicit depth                                                           |
| Using `time.sleep()` / `std::this_thread::sleep_for`      | Blocks the executor thread                           | Use `create_wall_timer` or a dedicated thread                                                 |
| Monolithic launch file for everything                     | Unmanageable past 10 nodes                           | Compose launch files with `IncludeLaunchDescription`                                          |
| Skipping `package.xml` dependencies                       | Builds locally, breaks CI and Docker                 | Declare every dependency explicitly                                                           |
| Publishing in constructor                                 | Subscribers may not be ready, messages lost          | Publish in `on_activate` or after a short timer                                               |
| Ignoring QoS compatibility                                | Silent communication failure                         | Match publisher/subscriber QoS or check with `ros2 topic info -v`                             |
| Creating timers/subs in callbacks                         | Resource leak, unpredictable behavior                | Create all entities in constructor or `on_configure`                                          |
| Synchronous service call in callback                      | Deadlocks the executor thread                        | Use `async_send_request` with a callback or dedicated thread                                  |
| Service client in same callback group as caller           | Deadlocks even with async in `MultiThreadedExecutor` | Put service client in a separate `MutuallyExclusiveCallbackGroup`                             |
| No safe command on shutdown                               | Motors hold last velocity after node exits           | Send zero-velocity in `on_deactivate` AND destructor (see `references/hardware-interface.md`) |
| Dynamic subscriptions with `StaticSingleThreadedExecutor` | New subs are never picked up after `spin()`          | Use `SingleThreadedExecutor` or `MultiThreadedExecutor` for dynamic entities                  |
| CPU frequency governor left on `powersave`/`ondemand`     | 10-100 ms latency spikes in RT path                  | Set `performance` governor, disable turbo boost (see `references/realtime.md`)                |

## AI pitfalls — traps this skill has learned from

These are mistakes AI agents repeatedly make when generating ROS 2 code.
**Add a new line here every time a failure is discovered in practice.**

| #   | Pitfall                                                            | What goes wrong                                                                                    | Correct approach                                                                                                         |
| --- | ------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| 1   | Using `spin_until_future_complete` inside a callback               | Deadlocks the executor — the callback blocks waiting for a response that can never be delivered    | Use `async_send_request` with a response callback; put the service client in a separate `MutuallyExclusiveCallbackGroup` |
| 2   | Generating Foxy-era API for Jazzy/Kilted                           | `node_executable` is deprecated, `export_state_interfaces()` signature changed in ros2_control 4.x | Always check the distro feature matrix above before generating code                                                      |
| 3   | Omitting QoS in publisher/subscriber creation                      | Defaults silently mismatch — publisher sends but subscriber receives nothing                       | Always specify QoS explicitly; use the QoS defaults table in Principle 6                                                 |
| 4   | Creating a `msg/` directory inside a non-interfaces package        | Builds locally but fails in CI — interface packages need `rosidl_generate_interfaces`              | Put messages in a dedicated `*_interfaces` package                                                                       |
| 5   | Hardcoding `/opt/ros/humble/` paths in launch files                | Breaks on any other distro or install prefix                                                       | Use `FindPackageShare`, `PathJoinSubstitution`, or environment substitutions                                             |
| 6   | Forgetting `<depend>` tags in `package.xml`                        | `colcon build` works in overlay but `rosdep install` and Docker builds fail                        | Declare every `find_package()` / `import` as `<depend>` in package.xml                                                   |
| 7   | Using `time.sleep()` for rate control in rclpy                     | Blocks the executor thread; timers and subscriptions stop firing                                   | Use `create_timer()` or `Rate` with a `MultiThreadedExecutor`                                                            |
| 8   | Not sending zero-velocity on deactivate/shutdown                   | Robot holds last commanded velocity when the node crashes                                          | Send zero-command in both `on_deactivate` and the destructor                                                             |
| 9   | Mixing `ament_target_dependencies()` and `target_link_libraries()` | Kilted deprecated `ament_target_dependencies` — mixing causes link errors                          | Use `target_link_libraries()` with modern CMake targets for Kilted+; `ament_target_dependencies()` for Humble/Jazzy      |
| 10  | Generating `rospy` / `roscpp` code instead of `rclpy` / `rclcpp`   | ROS 1 patterns in a ROS 2 context — nothing compiles                                               | This skill is ROS 2 only — always use `rclpy`/`rclcpp` APIs                                                              |
| 11  | Ignoring `use_sim_time` parameter in simulation                    | Real clock diverges from Gazebo clock — tf lookups fail, controllers drift                         | Set `use_sim_time:=true` in launch and pass `--clock` to `ros2 bag play`                                                 |
| 12  | Publishing before subscribers connect (no TRANSIENT_LOCAL)         | First N messages lost — map, URDF, or initial config never received                                | Use `TRANSIENT_LOCAL` durability for latched-style data, or publish in `on_activate` with a startup delay                |

> **Maintenance rule:** When you encounter a new AI failure pattern while using this
> skill, append it to this table with the next sequential number. The pitfall list
> is the single most valuable section for preventing repeated mistakes.

## Distro-specific migration notes

<!-- LAST_UPDATED: 2026-03-30 — Keep in sync with the distro table in Principle 1. -->

When upgrading between distributions, check these breaking changes first:

**Foxy → Humble:**

- Complete API overhaul. Foxy packages require significant rework.
- `ros2_control` was not bundled in Foxy — must be built separately.
- Lifecycle node API stabilized in Humble.
- Action server/client API changed significantly.

**Humble → Jazzy:**

- `ros2_control` API changed from 2.x to 4.x — `export_state_interfaces()` and
  `export_command_interfaces()` are now auto-generated by the framework. Manual
  overrides use `on_export_state_interfaces()`. See `references/hardware-interface.md`.
- Handle `get_value()` deprecated → use `get_optional<T>()` on `LoanedStateInterface` /
  `LoanedCommandInterface` (controller side). Hardware interfaces use `set_state()` /
  `get_state()` / `set_command()` / `get_command()` helpers with fully qualified names.
- All joints in `<ros2_control>` tag must exist in the URDF.
- Controller parameter loading changed — use `--param-file` with spawner.
- Default bag format changed from sqlite3 to **MCAP**. Use `storage_id='mcap'`.
- Default middleware changed internal config paths. Regenerate DDS profiles.
- `nav2_params.yaml` schema changes — `recoveries_server` renamed to `behavior_server`.
- `ROS_AUTOMATIC_DISCOVERY_RANGE` replaces `ROS_LOCALHOST_ONLY` (values: `LOCALHOST`,
  `SUBNET`, `OFF`, `SYSTEM_DEFAULT`).
- `launch_ros` actions have new parameter handling — test launch files explicitly.

**Jazzy → Kilted (non-LTS):**

- **Zenoh promoted to Tier 1 middleware** — `rmw_zenoh` is production-ready.
  Install: `sudo apt install ros-kilted-rmw-zenoh-cpp`, set
  `RMW_IMPLEMENTATION=rmw_zenoh_cpp`. Supports router/peer/client modes.
- **EventsExecutor graduated from experimental** — available in `rclcpp::executors`
  (no `experimental` namespace). Also ported to rclpy.
- **`ament_target_dependencies()` deprecated** — use `target_link_libraries()` with
  modern CMake targets (e.g. `rclcpp::rclcpp`, `std_msgs::std_msgs__rosidl_typesupport_cpp`).
- Multi-bag replay support in `ros2 bag play`.
- Gazebo **Ionic** is the paired simulator (Harmonic was Jazzy; Ionic is the Kilted pairing).

**ROS 1 → ROS 2:**

- See `references/migration-ros1.md` for a step-by-step strategy.

## Quick reference — ros2 CLI

See **`references/debugging.md` §10 "Quick CLI reference"** for the full
command cheat sheet (workspace, introspection, ros2_control, debugging,
lifecycle). Kept out of this always-loaded file to preserve context budget.
