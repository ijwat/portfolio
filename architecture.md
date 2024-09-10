
### Technical Writing Project

This is my Technical Writing Project. This focuses on NetApp's FabricPool Architecture documentation. 


Understanding FabricPool's Architecture

Key components for FabricPool's architecture are object storage, block temperature, and cloud tiering licenses. The architecture is designed to optimize storage instructure based on changing needs.

FabricPool Architecture Overview:
---------------------------------

-   FabricPool is a feature in ONTAP that connects a local storage tier with a cloud tier. The cloud tier is an external object storage. Both of these tiers make a FabricPool, letting volumes within the tiers manage data efficiently and promptly. Hot(active) data stays in the local tier for high performance, while cold(inactive) data is moved to the cloud tier.

-   Understanding FabricPool is essential to building effective storage solutions with ONTAP.

Block Temperature 101:
----------------------

-   Blocks are assigned a temperature when written to the local tier(hot).

-   A background cooling scan cools blocks over time.

-   NOTE - The All volume tiering policy is an exception to this. Blocks in volumes using the all tiering policy are immediately identified as cold and marked for tiering.

Object Creation 101:
--------------------

-   FabricPool works at the WAFL block level, making 4MB objects from cooled blocks and sending them to the cloud tier.

-   Each object is made up of 1,024 4KB blocks.

-   The fixed object size is 4MB, but actual sizes may be less due to ONTAP storage efficiencies.

Understanding Data Movement 101:
--------------------------------

-   Cold blocks marked for tiering are collected into 4MB objects and moved to the cloud tier.

-   The tiering fullness threshold decides when tiering happens, with the default set at >50% local tier capacity. (In ONTAP 9.5, the 50% tiering fullness threshold can be changed).

-   Write-back prevention occurs if the local tier is >90% full. This affirms that the local tier is used for active data.

