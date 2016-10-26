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
10. can add Slack notifications to Drone that work via webhooks
    - notifications can be configured to only run on certain branches (master, etc) events (push, pull-request, etc.) and build results (pass, fail)
11. adding a publish command that pushes the successful build out to AWS ECR
12. adding a deploy command for development builds
    - pushes to the same briks stack/service in Rancher
    - can do auto upgrades when pushed
    
- Drone has a concept called "service containers" that are useful for creating additional Docker images that support the build
    - e.g. a Postgres DB that's independent of any other dev DBs
    - e.g. a Chrome driver image for doing in-browser testing without actually using a user's browser
    
- Drone doesn't cache depedencies between builds, so it has to pull everything every time
    - this can be configured, but it does come with some gotchas
    - John recommends only caching on master

### Drone vs. Jenkins
- Drone uses the GitHub webhooks to listen for events
- Drone is build on Docker and makes tons of use of containers
- configs are stored in source control and are portable and isolated (which version of Java, Node, etc)
    - which means as applications and drone builds are updated, it's possible to go back in time to earlier builds and have the correct Drone config available
    - configs are YAML files
- Drone doesn't store every build the way Jenkins does, so it requires further configuration to track stats and keep track of what has happened with each build
- Point is simply to run builds as quickly as possible to verify code success

### Build Dockerfile
10:40am
led by OPI David

- can create an image by starting with an empty Docker image and building within
- recommends creating a Dockerfile to programmatically build the image so it's repeatable
    1. starts FROM the same openJDK image used for the build
        - aside: Docker containers are layered so if multiple applications use the same base image, they don't have to download the entire thing every time; just the changes
        - every step in the Dockerfile is a layer
    2. ADDs the distributed output of the build (.tar in this case)
    3. runs a CMD to extract the .tar
    4. EXPOSE a port on which the application runs
        - don't need to explicitly do this in the Dockerfile, but it's helpful for people to see where the app will run
    5. after the build is done, run the finished Docker container to start up the server
    6. then map the port on the container to the host computer's port so test/view in the host browser
    
### On Rancher
11:25am or so

1. created a stack on Rancher called briks
2. created an empty service called briks
3. the Drone build will automatically push successful builds to the service in Rancher when a pull request is merged
4. added a rule on the load balancer to route requests to `bricks.*` on port 80 to the briks container on port 9000

(really good side conversation about logging and where it goes. OPI has recommendations for getting logs into a service that can search and filter and do all sorts of manipulation on logs that can't be done in normal circumstances)
    - uses stdout and stderror as the way to capture errors instead of writing to files
    
#### Additions to the Drone build
(#s 11 and 12 above)
    
