# API Overview

The Bitsec-AI API provides programmatic access to subnet functionality for advanced users and developers.

## Authentication

All API requests require authentication using API keys:

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
     https://api.bitsec.ai/v1/status
```

## Base URL

```
https://api.bitsec.ai/v1
```

## Endpoints

### Health Check

```http
GET /health
```

Returns the current status of the API service.

### Validator Status

```http
GET /validators/{validator_id}/status
```

Get the current status of a specific validator.

### Miner Status

```http
GET /miners/{miner_id}/status
```

Get the current status of a specific miner.

### Subnet Metrics

```http
GET /subnet/metrics
```

Retrieve current subnet performance metrics.

## Rate Limits

- 1000 requests per hour for authenticated users
- 100 requests per hour for unauthenticated requests

## SDK

We provide official SDKs for popular programming languages:

- Python: `pip install bitsec-sdk`
- JavaScript: `npm install @bitsec/sdk`
- Go: `go get github.com/bitsec-ai/go-sdk`

## Support

For API support:

- Check our [troubleshooting guide](../subnet/troubleshooting.md)
- Visit our GitHub repository
- Contact our developer support team