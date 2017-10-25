# Bulk PVC Delete Test

## Test Env.
1 master, 1 infra, 1 compute: m4.4xlarge
3 cns, 1 heketi: m4.xlarge

## Pre-actions

Move pods to desired nodes and label the nodes as described [glusterFS_stress.md](glusterFS_stress.md).


## Run the test


## PVC delete test

### Results

#### oc (v3.7.0-0.143.7 + ol2 + sc)

glusterfs (3.3.0-12) and heketi (3.3.0-9)

| #PVC | 10  | 20 | 30 | 50 | 100 | 200 | 250 |
|------|-----|----|----|----|-----|-----|-----|
| #sec | 94  |    |    |    |     |     |     |
| avg  | 9.4 |    |    |    |     |     |     |

gp2

| #PVC | 10  | 20 | 30 | 50 | 100 | 200 | 250 |
|------|-----|----|----|----|-----|-----|-----|
| #sec | 6  |  5  |    |    |     |     |     |
| avg  | 0.6 |  0.25  |    |    |     |     |     |


