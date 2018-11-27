# Keep up execution after SUT is reset

The SUT will be reset frequently in firmware related test. The MPTF Test Suites/Test Suite can continue to be executed if previous test is interrupted by platform reset. 


# How 
* Make sure the MPTF will be launched after platform reset
  
  According to introduction from https://software.intel.com/en-us/articles/efi-shells-and-scripting, the startup.nsh on root level of a disk will be launched automatically when system is turned on. Note, the startup.nsh here is not what's inside the MpyTest folder. By modifying the startup.nsh like below to launch the MPTF.

  ```
  fs0:    
  cd MpyTest  
  startup.nsh -a [ia32 | x64] -k
  # The -k is not required. You can just write something like 'startup.nsh -a [ia32 | x64] [-s suite_name | -ss suites_name]. The MPTF   will continue the execution of Test Suite|Test Suites if previous test is not finished
  ```
* Make the test case atomic 
  
  The atomic here means the Test Case shouldn't be interrupted by system reset. Please reset platform at the end of the Test Case after invoking `mptf.close()`. 

* What behind it
  
  The Finish_status.json under `MpyTest\Config` folder is introduced to record the test execution status. The json file identifies the uniqueness of the test case with the three attributes (Name, Type, Timestamp) and determines whether it has finished executing with finished:[no|yes]. Here is one sample of the Finish_status.json.

  ```
  {"last": {"finished": "yes", "type": "suite", "name": "nt32_test_cases2", "timestamp": "2017_09_21__06_47_50"}}
  ```
 
* Sample 
  
  A test case to reset the boot(MPTF:\MpyTest\Scripts\reset_test.py)
  ```  
   obj = mptf.mptf(logpath)
   obj.Close()
   obj.Input('reset' + mptf.Enter) 
  ```
