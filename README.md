# Amazon EKS Custom AMI Build Specification

This repository contains sample scripts and [AWS CloudFormation](https://aws.amazon.com/cloudformation/) templates for building Amazon EKS worker nodes. Currently all CloudFormation templates make use of [Amazon EC2 Image Builder](https://aws.amazon.com/image-builder/) for AMI builds. These templates make use of the [awslabs Amazon EKS AMI Build Specification repository](https://github.com/awslabs/amazon-eks-ami) and the [Red Hat Enterprise Linux (RHEL) specific variant repository](https://github.com/aws-samples/amazon-eks-ami-rhel).

## 🚀 Getting started

If you are new to Amazon EKS, we recommend that you follow our [Getting Started](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html) chapter in the Amazon EKS User Guide. If you already have a cluster, and you want to launch a node group with your new AMI, see [Launching Amazon EKS Worker Nodes](https://docs.aws.amazon.com/eks/latest/userguide/launch-workers.html).

## 🔢 Pre-requisites

* AWS Account with permissions to build AMIs using EC2 Image Builder.
* Internet connectivity from EC2 for file downloads OR files stored locally in S3 bucket.

## 🗂️ Cloning the Github repository
```bash
git clone https://github.com/aws-samples/amazon-eks-custom-builds.git && cd amazon-eks-custom-builds

```

## 👷 Building the AMI

Building an EKS worker node is as simple as modifying one of the sample CloudFormation templates to your liking, deploying it your AWS account with your parameter decisions, and executing the resulting [EC2 Image Builder Pipeline](https://docs.aws.amazon.com/imagebuilder/latest/userguide/how-image-builder-works.html).

This could be done using an AWS CLI command using the default parameters defined in the CloudFormation template.
```bash
aws cloudformation create-stack --stack-name my-eks-worker-stack --template-body file://path/to/your/template.yaml

```

## 🔒 Security

For security issues or concerns, please do not open an issue or pull request on GitHub. Please report any suspected or confirmed security issues to AWS Security https://aws.amazon.com/security/vulnerability-reporting/

## ⚖️ License Summary

This library is licensed under the MIT-0 License. See the LICENSE file.

## 📝 Legal Disclaimer

The sample code; software libraries; command line tools; proofs of concept; templates; or other related technology (including any of the foregoing that are provided by our personnel) is provided to you as AWS Content under the AWS Customer Agreement, or the relevant written agreement between you and AWS (whichever applies). You should not use this AWS Content in your production accounts, or on production or other critical data. You are responsible for testing, securing, and optimizing the AWS Content, such as sample code, as appropriate for production grade use based on your specific quality control practices and standards. Deploying AWS Content may incur AWS charges for creating or using AWS chargeable resources, such as running Amazon EC2 instances or using Amazon S3 storage.