# kitchen-extensive-terraform
A detailed review of the features of Kitchen-Terraform.

# Purpose
This repository is based on tutorial https://newcontext-oss.github.io/kitchen-terraform/tutorials/extensive_kitchen_terraform.html . Amd that tutorial provides a detailed review of the features of Kitchen-Terraform by developing a Terraform module that configures resources on the [Amazon Web Services - (AWS)](https://aws.amazon.com/) platform.

Kitchen-Terraform is assumed to be installed on the development system according to the instructions in the [Kitchen-Terraform ReadMe](https://github.com/newcontext-oss/kitchen-terraform/blob/master/README.md#installation).


Note - it is NOT A REPLACEMENT and not a REPEAT of the original tutorial. You still should reference that step-by-step first. Instead, what this repository contains - is the actual code, some logs, plus a description of some changes that you need to do to run it on the modern  Terraform (v0.12.9) and KitchenCI (v1.25.0) with Kitchen-Terraform gem 5.1.1. 

# Notes

Follow Tutorial first from here: https://newcontext-oss.github.io/kitchen-terraform/tutorials/extensive_kitchen_terraform.html, 

The difference from the original Tutorial :
1. I've used `rbenv` to work with the local version of Ruby 2.3.1 (as the current version in macOS Mojave is 2.6.4)
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
4. They missing in the tutorial populating files with OS-specific attributes. e.g. 2 additional files must present in [test/integration/extensive_suite/](test/integration/extensive_suite/) :
    - `centos_attributes.yml` with content : 
        ```
        instances_ami_operating_system_name: centos
        ```
    - `ubuntu_attributes.yml` with content : 
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
5. One more small bug connected wih the files above -  in handling attributes reading vs input variables in the [operating_systems.rb](test/integration/extensive_suite/controls/operating_system.rb) test. 
    It should be written down as follows : 
    ```ruby
    instances_ami_operating_system_name = attribute('instances_ami_operating_system_name', description: 'name of the operating system on the targeted host')
        control "operating_system" do
        desc "Verifies the name of the operating system on the targeted host"
        describe os.name do
            it { should eq instances_ami_operating_system_name}
        end
    end
    ```
    As original file just defines attribute attribute and its value for example in [ubuntu_attributes.yml](test/integration/extensive_suite/ubuntu_attributes.yml), but do not define them as variables in control. So, unless you apply the fix you can have this error : 
    ```shell
    Profile: Extensive Kitchen-Terraform (extensive_suite)
    Version: 0.1.0
    Target:  ssh://ubuntu@ec2-34-216-20-49.us-west-2.compute.amazonaws.com:22

    ×  operating_system: ubuntu
        ×  ubuntu 
        Profile 'extensive_suite' does not have an input with name 'instances_ami_operating_system_name'
    ✔  reachable_other_host: Host 52.37.61.236

        ✔  Host 52.37.61.236
        is expected to be reachable


    Profile Summary: 1 successful control, 1 control failure, 0 controls skipped
    Test Summary: 1 successful, 1 failure, 0 skipped
    ```
6. From Terrraform 0.12.07 the format of outputs for arrays had changed slighlty, actually simplifying code, so, I've edited `outputs.tf` in both fixtures and main repo location correspondingly to : 
    - Wrapper fixture : [test/fixtures/wrapper/outputs.tf](test/fixtures/wrapper/outputs.tf)
    ```terraform
    output "remote_group_public_dns" {
      description = "This output is used to obtain targets for InSpec"

      value = "${module.extensive_kitchen_terraform.remote_group_public_dns}" # no [] - brackets here
    }
    ```
    - Main one : [outputs.tf](outputs.tf)
    ```terraform
    and main `outputs.tf` :
    output "remote_group_public_dns" {
      description = "The list of public DNS names of the remote_group instances"
      value       = "${aws_instance.remote_group.*.public_dns}" # no [] - brackets here
    }
    ```
7. **macOS Mojave specific!!!** The way how default "ssh-keygen" in macOS Mojave configured ( Default is now to generate RFC4716 key pairs, which won't work with this environmen) you are going to experience a problem with SSH key, so append `-m PEM` to the command for ssh key generation. It should look like :
    ```sh
    ssh-keygen \                                   
      -b 4096 \       
      -C "Kitchen-Terraform AWS provider tutorial" \
      -f test/assets/key_pair \
      -N "" \
      -t rsa \
      -m PEM
    ```
    
- See short run log at the end of the readme - [here](#run-logs)
- Full run log file for 'us-west-2' can be found here [test.log](test.log)

    
# Todo


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
- [x] find out why KitchenCI failing on some test with "remote: no implicit conversion of Array into String"
- [x] fix the error on test 5
- [x] sanitize logs and docs

# Run logs

## Short sanitized version for successful tests :
Result of execution verify for `us-west-2 ` :

```shell
-----> Starting Kitchen (v1.25.0)
-----> Verifying <extensive-suite-ubuntu>...
$$$$$$ Running command `terraform workspace select kitchen-terraform-extensive-suite-ubuntu` in directory /Users.../kitchen-extensive-terraform/test/fixtures/wrapper
$$$$$$ Running command `terraform output -json` in directory /Users.../kitchen-extensive-terraform/test/fixtures/wrapper
local: Verifying

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
remote: Verifying host ec2-34-216-20-49.us-west-2.compute.amazonaws.com

Profile: Extensive Kitchen-Terraform (extensive_suite)
Version: 0.1.0
Target:  ssh://ubuntu@ec2-34-216-20-49.us-west-2.compute.amazonaws.com:22

  ✔  operating_system: ubuntu
     ✔  ubuntu is expected to eq "ubuntu"
  ✔  reachable_other_host: Host 52.37.61.236

     ✔  Host 52.37.61.236
      is expected to be reachable


Profile Summary: 2 successful controls, 0 control failures, 0 controls skipped
Test Summary: 2 successful, 0 failures, 0 skipped
remote: Verifying host ec2-54-184-125-225.us-west-2.compute.amazonaws.com

Profile: Extensive Kitchen-Terraform (extensive_suite)
Version: 0.1.0
Target:  ssh://ubuntu@ec2-54-184-125-225.us-west-2.compute.amazonaws.com:22

  ✔  operating_system: ubuntu
     ✔  ubuntu is expected to eq "ubuntu"
  ✔  reachable_other_host: Host 52.37.61.236

     ✔  Host 52.37.61.236
      is expected to be reachable


Profile Summary: 2 successful controls, 0 control failures, 0 controls skipped
Test Summary: 2 successful, 0 failures, 0 skipped
       Finished verifying <extensive-suite-ubuntu> (0m15.73s).
-----> Kitchen is finished. (0m17.65s)
```
