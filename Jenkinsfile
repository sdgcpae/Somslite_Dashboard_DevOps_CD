def propfile

pipeline {
	agent {
    		kubernetes {
			label 'SpringBootRestApp'
			defaultContainer 'jnlp'
			yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  containers:
  - name: gradle
    image: gradle:3.5-jdk8-alpine
    command:
    - cat
    tty: true
"""
		}
	}
	stages {
		stage('Build & Unit Test') {
			steps {
				container('gradle') {
					script {
						withMaven(maven: 'MAVEN-3.6.3') {
               echo 'Building'
		propfile = readProperties(file: './project.properties')
		echo  propfile['war_file']				
                cd SomsLite
               // gradle --no-daemon somslitebuild
                pwd
                cd ..
                echo 'copy property file'
                cp -r SterlingESBAux/properties/dev1 properties/
                cp -r SterlingESBAux/properties/uat1 properties/
                cp -r SterlingESBAux/properties/perf1 properties/
                cp -r SterlingESBAux/properties/perf2 properties/
                cp -r SterlingESBAux/properties/prod1 properties/
                cp -r SterlingESBAux/properties/prod2 properties/

                ls -la properties
                rm -rf $WORKSPACE/artifacts
                mkdir -p $WORKSPACE/artifacts
                for i in SomsLite;do cp -rp ${i}/dist/libs/* $WORKSPACE/artifacts/ ;done
                cd $WORKSPACE/artifacts/

                ls -lrt
                cd ..

                cp -rp properties $WORKSPACE/artifacts/
                cd artifacts
                ls -lrt
                }
              }
            }
          }
        }
        stage("Deploy") {
	       when { expression {env.GIT_BRANCH == 'dev' || env.GIT_BRANCH == 'release'|| propfile['feature_deploy'] == "true" }}
		     steps {
				container('gradle') {
					script {
          withMaven(maven: 'MAVEN-3.6.3') {
							propfile = readProperties(file: './project.properties')
							if (propfile['feature_deploy'] == "true" ) {
									USERNAME=propfile['USERNAME_FEATURE_DEPLOY']
									HOSTS=propfile['HOSTS_FEATURE_DEPLOY']
								}
								if (env.GIT_BRANCH == 'dev' ) {
									USERNAME=propfile['USERNAME_DEV_DEPLOY']
									HOSTS=propfile['HOSTS_DEV_DEPLOY']
								}
								if (env.GIT_BRANCH == 'release') {
									USERNAME=propfile['USERNAME_RELEASE_DEPLOY']
									HOSTS=propfile['HOSTS_RELEASE_DEPLOY']
								}
								HOSTS.tokenize(',').each { HOSTNAME ->
                
                cat << EOF > propfile['deployment_cli_file']
              if (outcome != success) of /deployment=propfile['war_file']:read-resource
              echo ###
              echo ### Deploy propfile['war_file'] to somslite-cluster
              echo ###
              deploy propfile['war_deploy_dir']/propfile['war_file'] --server-groups=propfile['server_groups']
              else
              echo ###
              echo ### Module propfile['war_file'] already installed, deploying with rolling update
              echo ###
              deploy propfile['war_deploy_dir']/propfile['war_file'] --name=propfile['war_file'] --runtime-name=propfile['war_file'] --headers={rollout propfile['server_groups'](rolling-to-servers=true)} --force
              end-if

              echo
              echo ###
              echo ### Restarting servers
              echo ###

              echo
              echo ### Restarting somslite-server01
              if (outcome != success) of /host=node1/server-config=somslite-server01:restart
              echo ### Server somslite-server01 restart failed
              echo
              else
              echo ### Server somslite-server01 restarted
              echo
              end-if

              EOF
              echo 'coping property file into server'
        mkdir -p $WORKSPACE/artifacts
        cp -R /C/windows/system32/config/systemprofile/AppData/Local/Jenkins/.jenkins/workspace/DevOps_SomsLite/DevOps_SomsLite_release-2021_3.2/artifacts/* $WORKSPACE/artifacts/
        cd $WORKSPACE/artifacts/properties/
        mv dev_app_1_config-mule.properties config-mule.properties
        mv dev_app_1_log4j.properties log4j.properties
        mv dev_app_1_config.properties config.properties
        scp -i /c/Users/ernesto.alvarez/.ssh/id_rsa_jenkins -o StrictHostKeyChecking=no -rp $WORKSPACE/artifacts/propfile['war_file'] $target_user@997725-DEV-EPIC-APP-01.signetomni.com:propfile['war_deploy_dir']
        scp -i /c/Users/ernesto.alvarez/.ssh/id_rsa_jenkins -o StrictHostKeyChecking=no -rp $WORKSPACE/artifacts/properties/config-mule.properties $target_user@997725-DEV-EPIC-APP-01.signetomni.com:propfile['property1_deploy_dir']
        scp -i /c/Users/ernesto.alvarez/.ssh/id_rsa_jenkins -o StrictHostKeyChecking=no -rp $WORKSPACE/artifacts/properties/log4j.properties $target_user@997725-DEV-EPIC-APP-01.signetomni.com:propfile['property1_deploy_dir']
        scp -i /c/Users/ernesto.alvarez/.ssh/id_rsa_jenkins -o StrictHostKeyChecking=no -rp $WORKSPACE/artifacts/properties/config.properties $target_user@997725-DEV-EPIC-APP-01.signetomni.com:propfile['property2_deploy_dir']
        scp -i /c/Users/ernesto.alvarez/.ssh/id_rsa_jenkins -o StrictHostKeyChecking=no -rp $WORKSPACE/propfile['deployment_cli_file'] propfile['target_user']@997725-DEV-EPIC-APP-01.signetomni.com:propfile['war_deploy_dir']
        echo' deploying'
        ssh -i /c/Users/ernesto.alvarez/.ssh/id_rsa_jenkins -o StrictHostKeyChecking=no propfile['target_user']@997725-DEV-EPIC-APP-01.signetomni.com "propfile['cli_script'] --connect --controller=997725-DEV-EPIC-APP-01.signetomni.com:propfile['target_port'] --timeout=20000 --file=propfile['war_deploy_dir']/propfile['deployment_cli_file'] --user=propfile['jbosscli_offline_user'] --password='98&OpUw2ZgBTE*R2'"
			}
		}
	}
   }
 }
}
}
