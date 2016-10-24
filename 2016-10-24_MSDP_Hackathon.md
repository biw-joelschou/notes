# Modern Software Development Practices Hackathon
2016-10-24

## Terraform pt.1 : intro and practice
9:30am or so
Led by OPI John

- Got an overview of Terraform and how it works. Saw some examples of configurations from OPI Andy
- Important note: _always_ store your AWS credentials locally and _never ever ever_ commit them to any sort of repo, local or otherwise
- Diving straight into practicing with Terraform
- Broke into two teams
    1. Walter
        - Ashby
        - Chris S.
        - Ravi K.
        - OPI John
        - Dave E.
    2. Adam
        - Ron
        - Prabu
        - Court
        - Haak
        - OPI Andy
- Adam set up a bucket on AWS with permissions and such for us to access

### Things to do
_(Note: I'm hanging out with the Adam team for these notes)_

1. VPC
    - set up a variety of components
    - public vs. private
    - security groups
    - etc.
2. subnets
    - if making more than one (public and private), the cidr_block needs to be different (won't fail in TF, will fail in AWS)
        - as we build more and more AWS environments, we should have some sort of plan about how we name/number our blocks
        - use of public and private could vary per project and for what code ends up where
    - set the private subnet to not have a public IP
3. AMI instances
    - used a data object to get the most recent Ubuntu to use for a new instances
    - then created an instance (tried t2.micro first, it failed due to some virtualization config, went to t1.micro)
    - assigned a resource to the appropriate subnet ID
4. modules
    - good place to do the networking layer
    - can reference things that are local, on GH, other repos, generic HTTP locations, etc.
    - adam set up a module to handle the creation of the subnets: three different ones on three different availability zones
    - any time you create a module, even if it is local, you need to run `terraform get` to import it
    
#### Best practices
- Break your components into individual Terraform files: VPC, security groups, etc.
- TF just compiles everything together, it doesn't care about individual files
- Generated tfstate file from TF goes to AWS, but the dev doesn't really care about the contents
- use modules to share common configs (true?)
- make as many things variables as possible, but give defaults
    - variables can be imported from separate files

## Terraform pt.2 : implementation for real
10:45am
Led by OPI John

### Networking setup
- giving a small overview of how AMZ's geography works
    - regions > availability zones > data centers, etc.
    - infra between regions is considered a WAN with all associated latency considerations
    - infra within a region is considered a LAN with low-latency
- create a VPC with public and private subnets
    - three of each across three different buildings (one of each per zone)
- can create network ACLs between subnets (recommends against that because subnets are stateless and then you have to open ports and defeat the purpose of the private subnet)
    - create security groups instead

### Rancher
- (I'm totally not following this topic so bad notes)
- (Production setup will have redundancy for Rancher and DBs, but for the purposes of this hack we're not going to go through the work required to balance that ... something about UDP load-balancing limits in AMZ)
- DBs will live inside the private subnets
- Racher server will live in the public ones

- "Rancher is a container orchestration platform"
- Two pieces to Rancher
    1. Server
        - API
        - UI
        - Scheduling for containers
        - State
    2. Agent
        - Boots and calls server ("I'm joining this environment")
        - multiple machines in the cluster (called an "environment") will all call Rancher server to get themselves set up
            - each machine will create a networking agent for talking to each other on specific UDP ports (500, 4500) using a VPN tunnel
        - each container on a machine will talk to other containers on other machines within that cluster using this ^ local networking agent
- (plenty of talk and questions about why to use Rancher and what we get with it vs. bare Kubernetes w/o Rancher)
    - Rancher's goal is to build a clean platform that's more "click and go" and do the management on top of the tools
    - Rancher doesn't mask any access to the containers themselves, so we don't lose any access for troubleshooting and whatnot
    - Amazon Load Balancer sits in front of all the containers in an environment to distribute requests
        - can be a layer of "proxy" machines bewteen the load balancer and the environment containers (didn't catch why)
    - Racher's metadata API is something John is touting as extremely powerful

### Diving into Rancher
11:40am or so

#### Our setup
(Andy put this together)

- hosts, stacks, containers... I'm lost
- naming scheme: stackname_containername_# (??? double check this)
- sidekick containers move together (??? right?)
    - all separately-running Docker containers with their own IPs
- links allow you to reference a different stack's containers without being in the same stack
- can view logs, execute shell, and a few other things through the Rancher UI

#### More Rancher fun
- Rancher has a catalog of containers which can be installed with minimal configuration
    - Can create a private catalog of containers we create
- (lots of things I didn't feel like noting, mainly because lunch)

## Building Briks
(that's our new name)
12:45pm

- Ravi and Chris getting started
    - Postgres
    - question about deploy process - local? remote? Docker?
- Ashby is going to take a good amount of GQ admin code to lay on top of the API
