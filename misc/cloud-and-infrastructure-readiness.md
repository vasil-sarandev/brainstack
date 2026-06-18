# Cloud & Infrastructure Readiness

## Resources and tools

1. **Docker**
2. **Github actions**
3. **AWS:** Learning centre(AWS Cloud Practitioner course) is a great place to start. S3, EC2, ECS, RDS, EKS, ALB & IAM should definitely be looked at. 
4. **DevOps fundamentals**: roadmaps.sh
5. **Gateways** - AWS Gateway
6. **Load Balancers -** ALB
7. **Monitoring:** Prometheus, Grafana, Coralogix, Open Telemetry
8. **Kubernetes**

## Putting the resources into practice

### **First hands-on** 

(The focus here is Github Actions and Docker images).

Create an automated pipeline that runs the tests, pushes an image to ECR or GHCR and deploys it to ECS, set it up with GitHub actions and Github Secrets for the env stuff. 

### **Second hands-on**

(Putting the some of the AWS stuff into practice) 

- Create an automated pipeline that runs the tests, pushes an image to a registry and deploys it to ECS, set it up with GitHub actions and Github 
- Secrets for the env stuff. Also possibly 2 environments (dev-stage);
- Spin up a Postgres Database in RDS and connect it to the app deployment
- Setup the load balancer (ALB)
- Setup the autoscaling rules
- Set up IAM.
- Possibly think of something to do with S3

### **Third hands-on:**

- Move the deployment to Kubernetes and use EKS instead of ECS
- Should still work with health monitoring, logs, autoscaling rules, …etc