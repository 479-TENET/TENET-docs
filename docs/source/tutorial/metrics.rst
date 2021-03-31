Metrics Computation
===================

We continue to use GEMM for two 512*512 matrix as example::

    Statement:
    {S[i][j][k]: 0<=i<512 and 0<=j<512 and 0<=k<512}
    {S[i][j][k]->A[i][k]}
    {S[i][j][k]->B[k][j]}
    {S[i][j][k]->C[i][j]}
    
    PE Interconnect:
    {PE[i][j]: 0<=i<8 and 0<=j<8}
    {PE[i][j]->PE[i+1][j]; PE[i][j]->PE[i][j+1]}

    Mapping:
    {S[i][j][k]->PE[i%8][j%8]}
    {S[i][j][k]->T[floor(i/8)][floor(j/8)][floor(k/8)][i%8+j%8+k%8]}


Assume we want to compute some metrics of input matrix A
First, we need to recongnize which groups of statement instance use the same data for computation.
TENET can find any two statement instances S[i][j][k] and S[i][j'][k] access the same A[i][k], and construct a *statement->statement* mapping {S[i][j][k]->S[i][j'][k]}

Then TENET constructs a *time->time* mapping to express any two timestamps are continuous, and we denote it *{T[prev_t]->T[next_t]}* (for example T[0][1]->T[0][2], T[0][59]->T[1][0] are two continuous pair if T[i][j] denotes minute j, hour i)
As well, we construct two Identical mapping: {PE[i][j]->PE[i][j]}, {T[t]->T[t]}

All possible reuse of one data element can only occur in three circumstance::

    1. Temporal reuse: one PE access the element in two continuous timestamps
    2. Spatial reuse: two PEs connected by boardcast wire access the element at the same time
    3. Temporal-Spatial reuse: two PEs connected by sending-forward wire access the element in succession

Next, we construct a (space,time)->(space,time) mapping with two space->space and time->time mapping using the rule below:
F({space1->space2}, {time1->time2}) => {(space1,time1)->(space2->time2)}

Then apply the rule to different space->space, time->time mapping pair:
1. F({PE[i][j]->PE[i][j]}, {T[prev_t]->T[next_t]}) => temporal reuse opportunity.
2. F(PE Interconnect, {T[t]->T[t]}) => spatial reuse opportunity.
3. F(PE Interconnect, {T[prev_t][next_t]}) => temporal-spatial reuse opportunity.

To get exact reuse of input Matrix A, we should 