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

## Running the provided demonstrators

### Windows

The main entry point is `orchestrator.exe`. You can use `orchestrator.exe -h` to see how to configure the run paths for each trial so that
they won't overlap. You call the orchestrator like:

    .\orchestration.exe .\sdf\small_explainable\a_sobel.hsdf.xml .\sdf\small_explainable\bus_small_platform.fiodl

And the tool should do the rest automatically. `fiodl` files are part of the YyyYyYyyYY framework and can be expected directly with any text editor; they are not binary files.

### Linux

The main entry point is `orchestrator`. The same logic from windows applies here.

## Creating new design models

It is possible to create new design models so that XXxXxXx can identify its design space and solve it.
The demonstrators provided already contain the necessary information that one could replicate to create new design models,
but here is some extra information that can be given while maintaining anonymosity.

### SDF applications

Because of the YyyYyYyyYY framework, XXxXxXx directly consume [SDF3](https://www.es.ele.tue.nl/sdf3/manuals/xml/sdf/) XML specification files.
This applies especially to the application graphs. One must be careful when specifying the computational requirements of each actor, since
the identification rules in XXxXxXx must be able to match them with the computational provisions in the platform decision models available.

SDF applications can also be specified directly as `fiodl` files of the YyyYyYyyYY framework, but these are more general than SDF3, so we opt
to use SDF3 files directly for the sake of comprehension.

### Periodic Workload

The periodic workload design model is always given in `fiodl` files. 
The abstraction follows a separation between triggering and data propagation.
That is, the activation between different processes follows a rather intuitive multi-rate synchronous ideas, like:

    Task1 -> Task2

Implies that every time Task1 finishes, Task2 must start and finish sometime in the future. 
The notable difference here is the possibility of using upsampling and downsampling,

    Task1 -> Upsample by 10 -> Task2
or
    Task1 -> Downsample by 10 -> Task2

Now every 10th Task2 executes _after_ Task1, or, Task2 executes _after_ every 10th execution of Task1.
Note the after, which respects the direction of the activation flow between the processes.

The data propagation is decoupled with this, though still living inside the same "graph" design model:

    Task1 -> Data -> Task2

Now Task1 writes to Data and Task2 reads from Data, but it is not specified in which order, priority or even
the different rates of reading and write. Naturally, it is a waste to overwrite the data element without
reading it; keep in mind that this is a _data propagation graph_, which means that Data can come and go
from multiple producers and consumers, regardless of semantic coherence.

    Task1 -> Data -> Task2
    Task3 ---^  |----->Task4
                |---->Task5

The same graph remark idea applies to the activation discussion, where the name of the activation flow is called _trigger graph_.

In summary, if one would like to have the classic Liu & Layland's independent periodic task design model, it is enough
to not make any triggering connections. Likewise, the tasks can propagate zero data, and still activate each other.

The `flight-information-function` contains a design model without any triggering between the tasks, with only data propagation;
The `radar-aesa-function` contains a design model with data propagation _and_ triggering; which can be used as the template for new models.

### Platform

The platform used here is quite coarse-grain, but good enough to represent architectures at a system-level (according to [Gajski](https://www.bokus.com/bok/9781441905031/embedded-system-design/)).
You can have processing elements, memory elements and communication elements. They must be connected so that
every processing element reaches (directly or through a network of communication elements) a memory element.
If the memory elements are not private, the platform is identified to be a "tiled" architecture, and can be used for SDF solutions;
otherwise, it is simply a shared memory architecture.

Basically all `fiodl` files contain an example of how the platform is given.

### KGT files

Because of the YyyYyYyyYY framework, `kgt` files of the [KIELER](https://www.rtsys.informatik.uni-kiel.de/en/archive/kieler/welcome-to-the-kieler-project)
framework can be generated that enables one to see a bit better the `fiodl` files.
For the sake of blind-reviewing, the tool which performs this conversion is unfortunately not included in this repository, but some of the demonstrators
given are the results of the tool.

You can visualize it by opening any of the files with the KEILER extensions for visual code, which will then open a diagram view of the KGT file.

