# How to generate synthetic dataset using a PX4 drone
General speaking, we firstly make vehicles spawned in virtual world, running them and capture their runtime-position(path) in gazebo, then we spawn a camera ni both orignal world and tagged world, which follow the path of vehicles. Lastly, we can get our images. 
## Files
- In  `~/.gazebo/`: 
  - `models`: models depended by the worlds in husky simulation 
  - `gazebo_models`: all models fetched from models.gazebosims.org 
  - `px4_models`: models depened by the worlds in px4 simulations. 

- In `~/catkin_ws`: root folder for husky simulation and stereo camera spawn. 
  - `src/simulator/natual_environment/launch`: where we store our launch files 

- In `~/projects/PX4-autopilot`: root folder of px4 drones 

## Steps 

### Step 0: preperation
- Two virtural worlds: 
  - one real world with textures 
  - one tagged world labeled by color 
- A flight plan for mavros(you can also manually control it)


### Step 1: Run px4 simulation on gazebo and record its path. 
In px4-gazebo simulation, we use mavros as our ROS master. In this step, we concentrate on: 
1. how to control vehicles 
2. how to capture its path 
In the rostopic boardcasted by mavros, ![](https://www.researchgate.net/profile/Muhamad-Akbar-2/publication/340856977/figure/fig1/AS:883472742756354@1587647724649/List-of-Mavros-topics.ppm), we particularly focus on these topics: 
- About Status: 
  - `mavros/local_position/pose` (PoseStamped: http://docs.ros.org/en/api/geographic_msgs/html/msg/GeoPoseStamped.html): the (x,y,z) position in gazebo
  - `mavros/global_position/global` (GeoPoseStamped:http://docs.ros.org/en/api/geographic_msgs/html/msg/GeoPoseStamped.html): the LLA position to the flight controller. 进度纬度

- About Control: 
  - `mavros/setpint_raw/local`: set xyz position directly. 
  - `mavros/setpiont_raw/gloabal`: set LLA position directly
  - `mavros/misson/*`( http://wiki.ros.org/mavros#mavros.2FPlugins.waypoint ): use waypoints to control px4 drones. 

In summary, we can get both the local and global positions of drones, and we can control the drones using 3 different appoaroaches: set local position, set global position, set waypoints. 
In terminals: 
1. initialize px4 simulation with: 
   1. drone home position 
   2. world position 
2. record the path: `rosbag record /mavros/local_position/pose /mavros/global_position/global -o output.bag` 
3. move our drones: 
   - manually: use QGC
   - automatically: 
     - set position directly 
     - use mission plans, setting waypionts 

### Step 2: Replay flight path on spawned camera with real world 
We use Gazebo-ros to spawn cameras, which only support local positions (xyz position in gazebo)
1. add a launch file in `~/catkin_wssrc/simulator/natual_environment/launch`, which used to launch your world. 
2. add a new launch file, which we add a camera in your world, you should specify the init position of the camera. 
3. launch the new world, and add your camera in it. 
4. replay the bag file by running `python ~/catkin/test_px4_real_cam.py -i input path.bag -o output.bag` 

### Step 3: Replay flight path on spawned camera with tagged world 
Just like step 2. 

### Step 4: extract images from bag files 
put your bag file name in `~/catkin/only_image.py` and execute it. 

