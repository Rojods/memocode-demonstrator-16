# memocode-demonstrator-16

This is the companion repository for the MEMOCODE'23 submission n. 16.

Regardless if you are checking this repository from a Linux or Windows OS, you need at least Java 11 to run the I-modules and E-modules.
We recommend using a versioning installer like [jabba](https://github.com/shyiko/jabba) and installing the lastest stable JVM possible.
The modules were tested with Amazon Coretto and the Graal JVM.

If you plan to run the simulink example, you must also have `matlab` callable in your PATH. This is typically done automatically by the MATLAB installer
on Windows, and there is good chance you know how to do it this already if you are running from Linux. 
Otherwise, follow [this link for windows](https://se.mathworks.com/matlabcentral/answers/94933-how-do-i-edit-my-system-path-in-windows)
or [this link for Linux](https://unix.stackexchange.com/questions/244941/how-to-add-my-matlab-to-path) to get a good start.
Sadly, here Google (and alternatives) is your friend; it is impossible to predict your exact setup.

## Windows

The main entry point is `orchestrator.exe`. You can use `orchestrator.exe -h` to see how to configure the run paths for each trial so that
they won't overlap. You call the orchestrator like:

    .\orchestration.exe .\sdf\small_explainable\a_sobel.hsdf.xml .\sdf\small_explainable\bus_small_platform.fiodl

And the tool should do the rest automatically. `fiodl` files are part of the YyyYyYyyYY framework and can be expected directly with any text editor; they are not binary files.

## Linux

The main entry point is `orchstrator`. The same logic from windows applies here.
