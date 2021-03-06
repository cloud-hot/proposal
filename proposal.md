# <center>Condition Variables for RTEMS</center>
###### <center>Google Summer of Code Program 2014 Project Proposal</center>###### <center>json.a.zhang@gmail.com</center>
###### <center>Sichuan University</center>###### <center>IRC name: Json</center>
## Project AbstractThis project is to implement condition variables Abbreviated to `CV` for RTEMS Classic API and score component. In RTEMS there is already a CV implementation for PThread API. So this project’ goal is to support CV for RTEMS Classic API which should be based on consistent implementation as PThread API. In order to achieve this goal the first step should implement a basis CV core under RTEMS supercore. The task also should consider to avoid potential priority inversion issues of CV in combination with priority based schedulers and also should consider to work under SMP environment.## Project Description### Overview of Condition VariablesCondition variables are a powerful synchronization mechanism which provides a different type of synchronization than locking mechanisms like mutexes. For instance, a mutex is used to cause other threads to wait while the thread holding the mutex executes code in a critical section. In contrast, a condition variable is typically used by a thread to make itself wait until an expression involving shared data attains a particular state.

In general, condition variables are more appropriate than mutexes for situations involving complex condition expressions or scheduling behaviors. For instance, condition variables are often used to implement thread-safe message queues, which provide a `producer/consumer` communication mechanism for passing messages between multiple threads. In this use-case, condition variables block producer threads when a message queue is `full` and block consumer threads when the queue is `empty`.

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

### Status of Condition Variables support in RTEMS

Now there is the only posix CV implementation in RTEMS which is based on score thread queue API and posix mutex API. If the score CV is implemented this part of code should based on score CV API. So the task contains the following works:

- Refactor the POSIX CV to a supercore implementation and pass posix CV testcase.
- Create a basic CV implementation for classic API and add related testcase.
- Improve classic CV capabilities, such as dealing with priority inversion.

#### Refact the posix CV implementation into score

- Refine the related struct for score CV like struct `POSIX_Condition_variables_Control` in posix.
- Reuse the CV implementation mechanism in posix, like mutex and thread queue. But replace its api from posix into score API.

#### score CV API

	void _Condition_variable_Initialize(
				Condition_variable_Control *the_cv,
				Condition_variable_Attributes *the_cv_attributes);

	void _Condition_variable_Wait(
				Condition_variable_Control *the_cv,
				CORE_mutex_Control *the_mutex);

	void _Condition_variable_Timeoutwait(
				Condition_variable_Control *the_cv,
				CORE_mutex_Control *the_mutex,
				Watchdog_Interval _timeout);

	void _Condition_variable_Signal(
				Condition_variable_Control *the_cv);

	void _Condition_variable_Broadcast(
				Condition_variable_Control *the_cv);

#### classic CV API

	void _Condition_variable_Manager_initialization(void);
	rtems_status_code rtems_condition_variable_create(
				rtems_name          name,
				uint32_t            count,
				rtems_attribute     attribute_set,
				rtems_task_priority priority_ceiling,
				rtems_id            *id);
	rtems_status_code rtems_condition_variable_delete(
				rtems_id       id);
	rtems_status_code rtems_condition_variable_wait(
				rtems_id       sem_id,
				rtems_id       cv_id,
				rtems_option   option_set,
				rtems_interval timeout)；
	rtems_status_code rtems_condition_variable_signal(
				rtems_id       id);
	rtems_status_code rtems_condition_variable_broadcast(
				rtems_id       id);


#### Test case for score CV and classic CV API

- test whether the api is valid
	- psxcond01 is this type test case for posix CV API
- test whether the function is valid
	- psx10 is this type test case for posix CV API.
- test performance of CV
	- psxtmcond01 - psxtmcond10 is this type test case for posix CV API.

#### dealing with priority inversion for CV

Because when a high priority task waiting for a CV encounters a priority inversion the mutex Priority Inheritance cannot help, because the higher- priority task waiting for the condition to become true drops the mutex lock before suspending through a wait operation on the condition variable, and the lower-priority task(s) that need to progress in their computations so as to perform the notify operation on the condition variable do not hold any mutex lock while they are computing.

