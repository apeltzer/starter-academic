+++
title = "Running nf-co.re pipelines with AWSBatch"

date = 2018-08-21T00:00:00
lastmod = 2018-08-22T22:15:00
draft = false

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Alexander Peltzer", "Tobias Koch"]

tags = ["Bioinformatics", "NF-Core", "Nextflow", "AWS", "Cloud Computing"]
summary = "How to run AWSBatch with NF-Core pipelines using Nextflow."
+++

This document is based on experiences provided by [Maxime Garcia](https://maxulysse.github.io/2017/11/16/Running-CAW-with-AWS-Batch/) at his blogpost on running CAW with AWSBatch and our own experiences. We just wanted to acknowledge the effort that Maxime also put into making this work!

Basic Requirements
------------------

In order to run this, you need to have a [AWS
account](https://www.google.com/url?q=http://aws.amazon.com&sa=D&ust=1534850604063000) set up and create an IAM user for it. More detailed information on how to do this can be found under the above link. The IAM set up is described in the [next section](#h.iamsetup).

### [IAM User setup](#h.iamsetup)

Please follow the instructions on AWS to set up an IAM user for running
Batch jobs on AWSBatch. This user has to be provided with the required
permissions to run Batch jobs. Permissions on AWS are set up using [the IAM service](https://www.google.com/url?q=https://console.aws.amazon.com/iam&sa=D&ust=1534850604070000) and you have to start by creating a new user using the "Add user" interface. Please attach the following permissions to your newly created user afterwards:

- `AmazonEC2FullAccess`
- `AmazonS3FullAccess`
- `AWSBatchFullAccess`

This should already suffice in terms of user permissions to run jobs on
AWS Batch.

### [Service role setup](#h.servicerolesetup)

Next, we need to set up permissions for AWS Batch to e.g. run/stop EC2
instances for us by generating the required roles on IAM. These are
managed as separate roles in IAM under "Roles". Some of these are
created automatically for you when you set up a Batch job at the first
time, but we can make sure that these are already present and configured
properly beforehand. 

You need the service roles:

- `AWSBatchServiceRole`
- `ecsInstanceRole`
- `AWSServiceRoleForEC2SpotFleet`
- `AmazonEC2SpotFleetRole`

The latter two are not required if you are not intending to use spot
instances (you should use them to save money!).

#### AWSBatchServiceRole

This role should have the policy `AWSBatchServiceRole` attached.
![](https://raw.githubusercontent.com/apeltzer/starter-academic/master/static/img/2018-08-21_awsnfcore//image7.png)

#### ecsInstanceRole

This role should have the policies `AmazonS3FullAccess` and
`AmazonEC2ContainerServiceforEC2Role` set.

![](https://raw.githubusercontent.com/apeltzer/starter-academic/master/static/img/2018-08-21_awsnfcore//image1.png)

#### AWSServiceRoleForEC2SpotFleet

This role should have the policy `AWSEC2SpotFleetServiceRolePolicy`
set.

![](https://raw.githubusercontent.com/apeltzer/starter-academic/master/static/img/2018-08-21_awsnfcore//image6.png)

#### AmazonEC2SpotFleetRole

This role should have the policies `AmazonEC2SpotFleetRole` and
`AmazonEC2SpotFleetTaggingRole` set accordingly.

![](https://raw.githubusercontent.com/apeltzer/starter-academic/master/static/img/2018-08-21_awsnfcore//image8.png)

Once you have set these roles, the permissions for running a job with
the selected IAM user can be used to configure a compute environment
(CE) and a job queue to run your jobs.

You need to get the accesskey/token combination for using AWS and need
to have these present on the machine starting the jobs in a file
`~/.aws/credentials` such as:

```bash
[default]
aws_access_key_id = KEY
aws_secret_access_key = secretKEY
```

AMI
---

Now that you have all the permissions set up you need to create a
custom amazon machine image (AMI). This will be used later by the EC2
instances started by AWS batch.

### AMI preparation

First you have to set up an EC2 instance which will later be converted
into an AMI. Please use the ECS-Optimized Amazon Linux AMI since it
provides docker installation and configuration.

![](https://raw.githubusercontent.com/apeltzer/starter-academic/master/static/img/2018-08-21_awsnfcore/image3.png)

Choose a t2.micro instance for the instance type since the instance
type does not impact the AMI.

![](https://raw.githubusercontent.com/apeltzer/starter-academic/master/static/img/2018-08-21_awsnfcore/image4.png)

Configuration of the Instance Details is not needed as the default
configuration should suffice.

![](https://raw.githubusercontent.com/apeltzer/starter-academic/master/static/img/2018-08-21_awsnfcore/image9.png)

Depending on the docker image sizes you expect your Batch instances to
handle, choose a larger EBS storage in the storage configuration.

![](https://raw.githubusercontent.com/apeltzer/starter-academic/master/static/img/2018-08-21_awsnfcore/image5.png)

Optional Tags can be added in Step 5.

![](https://raw.githubusercontent.com/apeltzer/starter-academic/master/static/img/2018-08-21_awsnfcore/image2.png)

In the last step configure your Security Group. Make sure you can
connect to the EC2 instance when doing so. You can also let AWS create a
security group for you.Now launch the instance.

Connect to your t2.micro instance and perform the following
steps:

- Update the yum repository
- `sudo yum update`
- Check the docker configuration and make sure the docker storage size matches your configured one.
- `docker info`
- Check whether awscli is installed
- `aws --version`
- Install awscli using miniconda

```bash
wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86\_64.sh
bash Miniconda3-latest-Linux-x86\_64.sh -p /home/ec2-user/miniconda
/home/ec2-user/miniconda/bin/conda install -c conda-forge awscli
/home/ec2-user/miniconda/bin/aws --version
```

### AMI creation

Now that you have configured your EC2 instance you can logout and stop
the instance.

In the EC2 Management Console select the stopped instance and choose
the Action Image-\>Create Image

Choose a name and a description and create the AMI.

## Set up QUEUE / Compute Environment 


In order to submit jobs to AWS Batch, you need to have a working
compute environment (CE) and a JobQueue set up. Start with the CE and
then create a compute environment.

### Compute Environment 

- Log in to AWS Batch
- Navigation menu: "Compute environments"
- "Create a new compute environment"
- Select "Managed"
- Provide a name for the Compute Environment
- Service role `AWSBatchServiceRole`
- Instance role `ecsInstanceRole`
- EC2 key pair: The key pair of the IAM user you intend to use for
    running AWS Batch jobs
- Select "on-demand" or "spot" (spot is cheaper)
- Select Maximum Price you're willing to pay for spot instances
- Spot fleet role `AmazonEC2SpotFleetRole`
- Enable user-specified AMI id and provide the
    AMI id you created
- If desired, attach a Key/Value pair to make it possible to later
    generate e.g. billing information on a per compute environment
    basis

Select create and you're all set!

### JobQueue set up

- Log in to AWS Batch
- Select "Job queues" in Navigation menu
- Provide a name you remember (!) for your job queue
- Set a priority
- Select the previously created compute environment for running jobs

Select create and you're all set!

## AWS Configuration Nextflow

AWS Batch configuration in Nextflow requires a couple of things:

- Publicly accessible Docker containers per process or per pipeline
- Executor set to `awsbatch`
-   JobQueue set to an enabled AWS Batch JobQueue
- An AWS region, e.g. `'us-east-1'`
- AWSCli present in the AMI which runs the jobs
- S3 Buckets for temporary/work files and results

An example on how to deal with things is to ask your users to specify
the missing parts on pipeline execution. A working example is set in
[ICGC-featureCounts](https://github.com/qbicsoftware/ICGC-featureCounts/blob/master/conf/awsbatch.config&sa=D&ust=1534850604086000)
where the `awsbatch.config` specifies the required parameters for
execution on AWSBatch.

- workDir
- Executor
- Queue
- Region 
- And a custom path to AWScli inside the customized AMI to schedule jobs

This is then set to default values in the
[nextflow.config](https://github.com/qbicsoftware/ICGC-featureCounts/blob/master/nextflow.config&sa=D&ust=1534850604088000) of the pipeline and set according to user specified parameters when executing the pipeline. This way, users don't need to change the `awsbatch.config` file but can instead rely on using a set of parameters `--workDir, --awsqueue, --awsregion` and should be fine.


## Running a basic job with AWS Batch

You can execute an AWSBatch job on your local workstation or on a
running EC2 instance. For longer running jobs, it might make sense to
use a small EC2 instance (t2.micro) to run your job, as you're not
relying on network connections between your local workstation and the
AWS Batch CE/JobQueue then. 

You can use `nextflow cloud create <your-cloud-name>` to launch
a simple instance that has nextflow installed to create a simple
instance for running AWS Batch jobs.

For this instance additional configuration can be provided in your
`nextflow.config`or `~/.nextflow/config`file. 

```bash
cloud {
 imageId = 'ami-xxxx'
 instanceType = 't2.micro'
 keyName = <AWS keyname>
 userName = 'ec2-user'
}
```

## Troubleshooting

### I don't want to risk requesting instances when nothing runs anymore - what should I do?

Simply disable the JobQueue and the Compute Environment and nothing
will be requested when there are no jobs running. In our experience,
failing jobs will also result in instance termination.

### I have configured nextflow according to your suggestion but every job fails with Essential container in task exited - how can I fix that?

Open CloudWatch and look at the log for the failed task. If the error
is `aws command not found` or `bash: /home/ec2-user/miniconda/bin/aws`:
No such file or directory' then make sure aws is installed and there are
no typos in your `nextflow.config`.

### I found an AMI option for the nextflow.config file but it doesn't work - is it broken?

No, the `cloud.ami` configuration is only used for cluster creation. AWS
Batch is a different service and the AMI needs to be specified inside
the compute environment.

### There is no `--workDir` option. Where can I find it?

The `--workDir, --awsqueue ,--awsregion` options are provided by the
pipeline. If your pipeline is no nf-core pipeline those options won't be
present. You would have to specify queue and region in your
`nextflow.config` file. You can also provide the work directory with the
nextflow parameter `-work-dir / -w`.