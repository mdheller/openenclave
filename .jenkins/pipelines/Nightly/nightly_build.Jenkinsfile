// Copyright (c) Open Enclave SDK contributors.
// Licensed under the MIT License.

@Library("OpenEnclaveCommon") _
oe = new jenkins.common.Openenclave()

GLOBAL_TIMEOUT_MINUTES = 240
CTEST_TIMEOUT_SECONDS = 480
GLOBAL_ERROR = null

DOCKER_TAG = env.DOCKER_TAG ?: "latest"
AGENTS_LABELS = [
    "acc-ubuntu-16.04": env.UBUNTU_1604_CUSTOM_LABEL ?: "ACC-1604",
    "acc-ubuntu-18.04": env.UBUNTU_1804_CUSTOM_LABEL ?: "ACC-1804",
    "ubuntu-nonsgx":    env.UBUNTU_NONSGX_CUSTOM_LABEL ?: "nonSGX",
    "acc-rhel-8":       env.RHEL_8_CUSTOM_LABEL ?: "ACC-RHEL-8",
    "acc-win2016":      env.WINDOWS_2016_CUSTOM_LABEL ?: "SGXFLC-Windows",
    "acc-win2016-dcap": env.WINDOWS_2016_DCAP_CUSTOM_LABEL ?: "SGXFLC-Windows-DCAP",
    "windows-nonsgx":   env.WINDOWS_NONSGX_CUSTOM_LABEL ?: "nonSGX-Windows"
]

properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '90',
                                      artifactNumToKeepStr: '180',
                                      daysToKeepStr: '90',
                                      numToKeepStr: '180')),
            [$class: 'JobRestrictionProperty']])

try {
    oe.emailJobStatus('STARTED')
    parallel (
      "Agnostic Linux" : {
          build job: '/pipelines/Agnostic-Linux',
                parameters: [string(name: 'REPOSITORY_NAME', value: env.REPOSITORY_NAME),
                             string(name: 'BRANCH_NAME', value: env.BRANCH_NAME),
                             string(name: 'DOCKER_TAG', value: DOCKER_TAG),
                             string(name: 'UBUNTU_NONSGX_CUSTOM_LABEL', value: AGENTS_LABELS["ubuntu-nonsgx"])]
       },
       "Azure Linux" : {
           build job: '/pipelines/Azure-Linux',
                 parameters: [string(name: 'REPOSITORY_NAME', value: env.REPOSITORY_NAME),
                              string(name: 'BRANCH_NAME', value: env.BRANCH_NAME),
                              string(name: 'DOCKER_TAG', value: DOCKER_TAG),
                              string(name: 'UBUNTU_1604_CUSTOM_LABEL', value: AGENTS_LABELS["acc-ubuntu-16.04"]),
                              string(name: 'UBUNTU_1804_CUSTOM_LABEL', value: AGENTS_LABELS["acc-ubuntu-18.04"]),
                              string(name: 'UBUNTU_NONSGX_CUSTOM_LABEL', value: AGENTS_LABELS["ubuntu-nonsgx"]),
                              string(name: 'RHEL_8_CUSTOM_LABEL', value: AGENTS_LABELS["acc-rhel-8"]),
                              string(name: 'WINDOWS_NONSGX_CUSTOM_LABEL', value: AGENTS_LABELS["windows-nonsgx"])]
       },
       "Azure Windows" : {
           build job: '/pipelines/Azure-Windows',
                 parameters: [string(name: 'REPOSITORY_NAME', value: env.REPOSITORY_NAME),
                              string(name: 'BRANCH_NAME', value: env.BRANCH_NAME),
                              string(name: 'DOCKER_TAG', value: DOCKER_TAG),
                              string(name: 'UBUNTU_NONSGX_CUSTOM_LABEL', value: AGENTS_LABELS["ubuntu-nonsgx"]),
                              string(name: 'WINDOWS_2016_CUSTOM_LABEL', value: AGENTS_LABELS["acc-win2016"]),
                              string(name: 'WINDOWS_2016_DCAP_CUSTOM_LABEL', value: AGENTS_LABELS["acc-win2016-dcap"])]
       }
    )
} catch(Exception e) {
    println "Caught global pipeline exception :" + e
    GLOBAL_ERROR = e
    throw e
} finally {
    currentBuild.result = (GLOBAL_ERROR != null) ? 'FAILURE' : "SUCCESS"
    oe.emailJobStatus(currentBuild.result)
}
