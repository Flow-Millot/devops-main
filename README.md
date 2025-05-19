# devops-main

## TP part 01

### 1-1 For which reason is it better to run the container with a flag -e to give the environment variables rather than put them directly in the Dockerfile?
'''
Security is the main reason because when we are defining environment variables like POSTGRES_PASSWORD inside the Dockerfile, they become part of the image permanently.
'''
