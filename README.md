# Bamboo Elastic Agent Security Groups

This small offering solves a problem I had with launching VPC-bound Bamboo EC2 elastic agents in a subnet with restricted access to the outside world; in my case being that the only available DNS server resides in another subnet within the VPC for which a specific security group was required to be present on instances to grant access to that DNS server. DNS is required to locate the downloads required to initialise the elastic agent on the new instance. You may find for instance that specific security groups are required in your setup to permit access to NAT gateways etc. The key thing is that the new instance has all the access it requires to access Atlassian resources on the Internet - DNS + egress rules to the outside world.

Bamboo, when it lauches instances only assigns security groups of its own creation which are suffice to Bamboo's needs and to grant access to the outside world (egress rules and subnet NACLs permitting).

What we have in this CloudFormation template is a CloudWatch event that triggers when a new instance is launched. This triggers the lambda which waits up to 10 seconds for instance tags to be present. When the tags appear, the `Name` tag is checked for a value that matches an elastic bamboo instance. If the instance is recognised as an elastic bamboo instance, the security group IDs passed as a stack parameter (and rendered into the Lambda's environment variables) are added to the instance's security groups. All this happens prior to the instance getting far enough into its startup sequence for the agent load to begin.

## How an Elastic Bamboo Instance is recognised

The lambda uses a regular expression to detect an elastic instance by EC2 tag `Name`. This regex is loosely based on observation that the value of the name tag is `bam::servername::servername` where servername is the name of the bamboo server and matches `[A-Z0-9\-]+`. This may or may not be strictly correct - corrections welcome!

# See Also

This repo provides the accepted solution to my own question on [Atlassian Commmunity Forums](https://community.atlassian.com/t5/Bamboo-questions/EC2-elastic-agent-security-groups/qaq-p/1148160)
