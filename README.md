## Requirements

Before you start with Salt Stack, Kitchen and InSpec you have to install:

 - Vagrant 1.9.1+
 - Ruby 1.9.3+
 - Ruby Gems (`gem` command should be available)
 - install following Ruby Gems:
   - `test-kitchen`
   - `kitchen-salt`
   - `kitchen-inspec`
   - `inspec`
   
## Test Kitchen

**Test Kitchen** - tool for local development of CM code (Chef Cookbook, Ansible Roles, Salt States etc.)
 and tool for integration/acceptance testing of this code.

Test kitchen supports many type of infrastructure provisioners (Vagrant, Docker, OpenStack etc.)
 and testing tools (InSpec, ServerSpec, Bats, Pester).
 
Test Kitchen consists of ([Kitchen Ecosystem](https://github.com/test-kitchen/test-kitchen/blob/master/ECOSYSTEM.md])):
 
 - `driver` - driver for infrastructure provisioning, like OpenStack, Vagrant, Docker etc.
 - `transport` - how Kitchen will copy files/connect to server, by default `ssh` (for Windows WinRM).
 - `provisioner` - which tool will provision application on server, like `chef_solo`, `salt_solo`, `ansible` etc.
 - `verifier` - which tool will be used for integration/acceptance testing. By default used [Busser](https://github.com/test-kitchen/busser), but for *nix systems recommended InSpec (see below). 
 - `platforms` - describe on which platforms/OS`es your code will be tested.
 - `suites` - test cases in which you want to test your code (see examples below).
 
Kitchen config example (`.kitchen.yml`):

    ---
    driver:
      name: vagrant
        
    verifier:
      name: inspec
    
    provisioner:
      name: salt_solo
      salt_version: 2015.8.8
      is_file_root: true
      state_top:
        base:
          '*':
            - mc
    
    platforms:
      - name: centos/7
      - name: ubuntu-14

    suites:
      - name: default


Here you can see that Vagrant used as infrastructure backend, provisioner - Salt Stack, 
 for acceptance/integration tests used InSpec. Code will be tested on CentOS 7 and Ubuntu 14 with default configuration.

For each your platform Kitchen will create all suites, for example:

    ~> kitchen list
    
    Instance          Driver   Provisioner  Verifier  Transport  Last Action
    default-centos-7  Vagrant  SaltSolo     Inspec    Ssh        Converged
    default-ubuntu-14 Vagrant  SaltSolo     Inspec    Ssh        Converged

More about Test Kitchen can be found here:

 - https://kitchen.ci/
 - https://github.com/test-kitchen/test-kitchen
 - https://github.com/simonmcc/kitchen-salt/blob/master/INTRODUCTION.md


### InSpec for integration/acceptance testing

InSpec - tool for integration/acceptance testing of infrastructure, its syntax based on RSpec.

If you want to test your code with InSpec and Kitchen, specify `inspec` as verifier and 
 create following folder structure:
 
    test/
      integration/
        default/
          test_spec.rb
          
Tests in folder with name `default` will be used for `default` Kitchen suite. Files with pattern `*_spec.rb`
will be treated as InSpec tests.

Example of InSpec test for Apache Web Server: 

    describe package('httpd') do
      it { should be_installed }
    end
    
    describe service('httpd') do
      it { should be_installed }
      it { should be_enabled }
      it { should be_running }
    end

Example of InSpec test for IIS:

    describe windows_feature('IIS-Webserver') do
      it{ should be_installed }
    end

    describe service('W3SVC') do
      it { should be_installed }
      it { should be_running }
    end

Also InSpec can be used as independent too for testing of your infrastructure,
more about this you can find here - https://github.com/chef/inspec#command-line-usage
 

More about InSpec, resources and so on can be found here:

 - http://inspec.io/docs/reference/resources/
 - https://github.com/chef/inspec
