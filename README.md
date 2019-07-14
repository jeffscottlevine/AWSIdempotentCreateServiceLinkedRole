# AWSIdempotentCreateServiceLinkedRole

## Overview

This file is an AWS CloudFormation template that is used to provide a CloudFormation custom resource that creates
a service-linked role idempotently.  This means:

(1) If the role doesn't exist, create the role.  If the role is successfully created, return SUCCESS otherwise return FAILED.
(2) If the role already exists, return SUCCESS.

## Background

Service-linked roles provide the IAM privleges needed by an AWS service.  In some cases, that role name is fixed.
For example, the name of the service-linked role for Amazon Inspector is AWSServiceRoleForAmazonInspector.
Since role names must be unique, you cannot run a CloudFormation template that creates a service-linked role
in account multiple times.
Some (but not all) service-linked roles support the notion of a suffix, which allows you to add a suffix
(perhaps a random value) to the name to make it easier to create the role in CloudFormation.
Other services (such as Inspector) do not currently support suffixes, hence the need for a custom resource
to support idempotency.

## Notes

1. The function currently only supports Inspector.  You can add more services to the Python dictionary.

2. The function does not support updates or deletes because the role that might have been created may already
be used by another application.

3. This function is not atomic.  It first calls the get_role API.  If the role does not exist, the function will
attempt to create it.  There is a window of time between the calling of the get_role API and the create_service_linked_role API.
If the Lambda function is called multiple times within the same account in the same region, the function could fail.
