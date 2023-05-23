# memocode-demonstrator-16

This is the companion repository for the MEMOCODE'23 submission n. 16.

## Downloading the anonymous repository

Due to the size restrictions of GitHub (and therefore open4science), the zipped repository has been uploaded
to another website which is provided by MEMOCODE'23. 

[You are welcome to download the `zip` file here in order to avoiding downloading one file at a time from the anonymised repository.](https://hs-niederrhein.sciebo.de/s/QwUO8tUPztjKVrD)

## Requirements for running the demonstrators

Regardless if you are checking this repository from a Linux or Windows OS, you need at least Java 11 to run the I-modules and E-modules.
On Linux you can use the distrubition's package manager, e.g. `apt-get` or `dnf`, to install the JVM system wide.
On Windows you can likely find official installer on all major JVM distributions like [Oracle](https://www.oracle.com/java/technologies/downloads/).

We recommend using a versioning installer like [jabba](https://github.com/shyiko/jabba) and installing the lastest stable JVM possible.
The modules were tested with [Amazon Coretto](https://aws.amazon.com/corretto/?filtered-posts.sort-by=item.additionalFields.createdDate&filtered-posts.sort-order=desc) and the [Graal VM](https://www.graalvm.org/).

If you fail to meet the Java requirement, you might get an output like:

    [2023-05-22T14:01:17Z INFO ] Run directory is run
    [2023-05-22T14:01:17Z INFO ] Final output set to explored_and_integrated.fiodl
    [2023-05-22T14:01:28Z INFO ] Identified 0 decision model(s)
    [2023-05-22T14:01:28Z INFO ] Computed 0 dominant bidding(s)
    [2023-05-22T14:01:28Z INFO ] Starting exploration until completion.
    [2023-05-22T14:01:28Z INFO ] Finished exploration with 0 solution(s).

For the case studies where *we know* there should be identifications.
This is simply the orchestrator being fault tolerant and skipping problematic modules whenever possible.
Therefore, make sure that `java` is available in your machine and that it indeed 11 or higher. You can check this with:

    > java -version
    openjdk version "11.0.11" 2021-04-20
    OpenJDK Runtime Environment GraalVM CE 21.1.0 (build 11.0.11+8-jvmci-21.1-b05)
    OpenJDK 64-Bit Server VM GraalVM CE 21.1.0 (build 11.0.11+8-jvmci-21.1-b05, mixed mode, sharing)

If you plan to run the simulink example, you must also have `matlab` callable in your PATH. This is typically done automatically by the MATLAB installer
on Windows, and there is good chance you know how to do it this already if you are running from Linux. 
Otherwise, follow [this link for windows](https://se.mathworks.com/matlabcentral/answers/94933-how-do-i-edit-my-system-path-in-windows)
or [this link for Linux](https://unix.stackexchange.com/questions/244941/how-to-add-my-matlab-to-path) to get a good start.
Sadly, Google (and alternatives) is your friend here; it is impossible to predict your exact setup.

## Running the provided demonstrators

The orchestrator should **always** run at the top of this folder tree, which is also the XXxXxXx workspace.
If you move the orchestrator binary around or try to run it from another folder, it might not find the I-modules
and E-modules in their respective `imodules` and `emodules` folders. 

Be very mindful of the `run path` that the orchestrator is using. 
As the tool tries to be incremental due to the DSI techniques, 
if you run multiple case studies in the same `run path` there is a risk that you will hit corner cases where the orchestrator did not clean the workspace properly and composedly identifies design models that do not exist anymore.
If you _only_ add new inputs there is no problem; if you _change_ inputs there might be a problem.

Use the `--run-path` option of the orchestrator to enable multiple run folders side-by-side to avoid these problems.

Finally, use the `-p` or `--parallel-jobs` option with a number to increase the parallelisation of non-exploration
procedures. This is mainly done from the orchestrator side, so make sure you have a machine with enough resources to
support multiple concurrent processes running. If you don't, there is little to no value on using the `-p` option.
Note that parallel exploration is depends on the capabilities of the explorer being called. 
The choco solver currently uses no parallel techniques.

### Windows

#### 1) Avionics

You can run the Flight Function Information through,

    .\orchestrator.exe --run-path run_avionics_fif .\avionics\flight-information-function\case_study.fiodl

which should give you:

    [2023-05-23T12:55:28Z INFO ] Run directory is run_avionics_fif
    [2023-05-23T12:55:28Z INFO ] Final output set to explored_and_integrated.fiodl
    [2023-05-23T12:55:39Z INFO ] Identified 5 decision model(s)
    [2023-05-23T12:55:40Z INFO ] Computed 1 dominant bidding(s)
    [2023-05-23T12:55:40Z INFO ] Starting exploration until completion.
    [2023-05-23T12:55:45Z INFO ] Finished exploration with 1 solution(s).

This one solution is the single-core solution that satisfies all the timing requirements. 
You can open the results `fiodl` file in the `reversed` folder to check its contents, namely the `run_avionics_fif\reversed\reversed_0_YyyYyYyIdentificationModule.fiodl` file:

    systemgraph {
        vertex "60Hz"
        [execution::PeriodicStimulus, visualization::Visualizable]
        (activated)
        {
            "offsetDenominator": 1_l,
            "periodDenominator": 1000000_l,
            "offsetNumerator": 0_l,
            "periodNumerator": 16666_l
        }
        vertex "30Hz"
        [execution::PeriodicStimulus, visualization::Visualizable]
    ...

You can find in this design model file the periodic tasks that are "LoopingTasks", i.e. that execute a block of code forever. For example, we have "FormatSpeedVectorCalc":

    vertex "FormatSpeedVectorCalc"
    [...execution::CommunicatingTask, execution::LoopingTask...] ...

which is part of an arc that starts at the task and points to the scheduler responsible for scheduling, as obtained from the exploration results:

    edge [decision::AbstractScheduling] from "FormatSpeedVectorCalc" port "schedulers" to "CPM1_Core1_Runtime.CMP1_Core1_FP_Runtime_Scheduler" 

you could then inspect this scheduler in the file to get further insight on what type of scheduler it is, although the name is arguably self-descriptive:

    vertex "CPM1_Core1_Runtime.CMP1_Core1_FP_Runtime_Scheduler" [...platform::PlatformElem, platform::runtime::FixedPriorityScheduler...]

Note that the information in this explored `fiodl` model was already contained in `avionics\flight-information-function\case_study.fiodl`, except for the edges that connect the tasks to the schedulers, indicating the results of the exploration.

if you would wish to see more information as the orchestrator goes through its steps, you can issue

    .\orchestrator.exe -v DEBUG --run-path run_avionics_fif .\avionics\flight-information-function\case_study.fiodl

which gives the more detailed output:

    [2023-05-23T13:06:43Z INFO ] Run directory is run_avionics_fif
    [2023-05-23T13:06:43Z INFO ] Final output set to explored_and_integrated.fiodl
    [2023-05-23T13:06:43Z DEBUG] Copying input files
    [2023-05-23T13:06:43Z DEBUG] Registering external identification module with identifier C:\Users\XXX\gits\memocode-demonstrator-16\imodules\scala-bridge-devicetree.jar
    [2023-05-23T13:06:43Z DEBUG] Registering external identification module with identifier C:\Users\XXX\gits\memocode-demonstrator-16\imodules\scala-bridge-matlab.jar
    [2023-05-23T13:06:43Z DEBUG] Registering external identification module with identifier C:\Users\XXX\gits\memocode-demonstrator-16\imodules\scala-common.jar
    [2023-05-23T13:06:43Z DEBUG] Registering external identification module with identifier C:\Users\XXX\gits\memocode-demonstrator-16\imodules\scala-ourmdetool.jar
    [2023-05-23T13:06:43Z DEBUG] Registering external exploration module with identifier C:\Users\XXX\gits\memocode-demonstrator-16\emodules\scala-choco.jar
    [2023-05-23T13:06:51Z DEBUG] 5 total decision models identified at step 1
    [2023-05-23T13:06:53Z DEBUG] 5 total decision models identified at step 2
    [2023-05-23T13:06:53Z INFO ] Identified 5 decision model(s)
    [2023-05-23T13:06:55Z INFO ] Computed 1 dominant bidding(s)
    [2023-05-23T13:06:55Z DEBUG] Proceeding to explore PeriodicWorkloadToPartitionedSharedMultiCore with C:\Users\XXX\gits\memocode-demonstrator-16\emodules\scala-choco.jar
    [2023-05-23T13:06:55Z INFO ] Starting exploration until completion.
    [2023-05-23T13:06:57Z DEBUG] Found a new solution. Total count is 1.
    [2023-05-23T13:06:59Z DEBUG] Reverse identified the design model YyyYyYyDesignModel at run_avionics_fif\reversed
    [2023-05-23T13:06:59Z INFO ] Finished exploration with 1 solution(s).

#### 2) Simulink and device tree

Before starting this case study, make sure the [requirements](#requirements-for-running-the-demonstrators) are satisfied.
Otherwise, you'll likely get 0 explorable decision models as Matlab may not be callable from your PATH.

Once that is done, you can supply the required inputs file via:

    .\orchestrator.exe --run-path run_simudt .\simulink_devicetree\runtimes.yaml .\simulink_devicetree\test_model.slx .\simulink_devicetree\tile0.dts .\simulink_devicetree\tile1.dts

This one might take a good amount of time to run due to the huge overhead that starting Matlab/Simulnk brings: Matlab takes a lot of time to process every request to load `slx` file in its memory. This case study can take up to 6 minutes due to this external sluggishness. We highly recommend using the `-p` parallel capabilities here based on how many Matlab sessions your machine can start.

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

You can visualize it by opening any of the files with the KIELER extensions for visual code, which will then open a diagram view of the KGT file.

