---
layout: default
title: ROS2 Autonomy Stack
---

<div class="project-header" style="margin-bottom: 30px;">
  <h1>ROS2 Autonomy Stack</h1>
  <p class="subtitle" style="color: #666; font-size: 1.2rem;">
    Frontier-based exploration, global planning, and low-level control for a TurtleBot3 running ROS2 Humble.
  </p>

  <div style="margin-top: 15px;">
    <a href="https://github.com/Tito004/section6" class="btn" style="background: #24292e; color: white; padding: 8px 15px; text-decoration: none; border-radius: 5px; margin-right: 10px;">
      View Code on GitHub
    </a>
  </div>
</div>

## 1. System Architecture

This project implements a ROS2 autonomy stack on a simulated TurtleBot3, integrating exploration, global planning, and low-level control into separate nodes that communicate over standard ROS2 topics. Frontier-based exploration drives the robot toward boundaries between known free space and unknown regions in an occupancy grid map, a common strategy for autonomous mapping tasks in ROS.

The stack is built on top of `asl_tb3_lib` utilities for TurtleBot3, using stochastic occupancy grids, navigation primitives, and controller base classes from the AA174/274 autonomy labs. All logic is written in Python and targets ROS2 Humble, making it easy to extend, visualize in RViz, and deploy in Gazebo-based simulations.

---

## 2. Demo Video

<div class="video-holder" style="position: relative; width: 100%; height: 0; padding-bottom: 56.25%; margin-bottom: 20px;">
  <!-- Replace VIDEO_ID with your YouTube ID, or swap this iframe for a local .mp4 <video> tag -->
  <iframe
    src="https://www.youtube.com/embed/6-0zsNpcgs0?si=4d6usGVvHEePQJLa"
    title="Frontier Exploration Demo"
    style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"
    frameborder="0"
    allowfullscreen>
  </iframe>
</div>

<p style="color: #666;">
The video shows the TurtleBot3 performing frontier-based exploration in simulation, building a map in real time while planning and executing paths to new frontiers.
</p>

---

## 3. Frontier Exploration Node

The <strong>Frontier Explorer</strong> node (<code>frontier_explorer.py</code>) subscribes to the global occupancy grid (<code>/map</code>) and robot state (<code>/state</code>), storing the map in a <code>StochOccupancyGrid2D</code> that tracks probabilities for free, occupied, and unknown cells. It also listens to a navigation success flag and a stop-sign detection topic, enabling it to coordinate exploration episodes and pause when the perception system detects a stop condition.

To detect frontiers, the node constructs masks for unknown, free, and occupied space from the probabilistic grid and convolves each mask with a 13×13 uniform kernel using <code>scipy.signal.convolve2d</code>, efficiently aggregating local neighborhood information. A cell is marked as a valid frontier when the convolved unknown probability is at least 0.2, the free probability is at least 0.3, and the occupied probability is exactly zero, which captures boundaries between navigable space and unexplored regions while avoiding obstacles.



Once the frontier set is computed, the node converts candidate cells back into world coordinates and selects the frontier with minimum Euclidean distance to the robot’s current pose as the next exploration goal. This goal is published on a navigation command topic that the global planner consumes, and the node tracks a simple internal state machine (waiting for map, exploring, waiting for navigation success) to decide when to send new goals.

The explorer also integrates a stop-sign flag and a timer: when a stop event is received, it records the current time, stops sending new goals, and waits a fixed duration before resuming exploration, ensuring safe behavior around detected signs. A ROS2 parameter named <code>active</code> allows the entire frontier exploration behavior to be toggled on or off at runtime without changing code.

---

## 4. Global Planner & Trajectory Tracking

The <strong>Navigator</strong> node (<code>navigator.py</code>) extends <code>BaseNavigator</code> from <code>asl_tb3_lib</code> and is responsible for taking high-level navigation goals and turning them into dynamically feasible trajectories for the robot. It subscribes to robot state and occupancy grid topics, and it receives target positions from the Frontier Explorer via a navigation command interface.

Inside <code>navigator.py</code>, an <code>AStar</code> class implements a grid-based motion planner over the stochastic occupancy grid, discretizing the state space and using a heuristic (such as Euclidean distance) to guide the search from the robot’s current cell to the goal cell while avoiding occupied regions. The planner uses the underlying occupancy grid to reject states with occupancy probability above a threshold, ensuring the resulting path remains in free space.

Once a sequence of waypoints is found, the navigator fits cubic B-splines to the \(x\) and \(y\) coordinates using <code>scipy.interpolate.splrep</code> and evaluates these splines via <code>splev</code> to obtain smoothed trajectories and their derivatives. The node then applies a tracking controller that uses these derivatives to compute desired accelerations and converts them into linear and angular velocity commands for the differential-drive robot.

The tracking law combines feedforward terms from the spline (desired velocity and acceleration) with feedback terms on position and velocity errors, using gains such as \(k_{px}\), \(k_{py}\), \(k_{dx}\), and \(k_{dy}\) to stabilize the motion. The final control command is packaged into a <code>TurtleBotControl</code> message that sets the robot’s forward velocity and yaw rate, and the node includes logic to avoid re-planning excessively when the previous command’s velocity is essentially zero.

---

## 5. Heading & Perception Controllers

The <strong>Heading Controller</strong> node (<code>heading_controller.py</code>) is built on <code>BaseHeadingController</code> and is responsible for aligning the robot’s orientation with a desired heading derived from the navigation stack. It declares a proportional gain parameter <code>kp</code> at startup and provides a property interface to read the current value, allowing real-time tuning via ROS2’s parameter server without restarting the node.

Its control law computes the heading error as the wrapped difference between the target yaw and the current yaw using <code>wrap_angle</code>, and the controller outputs an angular velocity command
\(\omega = k_p \cdot \text{wrap\_angle}(\theta_{\text{goal}} - \theta_{\text{current}})\).
By relying on angle wrapping, the controller avoids discontinuities at \(\pm \pi\) and ensures the robot always turns along the shortest rotational direction toward the goal orientation.

The <strong>Perception Controller</strong> node (<code>perception_controller.py</code>) coordinates with the vision or perception pipeline through Boolean detection topics and a simple time-based state machine. It declares an <code>active</code> parameter to represent whether autonomous motion is allowed and keeps track of the last time this parameter changed, enabling it to enforce a cooldown window before re-activating motion after a stop event.

In its <code>compute_control</code> method, the node initializes a <code>TurtleBotControl</code> message and, when active, sets a nominal angular velocity (for example, a slow rotation) to help with perception tasks like scanning for signs or landmarks. When the <code>active</code> flag is false, it sets the angular velocity to zero, checks whether the cooldown period has elapsed, and, if so, flips the <code>active</code> parameter back to true so that the autonomy stack can resume motion.

---

## 6. Results & Takeaways

In simulation, the combined stack is able to autonomously explore unknown environments by repeatedly selecting frontiers, planning collision-free paths, and tracking them while visualizing progress in RViz. The integration of a stop-sign flag and perception-aware pause logic shows how higher-level semantics from vision can be cleanly integrated into a navigation pipeline using ROS2 topics and parameters.

This project demonstrates that decomposing autonomy into loosely coupled ROS2 nodes—frontier exploration, planning, heading control, and perception coordination—makes the system easier to debug, extend, and reuse across different tasks and environments. It also mirrors common patterns in the broader ROS ecosystem, where frontier-based exploration and occupancy-grid planners are standard building blocks for mapping and navigation on robots like TurtleBot3.

