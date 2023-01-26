# Kaniko Demo

Combines [Kaniko - Building Container Images In Kubernetes Without Docker](https://youtu.be/EgwVQN6GNJg) with [tekton-examples](https://github.com/CariZa/tekton-examples/blob/master/kaniko/TUTORIAL.md).

## Usage
* Setup k8s secrets
    ```bash
    kubectl create secret docker-registry regcred \
    --docker-server=https://index.docker.io/v1/ \
    --docker-username=xkcdrules \
    --docker-password='correcthorsebatterystaple' \
    --docker-email='name@acme.co'
    ```
* Replace registry/repo info
  * [local directory](kaniko-dir.yml)
    ```json
    args:
        [
          "--dockerfile=/workspace/dockerfile",
          "--context=dir://workspace",
          "--destination=jollygoodhorsepower/kaniko-demo-image:latest",
        ]
    ```
    * Edit [ConfigMap](configmap.yml) for local directory
        ```yml
        dockerfile: |
          FROM ubuntu
          ENTRYPOINT ["/bin/bash", "-c", "echo hello"]
          ```
  * [git repo](kaniko-git.yml)
    ```json
    args:
        [
          "--context=git://github.com/pythoninthegrass/kaniko_demo",
          "--destination=jollygoodhorsepower/kaniko-demo-image:latest",
        ]
    ```
* Apply manifest
    ```bash
    kubectl replace -f kaniko-git.yml --force   # kaniko-dir.yml
    ```
* Destroy
    ```bash
    kubectl delete -f kaniko-git.yml
    ```

## Troubleshooting
* Test `~/.docker/config.json`
  ```
    λ docker run --rm -it --entrypoint /bin/sh docker
    / # docker login
    Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
    Username: jollygoodhorsepower
    Password:
    WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
    Configure a credential helper to remove this warning. See
    https://docs.docker.com/engine/reference/commandline/login/#credentials-store

    Login Succeeded
    / # cat ~/.docker/config.json
    {
        "auths": {
            "https://index.docker.io/v1/": {
                "auth": "<SNIP>"
            }
        }
    }/ # exit
  ```
* Replace secret
    ```bash
    # edit
    kubectl edit secret regcred

    # delete
    kubectl delete secret regcred
    ```
* Check logs
    ```bash
    λ k logs kaniko -f
    Enumerating objects: 196, done.
    Counting objects: 100% (18/18), done.
    Compressing objects: 100% (12/12), done.
    Total 196 (delta 11), reused 7 (delta 6), pack-reused 178
    <SNIP>
    INFO[0014] Pushed index.docker.io/jollygoodhorsepower/kaniko-demo-image@sha256:e9ae708cddfb30e0dc0ec5fa4580e878b045ba372f46c54a1500a81750aa7205
    ```

## TODO
* Add [Dockerfile](Dockerfile) instead of using [ConfigMap](configmap.yml) in [kaniko-dir.yml](kaniko-dir.yml)
* Create justfile

## Further Reading
[Pull an Image from a Private Registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

[tiangolo/uwsgi-nginx-flask-docker](https://github.com/tiangolo/uwsgi-nginx-flask-docker)

[Secrets](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/)

[ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)
