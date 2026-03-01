# EC2 ImageBuilder Updates to Match Packer Templates

## Overview
Updated EC2 ImageBuilder components and CloudFormation templates to align with the Packer template functionality for building EKS worker node AMIs.

## Key Changes

### Amazon Linux 2023 (AL2023)

#### ImageBuilder Component (`ImageBuilderComponents/al2023.yaml`)
1. **Added GPU support directory** - Now copies GPU-related files from `templates/al2023/runtime/gpu/`
2. **Added SetClockSource step** - Runs `set-clocksource.sh` provisioner
3. **Updated LimitCStates step** - Now uses shared provisioner path (`templates/shared/provisioners/limit-c-states.sh`)
4. **Enhanced environment variables**:
   - Added `INSTALL_CONTAINERD_FROM_S3` to install-worker.sh
   - Added AWS credentials (empty) to all provisioner scripts for consistency
   - Added `BINARY_BUCKET_NAME`, `BINARY_BUCKET_REGION` to install-efa.sh
   - Added `NVIDIA_REPOSITORY`, `EC2_GRID_DRIVER_S3_BUCKET` to install-nvidia-driver.sh
   - Added `WORKING_DIR` to install-neuron-driver.sh
5. **Separated cleanup steps**:
   - Cleanup
   - Validate (with `ENABLE_ACCELERATOR` env var)
   - GenerateVersionInfo
   - FinalCleanup
   - ConfigureSELinux
5. **Updated default versions**:
   - ContainerdVersion: `1.7.*` → `2.1.*`
   - NodeAdmBuildImage: `golang:1.24` → `golang:1.25`
   - NvidiaDriverMajorVersion: `560` → `580`
   - PauseContainerImage: Updated to use public.ecr.aws
6. **Added new constants**:
   - `InstallContainerdFromS3`
   - `NvidiaRepositoryUrl`
   - `NvidiaGridRunfileBucketName`

#### CloudFormation Template (`CloudFormation/imagebuilder-al2023.yaml`)
1. **Added new parameters**:
   - `InstallContainerdFromS3` (default: 'false')
   - `NvidiaRepositoryUrl` (default: '')
   - `NvidiaGridRunfileBucketName` (default: 'ec2-linux-nvidia-drivers')
2. **Updated default parameter values** to match Packer:
   - ContainerdVersion: `1.7.*` → `2.1.*`
   - NodeAdmBuildImage: `golang:1.24` → `golang:1.25`
   - NvidiaDriverMajorVersion: `560` → `580`
   - PauseContainerImage: Updated to public.ecr.aws
3. **Enhanced component data** with all new constants and environment variables
4. **Updated provisioner steps** to match ImageBuilder component changes

### Red Hat Enterprise Linux (RHEL)

#### ImageBuilder Component (`ImageBuilderComponents/rhel.yaml`)
1. **Added SetClockSource step** - Runs `set-clocksource.sh` provisioner
2. **Updated LimitCStates step** - Now uses shared provisioner path
3. **Enhanced environment variables**:
   - Added `SOCI_URL` and `SOCI_VERSION` to install-worker.sh
   - Added `AWS_CLI_URL` to install-worker.sh
   - Added AWS credentials (empty) to all provisioner scripts
   - Added `BINARY_BUCKET_NAME`, `BINARY_BUCKET_REGION` to install-efa.sh
4. **Separated cleanup steps**:
   - Cleanup
   - Validate
   - GenerateVersionInfo
   - FinalCleanup
   - ConfigureSELinux
5. **Updated default versions**:
   - NodeAdmBuildImage: `golang:1.24` → `golang:1.25`
6. **Added new constants**:
   - `SociUrl`
   - `SociVersion`

**Note:** AWS CLI installation is handled by the CloudFormation template (via the aws-cli-version-2-linux component) before custom components execute, so no AWS CLI logic is needed in the custom components.

#### CloudFormation Template (`CloudFormation/imagebuilder-rhel.yaml`)
1. **Added new parameters**:
   - `SociUrl` (default: 'https://api.github.com/repos/awslabs/soci-snapshotter/releases')
   - `SociVersion` (default: '*')
2. **Removed parameters**:
   - `AWSCliUrl` - AWS CLI is installed via the aws-cli-version-2-linux component
   - `EnableAwsCliComponent` - AWS CLI component is always included
3. **Updated default parameter values**:
   - NodeAdmBuildImage: `golang:1.24` → `golang:1.25`
4. **Enhanced component data** with all new constants and environment variables
5. **Updated CleanupComponent** to separate validation, version info generation, and SELinux configuration steps
6. **Simplified Setup step** - Removed AWS CLI installation logic since it's handled by the aws-cli-version-2-linux component

## Alignment with Packer Templates

The ImageBuilder components now match the Packer template provisioning flow:

### Provisioning Order (Both AL2023 and RHEL)
1. Setup - Clone repo, create directories, copy files
2. SetClockSource - Configure system clock source
3. EnableFIPS - Enable FIPS mode if requested
4. LimitCStates - Configure C-states for EFA if enabled
5. InstallWorker - Install Kubernetes components, containerd, runc
6. InstallNodeAdm - Install nodeadm tool
7. CachePauseContainer - Pre-pull pause container image
8. InstallNeuronDriver (AL2023 only) - Install AWS Inferentia drivers
9. InstallNvidiaDriver - Install NVIDIA GPU drivers
10. InstallEFA - Install Elastic Fabric Adapter drivers
11. Cleanup - Clean temporary files
12. Validate - Validate installation
13. GenerateVersionInfo - Generate version metadata
14. FinalCleanup - Remove working directories
15. ConfigureSELinux - Configure SELinux policies

### Environment Variables
All provisioner scripts now receive the same environment variables as in Packer templates, including:
- AWS credentials (set to empty for IAM role usage)
- Binary bucket configuration
- Version specifications
- Feature flags (FIPS, EFA, accelerators)
- Working directories

### Version Alignment
- Containerd: 2.1.* (AL2023)
- Golang build image: 1.25
- NVIDIA driver: 580 (AL2023)
- Pause container: Using public ECR images
- SOCI snapshotter support (RHEL)

## Benefits

1. **Consistency** - ImageBuilder builds now follow the same process as Packer builds
2. **Feature Parity** - All Packer features are now available in ImageBuilder
3. **Maintainability** - Easier to maintain both build methods in parallel
4. **Flexibility** - New parameters allow customization of build process
5. **Debugging** - Separated steps make it easier to identify issues
