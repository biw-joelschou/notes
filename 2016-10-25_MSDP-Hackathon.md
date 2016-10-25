# Modern Software Development Practices Hackathon
2016-10-25

## Rancher Registries
(missed the importance of registries. Someone please fill in)

- set up by environment
    - Rancher sends everything to all the containers
- ECR is a registry provided by Amazon
    - have to use AWS command line tools + Amazon's APIs to interact with it if you need to interact with the registry itself
    - same thing from within Rancher - OPI provided a little tool that handles the credential updates
- Can host our own registry, but would require rolling our own authentication process

## Drone
- standard setup from OPI
    - DB schema
    - drone itself
    - drone load balancer
    
- have an ELB that accepts all requests to *.biw-labs.com on port 80
    - (anything more specific than * is DNS routed independently)
- forwards to the host on port 80 as well
- Rancher load balancer runs on the host with some layer 7 rules to forward (?) to drone

(Interesting tangent/discussion on how to implement this entire concept in our world with 600+ discreet applications running on servers and needing to track who is using resources in AWS for billing purposes)
- John recommends a single Rancher server that manages environments for each and every application running in their own AWS accounts

