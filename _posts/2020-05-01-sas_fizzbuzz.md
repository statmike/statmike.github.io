---
title: 'Easy Threading with SAS Viya'
date: 2020-05-02 00:00:00
description: An example of simple threaded programming with SAS Viya's CASL language
featured_image: '/images/blog/fizzbuzz/fizzbuzz.png'
tags: types-of-computing sas sas-viya casl
---

## in progress

Preface
- Introduce the reason and the challenge
- Motivate concurrency
- Incremental approach
    - logic
    - logic in sas code
    - SAS 9
    - SAS Viya - CAS single threaded
    - SAS Viya - CAS concurrent
    - SAS Viya - CAS concurrent spread out
    - SAS Viya - CASL
    - Python - SWAT to CASL
- Other uses!

---

![](/images/blog/fizzbuzz/fizzbuzz.png)


general logic
```
Loop over positive integers
	if divisble by 3 then fizz
	if divisible by 5 then buzz
	if divisible by 3 & 5 then fizzbuzz
```

efficiency
```
Loop over positive integers
	if divisible by 3 then fizz
		if divisible by 5 then fizzbuzz
	else if divisible by 5 then buzz
```

sas version of the logic
```sas
do until(i = 10000);
	i+1
	/* if divide by 3 */
	ifc(mod(i,3)=0,
		/* else if divide by 5 then fizzbuzz, else fizz */
		ifc(mod(i,5)=0,'FizzBuzz','Fizz'),
	/* id divide by 5 then buzz */
	ifc(mod(i,5)=0,'Buzz',
	/* else nothing '' */
	''))
end;									
```

sas version condenced and output
```sas
do until(i = 10000);
	i+1;
	result = strip(ifc(mod(i,3)=0,ifc(mod(i,5)=0,'FizzBuzz','Fizz'),ifc(mod(i,5)=0,'Buzz','')));
	if missing(result)=0 then output;
end;
```

FizzBuzz with SAS
```sas
/* SAS: FizzBuzz with Data Step */
	data FizzBuzz;
		do until(i = 10000);
			i+1;
			result = strip(ifc(mod(i,3)=0,ifc(mod(i,5)=0,'FizzBuzz','Fizz'),ifc(mod(i,5)=0,'Buzz','')));
			if missing(result)=0 then output;
		end;
	run;
```


Start a SAS Viya CAS session
```sas
/* setup a cas session */
	cas mysess;
	libname mycas cas sessref=mysess;
```


FizzBuzz with SAS Viya CAS - single thread
```sas
/* single thread version */
	data mycas.FizzBuzz / single=yes;
		do until(i = 10000);
			i+1;
			result = strip(ifc(mod(i,3)=0,ifc(mod(i,5)=0,'FizzBuzz','Fizz'),ifc(mod(i,5)=0,'Buzz','')));
			if missing(result)=0 then output;
		end;
	run;
```


FizzBuzz with SAS Viya CAS - all threads
```sas
/* many thread version */
	data mycas.FizzBuzz / single=no;
		do until(i = 10000);
			i+1;
			result = strip(ifc(mod(i,3)=0,ifc(mod(i,5)=0,'FizzBuzz','Fizz'),ifc(mod(i,5)=0,'Buzz','')));
			if missing(result)=0 then output;
		end;
	run;
```


How many computing threads are available to SAS Viya CAS in this environment?
```sas
/* how many threads? hosts? */
	data mycas.seethreads / single=no;
		host=_hostname_;
		thread=_threadid_;
		put host thread;
	run;
```

Log Results
```sas
NOTE: Running DATA step in Cloud Analytic Services.
sas-programming 3
sas-programming 1
sas-programming 7
sas-programming 2
sas-programming 4
sas-programming 5
sas-programming 6
sas-programming 8
NOTE: The table seethreads in caslib CASUSER(sasdemo) has 8 observations and 2 variables.
NOTE: DATA statement used (Total process time):
      real time           0.03 seconds
      cpu time            0.01 seconds
```


FizzBuzz on SAS Viya CAS - all threads doing unique work
```sas
/* do work on each thread in cas */
	%LET fbsize = 10000;
	data mycas.FizzBuzzMPP / single=no;
		thread=_threadid_;
		i = (thread - 1) * &fbsize; /* start value for i on the thread */
		s = i + &fbsize; drop s; /* stop value for i on the thread */
		do until(i = s);
			i+1;
			result = strip(ifc(mod(i,3)=0,ifc(mod(i,5)=0,'FizzBuzz','Fizz'),ifc(mod(i,5)=0,'Buzz','')));
			if missing(result)=0 then output;
		end;
	run;
```


FizzBuzz on SAS Viya CAS - all threads doing unique work
```sas
/* spread the work over all threads */
%LET threads = 370;
%LET fbsize = 10000;
data mycas.FizzBuzzMPP / single=no;
	extras = mod(&fbsize,&threads-1); drop extras;
	part_size = int(&fbsize/(&threads-1)); drop part_size;
	thread=_threadid_;

	i = (thread - 1) * part_size;
	s = i + part_size; drop s;
	if thread = &threads then do;
		i = &fbsize - extras;
		s = &fbsize;
	end;
	do until(i = s);
		i+1;
		result = strip(ifc(mod(i,3)=0,ifc(mod(i,5)=0,'FizzBuzz','Fizz'),ifc(mod(i,5)=0,'Buzz','')));
		if missing(result)=0 then output;
	end;
run;
```


FizzBuzz on SAS Viya CAS with CASL code from PROC CAS
```sas
%LET threads = 370;
%LET fbsize=10000;
proc cas;
	dscode="
		data FizzBuzzMPP;
			extras = mod(&fbsize,&threads-1); drop extras;
			part_size = int(&fbsize/(&threads-1)); drop part_size;
			thread=_threadid_;

			i = (thread - 1) * part_size;
			s = i + part_size; drop s;
			if thread = &threads then do;
				i = &fbsize - extras;
				s = &fbsize;
			end;
			do until(i = s);
				i+1;
				result = strip(ifc(mod(i,3)=0,ifc(mod(i,5)=0,'FizzBuzz','Fizz'),ifc(mod(i,5)=0,'Buzz','')));
				if missing(result)=0 then output;
			end;
		run;";
	datastep.runcode / code=dscode single='no';
run;
```

End SAS Viya CAS session
```sas
/* end the cas session */
	cas mysess clear;
```

Running from Python Via SWAT
```python
import swat
```