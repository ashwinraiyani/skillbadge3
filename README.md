# skillbadge3

# Task 1: Create development VPC manually
Make sure you create all resources in the us-east1 region and us-east1-b zone.

In the Google Cloud Console, navigate to VPC network > VPC networks <br/>
   Click on Create VPC network. <br/>
   Enter griffin-dev-vpc to the Name field. <br/>
   Select Custom for the Subnet creation mode. <br/>
   Add griffin-dev-wp subnet with the following parameters: <br/>

   Name:	griffin-dev-wp  <br/>
   Region:	us-east1  <br/>
   IP address range:	192.168.16.0/20  <br/>

Click + Add subnet and add griffin-dev-mgmt subnet with the following parameters


Name:	griffin-dev-mgmt  <br/>
Region:	us-east1 <br/>
IP address range:	192.168.32.0/20  <br/>
Click Create.


# Task 2: Create production VPC using Deployment Manager

Copy the Deployment Manager configuration files to Cloud Shell using the following command:  <br/>

*gsutil cp -r gs://cloud-training/gsp321/dm ~/ * <br/>

cd dm  <br/>

edit prod-network.yaml  <br/>

**Replace SET_REGION to *us-east1* in the editor, and then save the change.**  <br/>

Go back to the Cloud Shell, use the following command to create the production VPC network with the configuration files:  <br/>

*gcloud deployment-manager deployments create griffin-prod --config prod-network.yaml *  <br/>

# Task 3: Create bastion host  <br/>

In the Cloud Console, navigate to Compute Engine > VM instances. 
Click Create VM Instance <br/>
Use the following parameters to create the bastion host: <br/>

Name:	bastion  <br/>
Region:	us-east1  <br/>

Expand the Management, security, disks, networking, sole tenancy section.

In the Networking tab, add bastion to the Network tags.  <br/>
Click Add network interface, make sure that you set up two Network interfaces  <br/>

