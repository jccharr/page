## Sytsem requirements
<p style="text-align: justify;text-justify: inter-word;">
The following softwares are required to run the simulator :
  <ul>
    <li> <span style="font-weight: bold">Operating System:</span> The simulator has been tested on systems running OS X or linux operating systems. The simulator was not tested on the windows operating system but it should work with some tweaking. </li> 
    <li> <span style="font-weight: bold">Git:</span> The git software (available per default on linux based systems) is required to clone the source code from github but it can also be downloaded as an archive without git, see next paragraph.</li>
    <li> <span style="font-weight: bold">Java Development Kit (JDK):</span> To compile the source code and run the simulator a recent JDK must be installed on the machine.</li>
    <li> <span style="font-weight: bold">Java make:</span> The make utility (available per default on linux based systems) is used to compile le source files of the simulator. However, the shell commands found in the Makefile can be used to compile the code without this utility.</li>
  </ul>  
</p>

## Download
<p style="text-align: justify;text-justify: inter-word;">
To start using this simulator the source code must be downloaded on the filesystem of the machine (or machines) that will be running the simulation. The source can be downloaded using one of the following two ways:
  <ol>
    <li> From a terminal, execute the following GIT instruction, the git software must be already installed on the machine:<br/><span style="font-weight: bold">git clone https://github.com/jccharr/DistributedCloudSimulator.git</span></li>
    <li> Or just click on the following link to download an archive containing the source files and then uncompress the downloaded archive: <a href="https://github.com/jccharr/DistributedCloudSimulator/zipball/master"> download link</a></li> 
  </ol>
</p>

## Compile 
<p style="text-align: justify;text-justify: inter-word;">
  In a terminal change the current directory to the uncompressed directory: <span style="font-weight: bold">cd DistributedCloudSimulator</span> <br/>
  To compile just execute the following instruction from a terminal: <span style="font-weight: bold">make</span> <br/>
  The .class files will be generated in the bin directory. <br/>
  To remove the compiled .class files execute : <span style="font-weight: bold">make clean</span>
</p>

## Run a simulation on the local machine
<p style="text-align: justify;text-justify: inter-word;">
To run the simplest simulation with this simulator, at least four processes should be launched, one for each manager type: Warehouse, Cell, Rack and Server managers. In the following example, a cloud containing just one Rack consisting of two servers is simulated. Each of the follwing instructions should be executed in its own terminal tab and launches one of the managers. Since all the processes in this example will be executed on the local machine, the machine name is equal to 127.0.0.1 for all the processes and they only use differents ports to communicate with each other. As explained in Server's Resources definition section, the ServerLauncher class should be edited to specify the specifications of the servers.
  
  <ol>
    <li>To launch the Warehouse manager: <span style="font-weight: bold">java bin/examples/WarehouseLauncher 127.0.0.1 961 apps.xml</span><br/>The Warehouse manager uses the port 961 to communicate with the other components. The app.xml file contains the definition of the tasks that will be executed by the simulated cloud as explained in the Task and Application sections.</li>
    <li>To launch a Cell manager: <span style="font-weight: bold">java bin/examples/CellLauncher 127.0.0.1 962 127.0.0.1 961 </span><br/>The Cell manager uses the port 962 to communicate with the other components and requires the name of the machine running the upper node in the hierarchy (Warehouse manager) and its port number to connect to it.</li>
    <li>To launch a Rack manager connected to the Cell manager above: <span style="font-weight: bold">java bin/examples/RackLauncher 127.0.0.1 963 127.0.0.1 962 2 </span><br/>As for the Cell manager, it requires the name of the machine running the upper node in the hierarchy (Cell manager) and its port number to connect to it. Moreover, the user of the simulator must specify in the last argument, how manu servers will be connected to this Rack manager, in this example just 2 servers. </li>
    <li>To launch the first Server manager connected to the Rack manager above: <span style="font-weight: bold">java bin/examples/ServerLauncher 127.0.0.1 964 127.0.0.1 963 </span><br/>Similar to the other manager, it  requires the name of the machine running the upper node in the hierarchy (Rack manager) and its port number to connect to it.</li>
    <li>To launch the second Server manager connected to the Rack manager above: <span style="font-weight: bold">java bin/examples/ServerLauncher 127.0.0.1 965 127.0.0.1 963</span>
    </li>
  </ol>
  
Off course the simulation of a cloud consisting of many racks and tens of servers per rack, would not be practical if every manager should be lauched on its own as above. In the next section, an automated launch process is explained to launch the simulator over a cluster and to simulate a larger cloud.
</p>

## Run a simulation on a cluster
<p style="text-align: justify;text-justify: inter-word;">
<span style="font-weight: bold">Requirements: </span> In this section, it is assumed that the machines that are supposed to run the simulation has been reserved by the user of the simulator and that he can ssh from one machine to the other without requiring the input of the password. Moreover, it is assumed that those machines share the same network file system (NFS). Finally, it is assumed that the user has the permissions to execute the simulator over these machines and to write the output on the NFS.</p>

<p style="text-align: justify;text-justify: inter-word;">
<span style="font-weight: bold">The machines running the simulation: </span>The names of the machines to be used by the simulator should be listed in a file, one name per line. The same name can be added more than once into the file, one manager process will be launched for each occurence. In the following example, this file is called machinesFile. The script addPortsToMachinesFile.py takes the machine file as argument, associates a port number to each machine name in the machines file and saves them in a new file called "machinesFileWithPorts". Each line of this new file has the following format: machineFileName portNumber. If the same machine name appears more than once in the first file, the script will associate a different port number for each occurence.</p>

<p style="text-align: justify;text-justify: inter-word;">
<span style="font-weight: bold">Launching the simulator: </span> To launch the simulation on the reserved machines, the launch script ("launchScript.py") should be executed. It taskes the numbers of cells, racks per cell and servers per rack as input. It also requires the machines file with the ports numbers, the applications' description XML file and an optional output directory. The following command is an example for simulating a cloud consisting of 1 cell, 1 rack per cell and 2 servers per rack:
</p>

```shell
python launchScript.py cells=1 racks=1 servers=2 machinesFile=machinesFileWithPorts appsDescriptionFile=apps.xml outputDirectory=out
```
<p style="text-align: justify;text-justify: inter-word;">
<span style="font-weight: bold">Launching many simulations on the same cloud: </span> If the user would like to simulate many applications' senarios on the same cloud architecture, we have developped another python script called "loop_script.py" that takes almost the same parameters as the launch script. However, instead of an apps description file, it takes a directory containing the files describing the apps. The script launches a simulation with each apps description file in the directory. The simulations are executed successively, one after the other. A file called "loop_lock" is used to detect if the current simulation is finished and to launch the next simulation. This script uses the launch script to launch each simulation. The following command is an example for launching a series of simulations on the same cloud infrastructure consisting of 1 cell, 1 rack per cell and 10 servers per rack:
</p>

```shell
python loop_script.py  cells=1 racks=1 servers=10 machinesFile=machinesFileWithPorts appsDescriptionDirectory=appsDirectory outputDirectory=out
```
