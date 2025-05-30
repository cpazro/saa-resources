## S3 + CloudFormation
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Parameters:
  BucketName:
    Type: String
    Description: Name for the S3 bucket
    Default: yourname-s3-bucket

Resources:
  MyS3LoggingBucket:
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: !Sub '${BucketName}-access-logs'
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true

  MyS3Bucket:
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: !Ref BucketName
      VersioningConfiguration:
        Status: Enabled
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
      LoggingConfiguration:
        DestinationBucketName: !Ref MyS3LoggingBucket
        LogFilePrefix: access-logs/
      PublicAccessBlockConfiguration:
        BlockPublicAcls: false
        BlockPublicPolicy: false
        IgnorePublicAcls: false
        RestrictPublicBuckets: false

  MyS3BucketPolicy:
    Type: 'AWS::S3::BucketPolicy'
    Properties:
      Bucket: !Ref MyS3Bucket
      PolicyDocument:
        Statement:
          - Sid: PublicReadGetObject
            Effect: Allow
            Principal: '*'
            Action: 's3:GetObject'
            Resource: !Sub 'arn:aws:s3:::${MyS3Bucket}/*'

Outputs:
  BucketName:
    Description: Name of the created S3 bucket
    Value: !Ref MyS3Bucket
    Export:
      Name: !Sub '${AWS::StackName}-BucketName'

  BucketArn:
    Description: ARN of the created S3 bucket
    Value: !GetAtt MyS3Bucket.Arn
    Export:
      Name: !Sub '${AWS::StackName}-BucketArn'

  BucketDomainName:
    Description: Domain name of the S3 bucket
    Value: !GetAtt MyS3Bucket.DomainName

  BucketWebsiteURL:
    Description: Website URL of the S3 bucket
    Value: !GetAtt MyS3Bucket.WebsiteURL

  LoggingBucketName:
    Description: Name of the access logging bucket
    Value: !Ref MyS3LoggingBucket
    Export:
      Name: !Sub '${AWS::StackName}-LoggingBucketName'

  LoggingBucketArn:
    Description: ARN of the access logging bucket
    Value: !GetAtt MyS3LoggingBucket.Arn
```
## EFS
`aws ec2 create-security-group --group-name StorageLabs --description "SG for EFS storage"`

`aws ec2 authorize-security-group-ingress --group-name StorageLabs --protocol tcp --port 22 --cidr 0.0.0.0/0`

`aws ec2 run-instances --image-id ami-0440d3b780d96b29d --instance-type t2.micro --placement AvailabilityZone=us-east-1a --security-group-ids INSTANCE-SG`

`aws ec2 run-instances --image-id ami-0440d3b780d96b29d --instance-type t2.micro --placement AvailabilityZone=us-east-1b --security-group-ids INSTANCE-SG`

`aws ec2 authorize-security-group-ingress --group-id sg-01f9143b61c4ed7d5 --protocol tcp --port 2049 --source-group INSTANCE-SG`

### in instances (BOTH):

`sudo yum -y install nfs-utils`

`mkdir ~/efs-mount-point`

`sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport [FILESYSTEM ID].efs.us-east-1.amazonaws.com:/ ~/efs-mount-point`

## DynamoDB
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/getting-started-step-1.html

## Athena
