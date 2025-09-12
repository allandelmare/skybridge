(Work in progress -- use at your own risk - in development, ready for testing, not recommended for production) 
# SKY Bridge - Airbyte Connector for Blackbaud NXT

An open-source Airbyte connector that enables seamless data integration between Blackbaud NXT (via SKY API) and 50+ modern data platforms including Snowflake, BigQuery, PostgreSQL, and more.

## 🎯 Overview

SKY Bridge democratizes access to NXT data by providing a no-code/low-code solution for streaming your Blackbaud data directly into your data infrastructure. Built for the Blackbaud developer community, this connector eliminates the complexity of custom API development while providing enterprise-grade reliability.

## ✨ Features

- **20+ Data Streams**: Comprehensive access to NXT data including constituents, gifts, campaigns, and more
- **OAuth 2.0 Authentication**: Secure, automatic token management
- **Incremental Sync**: Efficient data updates with configurable date ranges
- **Custom Queries**: Execute NXT queries with parameter support
- **In Development**: Built-in pagination but currently not handling tokens (causing harmless 'non unique primary key' awarning), retry logic, and error handling (limited - WIP/to-do)
- **Open Source**: MIT licensed for community contribution and customization

## 📊 Supported Data Streams

- ✅ Constituents (with full profile data)
- ✅ Gifts and Gift Custom Fields
- ✅ Campaigns, Funds, and Appeals
- ✅ Opportunities
- ✅ Actions and Notes
- ✅ Addresses, Phones, and Emails
- ✅ Relationships
- ✅ Constituent Codes and Custom Fields
- ✅ Code Tables
- ✅ Custom Queries (with parameters)

## 🚀 Quick Start

### Prerequisites

1. **Airbyte Instance**: Airbyte Cloud (self hosted may require modification depending on CDK version)
2. **SKY API Access**: 
   - Blackbaud developer account
   - Registered SKY application
   - API subscription key

### Installation Steps

#### Step 1: Register Your SKY Application

1. Go to [SKY Developer Portal](https://developer.blackbaud.com)
2. Create a new application
3. Note your:
   - Application ID (Client ID)
   - Application Secret
   - Subscription Key

#### Step 2: Add Connector to Airbyte

1. In Airbyte, navigate to Builder
2. Click + New Custom Connector
3. Select Import a YAML manifest -> Import a YAML button
4. Choose sky-connector.yaml
5. Fill in required 'Inputs' (client ID, secret, API key)
6. Important: to properly configure your schema and tests in builder mode, you need to stub the access key with dummy data/starting value so its not null (example, just enter 12345678 - oauth will replace it).
7. Choose a stream (ie, 'funds'). Scroll to the oAuth section/button and click to authorize.
8. Click 'test'. This will populate your schema for that stream.
9. Test streams (or if not planning to use a stream, you can skip tests and ignore warnings).
10. Note that you'll need to change the Query ID for queries, or the code table ID for code table streams to match your environment.
11. Click 'Publish'

#### Step 3: Configure Connection

1. Create a new source using your custom connector (sources -> custom -> choose connector)
2. Enter your credentials:
   - **Client ID**: Your SKY application ID
   - **Client Secret**: Your SKY application secret
   - **Subscription Key**: Your SKY API subscription key
   - **Start Date**: Date from which to sync data (e.g., 2020-01-01T00:00:00Z which is the default if left blank)
3. Complete OAuth authorization when prompted
4. Test and save the connection

#### Step 4: Set Up Destination

Choose from 50+ destinations:
- **Data Warehouses**: Snowflake, BigQuery, Redshift, Databricks
- **Databases**: PostgreSQL, MySQL, MongoDB, MS SQL Server
- **Data Lakes**: S3, Azure Blob, GCS
- **And many more...**

#### Step 5: Configure Sync

1. Select the streams (which fields from which API's, and/or queries) you want to sync. Note that large data sets may consume your API limits - start small. 
2. Choose sync frequency (e.g., hourly, daily)
3. Configure sync mode (Full Refresh or Incremental)
4. Start syncing!

## 🔧 Advanced Configuration

### Custom Query Configuration

To use custom NXT queries, modify the query ID in the YAML -- or -- in the builder UI:

```yaml
gift_query:
  creation_requester:
    request_body:
      value:
        id: YOUR_QUERY_ID  # Replace with your NXT query ID
        parameters:
          DateFrom: "{{ stream_interval.start_time }}"
          DateTo: "{{ stream_interval.end_time }}"
```

### Code Table Configuration

To sync specific code tables, update the table ID:

```yaml
codetable_solicitcodes:
  requester:
    url: >-
      https://api.sky.blackbaud.com/codetable/v1/codetables/YOUR_TABLE_ID/tableentries
```

### Incremental Sync Settings

Adjust the incremental sync parameters:

```yaml
incremental_sync:
  step: P30D  # Sync window size (30 days)
  lookback_window: PT2H  # Overlap window (2 hours)
  cursor_field: date_modified  # Field to track changes
```

## 📖 Documentation

### Authentication Flow

The connector handles OAuth 2.0 authentication automatically:
1. Initial authorization via Blackbaud login
2. Automatic token refresh
3. Secure credential storage in Airbyte
4. SPECIAL NOTE: there is a bug with sliding window refresh token management. For now, as a bypass, refresh token is set to static = TRUE, which will keep the refresh token valid for 1 year, at which point it will require re-authentication. This will be improved to properly handle token refresh in future versions. 

### Pagination Strategies

Two pagination methods are supported:
- **Offset Pagination**: For standard endpoints. Note that a token based approach to better support this is in development. Current implementation causes harmless "unique primary key" warnings. 
- **Continuation Token**: For large result sets

### Error Handling

Built-in resilience features:
- Automatic retry with exponential backoff
- Graceful handling of rate limits
- Detailed error logging

## 🤝 Contributing

We welcome contributions from the Blackbaud developer community!

### How to Contribute

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

### Areas for Contribution

- Additional data streams
- Query templates
- Performance optimizations
- Documentation improvements
- pagination offset handling improvrment using token
- Bug fixes

## 📊 Use Cases

### Unified Analytics Platform
Stream NXT data to Snowflake alongside marketing and financial data for comprehensive analytics.

### Custom Applications
Sync constituent data to PostgreSQL to power donor portals and custom applications.

### Data Science & ML
Load data into Databricks for predictive modeling and advanced analytics.

### Compliance & Archival
Archive to S3 for long-term retention and regulatory compliance.

## 🐛 Troubleshooting

### Common Issues

**OAuth Error**: Ensure redirect URI in SKY app matches Airbyte's callback URL

**No Data Syncing**: Check that your API subscription includes the required scopes

**Slow Performance**: Consider reducing sync frequency or limiting date ranges

**Query Timeout**: For large queries, increase the timeout in the YAML configuration

### Getting Help

- Open an issue on GitHub
- Check existing issues for solutions
- Review Blackbaud SKY API documentation
- Join the Blackbaud developer community

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Blackbaud for providing the SKY API
- Airbyte for the open-source data integration platform
- The Blackbaud developer community for feedback and support

## 📬 Contact

**Developer**: Allan Delmare

**Project**: Created for bbdevdays 2025 Hackathon

**Track**: Data Quality/Hygiene

---

*SKY Bridge - Extending the power of NXT through open-source innovation*

## ⚡ Quick Reference

### Required Scopes
Ensure your SKY application has these scopes:
- NXT (Read)

### Sync Recommendations
- Initial Sync: Start with a limited date range
- Incremental: Use daily syncs for active data
- Full Refresh: Monthly for reference data

---

**Built with ❤️ for the Blackbaud Community**
