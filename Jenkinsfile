pipeline {
    agent any
    
    environment {
        // Target path on the VM
        VM_DIR = '/storage/wso2/wso2/MI-4.4.0/wso2mi-4.4.0/repository/deployment/server/carbonapps'
        
        // TODO: Update these variables
        VM_USER = 'usman.farooq4' 
        VM_IP = '10.50.13.105'
        // NOTE: Maven builds often append "-SNAPSHOT" to the version defined in your pom.xml. 
        // Double-check your pom.xml version to ensure this matches exactly.
        CAR_FILE_NAME = 'pipeline-testing-1.0.0-SNAPSHOT.car' 
    }
    
    stages {
        stage('Build CAR File') {
            steps {
                script {
                    // Make the maven wrapper executable (required for Linux/Ubuntu)
                    sh 'chmod +x mvnw'
                    
                    // Build the project using the maven wrapper
                    sh './mvnw clean package'
                }
            }
        }

        stage('Authenticate PAM') {
            steps {
                script {
                    // Pauses the pipeline to ask for your Kron PAM credentials
                    def userInput = input(
                        id: 'pam-login',
                        message: 'Generate your Kron PAM Passcode NOW (you have 30 seconds!)',
                        parameters: [
                            password(name: 'PAM_PASS', description: 'PAM Password'),
                            string(name: 'PAM_OTP', description: 'Kron PAM Passcode')
                        ]
                    )
                    env.PAM_PASS = userInput.PAM_PASS
                    env.PAM_OTP = userInput.PAM_OTP
                }
            }
        }
        
        stage('Deploy to VM') {
            steps {
                script {
                    // Generates a temporary expect script to bypass Kron PAM prompts
                    sh '''
                    cat << 'EOF' > deploy.exp
                    #!/usr/bin/expect -f
                    set timeout 25
                    set password $env(PAM_PASS)
                    set otp $env(PAM_OTP)
                    
                    # SCP grabs the newly built file from the Jenkins 'target' directory
                    spawn scp -o StrictHostKeyChecking=no target/$env(CAR_FILE_NAME) $env(VM_USER)@$env(VM_IP):$env(VM_DIR)/
                    
                    # Wait for the password prompt and send password
                    expect "*assword:*" 
                    send "$password\\r"
                    
                    # Wait for the passcode prompt and send OTP
                    expect "*asscode:*" 
                    send "$otp\\r"
                    
                    expect eof
                    EOF
                    
                    chmod +x deploy.exp
                    ./deploy.exp
                    rm deploy.exp
                    '''
                }
            }
        }
    }
}