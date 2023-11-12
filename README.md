# Pull Request Pipeline

This is to check and merge the given pull request and perform the CICD on the merged branch.
Below checks are done as before merging the PR
- PR is approved with the required number of approvers
- PR passes required PR checks (Future)
- PR is in a mergeable state (doesn't have any conflicts)


If the given PR is already merged, skip the merging and proceed with CICD by cloning the merged branch.
If the given PR is not merged and passes all conditions, merge the PR and proceed with CICD by cloning the merged branch.

Also, it adds the PR number to the deployment package name. Format: ${PACKAGE_NAME}_pr_${PR_NUMBER}

## Required Jenkins Plugins
1. [Pipeline Utility Steps](https://plugins.jenkins.io/pipeline-utility-steps/)
2. [HTTP Request](https://plugins.jenkins.io/http_request/)

## Environment Variables
Below environment variables below can be provided for the job run.
- *PR_MERGE_SLAVE_AGENT_LABEL*: The slave agent label to use (Ex: MacSTANDALONE). Default none.
- *PR_MERGE_APPROVAL_COUNT*: Required number of approvals (Ex: 2). Default 0.
- *PR_MERGE_STATUS_LABELS*: Required statuses to check. Provide as comma separated if more than one. This is for the future (Ex: unit-test,code-lint). Default none.

![jenkins-env-vars](https://github.com/kumvijaya/pr-merge-demo/blob/develop/images/env-vars.png)

## Required Credentials
This expects below GitHub credentials created in Jenkins in the Credentials store.
- *Type*: UsernamePassword
- *Username*: Github user-id
- *Password*: Github PAT (Personal Access Token, This should have read/write access to repo)
- *ID*: GITHUB_USER_PASS

![jenkins-github-creds](https://github.com/kumvijaya/pr-merge-demo/blob/develop/images/github-creds.png)

# Required Job Parameters
This expects job parameter *prUrl*

![jenkins-job-params](https://github.com/kumvijaya/pr-merge-demo/blob/develop/images/job-params.png)

Provide the valid PR URL here (Ex: https://github.com/kumvijaya/pr-merge-demo/pull/1)


# Future Thoughts

Merger utility can be kept as a Shared Jenkins pipeline library so that all the repos can leverage this without code duplication. 

Refer [this](https://www.jenkins.io/doc/book/pipeline/shared-libraries/) link for more details on the Jenkins Shared Pipeline library



