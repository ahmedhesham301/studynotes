to write a shell command in Jenkins script  
sh 'command'  

to run something at the end of the pipeline  
use  
```
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

to add environment variables add the next block to the pipeline  
```
environment{
    variableName = value
}
```
to use the variable use a `$` before it's name(ahh bash)  

to use a docker image put this block in the stage where it is needed  
```
agent{
    docker{
        image 'image:tag'
        
    }
}
```

to reuse the docker image add to` docker{}` `reuseNode true`

---
# Docker with Jenkins
## building docker image in Jenkins

requirements [Docker Pipeline](https://plugins.jenkins.io/docker-workflow) plugin
```
docker.build("imagename:tag")
```

you can also but an image in variable using
```
def customImage = docker.build("imagename:tag")
```


## Using docker hub credentials in Jenkins

requirements:docker personal access tokens added in Jenkins credentials as username/password.
add the following code to `environment{}`
```
DOCKERHUB_CREDENTIALS = 'dockerhub-credentials-id'
```

to login to docker hub 
```
docker.withRegistry('', DOCKERHUB_CREDENTIALS) {
	customimage.push("1.0") //pushes an image in a variable called customimage
}
```