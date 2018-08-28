---
title: "Building and deploying docker images from Travis to ECR"
date: "2016-12-22"
description: "I used to use Quay as a private registry but found that when heavily relying on AWS just using ECR gets the job done nicely and is a bit cheaper. However, AWS doesn’t give you a build environment like Quay does. So you have several options: Setting up your own, pushing images from the developer machine or simply using your already existing CI infrastructure to build and push your images."
categories:
  - "Software Engineering"
  - "Devops"
  - "Docker"
tags:
  - "Docker"
  - "Devops"
  - "Travis"
  - "CI"
---

<i>This article assumes that you're generally familiar with Docker, Amazon IAM, Travis CI and some basic bash scripting.</i>

I used to use Quay as a private registry but found that when heavily relying on AWS just using ECR gets the job done nicely and is a bit cheaper. However, AWS doesn't give you a build environment like Quay does. So you have several options: Setting up your own, pushing images from the developer machine or simply using your already existing CI infrastructure to build and push your images.

For one of my recent projects, I chose to use Travis to push the images to <a href="https://aws.amazon.com/ecr/">ECR</a>. Having a proper CI process in place ensures, that no images are pushed before they pass all the required tests. Since Travis can execute arbitrary scripts, it's quite easy to get it to build and push your image. No need for an extra Docker build environment.

<h2>The deploy script</h2>

Below you can see the little bash script, that is executed in <code>after_success</code> and does the magic. It's quite easy to follow and customize to your needs, so I won't describe it in detail. You can either place this script in your project repo or on a remote location and just curl it when you need it.

<pre>
#!/usr/bin/env bash

if ! [ $TRAVIS_PULL_REQUEST == "false" ]; then
  echo "This is a pull request. Skipping docker build and ECR deployment.";
  exit 0;
fi

TAG=`if [ "$TRAVIS_BRANCH" == "master" ]; then echo "latest"; else echo $TRAVIS_BRANCH ; fi`
COMMIT=${TRAVIS_COMMIT::8}

docker --version
pip install --user awscli
export PATH=$PATH:$HOME/.local/bin
eval $(aws ecr get-login --region us-east-1)

docker build -t $DOCKER_REPO .

docker tag $DOCKER_REPO $DOCKER_ECR/$DOCKER_REPO:$TAG
docker push $DOCKER_ECR/$DOCKER_REPO:$TAG
</pre>

<h2>Travis file</h2>

To call this script, you can simply add it to the <code>after_success</code> section in your <code>travis.yml</code>.

<pre>
sudo: required
language: node_js
node_js:
  - '6'
services:
  - docker
env:
  global:
    - DOCKER_ECR=ACCOUNTID.dkr.ecr.us-east-1.amazonaws.com
    - DOCKER_REPO=REPO_NAME
after_success:
  - ./deploy_ecr
</pre>

Don't forget to set your <code>AWS_ACCESS_KEY_ID</code> and <code>AWS_SECRET_ACCESS_KEY</code> environment variables.

<h2>IAM policy</h2>

To allow your IAM user to acquire a temporary auth token for ECR, you'll need to attach the following policy (or alternatively more fine grained if you want).

<pre>
{
	"Version": "2012-10-17",
	"Statement": [{
		"Effect": "Allow",
		"Action": [
			"ecr:GetAuthorizationToken"
		],
		"Resource": [
			"*"
		]
	}]
}
</pre>

<h2>ECR repository permissions</h2>

And to make things even more fun, you'll also need to set a policy on your actual ECR repo. Below you can see an example policy specifying a 'WritePolicy' for the IAM user you use to push your Docker images and a 'ReadPolicy' for an EC2 instance role to get images. This is just an example and you should adapt this to your own needs.

<pre>
{
	"Version": "2008-10-17",
	"Statement": [{
		"Sid": "WritePolicy",
		"Effect": "Allow",
		"Principal": {
			"AWS": "arn:aws:iam::123456789:user/ecr-deployer"
		},
		"Action": [
			"ecr:DescribeRepositories",
			"ecr:GetRepositoryPolicy",
			"ecr:ListImages",
			"ecr:DescribeImages",
			"ecr:DeleteRepository",
			"ecr:BatchDeleteImage",
			"ecr:SetRepositoryPolicy",
			"ecr:DeleteRepositoryPolicy",
			"ecr:GetDownloadUrlForLayer",
			"ecr:BatchGetImage",
			"ecr:BatchCheckLayerAvailability",
			"ecr:PutImage",
			"ecr:InitiateLayerUpload",
			"ecr:UploadLayerPart",
			"ecr:CompleteLayerUpload"
		]
	}, {
		"Sid": "ReadPolicy",
		"Effect": "Allow",
		"Principal": {
			"AWS": [
				"arn:aws:iam::123456789:role/SomeServerRole"
			]
		},
		"Action": [
			"ecr:GetDownloadUrlForLayer",
			"ecr:BatchGetImage",
			"ecr:BatchCheckLayerAvailability"
		]
	}]
}
</pre>

I think it's a fairly easy setup and if you already pay for Travis, why not use it also to build your Docker images. I hope this little guide was helpful and you can enjoy your new Travis/ECR setup.
