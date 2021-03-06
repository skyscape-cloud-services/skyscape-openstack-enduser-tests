[![Build Status](https://travis-ci.org/ukcloud/ukcloud-openstack-enduser-tests.svg?branch=master)](https://travis-ci.org/ukcloud/ukcloud-openstack-enduser-tests)

# UKcloud OpenStack End-user tests


Tempest is an OpenStack project dedicated to testing. It provides coverage in the following forms:
  * API - tests that the API functions as expected, both during success and failure
  * Scenario - Allows complex user based scenarios to be setup to allow full workflows to be tested
  * Stress - Allows for complex stress testing workloads to be executed

UKcloud exposes the end-user API and Scenbario tests that run to validate consitency of the platform between changes to help customers understand and contribute to the test coverage. The aim is to improve transparency of the platforms fucntionailty whilst also enabling customers to contribute tests that are pertinent to their infrastructure.

Users can contribute by either:

  * Making feature requests
  * Submitting pull requests for additional tests

## Overview

Tempest tests can be run from an end user machine with the tempest tooling installed and configured.

# Setup

Download the latest stable release of tempest.

```
git clone http://git.openstack.org/openstack/tempest
```

Create a new python virtual environment with all the required dependencies by running

```
cd tempest
./run_tempest.sh
```

source the new python virtual environment

```
source .venv/bin/activate
```

install tempest

```
pip install ../tempest/
```

# Configuration

## Config data

This repository can be used to build a basic a configuration. change up a directory and checkout this repostitory to complete the configuration.

```
cd ..
git clone git@github.com:ukcloud/ukcloud-openstack-enduser-tests.git
cd ukcloud-openstack-enduser-tests
```

Tempest requires two files to test. This guide only details a basic setup testing non-admin functionality. To review the configuration in more detail read the sample tempest.conf available in http://docs.openstack.org/developer/tempest/sampleconf.html#tempest-sampleconf. A basic configuration files with some defaults and placeholders can be found in etc/tempest.conf.

Note: The configuration is configured to use pre-defined accounts rather than dynamic account generation. More information can be found here: http://docs.openstack.org/developer/tempest/configuration.html (Credential Provider Mechanisms)

Edit the tempest config etc/tempest.conf configuring the following values

### URL endpoints
This is the authentication endpoints for both the v2.0 and v3 API's for the OpenStack platform e.g. 

```
uri = https://cor00005.cni.ukcloud.com:5000/v2.0
```

```
uri_v3 = https://cor00005.cni.ukcloud.com:5000/v3
```

### Test accounts

This needs to point to the relative path of the accounts file listing the credentials to perform operations in the OpenStack cloud. This can usually be left as the value below.

```
test_accounts_file = etc/accounts.yaml
```

### Credential methods

We are using provisioned credentials so dynamic credentials must be disabled

```
use_dynamic_credentials = False
```

You also need to define the default endpoint to use. Currently UKcloud use v2. However we enable tests for both.

```
auth_version = v2
api_v2 = true
api_v3 = true
```

### Compute Configuration 

You must supply a number of images that can be used to perform the scenario tests. The following values must be provided

```
image_ref = <image ref e.g. 602d57d9-ff8a-4ccc-9249-11dcea815cc2>
image_ref_alt = <image ref e.g. 602d57d9-ff8a-4ccc-9249-11dcea815cc2>
```

This information can be obtained by running the OpenStack tools

```

➜  code openstack image list

+--------------------------------------+-----------------------------------------------+--------+
| ID                                   | Name                                          | Status |
+--------------------------------------+-----------------------------------------------+--------+
| 956e49d1-c8a5-4816-820a-2732af1ce957 | glance-test                                   | active |
| d04e07c4-095d-4598-81ec-43c3309a2c99 | ubuntu-14.04-server-amd64-original-downloaded | active |
+--------------------------------------+-----------------------------------------------+--------+
```

```
flavor_ref = <image ref e.g. 2>
flavor_ref_alt = <image ref e.g. 3>
```

```
➜  code openstack flavor list

+----+-----------+-------+------+-----------+-------+-----------+
| ID | Name      |   RAM | Disk | Ephemeral | VCPUs | Is Public |
+----+-----------+-------+------+-----------+-------+-----------+
| 1  | m1.tiny   |   512 |    1 |         0 |     1 | True      |
| 2  | m1.small  |  2048 |   20 |         0 |     1 | True      |
+----+-----------+-------+------+-----------+-------+-----------+
```

The image to be used during testing must be matched with regex.

```
image_regex = ^cirros-0.3.1-x86_64-uec$
```

The user to be used to access SSH must also be sepcified.

```
ssh_user_regex = [["^.*[Cc]irros.*$", "cirros"]]
```

The image details must be added so the scenario tests can correctly upload, deploy and validate the images.

```
img_dir = <imagesDir>
img_file = cirros-0.3.1-i386-disk.vmdk
img_disk_format = vmdk
img_container_format = bare
```

### Network attributes

During the tests floating IP addresses are assigned so that machines can be connected to validate OS configurations. In order to achieve this the floating IP network pool must be defined.

```
floating_network_name = <network name>
```

This can be obtained by running the following OpenStack command

```
➜  code openstack ip floating pool list
+----------+
| Name     |
+----------+
| Internet |
+----------+
```

An internal network is also required to attach the VMs the deployed during the tests to.

```
fixed_network_name = <network name>
```

```
➜  code openstack network list

+--------------------------------------+----------+--------------------------------------+
| ID                                   | Name     | Subnets                              |
+--------------------------------------+----------+--------------------------------------+
| 03b63fc5-f231-44d2-9b2f-5aa5923db494 | Internet | 63b6a2db-a5e8-4b8c-a279-18abde81bdf9 |
+--------------------------------------+----------+--------------------------------------+
```

The connection method must be specified as floating.

```
connect_method = floating
```

### Volume attributes

Tests that check volumes can be attached and mounted are run. Based on the image used the volumes list as specific devices. This needs to be updated to match the volume device label that is listed in the image you chose. The default is vdb.

```
volume_device_name = vdb
```

### Volume Disk attributes

This is only required when using VMware as a backend. 

```
disk_formats = vmdk
```

### Validation attributes

The following settings must be configured for the scenario tests.

```
auth_method = keypair
ip_version_for_ssh = 4
ping_timeout = 120
connect_timeout = 60
ssh_timeout = 30
```

## Accounts file

The accounts file must be populated with a minimum of one account. If additional accounts are provided they must be associated with seperate projects. There is an example accounts file in etc/accounts.yaml.
 
```
- username: <username>
  tenant_name: <Project Name>
  password: <password>
```

# Running

Because most users only have a access to a single project for testing the command below only requires a single user in the accounts file by setting the number of workers to one with concurrency flag.

## API Tests
```
ostestr --regex '(?!.*\[.*\bslow\b.*\])(^tempest\.(api))' --pretty --color --concurrency 1
```

## Scenario tests
```
ostestr --regex '(?!.*\[.*\bslow\b.*\])(^tempest\.(scenario))' --pretty --color --concurrency 1
```

### Individual tests

Individual tests can be run against scenarios. This can useful to check a very specific workflow.

First instantiate the testr environment

```
testr init
```

```
testr run tempest.scenario.test_minimum_basic.TestMinimumBasicScenario --concurrency=1
```

