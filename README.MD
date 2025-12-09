# Preface

**Betty** is an MVP level serverless betting app that allows users to place and manage bets on various sports events.

The app has two user roles: Admins who oversee matchups, betting limits and analytics, and Users who can place bets, view their history, and track winnings.

Key features include categorized matchups, stake validation, bet status tracking, instant result notifications, and analytics.

The app is designed for scalability, using serverless computing and storage, and a pay-per-use model for cost optimization, with global content delivery via CloudFront.

The app offers an intuitive, mobile-friendly UI/UX, and robust monitoring through AWS CloudWatch, built with AWS services, React, and WebSockets for real-time updates.

The app provides secure authentication through Google.

~~The app is available online at https://dsqkkiozdrv6g.cloudfront.net~~ 

# Table Of Content

<!-- toc -->

- [Betty](#betty)
  - [Requirements](#requirements)
    - [User Roles](#user-roles)
    - [Key Features](#key-features)
    - [UI/UX](#uiux)
    - [Database](#database)
      - [Tables Schema](#tables-schema)
        - [a) Users](#a-users)
        - [b) Matchups and Bets](#b-matchups-and-bets)
        - [c) Stakes](#c-stakes)
      - [Relationships](#relationships)
      - [Additional Requirements](#additional-requirements)
  - [Solution](#solution)
    - [1. Overview](#1-overview)
    - [2. Architecture](#2-architecture)
      - [2.1 Backend](#21-backend)
      - [2.2 Frontend](#22-frontend)
    - [3. Security Considerations](#3-security-considerations)
    - [4. Deployment](#4-deployment)
    - [5. Scalability, Performance and Resiliency](#5-scalability-performance-and-resiliency)
      - [Eventual consistency](#eventual-consistency)
    - [6. Monitoring & Logging](#6-monitoring--logging)
    - [7. Cost Optimization](#7-cost-optimization)
    - [Appendix: Considerations for Choosing Between SQL and NoSQL for Betting Application](#appendix-considerations-for-choosing-between-sql-and-nosql-for-betting-application)
      - [Context](#context)
      - [Data Structure Discussion](#data-structure-discussion)
        - [Initial Proposal](#initial-proposal)
        - [Normalization vs. Denormalization](#normalization-vs-denormalization)
          - [Normalization](#normalization)
          - [Denormalization](#denormalization)
      - [Recommendation](#recommendation)
      - [SQL vs. NoSQL Considerations](#sql-vs-nosql-considerations)
        - [Using DynamoDB](#using-dynamodb)
        - [Conclusion](#conclusion)
      - [Decision](#decision)
        - [Example Calculation](#example-calculation)
      - [Next Steps](#next-steps)

<!-- tocstop -->

# Betty

## Requirements

### User Roles

- **Admin:**
  - Defines matchups, limits, and descriptions for games.
- **User:**
  - Places matchups and reviews previous matchups.
  - Views results and winnings.

### Key Features

- **Bet Overview:**
  - Displays all available matchups categorized (e.g., Football, Basketball).
  - Tabs: All, My Bets, and specific categories.
- **Placing Bets:**
  - Users input their stake (up to the defined maximum limit).
  - Validates the stake before submission.
  - Once a bet is placed, the "BET" button is disabled to prevent duplicate bets.
- **My Bets:**
  - Displays the user's closed and open bets.
  - Status indicators: Open, Closed, Won.
- **Results:**
  - Shows winnings.
- **Dynamic Updates:**
  - Changes made in the admin dashboard (e.g., descriptions, limits, ..) are instantly reflected in the app.

### UI/UX

- **Responsive Design:**
  - Optimized for mobile devices.
- **Navigation:**
  - Clearly separates categories and personal bets.
- **Feedback System:**
  - Confirmation upon successful bet placement.
  - Notifications for winnings.
- **Input Validation for Stakes:**
  - Must be within the allowed range as specified by the admin.
  - Users can place a bet only once per matchup.

### Database

#### Tables Schema

##### a) Users

- `userId` (Primary Key)
- `balance` (Available funds)

##### b) Matchups and Bets

- `matchupId` (Primary Key)
- `title` (matchup name)
- `description` (matchup description)
- `maxBet` (Maximum stake limit)
- `category` (matchup category)
- `status` (Open/Closed)

##### c) Stakes

- `matchupId` (Foreign Key)
- `userId` (Foreign Key)
- `amount` (Stake amount)
- `status` (Won/Lost)
- `createdAt`

#### Relationships

- Stakes have a 1:n relationship with Users.
- Each bet has multiple stakes, but each stake belongs to only one user.

#### Additional Requirements

- Store results and winning amounts.
- Real-time updates for admin changes.

## Solution

### 1. Overview

Betty is a serverless MVP-level betting application utilizing AWS, React, and WebSockets, designed for dynamic and real-time user experience. The architecture comprises a Node.js backend with AWS Lambda for serverless computing, API Gateway for WebSocket APIs, and DynamoDB for data storage, ensuring scalability and performance. The frontend is a responsive React SPA hosted on AWS S3 and delivered via CloudFront. Key features include user roles for admins and users, betting functionality, and real-time updates. Security is handled with HTTPS encryption and AWS Cognito for authentication. Cost efficiency is achieved with a pay-per-use model while monitoring and logging are handled by AWS CloudWatch. The decision to use DynamoDB allows rapid MVP development, although future considerations for normalization may arise as the application evolves.

The app is available online: [Deployment](#4-deployment)

### 2. Architecture

![Architecture diagram](https://lucid.app/publicSegments/view/7d2a62ab-0ba3-4cde-ae00-aa80b3236b44/image.jpeg)

#### 2.1 Backend

- **AWS Lambda** (Node.js) for serverless computing
- **AWS API Gateway** with **WebSocket** APIs
- **AWS Cognito** for API authentication
- **AWS ElastiCache Redis** to manage active user connection IDs and cache matchups
- **AWS DynamoDB** for data storage
- **AWS SQS** for decoupling and isolating Lambda functions
- **AWS EventBridge** for scheduling Lambda functions
- **AWS SNS** for notifying all registered users about new matchups
- **AWS SES** for emailing individual users about wins or losses
- **AWS Athena** for ad-hoc analytics on stakes data stored in S3
- **AWS Step Function** for an alternative ad-hoc analytics on stakes data stored in S3

#### 2.2 Frontend

- Single Page Application (SPA) developed with React
- Hosted on AWS S3
- Delivered globally via **AWS CloudFront**
- Technology stack: **React**, **Redux** (HOC), **TypeScript**

### 3. Security Considerations

- Data in transit is encrypted with **HTTPS**
- User authentication via AWS Cognito with **Google** integration
- Lambda functions and Elasticache Redis are in a **private subnet**
- IAM roles follow the least privilege principle

### 4. Deployment

- Uses AWS SAM (Serverless Application Model) for deployment
- Infrastructure is defined with CloudFormation templates
- Deploy with a single command: `sam build` and `sam deploy`
~~- The app is available online at https://dsqkkiozdrv6g.cloudfront.net~~

### 5. Scalability, Performance and Resiliency

- Serverless architecture enables automatic scaling
- Elasticache Redis enhances the scalability of read operations
- CloudFront provides low-latency content delivery

#### Eventual consistency

In this architecture, the user balance is updated in the Users table through a DynamoDB stream processor (a Lambda function) that processes updates from the Matchups table. The balance is only updated after the matchup update has been persisted. This approach can introduce a delay in balance updates, meaning users may not see their updated balance instantly. However, this delay is often acceptable in scenarios where immediate consistency is not critical, enabling more resilient systems.

Additionally, this design can improve scalability because the streams processor can handle a large volume of events asynchronously. This allows the primary application (the Generic Websocket Receiver —refer to the [Architecture diagram](#2-architecture)) to remain responsive and effectively manage incoming requests without being blocked by the balance update operations. Overall, this architecture separates concerns and facilitates more efficient resource utilization.

### 6. Monitoring & Logging

- Monitoring and logging via AWS CloudWatch

### 7. Cost Optimization

- Pay-per-use model for serverless components
- Elasticache and VPC resources have hourly costs

### Appendix: Considerations for Choosing Between SQL and NoSQL for Betting Application

#### Context

In the development of this betting application, a crucial decision lies between using a SQL database or a NoSQL database like DynamoDB.  
The requirements are to quickly develop a Minimum Viable Product (MVP) with the potential for rapid user growth and limited need for complex queries.

#### Data Structure Discussion

##### Initial Proposal

1. **Single Matchups Table**:
   - Initially, the application will have a flat `matchups` table to capture all necessary information: matchups as well as bets.
   - A matchup is the event itself. For example, NBA Team 1 vs Team 2.
   - A bet is specific outcome or condition that the user can wager on. For example, we can have two distinct bets based on the outcomes: Bet 1: Team 1 win by 5 points. Bet 2: Team 1 win by 20 points. Each bet will have odds, which is the likelihood of the winning, for example: Bet 1 might have odds of 2.0, and Bet 2 might have different odds, say 3.0.
2. **Future Changes**:
   - The application may introduce a `bets` table with a foreign key `matchupId` to the `matchups` table, allowing for normalization of data.

##### Normalization vs. Denormalization

###### Normalization

- **Pros**:
  - Reduced data redundancy.
  - Easier maintenance of consistency.
- **Cons**:
  - Increased complexity in querying.
  - Potential increased read costs.

###### Denormalization

- **Pros**:
  - Simplicity in querying.
  - Faster performance for read operations.
- **Cons**:
  - Data redundancy.
  - Risks of consistency issues.

#### Recommendation

- Adopt a denormalized structure in the initial stages to simplify querying.
- Implement processes for maintaining data consistency during updates.

#### SQL vs. NoSQL Considerations

##### Using DynamoDB

1. **Schema Flexibility**:
   - DynamoDB allows for rapid iterations without a predefined schema.
2. **Scaling**:
   - Designed for horizontal scaling, suitable for anticipated user growth.
3. **Complex Queries**:
   - Limited support for complex queries, which may pose challenges as the application evolves.
4. **Data Consistency**:
   - Offers eventual consistency, which may not be suitable for critical betting operations.

##### Conclusion

- **DynamoDB Suitability**:
  - Pros: Rapid development, high scalability for simple MVP.
  - Cons: Challenges in transitioning to a normalized structure and limited query capabilities.
- **Recommendation**:
  - If aiming for rapid MVP development without complex queries, DynamoDB is a suitable choice.
  - However, if complex relationships and queries are anticipated, considering a SQL database may be beneficial for long-term development.

#### Decision

To maintain simplicity and facilitate rapid development, the current matchups table will remain, enhanced by adding an array of bets. Each wager will detail options like:

- Wager description (e.g., "Home team under 4 points")
- Potential payout per 100 (e.g., "120" for a wager with a payout of 120:100)

This structure will allow for flexibility in managing betting options while keeping the data model straightforward.

##### Example Calculation

If a user inserts a stake of $1,000 for a wager with odds of 120:100, the potential payout would be calculated as follows:

- Winnings: ($1,000 × 120) / 100 = $1,200
- Total Payout: $1,000 (stake) + $1,200 (winnings) = $2,200

#### Next Steps

Discuss these considerations with the client to evaluate the best approach for the betting application's database architecture.
