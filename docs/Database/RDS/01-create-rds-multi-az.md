---
title: Create RDS Multi AZ
description: Create Multi-AZ RDS cluster using CloudFormation.
---

# Create RDS Multi AZ

## Aurora MySQL 5.7

### Engine Version

- `5.7.mysql_aurora.2.07.0`
- `5.7.mysql_aurora.2.07.1`
- `5.7.mysql_aurora.2.07.2`
- `5.7.mysql_aurora.2.07.3`
- `5.7.mysql_aurora.2.07.4`
- `5.7.mysql_aurora.2.07.5`
- `5.7.mysql_aurora.2.07.6`
- `5.7.mysql_aurora.2.07.7`
- `5.7.mysql_aurora.2.07.8`
- `5.7.mysql_aurora.2.07.9`
- `5.7.mysql_aurora.2.11.1`
- `5.7.mysql_aurora.2.11.2`

### Instance Type

??? note "`r3` series"

    - `db.r3.large`
    - `db.r3.xlarge`
    - `db.r3.2xlarge`
    - `db.r3.4xlarge`
    - `db.r3.8xlarge`

??? note "`r4` series"

    - `db.r4.large`
    - `db.r4.xlarge`
    - `db.r4.2xlarge`
    - `db.r4.4xlarge`
    - `db.r4.8xlarge`
    - `db.r4.16xlarge`

??? note "`r5` series"

    - `db.r5.large`
    - `db.r5.xlarge`
    - `db.r5.2xlarge`
    - `db.r5.4xlarge`
    - `db.r5.8xlarge`
    - `db.r5.12xlarge`
    - `db.r5.16xlarge`
    - `db.r5.24xlarge`

??? note "`r6g` series"

    - `db.r6g.large`
    - `db.r6g.xlarge`
    - `db.r6g.2xlarge`
    - `db.r6g.4xlarge`
    - `db.r6g.8xlarge`
    - `db.r6g.12xlarge`
    - `db.r6g.16xlarge`

??? note "`r6i` series"

    - `db.r6i.large`
    - `db.r6i.xlarge`
    - `db.r6i.2xlarge`
    - `db.r6i.4xlarge`
    - `db.r6i.8xlarge`
    - `db.r6i.12xlarge`
    - `db.r6i.16xlarge`
    - `db.r6i.24xlarge`
    - `db.r6i.32xlarge`

??? note "`t2` series"

    - `db.t2.small`
    - `db.t2.medium`
    - `db.t2.large`

??? note "`t3` series"

    - `db.t3.small`
    - `db.t3.medium`
    - `db.t3.large`

??? note "`t4g` series"

    - `db.t4g.medium`
    - `db.t4g.large`

### Commands

