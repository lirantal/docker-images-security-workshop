# Docker Image Security workshop

## 1. Challenge: Find the most vulnerability-free image

Demo OS-level vulnerabilities made possible by a vulnerability in a library installed on the operating system, and used by an application: [https://github.com/lirantal/goof-container-breaking-in](https://github.com/lirantal/goof-container-breaking-in)

In this challenge you should find and fix vulnerabilities in docker images that you test and build in a local environment, or from your CI.

Starting point is this goofy project called `docker-goof` which uses an old version of Node.js runtime as a docker image tag.

1. Clone the project: [https://github.com/snyk/docker-goof](https://github.com/snyk/docker-goof)
2. Inside it you will find a `Dockerfile` with the image to use
3. Find how many vulnerabilities (low, medium, and high) the image has. Are there any interesting vulnerabilities?

<details><summary>Hint</summary>
<br/>

You can use the open source Snyk CLI to scan the image.
See [Snyk CLI install instructions](https://support.snyk.io/hc/en-us/articles/360003812458-Getting-started-with-the-CLI) to get started.

<br/>
</details>

4. Did you find vulnerabilities in the Node.js runtime as well? give an example of one of them and which version of Node.js will fix it.
5. How would you fix these vulnerabilities?
6. Find which official Node.js image you could switch to in order to lower your vulnerabilities footprint without disrupting the product and development teams too much.

<details><summary>Hint</summary>
<br/>

If the Snyk CLI is provided with a Dockerfile it will give you a remediation advice so you can make a conscious decision of which image you could move to in order to lower the security vulnerabilities foot-print.

See [Snyk CLI install instructions](https://support.snyk.io/hc/en-us/articles/360003812458-Getting-started-with-the-CLI) to get started.

<br/>
</details>

<details><summary>Solution</summary>
<br/>

```
snyk test --docker node:10.4.0 --file=Dockerfile
```

<br/>
</details>

Don't forget -
Lowest vulnerabilities & ease of upgrade wins the challenge!

### Registry workflow

Find and fix vulnerabilities in docker images in a Docker Hub registry or others.

Bonus challenge - how do you monitor your docker images on a container registry and do you have an actionable advice as to which image and tag you should change to in order to lower the footprint of your image's security vulnerabilities?

## 2. Challenge: Don't steal my immutability!

Your team needs to update to the latest version of Node.js 10 LTS to get fixes and keep up to date with security vulnerabilities.

You do the obvious:

```
docker pull node:10
```

1. Can you think of some downsides to pulling the image this way?

### Use fixed tags for immutability

When you pulled the image you specified the image name and a tag. But what's the implications of using a tag like that?

> Each Docker image can have multiple tags, which are variants of the same images. The most common tag is _latest_, which represents the latest version of the image. Image tags are not immutable, and the author of the images can publish the same tag multiple times.
> This means that the base image for your Docker file might change between builds. This could result in inconsistent behavior because of changes made to the base image.

_source: [10 Docker Image Security Best Practices](https://snyk.io/blog/10-docker-image-security-best-practices/)_

The above best practices document stresses that we should be as specific as possible in our tags, and ideally we'd pull in the image by its SHA256 reference.

2. Pull in the image based on its SHA256

<details><summary>Hint 1</summary>
<br/>

If you pulled the `node:10` image, take a look at the output

<br/>
</details>

<details><summary>Hint2</summary>
<br/>

You're looking for the `Digest` key in the output of `docker pull`

<br/>
</details>

<details><summary>Solution: how to pull the image by its SHA256</summary>
<br/>
   
```
docker pull node@sha256:bdc6d102e926b70690ce0cc0b077d450b1b231524a69b874912a9b337c719e6e
```

<br/>
</details>

## 3. Challenge: Signed images for extra trust

Docker Hub is a software marketplace in the form of containers. Just like any other open marketplace, this means that Docker Hub is also a hub for supply-chain attacks and you may pull malicious images, either on their own or those of official sources.

To further provide you with safety net as authority and integrity of the image you pull it is possible to enable docker's content trust.

Enabling Docker's content trust policy:

```
export DOCKER_CONTENT_TRUST=1
```

Try your luck with some of your favorite sources for docker images and pull them away. Did it work?

No ideas for docker images? no worries!
Try to pull this docker image, which should fail because it isn't a signed image:

```
docker pull azukiapp/busybox
```

How can we check if an image is trusted?

<details><summary>Solution</summary>
<br/>
   
```
docker trust inspect --pretty azukiapp/busybox
```

And then inspect the output for a trusted image:

```
docker trust inspect --pretty node:10
```

<br/>
</details>

Enabling content trust means you are only able to pull, run or build trusted images.

### Let's build a signed docker image

Pre-requisite: you will need a Docker Hub account to be able to push your own signed image. If you don't have it, please create one.

1. In the `signed-image/` directory you will find a sample `Dockerfile` that we will use to build and then sign when we push it to the Docker Hub registry.
2. Build the docker image

<details><summary>Hint</summary>
<br/>
   
```
docker build -t lirantal/docker-image-security-workshop:signed .
```

Note: you probably want to change the `lirantal` account to your own.
<br/>

</details>

3. Push the image to Docker Hub, which will now prompt you for passphrases for the generated keys that it created during the process. Why is it now doing this? because we enabled Docker's Content Process with that environment variable before.

Note: provide it with passphrases that you will easily remember for the workshop. It's just an exercise, so I went with `passphrase1` and `passphrase2` for both keys.

<details><summary>Hint</summary>
<br/>
   
```
docker push lirantal/docker-image-security-workshop:signed
```

Note: you probably want to change the `lirantal` account to your own.
<br/>

</details>

4. Ask a friend to pull the signed image you just pushed. Did it work for them?

Both the `root key` and the `repository key` should be available in `~/.docker/trust/private/`. Want to learn more about how DCT works? [Learn here](https://docs.docker.com/engine/security/trust/content_trust/).

Wondering about how to automate this process in CI and build processes? [See this](https://docs.docker.com/engine/security/trust/trust_automation/) about key delegation

## 4. Challenge: Really bad practices by default

Reminder: if you're coming here from previous challenges, did you remember to turn Docker's Content Trust policy off? Right, you need to do that:

```
export DOCKER_CONTENT_TRUST=0
```

How do you help your devops and developer engineers to validate the proper configuration of a `Dockerfile` when they are building them? Aaaaaaaaa-utomation!

We'll visit a couple of static code analysis tools to help us find out issues in a `Dockerfile`, or what us in the JavaScript land like to call - linters. There are a couple I recommend, and you're welcome to try both of them:

1. [Dockle](https://github.com/goodwithtech/dockle) - Puts a focus on scanning the image through its layers.
2. [Hadolint](https://github.com/hadolint/hadolint) - Statically analyze the Dockerfile

For this challenge we'll work with the project in the `bad-defaults/` directory.
Use any of the linters above and test the Docker file/docker image.

Only after you found issues with either or both of the above linters should you continue to the issues below in order to fix them.

### Container is running as root

1. Build the bad defaults image

<details><summary>Hint</summary>
<br/>
   
```
docker build -t best-practices .
```

<br/>
</details>

2. Check if the container is running with the root user. If so, that's not good.

<details><summary>Solution</summary>
<br/>
   
```
docker run -it --rm best-practices:latest sh
```

Inside the container run:

```
whoami
```

<br/>
</details>

3. Fix the issue so that the container uses a non-privileged user

<details><summary>Solution</summary>
<br/>
Update the Dockerfile to make use of the built-in `node` user:
   
```
...
USER node
CMD node index.js
```

Now build the container and run again to check which is user is being used.

<br/>
</details>

## 5. Challenge: All your secrets belong to me!

Did the `Dockerfile` smell somewhat fishy to you in the previous challenge?
It was smelly of secrets and tokens!

I am specifically referring to this entry in the `Dockerfile`:

```
RUN echo "//registry.npmjs.org/:_authToken=$NPM_TOKEN" > .npmrc
```

1. First assignment, can you think about why this is bad?
   After all, we're building the image in an internal environment and only we pass that secret token at build time. No one else sees it.

<details><summary>Hint 1</summary>
<br/>

The `.npmrc` file contains sensitive information, such as a token which is used for read/write for private packages on a registry. If the container is compromised, users will be able to access it.

Can you think of a simple vulnerability in an application that will allow a malicious attacker to easily get to the `.npmrc` file?

You can build it yourself and try:

```
export NPM_TOKEN=<npm token>
docker build -t best-practices --build-arg NPM_TOKEN=$NPM_TOKEN .
```

Now login to the container and validate the value of `.npmrc`:

```
docker run -it --rm best-practices sh
```

<br/>
</details>

2. Can you think of some other ways to fix this issue of secrets leaking? Below are hints for 2 "solutions". They might prove like a good idea for you but we explain why they shouldn't be followed.

<details><summary>Bad practice 1</summary>
<br/>

You remember to remove the token, such as:

```
RUN echo "//registry.npmjs.org/:_authToken=$NPM_TOKEN" > .npmrc
RUN npm install
RUN rm .npmrc
```

Nice, but not really good.
Every `RUN` creates another layer, all of which are later inspect-able and leave a trace. This means that if the image itself is ever leaked or made public then sensitive data exists inside it in the form of the `.npmrc` file.

<br/>
</details>

<details><summary>Bad practice 2</summary>
<br/>

You understand the concept of Docker layers so you put all of this into one command to make sure there's no trace, something like this:

```
RUN echo "//registry.npmjs.org/:_authToken=$NPM_TOKEN" > .npmrc && \
    npm install && \
    rm .npmrc
```

However, Docker has this thing called commits history which it uses to save metadata about the way the image was built and this is why you should never really use environment variables such as build arguments for sensitive storage such as passwords, API keys, tokens.

Read more about [Docker history here](https://docs.docker.com/engine/reference/commandline/history/).

<br/>
</details>

<details><summary>Hint: for proper solution</summary>
<br/>

What if you could create a Docker image without the `.npmrc` file in it?

<br/>
</details>

<details><summary>Solution</summary>
<br/>

Let's use multi-stage builds to fix it!

Update the `Dockerfile` so that the first image is used as a base to install all of our npm dependencies and build what is required. To do that, update the FROM instruction as follows:

```
FROM bitnami/node:latest AS build
```

As well as remove the `CMD` instruction which isn't needed.
Then have another section in the Dockerfile for the "production" image, which should use the app directory which is now ready for use from the previous build image.

Following is an example:

```
FROM bitnami/node:latest
RUN mkdir ~/project
COPY --from=build /app/~/project ~/project
WORKDIR ~/project
CMD node index.js
```

<br/>
</details>
