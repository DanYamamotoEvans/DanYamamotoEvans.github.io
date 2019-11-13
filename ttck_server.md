## Tips on using TTCK server (King)

#### Login to node

```unix
qsub -I -l nodes=1:ppn=32
```

#### Delete all runnig qsub jobs

```unix
qselect -u <username> | xargs qdel
```
