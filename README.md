# kitchen-extensive-terraform
A detailed review of the features of Kitchen-Terraform.

# Purpose
This repositoy is based on tutorial https://newcontext-oss.github.io/kitchen-terraform/tutorials/extensive_kitchen_terraform.html . Amd that tutorial provides a detailed review of the features of Kitchen-Terraform by developing a Terraform module which configures resources on the [Amazon Web Services - (AWS)](https://aws.amazon.com/) platform.

Kitchen-Terraform is assumed to be installed on the development system according to the instructions in the [Kitchen-Terraform ReadMe](https://github.com/newcontext-oss/kitchen-terraform/blob/master/README.md#installation).


Note - it is NOT A REPLACEMNT and not a REPEAT of the original tutrial. You still should reference that step-bystep first. Insterad, what this repository contains - is the actual code,some logs, plus description of some changes that you need to do to run it on the modern  Terraform (v0.12.9) and KitchenCI (v1.25.0) with Kitchen-Terraform gem 5.1.1. 

# Notes

Follow Tutorial first from here: https://newcontext-oss.github.io/kitchen-terraform/tutorials/extensive_kitchen_terraform.html, 

Difference from original Tutorial :
1. I've used `rbenv` to work with the local version of Ruby 2.3.1 (as current version in macOS Mojave is 2.6.4)
That's why there is `.ruby-version` file in the repo
2.  As I have Terraform of version 0.12.09 I've replaced tags definition everywhere. 
- Replace this : 
    ```terraform
    tags {
    ```
- With such code
    ```terraform
    tags = {
    ```
3. Also `main.tf` should contain other version restrictions, mine file starts as follows : 
    ```terraform
    terraform {
    # The configuration is restricted to Terraform versions supported by
    # Kitchen-Terraform
    required_version = ">= 0.11.4, <= 0.12.9" # <-- this changed >
    }

    provider "aws" {
    version = "~> 2.31" # <-- this changed >
    }

    provider "random" {
    version = "~> 2.2" # <-- this changed >
    }
    ```
4. They missing in the tutorial populating files with OS-specific attributes. e.g. 2 additional files must present in `test/integration/extensive_suite/` :
`centos_attributes.yml` with content : 
    ```
    instances_ami_operating_system_name: centos
    ```
    and `ubuntu_attributes.yml` with contents : 
    ```
    instances_ami_operating_system_name: ubuntu
    ```
    Otherwise you are goign to have some fails : 
    ```
    >>>>>> ------Exception-------
    >>>>>> Class: Kitchen::ActionFailed
    >>>>>> Message: 1 actions failed.
    >>>>>>     Verify failed on instance <extensive-suite-ubuntu>.  Please see .kitchen/logs/extensive-suite-ubuntu.log for more details
    >>>>>> ----------------------
    >>>>>> Please see .kitchen/logs/kitchen.log for more details
    >>>>>> Also try running `kitchen diagnose --all` for configuration
    ```
    And corresponding error in log : 
    ```
    E, [2019-10-09T14:32:08.676632 #33979] ERROR -- extensive-suite-ubuntu: ---Nested Exception---
    E, [2019-10-09T14:32:08.676651 #33979] ERROR -- extensive-suite-ubuntu: Class: Kitchen::Terraform::Error
    E, [2019-10-09T14:32:08.676670 #33979] ERROR -- extensive-suite-ubuntu: Message: remote: Cannot find input file 'test/integration/extensive_suite/ubuntu_attributes.yml'. Check to make sure file exists.
    ```
5. Kitchen still can produce errors in the very end when parsing running time, I cant' confirm yet this 100%%, that's outside the scope of tutorial. But the tests are passing :
```
Verifying

Profile: Extensive Kitchen-Terraform (extensive_suite)
Version: 0.1.0
Target:  local://

  ✔  state_file: 0.12.9
     ✔  0.12.9 is expected to match /\d+\.\d+\.\d+/
  ✔  inspec_attributes: static terraform output
     ✔  static terraform output is expected to eq "static terraform output"
     ✔  static terraform output is expected to eq "static terraform output"


Profile Summary: 2 successful controls, 0 control failures, 0 controls skipped
Test Summary: 3 successful, 0 failures, 0 skipped
```
And looks like later in the execution process, at the very end, we hitting KitchenCI bug  
```
>>>>>> ------Exception-------
>>>>>> Class: Kitchen::ActionFailed
>>>>>> Message: 1 actions failed.
>>>>>>     Verify failed on instance <extensive-suite-ubuntu>.  Please see .kitchen/logs/extensive-suite-ubuntu.log for more details
>>>>>> ----------------------
>>>>>> Please see .kitchen/logs/kitchen.log for more details
>>>>>> Also try running `kitchen diagnose --all` for configuration
```
Where logs bearing these lines : 
```
E, [2019-10-09T15:02:21.590012 #35830] ERROR -- extensive-suite-ubuntu: ------Exception-------
E, [2019-10-09T15:02:21.590099 #35830] ERROR -- extensive-suite-ubuntu: Class: Kitchen::ActionFailed
E, [2019-10-09T15:02:21.590121 #35830] ERROR -- extensive-suite-ubuntu: Message: remote: no implicit conversion of Array into String
E, [2019-10-09T15:02:21.590141 #35830] ERROR -- extensive-suite-ubuntu: ----------------------
E, [2019-10-09T15:02:21.590161 #35830] ERROR -- extensive-suite-ubuntu: ------Backtrace-------
E, [2019-10-09T15:02:21.590180 #35830] ERROR -- extensive-suite-ubuntu: /Users/andrii/.rbenv/versions/2.3.1/lib/ruby/gems/2.3.0/gems/kitchen-terraform-4.9.0/lib/kitchen/verifier/terraform.rb:96:in `rescue in call'
E, [2019-10-09T15:02:21.590199 #35830] ERROR -- extensive-suite-ubuntu: /Users/andrii/.rbenv/versions/2.3.1/lib/ruby/gems/2.3.0/gems/kitchen-terraform-4.9.0/lib/kitchen/verifier/terraform.rb:90:in `call'
...
```
going to check later what's happening

# Todo
- [ ] find out why KitchenCI failing on some test with "remote: no implicit conversion of Array into String"

# Done
- [x] intro readme
- [x] - Configure Amazon Web Services
- [x] - Create InSpec Profile
- [x] - Generate SSH Key
- [x] - Create Test Fixture Terraform Configuration
- [x] - Create Terraform Module
- [x] - Create Test Kitchen Configuration
- [x] - Test Terraform Module
- [x] - write the required changes to the tutorial
