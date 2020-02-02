# codedeploy-blue-green
These artifacts are associated with the blog post on Blue/Green Deployments with CodeDeploy located at:

https://aws.amazon.com/blogs/devops/performing-bluegreen-deployments-with-aws-codedeploy-and-auto-scaling-groups/

Amazon Web Services offers services that enable organizations to leverage the power of the cloud for their development and deployment needs. AWS CodeDeploy makes it possible to automate the deployment of code to either Amazon EC2 or on-premises instances. AWS CodeDeploy now supports blue/green deployments. In this blog post, I will discuss the benefits of blue/green deployments and show you how to perform one.

# Step by step guide

Step 1: Create the initial environment

1. Download an archive containing the sample template from this [location](https://github.com/awslabs/codedeploy-blue-green) and save it in a convenient location.
2. Sign in to the AWS Management Console and open the AWS CloudFormation console at https://console.aws.amazon.com/cloudformation/.
3. If this is a new AWS CloudFormation account, click **Create New Stack**. Otherwise, click **Create Stack**.
4. Under **Upload a template to Amazon S3**, click **Choose File**, choose the YAML file from the archive you downloaded, and then click **Next**.
5. In **Specify Details**, in **Stack name**, type `bluegreen`.
6. In **AZName**, select one of the Availability Zones. (In this blog post, I am using us-east-1a.)
7. In **BlueGreenKeyPairName**, select the key pair to use.
8. In **NamePrefix**, use the default value of `bluegreen` unless you are already running an application with a name that starts with `bluegreen`. The name prefix is used to assign name tags to the created resources. Click **Next**.
9. On the Options page, click **Next**.
10. Select the acknowledgement box to allow the creation of IAM resources, and then click **Create**. It will take CloudFormation about 10 minutes to create the sample environment. In addition to creating the infrastructure resources shown in the diagram, the CloudFormation template also sets up an AWS CodeDeploy application and blue/green deployment group.

Step 2: Review initial environment

1. Look at the CloudFormation stack outputs. You should see something similar to the following. **WorkstationIP** is the IP address of the workstation instance. **AutoScalingGroup** and **LoadBalancer** are the DNS names created by CloudFormation for the Auto Scaling group and the Elastic Load Balancing load balancer.
   ![img](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2017/04/19/CloudFormationOutputs2-1.png)
2. Copy the **LoadBalancer** value into your browser and browse to that link. The following application should be displayed. This PHP application queries the Amazon EC2 instance metadata. If you refresh the page, you will see the IP address and instance ID change in accordance with the round-robin load balancing algorithm.
   ![img](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2017/04/19/appversion1-v2-1.png)
3. Go to the EC2 console and display the instances. You will see three running instances associated with this example: the workstation and the two web server instances created by the Auto Scaling group. The web server instances make up the blue environment.
   ![img](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2017/04/19/instances1-v2.png)

Step 3: Deploy the new version of code

1. [Connect to the workstation instance](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstances.html) at the address displayed in **WorkStationIP**. This instance is running the Ubuntu operating system, so the user name is **ubuntu**. After you sign in, you will see two directories. The scripts directory contains Bourne shell scripts. The newversion directory contains an update to the PHP application.
2. Here is the PHP code for the new version in newversion/content/index.php. The only difference from the initially installed code is the application version number.
   ![img](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2017/04/19/newappcode-1.png)
3. Now look at the following scripts/pushnewversion.sh shell script. It uses the **aws deploy push command** to bundle the code and upload it to Amazon S3.
   ![img](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2017/04/19/pushversion.png)
4. Run the pushnewversion.sh script. You will see a message that tells you how to deploy the code with the AWS command line interpreter, but we will use the AWS CodeDeploy console to do this instead.
5. Open the AWS CodeDeploy console at https://console.aws.amazon.com/codedeploy.
6. Click the link for bluegreen-app. If you chose a name other than the default for **NamePrefix**, click that name instead. Expand **Revisions**. You will see the revision you just pushed from the AWS CodeDeploy workstation. Click **Deploy revision**.
7. On the **Create deployment** page, select the bluegreen-app application and the bluegreen-dg deployment group. Leave all the other default values in place, and then click **Deploy**. AWS CodeDeploy will provision the Auto Scaling group and instances, deploy the code, set up health checks, and redirect traffic to the new instances. This process will take a few minutes. When the deployment is complete, the deployment should appear, as shown here. AWS CodeDeploy skips the termination of the original instances because of the settings in the deployment group.
   ![img](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2017/04/19/codedeployresults.png)

Step 4: Review the updated environment

1. Browse to the DNS name for the load balancer. You should see the new version of the application, as shown here. The application version has changed from 1 to 2, as expected.
   ![img](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2017/04/19/appversion2-v2.png)
2. Go to the EC2 console and display the instances. You will see four instances that have been tagged by the Auto Scaling group and launch configuration. The instances with IP addresses 10.200.11.11 and 10.200.11.192 are the ones we saw before in the blue environment. The deployment process created the instances with IP addresses 10.200.11.13 and 10.200.22 that are now part of the green environment.
   ![img](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2017/04/19/instances2-v2-1.png)
3. Go to the [Auto Scaling console](https://console.aws.amazon.com/ec2/autoscaling/home). You will see that there are now two Auto Scaling groups, each of which has two instances. The Auto Scaling group whose names begins with CodeDeploy was created during the deployment process.
   ![img](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2017/04/19/AutoScaling2-v2-1.png)

You have now successfully completed a blue/green deployment using AWS CodeDeploy.

Step 5: Cleanup

1. Return to the session on the AWS CodeDeploy workstation.
2. Run the scripts/cleanup.sh script. This will remove the deployment bundle and shut down the Auto Scaling groups.
3. Go to the CloudFormation console, select the stack you created, and delete it.
