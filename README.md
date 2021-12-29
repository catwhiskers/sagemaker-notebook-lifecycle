# sagemaker-notebook-lifecycle

### Setup SageMaker Domain 
1. configure user and associated execution role 
2. configure VPC and subnets 
### Using Cloud9 to setup sagemaker studio notebooks lifecycle 
1. select proper instance type and operating system 
2. update aws cli 
    1. download aws cli 
      ```
      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" 
      unzip awscliv2.zip 
      ```
    2. install 
      ```
      sudo ./aws/install --update 
      PATH=/usr/local/aws-cli/v2/2.4.7/bin/:$PATH
      echo $PATH
      ```
    3. validate aws cli version     
      ```
      aws --version
      ```
    4. install jq for later use 
      ```
      sudo yum install jq
      ```
### Install custom packages on base kernel images
1. Download script 
    ```
    wget https://gist.githubusercontent.com/catwhiskers/20237b2667e41a46bdedd910ee404cae/raw/c8498315082ecb087f0fd1f44e87eef722454d40/install-package.sh 
    ```
2. Transform it to base64 format and assign the string to a variable 
    ```
    LCC_CONTENT=`openssl base64 -A -in install-package.sh`
    ```
    validate LCC_CONTENT 
    
    ```
    echo $LCC_CONTENT
    ```
3. Install the script to lifecycle configuration   

    ```
    aws sagemaker create-studio-lifecycle-config \
    --studio-lifecycle-config-name install-parrow-package-on-kernel \
    --studio-lifecycle-config-content $LCC_CONTENT \
    --studio-lifecycle-config-app-type KernelGateway 
    ```
     obtain lifecycle configuration arn 
   
    ```
   config_arn=`aws sagemaker describe-studio-lifecycle-config --studio-lifecycle-config-name install-parrow-package-on-kernel | jq -r   ".StudioLifecycleConfigArn"`
    ``` 
4. Binding the configuration to a specific user 
    get domain_id 
    ```
    aws sagemaker list-domains 
    ```

    ```
    domain_id=`aws sagemaker list-domains | jq -r '.Domains[0].DomainId'` 
    ```
    get user_name 
    ```
    aws sagemaker list-user-profiles
    ```
    
    ```
    user_name=`aws sagemaker list-user-profiles | jq -r '.UserProfiles[0].UserProfileName'`  
    ```
    
    ```
    aws sagemaker update-user-profile --domain-id $domain_id \
    --user-profile-name $user_name \
    --user-settings "{
      \"KernelGatewayAppSettings\": {
      \"LifecycleConfigArns\":
        [\""$config_arn"\"]
      }
    }"

    ```
5. Let's open a notebook!     

### Configure auto-shutdown of inactive kernels
1. download script 


      
