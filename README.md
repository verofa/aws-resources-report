# AWS Resources Reports Generation
AWS cli, jq and some extras Unix-likes utilities to generate csv reports files about existing resources in AWS Cloud

PREREQUISITE

- [AWS Cli](https://docs.aws.amazon.com/cli/latest/userguide/install-linux-al2017.html) 
- [JQ](https://stedolan.github.io/jq/download/)
- [Sed, a stream editor](https://www.gnu.org/software/sed/manual/sed.html)

- VCPs
To get the main information about existing VPCs

~~~~
#1 Generate VPCs JSON file
aws ec2 describe-vpcs | jq --raw-output '.Vpcs[]' > vpcs.json

#2 Get the data definition of the VPC
cat vpcs.json|jq -r '.VpcId +";"+ .CidrBlock +";"+ .State +";"+ .DhcpOptionsId +";"+ (.IsDefault|tostring) +";"+ ([.Tags[]?|select(.["Key"] == "Name")|.Value]|join(";"))' > vpcs_report.csv

#3 Add headers to the file report 
sed -i '1 i\VpcId;Cidr;State;DhcpId;IsDefault;VpcName' vpcs_report.csv
~~~~

After having executing the above commands you will get a csv file with the VPC information. Should be something like this:

~~~~
VpcId;Cidr;State;DhcpId;IsDefault;VpcName
vpc-abcde0ebc3ff0b56jj;10.222.16.0/20;available;dopt-080098555553eb2d5;false;prod_VPC
vpc-04416267;172.31.0.0/16;available;dopt-0855555;true;
vpc-0jklm2995493c1141;10.233.32.0/20;available;dopt-0333019555555ddbf;false;preprod_VPC
vpc-00241a8f8b6955555;10.288.16.0/20;available;dopt-0e578e7faf555550e;false;dev_VPC
~~~~

Then you can just copy and paste in the popular spreadsheet software and play with the data in the way that you like:

Generating the report in a spreadsheet software: ![alt text](/CreatingVPCReport.gif)

- Subnets

~~~~
#1 Generate Subnets JSON file
aws ec2 describe-subnets | jq --raw-output '.Subnets[]' > subnets.json

#2 Get the data definition of the Subnets
cat subnets.json | jq -r '([.Tags[]?|select(.["Key"] == "Name")|.Value]|join(";")) +";"+ .SubnetId +";"+ .CidrBlock +";"+ (.AvailableIpAddressCount|tostring) +";"+ .AvailabilityZone +";"+ .VpcId' > subnets_report.csv

#3 Add headers to the file report 
sed -i '1 i\SubnetName;SubnetId;CidrBlock;IpAddCount;AvailabilityZone;VpcId' subnets_report.csv
~~~~

- Routes Tables

~~~~
#1 Generate RouteTables JSON file
aws ec2 describe-route-tables| jq --raw-output '.RouteTables[]' > route_tables.json

#2 Get the data definition of the RouteTables
cat RouteTables.json | jq -r '([.Tags[]?|select(.["Key"] == "Name")|.Value]|join(";")) +";"+ (.Associations[]?|.SubnetId) +";"+ (.Associations[]?|.Main|tostring) +";"+ .RouteTableId +";"+ .VpcId +";"+ (.PropagatingVgws[]?|.GatewayId)' > route_tables.csv

# Add headers to the file report 
sed -i '1 i\RTName;SubnetId;MainRT;RouteTableId;VpcId;GatewayId' route_tables.csv
~~~~
