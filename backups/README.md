# Backup Strategy

The threat model for RAID data storage includes the following:

## Single drive failure

 1. Highly likely
 2. Slow rebuild times for RAID volumes 
 3. Need to source or have a spare drive that is the same as the others
 4. Lack of notifications and ability to easily determine which drive has failed 

## Complete RAID volume failure

 1. Less likely
 2. Need to source or have spare drives for all drives that have failed
 3. Extremely slow rebuild times for the entire RAID array if sufficently large
 4. Painfully slow to recover from backups
 5. Need to test recovery process to ensure it works
 6. Lack of notifications and ability to easily determine which drives have failed 

## Storage appliance or RAID controller failure

 1. Less likely
 2. Extremely painful and costly to recover

 - Mitigated by:
   - Offline cold storage backup
   - Redudant appliance

## Storage appliance theft

 1. Less likley
 2. Extremely painful and costly to recover
 3. Data compromise / leak
 4. Chance of private data being exposed

 - Mitigated by:
   - Physically secured appliance, bolted down, out of sight
   - Offline cold storage backup
   - Data needs to be encrypted on disk to prevent plain text exposure
   - Disks should have device level encryption
   - Strong authentication to appliance

## Accidential deletion or change of files

 1. Highly likley

 - Mitigated by:
   - Data snapshots
   - Regular backups
   - Offline cold storage backup

## Malicious deletion or corruption of files

 1. Semi likely
 2. Extremely painful and costly to recover
 3. Data compromise / leak
 4. Chance of private data being exposed

 - Mitigated by:
   - Regular backups
   - Offline cold storage backup
   - Data needs to be encrypted on disk to prevent plain text exposure

## Power failure

 1. Less likley

 - Mitigated by:
   - Dedicated UPS for power backups

## Network failure

 1. Less likley


Copyright 2025 Bret Jordan, All rights reserved.