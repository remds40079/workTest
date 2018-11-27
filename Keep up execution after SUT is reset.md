# Keep up execution after SUT is reset

The SUT will be reset frequently in firmware related test. The MPTF Test Suites/Test Suite can continue to be executed if previous test is interrupted by platform reset. 


# How 
* Make sure the MPTF will be launched after platform reset
  
  According to introduction from https://software.intel.com/en-us/articles/efi-shells-and-scripting, the startup.nsh on root level of a disk will be launched automatically when system is turned on. Note, the startup.nsh here is not what's inside the MpyTest folder. By modifying the startup.nsh like below to launch the MPTF.

  ```
  fs0:    
  cd MpyTest  
  startup.nsh -a [ia32 | x64] -k
  # The -k is not required. You can just write something like 'startup.nsh -a [ia32 | x64] [-s suite_name | -ss suites_name]. The MPTF will continue the execution of Test Suite|Test Suites if previous test is not finished
  ```
* Make the test case atomic 
  
  The atomic here means the Test Case shouldn't be interrupted by system reset. Please reset platform at the end of the Test Case after invoking `mptf.close()`. 

* What behind it
  
  The Finish_status.json under `MpyTest\Config` folder is introduced to record the test execution status. The json file identifies the uniqueness of the test case with the three attributes (Name, Type, Timestamp) and determines whether it has finished executing with finished:[no|yes]. Here is one sample of the Finish_status.json.

  ```
   {"last": {"finished": "yes", "type": "suite", "name": "nt32_test_cases2", "timestamp": "2017_09_21__06_47_50"}}
  ```
 
* Sample 
  
  A test case `m_helloworld` under the `Scripts` folder to reset platform:
  ```  
  import mptf
  
  def run(log_path):
    obj = mptf.mptf(log_path)
    obj.Input('cls' + mptf.ENTER)
    obj.Input('hello from micro python test framework...')
    obj.Input(mptf.UP + mptf.ENTER)
    obj.Pass('Say hello to world successfully...', True)
    obj.Close()
    obj.Input('reset' + mptf.Enter) 
  ```
  * Run the test Case `m_helloworld.py`:
  ```
    startup.nsh -a ia32 -c m_helloworld
  ```
  
  Add the test case to the test suite `nt32_reset_cases` defined in `Test_Suite.json` under the `Config` folder. And run the test case 10 times:
  ```
  {
    "test_suite": [
      { "name":"nt32_reset_cases",
        "run_times":10,
        "sequence":[
          {
            "name":"m_helloworld.py",
            "order": 1
          }
        ]
      },
      { "name":"nt32_test_cases",
        "run_times":5,
        "sequence":[
          {
            "name":"nt32_helloworld.py",
            "order": 1
          }
        ]
      }
    ]
  }
  ```
  * Run the test suite `nt32_reset_cases`:
  ```
    startup.nsh -a ia32 -s nt32_reset_cases
  ```
  Add the test suite and `nt32_test_cases` to the test suites `nt32_reset_suites` defined in `Test_Suites.json` under `Config` folder: 
  ```
  {
    "test_suites": [
      {
        "name": "nt32_reset_suites",
        "test_suite": [
          {
            "name": "nt32_reset_cases",
            "order": 1
          },
          {
            "name": "nt32_test_cases",
            "order": 2
          }
        ]
      }
    ]
  }
  ```
  * Run the group of test suites `nt32_reset_suites`:
  ```
    startup.nsh -a ia32 -ss nt32_reset_suites
  ```