=== ":simple-linux: Linux"

    ``` bash
    STACK_NAME=""
    PROJECT_NAME=""
    REGION=""

    ### Subnet Group Configuration
    SubnetGroupName=""              # [REQUIRED] he name of this launch template.
    Subnets=""                      # [REQUIRED] List of database subnet ids.

    ### Parameter Group Configuration
    ClusterParameterGroupName=""    # [REQUIRED] Name of database cluster parameter group.
    ParameterGroupName=""           # [REQUIRED] Name of database parameter group.

    ### Cluster Configuration - General
    ClusterIdentifier=""            # [REQUIRED] Identifier(name) used for database cluster.
    EngineVersion=""                # [REQUIRED] EngineVersion for database. (https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-rds-dbcluster.html#cfn-rds-dbcluster-engineversion)
    KmsKeyId=""                     # [optional] Arn of kms key for database cluster. (If don't specify this property, use default key.)
    DeletionProtection="true"       # `false`(default) or `true` | [optional] State for database deletion protection.

    ### Cluster Configuration - Network
    VpcId=""                        # [REQUIRED] ID of database vpc.
    Port="3306"                     # [REQUIRED] Port number for database instance.
    CreateSecurityGroup="Yes"       # `Yes`(default) or `No` | [REQUIRED] Create a new security group or using existed security group.
    SecurityGroupNameOrId=""        # [REQUIRED] New security group name or existed security group id.

    ### Cluster Configuration - Credential
    Username=""                     # [REQUIRED] Username for database access.
    Password=""                     # [REQUIRED] Password for database access.

    ### Instance Configuration
    InstanceClass=""                # [REQUIRED] Type of database instance type.
    Instance1Identifier=""          # [REQUIRED] Identifier(name) used for database instance 1 (maybe writer)
    Instance2Identifier=""          # [REQUIRED] Identifier(name) used for database instance 2 (maybe reader)
    MonitoringRoleName=""           # [REQUIRED] Name of database monitoring iam role.

    curl -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/rds/multi-az-cluster/aurora-mysql-5.7/rds-cluster.yaml

    aws cloudformation deploy \
        --template-file ./rds-cluster.yaml \
        --stack-name $STACK_NAME \
        --parameter-overrides \
            ProjectName=$PROJECT_NAME \
            SubnetGroupName=$SubnetGroupName \
            Subnets=$Subnets \
            SecurityGroupIds=$SecurityGroupIds \
            ClusterParameterGroupName=$ClusterParameterGroupName \
            ParameterGroupName=$ParameterGroupName \
            ClusterIdentifier=$ClusterIdentifier \
            EngineVersion=$EngineVersion \
            KmsKeyId=$KmsKeyId \
            DeletionProtection=$DeletionProtection \
            VpcId=$VpcId \
            Port=$Port \
            CreateSecurityGroup=$CreateSecurityGroup \
            SecurityGroupNameOrId=$SecurityGroupNameOrId \
            CreateSecurityGroup=$CreateSecurityGroup \
            Username=$Username \
            Password=$Password \
            InstanceClass=$InstanceClass \
            Instance1Identifier=$Instance1Identifier \
            Instance2Identifier=$Instance2Identifier \
            MonitoringRoleName=$MonitoringRoleName \
        --disable-rollback \
        --tags project=$PROJECT_NAME \
        --capabilities CAPABILITY_NAMED_IAM \
        --region $REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell
    $STACK_NAME=""
    $PROJECT_NAME=""
    $REGION=""

    ### Subnet Group Configuration
    $SubnetGroupName=""             # [REQUIRED] he name of this launch template.
    $Subnets=""                     # [REQUIRED] List of database subnet ids.

    ### Parameter Group Configuration
    $ClusterParameterGroupName=""   # [REQUIRED] Name of database cluster parameter group.
    $ParameterGroupName=""          # [REQUIRED] Name of database parameter group.

    ### Cluster Configuration - General
    $ClusterIdentifier=""           # [REQUIRED] Identifier(name) used for database cluster.
    $EngineVersion=""               # [REQUIRED] EngineVersion for database. (https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-rds-dbcluster.html#cfn-rds-dbcluster-engineversion)
    $KmsKeyId=""                    # [optional] Arn of kms key for database cluster. (If don't specify this property, use default key.)
    $DeletionProtection="true"      # `false`(default) or `true` | [optional] State for database deletion protection.

    ### Cluster Configuration - Network
    $VpcId=""                       # [REQUIRED] ID of database vpc.
    $Port="3306"                    # [REQUIRED] Port number for database instance.
    $CreateSecurityGroup="Yes"      # `Yes`(default) or `No` | [REQUIRED] Create a new security group or using existed security group.
    $SecurityGroupNameOrId=""       # [REQUIRED] New security group name or existed security group id.

    ### Cluster Configuration - Credential
    $Username=""                    # [REQUIRED] Username for database access.
    $Password=""                    # [REQUIRED] Password for database access.

    ### Instance Configuration
    $InstanceClass=""               # [REQUIRED] Type of database instance type.
    $Instance1Identifier=""         # [REQUIRED] Identifier(name) used for database instance 1 (maybe writer)
    $Instance2Identifier=""         # [REQUIRED] Identifier(name) used for database instance 2 (maybe reader)
    $MonitoringRoleName=""          # [REQUIRED] Name of database monitoring iam role.

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/rds/multi-az-cluster/aurora-mysql-5.7/rds-cluster.yaml

    aws cloudformation deploy `
        --template-file ./rds-cluster.yaml `
        --stack-name $STACK_NAME `
        --parameter-overrides `
            ProjectName=$PROJECT_NAME `
            SubnetGroupName=$SubnetGroupName `
            Subnets=$Subnets `
            SecurityGroupIds=$SecurityGroupIds `
            ClusterParameterGroupName=$ClusterParameterGroupName `
            ParameterGroupName=$ParameterGroupName `
            GlobalClusterIdentifier=$GlobalClusterIdentifier `
            ClusterIdentifier=$ClusterIdentifier `
            EngineVersion=$EngineVersion `
            KmsKeyId=$KmsKeyId `
            DeletionProtection=$DeletionProtection `
            VpcId=$VpcId `
            Port=$Port `
            CreateSecurityGroup=$CreateSecurityGroup `
            SecurityGroupNameOrId=$SecurityGroupNameOrId `
            CreateSecurityGroup=$CreateSecurityGroup `
            Username=$Username `
            Password=$Password `
            InstanceClass=$InstanceClass `
            Instance1Identifier=$Instance1Identifier `
            Instance2Identifier=$Instance2Identifier `
            MonitoringRoleName=$MonitoringRoleName `
        --disable-rollback `
        --tags project=$PROJECT_NAME `
        --capabilities CAPABILITY_NAMED_IAM `
        --region $REGION
    ```

## Aurora MySQL 8.0

### Engine Version

- `8.0.mysql_aurora.3.01.0`
- `8.0.mysql_aurora.3.01.1`
- `8.0.mysql_aurora.3.02.0`
- `8.0.mysql_aurora.3.02.1`
- `8.0.mysql_aurora.3.02.2`
- `8.0.mysql_aurora.3.02.3`
- `8.0.mysql_aurora.3.03.0`
- `8.0.mysql_aurora.3.03.1`

### Instance Type

??? note "`r5` series"

    - `db.r5.large`
    - `db.r5.xlarge`
    - `db.r5.2xlarge`
    - `db.r5.4xlarge`
    - `db.r5.8xlarge`
    - `db.r5.12xlarge`
    - `db.r5.16xlarge`
    - `db.r5.24xlarge`

??? note "`r6g` series"

    - `db.r6g.large`
    - `db.r6g.xlarge`
    - `db.r6g.2xlarge`
    - `db.r6g.4xlarge`
    - `db.r6g.8xlarge`
    - `db.r6g.12xlarge`
    - `db.r6g.16xlarge`

??? note "`r6i` series"

    - `db.r6i.large`
    - `db.r6i.xlarge`
    - `db.r6i.2xlarge`
    - `db.r6i.4xlarge`
    - `db.r6i.8xlarge`
    - `db.r6i.12xlarge`
    - `db.r6i.16xlarge`
    - `db.r6i.24xlarge`
    - `db.r6i.32xlarge`

??? note "`t3` series"

    - `db.t3.medium`
    - `db.t3.large`

??? note "`t4g` series"

    - `db.t4g.medium`
    - `db.t4g.large`

### Commands

=== ":simple-linux: Linux"

    ``` bash
    STACK_NAME=""
    PROJECT_NAME=""
    REGION=""

    ### Subnet Group Configuration
    SubnetGroupName=""              # [REQUIRED] he name of this launch template.
    Subnets=""                      # [REQUIRED] List of database subnet ids.

    ### Parameter Group Configuration
    ClusterParameterGroupName=""    # [REQUIRED] Name of database cluster parameter group.
    ParameterGroupName=""           # [REQUIRED] Name of database parameter group.

    ### Cluster Configuration - General
    ClusterIdentifier=""            # [REQUIRED] Identifier(name) used for database cluster.
    EngineVersion=""                # [REQUIRED] EngineVersion for database. (https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-rds-dbcluster.html#cfn-rds-dbcluster-engineversion)
    KmsKeyId=""                     # [optional] Arn of kms key for database cluster. (If don't specify this property, use default key.)
    DeletionProtection="true"       # `false`(default) or `true` | [optional] State for database deletion protection.

    ### Cluster Configuration - Network
    VpcId=""                        # [REQUIRED] ID of database vpc.
    Port="3306"                     # [REQUIRED] Port number for database instance.
    CreateSecurityGroup="Yes"       # `Yes`(default) or `No` | [REQUIRED] Create a new security group or using existed security group.
    SecurityGroupNameOrId=""        # [REQUIRED] New security group name or existed security group id.

    ### Cluster Configuration - Credential
    Username=""                     # [REQUIRED] Username for database access.
    Password=""                     # [REQUIRED] Password for database access.

    ### Instance Configuration
    InstanceClass=""                # [REQUIRED] Type of database instance type.
    Instance1Identifier=""          # [REQUIRED] Identifier(name) used for database instance 1 (maybe writer)
    Instance2Identifier=""          # [REQUIRED] Identifier(name) used for database instance 2 (maybe reader)
    MonitoringRoleName=""           # [REQUIRED] Name of database monitoring iam role.

    curl -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/rds/multi-az-cluster/aurora-mysql-8.0-instance/rds-cluster.yaml

    aws cloudformation deploy \
        --template-file ./rds-cluster.yaml \
        --stack-name $STACK_NAME \
        --parameter-overrides \
            ProjectName=$PROJECT_NAME \
            SubnetGroupName=$SubnetGroupName \
            Subnets=$Subnets \
            SecurityGroupIds=$SecurityGroupIds \
            ClusterParameterGroupName=$ClusterParameterGroupName \
            ClusterIdentifier=$ClusterIdentifier \
            EngineVersion=$EngineVersion \
            KmsKeyId=$KmsKeyId \
            DeletionProtection=$DeletionProtection \
            VpcId=$VpcId \
            Port=$Port \
            CreateSecurityGroup=$CreateSecurityGroup \
            SecurityGroupNameOrId=$SecurityGroupNameOrId \
            CreateSecurityGroup=$CreateSecurityGroup \
            Username=$Username \
            Password=$Password \
            InstanceClass=$InstanceClass \
            Instance1Identifier=$Instance1Identifier \
            Instance2Identifier=$Instance2Identifier \
            MonitoringRoleName=$MonitoringRoleName \
        --disable-rollback \
        --tags project=$PROJECT_NAME \
        --capabilities CAPABILITY_NAMED_IAM \
        --region $REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell
    $STACK_NAME=""
    $PROJECT_NAME=""
    $REGION=""

    ### Subnet Group Configuration
    $SubnetGroupName=""             # [REQUIRED] he name of this launch template.
    $Subnets=""                     # [REQUIRED] List of database subnet ids.

    ### Parameter Group Configuration
    $ClusterParameterGroupName=""   # [REQUIRED] Name of database cluster parameter group.
    $ParameterGroupName=""          # [REQUIRED] Name of database parameter group.

    ### Cluster Configuration - General
    $ClusterIdentifier=""           # [REQUIRED] Identifier(name) used for database cluster.
    $EngineVersion=""               # [REQUIRED] EngineVersion for database. (https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-rds-dbcluster.html#cfn-rds-dbcluster-engineversion)
    $KmsKeyId=""                    # [optional] Arn of kms key for database cluster. (If don't specify this property, use default key.)
    $DeletionProtection="true"      # `false`(default) or `true` | [optional] State for database deletion protection.

    ### Cluster Configuration - Network
    $VpcId=""                       # [REQUIRED] ID of database vpc.
    $Port="3306"                    # [REQUIRED] Port number for database instance.
    $CreateSecurityGroup="Yes"      # `Yes`(default) or `No` | [REQUIRED] Create a new security group or using existed security group.
    $SecurityGroupNameOrId=""       # [REQUIRED] New security group name or existed security group id.

    ### Cluster Configuration - Credential
    $Username=""                    # [REQUIRED] Username for database access.
    $Password=""                    # [REQUIRED] Password for database access.

    ### Instance Configuration
    $InstanceClass=""               # [REQUIRED] Type of database instance type.
    $Instance1Identifier=""         # [REQUIRED] Identifier(name) used for database instance 1 (maybe writer)
    $Instance2Identifier=""         # [REQUIRED] Identifier(name) used for database instance 2 (maybe reader)
    $MonitoringRoleName=""          # [REQUIRED] Name of database monitoring iam role.

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/rds/multi-az-cluster/aurora-mysql-8.0-instance/rds-cluster.yaml

    aws cloudformation deploy `
        --template-file ./rds-cluster.yaml `
        --stack-name $STACK_NAME `
        --parameter-overrides `
            ProjectName=$PROJECT_NAME `
            SubnetGroupName=$SubnetGroupName `
            Subnets=$Subnets `
            SecurityGroupIds=$SecurityGroupIds `
            ClusterParameterGroupName=$ClusterParameterGroupName `
            ParameterGroupName=$ParameterGroupName `
            ClusterIdentifier=$ClusterIdentifier `
            EngineVersion=$EngineVersion `
            KmsKeyId=$KmsKeyId `
            DeletionProtection=$DeletionProtection `
            VpcId=$VpcId `
            Port=$Port `
            CreateSecurityGroup=$CreateSecurityGroup `
            SecurityGroupNameOrId=$SecurityGroupNameOrId `
            CreateSecurityGroup=$CreateSecurityGroup `
            Username=$Username `
            Password=$Password `
            InstanceClass=$InstanceClass `
            Instance1Identifier=$Instance1Identifier `
            Instance2Identifier=$Instance2Identifier `
            MonitoringRoleName=$MonitoringRoleName `
        --disable-rollback `
        --tags project=$PROJECT_NAME `
        --capabilities CAPABILITY_NAMED_IAM `
        --region $REGION
    ```

## Aurora MySQL 8.0 Serverless

### Engine Version

- `8.0.mysql_aurora.3.02.0`
- `8.0.mysql_aurora.3.02.1`
- `8.0.mysql_aurora.3.02.2`
- `8.0.mysql_aurora.3.02.3`
- `8.0.mysql_aurora.3.03.0`
- `8.0.mysql_aurora.3.03.1`

### Commands

=== ":simple-linux: Linux"

    ``` bash
    STACK_NAME=""
    PROJECT_NAME=""
    REGION=""

    ### Subnet Group Configuration
    SubnetGroupName=""              # [REQUIRED] he name of this launch template.
    Subnets=""                      # [REQUIRED] List of database subnet ids.

    ### Parameter Group Configuration
    ClusterParameterGroupName=""    # [REQUIRED] Name of database cluster parameter group.
    ParameterGroupName=""           # [REQUIRED] Name of database parameter group.

    ### Cluster Configuration - General
    ClusterIdentifier=""            # [REQUIRED] Identifier(name) used for database cluster.
    EngineVersion=""                # [REQUIRED] EngineVersion for database. (https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-rds-dbcluster.html#cfn-rds-dbcluster-engineversion)
    KmsKeyId=""                     # [optional] Arn of kms key for database cluster. (If don't specify this property, use default key.)
    DeletionProtection="true"       # `false`(default) or `true` | [optional] State for database deletion protection.

    ### Cluster Configuration - Network
    VpcId=""                        # [REQUIRED] ID of database vpc.
    Port="3306"                     # [REQUIRED] Port number for database instance.
    CreateSecurityGroup="Yes"       # `Yes`(default) or `No` | [REQUIRED] Create a new security group or using existed security group.
    SecurityGroupNameOrId=""        # [REQUIRED] New security group name or existed security group id.

    ### Cluster Configuration - Credential
    Username=""                     # [REQUIRED] Username for database access.
    Password=""                     # [REQUIRED] Password for database access.

    ### Cluster Configuration - Capacity
    MinCapacity=""                  # [REQUIRED] Min capacity of serverless cluster. (0.5 ~ 128)
    MaxCapacity=""                  # [REQUIRED] Max capacity of serverless cluster. (1 ~ 128)

    ### Instance Configuration
    Instance1Identifier=""          # [REQUIRED] Identifier(name) used for database instance 1 (maybe writer)
    Instance2Identifier=""          # [REQUIRED] Identifier(name) used for database instance 2 (maybe reader)
    MonitoringRoleName=""           # [REQUIRED] Name of database monitoring iam role.

    curl -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/rds/multi-az-cluster/aurora-mysql-8.0-serverless/rds-cluster.yaml

    aws cloudformation deploy \
        --template-file ./rds-cluster.yaml \
        --stack-name $STACK_NAME \
        --parameter-overrides \
            ProjectName=$PROJECT_NAME \
            SubnetGroupName=$SubnetGroupName \
            Subnets=$Subnets \
            SecurityGroupIds=$SecurityGroupIds \
            ClusterParameterGroupName=$ClusterParameterGroupName \
            ParameterGroupName=$ParameterGroupName \
            ClusterIdentifier=$ClusterIdentifier \
            EngineVersion=$EngineVersion \
            KmsKeyId=$KmsKeyId \
            DeletionProtection=$DeletionProtection \
            VpcId=$VpcId \
            Port=$Port \
            CreateSecurityGroup=$CreateSecurityGroup \
            SecurityGroupNameOrId=$SecurityGroupNameOrId \
            CreateSecurityGroup=$CreateSecurityGroup \
            Username=$Username \
            Password=$Password \
            MinCapacity=$MinCapacity \
            MaxCapacity=$MaxCapacity \
            Instance1Identifier=$Instance1Identifier \
            Instance2Identifier=$Instance2Identifier \
            MonitoringRoleName=$MonitoringRoleName \
        --disable-rollback \
        --tags project=$PROJECT_NAME \
        --capabilities CAPABILITY_NAMED_IAM \
        --region $REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell
    $STACK_NAME=""
    $PROJECT_NAME=""
    $REGION=""

    ### Subnet Group Configuration
    $SubnetGroupName=""             # [REQUIRED] he name of this launch template.
    $Subnets=""                     # [REQUIRED] List of database subnet ids.

    ### Parameter Group Configuration
    $ClusterParameterGroupName=""   # [REQUIRED] Name of database cluster parameter group.
    $ParameterGroupName=""          # [REQUIRED] Name of database parameter group.

    ### Cluster Configuration - General
    $ClusterIdentifier=""           # [REQUIRED] Identifier(name) used for database cluster.
    $EngineVersion=""               # [REQUIRED] EngineVersion for database. (https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-rds-dbcluster.html#cfn-rds-dbcluster-engineversion)
    $KmsKeyId=""                    # [optional] Arn of kms key for database cluster. (If don't specify this property, use default key.)
    $DeletionProtection="true"      # `false`(default) or `true` | [optional] State for database deletion protection.

    ### Cluster Configuration - Network
    $VpcId=""                       # [REQUIRED] ID of database vpc.
    $Port="3306"                    # [REQUIRED] Port number for database instance.
    $CreateSecurityGroup="Yes"      # `Yes`(default) or `No` | [REQUIRED] Create a new security group or using existed security group.
    $SecurityGroupNameOrId=""       # [REQUIRED] New security group name or existed security group id.

    ### Cluster Configuration - Credential
    $Username=""                    # [REQUIRED] Username for database access.
    $Password=""                    # [REQUIRED] Password for database access.

    ### Cluster Configuration - Capacity
    $MinCapacity=""                 # [REQUIRED] Min capacity of serverless cluster. (0.5 ~ 128)
    $MaxCapacity=""                 # [REQUIRED] Max capacity of serverless cluster. (1 ~ 128)

    ### Instance Configuration
    $Instance1Identifier=""         # [REQUIRED] Identifier(name) used for database instance 1 (maybe writer)
    $Instance2Identifier=""         # [REQUIRED] Identifier(name) used for database instance 2 (maybe reader)
    $MonitoringRoleName=""          # [REQUIRED] Name of database monitoring iam role.

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/rds/multi-az-cluster/aurora-mysql-8.0-serverless/rds-cluster.yaml

    aws cloudformation deploy `
        --template-file ./rds-cluster.yaml `
        --stack-name $STACK_NAME `
        --parameter-overrides `
            ProjectName=$PROJECT_NAME `
            SubnetGroupName=$SubnetGroupName `
            Subnets=$Subnets `
            SecurityGroupIds=$SecurityGroupIds `
            ClusterParameterGroupName=$ClusterParameterGroupName `
            ParameterGroupName=$ParameterGroupName `
            ClusterIdentifier=$ClusterIdentifier `
            EngineVersion=$EngineVersion `
            KmsKeyId=$KmsKeyId `
            DeletionProtection=$DeletionProtection `
            VpcId=$VpcId `
            Port=$Port `
            CreateSecurityGroup=$CreateSecurityGroup `
            SecurityGroupNameOrId=$SecurityGroupNameOrId `
            CreateSecurityGroup=$CreateSecurityGroup `
            Username=$Username `
            Password=$Password `
            MinCapacity=$MinCapacity `
            MaxCapacity=$MaxCapacity `
            Instance1Identifier=$Instance1Identifier `
            Instance2Identifier=$Instance2Identifier `
            MonitoringRoleName=$MonitoringRoleName `
        --disable-rollback `
        --tags project=$PROJECT_NAME `
        --capabilities CAPABILITY_NAMED_IAM `
        --region $REGION
    ```

