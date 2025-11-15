# Auto Scaling Group with Load Balancer

## Introduction

In this project, I creates a scalable and reliable web infrastructure on AWS. I used an Application Load Balancer to evenly distribute incoming website traffic and Auto Scaling Groups to automatically manage the number of active servers.

The three Auto Scaling Groups are set up with different scaling strategies:

Home: Maintains a constant, static number of servers.

Mobile: Scales based on a schedule (e.g., adding servers during peak business hours).

Laptop: Scales dynamically based on real-time CPU usage.

Each group has its own target group, which is registered with the central Load Balancer to handle the routing.

## Architecture Diagram

![new1](./images/new1.png)

## Features Implemented

-- Application Load Balancer

Acts like a traffic manager for your website.

Sends incoming visitors to different groups of servers to spread out the load.

Makes sure the website stays online even if one server has a problem.

-- Auto Scaling Groups
I set up three groups that automatically control how many servers are running:

Home (Static):  Always keeps the same number of servers on.

Mobile (Scheduled):  Adds more servers at specific busy times (like lunch hour) and removes them later.

Laptop (Dynamic):  Automatically adds servers when traffic is high and removes them when it's quiet.

-- Target Groups

Each of the three server groups above has its own "Target Group."

The Load Balancer knows about all these groups and can send visitors to the right one.

## Deployment Steps

### Step1 : Launch Template

Launch 3 Templates : Home , Mobile and laptop with diffrent user data script.

Go to Launch Template click on create template

Name your template

Write Description

![as1](./images/as1.png)

Select AMI from Quick Start (I choosed Amazon Linux) 
You can choose your own AMI if u have created one.

![as2](./images/as2.png)

Select Instance Type

Select Keypair

![as3](./images/as3.png)

Select Network Settings. here my application is simple so i only need port 80 and port 22. you can select existing security group too.

![as4](./images/as4.png)

