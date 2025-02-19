# Backup Strategy

The threat model for RAID data storage includes the following:

## Single drive failure

 1. Highly likely
 2. Slow rebuild times for RAID volumes 
 3. Need to source or have a spare drive that is same as the others
 4. Lack of notifications and ability to determine about which drive has failed 

## Complete RAID volume failure

 1. Less likely
 2. Need to source or have spare drives for all drives that have failed
 3. Slow rebuild times to rebuild RAID array
 4. Painfully slow to recover from backups
 5. Need to test recovery to ensure it works
 6. Lack of notifications and ability to determine about which drives have failed 

## Storage appliance or RAID controller failure

 1. Less likely
 2. Extremely painful and costly to recover

 - Mitigated by:
   - Offline cold storage backup
   - Redudant appliance

## Storage appliance theft

 1. Less likley
 2. Need to make sure data is encrypted
 3. Appliance should be secured to prevent easy theft
 4. Need to ensure data is encrypted on drives
 5. Need to ensure appliance is secured to prevent access
 6. Extremely painful and costly to recover
 7. Chance of private data being exposed

 - Mitigated by:
   - Physically secured appliance, bolted down, out of sight
   - Offline cold storage backup
   - Encrypted data / encrypted drives
   - Strong authentication to appliance

## Accidential deletion or change of files

 1. Highly likley

 - Mitigated by:
   - Data snapshots
   - Regular backups

## Malicious deletion or corruption of files

 1. Semi likely
 2. Extremely painful and costly to recover
 3. Chance of private data being exposed

 - Mitigated by:
   - Regular backups
   - Offline cold storage backup
   - Encrypted files on disk to prevent plain text exposure

## Power failure

 1. Less likley

 - Mitigated by:
   - Dedicated UPS for power backups

## Network failure

 1. Less likley

 