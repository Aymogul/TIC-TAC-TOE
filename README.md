# TIC-TAC-TOE
name: Build,Analyze,scan

on:
  push:
    branches:
      - main


jobs:
  build-analyze-scan:
    name: Build
    runs-on: [self-hosted]
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
        with:
          fetch-depth: 0  # Shallow clones should be disabled for a better relevancy of analysis 
  
      - name: Deploy to container
        run: | 
          docker stop game
          docker rm game
        
      - name: Update kubeconfig
        run: aws eks --region ap-south-1 update-kubeconfig --name EKS_CLOUD

      - name: Deploy to kubernetes
        run: kubectl delete -f deployment-service.yml
  
      - name: Send a Slack Notification
        if: always()
        uses: act10ns/slack@v1
        with:
          status: ${{ job.status }}
          steps: ${{ toJson(steps) }}
          channel: '#githubactions-eks'
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}



- name: Docker build and push
        run: |
          # Run commands to build and push Docker images
          docker build -t tic-tac-toe .
          docker tag tic-tac-toe aymogul/tic-tac-toe:latest
          docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{ secrets.DOCKERHUB_TOKEN }}
          docker push aymogul/tic-tac-toe:latest
        env:
          DOCKER_CLI_ACI: 1