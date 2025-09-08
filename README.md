####  This project sends live NBA game scores to subscribed users via SMS or Email using AWS services. It acts like a personal sports alert bot in the cloud. ####

## Learning Goals
Understand how cloud services interact in a real-world workflow
Learn to fetch data from an external API (sportsdata.io)
Practice automated notifications using Amazon SNS
Automate tasks with Amazon EventBridge
Follow AWS security best practices
Use GitHub for version control
Document your system with an architecture diagram

 ##Architecture Diagram
NBA API (sportsdata.io) -> Lambda (fetch & process scores) -> SNS (SMS/Email notifications) -> Users
                                ↑
                                |
                        EventBridge (every 10 mins)


NBA API (sportsdata.io) -> Provides live NBA scores
AWS Lambda -> Fetches scores, formats results, and publishes notifications
Amazon SNS -> Sends alerts (SMS / Email) to subscribers
Amazon EventBridge -> Automates Lambda execution on schedule
GitHub -> Stores and version-controls Lambda code


## Prerequisites

A sportsdata.io account + API Key
An AWS account
Basic knowledge of Python
A GitHub account + Git installed
Any diagramming tool (optional, for architecture diagrams)

 ##Step-by-Step Setup
Step 1: Create an SNS Topic
In AWS Console -> SNS -> Topics -> Create topic.
Name it NBA-Score-Updates.

Add subscriptions:

Email -> enter your email, confirm via link.
SMS -> enter your number, confirm via reply.
-->SNS is now ready to deliver alerts.

Step 2: Create a Lambda Function

Go to Lambda → Create function.
Runtime: Python 3.x.
Name: nba-score-fetcher.

Add an IAM policy that allows sns:Publish to your SNS topic.

Example Python code (lambda_function.py):

import os
import json
import boto3
import requests

sns_client = boto3.client("sns")

def lambda_handler(event, context):
    api_key = os.environ["NBA_API_KEY"]
    sns_topic_arn = os.environ["SNS_TOPIC_ARN"]
    # Example API call (update with actual sportsdata.io endpoint)
    url = f"https://api.sportsdata.io/v3/nba/scores/json/GamesByDate/2025-01-01?key={api_key}"
    response = requests.get(url)
    games = response.json()
    
   messages = []
    for game in games:
        message = f"{game['HomeTeam']} {game['HomeTeamScore']} - {game['AwayTeam']} {game['AwayTeamScore']}"
        messages.append(message)
        
   final_message = "\n".join(messages) or "No games found"
    
   # Publish to SNS
   sns_client.publish(
        TopicArn=sns_topic_arn,
        Message=final_message,
        Subject="NBA Score Update"
    )
    
   return {"status": "ok", "message": final_message}


## Best Practice: Store your NBA_API_KEY and SNS_TOPIC_ARN in Lambda Environment Variables, not in code.

Step 3: Automate with EventBridge
Go to EventBridge → Scheduler → Create schedule.

Expression:
cron(0/10 * * * ? *)
--> Runs every 10 minutes.

Target: Select your Lambda function.

Role: Provide an IAM role with permission to invoke Lambda.
 Lambda will now run automatically every 10 minutes.

Step 4: Push Code to GitHub

On your computer:
git init
git add .
git commit -m "Initial commit - NBA score bot"

Create a repo on GitHub (e.g., nba-score-alerts).

Link and push:
git remote add origin https://github.com/USERNAME/nba-score-alerts.git
git push -u origin main


--> Your Lambda code is now version-controlled on GitHub.

## Deliverables

By the end, you will have:
A working NBA score alert system on AWS
An architecture diagram
A GitHub repo containing your Lambda function code

## Cleanup

To avoid unnecessary costs:
Delete the EventBridge schedule.
Delete the Lambda function.
Delete the SNS topic.
Remove IAM roles if not reused.