There is a implementation of CV priority inheritance we can refer to [it](http://rtsim.sssup.it)

##### References:
1. [Priority Inheritance on Condition Variables](http://ertl.jp/~shinpei/conf/ospert13/OSPERT13_proceedings_web.pdf#page=46)
2. [slides](http://ertl.jp/~shinpei/conf/ospert13/slides/TommasoCucinotta.pdf)

## Benefits to RTEMS

This project will add a new useful system synchronization primitives which will be a more appropriate solution than mutexes or semaphore for situations involving complex condition expressions or scheduling behaviors. And also it can provide a more appropriate solution to such a type of [bug](https://www.rtems.org/bugzilla/show_bug.cgi?id=1467).## Project Deliverables- 19 May (coding begins) 	- Collect all the reference materials and discuss with the mentor to determine the design strategy which includes how to avoid potential priority inversion. - 12 June 
	- Provide the definition of the data type for score CV
	- Provide the definition of function API for score CV

- 27 June (Midterm Evaluation)
	- Provide the compilable and runable source code of score CV
	- provide the compilable and runable source code of posix CV based on score CV.
	- Pass all posix CV test case

- 15 July
	- Provide score CV API valid test case
	- Provide score CV function valid test case
	- Provide the compilable and runable source code of classic CV based on score CV.

- 10 August
	- Provide classic CV API valid test case
	- Provide classic CV function valid test case
	- Improve the classic CV capabilities, such as dealing with priority inversion.

- 22 August
	- Integrate all the source code and related documents to RTEMS, if possible merge the patch to mainline.

## Proposed Schedule

- 21 April - 19 May (Preparation)

	- Discuss with the mentor to determine the design strategy, learn more about the details of the project, get familiar with the structure of the reference source code, and understand the related reference materials and source code.- 20 May - 12 June (Design-CV)	- Define the data type and function API for score CV
	- Refactor the posix CV code to score CV. It includes understand posix CV design and implement CV into score component with score related API and service.- 13 June - 27 June (Code-CV)
	- Code score CV
	- Implement the posix CV based on the score CV API
	- Make all the posix CV test case pass

- 28 June - 15 July (Code-CV)
	- Design the score CV API valid test case and score CV function valid test case
	- Implement the classic CV based on the score CV API

- 16 July - 10 August (Code-CV)
	- Design the classic CV API valid test case and classic CV function valid test case
	- Improve the classic CV capabilities, such as dealing with priority inversion.

- 11 August - 22 August (integration)
	- Integrate all the codes and documents for final deliverable submission.

## Continued Involvement

- With RTEMS SMP development the CV should be also updated, like using atomic lock instead of Giant lock.

- Integrate the CV into other RTEMS component.

## Future Improvements

- Improve the performance and compatibility for SMP environment

## Relevant Background Experience

- Participate in the development of porting ucos kernel to arm and powerpc board and also developing some simple drivers for RTEMS like uart, led and so on.
- Participate Linux kernel and driver development, the main work include: linux kernel porting, Bootloader U-BOOT porting.

## Personal

##### Where do you go to school?  What level are you?  What interests you about embedded systems?

Now i am currently working on my Master's in Embedded System at Sichuan University, in china. I partispated in GSOC2010 for RTEMS and the project is [RTEMS Sequenced Initialization and RTEMS System Events](http://docs.google.com/Doc?id=ddmthbb5_2g2mpc8ff), through this event i learned much more things and give me more confidence to contribute myself to open source. And in my spare time i play with some project in [my github](https://github.com/cloud-hot). Now I am very interested in the OS kernel and multi-core technology on embedded system.

##### How did you learn about RTEMS?

The RTEMS is my first open source project involved and from RTEMS i learned many knowledge about embedded system.

## Experience

##### Free Software Experience/Contributions:

- [GSOC 2010 RTEMS Sequenced Initialization and RTEMS System Events](http://docs.google.com/Doc?id=ddmthbb5_2g2mpc8ff).
	- After GSOC2010 we have choosen rtems as the platform of our project which is related to intelligent navigation car. The MCU is based on [LPC2300](http://www.zlgmcu.com/philips/NXP_ARM_2300.asp) and the board is designed by a china manufacturer zlgmcu. And in our project RTEMS is mainly responsible for task scheduling, peripheral control such as motor and sensor and bus communication such as modbus and can-bus(we also consider add wifi support for rtems, but i have few reference about this part, if anyone know please contact me). At the meantime i also participate to develope some linux driver and learn from lots of excellent projects from github.

- [My personal github repo](https://github.com/cloud-hot).

##### Language Skill Set

- Being proficient in C programming and being familiar with bash script programming and lua, ruby, python programming
- Being familiar with Linux, NetBSD, FreeBSD OS and RTEMS embedded operation system
- Being familiar with the ARM, MIPS and PowerPC architecture and according assembly language
- Having a basic knowledge of SPARC architecture and its assembly language
- Being familiar with version control system --- git.