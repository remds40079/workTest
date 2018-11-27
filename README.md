# How to support SUT reset

  In order to guarantee the integrity of the test case, we consider the case is atomic, and invoke reset operation after mptf.Close is called. Finish_status.json file at \Config folder has been added to hold the last executed case. The json file identifies the uniqueness of the test case with the three attibutes Name\Type\Timestamp, and determines whether it has finished executing with finished:no\yes.

  * The Finish_status.json file records the status of the last run case (MPTF:\MpyTest\Config\Finish_Status.json)
```
{"last": {"finished": "no", "type": "suite", "name": "nt32_test_cases2", "timestamp": "2017_09_21__06_47_50"}}
```
 
  * A test case to reset the boot(MPTF:\MpyTest\Scripts\reset_test.py)
```  
   obj = mptf.mptf(logpath)
   obj.Close()
   obj.Input('reset' + mptf.Enter) 
```

  Since startup.nsh in the root directory of the U disk will be automatically run after reboot, these sentences are added in this file. Make sure it goes into the MpyTest folder on the MPTF disk, and then runs the test case that was not finished last time. -k was added to represent keep up run the last test case. Add startup.nsh 
  * The startup.nsh (MPTF:\startup.nsh)
``` 
   fs0:
   cd \MpyTest
   startup.nsh -a [ia32 | x64] -k
```

  
