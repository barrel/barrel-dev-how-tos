### Barrel Development How-Tos

AWS Tutorials
------------------
- [Security Groups](#securitygroups)

### Security Groups
The steps below outlines a basic workflow to manage AWS security groups

#### Editing Security Groups. 

For this example we'll be adding SSH access for a new IP to an existing security group

1. Log into the AWS console
2. Click the EC2 link in the list of services
3. Click on "Security Groups" in the sidebar
4. Locate the "barrel launch wizard" security group (this security group will be used for Barrel access and basic http access)
5. Click on the "Inbound" tab and click "Edit"
6. Click "Add Rule"
7. Select "SSH" from the dropdown (the port range should automatically be updated to 22, update it manually if not)
8. Add new IP with “/32” at the end, so "108.176.17.234/32" for example. "/32" is the CIDR netmask, but no need to worry about what that is for the purposes of this tutorial.
9. Click save
