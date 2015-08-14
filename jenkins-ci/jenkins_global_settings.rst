.. _jenkins_global_settings:

Global Settings
=================

  .. note::
  
      HOME: /var/jenkins_home

      Workspace Root Directory: ${JENKINS_HOME}/workspace/${ITEM_FULLNAME}

      Build Record Root Directory: ${ITEM_ROOTDIR}/builds


  #. Docker Builder
   
     Docker URL: docker server REST API URL, http://172.17.42.1:2375 if jenkins is running in a docker container.
  

  #. Gerrit Trigger

     Hostname, Frontend URL, SSH Port, Username, SSH Keyfile(/var/jenkins_home/.ssh/id_rsa), SSH Keyfile Password

     Advanced::

        REST API -  HTTP Username && HTTP Password

        Enable code review

        Enable Verified


Integrate With Gerrit
======================

Concepts::

  refs/for/<branch name>
  refs/changes/nn/<change-id>/m
  git hook  commit-msg  Change-Id

  command line: ssh -P -p 29418 jenkins@127.0.0.1 gerrit
  get git hook: scp -P 29418 jenkins@127.0.0.1:/hooks/commit-msg ./


Gerrit settings
----------------

* User settings

  #. Add user jenkins on Gerrit Code review

       Email Address == git config user.email
  
  #. Add SSH Public Keys

  #. Generate HTTP Password for HTTP REST API

  #. Add jenkins to groups: Anonymous Users, Event Streaming Users, Non-Interactive Users

* Project settings

  #. Create a new project. Rights inherit from **All-Projects**

  #. Require Change-Id in commit message **TRUE** or **FALSE** if necessary

* Project Access settings

  #. Global Capabilities
     
     Stream Events: ALLOW - Event Streaming Users

     Stream Events: ALLOW - Non-Interactive Users

  #. Reference: refs/*

     Read: ALLOW - Non-Interactive Users

     Label Verified: +1/-1 - Non-Interactive Users

  #. Reference: refs/heads/*

     Push: ALLOW - Non-Interactive Users

     Label Code-Review: +1/-1 - Non-Interactive Users

     Label Verified: +1/-1 - Non-Interactive Users

     * Add ``Label Verified`` to All-Project

     ::      

         git init project
         git config user.name 'admin'
         git config user.email 'admin@example.com'
         git remote add origin ssh://admin@10.13.182.124:29418/All-Project
         git pull origin refs/meta/config
         cat << EOF >> project.config
         [label "Verified"]
         function = MaxWithBlock
         value = -1 Fails
         value =  0 No score
         value = +1 Verified
         EOF
         git add project.config
         git commit -a -m 'add Label Verified to All-Project'
         git push origin HEAD:refs/meta/config


Jenkins Job
-------------

Auto Verify
`````````````

  #. Source Code Management

       git repository: ssh://jenkins@10.13.182.124:29418/esTookit

       Credentials: passwords/ssh key

       Name: gerrit
 
       Refspec: $GERRIT_REFSPEC
  
       Branches to build: $GERRIT_BRANCH

       Additional Behaviours

          * Strategy for choosing what to build - Gerrit Trigger
          * Clean before checkout

  #. Build Triggers  - Gerrit event

       Choose a server
       Trigger on

          * Patchset Created
          * Draft Published

       Gerrit Project

          * Type - Plain
          * Pattern - esTookit
          * Branches 

              + Type - Path
              + Pattern - **

  #. Build

        Use ``Execute shell`` to load extern scripts. 
   
        Perform build & test
 
  #. Project will be automately checked with ``Label Verified`` if build succeed. So, no Post-build actions to perform.

Auto Publish
`````````````

  #. Gerrit Trigger
    
       Trigger on - Change Merged

  #. Publish to Cloud Foundry
   
       Post-build Actions
         Target, Credentials, Space, Allow self-signed certificate 
         Reset app if already exists
         Read configuration from a manifest file / Enter configuration in Jenkins

Tips && Suggestions
----------------------

  #. Parameterize Jenkins jobs

  #. Destruct complex Jenkins jobs
