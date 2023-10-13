# Deploying-AWS-Resources-Using-Terraform-Docker-Jenkins-Pipeline-Provision-EC2-Instance

***************************************************************************************************************************************************************************************
CREATE A S3 BUCKET FOR BACKEND
***************************************************************************************************************************************************************************************

It’s a best practice to treat your state file as a secret and store it remotely. This adds increased security, but also allows collaboration among team members. If the state file is stored locally on your computer then obviously your team doesn’t have easy access to it. One thing to keep in mind is state locking. You want to ensure your backend option locks your state when someone is provisioning resources. If two team members try to make changes at the same time, the state file could become corrupted. I say this because although most backend options do this automatically, S3 does not. For this project, I did not enable state locking as I was the only person working on the project and there was no risk of my changes colliding with someone else. If you’d like to add state locking to your S3 backend please review this Terraform documentation.

Create S3 Bucket using the AWS CLI. Be sure to create a unique name. The best way to accomplish this is to add random numbers and characters to the end of the bucket name. Feel free to name your S3 Bucket something different
1. $ aws s3api create-bucket --bucket <name of bucket with random numbers > --region <your region>
2. Run aws s3 ls to verify your new bucket has been created.


***************************************************************************************************************************************************************************************
CREATE JENKINS SERVER
***************************************************************************************************************************************************************************************

We could spin up an EC2 instance and configure Jenkins manually but to make things easier, we will just grab an AMI from the AWS Marketplace. There will be a small cost associated with this because of the instance type. Jenkins is open source and does incur a cost.

*If you would like to install Jenkins yourself for bonus points, check out my project to install Jenkins using an Ansible Playbook.*

Navigate to AWS Marketplace.
Click Discover products.
Search for “Jenkins” and select Jenkins Certified by Bitnami.

4. Feel free to review the Overview of the AMI. As you can see from the pricing, t2.micro is not offered for some reason, which would have placed us in the free tier. As long as you keep track of your usage the costs should be minimal for this project as we will be using a t3a.micro which is $0.009/hr.

5. Click the Continue to Subscribe button.

6. Click the Continue to Configuration button.

7. Click the Continue to Launch button.

8. For Choose Action select Launch from Website.

9. For EC2 Instance Type select t3a.micro. (If t2.micro is available then use that)

10. For VPC Settings select the VPC you’d like to use. I’m using my default VPC.

11. For Subnet Settings select your desired subnet. I’m using the default listed.

12. For Security Group Settings select a security group that has SSH, HTTP, HTTPS configured. If you don’t have one then you can create one by clicking the Create New Based On Seller Settings button. Be sure to lock down the permissions by restricting access to your IP address or a CIDR you trust.

13. For Key Pair settings select a key pair that you’d like to use to SSH into the instance later. If you don’t have one then you can create one using the Create a key pair in EC2 link.

14. Click Launch.

15. Navigate to EC2 Console.

16. Once Status check is done initializing click Actions > Monitor and troubleshoot > Get system log. Note: If you don’t see anything in the system log wait 5–10 minutes and check again.

17. Scroll down to find your credentials.

Bitnami Documentation https://docs.bitnami.com/aws/faq/get-started/find-credentials/

18. Using the Public IPv4 DNS, confirm that you can sign in using the provided username and password.


***************************************************************************************************************************************************************************************
CONFIGURE DOCKER, AND TERRAFORM ON JENKINS
***************************************************************************************************************************************************************************************

1. Install docker on Jenkins machine. check this link as per the distro
2. Create and assign a Role to the Jenkins machine so that Jenkins has permission to create resources like:-EC2, S3, VPC, etc . In this blog, we can create EC2 but I will give admin access so that we can use the same to make other resources with this Jenkins machine.
To create an IAM role

The following create-role command creates a role named Test-Role and attaches a trust policy to it:

aws iam create-role — role-name role-example — assume-role-policy-document file://trust-Policy.json
Below is the file trust-Policy.json

{
 “Version”: “2012–10–17”,
 “Statement”: [
 {
 “Effect”: “Allow”,
 “Action”: “*”,
 “Resource”: “*”
 }
 ]
}

3. Assign this role to the Jenkins machine

Select the jenkins instance

Click on Action > Security > change

Click on update IAM role.


************************************************************************************************************************************************************************************
CONFIGURE JENKINS PIPELINE
************************************************************************************************************************************************************************************

1. Click New Item.

2. Select Pipeline.

3. From the Definition drop down select Pipeline script from SCM.

4. Enter the following:

SCM: Git
Repository URL: Your GitHub Repo with your Jenkinsfile
Branch: Your primary branch

Repository browser: Auto
Script Path: “Jenkinsfile”

5. Click Save.


************************************************************************************************************************************************************************************
CONFIGURE JENKINS PIPELINE
************************************************************************************************************************************************************************************

1. Click New Item.

2. Select Pipeline.

3. From the Definition drop down select Pipeline script from SCM.

4. Enter the following:

SCM: Git
Repository URL: Your GitHub Repo with your Jenkinsfile
Branch: Your primary branch

Repository browser: Auto
Script Path: “Jenkinsfile”

5. Click Save.


************************************************************************************************************************************************************************************
RUN JENKINS PIPELINE
************************************************************************************************************************************************************************************

Click on Job > Build with parameters

Insert the below parameters and select "plan" to see what will get created

Once everything looks good Rebuild

Type "apply" and click on the Rebuild button

Job Success!!!


**************************************************************************************************************************************************************************************
DESTROYING OUR INFRASTRUCTURE
**************************************************************************************************************************************************************************************

Same as above click on Rebuild

Type "destroy"

and rebuild

This will destroy all the Ec2 instances which were created by this Jenkins job


CONGRATULATION!!!