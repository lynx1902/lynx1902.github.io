# Vertical State Estimator
Github Repository: https://github.com/lynx1902/vertical-estimator
### Objective
 - To design a vertical state estimator capable of achieving accurate position estimation comparable to the garmin rangefinder sensor.
- Ideal simulation setup described in *session_vio.yml* involves 4 UAVs equipped with the UVDAR system 

### Working Algorithm

- In the swarm of 4 UAVs, one of them is labelled as the focal UAV.
- The algorithm tries to estimate the vertical height of the focal UAV
- This estimation is based on fundamental 3D Line Geometry

*Note: In the code, the states of the neighboring and focal UAV are tracked with Linear Kalman Filter using the mrs_lib LKF.*
*The states initialised in the filter are*: ==$x, \dot{x}, y, \dot{y}, z, \dot{z}, \ddot{z}$==
*Our initial aim is to obtain the most accurate estimation for the  $z$ coordinate of the focal UAV.*

 - Equation for the line from a neighboring UAV to the focal UAV is given by
==$<x, y, z> = <A.x, A.y, A.z> + t * (B.x - A.x, B.y - A.y, B.z - A.z)$==
 - Here, $A$ represents the neighbor UAV's position obtained from the filter, while $B$ is a point whose $x$ and $y$ coordinates are calculated using the *horizontal(heading)* angle and $z$ coordinate is calculated using *elevation* angle between the focal and neighboring UAV
 - Ideally, the **focal UAV** will be located at the **intersection** of these lines
![intersection.jpg](https://hackmd.io/_uploads/S1LOYm8mp.jpg)
 - However, cases like skew and parallel lines are also accounted for in the code
 - For skew and parallel line cases, we calculate the **minimum distance** between the lines and assume that the focal UAV lies at the **midpoint** of this distance (red box indicates the focal UAV)
![edited.png](https://hackmd.io/_uploads/BJ7b8Y87T.png)

### Neighbor State Tracking and Additional Sensors
 - To keep track of heights and velocities of neighboring UAVs, we use the communicated states available on the odometry topics. This is important because UVDAR alone may not provide accurate estimation in every iteration
 - To improve estimation results, data from the IMU sensor is also incorporated in the filter

### Next Steps
 - The proposed algorithm seems to be estimating the height of the focal UAV with decent accuracy
 - The estimator output needs to be fed to the **odometry loop**, so that the UAV can use the estimator as the source of the height data instead of the garmin rangefinder. This can be done in possibly two ways:
     - Remapping the **range_topic** for the odometry loop to a custom name for the focal UAV(eg: *garmin/range2* instead of default *garmin/range*). The remaining UAVs should fly using unchanged odometry [*Note: The updated MRS-UAV system might have a straightforward way to do this*]
     - Writing a **ROS republisher** to publish either estimator output or garmin rangefinder data on the range topic depending on a certain condition 
 - Testing the estimator output on **uneven_plane** in gazebo. All tests conducted so far were using the grass_plane environment in gazebo 

### Sample Estimation Results
 - uav29 is the focal UAV
 - uav30, uav38 and uav39 fly without the estimator enabled
*Note: the estimator output was not used in the odometry loop*
![MicrosoftTeams-image.png](https://hackmd.io/_uploads/HkpiWKLmT.png)
![MicrosoftTeams-image.png](https://hackmd.io/_uploads/SkWUWFU76.png)

