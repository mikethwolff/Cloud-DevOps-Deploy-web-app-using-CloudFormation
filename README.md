# Cloud DevOps Engineer Projects Udacity

The listed projects were part of my Cloud DevOps Engineer Nanodegree and are displayed in derivated form due to copyright restrictions.

Form and requirements of my projects may differ from present projects. Udacity may vary projects and this repo is representing my findings at this point in time and may not be representative for future Cloud DevOps Engineer projects. If you are currently enrolled into Udacity's Cloud DevOps Engineer Nanodegree please keep in mind that my findings can help you as a guide but copying these findings will go against [Udacity Honor Code](https://udacity.zendesk.com/hc/en-us/articles/210667103-Udacity-Honor-Code).


## Project 2: Deploy a high-availability web app using CloudFormation (IAC)

In this project, you'll be faced with a real scenario. Creating this project will give you the hands-on experience you need to confidently talk about infrastructure as code.

There will be two parts to this project:

* Diagram: You'll first develop a [diagram](https://github.com/mikethwolff/Cloud-DevOps-Engineer-Projects-Udacity/tree/main/Deploy%20a%20high-availability%20web%20app%20using%20CloudFormation%20(IAC)) that you can present as part of your portfolio and as a visual aid to understand the CloudFormation script.
* [Script](https://github.com/mikethwolff/Cloud-DevOps-Engineer-Projects-Udacity/tree/main/Deploy%20a%20high-availability%20web%20app%20using%20CloudFormation%20(IAC)) (Template and Parameters): The second part is to interpret the instructions and create a matching CloudFormation script.

**Solution:** [Deploy a high-availability web app using CloudFormation](https://github.com/mikethwolff/Cloud-DevOps-Engineer-Projects-Udacity/tree/main/Deploy%20a%20high-availability%20web%20app%20using%20CloudFormation%20(IAC))

## Project 2 Solution:
## (Infrastructure as code)

**Scenario:**

Your company is creating an Instagram clone called Udagram. Developers pushed the latest version of their code in a zip file located in a public S3 Bucket.

You have been tasked with deploying the application, along with the necessary supporting software into its matching infrastructure.

This needs to be done in an automated fashion so that the infrastructure can be discarded as soon as the testing team finishes their tests and gathers their results.


**Server specs**


You'll need to create a Launch Configuration for your application servers in order to deploy four servers, two located in each of your private subnets. The launch configuration will be used by an auto-scaling group.
You'll need two vCPUs and at least 4GB of RAM. The Operating System to be used is Ubuntu 18. So, choose an Instance size and Machine Image (AMI) that best fits this spec.
Be sure to allocate at least 10GB of disk space so that you don't run into issues.

**Solution:**

![alt text](https://github.com/mikethwolff/Cloud-DevOps-Engineer-Projects-Udacity/blob/main/Deploy%20a%20high-availability%20web%20app%20using%20CloudFormation%20(IAC)/UdacityDevOpsProject.png)

network.yml
```
Description: 
            Udacity DevOps Project
            This template deploys NETWORK resources for the Udacity high-available-website-project
Parameters:
  EnvironmentName:
    Description: An environment name that will be prefixed to the resources
    Type: String

  VpcCIDR:
    Description: Please enter the IP range (CIDR notation) 
    Type: String
    Default: 10.0.0.0/16

  PublicSubnet1CIDR:
    Description: Please enter the IP range (CIDR notation) for the Network
    Type: String
    Default: 10.0.0.0/24

  PublicSubnet2CIDR:
    Description: Please enter the IP range (CIDR notation) for the Network
    Type: String
    Default: 10.0.1.0/24

  PrivateSubnet1CIDR:
    Description: Please enter the IP range (CIDR notation) for the Network
    Type: String
    Default: 10.0.2.0/24

  PrivateSubnet2CIDR:
    Description: Please enter the IP range (CIDR notation) for the Network
    Type: String
    Default: 10.0.3.0/24

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCIDR
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: !Ref EnvironmentName

  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: !Ref EnvironmentName

  InternetGatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      InternetGatewayId: !Ref InternetGateway
      VpcId: !Ref VPC

  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: !Select [0, !GetAZs ""]
      CidrBlock: !Ref PublicSubnet1CIDR
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Public Subnet (AZ1)

  PublicSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: !Select [1, !GetAZs ""]
      CidrBlock: !Ref PublicSubnet2CIDR
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Public Subnet (AZ2)

  PrivateSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: !Select [0, !GetAZs ""]
      CidrBlock: !Ref PrivateSubnet1CIDR
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Private Subnet (AZ1)

  PrivateSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: !Select [1, !GetAZs ""]
      CidrBlock: !Ref PrivateSubnet2CIDR
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Private Subnet (AZ2)

  NatGateway1EIP:
    Type: AWS::EC2::EIP
    DependsOn: InternetGatewayAttachment
    Properties:
      Domain: vpc

  NatGateway2EIP:
    Type: AWS::EC2::EIP
    DependsOn: InternetGatewayAttachment
    Properties:
      Domain: vpc

  NatGateway1:
    Type: AWS::EC2::NatGateway
    Properties:
      AllocationId: !GetAtt NatGateway1EIP.AllocationId
      SubnetId: !Ref PublicSubnet1

  NatGateway2:
    Type: AWS::EC2::NatGateway
    Properties:
      AllocationId: !GetAtt NatGateway2EIP.AllocationId
      SubnetId: !Ref PublicSubnet2

  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Public Routes

  DefaultPublicRoute:
    Type: AWS::EC2::Route
    DependsOn: InternetGatewayAttachment
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  PublicSubnet1RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PublicSubnet1

  PublicSubnet2RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PublicSubnet2

  PrivateRouteTable1:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Private Routes (AZ1)

  DefaultPrivateRoute1:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId: !Ref PrivateRouteTable1
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NatGateway1

  PrivateSubnet1RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PrivateRouteTable1
      SubnetId: !Ref PrivateSubnet1

  PrivateRouteTable2:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Private Routes (AZ2)

  DefaultPrivateRoute2:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId: !Ref PrivateRouteTable2
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NatGateway2

  PrivateSubnet2RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PrivateRouteTable2
      SubnetId: !Ref PrivateSubnet2

Outputs:
  VPC:
    Description: A reference to the created VPC
    Value: !Ref VPC
    Export:
      Name: !Sub ${EnvironmentName}-VPCID

  VPCPublicRouteTable:
    Description: Public Routing
    Value: !Ref PublicRouteTable
    Export:
      Name: !Sub ${EnvironmentName}-PUB-RT

  VPCPrivateRouteTable1:
    Description: Private Routing AZ1
    Value: !Ref PrivateRouteTable1
    Export:
      Name: !Sub ${EnvironmentName}-PRI1-RT

  VPCPrivateRouteTable2:
    Description: Private Routing AZ2
    Value: !Ref PrivateRouteTable2
    Export:
      Name: !Sub ${EnvironmentName}-PRI2-RT

  PublicSubnets:
    Description: A list of the public subnets
    Value: !Join [",", [!Ref PublicSubnet1, !Ref PublicSubnet2]]
    Export:
      Name: !Sub ${EnvironmentName}-PUB-NETS

  PrivateSubnets:
    Description: A list of the private subnets
    Value: !Join [",", [!Ref PrivateSubnet1, !Ref PrivateSubnet2]]
    Export:
      Name: !Sub ${EnvironmentName}-PRIV-NETS

  PublicSubnet1:
    Description: A reference to the public subnet in the 1st Availability Zone
    Value: !Ref PublicSubnet1
    Export:
      Name: !Sub ${EnvironmentName}-PUB1-SN

  PublicSubnet2:
    Description: A reference to the public subnet in the 2nd Availability Zone
    Value: !Ref PublicSubnet2
    Export:
      Name: !Sub ${EnvironmentName}-PUB2-SN

  PrivateSubnet1:
    Description: A reference to the private subnet in the 1st Availability Zone
    Value: !Ref PrivateSubnet1
    Export:
      Name: !Sub ${EnvironmentName}-PRI1-SN

  PrivateSubnet2:
    Description: A reference to the private subnet in the 2nd Availability Zone
    Value: !Ref PrivateSubnet2
    Export:
      Name: !Sub ${EnvironmentName}-PRI2-SN
```

network-parameters.json
```
[
  {
    "ParameterKey": "EnvironmentName",
    "ParameterValue": "UdacityDevOpsProject"
  },
  {
    "ParameterKey": "VpcCIDR",
    "ParameterValue": "10.0.0.0/16"
  },
  {
    "ParameterKey": "PublicSubnet1CIDR",
    "ParameterValue": "10.0.0.0/24"
  },
  {
    "ParameterKey": "PublicSubnet2CIDR",
    "ParameterValue": "10.0.1.0/24"
  },
  {
    "ParameterKey": "PrivateSubnet1CIDR",
    "ParameterValue": "10.0.2.0/24"
  },
  {
    "ParameterKey": "PrivateSubnet2CIDR",
    "ParameterValue": "10.0.3.0/24"
  }
]

```

servers.yml
```
Description: 
            Udacity DevOps Project
            This template deploys SERVER resources for the Udacity high-available-website-project
Parameters:
  EnvironmentName:
    Description: An Environment name that will be prefixed to resources
    Type: String
  MinAutoScalingSize:
    Description: The minimum size for the auto scaling group
    Type: String
  MaxAutoScalingSize:
    Description: The maximum size for the auto scaling group
    Type: String

Resources:

#-- allowing EC2 access to S3
  UdacityS3ReadOnlyEC2:
      Type: AWS::IAM::Role
      Properties:
          RoleName: 
              !Sub ${EnvironmentName}-Role
          AssumeRolePolicyDocument:
              Version: "2012-10-17"
              Statement:
              -   Effect: Allow
                  Principal:
                      Service:
                      - ec2.amazonaws.com
                  Action:
                  - sts:AssumeRole
          Path: "/"

  RolePolicies:
      Type: AWS::IAM::Policy
      Properties:
          PolicyName: AmazonS3ReadOnlyAccess
          PolicyDocument:
              Version: '2012-10-17'
              Statement:
              - 
                  Effect: Allow
                  Action: 
                  -   s3:Get*
                  -   s3:List*
                  Resource: 
                  -   arn:aws:s3:::udacitymwdevopsbucket
                  -   arn:aws:s3:::udacitymwdevopsbucket/*
          Roles:
          -   Ref: UdacityS3ReadOnlyEC2

  ProfileWithRolesForOurApp:
      Type: AWS::IAM::InstanceProfile
      Properties:
          Path: "/"
          Roles:
          - Ref: UdacityS3ReadOnlyEC2

#-- Security groups
  LoadBalancerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow http to the loadbalancer
      VpcId:
        Fn::ImportValue: !Sub "${EnvironmentName}-VPCID"
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
      SecurityGroupEgress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow http to our webApp Hosts and ssh only from local
      VpcId:
        Fn::ImportValue: !Sub "${EnvironmentName}-VPCID"
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
      SecurityGroupEgress:
        - IpProtocol: tcp
          FromPort: 0
          ToPort: 65535
          CidrIp: 0.0.0.0/0

#-- Launch configuration & auto scaling 
  WebAppLaunchConfig:
    Type: AWS::AutoScaling::LaunchConfiguration
    Properties:
      UserData:
        Fn::Base64: !Sub |
            #!/bin/bash
            apt-get update -y
            apt-get install unzip awscli -y
            apt-get install apache2 -y
            systemctl start apache2.service
            cd /var/www/html
            aws s3 cp s3://udacitymwdevopsbucket/udacity.zip .
            unzip -o udacity.zip
      ImageId: ami-0ac73f33a1888c64a
      #-ImageId: ami-03d5c68bab01f3496
      KeyName: DevOpsKey
      SecurityGroups:
        - Ref: WebServerSecurityGroup
      IamInstanceProfile: !Ref ProfileWithRolesForOurApp
      InstanceType: t3.medium
      BlockDeviceMappings:
        - DeviceName: "/dev/sdk"
          Ebs:
            VolumeSize: '10'
  WebAppGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      VPCZoneIdentifier:
        - Fn::ImportValue: !Sub "${EnvironmentName}-PRIV-NETS"
      MinSize: !Ref MinAutoScalingSize
      MaxSize: !Ref MaxAutoScalingSize
      LaunchConfigurationName:
        Ref: WebAppLaunchConfig
      TargetGroupARNs:
        - Ref: WebAppTargetGroup

#-- Load balancer 
  WebApploadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Subnets:
        - Fn::ImportValue: !Sub "${EnvironmentName}-PUB1-SN"
        - Fn::ImportValue: !Sub "${EnvironmentName}-PUB2-SN"
      SecurityGroups:
        - Ref: LoadBalancerSecurityGroup
  Listener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      DefaultActions:
        - Type: forward
          TargetGroupArn:
            Ref: WebAppTargetGroup
      LoadBalancerArn:
        Ref: WebApploadBalancer
      Port: 80
      Protocol: HTTP
  ALBListenerRule:
    Type: AWS::ElasticLoadBalancingV2::ListenerRule
    Properties:
      Actions:
        - Type: forward
          TargetGroupArn:
            Ref: WebAppTargetGroup
      Conditions:
        - Field: path-pattern
          Values: [/]
      ListenerArn:
        Ref: Listener
      Priority: 1

#-- Target group
  WebAppTargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      HealthCheckIntervalSeconds: 10
      HealthCheckPath: /
      HealthCheckProtocol: HTTP
      HealthCheckTimeoutSeconds: 8
      HealthyThresholdCount: 2
      Port: 80
      Protocol: HTTP
      UnhealthyThresholdCount: 5
      VpcId:
        Fn::ImportValue:
          Fn::Sub: "${EnvironmentName}-VPCID"

#-- Link
Outputs:
  LoadBanlancerEndpoint:
    Description: this is the endpoint to use for accessing the loadbanlancer
    Value: !Join ["", ["http://", !GetAtt WebApploadBalancer.DNSName]]
    Export:
      #Name: !Sub ${EnvironmentName}-LBURL
      Name: !Sub ${EnvironmentName}-DNS-NAME

```


servers-parameters.json
```
[
  {
    "ParameterKey": "EnvironmentName",
    "ParameterValue": "UdacityDevOpsProject"
  },
  {
    "ParameterKey": "MinAutoScalingSize",
    "ParameterValue": "2"
  },
  {
    "ParameterKey": "MaxAutoScalingSize",
    "ParameterValue": "4"
  }
]
```

create.sh
```
aws cloudformation create-stack \
    --stack-name $1 \
    --template-body file://$2 \
    --parameters file://$3 \
    --capabilities "CAPABILITY_NAMED_IAM" \
    --region=us-west-2
```

update.sh
```
aws cloudformation update-stack \
    --stack-name $1 \
    --template-body file://$2 \
    --parameters file://$3 \
    --capabilities "CAPABILITY_NAMED_IAM" \
    --region=us-west-2
```


Run commands:
```
./create.sh [stackName1] network.yml network-parameters.json
./create.sh [stackName2] servers.yml servers-parameters.json
```
Update commands:
```
./update.sh [stackName1] network.yml network-parameters.json
./update.sh [stackName2] servers.yml servers-parameters.json
```

## Project 1: Deploy Static Website on AWS

For this project 

Topics Covered:
* S3 bucket creation
* S3 bucket configuration
* Website distribution via CloudFront
* Access website via web browser

**Read on:** [Deploy Static Website on AWS](https://github.com/mikethwolff/Cloud-DevOps-Engineer-Projects-Udacity/tree/main/Deploy%20Static%20Website%20on%20AWS)


