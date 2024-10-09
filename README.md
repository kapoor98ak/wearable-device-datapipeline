# Wearable Device Data Engineering System

## Project Overview

This project involves the design and implementation of a scalable **data engineering system** for a wearable device company. The system collects, processes, and analyzes health data from wristwatch-like devices that continuously send data such as heart rate events, user activity, and workout sessions.

The platform leverages the **Lakehouse architecture** to support both batch and real-time data ingestion, providing analytics and reporting on user health metrics and activity.

## Architecture
![Architecture Diagram](https://github.com/user-attachments/assets/67fd819a-87cb-427a-9dab-74fac6814215)



The system follows the **Medallion architecture** consisting of three key layers:

1. **Bronze Layer**: Ingests raw data from both batch and streaming sources.
2. **Silver Layer**: Cleans, processes, and deduplicates data, ensuring data quality.
3. **Gold Layer**: Aggregates the data into reporting tables for business analytics.

### Key Data Sources
- **Device Registration Data**: Stored in the cloud at the time of sale.
- **User Profile Data (CDC)**: Captured via a mobile app and sent to a Kafka topic.
- **Heart Rate Events**: High-volume data stream of user heart rates.
- **Login/Logout Events**: Tracks user entries/exits from fitness centers.
- **Workout Session Events**: Logs workout session start and stop times.

## Technologies

The system is built using the following technologies:

- **Databricks**: Used for data processing and orchestration.
- **Azure ADLS Gen2**: Cloud storage for raw and processed data.
- **Apache Kafka**: Manages real-time data ingestion.
- **Delta Lake**: Ensures ACID transactions and data reliability.
- **Azure Data Factory**: Orchestrates batch data ingestion.
- **Databricks Unity Catalog**: Implements fine-grained access control and data governance.

## Key Features

- **Batch and Streaming Data Processing**: Supports both real-time data ingestion via Kafka and batch processing through cloud databases.
- **Data Governance**: Role-based access control (RBAC) using **Databricks Unity Catalog** to ensure data security and governance across multiple environments (development, testing, production).
- **Scalable Architecture**: The **Lakehouse platform** decouples data ingestion from processing, allowing for independent scaling.
- **Automated CI/CD Pipelines**: Continuous integration and deployment pipelines using **Azure DevOps** to automate testing and deployments.

## Data Ingestion and Processing

The data ingestion strategy is divided into:
- **Batch Data**: Ingested using **Azure Data Factory** from SQL databases.
- **Streaming Data**: Ingested from **Kafka** topics for real-time data (e.g., heart rate and workout session events).

Data is first ingested into the **Bronze Layer** as raw data. The **Silver Layer** then cleans and processes this data to remove duplicates, apply CDC, and ensure quality. Finally, the **Gold Layer** aggregates data into reporting tables for analytics and business insights.

### Key Reporting Metrics

- **Workout BPM Summary**: Summarizes workout sessions by user, calculating minimum, average, and maximum BPM during workouts.
- **Gym Activity Summary**: Aggregates user activity and time spent in fitness centers.

## Deployment and Cost Management

We used **Databricks cluster policies** to manage resource usage and optimize costs by controlling the type, size, and number of clusters that can be created. Autoscaling is enabled to ensure efficient resource usage.

### Environments

- **Development**: For code testing and pipeline execution.
- **Testing**: To ensure correctness and data quality before production deployment.
- **Production**: The live environment that processes real-time data and provides analytics.

## CI/CD Pipeline

The project includes an automated **CI/CD pipeline** using **Azure DevOps**:
- **Build Pipeline**: Automatically tests the code and generates artifacts.
- **Release Pipeline**: Deploys code to development, testing, and production environments.

The pipeline supports **continuous integration (CI)** and **continuous deployment (CD)**, ensuring that all code changes are tested and validated before they are released into production.

## Challenges and Solutions

- **Batch and Streaming Integration**: We decoupled data ingestion and processing pipelines to support both batch and real-time data streams without dependency conflicts.
- **Data Governance**: Implemented fine-grained access control using **Databricks Unity Catalog** to secure data across multiple environments.
- **Cluster Management**: Controlled cluster resource usage with **Databricks cluster policies** to optimize costs and prevent resource overuse.

## Future Enhancements

- **Machine Learning Integration**: Future iterations could include predictive analytics and machine learning models using the processed data.
- **Additional Data Sources**: The system could be expanded to ingest data from other IoT devices or third-party APIs.

## Conclusion

This project delivers a scalable, efficient, and secure data engineering solution that processes both real-time and batch data from wearable devices. The use of **Lakehouse architecture** ensures data reliability, governance, and performance, providing valuable insights for both the company and its users.



