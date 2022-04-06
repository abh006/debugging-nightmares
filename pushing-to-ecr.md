### Pushing container images to ECR
The Checklist:
- Do the Docker Log In
```
aws ecr get-login-password --region region | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.region.amazonaws.com
```
- Build the Image
```
docker build -t aws_account_id.dkr.ecr.region.amazonaws.com/repo_name:image_tag
```
- Almighty Push
```
docker push aws_account_id.dkr.ecr.region.amazonaws.com/repo_name:image_tag
```

PS: Never forget to create repository WITHOUT SPELLING MISTAKES in ECR before pushing
