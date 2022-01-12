### Docker build using -

## TL:DR;

When ever the dockerfile is piped into the build command like `docker build - < Dockerfile`, there wont be any context as in from which folder you are running the build. 
No files in that folder will be considered for building. That means, `COPY <this-file> <there>` wont work for any files in the folder you are working from. 

I simply renamed my dockerfile as `Dockerfile` and did the `docker build -t latest .` which builds from the current folder using the `Dockerfile` by default.

###  That long story

I was trying to build a docker image of my code base and push it into a registry. 
My `actions.yaml` looked like this:
```
...
- name: Build
  id: build-image
  run: |
    docker build -t latest - < Dockerfile.prod
...
```

For some reasons, I had named my dockerfile `Dockerfile.prod`. 

The problem was that I was getting this error while running GH actions. 
```
failed to compute cache key: "package.json" not found: not found
```

That means there is no such file named package.json. Well, It actually is there, in the folder 

I started running these actions locally (Thanks to act). And still no luck. (well, sometimes it works locally)

I tried doing an `ls` to see what all files where there in the build env. Unfortunately it was listing all the files in my local folder.
 
Finally, as usual, Stackoverflow to the rescue!  Well, kind of.

There was a guy shouting out the importance of `.` at the end of the command. Well, I didn't have any. 

It means the build command I was doing, didn't have any context to copy from. Piping a docker file simply builds it out without considering from which folder we are running it.
