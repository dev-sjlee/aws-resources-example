# Build with Custom Image

## Using ECR Private Repository

### Add Resource-Based Policy

``` json title="ecr-repository-permission.json" hl_lines="17 18"
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Sid":"CodeBuildAccessPrincipal",
         "Effect":"Allow",
         "Principal":{
            "Service":"codebuild.amazonaws.com"
         },
         "Action":[
            "ecr:GetDownloadUrlForLayer",
            "ecr:BatchGetImage",
            "ecr:BatchCheckLayerAvailability"
         ],
         "Condition":{
            "StringEquals":{
               "aws:SourceArn":"arn:aws:codebuild:<region>:<aws-account-id>:project/<project-name>",
               "aws:SourceAccount":"<aws-account-id>"
            }
         }
      }
   ]
}
```