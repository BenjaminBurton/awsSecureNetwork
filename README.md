# Secure Network using encryption with AWS CLI

### To create a secure network that is encrypted on AWS using the CLI, you can use a combination of AWS CloudFormation and AWS Key Management Service (KMS). Here are the general steps:

1. Create an AWS KMS customer master key (CMK) for encrypting your network traffic. You can do this with the following command:

```yaml
aws kms create-key --description "My network encryption key"
```

2. Create an AWS CloudFormation stack that includes the following resources:
- A VPC with a public and private subnet
- An internet gateway and route table for the public subnet
- A NAT gateway and route table for the private subnet
- A network interface in each subnet
- A security group for each network interface

```yaml
Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16

  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      CidrBlock: 10.0.1.0/24
      VpcId: !Ref VPC

  PrivateSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      CidrBlock: 10.0.2.0/24
      VpcId: !Ref VPC

  InternetGateway:
    Type: AWS::EC2::InternetGateway

  InternetGatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      InternetGatewayId: !Ref InternetGateway
      VpcId: !Ref VPC

  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC

  PrivateRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC

  PublicRoute:
    Type: AWS::EC2::Route
    Properties:
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway
      RouteTableId: !Ref PublicRouteTable

  NATGateway:
    Type: AWS::EC2::NatGateway
    Properties:
      AllocationId: !GetAtt ElasticIP.AllocationId
      SubnetId: !Ref PublicSubnet

  ElasticIP:
    Type: AWS::EC2::EIP

  PrivateRoute:
    Type: AWS::EC2::Route
    Properties:
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NATGateway
      RouteTableId: !Ref PrivateRouteTable

  PublicNetworkInterface:
    Type: AWS::EC2::NetworkInterface
    Properties:
      SubnetId: !Ref PublicSubnet
      GroupSet:
        - !Ref PublicSecurityGroup

  PrivateNetworkInterface:
    Type: AWS::EC2::NetworkInterface
    Properties:
      SubnetId: !Ref PrivateSubnet
      GroupSet:
        - !Ref PrivateSecurityGroup

  PublicSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: "Allow SSH and HTTP access from anywhere"
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 
```
