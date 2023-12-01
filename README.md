# Code to Cloud Attack Defend Lab

## Overview

In this lab, we will set up the Jenkins server with the Prisma Cloud IDE plugin and create a CI/CD pipeline where we will fix the issue in every stage. Our goal is to take find and fix problems in the early stage and demonstrate how we can stop issues from getting pushed on the production server. Also, we are going to explore Prisma Code Cloud Security features.

## Scenario Resources
1 VPC with:
EC2 x 3
Scenario Start(s)
We are setting up the Prisma IDE plugin and injecting Security at every stage in the pipeline.
Scenario Goal(s)
Setup pipeline in Jenkins and fail the stage based on Critical/High issues.

## Pre-requisite 

Download and install visual studio code from the Microsoft website: https://visualstudio.microsoft.com/

## Lab Setup

1. Download Terraform script : https://drive.google.com/drive/folders/1q4dsTymW4oBc-PcnzuSaJZWt4xEF7ta0?usp=sharing and install Terraform

2. Unzip and run the script by moving into attack_lab_script/aws directory # malware-detection
