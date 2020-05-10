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


```sas {.line-numbers}
/* SAS: FizzBuzz with Data Step */
	data FizzBuzz;
		do until(i = 10000);
			i+1;
			result = strip(ifc(mod(i,3)=0,ifc(mod(i,5)=0,'FizzBuzz','Fizz'),ifc(mod(i,5)=0,'Buzz','')));
			if missing(result)=0 then output;
		end;
	run;
```



```sas
/* setup a cas session */
	cas mysess;
	libname mycas cas sessref=mysess;
```



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



```sas
/* single thread version */
	data mycas.FizzBuzz / single=no;
		do until(i = 10000);
			i+1;
			result = strip(ifc(mod(i,3)=0,ifc(mod(i,5)=0,'FizzBuzz','Fizz'),ifc(mod(i,5)=0,'Buzz','')));
			if missing(result)=0 then output;
		end;
	run;
```



```sas
/* how many threads? hosts? */
	data mycas.seethreads / single=no;
		host=_hostname_;
		thread=_threadid_;
		put host thread;
	run;
```



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



```sas
/* do the same size work on each thread in cas */
	data mycas.FizzBuzzMPP / single=no;
		thread=_threadid_;
		i = (thread - 1) * 10000; /* start value for i on the thread */
		s = i + 10000; drop s; /* stop value for i on the thread */
		do until(i = s);
			i+1;
			result = strip(ifc(mod(i,3)=0,ifc(mod(i,5)=0,'FizzBuzz','Fizz'),ifc(mod(i,5)=0,'Buzz','')));
			if missing(result)=0 then output;
		end;
	run;
```



```sas
/* spread the work for &l over all threads */
%let threads = 370;
data mycas.FizzBuzzMPP / single=no;
	extras = mod(&l,&threads-1);
	part_size = int(&l/(&threads-1)); drop part_size;
	thread=_threadid_;

  i=(thread-1)*part_size;
  s=i+part_size; drop s;
  if thread = &threads then do;
    i=&l-extras;
    s=&l;
  end;
  do until(i=s);
      i+1;
      result = strip(ifc(mod(i,3)=0,ifc(mod(i,5)=0,'FizzBuzz','Fizz'),ifc(mod(i,5)=0,'Buzz',put(i,8.))));
      if result in ('Fizz','Buzz','FizzBuzz') then output;
  end;
run;
```


```sas
%let l=1000000; * make this larger and rerun to show scalability;
proc cas;
	datastep.runcode / code="" single='no';
run;
```


```sas
/* end the cas session */
	cas mysess clear;
```
