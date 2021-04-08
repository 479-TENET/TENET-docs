===================
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

1. Space-time pairs implying data reuse relation
------------------------------------------------
Assume we want to compute some metrics of input matrix A

First, we need to recongnize which groups of statement instance use the same data for computation.

TENET can find any two statement instances ``S[i][j][k]`` and ``S[i][j'][k]`` accessing the same ``A[i][k]``, and construct a *statement->statement* mapping ``{S[i][j][k]->S[i][j'][k]}``.

| Then TENET constructs a *time->time* mapping to express any two timestamps are continuous, and we denote it ``{T[prev_t]->T[next_t]}``.
| (for example T[0][1]->T[0][2] 00:01->00:02, T[0][59]->T[1][0] 00:59->01:00)
| As well, we construct two Identical mapping: ``{PE[i][j]->PE[i][j]}``, ``{T[t]->T[t]}``.

All possible reuse of one data element can only occur in three circumstance:

1. **Temporal reuse**: one PE access the element in two continuous timestamps.

2. **Spatial reuse**: two PEs connected by boardcast wire access the element at the same time.

3. **Temporal-Spatial reuse**: two PEs connected by sending-forward wire access the element in succession.

Next, we construct a *space-time->space-time* mapping with two *space->space* and *time->time* mapping using the rule below::

    F({space1->space2}, {time1->time2}) => {(space1,time1)->(space2->time2)}

Then apply the rule to different *space->space*, *time->time* mapping pair:

1. F({PE[i][j]->PE[i][j]}, {T[prev_t]->T[next_t]}) => temporal reuse opportunity **R_t**.

2. F(PE Interconnect, {T[t]->T[t]}) => spatial reuse opportunity **R_s**.

3. F(PE Interconnect, {T[prev_t][next_t]}) => temporal-spatial reuse opportunity **R_st**.


2. Reuse volume and reuse factor
--------------------------------
To get exact reuse of input Matrix A, we should map data element of A to corresponding *space-time* pair.

If *data_i->space1-time1*, *data_i->space2-time2* exist in *data->space-time* mapping, s.t.*space1-time1->space2-time2* in **R_t** or **R_s** or **R_st**.

It means that data i accessed by space1 at time1 will be reused by space2 at time2, and TENET denotes occurrence number of such case **temporal/spatial/temporal-spatial reuse volume**.

The total number of data access, which is the volume of *statement instance->data access* mapping, is denoted as **Total data-use volume**.

**Reuse factor** is defined as ``total use volume/(total use volume - total reuse volume)``. 

The larger reuse volume is, the greater reuse factor is. Moreover, Because data must be loaded at least once before being reused, the total reuse volume can never reach total use volume, 
promising the divisor to be positive.

3. PE utilization and time latency
----------------------------------
**Average PE utilization** can be used to assess how many compute resources are active during the computation:``#space-time-pair/#time-domain``.

**Input latency** (ingress delay) is computed by dividing ``ingress_volume`` (UniqueVolume of this input tensor × data size) by ``bandwidth + average latency before emitting first bits``.

**Output latency** (egress delay) is computed by dividing ``egress_volume`` (UniqueVolume of this output tensor × data size) by ``bandwidth + average latency before emitting first bits``.

**Compute latency** is computed by dividing ``#MAC operation`` (#statement instance × #MAC/instance) by ``Average PE utilization``.

Assuming input, output, compute latency are overlapped, then the **overall latency** is ``max(input latency, compute latency, output latency)``.

* Note that although we illustrate some important metric computation process here, you are not bothered to do the complex computation by yourself, as TENET already implemented these stuff for you.