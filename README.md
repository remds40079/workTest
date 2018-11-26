# How to support SUT reset

In order to guarantee the integrity of the test case, we consider the case is atomic, and invoke reset operation after mptf.Close is called. Finish_status.json file at \Config folder has been added to hold the last executed case. The json file identifies the uniqueness of the test case with the three attibutes Name\Type\Timestamp, and determines whether it has finished executing with finished:no\yes.

* The Finish_status.json file
  * {"last": {"finished": "no", "type": "suite", "name": "nt32_test_cases2", "timestamp": "2017_09_21__06_47_50"}}
 
* The test case to reset the boot
  * obj = mptf.mptf(logpath)
  * obj.Close()
  * obj.Input('reset' + mptf.Enter) 

-k was added to represent keep up to run the last test case.Add startup.nsh 
