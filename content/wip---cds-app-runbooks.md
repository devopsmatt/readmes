# Troubleshooting AWS

## Table of Contents

[Introduction](#introduction)

[ECS](#ecs)

[Common ECS Deployment Failures](#common-ecs-deployment-failures)

[1. Failure to Start Tasks](#1.-failure-to-start-tasks)

[2. Service Slow/High Load/Out of Resources Issues](#2.-service-slow/high-load/out-of-resources-issues)

## Introduction

The contents of this document are generalized to make it applicable to all CDSP applications and services.  Some references are represented as $variables, which are meant to be substituted with appropriate strings corresponding to your particular application, workload, etc.

| Variable | Example replacement | Description |
|---|---|---|
| $APPSHORT | medcalc-api, myworkweek, po-service, etc | Service name - reference |
| $ENVIRONMENT | dev, stage, prod | Environment where the issue occurs |
|  |  |  |

**Warning:**

While making manual changes in the AWS console may be the fastest way to recover from a failure, all manual changes will not persist unless they are also reflected in the Infrastructure as Code (IaC), which is the Terraform code living in your services github repository.  Unless the manual changes are reflected in the Terraform code, they will be reverted on the next infrastructure deployment Github Workflow run.

**Note:**

If you need assistance with updating Terraform code, please reach out to the DevOps team in [slack](https://dsva.slack.com/archives/C06EL19RZR9) and create a ticket describing your modification details in github, with tag `TEAM:DevOps` as soon as possible after making manual changes.  If you are not comfortable with making infrastructure changes or are unsure what to do, please create a ticket for the DevOps team, tagged with `TEAM: DevOps` and notify us in the `#cds-devops-public` slack channel.

# ECS

### **Common ECS Deployment Failures**

#### **1. Failure to Start Tasks**

**Symptoms:** Tasks fail to start or do not register as healthy after deployment.

**Diagnosis Steps:**

1. ECS Console Events: Check the ECS service events by navigating to the[ ECS Console Services](https://us-gov-east-1.console.amazonaws-us-gov.com/ecs/v2/clusters?region=us-gov-east-1), selecting the *environment-specific (dev, stage, prod) *Fargate cluster:

- [Dev cluster](https://us-gov-east-1.console.amazonaws-us-gov.com/ecs/v2/clusters/dev-fargate-cluster/services?region=us-gov-east-1)

- [Stage cluster](https://us-gov-east-1.console.amazonaws-us-gov.com/ecs/v2/clusters/stage-fargate-cluster/services?region=us-gov-east-1)

- [Prod cluster](https://us-gov-east-1.console.amazonaws-us-gov.com/ecs/v2/clusters/prod-fargate-cluster/services?region=us-gov-east-1)

 then clicking on your service name, and inspecting entries under the "Events" tab for errors.

2. Ensure your service?s health check is not failing: Navigate to the environment-specific Fargate cluster:

- [Dev cluster](https://us-gov-east-1.console.amazonaws-us-gov.com/ecs/v2/clusters/dev-fargate-cluster/services?region=us-gov-east-1)

- [Stage cluster](https://us-gov-east-1.console.amazonaws-us-gov.com/ecs/v2/clusters/stage-fargate-cluster/services?region=us-gov-east-1)

- [Prod cluster](https://us-gov-east-1.console.amazonaws-us-gov.com/ecs/v2/clusters/prod-fargate-cluster/services?region=us-gov-east-1)

then click on your service name (format: $ENVIRONMENT-$APPSHORT, i.e. `dev-myapp`), then inspect the information under the ?Health and Metrics? tab.  Ensure the health check path is correct.  If it is not, modify Terraform?s configuration JSON file corresponding to your application and environment to point to the correct endpoint (MedCalc Dev [reference](https://github.com/department-of-veterans-affairs/medical-calculators-api/blob/dev/terraform/resources/mc-api/dev.auto.tfvars.json#L11)).

3. Task and Service Definitions Review: Examine the task and service definitions for configuration errors.

4. Logs: Review application logs in [Datadog](https://cpm.ddog-gov.com/logs?query=&agg_m=count&agg_m_source=base&agg_t=count&cols=host%2Cservice&fromUser=true&messageDisplay=inline&refresh_mode=sliding&storage=live&stream_sort=desc&viz=stream&from_ts=1714505364943&to_ts=1714506264943&live=true) for your workload?s collected stdout, stderr and custom logs for errors and context.

**Resolution Steps:**

1. Confirm the container image and repository access.

2. Ensure that the health check path is correct after navigating to the appropriate Fargate cluster and clicking your workload?s name, under the ?Health and Metrics? tab.  If it is incorrect, navigate to

3. Verify task and service definitions are error-free.  If errors aree found create a new revision with your modifications - be sure to check the 'force redeployment' checkbox to redeploy the latest version of your service known to ECS.

4. Datadog will send service error alerts to your application?s alerts Slack channel with a link to the corresponding log entries.  Please review them and make necessary adjustments to resolve the issue.

#### **2. Service Slow/High Load/Out of Resources Issues**

**Symptoms:** ECS service tasks are slow, crashing on launch or running at high CPU/memory load.

**Diagnosis Steps:**

1. Look at your workload?s operating environment load in the AWS console ( `https://us-gov-east-1.console.amazonaws-us-gov.com/ecs/v2/clusters/(``$ENVIRONMENT``-fargate-cluster/services/``$APPSHORT``/health?region=us-gov-east-1` )  If you see high memory or CPU you may want to add resources to your workload.

2. Health Check Configurations: Confirm health check configurations and ensure they align with expected workload responses.

3. Review Datadog logs and identify the source of issue.

Resolution Steps:

1. Add resources by modification of the task definition.

2. Ensure the path to your workload?s health check endpoint is correct and that health check settings if necessary.

3. Verify and update security group and network ACL settings to permit load balancer traffic.