-   ![](https://lh7-us.googleusercontent.com/6AdU7SXxmSf8Fi8aYenORRMaGoHdOORD0W1_UjdldH8tuJ-yIPJdYgc6XTSIrc-4U-bYoPqqp6qpXodBcnif1V_UdFHxJIEMJbPX4aUksctqUSwjTG9hI12gL0Kw1O-MX1PpcD4eg8Z6ssRPg4XPbik)

-   ![](https://lh7-us.googleusercontent.com/phjnGEzKwK5Kg-O3CUpXGE-RB04tz9y6MDzfiMbMLv9vtmF_vMHyqUEh8LA3k3K-Rl_hop04WG0NTvqZesW2pZWMjk2JJWWqfqsjZPe9czZ8mu2OTsDqySynKNJ2kUNBF2cERubfKgYGp925iN7SXqo)

SnapMirror Behavior:
--------------------

-   SnapMirror controls the movement of data between the local and cloud tiers based on tiering policies as seen in Table 1.

-   ![](https://lh7-us.googleusercontent.com/Cuc2MnQ7m6LKvr3o4Z5QvvOIzWZQxgHx4B6mg5ZjqCJOB-SXhL7qQGcedVvkqHJCYzwLUKEoPlhns7gc6DgBESzhTIR_mlkiUzo3Fwb4aQyUl45wsi8Lc1d9AOy47nf-1JhhEGGQYEH-Gd4AmjOahNY)

Volume Move:
------------

-   Volume move(vol move) is a non-disruptive function that moves a volume from one local tier(source) to another destination. 

### Destination Local Tier:

-   If the volume's destination local tier does not have a connected cloud tier, then data on the source volume's cloud tier moves to the local tier on the destination.

-   Starting with ONTAP 9.6, data in the same bucket doesn't move back which promotes network efficiency. 

-   NOTE - Some configurations are incompatible with optimized volume moves:

-   Changing tiering policy during volume move

-   Source and destination aggregates use different encryption keys

-   FlexClone volumes

-   FlexClone parent volumes

-   MetroCluster

-   Unsynchronized FabricPool Mirror buckets

-   Different Amazon S3 naming formats for source and destination aggregates

### Destination Local tier with a  Cloud tier:

-   If the destination local tier has an attached cloud tier, Cloud-tiered data is written to the local tier first, then to the cloud tier.

-   This enhances volume move performances and significantly reduces cutover time.

Volume Tiering Policy:
----------------------

-   If no tiering policy is indicated, the destination volume uses the source volume's policy.

-   Different policies can be specified during volume move, impacting destination volume creation.

-   Source and destination volumes in SVM DR are required to use the same tiering policy.

### Minimum Cooling Days:

-   In Pre-ONTAP 9.8, moving a volume resets block inactivity period on the local tier. For example, a volume using the auto volume tiering policy with data on the local tier that has been inactive for 20 days has data inactivity reset to 0 days after a volume move.

-   In ONTAP 9.8+, blocks inactive longer than destination volume's minimum cooling days are tiered. Blocks with less inactivity reset to 0 days.

### How to configure Tiering Policies:

-   The Auto tiering policy moves all data to the local tier  first:

-   ![](https://lh7-us.googleusercontent.com/92GV7AHOhnuixCnkvHpPLIyUqFYD2ZjULdSgRRJdy94Ydzm2nPEeT1hvxQJptg3hdCulEe9qQZK23J3R0yf0_u9TaL90XW2hKp5bJw7Ai33vCO7wwWcXYnzLukL9GdzVMKhl49Nw69N6zJHq04B0nqY)

-   The Snapshot-Only tiering policy moves data variably, prioritizing the local tier. COld Snapshot blocks move to the cloud tier:

-   ![](https://lh7-us.googleusercontent.com/GrCiovM2xwBoWQvBxMs3mlSwvt3JEoMSWfv2ze9my9egGt-2ob6u87y35MEcuLLrvbiXX2P0Cdiuq5aMAQ5iNCwDp0DhrKWjRWnoyD54uk-CqfuIL3DoXvo9i-RGq76gul81Y8InlpQOmY7ZNVt73x8)

-   The All tiering policy immediately identifies cold data, writing to the cloud tier: 

-   ![](https://lh7-us.googleusercontent.com/oIMJlAfJtfyH4aXgcMMH_qjyqIx5CV0962ovvFlSm7Gu1ub26UP5z-C-Sg8Nh_ut9pB7DD6XjzqPwFbeSqdDusLqMFaD0XtXJyKMzVK19NALR1-Iy56l_He9obTFA0cXm_EE35eOR9AP0lpec6YqPrQ)

-   The None tiering policy writes data to the local tier:

-   ![](https://lh7-us.googleusercontent.com/zHp-2Dsq7-OmTDHXQY59s11zF0sxc0kSSbt6rqqYFLISHjyQAuQ-yaD_DV_5tXHurSdWPoQs1MhLfJrFczjN8FZvTTVQCC9AdKpJqiRr9OzJEzZ_uY9aHkA06A7y0FDsxwOzNLaYAs-18PQjCEYhadc)

### How to perform a volume move with ONTAP System Manager:

1.  To perform a volume move with ONTAP system manager, follow these steps:

2.  Click STORAGE  

3.  Click Volumes 

4.  Select the volume you want to move 

5.  Click More 

6.  Click Move 

7.  Select a destination local tier 

8.  Selecting a tiering policy 

9.  Click Move as seen in Figure 6.

10. ![](https://lh7-us.googleusercontent.com/e1U3Q8mtCQq8AM-a3CZum1s6VQjSm6TAW09n1VU3ZJ3ggmq3EwqkZb-3lzBHkFQPQfsM6pcdHf4LdCVqvy-NOo5a6nKYXB9mvw1bMi2S2dyCQ9ANfVFhDBDy1g82MgZC1QeM1mM59MqkJUWs2pkUvgc)

### How to perform a volume move using ONTAP CLI:

-   To perform a volume moving using the ONTAP CLI, run the following command:

-   ![](https://lh7-us.googleusercontent.com/Cq4lp7NOQdWfopCPhg_eSJEOuvUu92F9nRiA_7Wsx2xhRkpEMKMBccx3W2NIH3i0oJoQKbD3ocP1NJLSBi0ctLF1qv_Urmuue6SmnpOSHgRcUEJm2w8qHZAmzCrTEq0vtMNT87s3wBoZBKlTVuI4sfQ)

### FlexClone volumes:

-   FlexClone volumes are copies of a parent FlexVol volume. These copies inherit the volume tiering policy and the tiering-minimum-cooling days settings of the parent FlexVol volume.

-   NetApp recommends using tiering settings on the parent FlexVol that are either equal to or less aggressive than any of the clones. As a best practice, this keeps more data owned by the parent volume on the local tier, increasing the performance of the clone volumes.

-   If a FlexClone volume is split (volume clone split) from its parent volume, the copy operation writes the FlexClone volume's blocks to the local tier.

### FlexGroup volumes:

-   A FlexGroup volume is a single namespace that is made up of multiple constituent member volumes but is managed as a single volume. Individual files in a FlexGroup volume are allocated to individual member volumes and are not striped across volumes or nodes.

-   FlexGroup volumes are not constrained by the 100TB and two-billion file limitations of FlexVol volumes. Instead, FlexGroup volumes are only limited by the physical maximums of the underlying hardware and have been tested to 20PB and 400 billion files. Architectural maximums could be higher.

-   Volume tiering policies apply at the FlexGroup volume level.

-   FabricPool Provisioning is recommended for each cluster node but not mandatory.

Object Storage:
---------------

-   Object storage manages data as objects in containers or buckets.

-   Objects are put inside a container and are not nested as files inside a directory.

-   Object storage offers less performance than file or block storage.

-   This offers scalability beyond traditional file or block storage.

### FabricPool Cloud Tiers:

-   FabricPool supports various cloud providers for object storage like Amazon, Google, IBM, etc for cloud tiers.

-   More than one type of cloud tier can be used in a cluster.

-   In ONTAP 9.7+, FabricPool mirror enables the attachment of two cloud tiers to one single local tier.

ONTAP S3:
---------

-   ONTAP 9.8+ supports tiering to buckets created using ONTAP S3.

-   When tiering more than 300TB of inactive data, NetApp recommends using StorageGRID.

### Object Deletion and Defragmentation:

-   FabricPool deletes entire objects based on unreferenced blocks.

-   For example, there are 1,024 4KB blocks in a 4MB object tiered to Amazon S3. Defragmentation and deletion do not occur until less than 205 4KB blocks (20% of 1,024) are being referenced by ONTAP. When enough (1,024) blocks have zero references, their original 4MB objects are deleted, and a new object is created.

-   You can change the percentage.

-   ![](https://lh7-us.googleusercontent.com/k55GaaPMdZLRXLfGr2h1kYgB5XH-g0mBsGVOGzm0kIf0r0UXUzAMXBsJm6QLYKGm-I5O5dYROlbfsd0yikrCMrRpxt89LE-UkP1e5w_gfZVZDgVv7Li630EJpt4XcnITcO23U_9VWG26KvpLhhJZBfA)

-   Alternatively, consider increasing unreclaimed space thresholds if object fragmentation causes significantly more object store capacity to be used than necessary for the data being referenced by ONTAP. For example, using an unreclaimed space threshold of 20% in a worst-case scenario where all objects are equally fragmented to the maximum allowable extent means that it is possible for 80% of total capacity in the cloud tier to be unreferenced by ONTAP. For example:

-   2TB referenced by ONTAP + 8TB unreferenced by ONTAP = 10TB total capacity used by the cloud tier.

-   ![](https://lh7-us.googleusercontent.com/1IqnqWuyqiIiPOM6Bkb1iLUr6vAO0z85ITjDEXscRPa_f5EPY7e-fDp3295OzrHqkf28lJq3C0dlD-J2fV5DeYqDWjpNwxDWG7uJvy9tdhLx_ycfc5tm703AUvcpPlPMBU1ElcTVVv0YMt5apwO8W_U)

### ONTAP storage efficiencies:

-   Compression, deduplication, and compaction are preserved when moving data to the cloud.

-   NOTE - Third-party deduplication has not been qualified by NetApp.

### Temperature-Sensitive Storage Efficiency(TSSE):

-   Beginning with ONTAP 9.8+, compression adjustments are based on data temperature.

-   Supported on FabricPool-enabled local tiers with ONTAP 9.10.1+

-   TSSE scans the temperature of the data and compresses larger or smaller blocks of data accordingly.

-   TSSE makes data storage more efficient.
