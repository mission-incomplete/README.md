# README.md

 CSE 520 PROJECT REPORT

Name :Jaya Shankar Nalanagula

ASU ID: 1222514046
 Markup : * Branch Prediction attempts to guess whether a conditional jump will be taken or not. 
              * Nested bullet
                  * Sub-nested bullet etc
          * Branch Prediction attempts to guess whether a conditional jump will be taken or not. 

-OR-

 Markup : - Bullet list
              - Nested bullet
                  - Sub-nested bullet etc
          - Bullet list item 2 
**Branch Prediction**:

*Branch Prediction attempts to guess whether a conditional jump will be taken or not. 
*The purpose of the branch prediction is to improve the flow in the instruction pipeline. 
*Branch Predictors play an important role in achieving high effective performance in many modern pipelined microprocessor architectures. 
*Without branch prediction, the processor would have to wait until the conditional jump instruction has passed the execute stage before the next instruction can enter the fetch stage in the pipeline. 
*The branch predictor attempts to avoid this waste of time by trying to guess whether the conditional jump is most likely to be taken or not taken.

**2-bit Local Predictor**: 

*It is a dynamic branch predictor and uses information about taken or not taken branches gathered at run-time to predict the outcome of a branch.
*A 2bit local counter makes use of a 2 bit saturating counter to make prediction. It works as a state machine with four states: 00 (Strongly not taken), 01 (Weakly Not taken), 10 (Weakly taken), 11 (Strongly Taken). 
*It takes two mispredictions  for the predictor to change its prediction. A second check is made to make sure that a short & temporary change of direction does not change the prediction away from the dominant direction.
*It has a separate history buffer for each conditional jump instruction, while the pattern history table may be separate as well or it may be shared between all conditional jumps.
*The 2-bit local predictor table is indexed with the instruction address bits, so that the processor can fetch a prediction for every instruction before the instruction is decoded.

**Tournament Predictor**: 
*Tournament predictor is a hybrid predictor as it implements more than one prediction mechanism. The final prediction is based either on a meta-predictor that remembers which of the predictors has made the best predictions in the past, or a majority vote function based on an odd number of different predictors.
*Tournament predictor uses multiple predictors, usually one based on global information and one based on local information, and combining them with a selector. Tournament predictors can achieve both better accuracy at medium sizes (8K–32K bits) and also make use of very large numbers of prediction bits effectively.
*Tournament predictors use a 2-bit saturating counter per branch to choose among two different predictors based on which predictor (local, global) was most effective in recent predictions. As in a simple 2-bit predictor, the saturating counter requires two mispredictions before changing the identity of the preferred predictor.
*It makes correlated Prediction based on the last m branches, assessed by the global history.

**Global History Divide And Conquer(gDAC) Predictor**:
*partition the branch history register into smaller segments, each of which can be handled by a small, implementable PHT. 
*A final table-based predictor combines all of these per-segment predictions to provide the overall decision.
*It divides the long global branch history register into non-overlapping segments. 
*Each segment provides a branch history input to a PHT-based predictor that provides a branch prediction for that segment. 
*A final root predictor takes all of the per-segment predictions as part of its input vector and computes the final prediction.

The per-segment predictions form an index that selects a counter from a line of the prediction fusion table. gDAC root predictor is a simple fusion predictor. The most recent global history is hashed with the branch address, and then concatenated with the per-segment predictions to form a final index. The index selects a 4-bit counter from a standard PHT structure where the most significant bit of the counter determines the final prediction. 

**Implementation**:
*A class named gDACBP inheriting base predictor class is created in BranchPredictor.py and parameters like global predictor size, fusion predictor size, choice predictor size, and counter bits for the three predictors, segment sizes and global history register bits.
*Two files named gdac.hh and gdac.cc are added in the gem5/src/cpu/pred folder. gdac.hh file is same as the bimode.hh file but with few additions of new declared variables and functions as we are building gDAC on the concept of Bimode Predictor.
*In gdac.cc file, the parameters given in the BranchPredictor.py file are linked in the constructor. In lookup function, the segments are extracted from the GHR. Indices to per segment Global Predictors, choice predictor and Fusion predictor are calculated using hashing. It also uses a new function hash() to calculate indices. 
*Using the global history indices, we access the per segment global predictors and get prediction using the direction bit. The hysteresis bit tell whether the prediction was strong or weak. The choice predictor (accessed using the choicehistory index) selects a prediction from taken and not taken of each global predictor and gives a final prediction from each global predictor.
*The final prediction bits of each global predictor and the PC and GHR to form the index to the Fusion predictor which gives the Final Prediction as to whether the branch should be taken or not using the direction bit of the 4 bit counter.
*The update function updates the counter values of choice predictor, global predictors and the fusion predictor depending on whether it was a misprediction or not. 
*Btbupdate function is called only if there is a miss in Branch Target Buffer (BTB). It means the branch prediction does not know where to jump even though the predictor can accurately predict the branch outcome. In this case, you will predict a branch as not taken so that the target branch address is not required. Thus, this function is typically to correct what you did in the lookup function into a result that you predict a branch as non-taken.
*If there is a branch misprediction, everything changed (counters, registers, cache, memory, etc) due to the outstanding instructions after this branch need to be reverted back into the state before the branch instruction was issued. This includes those victim branch instructions following the branch misprediction. Squash function is primarily called to erase what the outstanding branches did to those history counters by restoring them with the backup value stored in update function.

Hysteresis is also taken care of by adding a gdac_counter.hh in satcounter.hh file which increments and decrements the counter values of taken and not taken counters accordingly. 

**Observations**:
From the above tables and graphs following observation can be made:
*	Local Predictor accuracy is over 95% and IPC is in the range of 0.8 to 1. It’s accuracy doesn’t change significantly for the three benchmarks.
*Tournament predictor shows slightly better performance than the 2 bit local predictor for both hardware budgets. Tournament Predictor shows best performance for MST_opt benchmark.
*The gDAC predictor shows lower accuracy and IPC as compared to 2 bit local and Tournament predictors. This is because ahead pipelining isn’t implemented in this gDACBP class which is an important feature of gDAC predictor. Implementing ahead pipelining (making predictions 3 cycles ahead) would prominently increase accuracy and IPC. 
*The implementation considering the direction and hysteresis bits showed an increase in IPC and accuracy of gDAC predictor as compared to the normal gDAC implementation.
