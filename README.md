# CDK deployment
You can choose the CDK (Cloud Deveployment Kit) for deployment.

[CDK Completable Version][cdk-completable-version]


# Referemce

1. [maarten-thoelen][maarten-thoelen]

# Prerequisites
- Install Node.js (https://nodejs.org/en/download/)
- Install AWS CLI (https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)
- Install AWS SAM (https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)
- Install Docker (https://docs.docker.com)

# Development / testing

1. Create RDS

    To test our code we will use the AWS CLI to setup a MySQL database on a db.t2.micro instance.( Wating the status from `creating` to `active` - around 10 - 15 min)

        $ aws rds create-db-instance --db-instance-identifier schedulelambda-dev-mysql-test --db-instance-class db.t2.micro --engine mysql --allocated-storage 20 --master-username admin --master-user-password ${YOUR-RDS-PASSWORD}

&thinsp;

2. Create S3

    AWS SAM will need a S3 bucket to store its deployment artifacts. We create one using the cmd below.

        $ aws s3 mb s3://aws-sam-db-auto-shutdown-deployment-artifacts

&thinsp;

3. Testing

    ##### Invoke lambda on local machine (needs Docker to be installed)

    Once the database is active, let’s see if we can get it back to life by running the command below.

    ```bash
    $ sam local invoke --no-event DBShutDownFunction
    ```

    As expected, we see that the database is `Active` and becomes `stopped` after a while.

&thinsp;

4. Deployment
   
    We are now sure our code works, so let’s package our Lambda functions and deploy them to AWS.

    ### Package

    ```bash
    $ sam package --template-file template.yaml --s3-bucket aws-sam-db-auto-shutdown-deployment-artifacts --output-template-file outputTemplate.yaml
    ```

    This command will upload the artifacts to the S3 bucket we specified and output a new template file called outputTemplate.yaml. We will use this new template file in the next step.

&thinsp;

5. Deploy

    This command will create a changeset and execute it. The execution in our case will create a new CloudFormation stack.

    ```bash
    $ sam deploy --template-file outputTemplate.yaml --stack-name aws-sam-db-auto-shutdown-deployment-stack --capabilities CAPABILITY_IAM
    ``` 

    Now that our code is deployed, we should find our Lambda functions in the AWS Management Console.


# Clean up

A word of advice, always clean up any resources you aren’t using anymore.

In order to delete the CloudFormation stack and all the resources it created, just run the command below.

    $ aws cloudformation delete-stack --stack-name aws-sam-db-auto-shutdown-deployment-stack 

Delete the test database

    $ aws rds delete-db-instance --db-instance-identifier schedulelambda-dev-mysql-test --skip-final-snapshot

Delete S3 bucket

    $ aws s3 rb s3://aws-sam-db-auto-shutdown-deployment-artifacts --force

<!-- Reference -->

[maarten-thoelen]:https://medium.com/hatchsoftware/saving-money-by-automatically-shutting-down-rds-instances-using-aws-lambda-and-aws-sam-925fd86592b5

[cdk-completable-version]:https://github.com/Lcmkey/aws-cdk-scheduled-lambda-with-rds