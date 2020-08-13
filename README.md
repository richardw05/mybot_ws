# Simulating the ROS Navigation Stack (part 3)

This tutorial updates http://www.moorerobots.com/blog/post/3 to run on ROS Melodic (Ubuntu 18.04)

## Mapping and Navigation: Turtlebot3 (Optional)
In ROS melodic, turtlebot3 has replaced the older turtlebot packages.  This
version of the tutorial takes you through map building/making with turtlebot3,
then with our `mybot` robot.

### Setup
1. First, install the turtlebot3 packages:
```
sudo apt install ros-melodic-turtlebot3_/*
sudo apt install ros-melodic-slam-gmapping ros-melodic-gmapping ros-melodic-openslam-gmapping
```

### Creating the map
I had to change the turtlebot3 gazebo sim to run with no gui (slow computer).  If you don't need to do this, you can specify `gui:=true` or not change the `turtlebot3_world.launch` file.

1. Copy the file
    ```
        cp `rospack find turtlebot3_gazebo`/launch/turtlebot3_world.launch `rospack find mybot_gazebo`/launch
    ```
2. Using your [favorite text editor](http://vim.org), edit `turtlebot3_world.launch` and add
`<arg name="gui" default="false"/>` below launch.  Now change
`  <arg name="gui" value="true"/>`
to
`  <arg name="gui" value="$(arg gui)"/>`.  

2.  In Terminal 1, start up gazebo

```
export TURTLEBOT3_MODEL="waffle"
roslaunch mybot_gazebo turtlebot3_world.launch gui:=false
```

3. In Terminal 2, start map building (also starts rviz)
```
export TURTLEBOT3_MODEL="waffle"
roslaunch turtlebot3_slam turtlebot3_slam.launch slam_methods:=gmapping
```

4. In Terminal 3, start teleop
```
export TURTLEBOT3_MODEL=waffle
roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
```

5. drive around till you have a reasonable map

### Saving the map
1. In Terminal 4, save the map to some file path
```
rosrun map_server map_saver -f /tmp/test_map
```
Close all previous terminals.

### Loading the map
Now, we'll use the created map to localize while moving around.  Once
loaded, we'll use rviz to set navigation waypoints and the robot should move
autonomously.

1. Install local planner (if needed)
```
sudo apt install ros-melodic-dwa-local-planner
```

2. In Terminal 1, launch the Gazebo world
```
roslaunch mybot_gazebo turtlebot3_world.launch
```

3. In Terminal 2, start map building (will start rviz too)
```
roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:=/tmp/test_map.yaml
```

4. In rviz, estimate initial pose - click `2D Pose Estimate` and click the approximate location of the robot on the map, and drag to indicate the direction.

5. In Terminal 3, start teleop and move the robot around.  The estimated positions should converge on the true position pretty quickly.
```
roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
```

6. In rviz, send a few `2D Navigation Goals` (click on ``2D Nav Goal`` and click/drag to set position/orientation) and watch the robot autonomously navigate to the goal.

Close all terminals, you are done with the turtlebot3 simulation.

## Mapping and Navigation: MyBot
We follow the same steps for our own differential drive robot.

#### Creating the map
Run the following commands.  Use teleop to move the robot around and create an accurate and thorough map.

1. Change the robot description in turtlebot3_world.launch:
    * `roscd mybot_description/launch`
    * `cp turtlebot3_world.launch mybot_tb3_world.launch`
    * change line 17 from:
```
         <param name="robot_description" command="$(find xacro)/xacro --inorder $(find turtlebot3_description)/urdf/turtlebot3_$(arg model).urdf.xacro" /> 
```
to
```
<param name="robot_description" command="$(find xacro)/xacro --inorder $(find mybot_description)/urdf/mybot.xacro" /> 
```

2. Copy and modify `turtlebot3_slam.launch`
    * cp ``rospack find turtlebot3_slam``/launch/turtlebot3_slam.launch ``rospack find mybot_navigation``/launch/mybot_slam.launch
    * change line 8-11 from:
```
  <!-- TurtleBot3 -->
  <include file="$(find turtlebot3_bringup)/launch/turtlebot3_remote.launch">
    <arg name="model" value="$(arg model)" />
  </include>
```
to
```
  <!-- MyBot -->
  <param name="robot_description" command="$(find xacro)/xacro.py '$(find mybot_description)/urdf/mybot.xacro'"/>
  <node name="joint_state_publisher" pkg="joint_state_publisher" type="joint_state_publisher">
    <param name="use_gui" value="False"/>
  </node>
  <node name="robot_state_publisher" pkg="robot_state_publisher" type="robot_state_publisher"/>
  <!-- mybot has chassis, turtlebot has base_footprint -->
  <node pkg="tf" type="static_transform_publisher" name="base" args="0 0 0 0 0 0 chassis base_footprint 100" />
```

3. In `mybot_description/urdf/mybot.gazebo` change the scan topic in line 90 from:
   ` /mybot/laser/scan` to `/scan`
 to match what turtlebot_slam expects.

4. In Terminal 1, launch the gazebo world
```
roslaunch mybot_gazebo mybot_tb3_world.launch
```

5. In Terminal 2, start map building (also starts rviz)
```
roslaunch mybot_navigation mybot_slam.launch slam_methods:=gmapping
```

6. In Terminal 3, start teleop
```
roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
```

### Saving the map
1. In Terminal 4, save the map to some file path
```
rosrun map_server map_saver -f /tmp/test_map
```

### Loading the map
Close all previous terminals and run the following commands below.  Once loaded, use rviz to set navigation waypoints and the robot should move autonomously.

1. Copy and modify `turtlebot3_navigation.launch`

    * cp ```rospack find turtlebot3_navigation```/launch/turtlebot3_navigation.launch ``rospack find mybot_navigation``/launch/turtlebot3_navigation.launch
    * change line 8-11 from:
```
  <!-- TurtleBot3 -->
  <include file="$(find turtlebot3_bringup)/launch/turtlebot3_remote.launch">
    <arg name="model" value="$(arg model)" />
  </include>
```
to
```
  <!-- MyBot -->
  <param name="robot_description" command="$(find xacro)/xacro.py '$(find mybot_description)/urdf/mybot.xacro'"/>
  <node name="joint_state_publisher" pkg="joint_state_publisher" type="joint_state_publisher">
    <param name="use_gui" value="False"/>
  </node>
  <node name="robot_state_publisher" pkg="robot_state_publisher" type="robot_state_publisher"/>
  <!-- mybot has chassis, turtlebot has base_footprint -->
  <node pkg="tf" type="static_transform_publisher" name="base" args="0 0 0 0 0 0 chassis base_footprint 100" />
```

2. In Terminal 1, launch the Gazebo world
`roslaunch mybot_gazebo mybot_tb3_world.launch`

3. In Terminal 2, start navigating with the map (will start rviz too)
`roslaunch mybot_navigation mybot_navigation.launch map_file:=/tmp/test_map.yaml`

4. In rviz, estimate initial pose - click `2D Pose Estimate` and click the approximate location of the robot on the map, and drag to indicate the direction.

5. In Terminal 3, start teleop and move the robot around.  The estimated positions should converge on the true position pretty quickly.
`roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch`

6. In rviz, send a few `2D Navigation Goals` (click on `2D Nav Goal and click/drag to set position/orientation) and watch the robot autonomously navigate to the goal.

Close all terminals, you are done with mybot navigation.
