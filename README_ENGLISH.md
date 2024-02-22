Recognizing the limitations of the strategy, I opted to implement "splitting leaf nodes". This caters to scenarios where users anticipate a significant amount of conflict due to high workloads, especially when internal structural changes might exceed the project's deadline. While I acknowledge that splitting leaf nodes is not a fundamental solution, its implementation and performance comparison offer valuable insights.

## Problem Definition
#### Problem situation
In services requiring intensive modification of specific key information, such as concert ticketing where thousands of users access a specific seat, response time becomes crucial for a satisfactory user experience. The issue is evident in [Table-1], where we see significant delays in response times when congestion occurs at 64 elements.
![image](https://user-images.githubusercontent.com/96645965/216049878-15744ca6-9f01-40a6-a94e-8281c5cc9879.png)
[Table-1]
#### [Table-1] analysis
This table presents experimental results examining the bw-tree's ability to handle a large number of modifications on a narrow key range. The experiment involved creating a bw-tree with 1,024x1,024 elements and having 20 threads each attempt to access a narrow range of keys 1,024x1,024 times. The operations included deleting and then inserting an element with a specific key. We measured the total time consumed and the success ratio of these access operations.

#### Key Observations
Each leaf node in our 1,024x1,024-element bw-tree contained 64 elements. The operations involved traversing the tree structure and appending delta information to the leaf node if the number of deltas exceeded 8. We observed a positive correlation between time consumption and success ratio when the number of available keys ranged from 2 to 64. This suggests that as the congestion degree of a node increases, the overhead for modifications also increases linearly. However, when the number of available keys ranges from 64 to 1,024x1,024, the success ratio remains similar, and performance improves as the congestion degree decreases.

## New Version Design
#### Concept
The new design is akin to adding new roads to alleviate traffic congestion. By splitting a congested leaf node, we can create new paths to the same key range, improving overall response times. This design assumes that users can predict future workloads' congestion levels and choose to activate the splitting option for better performance.

#### Implementation
A boolean member, congestion_control, has been added to the class tree. Users can activate the new policy by setting tree->congestion_control to true.

#### Design Rational and Corretness
The new splitting policy requires criteria for identifying congestion levels. We have introduced new tables (success_count, op_count, op_base, and node_flag) alongside the existing mapping_table. These tables count operation attempts and successes for each leaf node, providing data to assess congestion levels. The decision to split or not is based on a success ratio threshold of 90%. Additionally, we use node_flag as an indicator to prevent merging of certain nodes.

### custom test cases
I developed specific test cases in 'bwtree_test_density'.
- BwtreeTest_db_init: Measures time to create a bw-tree structure with 1M keys.
- BwtreeTest_density_with_exist_db: Measures modification time within a given key range.
- BwtreeTest_density_with_exist_db_with_split_logic: Similar to the above but includes the split logic.

## Performance analysis
[Table-2] shows the performance differences between the old and new versions, highlighting the efficacy of the split logic in managing key congestion. 
![image](https://user-images.githubusercontent.com/96645965/216049921-7d82db04-03f3-40b3-8ca7-34f1c4d63e6c.png)
[Table2]   