## Aurora PostgreSQL

### Cluster Parameter Group Family

- `aurora-postgresql11`
- `aurora-postgresql12`
- `aurora-postgresql13`
- `aurora-postgresql14`
- `aurora-postgresql15`

### Engine Version

??? note "`11` versions"

    - `11.9`
    - `11.13`
    - `11.14`
    - `11.15`
    - `11.16`
    - `11.17`
    - `11.18`
    - `11.19`

??? note "`12` versions"

    - `12.8`
    - `12.9`
    - `12.10`
    - `12.11`
    - `12.12`
    - `12.13`
    - `12.14`

??? note "`13` versions"

    - `13.4`
    - `13.5`
    - `13.6`
    - `13.7`
    - `13.8`
    - `13.9`
    - `13.10`

??? note "`14` versions"

    - `14.3`
    - `14.4`
    - `14.5`
    - `14.6`
    - `14.7`

??? note "`15` versions"

    - `15.2`

### Instance Type

??? note "`r4` series"

    - `db.r4.large`
    - `db.r4.xlarge`
    - `db.r4.2xlarge`
    - `db.r4.4xlarge`
    - `db.r4.8xlarge`
    - `db.r4.16xlarge`

??? note "`r5` series"

    - `db.r5.large`
    - `db.r5.xlarge`
    - `db.r5.2xlarge`
    - `db.r5.4xlarge`
    - `db.r5.8xlarge`
    - `db.r5.12xlarge`
    - `db.r5.16xlarge`
    - `db.r5.24xlarge`

