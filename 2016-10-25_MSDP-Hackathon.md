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

> (Interesting tangent/discussion on how to implement this entire concept in our world with 600+ discreet applications running on servers and needing to track who is using resources in AWS for billing purposes)
- John recommends a single Rancher server that manages environments for each and every application running in their own AWS accounts
- outlined an alternative that involves more shared servers for _n_ applications and billing based on total cost / _n_ but I didn't catch all the details

_(Super random thing I've learned after the past two days: developers just slap stickers on laptops with no aesthetic consideration ;-) )_

- Currently running Drone 0.4. There's a 0.5 in beta but OPI wasn't ready to try that with us yet

### What do we want to do with Drone
9:10 or so
led by OPI David

- Obviously some sort of continuous integration
    - CI is the practice of developers continuously integrating all their code to a common branch
    - executes on the CI server for building and testing
    
1. cloned the repo locally
2. made the gradle wrapper executable
    - the wrapper will always make sure everyone is on the right version of the script
3. could build the application successfully
4. have to activate Drone on the specific repo so it can listen to events
    - the user who clicks the activate button has to be an admin on the repo
    - it does not poll for updates or listen to folders - events only
    - after activation, Drone adds webhooks to the github repo
        - comes with a specific token that should be hidden behind SSL connections so said token stays private
5. once Drone is activated, he has to create builds and listen to events
    - by default, only triggered on pushes and pulls
    - recommends adding the tag hook
6. create a new `.drone.yml` file for configuration
    1. specify the Docker container image to use (Java, Node, etc.) and version
        - aside: everything in Drone, including each plugin, is a Docker container
        - aside: can write our own plugins in containers to do specific tasks
    2. How it works:
        1. clones the repo as a Docker container that is shared to each container for each step of the build
        2. passes meta data and references along as each step exits as successful
7. returns callback to GH when the build is done done, and GH can tell if the build passed or not
8. GH can then allow/disallow PRs or other commits if a build doesn't pass
9. GH's permissions can then take over and require code reviews if desired

#### Drone v. Jenkins
- Drone uses the GitHub webhooks to listen for events
- Drone is build on Docker and makes tons of use of containers
- configs are stored in source control and are portable and isolated (which version of Java, Node, etc)
    - which means as applications and drone builds are updated, it's possible to go back in time to earlier builds and have the correct Drone config available
    - configs are YAML files
- Drone doesn't store every build the way Jenkins does, so it requires further configuration to track stats and keep track of what has happened with each build
- Point is simply to run builds as quickly as possible to verify code success
