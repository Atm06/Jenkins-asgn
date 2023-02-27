pipeline{
  agent any
  
  Parameters{
    string(name: 'APP_NAME', defaultValue: 'my_app', description: 'Name of the application')
    string(name: 'DOCKERFILE', defaultValue: 'Dockerfile', description: 'Name of Dockerfile')
    string(name: 'AWS_REGION', defaultValue: 'ap-south-1', description: 'AWS region where ECR and ECS are located')
    string(name: 'AWS_ACCOUNT_ID', defaultValue: '444335424395', description: 'AWS account Id')
    string(name: 'ECR_REPOSITORY', defaultValue: 'jenkins-pipeline', description: 'Name of the ECR repository')
    string(name: 'ECS_CLUSTER', defaultValue: 'Jenkins-Demo-Cluster', description: 'Name of the ECS Cluster')
    string(name: 'ECS_SERVICE', defaultValue: 'Jenkins-Demo-Cluster', description: 'Name of the ECS Service')
    string(name: 'ECS_TASK_ROLE', defaultValue: 'Jenkins-pipeline-role', description: 'Name of the ECS task role')
    string(name: 'ECS_TASK_MEMORY', defaultValue: '512', description: 'Memory in MB allocated to the ECS task')
    string(name: 'ECS_TASK_CPU', defaultValue: '256', description: 'CPU units the ECS task')
  }
  
  stages{
    stage('clone Github repository'){
      steps{
        git branch: 'main', url: 'https://github.com/Atm06/Jenkins-asgn.git'
      }
    }
    
    stage('Build Docker Image'){
      steps{
        script{
          docker.build("${params.APP_NAME}:${BUILD_NUMBER}", "-f ${params.DOCKERFILE} .")
        }
      }
    }
    
    stage('Push Docker Image to ECR'){
      steps{
        withAWS(region: "${params.AWS_REGION}", credentials: 'aws-creds'){
          def ecr = "AWS_ACCOUNT_ID.dkr.ecr.${params.AWS_REGION}.amazonaws.com/${params.ECR_REPOSITORY}"
          docker.withRegistry("${ecr}", 'ecr'){
            def image = docker.image("${params.APP_NAME}:${BUILD_NUMBER}")
            image.push()
          }
        }
      }
    }
    
    stage('Deploy to ECS'){
      steps{
        withAWS(region: "${params.AWS_REGION}", credentials: 'aws-creds'){
          def ecr = "AWS_ACCOUNT_ID.dkr.ecr.${params.AWS_REGION}.amazonaws.com/${params.ECR_REPOSITORY}"
          def taskDefinition = createTaskDefinition("{params.APP_Name}:${BUILD_NUMBER}", "{ecr}")
          def ecsService = updateEcsService("${params.ECS_CLUSTER}", "${params.ECS_SERVICE}", "${taskDefinition}")
        }
      }
    }
  }
  
  def createTaskDefinition(image, ecr){
    def taskDefinition = awsEcsTaskDefinition(
      family: "${params.APP_NAME}",
      networkMode: 'awsvpc',
      requiresCompatibilities: ['FARGATE'],
      cpu: "${params.ECS_TASK_CPU}",
      memory: "${params.ECS_TASK_ROLE}",
      taskRoleArn: 'ecsTaskExecutionRole',
      containerDefinitions: [
        awsEcsContainerDefinition(
          name: "${params.APP_NAME}",
          image: "${ecr}:${image}",
          portMappings: [awsEcsPortMapping(containerPort: 80, protocol: 'tcp')],
          logConfiguraion: [
            awsEcsLogConfiguration(
              logDriver: 'awslogs',
              options: [
                'awslogs-group': '/ecs/${params.APP_NAME}',
              
