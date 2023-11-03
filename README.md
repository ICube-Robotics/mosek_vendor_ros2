# mosek_vendor_ros2
Vendor package to use the solver MOSEK in ROS2 applications.

## Installation

1) Install the package

```bash
cd <ros_ws>/src
git clone <this_repos>
cd ..
colcon build
```

2) Get a license

Note that a license is mandatory to run MOSEK.
See the [website](https://docs.mosek.com/10.1/licensing/quickstart.html#i-don-t-have-a-license-file-yet) for the detailed procedure.

The file `mosek.lic` you will obtain has to be copied to your personal home folder in `~/mosek/mosek.lic`.


## Usage example

> `package.xml` :
```xml
...
<depend>mosek_vendor_ros2</depend>
...
```

> `CMakeLists.txt` :
```cmake
...
find_package(mosek_vendor_ros2)
...
ament_target_dependencies(<the_target> <type> mosek_vendor_ros2)
...
```
> `xxx.cpp` :
```cpp
#include <mosek_vendor_ros2/fusion.h>
