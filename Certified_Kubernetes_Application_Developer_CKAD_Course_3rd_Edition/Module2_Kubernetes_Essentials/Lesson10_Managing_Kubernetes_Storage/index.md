# Lesson 10: Managing Kubernetes Storage

## Overview
This lesson covers Kubernetes storage concepts and how to manage persistent data in your applications. You'll learn about different storage options, how to configure persistent storage, and best practices for managing data in a Kubernetes environment.

## Learning Objectives
- Understand Kubernetes storage architecture
- Configure different types of volumes
- Work with PersistentVolumes (PV) and PersistentVolumeClaims (PVC)
- Implement StorageClasses for dynamic provisioning
- Manage storage for stateful applications
- Troubleshoot storage-related issues

## Lessons

### [10.1 Understanding Kubernetes Storage Options](10_1_Understanding_Kubernetes_Storage_Options/10_1_Understanding_Kubernetes_Storage_Options.md)
- Ephemeral vs. persistent storage
- Volume types and use cases
- Storage architecture components
- Container Storage Interface (CSI)

### [10.2 Configuring Pod Volume Storage](10_2_Configuring_Pod_Volume_Storage/10_2_Configuring_Pod_Volume_Storage.md)
- Volume types overview
- Configuring emptyDir volumes
- Using hostPath volumes
- Volume lifecycle

### [10.3 Configuring PV Storage](10_3_Configuring_PV_Storage/10_3_Configuring_PV_Storage.md)
- PersistentVolume concepts
- Static provisioning
- Access modes
- Reclaim policies

### [10.4 Configuring PVCs](10_4_Configuring_PVCs/10_4_Configuring_PVCs.md)
- PersistentVolumeClaim basics
- Storage requests and limits
- Binding modes
- Access control

### [10.5 Configuring Pod Storage with PV and PVC](10_5_Configuring_Pod_Storage_with_PV_and_PVC/10_5_Configuring_Pod_Storage_with_PV_and_PVC.md)
- Connecting Pods to persistent storage
- Volume mounting options
- SubPath usage
- Read-write-once vs. read-write-many

### [10.6 Understanding StorageClass](10_6_Understanding_StorageClass/10_6_Understanding_StorageClass.md)
- Dynamic provisioning
- StorageClass parameters
- Default StorageClass
- Volume binding modes

## Hands-on Exercises
1. Create and use an emptyDir volume
2. Configure a Pod with hostPath volume
3. Set up static provisioning with PV and PVC
4. Implement dynamic provisioning with StorageClass
5. Mount the same volume to multiple containers
6. Troubleshoot storage attachment issues

## Storage Patterns
- **Shared Storage**: ReadWriteMany volumes
- **Database Storage**: StatefulSets with persistent storage
- **Configuration Data**: ConfigMaps and Secrets as volumes
- **Temporary Storage**: emptyDir for caching

## Best Practices
- Use StorageClass for dynamic provisioning
- Implement proper access modes
- Set appropriate storage sizes
- Monitor storage usage
- Implement backup strategies
- Secure sensitive data

## Common Issues and Solutions
- **Volume Mount Fails**: Check volume existence and permissions
- **PVC Pending**: Verify storage class and capacity
- **Data Loss**: Understand reclaim policies
- **Performance Issues**: Choose appropriate storage backend

## Next Steps
After mastering Kubernetes storage, proceed to [Lesson 11: Managing ConfigMaps and Secrets](../Lesson11_Managing_ConfigMaps_and_Secrets/index.md) to learn how to manage configuration data and sensitive information in your applications.