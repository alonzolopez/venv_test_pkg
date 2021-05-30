# Install

1.  First create the package
    ```
    cd ~/catkin_ws/src/
    catkin_create_pkg venv_test_pkg
    mkdir -p src
    cd src
    ```
2.  Next, create the venv that seemingly clones the python executable at `PYTHONPATH`
    ```
    python3 -m venv test-env
    source test-env/bin/activate
    ```
    Use `pip list` to see what packages the venv comes with. It's not a fresh install.

3.  Next, create another venv but this time unset the `PYTHONPATH` so it's a fresh venv.
    ```
    unset PYTHONPATH 
    python3 -m venv test-env2
    source test-env2/bin/activate
    ```
    Again, use `pip list` to see what packages the venv comes with. This time it's a fresh install.
4. Create a python file [python-test](./src/python-test) and use it to test the different venvs. In one terminal session:
    ```
    cd ~/catkin_ws/src/venv_test_pkg
    source test-env/bin/activate
    cd src
    python3 python-test
    ```
    Should output something similar to:
    ```
    Python version
    3.8.5 (default, Jan 27 2021, 15:41:15) 
    [GCC 9.3.0]
    Version info.
    sys.version_info(major=3, minor=8, micro=5, releaselevel='final', serial=0)
    Executable path
    /home/alonzo/catkin_ws/src/venv_test_pkg/test-env/bin/python3
    ```
    In another terminal session, run:
    ```
    cd ~/catkin_ws/src/venv_test_pkg
    source test-env2/bin/activate
    cd src
    python3 python-test
    ```
    which should output something similar to:
    ```
    Python version
    3.8.5 (default, Jan 27 2021, 15:41:15) 
    [GCC 9.3.0]
    Version info.
    sys.version_info(major=3, minor=8, micro=5, releaselevel='final', serial=0)
    Executable path
    /home/alonzo/catkin_ws/src/venv_test_pkg/test-env2/bin/python3
    ```
    Notice the different Executable paths when run in each venv.

## Using the System's Python3 Executable
1. Create a python file with no `.py` suffix like [python-venv-test](./src/python-venv-test).
2. Modify CMakeLists.txt to install our python scripts so we can run them with ROS
    ```
    catkin_install_python(PROGRAMS
        src/python-venv-test
        src/python-venv2-test
        DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
    )
    ```
3. Build
    ```
    cd ~/catkin_ws
    catkin_make
    source devel/setup.bash
    ```
    `catkin_make` outputs:
    ```
    -- Using PYTHON_EXECUTABLE: /usr/bin/python3
    ``` 
    and
    ```
    -- +++ processing catkin package: 'venv_test_pkg'
    -- ==> add_subdirectory(venv_test_pkg)
    -- Installing devel-space wrapper /home/alonzo/catkin_ws/src/venv_test_pkg/src/python-venv-test to /home/alonzo/catkin_ws/devel/lib/venv_test_pkg
    -- Installing devel-space wrapper /home/alonzo/catkin_ws/src/venv_test_pkg/src/python-venv2-test to /home/alonzo/catkin_ws/devel/lib/venv_test_pkg
    ```
4. Run the Python script as a ROS node which prints the python executable information as well as numpy version held by the executable:
    ```
    rosrun venv_test_pkg python-venv-test
    ```
    Even if you're in a venv, the above script will run using the system's python executable and packages (because there's no shebang saying otherwise).
## Running a ROS node out of a venv
To demo ROS nodes as Python scripts each with their own venv, we'll set up two venvs each with a different version of numpy and check the version of numpy imported by the program.

### venv 1 with numpy==1.18.0
Let's use `numpy==1.18.0` in venv 1. Assuming you already created `test-env` above:
1. Activate the venv and install numpy==1.18.0
    ```
    source test-env/bin/activate
    pip install numpy==1.18.0
    ```
2. Create the python script that imports numpy and prints its version. This script must have the appropraite shebang pointing to the python executable associated with test-env2. See [python-test-4.py](./src/python-test-4.py) as an example.
3. Make the script executable
    ```
    chmod u+x python-test-4.py
    ```
    Note that we DO NOT need to mofidy the `catkin_install_python()` function in CMakeLists.txt for this script because we make it executable and run it directly. You shouldn't do both - make it executable and use `catkin_install_python()` - because this could create two executables of the same name.
4. Activate the venv and run the ROS node. **Important: make sure you run it in the same terminal session where you activated test-env**
    ```
    rosrun venv_test_pkg python-test-4.py
    ```
    output:
    ```
    Python version
    3.8.5 (default, Jan 27 2021, 15:41:15) 
    [GCC 9.3.0]
    Version info.
    sys.version_info(major=3, minor=8, micro=5, releaselevel='final', serial=0)
    Executable path
    /home/alonzo/catkin_ws/src/venv_test_pkg/test-env/bin/python3
    1.18.0
    ```
5. Deactivate the venv
    ```
    deactivate
    ```

### venv 2 with numpy==1.20.0
Let's use `numpy==1.20.0` in venv 2. Assuming you already created `test-env2` above:
1. Activate the venv and install numpy==1.20.0
    ```
    source test-env2/bin/activate
    pip install numpy==1.20.0
    ```
2. Create the python script that imports numpy and prints its version. This script must have the appropraite shebang pointing to the python executable associated with test-env2. See [python-test-4.py](./src/python-test-4.py) as an example.
3. Make the script executable
    ```
    chmod u+x python-test-4.py
    ```
    Note that we DO NOT need to mofidy the `catkin_install_python()` function in CMakeLists.txt for this script because we make it executable and run it directly. You shouldn't do both - make it executable and use `catkin_install_python()` - because this could create two executables of the same name.
4. Activate the venv and run the ROS node. **Important: make sure you run it in the same terminal session where you activated test-env2**
    ```
    source test-env2/bin/activate
    rosrun venv_test_pkg python-test-4.py
    ```
    output:
    ```
    Python version
    3.8.5 (default, Jan 27 2021, 15:41:15) 
    [GCC 9.3.0]
    Version info.
    sys.version_info(major=3, minor=8, micro=5, releaselevel='final', serial=0)
    Executable path
    /home/alonzo/catkin_ws/src/venv_test_pkg/test-env2/bin/python3
    1.20.0
    ```
5. Deactivate the venv
    ```
    deactivate
    ```


## Next steps
Look into [catkin_virtualenv](https://github.com/locusrobotics/catkin_virtualenv/blob/master/README.md) to maybe properly use venvs with ROS packages.

Otherwise, how would we do this properly when using roslaunch to launch multiple Python ROS nodes each with their own venv?