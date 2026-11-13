# EC2 Storage Audit Commands (Anga Space)

Read-only commands. Replace `REGION` with the client's region, for example `eu-west-1`.
Only run these on an account you have written permission to audit.

## Unattached EBS volumes (wasted spend)
```bash
aws ec2 describe-volumes --region REGION \
  --filters Name=status,Values=available \
  --query 'Volumes[*].[VolumeId,Size,VolumeType,AvailabilityZone]' --output table
```

## Unencrypted EBS volumes
```bash
aws ec2 describe-volumes --region REGION \
  --filters Name=encrypted,Values=false \
  --query 'Volumes[*].[VolumeId,Size,State]' --output table
```

## Snapshots owned by this account
```bash
aws ec2 describe-snapshots --region REGION --owner-ids self \
  --query 'Snapshots[*].[SnapshotId,VolumeId,StartTime,Encrypted]' --output table
```

## Is encryption by default enabled?
```bash
aws ec2 get-ebs-encryption-by-default --region REGION
```
`"EbsEncryptionByDefault": true` is the good result.

## EFS file systems
```bash
aws efs describe-file-systems --region REGION \
  --query 'FileSystems[*].[FileSystemId,Encrypted,AvailabilityZoneName]' --output table
```
An empty `AvailabilityZoneName` means Regional. A value means One Zone, which is a risk for production.

## Tested
Not yet. Update this line with the date and environment after a sandbox run.
