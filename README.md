# Say hello
Let's move into years of sharing, my work is for developers who love「CI/CD」,「infras-as-code」and「immutable-infrastructure」. This work is a kickstarter sample, not a generation tool so that the source code must be clear to understand.

# Index
1. [TL;DR version](#tldr-version)
1. [AWS console preparation](#aws-console-preparation)
1. [Credentials preparation](#credentials-preparation)
1. [Running](#running)
1. [Furthermore](#furthermore)
1. [Best practices](#best-practices)
1. [License](#license)

# Next features
1. Ansible roles for stack services such as: Jenkins, PHP, Nginx, DevelopmentEnv, TestEnv, ProdEnv, etc (Just for now, please go with「phansible.com」to prepare your stacks as mentioned at [#Furthermore](#furthermore)
1. Support various image types
    * Docker Image
    * Vagrant Box
1. CloudFormation stack includes various serivces: EC2 (already supported), ASG, ELB, SQS, RDS, ElasticSearch, etc. So that we can instance a big stack at a blink.

# TL;DR version
1. [AWS console preparation](#aws-console-preparation)
1. [Credentials preparation](#credentials-preparation)
1. Change working dir to packer-ansible-cloudformation
    * `cd packer-ansible-cloudformation`
1. Building AWS AMI
    * `packer build packer_aws.json`
1. Export golden image value
    * `export AWS_EC2_GOLDEN_IMAGE=PLEASE_place_above_AMI_ID_here`
1. Building new stacks
    * `aws cloudformation create-stack --stack-name CloudFormationDemo --template-body file:///$PWD/cloudformation.template --parameters ParameterKey=ImageIdParameter,ParameterValue=$AWS_EC2_GOLDEN_IMAGE ParameterKey=InstanceTypeParameter,ParameterValue=$AWS_EC2_INSTANCE_TYPE ParameterKey=KeyNameParameter,ParameterValue=$AWS_EC2_KEY_PAIR ParameterKey=SecurityGroupParameter,ParameterValue=$AWS_EC2_SECURITY_GROUP ParameterKey=NameTag,ParameterValue=$AWS_EC2_NAME_TAG`
        * Output successfully
            ```
            {
                "StackId": "arn:aws:cloudformation:......"
            }
            ```
# AWS console preparation
1. Key pairs
    * Go to AWS Console → Services → EC2 → Key Pairs
        * Create new key pairs
1. Security group
    * Go to AWS Console → Services → EC2 → Security Groups
        * Create new security group

# Credentials preparation 
1. Create new file「.bash_aws」for exporting environment variables
    * `vi ~/.bash_aws`
        ```
        export AWS_ACCESS_KEY_ID=PLACE_YOUR_AWS_ACCESS_KEY_ID_HERE
        export AWS_SECRET_ACCESS_KEY=PLACE_YOUR_AWS_SECRET_ACCESS_KEY_HERE
        export AWS_DEFAULT_REGION=PLACE_YOUR_AWS_DEFAULT_REGION_HERE
        export AWS_PROFILE=PLACE_YOUR_AWS_PROFILE_HERE 
        export AWS_CONFIG_FILE=PLACE_PATH_FILE_OF_YOUR_AWS_CONFIG_FILE
        export AWS_SOURCE_AMI=PLACE_YOUR_DEFAULT_AWS_SOURCE_AMI
        export AWS_EC2_INSTANCE_TYPE=PLACE_YOUR_DEFAULT_AWS_EC2_INSTANCE_TYPE
        export AWS_EC2_SSH_USERNAME=PLACE_YOUR_DEFAULT_AWS_EC2_SSH_USERNAME
        export AWS_EC2_SSH_TIMEOUT=PLACE_YOUR_DEFAULT_AWS_EC2_SSH_TIMEOUT
        export AWS_EC2_KEY_PAIR=PLACE_YOUR_DEFAULT_AWS_EC2_KEY_PAIR_NAME
        export AWS_EC2_SECURITY_GROUP=PLACE_YOUR_DEFAULT_AWS_EC2_SECURITY_GROUP
        export AWS_EC2_NAME_TAG=PLACE_YOUR_DEFAULT_AWS_EC2_NAME_TAG
        export AWS_AMI_NAME_PREFIX=packer-demo
        ```
1. Calling「.bash_aws」from「.bashrc」
    * `vi ~/.bashrc`
        ```
        # AWS Credentials
        [[ -f ~/.bash_aws ]] && . ~/.bash_aws
        ```
1. Running for exporting environment variables
    * `. ~/.bashrc` or relogin, restart
1. Reference
    * HowTo: Install AWS CLI - Security Credentials
        * http://www.dowdandassociates.com/blog/content/howto-install-aws-cli-security-credentials/
        * http://docs.aws.amazon.com/cli/latest/userguide/cli-environment.html
        * http://docs.aws.amazon.com/cli/latest/topic/config-vars.html
# Running 
1. cd to packer-ansible-cloudformation
    * `cd packer-ansible-cloudformation`
1. Run builder
    * `packer build packer_aws.json`
1. Checking
    * Console output after built successfully
        ```
        ==> Builds finished. The artifacts of successful builds are:
        --> amazon-ebs: AMIs were created:
        your-region: your-ami-id
        ```
    * Go to `AWS Console → Services → EC2 → AMIs`, to see your new AMI
1. Launching
    * Manually 
        * Right click your new AMI and click「Launch」
    * With CloudFormation
        * Export golden image value
            * `export AWS_EC2_GOLDEN_IMAGE=PLEASE_place_above_AMI_ID_here`
        * Building new stacks
            * `aws cloudformation create-stack --stack-name CloudFormationDemo --template-body file:///$PWD/cloudformation.template --parameters ParameterKey=ImageIdParameter,ParameterValue=$AWS_EC2_GOLDEN_IMAGE ParameterKey=InstanceTypeParameter,ParameterValue=$AWS_EC2_INSTANCE_TYPE ParameterKey=KeyNameParameter,ParameterValue=$AWS_EC2_KEY_PAIR ParameterKey=SecurityGroupParameter,ParameterValue=$AWS_EC2_SECURITY_GROUP ParameterKey=NameTag,ParameterValue=$AWS_EC2_NAME_TAG`
                * Output successfully
                    ```
                    {
                        "StackId": "arn:aws:cloudformation:......"
                    }
                    ```

# Furthermore
1. For Ansibles role provisioning, the below website supports us to get started.
    * http://phansible.com/

# Best practices 
1. Using `yum search` or `yum info` to check packages existing or not when preparing Ansible provision
    * Example with `yum search`
        ```
        $ yum search php71-cli
        Loaded plugins: priorities, update-motd, upgrade-helper
        github_git-lfs                                                                                                                           36/36
        github_git-lfs-source                                                                                                                    33/33
        2 packages excluded due to repository priority protections
        ================= N/S matched: php71-cli =================
        php71-cli.x86_64 : Command-line interface for PHP
        ```
    * Example with `yum info`
        ```
        $ yum info php71-cli
        Loaded plugins: priorities, update-motd, upgrade-helper
        2 packages excluded due to repository priority protections
        Available Packages
        Name        : php71-cli
        Arch        : x86_64
        Version     : 7.1.7
        Release     : 1.26.amzn1
        Size        : 4.3 M
        Repo        : amzn-main/latest
        Summary     : Command-line interface for PHP
        URL         : http://www.php.net/
        License     : PHP and Zend and BSD and MIT and ASL 1.0
        Description : The php-cli package contains the command-line interface
                    : executing PHP scripts, /usr/bin/php, and the CGI interface.
        ```
    * Example for checking multi packages
        * With `yum search`
            ```
            yum search php71 php71-cli php71-common php71-devel php71-gd php71-mbstring php71-pdo php71-pecl-apcu php71-xml
            ```
        * With `yum info`
            ```
            yum info php71 php71-cli php71-common php71-devel php71-gd php71-mbstring php71-pdo php71-pecl-apcu php71-xml
            ```
1. Make sure packages installed by `yum list installed`
    * `yum list installed php71 php71-cli php71-common php71-devel php71-gd php71-mbstring php71-pdo php71-pecl-apcu php71-xml`
        * Reference
            * https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/sec-Listing_Packages.html

# License

[The MIT License](https://opensource.org/licenses/MIT)

Copyright 2017 Sơn Cao

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
