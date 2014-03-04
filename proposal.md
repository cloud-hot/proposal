# <center>Condition Variables for RTEMS</center>
###### <center>Google Summer of Code Program 2014 Project Proposal</center>###### <center>json.a.zhang@gmail.com</center>
###### <center>Sichuan University</center>###### <center>IRC name: Json</center>
## Project AbstractThis project is to implement condition variables Abbreviated to `CV` for RTEMS Classic API and score component. In RTEMS there is already a CV implementation for PThread API. So this project’ goal is to support CV for RTEMS Classic API which should be based on consistent implementation as PThread API. In order to achieve this goal the first step should implement a basis CV core under RTEMS supercore. The task also should consider to avoid potential priority inversion issues of CV in combination with priority based schedulers and also should consider to work under SMP environment.## Project Description### Overview of Condition VariablesCondition variables are a powerful synchronization mechanism which provides a different type of synchronization than locking mechanisms like mutexes. For instance, a mutex is used to cause other threads to wait while the thread holding the mutex executes code in a critical section. In contrast, a condition variable is typically used by a thread to make itself wait until an expression involving shared data attains a particular state.

In general, condition variables are more appropriate than mutexes for situations involving complex condition expressions or scheduling behaviors. For instance, condition variables are often used to implement thread-safe message queues, which provide a `producer/consumer` communication mechanism for passing messages between multiple threads. In this use-case, condition variables block producer threads when a message queue is `full` and block consumer threads when the queue is `empty`.

### RTEMS POSIX API of Condition Variables
	
Now the implementation of RTEMS POSIX CV is based on score thread queue API and posix mutex API. If the score CV is implemented this part of code should based on score CV API.

### RTEMS score Condition Variables

#### score CV API

	void _CORE_condition_variable_Initialize(
				CORE_condition_variable_Control *the_cv,
				CORE_condition_variable_Attributes *the_cv_attributes);

	void _CORE_condition_variable_wait(
				CORE_condition_variable_Control *the_cv,
				CORE_mutex_Control *the_mutex);
				
	void _CORE_condition_variable_timeoutwait(
				CORE_condition_variable_Control *the_cv,
				CORE_mutex_Control *the_mutex,
				Watchdog_Interval _timeout);

	void _CORE_condition_variable_signal(
				CORE_condition_variable_Control *the_cv);

	void _CORE_condition_variable_broadcast(
				CORE_condition_variable_Control *the_cv);

#### The CV programming idiom

- Waiting for a resource

	<pre><code>
	Acquire( mutex ); 
	while ( no resource )		wait( mutex, cond );	...	(use the resource)	...	Release( mutex);	</pre></code>

- Make a resource available

	<pre><code>
	Acquire( mutex );
	...	(make resource available)	...
	Signal( cond );	/* or Broadcast( cond );	Release( mutex);
	</pre></code>

#### The CV Primitives

- cv_wait( mutex, cond )

	<pre><code>
	Enter the critical section	(min busy wait)	Release mutex	Put my TCB on cond’s queue	Call scheduler	Exit the critical section	. . . (blocked)	Waking up:		Acquire mutex
		Resume

- cv_signal( cond )

	<pre><code>
	Enter the critical section	(min busy wait)	Wake up a TCB in cond’s queue	Exit the critical section
	</pre></code>

#### The design of CV for RTEMS


### RTEMS Classic API of Condition Variables

## Benefits to RTEMSThis project will add a new useful system synchronization primitives which will be a more appropriate solution than mutexes or semaphore for situations involving complex condition expressions or scheduling behaviors. And also it can provide a more appropriate solution to such a type of [bug](https://www.rtems.org/bugzilla/show_bug.cgi?id=1467).## Project Deliverables- 19 May (coding begins) 	- Collect all the reference materials and discuss with the mentor to determine the design strategy which includes how to avoid potential priority inversion. - 12 June 
	- .- 27 June (Midterm Evaluation) 
	- .- 15 July 
	- .
	- 10 August 
	- .- 22 August 
	- Integrate all the source code and related documents to RTEMS, if possible merge the patch to mainline.## Proposed Schedule- 21 April - 19 May (Preparation)	- Discuss with the mentor to determine the design strategy, learn more about the details of the project, get familiar with the structure of the reference source code, and understand the related reference materials and source code.- 20 May - 12 June (Design-CV)	- . - 13 June - 27 June (Code-CV)	- .- 28 June - 15 July (design+code for lock debug tool)	- .- 16 July - 10 August ()	- .
- 11 August - 22 August (integration)	- Integrate all the codes and documents for final deliverable submission.## Continued Involvement## Future Improvements## Relevant Background Experience- Participate in the development of porting ucos kernel to arm and powerpc board and also developing some simple drivers for RTEMS like uart, led and so on.- Participate Linux kernel and driver development, the main work include: linux kernel porting, Bootloader U-BOOT porting.## Personal##### Where do you go to school?  What level are you?  What interests you about embedded systems?  Now i am currently working on my Master's in Embedded System at Sichuan University, in china. I partispated in GSOC2010 for RTEMS and the project is [RTEMS Sequenced Initialization and RTEMS System Events](http://docs.google.com/Doc?id=ddmthbb5_2g2mpc8ff), through this event i learned much more things and give me more confidence to contribute myself to open source. And in my spare time i play with some project in [my github](https://github.com/cloud-hot). Now I am very interested in the OS kernel and multi-core technology on embedded system.##### How did you learn about RTEMS?The RTEMS is my first open source project involved and from RTEMS i learned many knowledge about embedded system.## Experience
##### Free Software Experience/Contributions:
- [GSOC 2010 RTEMS Sequenced Initialization and RTEMS System Events](http://docs.google.com/Doc?id=ddmthbb5_2g2mpc8ff).
- [My personal github repo](https://github.com/cloud-hot).##### Language Skill Set- Being proficient in C programming and being familiar with bash script programming and lua, ruby, python programming- Being familiar with Linux, NetBSD, FreeBSD OS and RTEMS embedded operation system- Being familiar with the ARM, MIPS and PowerPC architecture and according assembly language- Having a basic knowledge of SPARC architecture and its assembly language- Being familiar with version control system --- git.