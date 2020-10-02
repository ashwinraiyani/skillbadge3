# skillbadge3

Task 1: Create development VPC manually
Make sure you create all resources in the us-east1 region and us-east1-b zone.

In the Google Cloud Console, navigate to VPC network > VPC networks
Click on Create VPC network.
Enter griffin-dev-vpc to the Name field.
Select Custom for the Subnet creation mode.
Add griffin-dev-wp subnet with the following parameters:

Field	Value
Name:	griffin-dev-wp
Region:	us-east1
IP address range:	192.168.16.0/20
Click + Add subnet and add griffin-dev-mgmt subnet with the following parameters

Field	Value
Name:	griffin-dev-mgmt
Region:	us-east1
IP address range:	192.168.32.0/20
Click Create.
