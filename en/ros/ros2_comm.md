# PX4-ROS2 bridge

This section of the guide focus on the bridge between PX4 and ROS 2, made available through the *microRTPS* bridge and resulting products to interface with ROS 2, *px4_ros_com* and *px4_msgs* ROS 2 packages.
It also provides information in how to:
1. Connect ROS 2 nodes with PX4 (via the *microRTPS* Bridge, and using the `px4_ros_com` package)
1. Connect ROS (ROS "version 1") nodes with PX4 by additionally using the `ros1_bridge` package to bridge ROS2 and ROS (1).

:::note
For more details on the *microRTPS* bridge without using ROS 2, please check the [RTPS/DDS Interface section](../middleware/micrortps.md).
:::

:::note
For a more detailed and visual explanation of the bridge with ROS 2, check the following talks/presentation videos:
1. [PX4 Dev Summit 2019 - "ROS2 Powered PX4"](https://www.youtube.com/watch?v=2Szw8Pk3Z0Q)
1. [ROS World 2020 - Getting started with ROS 2 and PX4](https://www.youtube.com/watch?v=qhLATrkA_Gw)
:::


## Code generation

:::note
[Fast RTPS (DDS) 2.0.0 and FastRTPSGen 1.0.4 or later must be installed](../dev_setup/fast-rtps-installation.md) in order to generate the required code!
:::


### ROS2/ROS application pipeline

The application pipeline for ROS2 is very straightforward!
Because ROS2 uses DDS/RTPS as its native communications middleware, you can create a ROS2 listener or advertiser node to publish and subscribe to uORB data on PX4, via the *PX4 Fast RTPS Bridge*.
This is shown below.

:::note
You do need to make sure that the message types, headers and source files used on both client and agent side (and consequently, on the ROS nodes) are generated from the same Interface Description Language (IDL) files.
The `px4_ros_com` package provides the needed infrastructure for generating messages and headers needed by ROS2.
:::

![Architecture with ROS2](../../assets/middleware/micrortps/architecture_ros2.png)

The architecture for integrating ROS applications with PX4 is shown below.

![Architecture with ROS](../../assets/middleware/micrortps/architecture_ros.png)

Note the use of [ros1_bridge](https://github.com/ros2/ros1_bridge), which bridges messages between ROS2 and ROS.
This is needed because the first version of ROS does not support RTPS.


### ROS2/ROS applications

The [px4_ros_com](https://github.com/PX4/px4_ros_com) package, when built, generates everything needed to access PX4 uORB messages from a ROS2 node (for ROS you also need [ros1_bridge](https://github.com/ros2/ros1_bridge)).
This includes all the required components of the *PX4 RTPS bridge*, including the `micrortps_agent` and the IDL files (required by the `micrortps_agent`).

The ROS and ROS2 message definition headers and interfaces are generated from the [px4_msgs](https://github.com/PX4/px4_msgs) package, which match the uORB messages counterparts under PX4-Autopilot.
These are required by `px4_ros_com` when generating the IDL files to be used by the `micrortps_agent`.

Both `px4_ros_com` and `px4_msgs` packages have two separate branches:
- a `master` branch, used with ROS2.
  It contains code to generate all the required ROS2 messages and IDL files to bridge PX4 with ROS2 nodes.
- a `ros1` branch, used with ROS.
  It contains code to generate the ROS message headers and source files, which can be used *with* the `ros1_bridge` to share data between PX4 and ROS.

Both branches in `px4_ros_com` additionally include some example listener and advertiser example nodes.


## Agent interfacing with a ROS2 middleware

Building `px4_ros_com` automatically generates and builds the agent application, though it requires (as a dependency), that the `px4_msgs` package also gets build on the same ROS2 workspace (or overlaid from another ROS2 workspace).
Since it is also installed using the [`colcon`](http://design.ros2.org/articles/build_tool.html) build tools, running it works exactly the same way as the above.
Check the **Building the `px4_ros_com` package** for details about the build structure.


## Building the `px4_ros_com` and `px4_msgs` package

Install and setup both ROS2 and ROS environments on your development machine
and separately clone the `px4_ros_com` and `px4_msgs` repo for both the `master` and `ros1` branches (see [above for more information](#px4_ros_com)).

:::note
Only the master branch is needed for ROS2 (both are needed to target ROS).
:::


### Installing ROS2 and respective dependencies

:::note
This install and build guide covers ROS2 Foxy in Ubuntu 20.04.
:::

In order to install ROS Melodic and ROS2 Dashing (officially supported) on a Ubuntu 18.04 machine, follow the links below, respectively:
1. [Install ROS2 Foxy](https://index.ros.org/doc/ros2/Installation/Foxy/Linux-Install-Debians/)
1. The install process should also install the *colcon* build tools, but in case that doesn't happen, you can install the tools manually:

   ```sh
   sudo apt install python3-colcon-common-extensions
   ```

1. *eigen3_cmake_module* is also required, since Eigen3 is used on the transforms library:

   ```sh
   sudo apt install ros-dashing-eigen3-cmake-module
   ```

1. *setuptools* must also be installed (using *pip* or *apt*):

   ```sh
   sudo pip3 install -U setuptools
   ```

   :::caution
   If you want to use ROS 1 topics, in communication with ROS 2, you need to build the `ros1_bridge` from source,
   and setup a separate workspace. Check the Setting up the workspaces -> ROS 1 workspace section.
   :::

### Setting up the workspaces

Since the ROS2 and ROS require different environments you will need a separate workspace for each ROS version.
As an example:


#### ROS 2 workspace

   For ROS2, create a workspace using:
   ```sh
   mkdir -p ~/px4_ros_com_ros2/src
   ```

   Then, clone the respective ROS2 (`master`) branch to the `/src` directory:
   ```sh
   $ git clone https://github.com/PX4/px4_ros_com.git ~/px4_ros_com_ros2/src/px4_ros_com # clones the master branch
   $ git clone https://github.com/PX4/px4_msgs.git ~/px4_ros_com_ros2/src/px4_msgs
   ```

#### ROS (1) workspace
   For ROS, follow exactly the same process, but create a different directory and clone a different branch:
   ```sh
   mkdir -p ~/px4_ros_com_ros1/src
   ```

   Then, clone the respective ROS2 (`ros1`) branch to the `/src` directory:
   ```sh
   $ git clone https://github.com/PX4/px4_ros_com.git ~/px4_ros_com_ros1/src/px4_ros_com -b ros1 # clones the 'ros1' branch
   $ git clone https://github.com/PX4/px4_msgs.git ~/px4_ros_com_ros1/src/px4_msgs -b ros1
   ```

### Building the workspaces

The directory `px4_ros_com/scripts` contains multiple scripts that can be used to build both workspaces.

To build both (ROS (1) and ROS 2) workspaces with a single script, use the `build_all.bash`.
Check the usage with `source build_all.bash --help`.
The most common way of using it is by passing the ROS(1) workspace directory path and also the PX4-Autopilot directory path:

```sh
$ source build_all.bash --ros1_ws_dir <path/to/px4_ros_com_ros1/ws>
```

:::note
Using the `--verbose` argument will allow you to see the full *colcon* build output.
:::

:::note
The build process will open new tabs on the console, corresponding to different stages of the build process that need to have different environment configurations sourced.
:::

One can also use the following individual scripts in order to build the individual parts:

- `build_ros1_bridge.bash`, to build the `ros1_bridge`;
- `build_ros1_workspace.bash` (only on the `ros1` branch of `px4_ros_com`), to build the ROS1 workspace to where the `px4_ros_com` and `px4_msgs` `ros1` branches were cloned;
- `build_ros2_workspace.bash`, to build the ROS2 workspace to where the `px4_ros_com` and `px4_msgs` `master` branches were cloned.

The steps below show how to *manually* build the packages (provided for your information/better understanding only).

To build the ROS 2 workspace only:

1. `cd` into `px4_ros_com_ros2` dir and source the ROS2 environment.
   Don't mind if it tells you that a previous workspace was set before:

   ```sh
   cd ~/px4_ros_com_ros2
   source /opt/ros/foxy/setup.bash
   ```

1. Build the workspace:

   ```sh
   colcon build --symlink-install --event-handlers console_direct+
   ```        

To build both workspaces (in replacement of the previous steps):

1. `cd` into `px4_ros_com_ros2` dir and source the ROS2 environment.
   Don't mind if it tells you that a previous workspace was set before:

   ```sh
   source /opt/ros/foxy/setup.bash
   ```

1. Clone the `ros1_bridge` package so it can be built on the ROS2 workspace:

   ```sh
   git clone https://github.com/ros2/ros1_bridge.git -b dashing ~/px4_ros_com_ros2/src/ros1_bridge
   ```

1. Build the `px4_ros_com` and `px4_msgs` packages, excluding the `ros1_bridge` package:

   ```sh
   colcon build --symlink-install --packages-skip ros1_bridge --event-handlers console_direct+
   ```

   :::note
   `--event-handlers console_direct+` only serves the purpose of adding verbosity to the `colcon` build process and can be removed if one wants a more "quiet" build.
   :::

1. Then, follows the process of building the ROS(1) packages side.
   For that, one requires to open a new terminal window and source the ROS(1) environment that has installed on the system:

   ```sh
   source /opt/ros/melodic/setup.bash
   ```

1. On the terminal of the previous step, build the `px4_ros_com` and `px4_msgs` packages on the ROS end:

   ```sh
   cd ~/px4_ros_com_ros1 && colcon build --symlink-install --event-handlers console_direct+
   ```

1. Before building the `ros1_bridge`, one needs to open a new terminal and then source the environments and workspaces following the order below:

   ```sh
   source ~/px4_ros_com_ros1/install/setup.bash
   source ~/px4_ros_com_ros2/install/setup.bash
   ```

1. Finally, build the `ros1_bridge`.
   Note that the build process may consume a lot of memory resources.
   On a resource limited machine, reduce the number of jobs being processed in parallel (e.g. set environment variable `MAKEFLAGS=-j1`).
   For more details on the build process, see the build instructions on the [ros1_bridge](https://github.com/ros2/ros1_bridge) package page.
   ```sh
   cd ~/px4_ros_com_ros2 && colcon build --symlink-install --packages-select ros1_bridge --cmake-force-configure --event-handlers console_direct+
   ```

### Cleaning the workspaces

After building the workspaces there are many files that must be deleted before you can do a clean/fresh build (for example, after you have changed some code and want to rebuild).
Unfortunately *colcon* does not currently have a way of cleaning the generated **build**, **install** and **log** directories, so these directories must be deleted manually.

The **clean_all.bash** script (in **px4_ros_com/scripts**) is provided to ease this cleaning process.
The most common way of using it is by passing it the ROS(1) workspace directory path (since it's usually not on the default path):

```sh
$ source clean_all.bash --ros1_ws_dir <path/to/px4_ros_com_ros1/ws>
```

## Creating a ROS2 listener

With the `px4_ros_com` built successfully, one can now take advantage of the generated *microRTPS* agent app and also from the generated sources and headers of the ROS2 msgs from `px4_msgs`, which represent a one-to-one matching with the uORB counterparts.

To create a listener node on ROS2, lets take as an example the `sensor_combined_listener.cpp` node under `px4_ros_com/src/examples/listeners`:

```cpp
#include <rclcpp/rclcpp.hpp>
#include <px4_msgs/msg/sensor_combined.hpp>
```

The above brings to use the required C++ libraries to interface with the ROS2 middleware.
It also includes the required message header file.

```cpp
/**
 * @brief Sensor Combined uORB topic data callback
 */
class SensorCombinedListener : public rclcpp::Node
{
```

The above creates a `SensorCombinedListener` class that subclasses the generic `rclcpp::Node` base class.

```cpp
public:
	explicit SensorCombinedListener() : Node("sensor_combined_listener") {
		subscription_ = this->create_subscription<px4_msgs::msg::SensorCombined>(
			"SensorCombined_PubSubTopic",
			10,
			[this](const px4_msgs::msg::SensorCombined::UniquePtr msg) {
			std::cout << "\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n";
			std::cout << "RECEIVED SENSOR COMBINED DATA"   << std::endl;
			std::cout << "============================="   << std::endl;
			std::cout << "ts: "          << msg->timestamp    << std::endl;
			std::cout << "gyro_rad[0]: " << msg->gyro_rad[0]  << std::endl;
			std::cout << "gyro_rad[1]: " << msg->gyro_rad[1]  << std::endl;
			std::cout << "gyro_rad[2]: " << msg->gyro_rad[2]  << std::endl;
			std::cout << "gyro_integral_dt: " << msg->gyro_integral_dt << std::endl;
			std::cout << "accelerometer_timestamp_relative: " << msg->accelerometer_timestamp_relative << std::endl;
			std::cout << "accelerometer_m_s2[0]: " << msg->accelerometer_m_s2[0] << std::endl;
			std::cout << "accelerometer_m_s2[1]: " << msg->accelerometer_m_s2[1] << std::endl;
			std::cout << "accelerometer_m_s2[2]: " << msg->accelerometer_m_s2[2] << std::endl;
			std::cout << "accelerometer_integral_dt: " << msg->accelerometer_integral_dt << std::endl;
		});
	}
```

This creates a callback function for when the `sensor_combined` uORB messages are received (now as RTPS/DDS messages).
It outputs the content of the message fields each time the message is received.

```cpp
private:
	rclcpp::Subscription<px4_msgs::msg::SensorCombined>::SharedPtr subscription_;
};
```

The above create a subscription to the `sensor_combined_topic` which can be matched with one or more compatible ROS publishers.

```cpp
int main(int argc, char *argv[])
{
	std::cout << "Starting sensor_combined listener node..." << std::endl;
	setvbuf(stdout, NULL, _IONBF, BUFSIZ);
	rclcpp::init(argc, argv);
	rclcpp::spin(std::make_shared<SensorCombinedListener>());

	rclcpp::shutdown();
	return 0;
}
```

The instantiation of the `SensorCombinedListener` class as a ROS node is done on the `main` function.


## Creating a ROS2 advertiser

A ROS2 advertiser node publishes data into the DDS/RTPS network (and hence to PX4).

Taking as an example the `debug_vect_advertiser.cpp` under `px4_ros_com/src/advertisers`:

```cpp
#include <chrono>
#include <rclcpp/rclcpp.hpp>
#include <px4_msgs/msg/debug_vect.hpp>

using namespace std::chrono_literals;
```

Bring in the required headers, including the `debug_vect` msg header.

```cpp
class DebugVectAdvertiser : public rclcpp::Node
{
```

The above creates a `DebugVectAdvertiser` class that subclasses the generic `rclcpp::Node` base class.

```cpp
public:
	DebugVectAdvertiser() : Node("debug_vect_advertiser") {
		publisher_ = this->create_publisher<px4_msgs::msg::DebugVect>("DebugVect_PubSubTopic", 10);
		auto timer_callback =
		[this]()->void {
			auto debug_vect = px4_msgs::msg::DebugVect();
			debug_vect.timestamp = std::chrono::time_point_cast<std::chrono::microseconds>(std::chrono::steady_clock::now()).time_since_epoch().count();
			std::string name = "test";
			std::copy(name.begin(), name.end(), debug_vect.name.begin());
			debug_vect.x = 1.0;
			debug_vect.y = 2.0;
			debug_vect.z = 3.0;
			RCLCPP_INFO(this->get_logger(), "\033[97m Publishing debug_vect: time: %llu x: %f y: %f z: %f \033[0m",
                                debug_vect.timestamp, debug_vect.x, debug_vect.y, debug_vect.z);
			this->publisher_->publish(debug_vect);
		};
		timer_ = this->create_wall_timer(500ms, timer_callback);
	}

private:
	rclcpp::TimerBase::SharedPtr timer_;
	rclcpp::Publisher<px4_msgs::msg::DebugVect>::SharedPtr publisher_;
};
```

This creates a function for when messages are to be sent.
The messages are sent based on a timed callback, which sends two messages per second based on a timer.

```cpp
int main(int argc, char *argv[])
{
	std::cout << "Starting debug_vect advertiser node..." << std::endl;
	setvbuf(stdout, NULL, _IONBF, BUFSIZ);
	rclcpp::init(argc, argv);
	rclcpp::spin(std::make_shared<DebugVectAdvertiser>());

	rclcpp::shutdown();
	return 0;
}
```

The instantiation of the `DebugVectAdvertiser` class as a ROS node is done on the `main` function.

## Creating a ROS(1) listener

The creation of ROS nodes is a well known and documented process.
An example of a ROS listener for `sensor_combined` messages can be found in the `ros1` branch repo, under `px4_ros_com/src/listeners`.


## Offboard example


## Testing the PX4-FastRPTS bridge

To quickly test the package (using PX4 SITL with Gazebo):

1. Start PX4 SITL with Gazebo using:
   ```sh
   make px4_sitl_rtps gazebo
   ```

To only use ROS 2:

1. Make sure to source the workspace configuration first:

   ```sh
   source ~/px4_ros_com_ros2/install/setup.bash
   ```

1. On a terminal, source the ROS2 workspace and then start the `micrortps_agent` daemon with UDP as the transport protocol:
   ```sh
   $ source ~/px4_ros_com_ros2/install/setup.bash
   $ micrortps_agent -t UDP
   ```

1. On the [NuttShell/System Console](../debug/system_console.md) of the SITL window, start the `micrortps_client` daemon also in UDP:
   ```sh
   pxh> micrortps_client start -t UDP
   ```

To use ROS (1) and ROS 2:

1. On one terminal, source the ROS2 environment and workspace and launch the `ros1_bridge` (this allows ROS2 and ROS nodes to communicate with each other).
   Also set the `ROS_MASTER_URI` where the `roscore` is/will be running:
   ```sh
   $ source /opt/ros/dashing/setup.bash
   $ source ~/px4_ros_com_ros2/install/local_setup.bash
   $ export ROS_MASTER_URI=http://localhost:11311
   $ ros2 run ros1_bridge dynamic_bridge
   ```

1. On another terminal, source the ROS workspace and launch the `sensor_combined` listener node.
   Since you are launching through `roslaunch`, this will also automatically start the `roscore`:
   ```sh
   $ source ~/px4_ros_com_ros1/install/setup.bash
   $ roslaunch px4_ros_com sensor_combined_listener.launch
   ```

1. On a terminal, source the ROS2 workspace and then start the `micrortps_agent` daemon with UDP as the transport protocol:
   ```sh
   $ source ~/px4_ros_com_ros2/install/setup.bash
   $ micrortps_agent -t UDP
   ```

1. On the [NuttShell/System Console](../debug/system_console.md), start the `micrortps_client` daemon also in UDP:
   ```sh
   > micrortps_client start -t UDP
   ```

For both cases above, in case of success. you will be able to see the data being printed on the terminal/console where you launched the ROS listener:

   ```sh
   RECEIVED DATA FROM SENSOR COMBINED
   ================================
   ts: 870938190
   gyro_rad[0]: 0.00341645
   gyro_rad[1]: 0.00626475
   gyro_rad[2]: -0.000515705
   gyro_integral_dt: 4739
   accelerometer_timestamp_relative: 0
   accelerometer_m_s2[0]: -0.273381
   accelerometer_m_s2[1]: 0.0949186
   accelerometer_m_s2[2]: -9.76044
   accelerometer_integral_dt: 4739
   ```

You can also verify the rate of the message using `rostopic hz`.
   For the case of `sensor_combined`:
   ```sh
   average rate: 248.187
   	min: 0.000s max: 0.012s std dev: 0.00147s window: 2724
   average rate: 248.006
   	min: 0.000s max: 0.012s std dev: 0.00147s window: 2972
   average rate: 247.330
   	min: 0.000s max: 0.012s std dev: 0.00148s window: 3212
   average rate: 247.497
   	min: 0.000s max: 0.012s std dev: 0.00149s window: 3464
   average rate: 247.458
   	min: 0.000s max: 0.012s std dev: 0.00149s window: 3712
   average rate: 247.485
   	min: 0.000s max: 0.012s std dev: 0.00148s window: 3960
   ```

1. You can also test the `sensor_combined` ROS2 listener by launching the example launch file in the terminal:
   ```sh
   $ source ~/px4_ros_com_ros2/install/local_setup.bash
   $ ros2 launch px4_ros_com sensor_combined_listener.launch.py
   ```

And it should also get data being printed to the console output.

:::note
If ones uses the `build_all.bash` script, it automatically open and source all the required terminals so one just has to run the respective apps in each terminal.
:::
