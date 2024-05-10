pipeline {
    agent any
    stages {
        stage('Pull from github') {
            steps {
                git branch: 'main', url: 'https://github.com/shalom2012/ecommerce-django-react.git'
            }
        }

        stage('Build') {
            steps {
                println "================================================================================================================="
                println "step 2 - prep to build locally"
                println "================================================================================================================="
            
                sh """
                sudo apt-get update
                sudo apt-get upgrade -y
                
                sudo apt-get install pip git
                sudo pip install -r requirements.txt
                sudo pip install pytest-html-reporter selenium webdriver-manager
                
                ##################################

                sudo apt-get install -y unzip xvfb libxi6 libgconf-2-4 fonts-liberation libcairo2 libgbm1 libgtk-3-0 libgtk-4-1 libpango-1.0-0 libu2f-udev libvulkan1 libxdamage1 libxkbcommon0 xdg-utils

                if [ ! -f "chrome.deb" ]; then
                    # Download and install Chrome
                    wget -q -O chrome.deb "https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb"
                    sudo dpkg -i chrome.deb
                    sudo apt-get install -f -y
                fi
                """

            }
        }

        stage('Runing Tests, report failure or push successful build to repo') {
            steps {
                println "================================================================================================================="
                println "step 3 - try to build, run and run tests"
                println "================================================================================================================="

                sh """

                # try to dockerize the shop (check dockerfile)
                sudo docker build -t eshop .
                # try to dockerize the shop (check dockerfile and volume use)
                sudo docker run -d -p 5000:5000 -v .:/code eshop
                # run the code tests
                sleep 5
                pytest --html-report=report.html
                """
            }
        
            post {
                always {
                    // Cleanup
                   
                   
                    
                    '''

                }
                
                success {
                    // Push image to repository
                    println "================================================================================================================="
                    println "step 5 - Success - Push image to dockerhub"
                    println "================================================================================================================="
                     
                    


                    script {
                        withCredentials([string(credentialsId: "DockerHubPW", variable: 'DOCKERHUB_PASSWORD')]) {
                            sh """
                            sudo rm report.html
                            docker login -u shalom2012 -p ${DOCKERHUB_PASSWORD}
                            docker tag eshop amitmenashe/ecommerce-django-react
                            docker push shalom2012/ecommerce-django-react
                            """
                            build job: 'success_pipeline'
                        }
                    }
                    sh '''
                    if [ "$(docker ps -q)" ]; then
                        sudo docker kill $(docker ps -q)
                        sudo docker rm $(docker ps -qa)
                    fi

                    #sh 'docker run  ... new container 80:5000
                }

                failure {
                    // Generate report and send by email
                    println "================================================================================================================="
                    println "step 5 - Failure - publish report and email to developer(s)"
                    println "================================================================================================================="
                    
                    script {
                        #build job: 'failure_pipeline'
                        println ' tests failed'
                    }
                    
                }
            }
        }
    }
}
