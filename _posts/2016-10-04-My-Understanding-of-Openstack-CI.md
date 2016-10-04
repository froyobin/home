---
layout: post
title: Understanding of Openstack CI
description: ""
category: study
tags: [study,Openstack,CI]
imagefeature:
comments: true
share: true
---


# **1. Introduction**

For most cases, we can use the test environment that already exits. But for some test cases that need the special test environment, we have to create it by ourselves. This artical mainly focuses on how to create the job in community CI system.

We will go through the [Openstack-infra/project-config](https://github.com/openstack-infra/project-config) the [Zuul](http://docs.openstack.org/infra/system-config/zuul.html) and the [devstack-gate](https://github.com/openstack-infra/devstack-gate) to see how these project works in CI system.





# **2. Openstack Infra/Project-config**
When we need to add a new test job to the community CI system, we need to modify this Projects. Now, lets go throught these names and configure files.

The CI system works like this, when we create/update a patch to the gerrit, gerrit will pushes out a notification to the event stream. This event stream has a number of subscribers, and one of them is called Zuul. Zuul is designed to manage the graphs of interdependent branch merge proposals in the upstream system. It will monitors in-progress jobs for a set of related patches. More than that, Zuul is responsible for constructing the pipelines of jobs that would be executed on various events. There are plenty of pipelines(gate,check,release-post .....) and you can find [pipelines](http://git.openstack.org/cgit/openstack-infra/project-config/tree/zuul/layout.yaml#n4) here. Zuul will read the pipeline configuration here and run the jobs in each pipeline.

##1.Pipeline

Let's take a look at one of the important pipe the **gate** for exmaple.


      - name: gate
    description: Changes that have been approved by core developers are enqueued in order in this pipeline, and if they pass tests in Jenkins, will be merged.
    success-message: Build succeeded (gate pipeline).
    failure-message: Build failed (gate pipeline).  For information on how to proceed, see http://docs.openstack.org/infra/manual/developers.html#automated-testing
    manager: DependentPipelineManager
    source: gerrit
    precedence: high
    require:
		 open: True
      current-patchset: True
      approval:
        - verified: [1, 2]
          username: jenkins
        - workflow: 1
    trigger:
      gerrit:
        - event: comment-added
          approval:
            - workflow: 1
        - event: comment-added
          approval:
            - verified: 1
          username: jenkins
    start:
      gerrit:
        verified: 0
    success:
      gerrit:
        verified: 2
        submit: true
    failure:
      gerrit:
        verified: -2
    window-floor: 20
    window-increase-factor: 2
    
From the section above, we can clearly see that this pipeline(name:gate) is triggered by the event of comment-added and sometime, we use
	          `comment: (?i)^(Patch Set [0-9]+:)?( [\w\\+-]*)*(\n\n)?\s*(recheck|reverify)`
to trigger the pipeline with special comments.
Once the appropriate pipeline is matched, Zuul executes that particular pipeline for the project that had a patch proposed. 

##2. jenkins' jobs

Now, we need to add jobs into the pipelines. Each job should belong to a special projects, so the logic is Openstack projects has some pipelines and each pipeline contains some jobs.

Here I paste part of the neutron configuration in [layout.yaml](http://git.openstack.org/cgit/openstack-infra/project-config/tree/zuul/layout.yaml#n11573). If you need to add a new test job for neutron, just add the name of the test under the special pipleline under which you want it to run. Please notice the template here, it helps people to put the common jobs together thus you do not need to write the same jobs for different Openstack projects.


      - name: openstack/neutron
    template:
      - name: merge-check
      - name: python-jobs
      - name: python34-jobs
      - name: python35-jobs
      - name: openstack-server-publish-jobs
      - name: openstack-server-release-jobs
      - name: periodic-liberty
      - name: periodic-mitaka
      - name: periodic-newton
      - name: periodic-jobs-with-oslo-master
      - name: periodic-jobs-with-neutron-lib-master
      - name: check-requirements
      - name: integrated-gate
      - name: translation-jobs
      - name: translation-jobs-mitaka
      - name: translation-jobs-newton
      - name: experimental-tripleo-jobs
      - name: release-notes-jobs
    check:
      - neutron-coverage-ubuntu-xenial
      - gate-neutron-dsvm-api-ubuntu-trusty
      - gate-neutron-dsvm-functional-ubuntu-trusty
      - gate-neutron-dsvm-fullstack-ubuntu-trusty
      - gate-rally-dsvm-neutron-neutron
      - gate-tempest-dsvm-neutron-dvr-ubuntu-trusty
      - gate-tempest-dsvm-neutron-dvr-ubuntu-xenial
      - gate-tempest-dsvm-neutron-identity-v3-only-full-ubuntu-xenial-nv
      - gate-tempest-dsvm-neutron-linuxbridge-ubuntu-trusty
      - gate-tempest-dsvm-neutron-linuxbridge-ubuntu-xenial
      - gate-neutron-lbaasv2-dsvm-minimal
      - gate-grenade-dsvm-neutron-multinode
      - gate-grenade-dsvm-neutron-dvr-multinode
      - gate-tempest-dsvm-neutron-multinode-full-ubuntu-trusty
      - gate-tempest-dsvm-neutron-dvr-multinode-full-ubuntu-trusty
      - gate-tempest-dsvm-neutron-multinode-full-ubuntu-xenial
      - gate-tempest-dsvm-neutron-dvr-multinode-full-ubuntu-xenial
      - gate-tempest-dsvm-ironic-ipa-wholedisk-agent_ssh-tinyipa-nv
    gate:
      - neutron-coverage-ubuntu-xenial
      - gate-neutron-dsvm-api-ubuntu-trusty
      - gate-tempest-dsvm-neutron-dvr-ubuntu-trusty
      - gate-tempest-dsvm-neutron-dvr-ubuntu-xenial
      - gate-tempest-dsvm-neutron-linuxbridge-ubuntu-trusty
      - gate-tempest-dsvm-neutron-linuxbridge-ubuntu-xenial
      - gate-grenade-dsvm-neutron-multinode
      - gate-grenade-dsvm-neutron-dvr-multinode
    post:
      - neutron-coverage-ubuntu-trusty
      - neutron-coverage-ubuntu-xenial
    experimental:
      - gate-neutron-dsvm-functional-py34-ubuntu-trusty
      - gate-neutron-dsvm-functional-ubuntu-xenial
      - gate-neutron-dsvm-functional-py35-ubuntu-xenial
      - gate-neutron-dsvm-fullstack-ubuntu-xenial
      - gate-tempest-dsvm-neutron-scenario
      - gate-tempest-dsvm-neutron-scenario-linuxbridge
      - gate-grenade-dsvm-neutron-forward
      - gate-neutron-vpnaas-dsvm-functional
      - gate-neutron-vpnaas-dsvm-functional-sswan
      - gate-tempest-dsvm-neutron-ipv6only
      - gate-tempest-dsvm-neutron-serviceipv6
      - gate-tempest-dsvm-neutron-ovs-native
      - gate-tempest-dsvm-neutron-dvr-ovs-native
      - gate-neutron-dsvm-api-ubuntu-xenial
      - gate-neutron-dsvm-api-pecan-ubuntu-trusty
      - gate-neutron-dsvm-api-pecan-ubuntu-xenial
      - gate-tempest-dsvm-neutron-pecan
      - gate-tempest-dsvm-neutron-src-neutron-lib
      - gate-grenade-dsvm-neutron-linuxbridge-multinode-nv
      - gate-tempest-dsvm-neutron-pg-full-ubuntu-trusty
      - gate-tempest-dsvm-neutron-pg-full-ubuntu-xenial

##3. Jenkins Job Builder
As more and more jobs added to the jenkins, it would be virtually impossible to manage the configuration of so many jobs using human-based processes. To solve this dilemma, the Jenkins Job Builder (**JJB**) python tool was created. JJB consumes YAML files that describe both individual Jenkins jobs as well as templates for parameterized Jenkins jobs, and writes the config.xml files for all Jenkins jobs that are produced from those templates. **Important: Note that Zuul does not construct Jenkins jobs. JJB does that.** Zuul simply configures which Jenkins jobs should run for a project and a pipeline. 
    
There is a master [projects.yaml](http://git.openstack.org/cgit/openstack-infra/project-config/tree/jenkins/jobs/projects.yaml#n7437) file in the same directory that lists the “top-level” definitions of jobs for all projects, and it is in this file that many of the variables that are used in job template instantiation are defined (including the {name} variable, which corresponds to the name of the project. 

When JJB build the jobs, it reads the projects.yaml. let's skip the first jobs like python-jobs or coverage-jobs. If we take a look at the job like *grenade-dsvm-neutron-multinode* we can see that this job request the node configuration of **ubuntu-trusty-2-node** from node pool. It is quite simple to understand the configurations here.       

   
     - project:
	    name: neutron
	    tarball-site: tarballs.openstack.org
	    doc-publisher-site: docs.openstack.org

    jobs:
      - coverage-jobs
      - python-jobs
      - cross-python-jobs
      - 'gate-{name}-python35{suffix}':
          suffix: ''
      - python-liberty-bitrot-jobs
      - python-mitaka-bitrot-jobs
      - python-newton-bitrot-jobs
      - openstack-publish-jobs
      - openstack-releasenotes-jobs
      - openstack-server-release-jobs
      - translation-jobs
      - translation-jobs-mitaka
      - translation-jobs-newton
      - gate-rally-dsvm-neutron-{name}
      - '{pipeline}-grenade-dsvm-neutron-multinode{job-suffix}':
          pipeline: gate
          node: ubuntu-trusty-2-node
          job-suffix: ''
          branch-override: default
      - '{pipeline}-grenade-dsvm-neutron-dvr-multinode{job-suffix}':
          pipeline: gate
          node: ubuntu-trusty-2-node
          job-suffix: ''
          branch-override: default
          
          **********skip************

Now, we take the [python-jobs](http://git.openstack.org/cgit/openstack-infra/project-config/tree/jenkins/jobs/python-jobs.yaml#n625) as an example to show how the jobs are grouped together. It is obvious that this job contains 6 different jobs, as these jobs are general jobs for all the projects so they are grouped together. The {name} prefix will be replace by the project name that starts this job.

	-job-group:
    name: python-jobs
    node:
      - ubuntu-trusty
      - ubuntu-xenial
    jobs:
      - 'gate-{name}-pep8-{node}'
      - 'gate-{name}-python27-{node}'
      - 'gate-{name}-python34'
      - 'gate-{name}-docs-{node}'
      - 'gate-{name}-requirements'
      - '{name}-branch-tarball'
      # pylint isn't standard
      # pypy isn't standard
      # gate-{name}-tox-{envlist} also isn't standard, but is reserved for
      # projects that want to run specific jobs via tox

Let's take a look at [gate-{name}-pep8-{node}](http://git.openstack.org/cgit/openstack-infra/project-config/tree/jenkins/jobs/python-jobs.yaml#n68)

The most import section is builders, it instructs how the JJB runs the tests.

	- job-template:
    name: 'gate-{name}-pep8-{node}'

    builders:
      - print-template-name:
          template-name: "{template-name}"
      - zuul-git-prep-upper-constraints
      - install-distro-packages
      - revoke-sudo
      - pep8:
          env: pep8

    publishers:
      - test-results
      - console-log

    node: '{node}'


Let's read the **print-template-name**, it is in [macros.yaml](http://git.openstack.org/cgit/openstack-infra/project-config/tree/jenkins/jobs/macros.yaml#n8). It is quite simple, it just print the name of the template, the varialbe {template-name} is passed by the maping "template-name: "{template-name}""

	- builder:
    name: print-template-name
    builders:
      - shell: 'echo JJB template: {template-name}'



##3. Devtack Gate
The real world is not always so simple. We often want to run the test environment in a real Openstack Environment. This work is done by [devstack-gate](http://docs.openstack.org/infra/system-config/devstack-gate.html). 

To show how it works, I pasted the paragrah from [devstack-gate document](http://docs.openstack.org/infra/system-config/devstack-gate.html) here. 
 
When a proposed change is approved by the core reviewers, Jenkins triggers the devstack gate test itself. This job runs on one of the previously configured nodes and invokes the devstack-vm-gate-wrap.sh script which checks out code from all of the involved repositories, and merges the proposed change. That script then calls devstack-vm-gate.sh which installs a devstack configuration file, and invokes devstack. Once devstack is finished, it runs exercise.sh and Tempest, which perform integration testing. After everything is done, devstack-gate copies and formats all of the logs for archival. A jenkins jobs then copies these logs to the log archive.


There are two importnat concepts.


* **Nodepool**— Provides virtual machine instances to Jenkins masters for running complex, isolation-sensitive Jenkins jobs. We just need to pick up the suitable node we need to run the tests from the nodepool. The configration of different node can be found [here](http://git.openstack.org/cgit/openstack-infra/project-config/tree/nodepool/nodepool.yaml). The node will be configured to be ready to run the suitalbe tests. It is controleed by the scripts like [prepare_node_devstack.sh](http://git.openstack.org/cgit/openstack-infra/project-config/tree/nodepool/scripts/prepare_node_devstack.sh)(it will call [prepare_node.sh](http://git.openstack.org/cgit/openstack-infra/project-config/tree/nodepool/scripts/prepare_node.sh) firstly.).

* **Devstack-Gate**— Scripts that create an OpenStack environment with Devstack, run tests against that environment, and archive logs and results


The devstack-gate project is [here](https://github.com/openstack-infra/devstack-gate), you can run it locally if you have a openstack environment. When we have got the suitalbe node ready, we need to set up the devstack by running [devstack-vm-gate-wrap.sh](http://git.openstack.org/cgit/openstack-infra/devstack-gate/tree/devstack-vm-gate-wrap.sh). Let's take a look at [devstack-gate.yaml](http://git.openstack.org/cgit/openstack-infra/project-config/tree/jenkins/jobs/devstack-gate.yaml#n1076) from project-config. At the end of this file, it runs the scripts **safe-devstack-vm-gate-wrap.sh**

	- job-template:
    name: '{pipeline}-grenade-dsvm-neutron-dvr-multinode{job-suffix}'
    node: '{node}'

    wrappers:
      - build-timeout:
          timeout: 125
      - timestamps

    builders:
      - link-logs
      - net-info
      - devstack-checkout
      - shell: |
          #!/bin/bash -xe
          export PYTHONUNBUFFERED=true
          export DEVSTACK_GATE_NEUTRON=1
          export DEVSTACK_GATE_CONFIGDRIVE=0
          export DEVSTACK_GATE_GRENADE=pullup
          # Test DVR upgrade on multinode
          export PROJECTS="openstack-dev/grenade $PROJECTS"
          export DEVSTACK_GATE_NEUTRON_DVR=1
          export BRANCH_OVERRIDE={branch-override}
          if [ "$BRANCH_OVERRIDE" != "default" ] ; then
              export OVERRIDE_ZUUL_BRANCH=$BRANCH_OVERRIDE
          fi
          export DEVSTACK_GATE_TOPOLOGY="multinode"

          cp devstack-gate/devstack-vm-gate-wrap.sh ./safe-devstack-vm-gate-wrap.sh
          ./safe-devstack-vm-gate-wrap.sh

	    publishers:
   		   	- devstack-logs
   	   		- console-log


##4. destack-vm-gate.sh

The [devstack-vm-gate](http://git.openstack.org/cgit/openstack-infra/devstack-gate/tree/devstack-vm-gate.sh) will configure the node and finally run the devstack project.