??? note "`r6g` series"

    - `db.r6g.large`
    - `db.r6g.xlarge`
    - `db.r6g.2xlarge`
    - `db.r6g.4xlarge`
    - `db.r6g.8xlarge`
    - `db.r6g.12xlarge`
    - `db.r6g.16xlarge`

??? note "`r6i` series"

    - `db.r6i.large`
    - `db.r6i.xlarge`
    - `db.r6i.2xlarge`
    - `db.r6i.4xlarge`
    - `db.r6i.8xlarge`
    - `db.r6i.12xlarge`
    - `db.r6i.16xlarge`
    - `db.r6i.24xlarge`
    - `db.r6i.32xlarge`

??? note "`t3` series"

    - `db.t3.medium`
    - `db.t3.large`

??? note "`t4g` series"

    - `db.t4g.medium`
    - `db.t4g.large`

### Commands

=== ":simple-linux: Linux"

    ``` bash
    STACK_NAME=""
    PROJECT_NAME=""
    REGION=""

    ### Subnet Group Configuration
    SubnetGroupName=""              # [REQUIRED] he name of this launch template.
    Subnets=""                      # [REQUIRED] List of database subnet ids.

    ### Parameter Group Configuration
    ClusterParameterGroupName=""    # [REQUIRED] Name of database cluster parameter group.
    ParameterGroupName=""           # [REQUIRED] Name of database parameter group.
    ClusterParameterGroupFamily=""  # [REQUIRED] Name of database cluster parameter group family.

    ### Cluster Configuration - General
    ClusterIdentifier=""            # [REQUIRED] Identifier(name) used for database cluster.
    EngineVersion=""                # [REQUIRED] EngineVersion for database. (https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-rds-dbcluster.html#cfn-rds-dbcluster-engineversion)
    KmsKeyId=""                     # [optional] Arn of kms key for database cluster. (If don't specify this property, use default key.)
    DeletionProtection="true"       # `false`(default) or `true` | [optional] State for database deletion protection.

    ### Cluster Configuration - Network
    VpcId=""                        # [REQUIRED] ID of database vpc.
    Port="5432"                     # [REQUIRED] Port number for database instance.
    CreateSecurityGroup="Yes"       # `Yes`(default) or `No` | [REQUIRED] Create a new security group or using existed security group.
    SecurityGroupNameOrId=""        # [REQUIRED] New security group name or existed security group id.

    ### Cluster Configuration - Credential
    Username=""                     # [REQUIRED] Username for database access.
    Password=""                     # [REQUIRED] Password for database access.

    ### Instance Configuration
    InstanceClass=""                # [REQUIRED] Type of database instance type.
    Instance1Identifier=""          # [REQUIRED] Identifier(name) used for database instance 1 (maybe writer)
    Instance2Identifier=""          # [REQUIRED] Identifier(name) used for database instance 2 (maybe reader)
    MonitoringRoleName=""           # [REQUIRED] Name of database monitoring iam role.

    curl -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/rds/multi-az-cluster/aurora-postgresql-instance/rds-cluster.yaml

    aws cloudformation deploy \
        --template-file ./rds-cluster.yaml \
        --stack-name $STACK_NAME \
        --parameter-overrides \
            ProjectName=$PROJECT_NAME \
            SubnetGroupName=$SubnetGroupName \
            Subnets=$Subnets \
            SecurityGroupIds=$SecurityGroupIds \
            ClusterParameterGroupName=$ClusterParameterGroupName \
            ParameterGroupName=$ParameterGroupName \
            ClusterParameterGroupFamily=$ClusterParameterGroupFamily \
            ClusterIdentifier=$ClusterIdentifier \
            EngineVersion=$EngineVersion \
            KmsKeyId=$KmsKeyId \
            DeletionProtection=$DeletionProtection \
            VpcId=$VpcId \
            Port=$Port \
            CreateSecurityGroup=$CreateSecurityGroup \
            SecurityGroupNameOrId=$SecurityGroupNameOrId \
            CreateSecurityGroup=$CreateSecurityGroup \
            Username=$Username \
            Password=$Password \
            InstanceClass=$InstanceClass \
            Instance1Identifier=$Instance1Identifier \
            Instance2Identifier=$Instance2Identifier \
            MonitoringRoleName=$MonitoringRoleName \
        --disable-rollback \
        --tags project=$PROJECT_NAME \
        --capabilities CAPABILITY_NAMED_IAM \
        --region $REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell
    $STACK_NAME=""
    $PROJECT_NAME=""
    $REGION=""

    ### Subnet Group Configuration
    $SubnetGroupName=""             # [REQUIRED] he name of this launch template.
    $Subnets=""                     # [REQUIRED] List of database subnet ids.

    ### Parameter Group Configuration
    $ClusterParameterGroupName=""   # [REQUIRED] Name of database cluster parameter group.
    $ParameterGroupName=""          # [REQUIRED] Name of database parameter group.
    $ClusterParameterGroupFamily="" # [REQUIRED] Name of database cluster parameter group family.

    ### Cluster Configuration - General
    $ClusterIdentifier=""           # [REQUIRED] Identifier(name) used for database cluster.
    $EngineVersion=""               # [REQUIRED] EngineVersion for database. (https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-rds-dbcluster.html#cfn-rds-dbcluster-engineversion)
    $KmsKeyId=""                    # [optional] Arn of kms key for database cluster. (If don't specify this property, use default key.)
    $DeletionProtection="true"      # `false`(default) or `true` | [optional] State for database deletion protection.

    ### Cluster Configuration - Network
    $VpcId=""                       # [REQUIRED] ID of database vpc.
    $Port="5432"                    # [REQUIRED] Port number for database instance.
    $CreateSecurityGroup="Yes"      # `Yes`(default) or `No` | [REQUIRED] Create a new security group or using existed security group.
    $SecurityGroupNameOrId=""       # [REQUIRED] New security group name or existed security group id.

    ### Cluster Configuration - Credential
    $Username=""                    # [REQUIRED] Username for database access.
    $Password=""                    # [REQUIRED] Password for database access.

    ### Instance Configuration
    $InstanceClass=""               # [REQUIRED] Type of database instance type.
    $Instance1Identifier=""         # [REQUIRED] Identifier(name) used for database instance 1 (maybe writer)
    $Instance2Identifier=""         # [REQUIRED] Identifier(name) used for database instance 2 (maybe reader)
    $MonitoringRoleName=""          # [REQUIRED] Name of database monitoring iam role.

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/rds/multi-az-cluster/aurora-postgresql-instance/rds-cluster.yaml

    aws cloudformation deploy `
        --template-file ./rds-cluster.yaml `
        --stack-name $STACK_NAME `
        --parameter-overrides `
            ProjectName=$PROJECT_NAME `
            SubnetGroupName=$SubnetGroupName `
            Subnets=$Subnets `
            SecurityGroupIds=$SecurityGroupIds `
            ClusterParameterGroupName=$ClusterParameterGroupName `
            ParameterGroupName=$ParameterGroupName `
            ClusterParameterGroupFamily=$ClusterParameterGroupFamily `
            ClusterIdentifier=$ClusterIdentifier `
            EngineVersion=$EngineVersion `
            KmsKeyId=$KmsKeyId `
            DeletionProtection=$DeletionProtection `
            VpcId=$VpcId `
            Port=$Port `
            CreateSecurityGroup=$CreateSecurityGroup `
            SecurityGroupNameOrId=$SecurityGroupNameOrId `
            CreateSecurityGroup=$CreateSecurityGroup `
            Username=$Username `
            Password=$Password `
            InstanceClass=$InstanceClass `
            Instance1Identifier=$Instance1Identifier `
            Instance2Identifier=$Instance2Identifier `
            MonitoringRoleName=$MonitoringRoleName `
        --disable-rollback `
        --tags project=$PROJECT_NAME `
        --capabilities CAPABILITY_NAMED_IAM `
        --region $REGION
    ```

## Aurora PostgreSQL Serverless

### Cluster Parameter Group Family

- `aurora-postgresql13`
- `aurora-postgresql14`
- `aurora-postgresql15`

### Engine Version

??? note "`13` versions"

    - `13.6`
    - `13.7`
    - `13.8`
    - `13.9`
    - `13.10`

??? note "`14` versions"

    - `14.3`
    - `14.4`
    - `14.5`
    - `14.6`
    - `14.7`

??? note "`15` versions"

    - `15.2`

### Commands

=== ":simple-linux: Linux"

    ``` bash
    STACK_NAME=""
    PROJECT_NAME=""
    REGION=""

    ### Subnet Group Configuration
    SubnetGroupName=""              # [REQUIRED] he name of this launch template.
    Subnets=""                      # [REQUIRED] List of database subnet ids.

    ### Parameter Group Configuration
    ClusterParameterGroupName=""    # [REQUIRED] Name of database cluster parameter group.
    ParameterGroupName=""           # [REQUIRED] Name of database parameter group.
    ClusterParameterGroupFamily=""  # [REQUIRED] Name of database cluster parameter group family.

    ### Cluster Configuration - General
    ClusterIdentifier=""            # [REQUIRED] Identifier(name) used for database cluster.
    EngineVersion=""                # [REQUIRED] EngineVersion for database. (https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-rds-dbcluster.html#cfn-rds-dbcluster-engineversion)
    KmsKeyId=""                     # [optional] Arn of kms key for database cluster. (If don't specify this property, use default key.)
    DeletionProtection="true"       # `false`(default) or `true` | [optional] State for database deletion protection.

    ### Cluster Configuration - Network
    VpcId=""                        # [REQUIRED] ID of database vpc.
    Port="5432"                     # [REQUIRED] Port number for database instance.
    CreateSecurityGroup="Yes"       # `Yes`(default) or `No` | [REQUIRED] Create a new security group or using existed security group.
    SecurityGroupNameOrId=""        # [REQUIRED] New security group name or existed security group id.

    ### Cluster Configuration - Credential
    Username=""                     # [REQUIRED] Username for database access.
    Password=""                     # [REQUIRED] Password for database access.

    ### Cluster Configuration - Capacity
    MinCapacity=""                  # [REQUIRED] Min capacity of serverless cluster. (0.5 ~ 128)
    MaxCapacity=""                  # [REQUIRED] Max capacity of serverless cluster. (1 ~ 128)

    ### Instance Configuration
    Instance1Identifier=""          # [REQUIRED] Identifier(name) used for database instance 1 (maybe writer)
    Instance2Identifier=""          # [REQUIRED] Identifier(name) used for database instance 2 (maybe reader)
    MonitoringRoleName=""           # [REQUIRED] Name of database monitoring iam role.

    curl -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/rds/multi-az-cluster/aurora-postgresql-serverless/rds-cluster.yaml

    aws cloudformation deploy \
        --template-file ./rds-cluster.yaml \
        --stack-name $STACK_NAME \
        --parameter-overrides \
            ProjectName=$PROJECT_NAME \
            SubnetGroupName=$SubnetGroupName \
            Subnets=$Subnets \
            SecurityGroupIds=$SecurityGroupIds \
            ClusterParameterGroupName=$ClusterParameterGroupName \
            ParameterGroupName=$ParameterGroupName \
            ClusterParameterGroupFamily=$ClusterParameterGroupFamily \
            ClusterIdentifier=$ClusterIdentifier \
            EngineVersion=$EngineVersion \
            KmsKeyId=$KmsKeyId \
            DeletionProtection=$DeletionProtection \
            VpcId=$VpcId \
            Port=$Port \
            CreateSecurityGroup=$CreateSecurityGroup \
            SecurityGroupNameOrId=$SecurityGroupNameOrId \
            CreateSecurityGroup=$CreateSecurityGroup \
            Username=$Username \
            Password=$Password \
            MinCapacity=$MinCapacity \
            MaxCapacity=$MaxCapacity \
            Instance1Identifier=$Instance1Identifier \
            Instance2Identifier=$Instance2Identifier \
            MonitoringRoleName=$MonitoringRoleName \
        --disable-rollback \
        --tags project=$PROJECT_NAME \
        --capabilities CAPABILITY_NAMED_IAM \
        --region $REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell
    $STACK_NAME=""
    $PROJECT_NAME=""
    $REGION=""

    ### Subnet Group Configuration
    $SubnetGroupName=""             # [REQUIRED] he name of this launch template.
    $Subnets=""                     # [REQUIRED] List of database subnet ids.

    ### Parameter Group Configuration
    $ClusterParameterGroupName=""   # [REQUIRED] Name of database cluster parameter group.
    $ParameterGroupName=""          # [REQUIRED] Name of database parameter group.
    $ClusterParameterGroupFamily="" # [REQUIRED] Name of database cluster parameter group family.

    ### Cluster Configuration - General
    $ClusterIdentifier=""           # [REQUIRED] Identifier(name) used for database cluster.
    $EngineVersion=""               # [REQUIRED] EngineVersion for database. (https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-rds-dbcluster.html#cfn-rds-dbcluster-engineversion)
    $KmsKeyId=""                    # [optional] Arn of kms key for database cluster. (If don't specify this property, use default key.)
    $DeletionProtection="true"      # `false`(default) or `true` | [optional] State for database deletion protection.

    ### Cluster Configuration - Network
    $VpcId=""                       # [REQUIRED] ID of database vpc.
    $Port="5432"                    # [REQUIRED] Port number for database instance.
    $CreateSecurityGroup="Yes"      # `Yes`(default) or `No` | [REQUIRED] Create a new security group or using existed security group.
    $SecurityGroupNameOrId=""       # [REQUIRED] New security group name or existed security group id.

    ### Cluster Configuration - Credential
    $Username=""                    # [REQUIRED] Username for database access.
    $Password=""                    # [REQUIRED] Password for database access.

    ### Cluster Configuration - Capacity
    $MinCapacity=""                 # [REQUIRED] Min capacity of serverless cluster. (0.5 ~ 128)
    $MaxCapacity=""                 # [REQUIRED] Max capacity of serverless cluster. (1 ~ 128)

    ### Instance Configuration
    $Instance1Identifier=""         # [REQUIRED] Identifier(name) used for database instance 1 (maybe writer)
    $Instance2Identifier=""         # [REQUIRED] Identifier(name) used for database instance 2 (maybe reader)
    $MonitoringRoleName=""          # [REQUIRED] Name of database monitoring iam role.

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/rds/multi-az-cluster/aurora-postgresql-serverless/rds-cluster.yaml

    aws cloudformation deploy `
        --template-file ./rds-cluster.yaml `
        --stack-name $STACK_NAME `
        --parameter-overrides `
            ProjectName=$PROJECT_NAME `
            SubnetGroupName=$SubnetGroupName `
            Subnets=$Subnets `
            SecurityGroupIds=$SecurityGroupIds `
            ClusterParameterGroupName=$ClusterParameterGroupName `
            ParameterGroupName=$ParameterGroupName `
            ClusterParameterGroupFamily=$ClusterParameterGroupFamily `
            ClusterIdentifier=$ClusterIdentifier `
            EngineVersion=$EngineVersion `
            KmsKeyId=$KmsKeyId `
            DeletionProtection=$DeletionProtection `
            VpcId=$VpcId `
            Port=$Port `
            CreateSecurityGroup=$CreateSecurityGroup `
            SecurityGroupNameOrId=$SecurityGroupNameOrId `
            CreateSecurityGroup=$CreateSecurityGroup `
            Username=$Username `
            Password=$Password `
            MinCapacity=$MinCapacity `
            MaxCapacity=$MaxCapacity `
            Instance1Identifier=$Instance1Identifier `
            Instance2Identifier=$Instance2Identifier `
            MonitoringRoleName=$MonitoringRoleName `
        --disable-rollback `
        --tags project=$PROJECT_NAME `
        --capabilities CAPABILITY_NAMED_IAM `
        --region $REGION
    ```