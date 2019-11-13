## Tips on using TTCK server (King)

#### Login to node

```unix
qsub -I -l nodes=1:ppn=32
```

#### Delete all runnig qsub commands

```unix
qselect -u <username> | xargs qdel
```