add user datascript in advanced options. (you don't need to add this script if u hae selected your own AMI )

click on create template

![as5](./images/as5.png)

Similarly Create other templates for Mobile and Laptop

User datascript for mobile :

    #!/bin/bash   
    sudo yum update -y  
    sudo yum install -y httpd  
    sudo systemctl start httpd  
    sudo systemctl enable httpd

    sudo mkdir /var/www/html/mobile
    echo "<h1>This is the mobile page $(hostname -f)</h1>" >
    /var/www/html/mobile/index.html

User datascript for laptop :

    #!/bin/bash   
    sudo yum update -y  
    sudo yum install -y httpd  
    sudo systemctl start httpd  
    sudo systemctl enable httpd

    sudo mkdir /var/www/html/laptop
    echo "<h1>This is the laptop page $(hostname -f)</h1>" >
    /var/www/html/laptop/index.html

### Step2 : Creating AutoScalling Group (ASG)

Let's Start with creating ASG for mobile. It is a Sceduled ASG.

Go to Autoscalling Group and click on create autoscalling.

(You will see the 7 steps on lefthand side of the screen)

1. Choose Launch Template

    Give Name to autoscalling group.

    Select a Launch Template of mobile for mobile autoscalling group.

![as6](./images/as6.png)

click on next...go to next step..

![as7](./images/as7%20(2).png)

2. Choose instance launch option

    Select Availability Zone. (Atleast 2)

    go to next step..

![a51](./images/as51.png)


3. Integrate with other services

    We are going to integrate these ASG with Loadbalancer after creating all 3 ASG. So for now let's go to next step..

![as8](./images/as8.png)

4. Configure Group size and scalling

    Here for mobile we are giving : Desired = 3 , min = 2 , max = 7. (After creating mobile ASG we will make it sceduled)

    For Laptop (Dynamic) ASG select : Desired = 3 , min = 2 , max = 7.

    For Home (static) ASG select : Desired = 2 , min = 2 , max = 2.

![as9](./images/as9.png)

select target tracking scaling policy
( if your ASG is static you don't need policy so select No scaling policy )

select metric type - Here i choose CPU utiliztion.

select target value - I choose 50.

select instacnce warmup - i choose 300 sec i.e, 5 min.

 go to next step..

 

5. Add notification

![as10](./images/as10.png)

6. Add tags

![as11](./images/as11.png)

you dont have to do anything in step 5 and 6 so just click on next.

7. Review

    select create autoscaling group

![as12](./images/as12.png)

Similarly create other 2 ASG according to their type static and dynamic. Just need to change the group size as i mentioned.

![as13](./images/as13.png)

8. Now to make Mobile ASG Sceduled follow following steps :

    Click on Mobile ASG...(You will see the following interface)

![as14](./images/as14.png)

Scroll down to bottom..

Click on Create schedule action..

![as15](./images/as15.png)

Give name to your schedule.

Enter Desired , min and max values of instances.

Choose Cron option in recurrence and give specific time in side box ( sequence is - minute hour date month and day of week)

![as16](./images/as16.png)

Enter the ending time with date

click on Create

If you click on mobile ASG and scroll doen to Sceduled Action you will see the the Created scheduled action.

### Step3 : Creating Target Group For each Autoscalling Group

Go to Target Groups Click on Create Target Group

![as17](./images/as17.png)

Give a Name to your Target Group.

![as18](./images/as18.png)

Scroll down and go to health check section.

Give a path of your application in Health Check Path section.

[Note : i Give "/" as a path . Here "/ " represent the directory " /var/www/html/ " which is default directory of httpd]

Go to next step.

![as19](./images/as19.png)

Click on Create Target Group (don't need to change anything else)

![as20](./images/as20.png)

--Similarly Create Target Groups for Laptop and Mobile.

You only need to change the path in Health check section as given.

For Laptop :

![as21](./images/as21.png)

![as22](./images/as22.png)

For Mobile :

![as23](./images/as23.png)

![as24](./images/as24.png)

The 3 Target Groups are created.

![as25](./images/as25.png)

### Step4 : Connecting Autoscalling group to Target Goups
Select an Autoscalling Group .

Click on Action at top right corner.

Click on Edit

![as26](./images/as26.png)

Scroll down and you will see the Load balancing section.

Select a Target group for autoscalling group we selected.
(for laptop ASG select Lptop target group, for mobile ASG select Mobile Target group.)

![as27](./images/as27.png)

Enter Cooldown time and click on update.

Successfully connected the Autoscalling group and target groups.

Repeat process to connect each autoscalling group to its target group.

![as28](./images/as28.png)

### Step5 :  Creating Application Load Balancer and Connecting to Target Groups.

Go to Load Balancers and Click on Create Load Balancer

![as29](./images/as29.png)

Select Application Load Balancer. Click on Create

![as30](./images/as30.png)

Give name to the loadbalancer.

Select Internet facing option

![as31](./images/as31.png)

In Network Mapping section , Select Availability Zones atleast 2.

(choose AZ same as the AZ we selected while creating autoscalling group.)

![as32](./images/as32.png)

In Security Group section , select a existing security group.

(Select same security group we selected while creating autoscalling group)

![as33](./images/as33.png)

In Listners and Routing section

select forward to target group

Choose a default target group. (Here default target group is HomeTG)

no need to change port n protocol.

Click on Create Load Balance

![as34](./images/as34.png)

Successfully created Application loadbalancer.

![as35](./images/as35.png)

If you scroll down , you will see the HomeTG as defualt target group in Listeners and Rules section.

![as36](./images/as36.png)

To add other Target groups..

select HTTP80 i.e., HomeTG

Click on Manage Rules

Click on Add Rule

![as37](./images/as37.png)

Give name to rule.

In condition section , click on Add condition

![as38](./images/as38.png)

Give path for mobile application..



In Actions section

select forward to target group

select the target group i.e., MobileTG

Go to next step..

In Listners rule , give priority 1 for mobileTG

click on next

![as39](./images/as39.png)

Click on Add rule.

![as40](./images/as40.png)

Successfully connected Mobile Target Group to Loadbalancer

![as41](./images/as41.png)

Similarly connect Laptop target group to load balancer with priority 2.

![as42](./images/as42.png)

![as43](./images/as43.png)

![as44](./images/as44.png)

![as45](./images/as45.png)


Now if you check in instance dashboard you will see the Active instances got created automatically.

![as46](./images/as46.png)

To check if it is working properly

go to Load balacer.

click on the load balancer we created

scroll down and copy DNS name

search it through browser

![as47](./images/as47.png)

### Step6: Results

The ALB DNS successfully routed traffic to all three target groups.

Scaling actions were verified:

Home ASG maintained fixed instances.

Mobile ASG scaled according to schedule.

Laptop ASG scaled dynamically on CPU utilization.

Application remained available across multiple Availability Zones.

Home:

![as48](./images/as48.png)

Laptop:

![as49](./images/as49.png)

Mobile:

![as50](./images/as50.png)

## Technologies Used

Amazon EC2 – Virtual servers to host the application.

Application Load Balancer (ALB) – For distributing incoming traffic across target groups.

Launch Templates

Auto Scaling Groups (ASG) – To implement different scaling strategies (Static, Dynamic, Scheduled).

Target Groups – For routing traffic to the right set of instances.

Amazon Linux – Operating system used for EC2 instances.

Security Groups – For controlling inbound and outbound traffic.

## Conclusion

This project shows how to build a website on AWS that is reliable, can handle lots of visitors, and fixes itself automatically. I got hands-on experience by setting up three different ways to automatically add or remove servers:

Keeping a fixed number running.

Adding more at scheduled busy times.

Letting AWS add or remove them based on current traffic. 

All traffic entered through one web address, and the system automatically directed visitors to available servers. It successfully adjusted the number of servers and handled all the web requests.

This project gave me a strong, practical understanding of how to manage traffic and automate server scaling on AWS.