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
5. From Terrraform 0.12.07 the format of outputs for arrays had changed slighlty, actually simplifying code, so, I've edited outputs.tf in both fixtures and main repo location correspondingly to : 
```terraform
output "remote_group_public_dns" {
  description = "This output is used to obtain targets for InSpec"

  value = "${module.extensive_kitchen_terraform.remote_group_public_dns}" # no [] - brackets here
}
```terraform
and main `outputs.tf` :
output "remote_group_public_dns" {
  description = "The list of public DNS names of the remote_group instances"
  value       = "${aws_instance.remote_group.*.public_dns}" # no [] - brackets here
}
```
6. **macOS Mojave specific!!!** The way how default "ssh-keygen" in macOS Mojave tuned you going to expireince problem with ssh key, so appen `-m PEM` to the command for ssh key generation. It should look like :
```sh
ssh-keygen \                                   
  -b 4096 \       
  -C "Kitchen-Terraform AWS provider tutorial" \
  -f test/assets/key_pair \
  -N "" \
  -t rsa \
  -m PEM
```

# Todo
- [ ] fix the error on test 5

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
