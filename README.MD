<!-- toc -->

- [Betty](#betty)
  - [Requirements](#requirements)
    - [User Roles](#user-roles)
    - [Key Features](#key-features)
    - [User Interface (UI/UX)](#user-interface-uiux)
    - [Security](#security)
  - [Database Requirements](#database-requirements)
    - [Table Structure](#table-structure)
      - [a) Users](#a-users)
      - [b) Matchups and Bets](#b-matchups-and-bets)
      - [c) Stakes](#c-stakes)
    - [Relationships](#relationships)
    - [Additional Requirements](#additional-requirements)
- [Solution](#solution)
  - [1. Overview](#1-overview)
  - [2. Architecture Components](#2-architecture-components)
    - [2.1 Backend](#21-backend)
    - [2.2 Frontend](#22-frontend)
      - [State Management](#state-management)
      - [Dependencies](#dependencies)
  - [3. Security Considerations](#3-security-considerations)
  - [4. Deployment](#4-deployment)
  - [5. Scalability & Performance](#5-scalability--performance)
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
  - Once a bet is placed, the “BET” button is disabled to prevent duplicate bets.
- **My Bets:**
  - Displays the user’s closed and open bets.
  - Status indicators: Open, Closed, Won.
- **Results:**
  - Shows winnings.
- **Dynamic Updates:**
  - Changes made in the admin dashboard (e.g., descriptions, limits, ..) are instantly reflected in the app.

### User Interface (UI/UX)

- **Responsive Design:**
  - Optimized for mobile devices.
- **Navigation:**
  - Clearly separates categories and personal bets.
- **Feedback System:**
  - Confirmation upon successful bet placement.
  - Notifications for winnings.

### Security

- Input validation for stakes (e.g., within the allowed range).
- Ensures users can place a bet only once per game.

## Database Requirements

### Table Structure

#### a) Users

- `user_id` (Primary Key)
- `balance` (Available funds)

#### b) Matchups and Bets

- `matchup_id` (Primary Key)
- `title` (matchup name)
- `description` (matchup description)
- `max_bet` (Maximum stake limit)
- `category` (matchup category)
- `status` (Open/Closed)

#### c) Stakes

- `matchup_id` (Foreign Key)
- `user_id` (Foreign Key)
- `amount` (Stake amount)
- `status` (Won/Lost)
- `created_at`

### Relationships

- Stakes have a 1:n relationship with Users.
- Each bet has multiple stakes, but each stake belongs to only one user.

### Additional Requirements

- Store results and winning amounts.
- Real-time updates for admin changes.

# Solution

## 1. Overview

Betty is a serverless betting application built with AWS, React, and WebSockets.  
The application is available online at https://d1m0id10877d91.cloudfront.net/

## 2. Architecture Components

![Architecture diagram](https://lucid.app/publicSegments/view/d031250d-0a68-4ff3-8b7d-29bfc376e14c/image.jpeg)

### 2.1 Backend

- **AWS Lambda** functions (Node.js) for serverless compute
- **AWS API Gateway** with **WebSocket** APIs
- **AWS Cognito** for API authentication
- **AWS Elasticache Redis** to manage active user connection IDs and cache matchups
- **AWS DynamoDB** for storage
- **AWS SQS** for network isolation between Lambda functions

### 2.2 Frontend

- Single Page Application (SPA) built with **React**
- Hosted on **AWS S3**
- Accessed through **AWS CloudFront** for global delivery

#### State Management

- Uses Redux and Redux Thunk
- Components connect to Redux using `connect` HOC (Higher-Order Component)

#### Dependencies

- React
- Redux
- WebSocket API
- TypeScript

## 3. Security Considerations

- Data in transit is encrypted with **HTTPS**
- User authentication via AWS Cognito with **Google** integration
- Lambda functions and Elasticache are in a **private subnet**
- IAM roles follow the least privilege principle

## 4. Deployment

- Uses AWS SAM (Serverless Application Model) for deployment
- Infrastructure is defined with CloudFormation templates
- Deploy with a single command: `sam build` and `sam deploy`
- The application is available online at https://d1m0id10877d91.cloudfront.net/

## 5. Scalability & Performance

- Serverless architecture enables automatic scaling
- CloudFront provides low-latency content delivery

## 6. Monitoring & Logging

- Monitoring and logging via AWS CloudWatch

## 7. Cost Optimization

- Pay-per-use model for serverless components
- Elasticache and VPC resources have hourly costs

## Appendix: Considerations for Choosing Between SQL and NoSQL for Betting Application

### Context

In the development of this betting application, a crucial decision lies between using a SQL database or a NoSQL database like DynamoDB.  
The requirements are to quickly develop a Minimum Viable Product (MVP) with the potential for rapid user growth and limited need for complex queries.

### Data Structure Discussion

#### Initial Proposal

1. **Single Matchups Table**:
   - Initially, the application will have a flat `matchups` table to capture all necessary information: matchups as well as bets.
   - A matchup is the event itself. For example, NBA Team 1 vs Team 2.
   - A bet is specific outcome or condition that the user can wager on. For example, we can have two distinct bets based on the outcomes: Bet 1: Team 1 win by 5 points. Bet 2: Team 1 win by 20 points. Each bet will have odds, which is the likelihood of the winning, for example: Bet 1 might have odds of 2.0, and Bet 2 might have different odds, say 3.0.
2. **Future Changes**:
   - The application may introduce a `bets` table with a foreign key `matchupId` to the `matchups` table, allowing for normalization of data.

#### Normalization vs. Denormalization

##### Normalization

- **Pros**:
  - Reduced data redundancy.
  - Easier maintenance of consistency.
- **Cons**:
  - Increased complexity in querying.
  - Potential increased read costs.

##### Denormalization

- **Pros**:
  - Simplicity in querying.
  - Faster performance for read operations.
- **Cons**:
  - Data redundancy.
  - Risks of consistency issues.

### Recommendation

- Adopt a denormalized structure in the initial stages to simplify querying.
- Implement processes for maintaining data consistency during updates.

### SQL vs. NoSQL Considerations

#### Using DynamoDB

1. **Schema Flexibility**:
   - DynamoDB allows for rapid iterations without a predefined schema.
2. **Scaling**:
   - Designed for horizontal scaling, suitable for anticipated user growth.
3. **Complex Queries**:
   - Limited support for complex queries, which may pose challenges as the application evolves.
4. **Data Consistency**:
   - Offers eventual consistency, which may not be suitable for critical betting operations.

#### Conclusion

- **DynamoDB Suitability**:
  - Pros: Rapid development, high scalability for simple MVP.
  - Cons: Challenges in transitioning to a normalized structure and limited query capabilities.
- **Recommendation**:
  - If aiming for rapid MVP development without complex queries, DynamoDB is a suitable choice.
  - However, if complex relationships and queries are anticipated, considering a SQL database may be beneficial for long-term development.

### Decision

To maintain simplicity and facilitate rapid development, the current matchups table will remain, enhanced by adding an array of bets. Each bet will detail options like:

- Bet description (e.g., "Home team under 4 points")
- Potential payout per 100 (e.g., "120" for a bet with a payout of 120:100)

This structure will allow for flexibility in managing betting options while keeping the data model straightforward.

#### Example Calculation

If a user inserts a stake of $1,000 for a bet with odds of 120:100, the potential payout would be calculated as follows:

- Winnings: ($1,000 × 120) / 100 = $1,200
- Total Payout: $1,000 (stake) + $1,200 (winnings) = $2,200

### Next Steps

Discuss these considerations with the client to evaluate the best approach for the betting application's database architecture.
