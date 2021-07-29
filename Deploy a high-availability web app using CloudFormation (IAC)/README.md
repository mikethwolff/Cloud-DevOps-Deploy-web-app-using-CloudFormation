Scenario

Your company is creating an Instagram clone called Udagram. Developers pushed the latest version of their code in a zip file located in a public S3 Bucket.

You have been tasked with deploying the application, along with the necessary supporting software into its matching infrastructure.

This needs to be done in an automated fashion so that the infrastructure can be discarded as soon as the testing team finishes their tests and gathers their results.

Solution:

[[https://github.com/mikethwolff/Cloud-DevOps-Engineer-Projects-Udacity/blob/main/Deploy%20a%20high-availability%20web%20app%20using%20CloudFormation%20(IAC)/UdacityDevOpsProject.png|alt=Lucychart]]


Run commands:
```
./create.sh [stackName1] network.yml network-parameters.json
./create.sh [stackName2] servers.yml servers-parameters.json
```
Update commands:
```
./update.sh [stackName1] network.yml network-parameters.json
./update.sh [stackName2] servers.yml servers-parameters.json
```
