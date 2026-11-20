# Data Version Control (DVC) Tutorial

## Introduction

DVC is a CLI-based tool designed with small- to medium-scale research
and data science in mind.  It aims to solve several problems we care
about:

1. Storing data which is too large for traditional version control
   (git) alongside other files in a repo
2. Tracking versions of large data files over time, synchronized with
   the rest of the repo
3. Capturing dependency relationships between artifacts, so that
   out-of-date downstream data can be re-computed
4. Caching the inputs and outputs of large-data processing steps (3)
   to avoid wasteful re-computation

## Tutorial

### 1. Install DVC (and other tools used in this tutorial)

1. [DVC](https://doc.dvc.org/install)
   - Recommend the `pipx` method: `pipx install dvc`
2. [DuckDB](https://duckdb.org/docs/2.0/installation?platform=linux&environment=cli)
   - Note: I built this tutorial with duckdb version 1.5.6 (not the latest), so YMMV
3. [Kuva](https://github.com/Psy-Fer/kuva) (`cargo install kuva --features cli`)
   - [Rust/Cargo](https://rust-lang.org/tools/install/)

### 2. Create a repo

```shell
cd
mkdir -p tutorials/docclab_dvc_tutorial
cd tutorials/docclab_dvc_tutorial
git init .
```

### 3. Grab a source dataset

In this tutorial we will be working with an English language word
dictionary.  First we will create a directory in our repo for the
source data, then use curl to download it.

> Note: current directory should be the git repo you created in step 2

```shell
mkdir source_data
curl https://storage.googleapis.com/docclab-turtles-datasets/dictionary.txt -o source_data/dictionary.txt
head source_data/dictionary.txt
```

Example output:

```
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100  2.37M 100  2.37M   0      0  2.61M      0                              0

A
a
aa
aal
aalii
aam
Aani
aardvark
aardwolf
Aaron
```

> **Be sure to check the downloaded size and the output of head to
> make sure the file arrived ok!**

### 4. Initialize DVC in our repo

DVC needs some basic metadata files to track the state of the
artifacts it manages.  Let's create them now:

> Note: current directory should be the git repo you created in step 2

```shell
dvc init .
```

Example output:

```
Initialized DVC repository.

You can now commit the changes to git.

+---------------------------------------------------------------------+
|                                                                     |
|        DVC has enabled anonymous aggregate usage analytics.         |
|     Read the analytics documentation (and how to opt-out) here:     |
|             <https://dvc.org/doc/user-guide/analytics>              |
|                                                                     |
+---------------------------------------------------------------------+

What's next?
------------
- Check out the documentation: <https://dvc.org/doc>
- Get help and share ideas: <https://dvc.org/chat>
- Star us on GitHub: <https://github.com/treeverse/dvc>
```

### 5. Add our source data to DVC

> Note: current directory should be the git repo you created in step 2

Now we add `dictionary.txt` to DVC:

```shell
dvc add source_data/dictionary.txt
```

To see what has actually changed in our local repo, run `git status`:

```shell
git status
```

Example output:

```
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   .dvc/.gitignore
	new file:   .dvc/config
	new file:   .dvcignore

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	source_data/
```

Let's add the working tree to git:

```shell
git add .
git status
```

Example output:

```
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   .dvc/.gitignore
	new file:   .dvc/config
	new file:   .dvcignore
	new file:   source_data/.gitignore
	new file:   source_data/dictionary.txt.dvc
```

Note, `dictionary.txt` has *not* been staged; instead the `dvc add` command has created a `.gitignore` file under `source_data/` and populated it with the dataset:

```shell
ls -al source_data/
```

Example output:

```
total 2444
drwxr-xr-x 1 tastolfi tastolfi      84 Nov 19 09:53 .
drwxr-xr-x 1 tastolfi tastolfi      58 Nov 19 09:51 ..
-rw-r--r-- 1 tastolfi tastolfi 2493885 Nov 19 09:53 dictionary.txt
-rw-r--r-- 1 tastolfi tastolfi      97 Nov 19 09:53 dictionary.txt.dvc
-rw-r--r-- 1 tastolfi tastolfi      16 Nov 19 09:53 .gitignore
```

```shell
cat source_data/.gitignore
```

Example output:

```
/dictionary.txt
```

Notice that another file has been added: `dictionary.txt.dvc`.  What's in this file?

```shell
cat source_data/dictionary.txt.dvc
```

Example output:

```
outs:
- md5: f920f969528c6adf9fec6e803470083d
  size: 2493885
  hash: md5
  path: dictionary.txt
```

This is called a DVC _tracking file_.  Because it contains the size,
name, and a hash of the contents of the file it tracks, it acts as a
surrogate or proxy for the larger `dictionary.txt`.  The tracking file
is what Git sees and stores the history of; DVC's first important job
is to map the tracking files to the full data.

It's time to commit everything!

```shell
git commit -m 'Add some data!'
```

Example output:

```
 5 files changed, 12 insertions(+)
 create mode 100644 .dvc/.gitignore
 create mode 100644 .dvc/config
 create mode 100644 .dvcignore
 create mode 100644 source_data/.gitignore
 create mode 100644 source_data/dictionary.txt.dvc
```

### 6. Move the data into DuckDB so we can analyze it

> Note: current directory should be the git repo you created in step 2

```shell
mkdir -p db
duckdb db/dictionary.duckdb -c "create table words (word varchar); insert into words from read_csv('source_data/dictionary.txt');"
```

We can see what our new table looks like by running:

```shell
duckdb db/dictionary.duckdb -c "select * from words;"
```

Example output:

```
┌───────────────┐
│     word      │
│    varchar    │
├───────────────┤
│ a             │
│ aa            │
│ aal           │
│ aalii         │
│ aam           │
│ Aani          │
│ aardvark      │
│ aardwolf      │
│ Aaron         │
│ Aaronic       │
│ Aaronical     │
│ Aaronite      │
│ Aaronitic     │
│ Aaru          │
│ Ab            │
│ aba           │
│ Ababdeh       │
│ Ababua        │
│ abac          │
│ abaca         │
│   ·           │
│   ·           │
│   ·           │
│ zymosis       │
│ zymosterol    │
│ zymosthenic   │
│ zymotechnic   │
│ zymotechnical │
│ zymotechnics  │
│ zymotechny    │
│ zymotic       │
│ zymotically   │
│ zymotize      │
│ zymotoxic     │
│ zymurgy       │
│ Zyrenian      │
│ Zyrian        │
│ Zyryan        │
│ zythem        │
│ Zythia        │
│ zythum        │
│ Zyzomys       │
│ Zyzzogeton    │
└───────────────┘
   235975 rows 
   (40 shown)   
```

At this point, we could `dvc add db/dictionary.duckdb` and use git
add/commit to add the database to our repo, but DVC offers a better
way to do this that will save us work later on.  Let's delete the
database we just created:

```shell
rm -rf db/
```

Now let's create a [_DVC stage_](https://doc.dvc.org/start/data-pipelines/data-pipelines) to capture the relationship between `dictionary.txt` and `dictionary.duckdb`:

```shell
mkdir db/
dvc stage add --name db --deps source_data/dictionary.txt --outs db/dictionary.duckdb duckdb db/dictionary.duckdb -c "create table words (word varchar); insert into words from read_csv('source_data/dictionary.txt');"
```

Example output:

```
Added stage 'db' in 'dvc.yaml'                                                                  

To track the changes with git, run:

	git add dvc.yaml db/.gitignore

To enable auto staging, run:

	dvc config core.autostage true
```

Let's check in our changes to git:

```shell
git add . && git commit -m 'Add db creation stage.'
```

Example output:

```
[main d0071c3] Add db creation stage.
 2 files changed, 9 insertions(+)
 create mode 100644 db/.gitignore
 create mode 100644 dvc.yaml
```

Now we can check the status of this data processing stage by running:

```shell
dvc status
```

Example output:

```
db:                                                                                                                                  changed deps:
		modified:           source_data/dictionary.txt
	changed outs:
		deleted:            db/dictionary.duckdb
```

What is DVC telling us?  We have not modified `dictionary.txt`!  What
this means is that the `db` stage considers its dependencies to be
modified because they are newer than its output,
`db/dictionary.duckdb`, which does not exist!

The point of capturing data processing and experimentation steps in
DVC is _reproducibility_.  We can run the pipeline we defined to fix
the out-of-date outputs by running:

```shell
dvc repro
```

Example output:

```
'source_data/dictionary.txt.dvc' didn't change, skipping                                                                                                                                                                                                            
Running stage 'db':                                                                                                                                                                                                                                                 
> duckdb db/dictionary.duckdb -c "create table words (word varchar); insert into words from read_csv('source_data/dictionary.txt');"
Generating lock file 'dvc.lock'                                                                                                                                                                                                                                     
Updating lock file 'dvc.lock'

To track the changes with git, run:

	git add dvc.lock

To enable auto staging, run:

	dvc config core.autostage true
Use `dvc push` to send your updates to remote storage.
```

Now `dvc status` shows:

```
Data and pipelines are up to date.
```

Let's commit our changes!

```shell
git add . && git commit -m 'run pipelines to create database'
```

Example output:

```
[main 23efc3d] run pipelines to create database
 1 file changed, 15 insertions(+)
 create mode 100644 dvc.lock
 ```

### 7. Add some results

> Note: current directory should be the git repo you created in step 2

Let's do some statistical analysis of our data: what does the
distribution of word length look like?  DuckDB makes this sort of
thing pretty easy:

```shell
duckdb db/dictionary.duckdb -c 'select word_len, count(*) as count from (select len(word) as word_len from words) group by word_len order by word_len;'
```

Example output:

```
┌──────────┬───────┐
│ word_len │ count │
│  int64   │ int64 │
├──────────┼───────┤
│        1 │    51 │
│        2 │   160 │
│        3 │  1427 │
│        4 │  5285 │
│        5 │ 10239 │
│        6 │ 17713 │
│        7 │ 23881 │
│        8 │ 30001 │
│        9 │ 32412 │
│       10 │ 30882 │
│       11 │ 26018 │
│       12 │ 20468 │
│       13 │ 14941 │
│       14 │  9767 │
│       15 │  5926 │
│       16 │  3376 │
│       17 │  1814 │
│       18 │   842 │
│       19 │   428 │
│       20 │   198 │
│       21 │    82 │
│       22 │    41 │
│       23 │    17 │
│       24 │     5 │
│       28 │     1 │
└──────────┴───────┘
      25 rows     
```

Let's make a histogram bar chart to visualize this distribution.  For
this we can use kuva.  One of the nice things about kuva is that it
allows you to generate data plots directly to the terminal to preview
the visualization.  Let's try a horizonal bar char:

```shell
duckdb db/dictionary.duckdb -csv -c 'select word_len, count(*) as count from (select len(word) as word_len from words) group by word_len order by word_len;' | kuva bar --horizontal --terminal --x-label="count" --y-label="word length" --title "Dictionary Words by Length"
```

Example output:

```
                            Dictionary Words by Length                          
    28╶█                                                                        
    24╶█                                                                        
    23╶█                                                                        
    21╶█                                                                        
 w  20╶█                                                                        
 o  19╶██                                                                       
 r  17╶████                                                                     
 d  16╶███████                                                                  
    15╶████████████                                                             
 l  14╶██████████████████████████████                                           
 e  12╶█████████████████████████████████████████                                
 n  11╶████████████████████████████████████████████████████                     
 g  10╶█████████████████████████████████████████████████████████████            
 t   9╶████████████████████████████████████████████████████████████████         
 h   7╶███████████████████████████████████████████████                          
     6╶███████████████████████████████████                                      
     5╶█████████████████████                                                    
     4╶███████████                                                              
     2╶█                                                                        
     1╶█────────┼─────────┼─────────┼─────────┼─────────┼─────────┼────────┤    
       0      5000      10000     15000     20000     25000     30000    35000  
                                       count                                    
```


Perfect!  Let's add another DVC stage to store a PDF version of this plot under a new `figures/` folder:

```shell
dvc stage add --name word_count_hist --deps db/dictionary.duckdb --outs figures/word_count_hist.pdf "duckdb db/dictionary.duckdb -csv -c 'select word_len, count(*) as count from (select len(word) as word_len from words) group by word_len order by word_len;' | kuva bar --horizontal --x-label=\"count\" --y-label=\"word length\" --title \"Dictionary Words by Length\" -o figures/word_count_hist.pdf"
Added stage 'word_count_hist' in 'dvc.yaml'
```

And run the pipeline:

```shell
dvc repro
```

Now we can commit the results:

```shell
git add . 
git commit -m 'Add a plot.'
```

### 8. Push DVC Artifacts to a Remote

> Note: current directory should be the git repo you created in step 2

For others to reproduce our work, we need to make the DVC tracked
artifacts (and experimental results) publically available.  This is
very simple with DVC:

- Add a [Remote](https://doc.dvc.org/command-reference/remote#remote)
  storage location to the local DVC config
- Run: `dvc push`

Example:

```shell
dvc remote add --project -v -d gs.turtledb.keyvcr_data_sets gs://keyvcr_data_sets
```

To recreate the entire local working tree, you just have to run `dvc
pull` after `git clone`!

### 9. New Data Available; Re-run Experiments!

Let's say we've found a better dictionary and want to update our repo
and the experiments we ran.  First we download the new data,
overwriting the previous dictionary.txt:

```shell
curl https://storage.googleapis.com/docclab-turtles-datasets/better_dictionary.txt -o source_data/dictionary.txt
```

Now `dvc status` shows us the pipeline is out-of-date:

```
db:                                                                                                                              	changed deps:
		modified:           source_data/dictionary.txt
	changed command
source_data/dictionary.txt.dvc:
	changed outs:
		modified:           source_data/dictionary.txt
```

Let's update the `db` stage so it automatically deletes the old
database if one exists.  We can do this by editing `dvc.yaml`:

```yaml
stages:
  db:
    # EDITED:
	#    vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv
    cmd: mkdir -p db && rm -f db/dictionary.duckdb && duckdb db/dictionary.duckdb -c "create table words (word varchar); 
      insert into words from read_csv('source_data/dictionary.txt');"
    deps:
    - source_data/dictionary.txt
    outs:
    - db/dictionary.duckdb
  word_count_hist:
    cmd: duckdb db/dictionary.duckdb -csv -c 'select word_len, count(*) as count
      from (select len(word) as word_len from words) group by word_len order by 
      word_len;' | kuva bar --horizontal --x-label="count" --y-label="word 
      length" --title "Dictionary Words by Length" -o 
      figures/word_count_hist.pdf
    deps:
    - db/dictionary.duckdb
    outs:
    - figures/word_count_hist.pdf
```

And now `dvc repro`!

The database and figures have been updated (go ahead you can check!),
so let's `git add`, `git commit`, and `dvc push`.
