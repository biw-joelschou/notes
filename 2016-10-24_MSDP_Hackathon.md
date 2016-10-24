# Modern Software Development Practices Hackathon
2016-10-24

## Terraform
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
