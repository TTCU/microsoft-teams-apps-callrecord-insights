# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Microsoft Teams Call Record Insights is an Azure Functions application that retrieves, parses, flattens, and stores Microsoft Teams Call Records from Microsoft Graph API into Cosmos DB and Azure Data Explorer (Kusto) for analysis.

## Build Commands

```bash
# Build the solution
dotnet build CallRecordInsights.sln

# Build release
dotnet build CallRecordInsights.sln --configuration Release

# Publish for deployment
dotnet publish src/Functions/CallRecordInsights.Functions.csproj --configuration Release

# Clean build
dotnet clean CallRecordInsights.sln
```

## Project Structure

The solution contains two projects:

- **CallRecordInsights.Functions** (`src/Functions/`) - Azure Functions v4 application (.NET 6) that:
  - Subscribes to Microsoft Graph callRecords change notifications
  - Processes notifications from Event Hub
  - Downloads call records from Graph API
  - Stores flattened records in Cosmos DB

- **CallRecordInsights.Flattener** (`src/Flattener/`) - Library (.NET 6/7) that:
  - Flattens hierarchical call record JSON into tabular format
  - Transforms Graph API call records into `KustoCallRecord` objects suitable for Cosmos DB and Kusto

## Key Architecture Components

### Azure Functions (Triggers)
- `ProcessEventHubEventFunction` - Receives Graph notifications from Event Hub, queues call IDs for download
- `ProcessCallRecordFunction` - Queue-triggered, downloads call records from Graph API and stores in Cosmos
- `AddOrRenewCallRecordsSubscriptionFunction` - Timer-triggered, maintains Graph subscription
- `GetCallRecordInsightsHealthFunction` - Health check endpoint
- `GetCallRecordAdminFunction` / `CallRecordsFunctionsAdmin` - Admin HTTP endpoints

### Services
- `CallRecordsGraphContext` - Manages Graph API interactions for subscriptions and call record retrieval
- `CallRecordsDataContext` - Handles Cosmos DB operations for storing processed records
- `JsonFlattener` - Transforms nested JSON into flat key-value structure

### Data Flow
1. Graph API sends call record notifications to Event Hub
2. `ProcessEventHubEventFunction` queues call IDs to Storage Queue
3. `ProcessCallRecordFunction` downloads full call records from Graph API
4. `JsonFlattener` flattens records using configured JSON paths
5. Flattened records stored in Cosmos DB (and synced to Kusto via Cosmos DB connector)

## Configuration

Configuration is loaded from `appsettings.json` and environment variables:
- `AzureAd` section - Azure AD app registration for Graph API authentication
- `GraphSubscription` section - Graph notification subscription settings
- `CallRecordInsightsDb` section - Cosmos DB connection settings

Required environment variables for Functions:
- `EventHubConnection` - Event Hub connection string
- `CallRecordsQueueConnection` - Storage Queue connection string
- `GraphNotificationEventHubName` - Event Hub name
- `CallRecordsToDownloadQueueName` - Queue name

## Deployment

Deployment uses Bicep templates via PowerShell:

```powershell
# Full deployment
.\deploy\deploy.ps1 -ResourceGroupName <name> -SubscriptionId <id> -Location <region>
```

The deploy script provisions:
- Azure Functions App
- Event Hub for Graph notifications
- Storage Account for queues
- Cosmos DB account
- Optional: Azure Data Explorer (Kusto) cluster

Bicep templates are in `deploy/bicep/`.

## Multi-Tenant Support

The application supports multi-tenant scenarios:
- `CallRecordsGraphOptions.Tenants` can specify multiple tenant IDs
- `AzureIdentityMultiTenantGraphAuthenticationProvider` handles per-tenant authentication
- Each call record is stored with `CallRecordTenantIdContext` for tenant isolation

## Dependencies

Central package versions managed in `Directory.Packages.props`:
- Microsoft.Graph 5.x - Graph SDK for call records API
- Microsoft.Azure.Cosmos - Cosmos DB client
- Microsoft.Identity.Web - Azure AD authentication
- Azure Functions SDK v4


## AI_HANDOFF.md Protocol
(These rules govern the 'AI_HANDOFF.md' file found in the root)
1. **Transient State Only:** This file is a specific 'mutex' token for the next session. It is NOT a project history log.
2. **No Documentation:** Do not write architectural decisions or code snippets here. Use README.md for that.
3. **Clean Up:** When updating this file, REMOVE completed tasks. Do not mark them as [x] and leave them. The file should only ever contain the *current* state and *immediate* next steps.
4. **Size Limit:** Keep this file short (under 50 lines).
