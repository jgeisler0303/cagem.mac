# This project is now superseded by the complete rework in https://github.com/jgeisler0303/CADyn

Introduction
============
This is a replacement/translation of the [MuPAD](https://de.wikipedia.org/wiki/MuPAD) script `cagem.mu` from the [EasyDyn](http://hosting.umons.ac.be/html/mecara/EasyDyn/) software. The script is ported to [Maxima](http://maxima.sourceforge.net/) in order to make EasyDyn usable again with a completely free software tool-chain.

EasyDyn is a C++ library for the simulation of problems represented by second-order (or first-order) differential equations and, more particularly, multibody systems. For the simulation of multibody systems the script cagem (Computer-Aided Generation of Motion) is used to generate problem specific code of system's dynamic equations of motion form an analytic description of the system configuration (position and pose of each body) of the system. The description of the system is written in terms of the minimal coordinates (degrees of freedom) of the system thus eliminating the need to explicitly state (and handle) the constraints of the relative motion between the bodies. The equations of motion are analytically derived via the [principal of virtual work](https://en.wikipedia.org/wiki/Virtual_work) using the symbolic computation and simplification capabilities of the Computer Algebra System (CAS) running the script (in this case now Maxima). This combination of minimal coordinates and symbolically optimized equations generated into custom C++ code yields a very fast and computationally efficient means to simulate the behavior of multibody systems.


Usage
===============
The definition of the system to be simulated is written in the [Maxima language](http://maxima.sourceforge.net/docs/manual/maxima_toc.html#SEC_Contents) using helper functions for the formulation of the homogeneous transformations defining the bodies' positions and poses in dependence of the minimal coordinates  `q` (aka configuration variables or degrees of freedom). These helper functions are defined in the script file `cagem.mac`. Please take a look at the example files to get a feeling how this is done. A more complete manual may be written later. Meanwhile you may also refer to the manual supplied in the [download of EasyDyn](http://hosting.umons.ac.be/html/mecara/EasyDyn/EasyDyn-1.2.4.tgz) or [here](http://hosting.umons.ac.be/html/mecara/EasyDyn/EasyDyn.pdf).

The syntax of MuPAD and Maxima is very similar. The major difference is that definitions in Maxima are indicated by a colon (:) and completed by a semicolon (;). Further, the array index base was changed from zero to one. So, mass, inertia terms (Ixx, Iyy, Izz, Ixy, Ixz, Iyz), body pose (`T0G`) and minimal coordinates (`q`) all start with `mass[1]`, `Ixx[1]`, etc., `T0G[1]` for the first body resp. `q[1]` for the first configuration variable. Finally beware that in maxima the dot (.) is used for matrix products, the commonly used asterisk will perform elementwise multiplication and thus give you worng results!

The problem-specific code is generated with the function `cagem` also defined in the script file `cagem.mac`. The actual code is written using the the maxima package `gentran`. Unfortunately, the gentran package as supplied by the current maxima release is not fully functional. Therefore you will have to download the package from [my repo](https://github.com/jgeisler0303/maxima) and replace the files in the `maxima/share/contrib/gentran` folder of your maxima installation with the files from my repo. On my Linux computer I got gentran only to work using the [Steel Bank Common Lisp (SBCL)](http://www.sbcl.org/) implementation. Unfortunately, this meant that I had to recompile maxima from source. On Windows, SBCL is the default Lisp implementation supplied with the [wxMaxima](http://andrejv.github.io/wxmaxima/) installation.

The concrete workflow is as follows:

1. Write the system definition in the maxima script language defining all relevant information expected by the generator script.
1. Start Maxima or wxMaxima.
1. Load the cagem script: `load("cagem.mac");` (assuming you are in the folder where the script resides, otherwise add the path to the argument of the load command).
1. Run the generation script: `cagem("path_to_system_def/filename_of_system_def.mac");`. The custom code will be generated in the current folder with the name `filename_of_system_def.cpp`.
1. Compile the custom code and link it to the EasyDyn library e.g. using gcc: `g++ -Ipath_to_EasyDyn/include filename_of_system_def.cpp -Lpath_to_EasyDyn_Library -lEasyDyn -lgsl -lgslcblas -lm -o filename_of_system_def`.
1. Now you can run the simulation of your system according to the initial conditions and duration specified in the system definition script. This will create an ASCII file `filename_of_system_def.res` that holds the time series of the system configuration and their time derivatives. Additionally the files `filename_of_system_def.vol` and `filename_of_system_def.van` are generated which hold information that can be used to animate the motion of the bodies of the system using the [EasyAnim Tool](http://hosting.umons.ac.be/html/mecara/EasyDyn/EasyAnim-1.3.1.tgz) (also available with some usage instructions [here](https://www.mbdyn.org/?Software_Download___EasyAnim&search=easydyn)).
1. If you also generated a [GNU Plot](http://www.gnuplot.info/) script (by setting `PLOT: 1;` in the system definition script) and have GNU plot installed you may further create some neat PostScript figures of the simulated evolution of the system configuration by doing `gnuplot filename_of_system_def.plt`.

Note that for linking to the EasyDyn library you will also need to install the [GSL library](https://www.gnu.org/software/gsl/) and make its path known to gcc. On a Ubuntu system e.g. you may do this by `sudo apt-get install libgsl0-dev`. For further information please refer to chapter 1.3 of the EasyDyn manual.

Note also that the `bike3D.mac`-example will not run as generated by the cagem script. In order for the simulation to succeed you have to change the `main` function in the generated file `bike3D.cpp` to the code provided in the EasyDyn download `examples/cagem` folder.

