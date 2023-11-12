import groovy.json.JsonSlurper

properties([
    durabilityHint('PERFORMANCE_OPTIMIZED'),
    disableResume(),
    [$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator',  daysToKeepStr: '90', numToKeepStr: '50']],
    parameters([
        string(
            defaultValue: 'https://github.com/kumvijaya/pr-merge-demo/pull/1',
            name: 'prUrl',
            trim: true,
            description: 'Provide GitHub pull request URL: (Example: https://github.com/kumvijaya/pr-merge-demo/pull/1).'
        )
    ])
])

node(getEnvValue('PR_MERGE_SLAVE_AGENT_LABEL', '')) {
    def prInfo = processMerge(params.prUrl)
    echo "prInfo = ${prInfo}"
    if(prInfo.proceed_cicd) {
        stage('Checkout') {
            // Check out your source code repository as per pr target branch
            git branch: prInfo.targetBranch, url: prInfo.repoUrl
        }
        stage('Build') {
            // Build your application
            powershell 'npm install'
        }
        stage('Test') {
            // Run tests
            powershell 'npm test'
        }
        stage('Deploy') {
            // Deploy your application to a target environment
            powershell 'npm pack'
            appendPackageWithPRNumber(prInfo.pr_number)
        }
        // Additional stages or post-build actions can be added here
    }
}

def processMerge(prUrl) {
    def prInfo = [:]
    stage('Merge PR') {
        dir('merge') {
            git url: 'https://github.com/kumvijaya/pr-merge-demo.git', branch: 'feature/python-merge'
            withCredentials([usernamePassword(credentialsId: 'GITHUB_USER_PASS', usernameVariable: 'GITHUB_USER', passwordVariable: 'GITHUB_PASSWORD')]) {
                sh "python -m pip install requests==2.26.0 --user"
                sh "python git-merger.py -p ${params.prUrl}"
                prInfo = readJSON file: 'git_merge_ouput.json'
            }
            deleteDir()
        }
    }
    return prInfo
}

/**
* Gets the env variable value
*/
private String getEnvValue(String envKey, String defaultValue='') {
	String envValue = defaultValue
	def envMap = env.getEnvironment()
	envMap.each{ key, value ->
		if (key == envKey) {
			envValue = !isEmpty(value) ? value : defaultValue
			return envValue
		}
	}
	return envValue
}

/**
* Is given string empty
*/
private boolean isEmpty(String input) {
    return !input?.trim()
}

/**
* Appends package name with PR number
*/
private appendPackageWithPRNumber(prNumber) {
    String name = powershell(returnStdout: true, script:'npm pkg get name | xargs echo').trim()
    String version = powershell(returnStdout: true, script:'npm pkg get version | xargs echo').trim()
    String packageName = "${name}-${version}"
    String fileName = "${packageName}.tgz"
    String newFileName = "${packageName}_pr_${prNumber}.tgz"
    echo "Updating packageName (${fileName}) to ${newFileName}"
    sh "mv ${fileName} ${newFileName}"
    sh "ls -ltr"
}


// /**
// * Process PR merge.
// */
// private def processPRMerge(prUrl) {
//     def prInfo = [:]
//     stage('Merge Pull Request') {
//         prInfo['org'] = getOrg(prUrl)
//         prInfo['repo'] = getRepo(prUrl)
//         prInfo['number'] = getPRNumber(prUrl)
//         prInfo['repoUrl'] = getRepoUrl(prInfo)
//         prInfo['apiUrl'] = getPRApiUrl(prInfo)
//         populatePRInfo(prInfo)
//         echo "Received Pull Request Info ${prInfo}"
//         if(prInfo.merged) {
//             echo "Pull request found already merged, proceeding CICD on the merged branch."
//             prInfo['proceedCICD'] = true
//         }else {
//             validatePR(prInfo)
//             echo "Pull request found valid for merging, proceeding with merge"
//             def mergeResp = mergePR(prInfo)
//             echo "Pull request merged: ${mergeResp}"
//             if(mergeResp.containsKey('merged') && mergeResp.merged) {
//                 echo "Pull request merged successfully. proceeding with CICD on the merged branch"
//                 prInfo['proceedCICD'] = true
//             }else {
// 		prInfo['proceedCICD'] = false
//                 error "Pull request not merged. Please check the PR, Not proceeding with CICD"
//             }
//         }        
//     }
//     return prInfo
// }

// /**
// * Validate pull request.
// */
// private void validatePR(prInfo) {
//     if(prInfo.state != 'open') {
//         error "The given pull request state is in state: ${prInfo.state}. It should be in open state for merging"
//     }

//     int requiredApprovalcount = Integer.parseInt(getEnvValue('PR_MERGE_APPROVAL_COUNT', '2'))
//     if(prInfo.approvalCount < requiredApprovalcount) {
//         error "No of approvals found in for the given pull request as ${prInfo.approvalCount}. The required number of approvers: (${requiredApprovalcount})"
//     }
// }

// /**
// * Gets the PR Info for the given PR url.
// */
// private void populatePRInfo(prInfo) {
//     def pr = getRequest(prInfo.apiUrl)
//     prInfo['sourceBranch'] = pr.head.ref
//     prInfo['targetBranch'] = pr.base.ref
//     prInfo['state'] = pr.state
//     prInfo['merged'] = pr.merged
//     prInfo['mergeable'] = pr.mergeable
//     prInfo['mergeable_state'] = pr.mergeable_state
//     prInfo['approvalCount'] = getApprovalCount(prInfo)
//     prInfo['statusChecksSucceeded'] = statusCheckSuccessful(pr.statuses_url)
// }

// /**
// * Gets PR API Url
// */
// private String getPRApiUrl(prInfo)  {
//     return "https://api.github.com/repos/${prInfo.org}/${prInfo.repo}/pulls/${prInfo.number}"
// }

// /**
// * Gets PR API Url
// */
// private String getRepoUrl(prInfo)  {
//     return "https://github.com/${prInfo.org}/${prInfo.repo}.git"
// }

// /**
// * Gets PR Approval count
// */
// private int getApprovalCount(prInfo) {
//     String prReviewsUrl = "${prInfo.apiUrl}/reviews"
//     def reviews = getRequest(prReviewsUrl)
//     def approvedReviews = reviews.findAll { it.state == "APPROVED" }
//     return approvedReviews.size()
// }

// /**
// * Checks PR status check successful as per the given status desc
// */
// private boolean statusCheckSuccessful(String prStatusesUrl) {
//     def statuses = getRequest(prStatusesUrl)
//     String statusLables = getEnvValue('PR_MERGE_STATUS_LABELS', '')
//     def statusLableList = statusLables.split(',')
//     boolean allSucceeded = true
//     for (label in statusLableList) {
//         def status = statuses.find { it.context == label}
//         if(status != null && status.state != 'success') {
//             allSucceeded = false
//             break
//         }
//     }
//     return allSucceeded
// }

// /**
// * Invokes the get request 
// */
// private def getRequest(String requestUrl) {
//     def response = httpRequest authentication: 'GITHUB_USER_PASS', httpMode: 'GET',
//             validResponseCodes: '200',
//             url: requestUrl
//     def responseJson = new JsonSlurper().parseText(response.content)
//     return responseJson
// }

// /**
// * Invokes the post request 
// */
// private def putRequest(requestUrl, requestBody) {
//     def response = httpRequest authentication: 'GITHUB_USER_PASS',
//             acceptType: 'APPLICATION_JSON', 
//             contentType: 'APPLICATION_JSON',
//             httpMode: 'PUT',
//             requestBody: requestBody,
//             validResponseCodes: '200,201,204',
//             url: requestUrl
//     def responseJson = new JsonSlurper().parseText(response.content)
//     return responseJson
// }

// /**
// * Gets PR number from github pr url 
// */
// private String getPRNumber(String prUrl) {
//     return prUrl.tokenize('/').last()
// }

// /**
// * Gets the org name property from github pr url 
// */
// private String getOrg(String prUrl) {
//     String urlPart = prUrl.replace("https://github.com/", '')
//     return urlPart.tokenize('/').first()
// }

// /**
// * Gets the repo name property from github pr url 
// */
// private String getRepo(String prUrl) {
//     String urlPart = prUrl.replace("https://github.com/", '')
//     return urlPart.tokenize('/')[1]
// }

// /**
// * Gets the PR Info for the given PR url.
// */
// private def mergePR(prInfo) {
//     String mergeReqUrl = "${prInfo.apiUrl}/merge"
//     String mergeBody = getMergeBody()
//     return putRequest(mergeReqUrl, mergeBody)
// }

// /**
// * Gets the PR merge body json.
// */
// private String getMergeBody() {
//     String mergeTitle = 'PR merged by PR Utility (Jenkins)'
//     String mergeDesc = 'PR merged by PR Utility (Jenkins) with required checks'
//     return "{\"commit_title\":\"${mergeTitle}\",\"commit_message\":\"${mergeDesc}\"}"
// }





