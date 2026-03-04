# Lab Solution: IAM Policies and Roles

**Student Name:** Maryam Ahmadi  
**Date:** 04.02.2026
  
**Lab Completion Time:** 60 minutes

---

## Part 1: Understanding IAM Policy Structure

### Task 1: Policy Components Explanation

**Explain each component in your own words:**

**Version:**
```
Specifies the AWS policy language version. Always "2012-10-17", which is the latest stable version.
```

**Statement:**
```

A list of permissions and rules this policy enforces. Each Statement is a separate permission.

```

**Sid:**
```
An optional identifier for the Statement to make policies easier to organize and reference.

```

**Effect:**
```
Specifies whether the Action is allowed ("Allow") or denied ("Deny").

```

**Action:**
```
Lists AWS operations that are allowed or denied (e.g., s3:GetObject).

```

**Resource:**
```
Specifies the ARN of the resources the Actions apply to (e.g., a specific S3 bucket or object).

```

---

## Part 2: Custom IAM Policies Created

### S3 Read-Only Policy

**Policy Name:** S3-ReadOnly-IP-Restricted

**Bucket Name Used:** testmaryamlab

**Policy JSON:**
JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowListFromMyIP",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::testmaryamlab",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "94.139.28.25/32"
        }
      }
    },
    {
      "Sid": "AllowReadObjectsFromMyIP",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::testmaryamlab/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "94.139.28.25/32"
        }
      }
    }
  ]
}```

**Screenshot 1: S3 Custom Policy**
![S3 Policy](screenshots/01-s3-policy.png)

---

### EC2 Start/Stop Policy

**Policy Name:** EC2-StartStop-Only

**Policy ARN:** EC2-StartStop-Only

**Screenshot 2: EC2 Custom Policy**
![EC2 Policy](screenshots/02-ec2-policy.png)

---

### CloudWatch Logs Write Policy

**Policy Name:** CloudWatch-Logs-Write-Only

**Policy ARN:** arn:aws:iam::202145728564:policy/CloudWatch-Logs-Write-Only

**Screenshot 3: CloudWatch Logs Policy**
![CloudWatch Policy](screenshots/03-cloudwatch-policy.png)

---

## Part 3: Policy Attachments

### Policy Attached to User

**User Name:** test-developer

**Policy Attached:** S3-ReadOnly-IP-Restricted

**Attachment Method:**  ☑ Console ☐ CLI

**CLI Command (if used):**
for myself

```bash

aws iam attach-user-policy \
    --policy-arn arn:aws:iam::202145728564:policy/S3-ReadOnly-IP-Restricted \
    --user-name test-developer

```

**Screenshot 4: Policy Attached**
![Policy Attachment](screenshots/04-policy-attached.png)

---

## Part 4: IAM Roles Created

### EC2 Service Role

**Role Name:** EC2-S3-ReadOnly-Role


**Role ARN:** arn:aws:iam::202145728564:role/EC2-S3-ReadOnly-Role

**Trusted Entity:** EC2 (AWS Service)

**Attached Policies:**
1. AmazonS3ReadOnlyAccess
2. ___________________________

**Trust Relationship JSON:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

**Screenshot 5: EC2 Service Role**
![EC2 Role](screenshots/05-ec2-role.png)

---

### Lambda Execution Role

**Role Name:** Lambda-Basic-Execution-Role

**Role ARN:** arn:aws:iam::202145728564:role/Lambda-Basic-Execution-Role

**Attached Policies:**
1. AWSLambdaBasicExecutionRole
2. CloudWatch-Logs-Write-Only

**Screenshot 6: Lambda Role**
![Lambda Role](screenshots/06-lambda-role.png)

---

### Cross-Account Access Role

**Role Name:** CrossAccount-ReadOnly-Role

**Role ARN:** arn:aws:iam::202145728564:role/CrossAccount-ReadOnly-Role

**External Account ID:** 202145728564 (for testing, used same account)

**External ID:** test-external-id-001

**Attached Policies:**
1. ReadOnlyAccess

**Screenshot 7: Cross-Account Role**
![Cross-Account Role](screenshots/07-cross-account-role.png)

---

## Part 5: Policy Testing

### Policy Simulator Results

**Policy Tested:** S3-ReadOnly-IP-Restricted

**Test Results:**

Action	Expected Result	Actual Result	Pass/Fail
s3:GetObject	Allowed	Allowed	☑ Pass
s3:PutObject	Denied	Denied	☑ Pass
s3:DeleteObject	Denied	Denied	☑ Pass

**Screenshot 8: Policy Simulator**
![Policy Simulator](screenshots/08-policy-simulator.png)

---

### AWS CLI Testing

**Test 1: S3 List Bucket**
```bash
# Command:
aws s3 ls s3://testmaryamlab/ --profile test-developer

# Output:
2026-03-04 14:28:24          6 realfile.txt
List of files (if any) or empty, no AccessDenied

# Result: ☑ Success ☐ Access Denied
```

**Test 2: S3 Upload File**
```bash
# Command:
echo "test" > test.txt
aws s3 cp test.txt s3://testmaryamlab/ --profile test-developer

# Output:

Output: AccessDenied
upload failed: ./test.txt to s3://testmaryamlab/test.txt An error occurred (AccessDenied) when calling the PutObject operation: User: arn:aws:iam::202145728564:user/test-developer is not authorized to perform: s3:PutObject on resource: "arn:aws:s3:::testmaryamlab/test.txt" because no identity-based policy allows the s3:PutObject action


# Result: ☐ Success ☑  Access Denied (Expected)
```

**Test 3: S3 Download File**
```bash
# Command:

aws s3 cp s3://testmaryamlab/realfile.txt ./ --profile test-developer

