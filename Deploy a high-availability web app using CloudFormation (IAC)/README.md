Scenario

Your company is creating an Instagram clone called Udagram. Developers pushed the latest version of their code in a zip file located in a public S3 Bucket.

You have been tasked with deploying the application, along with the necessary supporting software into its matching infrastructure.

This needs to be done in an automated fashion so that the infrastructure can be discarded as soon as the testing team finishes their tests and gathers their results.

Solution:

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
  WebAppLaunchConfig:
    Type: AWS::AutoScaling::LaunchConfiguration
    Properties:
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          apt-get update
          apt-get install apache2 -y
          mv /var/www/html/index.html /var/www/html/index_old.html
          wget -P /var/www/html https://udacity-cloudformation-bucket.s3-us-west-2.amazonaws.com/index.html
          systemctl start apache2.service
      ImageId: ami-005bdb005fb00e791
      SecurityGroups:
        - Ref: WebServerSecurityGroup
      InstanceType: t3.small
      BlockDeviceMappings:
        - DeviceName: "/dev/sdk"
          Ebs:
            VolumeSize: "10"
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
      Port: "80"
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
  WebAppTargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      HealthCheckIntervalSeconds: 5
      HealthCheckPath: /
      HealthCheckProtocol: HTTP
      HealthCheckTimeoutSeconds: 4
      HealthyThresholdCount: 3
      Port: 80
      Protocol: HTTP
      UnhealthyThresholdCount: 3
      VpcId:
        Fn::ImportValue:
          Fn::Sub: "${EnvironmentName}-VPCID"
Outputs:
  LoadBanlancerEndpoint:
    Description: this is the endpoint to use for accessing the loadbanlancer
    Value: !Join ["", ["http://", !GetAtt WebApploadBalancer.DNSName]]
    Export:
      Name: !Sub ${EnvironmentName}-LBURL
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
    --capabilities "CAPABILITY_IAM" \
    --region=us-west-2
```

update.sh
```
aws cloudformation update-stack \
    --stack-name $1 \
    --template-body file://$2 \  
    --parameters file://$3 \
    --capabilities "CAPABILITY_IAM" \ 
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



