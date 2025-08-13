# MongoDB Service Template

A Backstage Software Template for provisioning MongoDB database instances using Crossplane.

## Overview

This template creates the necessary Crossplane resources to provision a MongoDB database instance in your Kubernetes cluster. It includes:

- **XRD (Composite Resource Definition)**: Defines the MongoDB API
- **Composition**: Implements the MongoDB provisioning logic
- **Example Claim**: Shows how to request a MongoDB instance

## Features

- 🚀 **Self-Service Provisioning**: Developers can provision MongoDB instances through Backstage
- ⚙️ **Configurable Storage**: Choose storage size from 1GB to 100GB
- 🔧 **Automatic DNS**: Creates DNS records for database access
- 📊 **Status Tracking**: Connection strings available in resource status

## Prerequisites

### Crossplane Installation

Ensure Crossplane is installed in your cluster:

```bash
kubectl create namespace crossplane-system
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm install crossplane --namespace crossplane-system crossplane-stable/crossplane
```

### Required Providers

This template requires the following Crossplane providers:

1. **Provider NOP** (for simulation/testing):
```bash
kubectl apply -f - <<EOF
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-nop
spec:
  package: xpkg.upbound.io/crossplane-contrib/provider-nop:v0.2.0
EOF
```

### Required Functions

Install the necessary Crossplane composition functions:

```bash
kubectl apply -f - <<EOF
apiVersion: pkg.crossplane.io/v1beta1
kind: Function
metadata:
  name: function-go-templating
spec:
  package: xpkg.upbound.io/crossplane-contrib/function-go-templating:v0.10.0
---
apiVersion: pkg.crossplane.io/v1beta1
kind: Function
metadata:
  name: function-auto-ready
spec:
  package: xpkg.upbound.io/crossplane-contrib/function-auto-ready:v0.2.1
---
apiVersion: pkg.crossplane.io/v1beta1
kind: Function
metadata:
  name: function-sequencer
spec:
  package: xpkg.upbound.io/crossplane-contrib/function-sequencer:v0.3.0
---
apiVersion: pkg.crossplane.io/v1beta1
kind: Function
metadata:
  name: function-environment-configs
spec:
  package: xpkg.upbound.io/crossplane-contrib/function-environment-configs:v0.1.0
EOF
```

## Usage

### Through Backstage

1. Navigate to the Software Catalog
2. Click "Create Component"
3. Select "MongoDB Database"
4. Fill in the required parameters:
   - **Instance Name**: Unique identifier for your MongoDB instance
   - **Namespace**: Kubernetes namespace (default: `default`)
   - **Storage Size**: Size in GB (1-100)
   - **Owner**: Team or user who owns this resource

### Manual Deployment

1. Apply the XRD:
```bash
kubectl apply -f content/definition.yaml
```

2. Apply the Composition:
```bash
kubectl apply -f content/composition.yaml
```

3. Create a claim:
```bash
kubectl apply -f content/example.yaml
```

## Parameters

| Parameter | Description | Type | Default | Required |
|-----------|-------------|------|---------|----------|
| `name` | Instance name | string | - | Yes |
| `namespace` | Kubernetes namespace | string | `default` | No |
| `storageSize` | Storage size in GB | integer | `10` | No |
| `owner` | Resource owner | string | `group:platform` | No |

## Architecture

The template creates a composite resource that provisions:

1. **Cluster Resource**: A simulated compute cluster for MongoDB
2. **DNS Record**: Automatic DNS entry for database access  
3. **ConfigMap**: Stores configuration values
4. **NOP Resource**: Simulates database provisioning delay (30s)

## Connection String

After provisioning, the connection string is available in the resource status:

```bash
kubectl get mongodb <instance-name> -n <namespace> -o jsonpath='{.status.connString}'
```

Format: `mongodb+srv://<username>:<password>@<fqdn>/<database>`

## Development

### Testing Locally

1. Clone this repository
2. Install the template in Backstage:
```yaml
catalog:
  locations:
    - type: url
      target: https://github.com/open-service-portal/service-mongodb-template/blob/main/template.yaml
```

### Customization

To customize the MongoDB provisioning:

1. Edit `content/definition.yaml` to add parameters
2. Modify `content/composition.yaml` to change provisioning logic
3. Update `template.yaml` to expose new parameters in Backstage

## Troubleshooting

### Common Issues

**MongoDB stuck in Creating state**
- Check if all required functions are installed
- Verify the NOP provider is running: `kubectl get providers`

**Connection string not appearing**
- Ensure DNS record creation succeeded
- Check composition logs: `kubectl describe composition mongodb`

**Permission denied errors**
- Verify Crossplane has necessary RBAC permissions
- Check ServiceAccount bindings

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Submit a pull request

## License

MIT License - see LICENSE file for details

## Support

For issues and questions:
- GitHub Issues: [open-service-portal/service-mongodb-template](https://github.com/open-service-portal/service-mongodb-template/issues)
- Discussions: [open-service-portal/discussions](https://github.com/orgs/open-service-portal/discussions)