# Output:
Success (if file exists and from same IP)



# Result: ☑ Success ☐ Access Denied
```

---

## Part 6: Least Privilege Implementation

### Custom Policy with Conditions

**Policy Name:** S3-ReadOnly-IP-Restricted

**Condition Type Used:** ☑ IP Address ☐ Time Window ☐ MFA ☐ Other: _______

**Policy JSON:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        
      ],
      "Resource": "",
      "Condition": {
        
      }
    }
  ]
}
```

**Rationale for this policy:**
```
Restricts S3 access only to the test-developer user and only from the specific IP.  
Even if access keys are exposed, access from other IPs is blocked.


```
---

## Part 7: Troubleshooting

### Issue Encountered (if any)

Commands Used to Diagnose:
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::202145728564:user/test-developer \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::testmaryamlab/realfile.txt

aws iam get-role \
  --role-name CrossAccount-ReadOnly-Role \
  --query 'Role.AssumeRolePolicyDocument'

aws sts assume-role \
  --role-arn arn:aws:iam::202145728564:role/CrossAccount-ReadOnly-Role \
  --role-session-name test-session \
  --external-id test-external-id-001

## Resolution
- The implicitDeny in simulation is expected and indicates the IP condition is working correctly.
- Assuming the role with the external ID was successful.


**Screenshot 9: Troubleshooting Output**
![Troubleshooting](screenshots/09-troubleshooting.png)

---

## Reflection Questions

### 1. Why are IAM roles preferred over access keys for EC2 instances?

**Your answer:**
```
IAM roles allow EC2 instances to access AWS resources securely without storing access keys.  
It provides temporary credentials, easier key management, and automatic rotation.


### 2. Explain the principle of least privilege and how you applied it in this lab.

**Your answer:**
```
Least Privilege means granting only the minimum permissions required.  
In this lab, policies were created with read-only permissions and restricted to the specific IP, applying least privilege for S3, EC2, and CloudWatch.


### 3. What is the difference between identity-based and resource-based policies?

**Your answer:**
```
- Identity-based: Attached to a user or role to grant permissions.  
- Resource-based: Attached to the resource itself (e.g., S3 bucket) to control who can access it.

### 4. When would you use an explicit "Deny" in a policy?

**Your answer:**
```
Use explicit Deny when you want to ensure an action cannot be performed, even if other permissions allow it.  
Example: Prevent terminating EC2 instances in the EC2-StartStop policy.

### 5. Describe a scenario where you'd use conditions in IAM policies.

**Your answer:**
```
To restrict access based on IP, MFA, time, or VPC.  
Example: In this lab, S3 access was allowed only from a specific IP.

---

## Summary of Resources Created

**IAM Policies:**
1. S3-ReadOnly-IP-Restricted  (ARN: arn:aws:iam::202145728564:policy/S3-ReadOnly-IP-Restricted))
2. EC2-StartStop-Only  (ARN: arn:aws:iam::202145728564:policy/EC2-StartStop-Only))
3. CloudWatch-Logs-Write-Only  (ARN: ARN: arn:aws:iam::202145728564:policy/CloudWatch-Logs-Write-Only))

**IAM Roles:**
1. EC2-S3-ReadOnly-Role  (ARN: ARN: arn:aws:iam::202145728564:role/EC2-S3-ReadOnly-Role)
2. Lambda-Basic-Execution-Role  (ARN: arn:aws:iam::202145728564:role/Lambda-Basic-Execution-Role))
3. CrossAccount-ReadOnly-Role  (ARN: arn:aws:iam::202145728564:role/CrossAccount-ReadOnly-Role))

**Users Modified:**
1. test-developer

---

## Cleanup Confirmation

- [ ok ] Detached all custom policies from users
- [ ok ] Deleted custom IAM policies
- [ ok ] Detached policies from roles
- [ ok ] Deleted test IAM roles
- [ ok ] Verified no resources remain

**Cleanup Commands:**
```bash
aws iam detach-user-policy --user-name test-developer --policy-arn arn:aws:iam::202145728564:policy/S3-ReadOnly-IP-Restricted
aws iam delete-policy --policy-arn arn:aws:iam::202145728564:policy/S3-ReadOnly-IP-Restricted

aws iam detach-role-policy --role-name EC2-S3-ReadOnly-Role --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
aws iam delete-role --role-name EC2-S3-ReadOnly-Role

aws iam detach-role-policy --role-name Lambda-Basic-Execution-Role --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
aws iam delete-role --role-name Lambda-Basic-Execution-Role

aws iam detach-role-policy --role-name CrossAccount-ReadOnly-Role --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
aws iam delete-role --role-name CrossAccount-ReadOnly-Role

---

## Self-Assessment

**Rate your understanding (1-5):**

Concept	Before Lab	After Lab	Improvement
IAM Policy Structure	3/5	5/5	+2
Custom Policy Creation	2/5	5/5	+3
IAM Roles	2/5	5/5	+3
Service Roles	2/5	5/5	+3
Trust Relationships	2/5	5/5	+3
Policy Testing	2/5	5/5	+3
Least Privilege	1/5	5/5	+4
Troubleshooting IAM	1/5	5/5	+4
---

## Instructor Verification

**Instructor Name:** ___________________________

**Date Reviewed:** ___________________________

**All policies validated:** ☑ Yes ☐ No

**Roles properly configured:** ☑ Yes ☐ No

**Comments:**
```
_____________________________________________________________
_____________________________________________________________
_____________________________________________________________
```

**Grade/Status:** ___________________________

---

**Lab Status:** ☐ Complete ☐ Needs Revision

**Submission Date:** 04.02.2026
