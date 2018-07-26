def version, mvnCmd = "mvn -s src/main/config/settings.xml"

pipeline {
    agent {
        label 'maven'
    }
    stages {
        stage('Build App') {
        	steps {
            	git url: "https://github.com/N3G5/cart-service.git"
            	script {
					sh "${mvnCmd} clean package"
            	}
        	}
  		}
  		stage('Integration Test') {
  			steps {
  				script {
	    			sh "${mvnCmd} verify"
  				}
  			}
    	}
    	stage('Code Analysis') {
      		steps {
        		script {
          			sh "${mvnCmd} sonar:sonar -Dsonar.host.url=http://sonarqube:9000 -DskipTests=true"
        		}
      		}
    	}
    	stage('Archive App') {
      		steps {
        		sh "${mvnCmd} deploy -DskipTests=true -P nexus3"
      		}
    	}
    	stage('Build Image') {
    		steps {
	    		script {
	    		    openshift.withCluster() {
	    		      openshift.withProject("cartexample") {
	    		          openshift.selector("bc", "cart").startBuild("--from-file=target/cart.jar --follow")
	    		      }
	    		   }
	    		}
    		}
		}
		stage('Deploy App') {
      		steps {
        		script {
          			openshift.withCluster() {
            			openshift.withProject("cartexample") { 
            				openshift.secelctor("dc", "cart").rollout().latest()
            			}
          			}
        		}
      		}
  		}
  		stage('Component Test') {
  			steps {
  			    script {
		    		sh "curl -s -X POST http://cart.dev.svc.cluster.local:8080/api/cart/dummy/666/1"
					sh "curl -s http://cart.dev.svc.cluster.local:8080/api/cart/dummy | grep 'Dummy Product'"
  			    }
  			}
  		}
  	}
}
