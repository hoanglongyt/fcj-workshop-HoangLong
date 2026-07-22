---
title: "Proposal"
date: 2026-06-30
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# TSL-SignMap
## AI-Integrated & Community-Driven Traffic Sign Location Management System

### 1. Executive Summary
TSL-SignMap is an innovative technological solution designed to build and maintain a real-time traffic sign database in Vietnam. The system integrates a user-friendly mobile application, open-source mapping services (OpenStreetMap), advanced Computer Vision (YOLO AI models for traffic sign detection and classification), and a crowdsourced data collection model. Featuring a tokenized economy (TSL Coin) and a multi-tiered reputation-based consensus and voting process, TSL-SignMap ensures data integrity, transparency, and accuracy, helping drivers navigate safely while supporting road authorities in efficient traffic management.

---

### 2. Problem Statement & Proposed Solution

#### What’s the Problem?
- **Outdated Traffic Data**: Traffic signs in Vietnam are constantly changing (newly added, relocated, obstructed, or removed), yet updates rely heavily on manual surveys that are costly and time-consuming.
- **Suboptimal Existing Mapping Apps**: Standard navigation tools often lack granular traffic sign data or fail to reflect recent regulatory sign updates along specific routes promptly.
- **Traffic Safety Risks**: The absence of timely and accurate traffic sign information increases the risk of unintentional traffic violations and compromised road safety for commuters.

#### The Solution (TSL-SignMap)
TSL-SignMap addresses these challenges through a collaborative smart ecosystem:
- **Real-Time Mobile Application**: Integrated with OpenStreetMap to display real-time traffic sign locations directly on an interactive map.
- **Community Crowdsourcing**: Drivers and local residents can report missing, inaccurate, or newly added traffic signs.
- **AI-Powered Detection & Classification**: Cutting-edge computer vision models (YOLO) automatically detect and classify traffic signs from user-uploaded images, minimizing manual verification effort.
- **Weighted Voting Consensus**: Validates user contributions based on upvotes/downvotes weighted by user reputation, spatial proximity (GPS distance), and expertise.
- **TSL Coin Economy**: Incentivizes active and high-quality community participation through a transparent coin reward and penalty system.

---

### 3. Functional Requirements

#### a. User Registration & Authentication
- Secure registration and authentication for mobile users.
- Initial welcome bonus of 20 TSL Coins upon account creation.
- Reputation score management and TSL Coin balance tracking.

#### b. Traffic Sign Display & Navigation
- Real-time map display powered by OpenStreetMap showing categorized traffic signs (regulatory, warning, mandatory, and informational signs).

#### c. Search & Advanced Filtering
- Search traffic signs by type or proximity radius.
- Advanced filtering capabilities costing 1 TSL Coin per query.

#### d. User Contributions
- Submit updates (new, missing, or incorrect signs) with GPS coordinates and optional descriptions/photos.
- Contribution submission fee: 5 TSL Coins.
- Uploaded photos are automatically scanned by the YOLO AI pipeline for validation and preliminary classification.

#### e. Voting Mechanism & Community Verification
- Qualified users (meeting minimum activity thresholds) can cast upvotes or downvotes on pending contributions.
- Earn 1 TSL Coin per matching vote (up to 5 TSL Coins/day).
- **Automated Verification Rules**:
  - Over 70% consensus (after 5+ votes or 7 days): Automatically approved and integrated into the database.
  - Under 30% consensus: Automatically rejected.
  - 30% - 70% consensus: Flagged for administrator review.

#### f. Administrator Web Dashboard
- Web-based dashboard for administrators to review flagged contributions, approve/reject submissions, and override votes if necessary.
- Adjust reward/penalty parameters and user reputation scores to maintain ecosystem balance.

#### g. TSL Coin Economic Lifecycle
- **Rewards**: 10+ TSL Coins for approved contributions; 1 TSL Coin for matching votes.
- **Expenditures**: Spend coins on map access (2 TSL Coins/day), contribution submission (5 TSL Coins), or advanced filters (1 TSL Coin).
- **Top-up**: Users can top up TSL Coins with fiat currency (e.g., $1 for 10 TSL Coins).

---

### 4. System Architecture & Technology Stack

#### Architecture Overview
- **Mobile Client**: Developed with Flutter / React Native, featuring an OpenStreetMap interface (MapLibre GL / Leaflet).
- **AI Processing Pipeline**: YOLO (You Only Look Once) model trained on Vietnamese traffic sign datasets for real-time object detection and classification.
- **Backend Services**: RESTful API & WebSocket Server (Node.js / Python FastAPI) for real-time synchronization.
- **Database Layer**: PostgreSQL + PostGIS (spatial database for sign locations), Redis (caching & leaderboard), Amazon S3 / Cloud Storage (image storage).
- **Admin Web Dashboard**: Built with React.js / Next.js.

---

### 5. Timeline & Milestones

- **Month 1: Data Collection & AI Model Training**
  - Gather dataset of Vietnamese traffic sign images.
  - Train and evaluate YOLO models for sign detection and classification.
  - Design spatial database schemas (PostGIS) and API contracts.

- **Month 2: Core Platform Development & Token Economy**
  - Develop mobile application with OpenStreetMap integration.
  - Implement weighted voting algorithm and TSL Coin balance management.
  - Integrate AI pipeline for automated submission validation.

- **Month 3: Admin Dashboard & End-to-End Testing**
  - Build Admin Web Dashboard.
  - Conduct thorough testing (End-to-End Testing), optimizing AI recognition accuracy and map synchronization stability.

- **Month 4 Onward: Community Pilot & Deployment**
  - Launch community pilot in key cities (e.g., Ho Chi Minh City / Hanoi).
  - Collect user feedback, fine-tune consensus algorithms, and scale system capacity.

---

### 6. Risk Assessment & Mitigation Strategies

| Risk | Impact | Probability | Mitigation Strategy |
| :--- | :--- | :--- | :--- |
| Spam Reports / False Submissions | High | Medium | Require 5 TSL Coins submission fee, filter invalid photos via AI, weight votes by GPS distance and reputation. |
| TSL Coin Fraud (Sybil Attack) | Medium | Medium | User identity verification, cap daily voting rewards (max 5 Coins/day), penalize reputation score for bad behavior. |
| AI Misclassification | Medium | Low | Use AI for preliminary screening; rely on community consensus and Admin overrides for final approval. |
| Offline / Weak Network Connection | Low | Medium | Provide offline storage on mobile devices and auto-sync when network connection is restored. |

---

### 7. Expected Outcomes

- **Real-Time Traffic Sign Database**: Provides drivers with an accurate, continuously updated traffic sign map for safer navigation.
- **Cost Reduction**: Cuts manual survey and maintenance costs for transport authorities by up to 80%.
- **Community Engagement**: Fosters a collaborative road-safety ecosystem through transparent and rewarding TSL Coin mechanics.