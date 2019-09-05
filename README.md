# ECS 101

### CREATE CLUSTER
``` sh
aws ecs create-cluster --cluster-name markgECS --region ap-southeast-1
```

### GET SECURITY GROUP WITH TAG GROUP=ECS
``` sh
SG_ID=$(aws ec2 describe-security-groups --region ap-southeast-1 --filters "Name=tag:group,Values=ecs" --query "SecurityGroups[*].GroupId" --output text)
```

### GET AMAZON AMI ID FOR ECS
``` sh
IMAGE_ID=$(aws ec2 describe-images --owners amazon --region ap-southeast-1 --filter 'Name=name,Values=amzn2-ami-ecs-hvm-2.0.20190815-x86_64-ebs' --query "Images[0].ImageId" --output text)
```

### LAUNCH INSTANCE
``` sh
aws ec2 run-instances --image-id $IMAGE_ID --count 1 --instance-type t2.micro --iam-instance-profile Name=ec2ECSrole --key-name markdxc --security-group-ids $SG_ID --user-data file://setup-ecs-config.txt --region ap-southeast-1 --tag-specifications 'ResourceType=instance,Tags=[{Key=group,Value=ecs}]' 'ResourceType=volume,Tags=[{Key=group,Value=ecs}]'
```

### GET INSTANCE ID
``` sh
INSTANCE_1=$(aws ec2 describe-instances --region ap-southeast-1 --filter "Name=tag:group,Values=ecs" --query "Reservations[*].Instances[0].InstanceId" --output text)
```

### GET CONTAINER INSTANCE ARN
``` sh
INSTANCE_ARN_1=$(aws ecs list-container-instances --cluster markgECS --region ap-southeast-1 --query "containerInstanceArns[0]" --output text)
```

### DESCRIBE CONTAINER INSTANCES
``` sh
aws ecs describe-container-instances --cluster markgECS --container-instances $INSTANCE_ARN_1 --region ap-southeast-1
```

### REGISTER TASK DEFINITION
``` sh
aws ecs register-task-definition --cli-input-json file://web-task-definition.json --region ap-southeast-1
```

### CREATE SERVICE
``` sh
aws ecs create-service --cluster markgECS --service-name web --task-definition web --desired-count 1 --region ap-southeast-1
```

### CHECK YOUR SITE IF ITS WORKING
``` sh
DNS=$(aws ec2 describe-instances --region ap-southeast-1 --filter "Name=tag:group,Values=ecs" --query "Reservations[*].Instances[0].PublicDnsName" --output text)
curl $DNS
```

### CLEANING UP
``` sh
aws ecs update-service --cluster markgECS --service web --task-definition web --desired-count 0 --region ap-southeast-1
aws ecs deregister-task-definition --task-definition web:1 --region ap-southeast-1
aws ecs delete-cluster --cluster markgECS --region ap-southeast-1
aws ec2 terminate-instances --instance-ids $INSTANCE_1 --region ap-southeast-1
```