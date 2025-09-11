# SKY Bridge Setup Guide

## Complete Setup Instructions for Blackbaud NXT Airbyte Connector

### Table of Contents
1. [Prerequisites](#prerequisites)
2. [SKY API Setup](#sky-api-setup)
3. [Airbyte Installation](#airbyte-installation)
4. [Connector Configuration](#connector-configuration)
5. [Testing & Validation](#testing--validation)
6. [Production Deployment](#production-deployment)

---

## Prerequisites

### Required Accounts
- [ ] Blackbaud account with NXT access
- [ ] SKY Developer account (free at developer.blackbaud.com)
- [ ] Airbyte instance (cloud recommended - self hosted may require adjustments)

### Technical Requirements
- [ ] Self-hosted Airbyte not tested, as the provided CDK is not current with the cloud version, causing version mismatch. Further investigation required. 
- [ ] See Airbyte docs for self-hosted totally open-source approach.
---

## SKY API Setup

### Step 1: Create Developer Account
1. Navigate to https://developer.blackbaud.com/skyapi/docs/getting-started
2. Follow steps to create developer account.

### Step 2: Create SKY Application
1. Log into SKY Developer Portal
2. Navigate to "My Applications"
3. Click "New Application"
4. Configure application:
   - **Application Name**: (example: "NXT Airbyte Connector")
   - **Application Details**: (example: "Syncs NXT data to Airbyte")
   - **Team**: Choose a team (if applicable)
   - **Publisher**: Enter your name or your organization
   - **Application Logo**: optional
   - **Application Website URL**: your domain/url

### Step 3: Configure OAuth Settings
1. In your application settings, add redirect URI:
   - For Airbyte Cloud: `https://cloud.airbyte.com/auth_flow`
   - For self-hosted: (your public-facing URL - example, ngrok - see Airbyte documentation for port forwarding)
2. Enable required scopes:
   - Raiser's Edge NXT (Read)
   - ...others as needed (Online Giving (Read) etc.)
3. - **Follow Documentation to ensure that application is added to an environment**


### Step 4: Get Credentials
Note these values from your application:
- **Application ID** (Client ID): _____________________
- **Application Secret**: _____________________
- **Primary Subscription Key**: _____________________

---

## Airbyte Installation

### Option A: Airbyte Cloud (Recommended for Quick Start)
1. Sign up at https://cloud.airbyte.com
2. Complete onboarding
3. Navigate to Sources section

### Option B: Self-Hosted Airbyte (https://docs.airbyte.com/platform/using-airbyte/getting-started/oss-quickstart)
```bash
curl -LsfS https://get.airbyte.com | bash -
```

---

## Connector Configuration

### Step 1: Add Custom Connector
1. In Airbyte, navigate to **Builder**
2. Click **+ New Custom Connector**
3. Select **Import a YAML manifest** -> **Import a YAML** button
4. Choose `sky-connector.yaml`
5. Fill in required 'Inputs' (if desired for testing) - OR click 'YAML' toggle at top to let draft save
7. Click **Publish**
8. Ignore warnings about untested streams (type 'ignore warnings')

### Step 2: Create Source Connection
1. Go to **Sources** → **+ New Source**
2. Go to **Custom** → Select "Sky connector"
3. Configure connection:
   ```
   Connection Name: (name, example 'NXT')
   Client ID: [Your Application ID]
   Client Secret: [Your Application Secret]
   Subscription Key: [Your Primary Key]
   Start Date: 2020-01-01T00:00:00Z <- default if left blank
   ```
4. Click **Authenticate** (OAuth window will open)
5. Log into Blackbaud
6. Authorize the application
7. Return to Airbyte
8. Click **Test & Save**

### Step 3: Configure Destination
Example: PostgreSQL Setup
1. Go to **Destinations** → **+ New Destination**
2. Select (example, Google Sheets or PostgreSQL)
3. Configure destination as appropriate. Follow instructions for setting up chosen destination. 
4. Test connection

### Step 4: Create Connection
1. Go to **Connections** → **+ Create a Connection**
2. Select Source: "NXT Production"
3. Select Destination: Your configured destination
4. Configure streams:
   - Select desired streams (constituents, gifts, etc.)
   - Choose sync mode:
     - **Incremental | Append**: For transaction data
     - **Full Refresh | Overwrite**: For reference data
5. Set schedule:
   - Recommended: Every 6 hours for incremental
   - Daily for full refresh
6. Click **Create Connection**

---

## Testing & Validation

### Initial Sync Test
1. Start with a single stream (e.g., "funds")
2. Run manual sync
3. Verify data in destination
4. Check for errors in logs

### Data Validation Checklist
- [ ] Record counts match NXT
- [ ] Date fields properly formatted
- [ ] Custom fields present
- [ ] No duplicate records
- [ ] Incremental sync captures changes

### Performance Testing
1. Monitor initial full sync time
2. Note API rate limit usage
3. Check destination write performance
4. Optimize based on findings

---

## Production Deployment

### Best Practices

#### Stream Selection
- Start with essential streams only
- Add streams gradually
- Monitor performance impact

#### Sync Scheduling
```
High-Priority Streams (Hourly):
- Gifts
- Constituents

Medium-Priority (Daily):
- Campaigns
- Appeals
- Opportunities

Low-Priority (Weekly):
- Code tables
- Custom fields
```

#### Monitoring Setup
1. Configure Airbyte notifications
2. Set up destination monitoring
3. Create data quality checks
4. Build alerting for sync failures

#### Security Considerations
- [ ] Rotate API keys 
- [ ] Limit NXT scopes to minimum required

### Troubleshooting Guide

#### Common Issues and Solutions

**Issue**: OAuth authentication fails
```
Solution:
1. Verify redirect URI matches exactly
2. Ensure user has proper NXT permissions
3. Ensure application is connected to environment
```

**Issue**: No data syncing
```
Solution:
1. Verify API subscription is active
2. Check date ranges in configuration
3. Ensure API limit not hit (403 error)
4. Review Airbyte logs for errors
```

**Issue**: Slow sync performance
```
Solution:
1. Reduce page size in YAML (default 100)
2. Limit date range for initial sync
3. Use incremental sync mode
4. Schedule during off-peak hours
```

**Issue**: Query timeout errors
```
Solution:
1. Reduce query complexity
2. Add date parameters to limit results
3. Use multiple smaller queries
```

---

## Maintenance

### Regular Tasks
- **Daily**: Check sync status
- **Weekly**: Review error logs
- **Monthly**: Validate data completeness
- **Quarterly**: Update connector YAML if needed

### Upgrading Connector
1. Backup current YAML configuration
2. Test changes in development environment
3. Update YAML in Airbyte
4. Run test sync
5. Deploy to production

---

## Support Resources

### Documentation
- [SKY API Documentation](https://developer.blackbaud.com/skyapi)
- [Airbyte Documentation](https://docs.airbyte.com)
- [Connector GitHub Repository](#)

### Community
- Blackbaud Developer Community
- Airbyte Slack Community
- GitHub Issues for bug reports

### Contact
For connector-specific issues, please open a GitHub issue.

---

*Last Updated: September 2025*