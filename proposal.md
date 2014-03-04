# <center>Condition Variables for RTEMS</center>

###### <center>Sichuan University</center>


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
	while ( no resource )

- Make a resource available

	<pre><code>
	Acquire( mutex );
	...
	Signal( cond );
	</pre></code>

#### The CV Primitives

- cv_wait( mutex, cond )

	<pre><code>
	Enter the critical section
		Resume

- cv_signal( cond )

	<pre><code>
	Enter the critical section
	</pre></code>

#### The design of CV for RTEMS


### RTEMS Classic API of Condition Variables

## Benefits to RTEMS
	- .
	- .
	- .
	
	- .
	- Integrate all the source code and related documents to RTEMS, if possible merge the patch to mainline.