**griffin-dev-mgmt  <br/>
griffin-prod-mgmt**   <br/>
![screen](https://github.com/ashwinraiyani/skillbadge3/blob/main/Screenshot%20(170).png)
Click **Create**

Now Navigate to VPC network > Firewall.  <br/>
Click CREATE FIREWALL RULE.  <br/>
Configure the rule with the following parameters: <br/>


*Name:	allow-bastion-dev-ssh  <br/>
Network:	griffin-dev-vpc  <br/>
Targets:	bastion  <br/>
Source IP ranges:	192.168.32.0/20  <br/>
Protocols and ports:	tcp: 22  <br/>
Click CREATE.*

![screen](https://github.com/ashwinraiyani/skillbadge3/blob/main/Screenshot%20(171).png)
![screen](https://github.com/ashwinraiyani/skillbadge3/blob/main/Screenshot%20(172).png)
Click CREATE FIREWALL RULE again.
Configure another rule with the following parameters:  <br/>

*Name:	allow-bastion-prod-ssh  <br/>
Network:	griffin-prod-vpc  <br/>
Targets:	bastion <br/>
Source IP ranges:	192.168.48.0/20 <br/>
Protocols and ports:	tcp: 22 <br/>
Click CREATE.*


# Task 4: Create and configure Cloud SQL Instance

In the Cloud Console, navigate to SQL.
Click CREATE INSTANCE.
Click Choose MySQL.
Use the following parameters to create the instance:

Field	Value
Name:	griffin-dev-db
Region:	us-east1
Zone:	us-east1-b
Root password:	e.g. 12345678
Note: In real practice, you must set a strong password.

Click Create.
**Wait for few minutes to create SQL instance it turned into green**
After green mark only Click the griffin-dev-db in the SQL pane after it has created.

Under Connect to this instance, click on Connect using Cloud Shell.
![screen](https://github.com/ashwinraiyani/skillbadge3/blob/main/Screenshot%20(173).png)
Go back to the Cloud Shell, run:

*gcloud sql connect griffin-dev-db --user=root --quiet*

Enter the Root password generated in Step 4.

In the SQL console, run the following query to create the wordpress database:
   CREATE DATABASE wordpress;  <br/>
   GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%" IDENTIFIED BY "stormwind_rules";  <br/>
   FLUSH PRIVILEGES;  <br/>
   
Enter exit to quit the SQL shell.

*Check the progress*

# Task 5: Create Kubernetes cluster

In the Cloud Console, navigate to Kubernetes Engine > Clusters.
Click Create cluster.
In the Cluster basics tab, configure:

**Name: griffin-dev <br/>
Zone: us-east1-b**

In the left pane, click default-pool under NODE POOLS and set

Number of nodes: 2

Click Nodes Under default-pool, and set

Series : N1
Machine type: n1-standard-4

Go to the Network tab, set

Network: griffin-dev-vpc
Node subnet: griffin-dev-wp

Click CREATE.

*check the progress*

# Task 6: Prepare the Kubernetes cluster
In the Cloud Shell, use the following command to copy the files for the Kubernetes:

*gsutil cp -r gs://cloud-training/gsp321/wp-k8s ~/ *
Open wp-k8s/wp-env.yaml with the Cloud Shell Editor.

cd ~/wp-k8s
edit wp-env.yaml

Replace username_goes_here and password_goes_here to wp-user and stormwind_rules, respectively.
![screen](https://github.com/ashwinraiyani/skillbadge3/blob/main/Screenshot%20(173).png)

Save the file change.
Run the following command to connect the cluster:
After the Kubernetes cluster has created, click on the Connect button.
![screen](https://github.com/ashwinraiyani/skillbadge3/blob/main/Screenshot%20(173).png)

*gcloud container clusters get-credentials griffin-dev --zone=us-east1-b*

Deploy the configuration to the cluster using:

*kubectl apply -f wp-env.yaml*

Use the command below to create the key, and then add the key to the Kubernetes environment:

   *gcloud iam service-accounts keys create key.json \
       --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
   kubectl create secret generic cloudsql-instance-credentials \
       --from-file key.json*
       
       
# Task 7: Create a WordPress deployment

Open wp-k8s/wp-deployment.yaml with the Cloud Shell Editor

*cd ~/wp-k8s*
*edit wp-deployment.yaml*

Replace **YOUR_SQL_INSTANCE** with **griffin-dev-db’s** Instance connection name.
to get Navigation Menu > SQL > click instance Scroll down .. Under *Connect to this instance* section you will find Instance connection name (it look like qwiklabs-gcp-00-0501993ef24d:us-east1:griffin-dev-db) 

![screen](https://github.com/ashwinraiyani/skillbadge3/blob/main/Screenshot%20(185).png)

Save the file change.

Go back to the Cloud Shell, run the following commands:

*kubectl create -f wp-deployment.yaml*
*kubectl create -f wp-service.yaml*

Now go to menu > Kubernetes Engine > Services and Ingress 

Find : External Endpoints and click on IP address. It will redirect to new page and Now click on The page you were on is trying to send you to * <<IP>>*.
  **Note: Note down this IP Address. It will use in Task 8**
  ![screen](https://github.com/ashwinraiyani/skillbadge3/blob/main/Screenshot%20(186).png)
  
  Wordpress Admin page open up. Enter the information like - User-name (wp_user), password(stormwind_rules) and email (qwiklabs username1) <br/>
  ![screen](https://github.com/ashwinraiyani/skillbadge3/blob/main/WrdPrss.png)
  
  Click Install Wordpress.
  
*check the progress*

# Task 8: Enable monitoring
Go back to the Cloud Console, and navigate to Monitoring.
In the Monitoring console, click Uptime checks in the left pane.
Click CREATE UPTIME CHECK.
Configure using the following parameters:


Title:	WordPress Uptime
Check Type:	HTTP
Resource Type: 	URL
Hostname	: IP address only from Task 7
Path:	/

![screen](https://github.com/ashwinraiyani/skillbadge3/blob/main/qwiklab-gsp321-uptime-check.jpg)
Click Test 
Click Save


Task 9: Provide access for an additional engineer
In the Cloud Console, navigate to IAM & Admin > IAM.
Click +ADD.
In the Add members to … pane, copy and paste the second user account for the lab to the New members field.
In the Role dropdown, select Project > Editor.
  ![screen](https://github.com/ashwinraiyani/skillbadge3/blob/main/Screenshot%20(183).png)
Click SAVE.
  
