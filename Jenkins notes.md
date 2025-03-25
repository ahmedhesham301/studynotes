to write a shell command in Jenkins script  
sh 'command'  

to run something at the end of the pipeline/stage  
use  
```groovy
post{
    always{}    //always runs what in it wether build fails or succeeds
    success{}   //runs only when build success
	failure{}   //always runs what int it when the buidl fails
}
```

to clear the work directory
`cleanWs()`

to save artifacts  
`archiveArtifacts artifacts: 'path'`


to use a docker image put this block in the stage where it is needed  
```groovy
agent{
    docker{
        image 'image:tag'
        
    }
}
```

to reuse the docker image add `reuseNode true`
to run as root add `args '--user root'`
## how to run stages in parallel
```groovy
stage('name') {
	parallel {
		stage('first stage'){}
		stage('second stage'){}
	}
}
```

---
# Docker with Jenkins

requirements [Docker Pipeline](https://plugins.jenkins.io/docker-workflow) plugin
## building docker image in Jenkins


```groovy
docker.build("imagename:tag")
```

you can also but an image in variable using
```groovy
def customImage = docker.build("imagename:tag")
```


## Using docker hub credentials in Jenkins

requirements:docker personal access tokens added in Jenkins credentials as username/password.
add the following code to `environment{}`
```groovy
DOCKERHUB_CREDENTIALS = 'dockerhub-credentials-id'
```

to login to docker hub 
```groovy
docker.withRegistry('', DOCKERHUB_CREDENTIALS) {
	customimage.push("1.0") //pushes an image in a variable called customimage
}
```

---
# Environment variables
to add environment **to all stages** variables add the next block to the pipeline to make it available to all stages or to the stage to make it available to only that stage 
```groovy
environment{
    variableName = value
}
```
to use the variable use a `$` before it's name(ahh bash)  

## To add environment variables during run
include a `script {}` block inside the `steps {}` section and define the variable as follows:
```groovy
steps {
    script {
        env.VAR_NAME = "<value>"
    }
}
```
### To pass output from command to an env variable
```groovy
env.VAR_NAME = sh(script: 'date', returnStdout: true)
```
### Usage Example:
> When referencing the variable in the `steps{}` block, always use double quotes (`"`) instead of single quotes (`'`)
```groovy
steps {
	echo "$env.VAR_NAME"
	# or
	echo "${env.VAR_NAME}"
}
```
```groovy
environment{
	VAR = "$env.VAR_NAME"
}
```