# Milestone 2

## Cleanup for MS2

* I need to start the clair server up as part of my Makefile. Make sure to use 
latest date for the DB.
* Why are policies run in a privileged container? Not even sure what this 
option means. Time to hit the docs.
    - Root users in a privileged container run as root on the host system.
    - Container isolation is essentially not in effect.
    - https://blog.trendmicro.com/trendlabs-security-intelligence/why-running-a-privileged-container-in-docker-is-a-bad-idea/
    - I'm still not sure why. My policies seem to work even though I'm not.
* Add more targets to Makefile. My build pipeline is too simplistic.

## Thoughts on security scan

* My Hugo-Builder image passed clair's scan, even when changing the version to 3.8. 
I tried clearing the build cache and using the latest clair db. I even moved the 
version back to 3, and still didn't turn up any CVE positives.
* Scanning the Fusionauth image turned up 93 vulnerabilities, 8 of them high.
* It looks like many system components are out of date, so likely the base image 
for this version of Fusionauth is too old.
* There are many unnecessary packages listed for a production build. This increases 
the attack surface. It looks like the base OS may be debian, when a smaller package 
like Alpine should be used.
* Realizing not getting CVE positives on old versions of Alpine may be part of the lesson. 
The slimmer OS has far fewer vulnerabilities even for an old base image. This shows the 
benefits of a greatly reduced attack surface.
* I had trouble determining the base image used for the 1.15.8 image of fusionauth. I think I 
need to learn docker inspect better.