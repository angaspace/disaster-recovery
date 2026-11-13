# Anga Space Emergency Playbook: EBS Snapshot Failover

## Incident Trigger
Primary EC2 instance failure or storage corruption in Availability Zone A.

## Targets (not guarantees)
- **RTO (Recovery Time Objective):** under 15 minutes, once the standby instance exists and this playbook has been practiced.
- **RPO (Recovery Point Objective):** equal to the age of the latest completed snapshot (for example under 24 hours with daily snapshots).
- Measure real times in a drill before promising these to a client.

## Prerequisites (set up BEFORE an incident)
- Automated snapshots of the volume (daily or better).
- A standby EC2 instance running in a healthy AZ, same OS and application installed.
- Snapshots are encrypted, or the new volume is created with `--encrypted`.
- CLI access with a role that allows `ec2:DescribeSnapshots`, `ec2:CreateVolume`, `ec2:AttachVolume`.

## Variables
```bash
REGION=eu-west-1
SOURCE_VOLUME=vol-xxxxxxx
HEALTHY_AZ=eu-west-1b
STANDBY_INSTANCE=i-zzzzzzz
```

## Rapid Remediation Steps

1. **Find the latest completed snapshot**
```bash
aws ec2 describe-snapshots --region $REGION --owner-ids self \
  --filters Name=volume-id,Values=$SOURCE_VOLUME Name=status,Values=completed \
  --query 'sort_by(Snapshots,&StartTime)[-1].[SnapshotId,StartTime]' --output text
```
Note the snapshot ID (`snap-xxxxxxx`) and its time. That time is your data loss window.

2. **Create a new encrypted volume in the healthy AZ**
```bash
NEW_VOL=$(aws ec2 create-volume --region $REGION --snapshot-id snap-xxxxxxx \
  --availability-zone $HEALTHY_AZ --volume-type gp3 --encrypted \
  --query VolumeId --output text)
aws ec2 wait volume-available --region $REGION --volume-ids $NEW_VOL
```

3. **Attach it to the standby instance**
```bash
aws ec2 attach-volume --region $REGION --volume-id $NEW_VOL \
  --instance-id $STANDBY_INSTANCE --device /dev/sdf
```

4. **Mount the filesystem and resume service** (run on the standby instance)
```bash
lsblk                                  # find the new disk, e.g. nvme1n1
sudo mkdir -p /data
sudo mount /dev/nvme1n1 /data          # use the partition if lsblk shows one (nvme1n1p1)
ls /data                               # verify data is present
sudo systemctl start <application-service>
```
**Do NOT run `mkfs` on this volume. It already has a filesystem and the data.**

5. **Verify and record**
- Test the application from outside (browser or `curl`).
- Confirm the Better Stack monitor returns to up.
- Post the incident time, snapshot time used, and recovery time in Slack `#alerts-critical`.

## After the Incident
- Make the mount permanent: add the volume's UUID (from `sudo blkid`) to `/etc/fstab` with the `nofail` option.
- Investigate the failed instance and AZ. Delete the old volume and unused snapshots when no longer needed.
- Update this playbook with anything that went wrong.

## Practice Drill (sandbox only)
Run these steps with a throwaway instance and volume at least once, and note the real time it took.
