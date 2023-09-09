---
title: "AWS: How to schedule launch and termination of an EC2 instance"
description: "A tutorial that shows how to automate the launch and termination of an AWS EC2 instance"
pubDate: "Sep 07 2021"
heroImage: "/hero-image-aws-how-to-schedule-launch-and-termination-of-an-ec2-instance.webp"
---

## A common scenario

After deploying application to an dev/testing server hosted in EC2, you may find that the developers only need access to this servers during their work hours.

So having your instances active during hours that you know the users don't really need access to that servers may represent an unnecessary waste of money.

## The solution: Schedule actions on auto-scaling groups.

The solution I'm going show you in this post uses the **schedule actions** of the auto-scaling groups.

You will see that with this approach you will be able to adjust the _Desired Capacity_, _Minimum Capacity_ and _Maximum Capacity_ of the group of instances you're working with at certain hours of the day.

For this example the _Desired Capacity_ will be set to 1 instances during work hours, and for inactive hours it will be again adjusted but now to 0 instances.

You can see in the following image the architecture of the solution:
![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/s83dbbhgz1snwzs57vum.png)

To create this solution you need to follow the next steps.

1. Create an AMI of the instance
2. Create a launch template using the AMI created
3. Add your launch template to a load balancer
4. Add your launch template to an auto-scaling group
5. Add an schedule action to scale-up and another to scale-in

## Full explanation step by step.

I will explain how to create this solution using the AWS web console. In a later tutorial I will explain how to create this same solution bit using an IaC tool like Terraform.

For this example I created a t2.micro instance and installed an apache web server on it. In the following image you can see the web server is running on port 80.

![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8sb6a2uj9xg93i34mh2l.png)

This instance executes the web server on each startup. This come will come in handy since we will be creating and terminating new instances every day.

### Step 1: Create an _AMI_ of the instance

Now using AWS web console, go to the instance of the EC2 service. Right click on the instance you want to work with and choose _Create image_.

![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/t45fqftrm0befi8byl6f.png)

In the next page give a name to your image and click on _Create image_.
![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wfva6qipq1y814wfxqoq.png)

Now if you navigate to the AMI section you will see a new image is created and is on pending status.

![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8gn7mbfdywu5m22nywzr.png)

### Step 2: Create a Launch Template using the AMI created

Navigate to the _Launch Template_ section and click on _Create Launch Template_.

![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8ntjw7fkgum1yg5ow86g.png)

In the Amazon machine image section search for the name of the image and select it. Also select the Latest version of the image.
![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/l8gwgerqo2o2h164hqlx.png)

Ensure to choose the same security group that your existing instance has.

![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/71q3yhnj7mm0jjik2nit.png)

Click on Create launch template

### Step 3: Add your launch template to a load balancer

Select Application Load Balancer
![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/iyrka9rppswsn7vsqifa.png)

Give a name to the load balancer
![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/azfbvihwfh3oa4beml5n.png)

Select at least two availability zones
![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4ymrlw0qma480rhsrzcf.png)

Use the default listener and click on create target group
![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yp44bbvtvr1l0tdc4ryw.png)

It will open a new browser tab to create the target group. Give a name and choose Instances for the instance type.
![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/oogy96zk0jt5rw8kmm7r.png)

Leave all the other options as default, click next and click on Create target group. Return to the tab of the load balancer creation.

Search for the target group in the default listener and select it.
![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ff1v1hpka4gzfw4bxh5x.png)

Click on Create load balancer.

### Step 4: Add your launch template to an auto-scaling group

Navigate to the Auto-scaling group section and click on Create auto-scaling group.

Give a name to the Auto-scaling group and choose the launch template we have created. Click on next.
![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bsc88ycukmqtathb3pcr.png)

Select the two Availability zones we choose on the creation of the load balancer. Click on next.
![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/65bqemawy56zw8osx04o.png)

In the next section choose Attach to an existing load balancer and select the load balancer we created in the previous step. Click on next.
![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/cxevl0wnpxwl704340e3.png)

For the group size I will set the values as showed in the image. Click on next.
![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8weot6p4zzn55b6bqhpw.png)

For Add notification and Add tags just click next. To finish, click Create auto scaling group.

### Step 5: Add an schedule action to scale-up and another to scale-in

Navigate to the Load balancers section, select the recently created load balancers from the list, and click on the Automatic scaling tab.

Scroll down until you see schedule actions. Click on Create scheduled action
![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/f3wvohtl4v2o35zlj4h1.png)

Give a name, set desired capacity to 1, for recurrence choose Cron, enter this cron expression `0 8 * * MON-FRI`, choose your timezone and select the start time of your action.

![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gpxr88r84u8qivf4nuvy.png)

This configuration will set the desired capacity to 1 from monday to friday at 8 am.

Now to scale-in create another schedule action but now setting the desired capacity to 0.

![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/chalmrs5ucgi6tq9hbl6.png)

### Terminate the existing instance

As this solution uses an image to launch new instances you can terminate the existing one and wait until the scale-out action get executed to see a new instance.

### Accessing the application

To access the application use the DNS name of the application load balancer.

![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qxkywkzz1az5lkcsr1w3.png)

![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dtnhymwuonbqp0p4b7w9.png)
