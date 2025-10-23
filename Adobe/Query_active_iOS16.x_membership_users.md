# Active iOS 16 Members (via ECID → Profile → Membership)

- Purpose: Count distinct Active members who used the iOS app on iOS 16.x.
- Method: ECIDs from iOS 16.x events → map ECID to costcomemberid via Profile Snapshot → filter to Active members.
- Datasets:
  - costco_mobilesdk_experience_event_dataset (events, ECID, iOS 16.x)
  - profile_snapshot_export_53562873_daa7_4302_b3b8_a747a4ee850f (identity stitching)
  - costco_membership_custom_class_dataset_updated (membership status/type)
- Result: active_ios16_member_count = 401,043

## SQL
```sql
WITH ios16_ecids AS (
  -- ECIDs seen on iOS 16.x events
  SELECT DISTINCT lower(trim(ec.id)) AS ecid
  FROM (
    SELECT explode(coalesce(identityMap['ECID'], identityMap['ecid'])) AS ec
    FROM costco_mobilesdk_experience_event_dataset
    WHERE environment.operatingsystem = 'iOS'
      AND environment.operatingsystemversion LIKE '16.%'
      AND coalesce(identityMap['ECID'], identityMap['ecid']) IS NOT NULL
  ) s
  WHERE ec.id IS NOT NULL
),
profile_memberids AS (
  -- First explode: get member IDs; carry identityMap forward
  SELECT
    explode(coalesce(identityMap['costcomemberid'], identityMap['costcoMemberId'])) AS pid,
    identityMap
  FROM profile_snapshot_export_53562873_daa7_4302_b3b8_a747a4ee850f
  WHERE identityMap IS NOT NULL
    AND coalesce(identityMap['costcomemberid'], identityMap['costcoMemberId']) IS NOT NULL
),
profile_pairs AS (
  -- Second explode (in a new SELECT): get ECIDs and pair with member IDs
  SELECT DISTINCT
    lower(trim(pm.pid.id)) AS memberid,
    lower(trim(ec.id))     AS ecid
  FROM (
    SELECT pid, explode(coalesce(identityMap['ECID'], identityMap['ecid'])) AS ec
    FROM profile_memberids
    WHERE coalesce(identityMap['ECID'], identityMap['ecid']) IS NOT NULL
  ) pm
  WHERE pm.pid.id IS NOT NULL
    AND ec.id IS NOT NULL
),
active_members AS (
  SELECT DISTINCT lower(trim(CAST(_costco.cc_MemberID AS STRING))) AS memberid
  FROM costco_membership_custom_class_dataset_updated
  WHERE _costco.membership.membershipType IS NOT NULL
    AND _costco.membership.membershipType <> ''
    AND _costco.membership.membershipStatus = 'Active'
)
SELECT COUNT(DISTINCT p.memberid) AS active_ios16_member_count
FROM profile_pairs p
JOIN ios16_ecids e ON p.ecid = e.ecid
JOIN active_members a ON a.memberid = p.memberid;
```
