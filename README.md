# Mintplex Labs Helm Charts

This is the central repository for Mintplex Labs Helm Charts, providing production-ready Kubernetes deployments for various applications.

## 📋 Table of Contents

- [Usage](#usage)
- [Available Charts](#available-charts)
- [Repository Architecture](#repository-architecture)
- [Development Workflow](#development-workflow)
- [GitHub Pages Integration](#github-pages-integration)
- [Contributing](#contributing)
- [Troubleshooting](#troubleshooting)

## 🚀 Usage

### Prerequisites

[Helm](https://helm.sh) must be installed to use the charts.
Please refer to Helm's [documentation](https://helm.sh/docs/) to get started.

### Adding the Repository

Once Helm is set up properly, add the repo as follows:

```bash
helm repo add mintplex-labs https://mintplex-labs.github.io/helm-charts
```

If you had already added this repo earlier, run `helm repo update` to retrieve the latest versions of the packages:

```bash
helm repo update
```

### Searching and Installing Charts

You can then run `helm search repo mintplex-labs` to see the charts:

```bash
helm search repo mintplex-labs
```

Install a chart with:

```bash
helm install my-release mintplex-labs/<chart-name>
```

### Alternative Installation Methods

#### OCI Registry (GitHub Container Registry)

Charts are also available via OCI registry:

```bash
# Install directly from OCI registry
helm install my-release oci://ghcr.io/mintplex-labs/helm-charts/<chart-name> --version <version>
```

#### Direct from GitHub Releases

Download chart packages directly from [GitHub Releases](https://github.com/mintplex-labs/helm-charts/releases):

```bash
# Download and install from release assets
wget https://github.com/mintplex-labs/helm-charts/releases/download/<chart-name>-<version>/<chart-name>-<version>.tgz
helm install my-release ./<chart-name>-<version>.tgz
```

## 🏗️ Repository Architecture

This repository follows a structured approach to manage and distribute Helm charts:

### Directory Structure

```
├── .github/
│   └── workflows/
│       └── helm.yml              # CI/CD pipeline for chart releases
├── charts/
│   └── <chart-name>/
│       ├── Chart.yaml            # Chart metadata
│       ├── values.yaml           # Default configuration values
│       ├── README.md             # Chart-specific documentation
│       └── templates/            # Kubernetes manifest templates
├── index.html                    # GitHub Pages landing page
└── README.md                     # This file
```

### Branching Strategy

- **`main`**: Primary development branch containing chart sources
- **`gh-pages`**: Auto-generated branch serving the Helm repository via GitHub Pages

## 🔄 Development Workflow

### How the Repository Works

This repository uses a sophisticated CI/CD pipeline that automatically:

1. **Builds and validates** charts when changes are pushed to `main`
2. **Publishes charts** to multiple distribution channels
3. **Updates the repository index** for seamless chart discovery

### Automated Release Process

When you push changes to the `charts/` directory on the `main` branch, the following happens automatically:

#### 1. Chart Validation

```yaml
# The CI pipeline performs:
- helm dependency build # Downloads chart dependencies
- helm lint # Validates chart syntax and best practices
- helm package # Creates distributable .tgz files
```

#### 2. Multi-Channel Publishing

**a) OCI Registry (GitHub Container Registry)**

- Charts are pushed to `ghcr.io/mintplex-labs/helm-charts/<chart-name>`
- Enables modern `helm install oci://` installation method
- Provides container-native distribution

**b) GitHub Releases**

- Creates individual releases tagged as `<chart-name>-<version>`
- Attaches chart `.tgz` files as release assets
- Generates rich release notes with installation instructions

**c) Traditional Helm Repository (GitHub Pages)**

- Updates the `gh-pages` branch with chart packages
- Generates/updates `index.yaml` for Helm repository discovery
- Serves charts via `https://mintplex-labs.github.io/helm-charts`

#### 3. Repository Index Updates

The pipeline automatically:

- Downloads all chart packages from GitHub Releases
- Generates a new `index.yaml` with all available charts and versions
- Updates the GitHub Pages site with the latest repository index
- Copies the main `index.html` landing page to the gh-pages branch

## 🌐 GitHub Pages Integration

### How GitHub Pages Serves the Helm Repository

The repository leverages GitHub Pages to provide a traditional Helm repository endpoint:

#### Repository Structure (gh-pages branch)

```
gh-pages/
├── README.md               # README (copied from main branch)
├── index.html              # Landing page (copied from main branch)
├── index.yaml              # Helm repository index (auto-generated)
```

#### Access Points

1. **Helm Repository**: `https://mintplex-labs.github.io/helm-charts`

   - Accessed via `helm repo add` commands
   - Serves `index.yaml` for chart discovery

2. **Landing Page**: `https://mintplex-labs.github.io/helm-charts`

   - User-friendly web interface
   - Installation instructions and chart documentation

3. **Direct Package Access**: `https://mintplex-labs.github.io/helm-charts/packages/<chart-name>-<version>.tgz`
   - Direct download links for chart packages

### Repository Index (`index.yaml`)

The `index.yaml` file is the heart of a Helm repository, containing:

- Chart metadata (name, version, description)
- Download URLs for chart packages
- Checksums for package integrity
- Dependencies and requirements

Example structure:

```yaml
apiVersion: v1
entries:
  chart-name:
    - name: chart-name
      version: 1.0.0
      appVersion: "1.0"
      description: Chart description
      urls:
        - https://mintplex-labs.github.io/helm-charts/packages/chart-name-1.0.0.tgz
      digest: sha256:...
      created: "2025-10-12T10:00:00Z"
```

## 🛠️ Contributing

### For Chart Developers

#### Adding a New Chart

1. **Create Chart Structure**:

   ```bash
   mkdir -p charts/my-new-chart
   cd charts/my-new-chart
   helm create .  # Or create manually
   ```

2. **Update Chart.yaml**:

   ```yaml
   apiVersion: v2
   name: my-new-chart
   description: A Helm chart for my application
   version: 0.1.0
   appVersion: "1.0"
   ```

3. **Configure values.yaml**:

   - Set sensible defaults
   - Document all configuration options
   - Follow consistent naming conventions

4. **Create README.md**:
   - Installation instructions
   - Configuration options table
   - Usage examples

#### Updating Existing Charts

1. **Bump Version**: Update `version` in `Chart.yaml`
2. **Update App Version**: Update `appVersion` if the underlying application version changed
3. **Document Changes**: Update the chart's README.md with changes
4. **Test Locally**:
   ```bash
   helm lint charts/my-chart
   helm template test charts/my-chart
   ```

#### Version Conventions

Follow [Semantic Versioning](https://semver.org/):

- **MAJOR**: Incompatible API changes
- **MINOR**: Backwards-compatible functionality additions
- **PATCH**: Backwards-compatible bug fixes

### Workflow Integration Points

#### CI/CD Pipeline (`.github/workflows/helm.yml`)

The workflow is triggered by:

- **Path Filter**: Only runs when files in `charts/**` are modified
- **Branch Filter**: Only runs on pushes to `main` branch

Key workflow steps:

1. **Setup**: Configures Helm, Git, and required tools
2. **Validation**: Lints and validates all charts
3. **Packaging**: Creates distributable chart packages
4. **Publishing**: Pushes to OCI registry and creates GitHub releases
5. **Repository Update**: Updates GitHub Pages with new chart index

#### Environment Variables

The workflow uses these environment variables:

- `REGISTRY`: OCI registry URL (`ghcr.io/mintplex-labs/helm-charts`)
- `GITHUB_TOKEN`: Automatically provided for authentication
- `GITHUB_ACTOR`: GitHub username for Git configuration

#### Permissions Required

The workflow requires these permissions:

```yaml
permissions:
  contents: write # For creating releases and updating gh-pages
  packages: write # For pushing to GitHub Container Registry
  pull-requests: write # For PR automation (future use)
```

## 🔍 Troubleshooting

### Common Issues

#### Chart Not Appearing in Repository

1. **Check Workflow Status**: Verify the GitHub Actions workflow completed successfully
2. **Verify Chart Structure**: Ensure `Chart.yaml` is valid and properly formatted
3. **Check Version**: Ensure you bumped the chart version in `Chart.yaml`
4. **Repository Cache**: Run `helm repo update` to refresh your local cache

#### OCI Registry Issues

1. **Authentication**: Verify GitHub Container Registry permissions
2. **Package Visibility**: Check if packages are public in GitHub settings
3. **Version Conflicts**: Ensure you're not trying to overwrite an existing version

#### GitHub Pages Not Updating

1. **Workflow Permissions**: Verify the workflow has `contents: write` permission
2. **Branch Protection**: Check if `gh-pages` branch has protection rules
3. **Pages Settings**: Ensure GitHub Pages is enabled and pointing to `gh-pages` branch

### Development Tips

#### Local Testing

```bash
# Lint charts
helm lint charts/*

# Test template rendering
helm template test-release charts/my-chart

# Test installation (with dry-run)
helm install test-release charts/my-chart --dry-run --debug

# Package locally
helm package charts/my-chart
```

#### Debugging Workflow

1. **Check Workflow Logs**: Navigate to Actions tab in GitHub
2. **Validate YAML**: Use `yamllint` to check chart YAML files
3. **Test Locally**: Reproduce the workflow steps locally before pushing

### Getting Help

- **Issues**: Create an issue in this repository for bugs or feature requests
- **Discussions**: Use GitHub Discussions for questions and community support
- **Documentation**: Refer to [Helm's official documentation](https://helm.sh/docs/)

---

## 📚 Additional Resources

- [Helm Documentation](https://helm.sh/docs/)
- [Helm Best Practices](https://helm.sh/docs/chart_best_practices/)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)

---

**Maintainer**: Mintplex Labs Team
**Repository**: https://github.com/mintplex-labs/helm-charts
**Website**: https://mintplexlabs.com

---

**Credit**: [Sam Culley](https://github.com/sculley) for the original repository structure and foundation.
