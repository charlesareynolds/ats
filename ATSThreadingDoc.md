# Draft Proposal for Handling Multi-threaded Applications in ATS #

Many client applications of ATS can run in a multi-threaded way.  These applications are already parallel (through the use of MPI) but have an additional capability to support multiple threads on each MPI process.  ATS needs to support the testing of applications that exhibit this "hybrid" parallelism.

# ATS Test Concepts #

In order to support tests that run in this hybrid way (MPI + threads), ATS needs the following concepts:

**Number** **of** **processes** (np): This is the familiar concept of number of MPI ranks.  There is one user level process per MPI rank.  On a typical modern HPC node with some collection of multi-core processors (e.g. two 8-core Intel SandyBridge processors), we assign one process to a core. Default value is 1.

**Number** **of** **threads** (nt): The number of logical threads per process.  Implicit in this definition is the idea that one can't specify different numbers of threads for each user level process in the parallel test.  Each process will be launched and scheduled such that it has resources for nt\*np total threads. Default value is 1.

**Threads** **per** **core** (threadsPerCore): This feature is here for systems that support Simultaneous Multi-Threading (SMT).  In a modern multi-core architecture with superscalar pipelining, SMT allows for the execution of multiple hardware threads on a single core.  Some architectures feature a full set of functional units and register sets for one or more additional hardware threads, but the basic requirement is the ability to dispatch instructions from multiple hardware threads to the superscalar core.  Default value is 1.

**ATS** **Test** **Resources:**

ATS assumes that a node on a given HPC resource (i.e. a machine with a specified Machine file) has a certain number of "processors" per node.  This might be 8 separate single core processors, 4 separate CPUs with 2 cores each, or 1 8-core CPU (like the new Intel Sandy Bridge).  All of these configurations would appear to ATS as a node with 8 processors.

**ATS Algorithm for Assigning Resources to a Test:**

Using the definitions above, a test is assigned resources as follows:

Let tpc =  (if threadsPerCore is specified for this test, then test.threadsPerCore.  Else, machine default threadsPerCore)

Let np = np value for this test.  If not specified, 1.

Let nt = nt value for this test.  If not specified, 1.

The number of "processors" this test will receive is N = (np\*nt)/tpc.

**Example #1:**

Some tests are to be run on a commodity Linux cluster.  A node consists of 3 CPUs, each with 4 cores.  The cores do not support SMT.

A test specifies 4 MPI tasks (np=4) and 2 threads per process (nt=2).  The test would be assigned (4\*2)/1 = 8 processors.

**Example #2:**

Some tests are to be run on a Blue Gene/Q machine, which supports SMT.  A custom Machine file was created to set the default value of threadsPerCore to 2, in order to try out the SMT features of the machine.

A test specifies 8 MPI tasks (np=8) and 4 threads per process (nt=4). The test would be assigned (8\*4)/2 = 16 processors, which is all the processors on a BG/Q node.

**SLURM Considerations:**

The value of np determines the integer value given to the -n argument of srun

The value of nt/threadsPerCore determines the integer value given to the  -c or --cpus-per-task argument of srun.

**"Wrap-around" Defaults:**

A Machine file may specify a max value for np, nt, or threadsPerCore (say maxNP, maxNT, maxThreadsPerCore).
If a test specifies -1 for nt, for example, nt is set to maxNT for that test.  The default value of maxNT would be npMax,
i.e. the number of processors per node.

Similarly, an application may wish to use all available SMT features of a machine, and so would set threadsPerCore to
-1, getting the value of maxThreadsPerCore for that machine.