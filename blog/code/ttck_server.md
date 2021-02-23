## Tips on using TTCK server (King)

#### Login to node

```unix
qsub -I -l nodes=1:ppn=32
```

#### Delete all runnig qsub jobs

```unix
qselect -u <username> | xargs qdel
```

#### Using Jupyter notebook on TTCK server

##### On the ssh server side;

- Install Jupyter notebook as:

```
pip install notebook
```

- Create a config file for jupyter notebook

```
jupyter-notebook --generate-config
```
This will give you a config fule at: 
```$HOME/.jupyter/jupyter_notebook_config.py
```
- Find comments with the following *c.NotebookApp.~*, and uncommnet/edit as follows;
```
c.NotebookApp.ip = 'localhost'
c.NotebookApp.open_browser = False
c.NotebookApp.port = 8889
c.NotebookApp.token = u''
```

##### On the local side;

When you conect to the ssh server, add the following to conect local host port 8889 to the 8889 port of the server.
```
ssh -L 8889:localhost:8889 [server]
```

Open jupyter-notebook on server while maintining this connection.

Enter the URL in your browzer.
