+++
date = '2021-04-14T12:05:28-05:00'
draft = false
title = 'Keep calm and ODBC'
+++

This post has many parents. I want to update where I am on the project that started this blog: studying the diffusion of unfair labor practices in union settings. I also recently solved a problem that took far longer than it should have. I want to document what I did, to remind myself and maybe to help others. And I want to get back to blogging after a truly hellacious year, for me and for everyone. If a rant about database connectivity kickstarts things, so be it!

# Changing languages

In late 2018 I discovered a trove of data on unfair labor practices dating back to the early 1960s. I had sought these data a decade earlier, without success. Now, here they were: copious megabytes of eighty-character punched-card records that needed deciphering and translating into a modern format. I saw this as a means for me to keep learning R.

In 2019, though, I used Python for more projects. Changes in my research also affected my choice of programming languages. Inequality and segregation research can privilege accounting statistics over inferential statistics. This makes the data-management capabilities of general-purpose languages relatively more valuable, and the installed base of routines in focused statistical languages relatively less. I work with masters-level research assistants at McGill, more than I do with PhD students, at least on my own projects; and masters students are more interested in learning Python than R. Thus last summer I chose to clean these data with Python. My existing R code base was small, thus switching costs were low.

The [Pandas](https://pandas.pydata.org/) library expedites data analysis in Python. Something that kept me devoted to Stata for so long, though, is its felicity at *cleaning* archival data--finding typos, fixing data types, and the like. I've not yet found a Python library to speed up this process, though of course Python's strength lies in the ease of defining simple helper functions. (Such functions probably merit a post of their own.)

# Where do you put the clean stuff?

Python it is, then. Coming from languages like Stata or R, though, it is striking how Pandas does not have a specific data-file type.

This is good and bad. I've struggled with proprietary data formats. In particular, I think SAS files should come with a trigger warning! CSV is so very slow, though, partly because most programs have to guess variable types while reading in data. Categorical data types can be extra problematic when you write data to plain-text files. If categorical value labels are long, file sizes can explode. You can write label dictionaries, but you must then manage multiple files. The same point holds for data dictionaries.

The best compromise is a relational database. The SQL database structure is non-proprietary and widely supported. Most statistical languages have built-in facilities for I/O with such databases. I've actually felt guilty for years that I *don't* store my most often-used datasets in such a format. Since Python doesn't give me an easy binary format by default, this seemed the right project to start with.

Writing a Pandas dataframe to SQL is straightforward. I find [SQLite](https://www.sqlite.org/index.html) much easier to use than, say, MySQL, since accessing files with SQLite does not require your mimicking a client-server interface on your local machine. Python has the `sqlite3` library as part of the base installation. You just need to create a "connection" to the file through which you pipe SQL code:

```python
import sqlite3 as sql

def CreateConnection(db_file):
    conn = None
    try: 
        conn = sql.connect(db_file)
        return conn
    except sql.Error as e:
        print(e)
    return conn

def CreateTable(conn, create_table_sql):
    try:
        c = conn.cursor()
        c.execute(create_table_sql)
    except sql.Error as e:
        print(e)
```

Imagine you want to put four columns of a dataframe called "cases" into a table called "ulp": _id, caseNumber, dateFiled, and state. Create a connection to the relevant database file. Define the SQL to lay out the table and pass it to `CreateTable()`, which puts the table into the database file's schema. Then use the Pandas `to_sql()` method to pass data from the dataframe into the database:

```python
conn = CreateConnection('ULPs.db')
sql_create_table = '''CREATE TABLE IF NOT EXISTS ulp (
    _id INTEGER PRIMARY KEY,
    caseNumber TEXT,
    dateFiled TIMESTAMP,
    state INTEGER
    );'''
CreateTable(conn, sql_create_table)
cases[['caseNumber', 'dateFiled', 'state']].to_sql('ulp', conn,
       if_exists='append', index_label='_id')
```

Here I have "state" stored as an integer. Really this is a categorical variable, and I would like to keep the value labels associated with it. I have a dictionary called `state_map` with those labels. Writing this dictionary to its own table is simple enough:

```python
sql_create_table = '''CREATE TABLE IF NOT EXISTS state (
    state_id INTEGER PRIMARY KEY,
    state TEXT
    );'''
CreateTable(conn, sql_create_table)
for key in state_map:
    sequel = '''INSERT INTO state(state_id, state) VALUES(?,?)'''
    c.execute(sequel, (key, state_map[key]))
    conn.commit()
```

Notice here that `sequel` is a SQL command that itself takes arguments `(?,?)`. Thus `c.execute()` sends the command *and* a key-value pair from `state_map` to the database. In other words, the variable expansion here happens on the SQL side, not the Python side. Simple enough, though passing variables defined in one language to commands in another is tricky the first time!

Datasets that have many categorical variables tend to have many sets of value lables. It would be better to define a loop to build all such tables. I found this head-scratching enough that a solution seems worth sharing. For each categorical variable foo I create a dictionary, `foo_map`, with value labels. I then create a dictionary of dictionaries:

```python
table_maps = {'foo': foo_map, 'bar': bar_map, 'baz': baz_map, ...}
```

I then put the preceding code into a loop that iterates over the key-value pairs in this dictionary. I must now include Python variables in the SQL code along with the SQL variables, and reference nested dictionaries:

```python
for key in table_maps:
    sql_create_table = '''CREATE TABLE IF NOT EXISTS %s (
    %s_id INTEGER PRIMARY KEY,
    %s TEXT
    );''' % (key, key, key)
    CreateTable(conn, sql_create_table)
    for k in table_maps[key]:
        sequel = '''INSERT INTO %s(%s_id, %s) VALUES(?,?)''' % (key,
            key, key)
        c.execute(sequel, (k, table_maps[key][k]))
        conn.commit()
```

This code rewards working through the variable expansions. And yes, it makes sense to build this code into a function, and then call the function for each item in `table_maps`.

At this point, somewhere, you have a SQLite database. Huzzah!

# Fine, but how do you *read* it?

Now we get to the part that has driven me nuts multiple times: how do you read data from such a file?

In Python this is trivial: open a connection as above, define a SQL query, and pipe the results into a Pandas dataframe:

```python
# Assume conn already exists
df = pd.read_sql_query("SELECT * FROM ulp", conn)
```

In that case, you'd have the data that we wrote to the table `ulp` previously. In R, presuming that you have installed the `RSQLite` library, it's virtually the same:

```R
conn <- dbConnect(RSQLite::SQLite(), "ULPs.db")
data <- dbSendQuery(conn, "SELECT * FROM ulp")
```

And in Stata it's...oh, *God damn it*.

Stata lacks this type of SQL integration. Instead it uses open database connectivity or [ODBC](https://en.wikipedia.org/wiki/Open_Database_Connectivity), "a standard API for accessing database management systems." Let me be clear that ODBC is a *good thing*. Out in the real world, having an interoperability layer between databases and applications is extremely important, and drivers are widely available. For an individual researcher, though, getting ODBC working just to read a SQLite file can feel like Carl Sagan's "To make an apple pie, first create the universe" line.

Stata has a series of `odbc` commands, but (per the nature of ODBC) these just talk to the operating system's *ODBC device manager*. This is where the fun starts. These days, OS X does not come with an ODBC device manager; they dropped `iodbc` with Snow Leopard, because Apple.

There are two main ODBC device managers, `unixodbc` and `iodbc`. I'll use the former, which you can install with Homebrew. (Don't have Homebrew installed? Yak shaving time.)

Thus in Stata I can `set odbcmgr unixodbc` (becuase Stata looks for the erstwhile `iodbc` on Mac) and then use the ODBC device manager to talk to the database. But the device manager does not automatically how to do this. You must specify an *ODBC device driver*, which, inevitably, is not installed. Things get squirrelly around device drivers. You can find DevArt charging US$169 for a desktop license for a SQLite ODBC driver; but you can also install `sqliteodbc`, again from Homebrew, for free. YMMV, but for research projects, I think you'll find the free version sufficient.

With these installed, you're almost there. However, `unixodbc` is very Unix-y, in that it expects to find information in various initialization files, notably `odbcinst.ini` and `odbc.ini`. These can live in several places (type `odbcinst -j` in terminal to see where it looks by default), but--in a grudging concession to usability--it will preferentially use copies in your home directory. Mine look like this:

```
.odbcinst.ini:
[SQLite Driver]
Driver = /usr/local/Cellar/sqliteodbc/0.9998/lib/libsqlite3odbc.dylib

.odbc.ini:
[ULPs]
Driver = SQLite Driver
Database = /Users/jpferg/Dropbox/research/ulp_diffuse/data/ulps.db
```

Once you have all of the things in the right places, then you can pull the relevant data with `odbc load, table("ulp")`. Short, simple command, but there's a pile of infrastructure behind it!

# The curse of thin documentation

I swear, I didn't write all this just to complain. I've tried to figure out ODBC in Stata for years. Each time, I bounced off the complication of lining these pieces up. Multiple device managers, multiple device drivers, an implied client-server model on the local machine...I can perhaps be forgiven for throwing up my hands and sticking with .dta files.

Here's the thing though: when you look at what I wrote to read SQL database records into Stata, it doesn't look complex:

1. Install `unixodbc` and `sqliteodbc` with Homebrew
2. Write `~/.odbcinst.ini`. List where the device driver lives and give it an easy-to-use name.
3. Write `~/.odbc.ini`. List where the database file lives, the driver it needs (using that easy name), and an easy-to-remember name for the file.

Now you can type `odbc list`, `odbc query`, and other commands inside Stata. These show databases you can access, which tables are in each, the tables' structure and the like. `odbc` also lets you do database I/O with SQL commands. Adding databases or drivers is just a matter of adding lines to those files. The Python code I excerpted in this post is *way* more complicated than the process I describe here. So why am I complaining about *this?*

Though simpler, this process was much harder to learn. This is the cause and the effect of Stata's fading prevalance. Stata is more than 30 years old; many of its user conventions are almost pre-Internet. For eons the canonical place to trade information was Statalist, an email list. The hardcore user community never migrated to Stackexchange or other successors. Because new people use different software, the posts that do exist for Stata/ODBC connectivity are increasingly ancient. This doesn't cause problems for Stata itself but, because odbc is separate software, changes that affect *its* availability sow confusion. I mentioned for example that OS X dropped `iodbc` from the base install with Snow Leopard. I learned that from posts on Statalist about people trying to configure `odbc` on Mac. But Snow Leopard was released twelve years ago!

Maybe Stata will include SQL support in version 17. They could bundle their own device drivers. But why? Almost anyone who uses Stata with SQL databases already knows how `odbc` works, and new people are unlikely to install Stata.

There is plenty of information about ODBC and SQL online, of course. But, crucially, there is little about using those them *with Stata*. That is where things break down. I don't want to learn a ton about ODBC--I just want to read a database! Yet Stata's own documentation stops at the modularity frontier.

As of this writing, nine of the first ten hits from a Google search of "reading SQL with R" are third-party pages walking through ways to do this. The oldest is 2015; most were posted within the last three years. By contrast, three of the first ten hits for "reading SQL with Stata" are from stata.com, and direct toward the `odbc` documentation. Five of the remaining seven are from between 2014 and 2017, and virtually all of them are rants about how Stata does *not* support SQL! One of the two remining is a query (from 2017) on Statalist asking about reading SQL. Tellingly, the person on the list who recommends `odbc` follows up with "I haven't used the `odbc` command in many years--it just doesn't come up in my work. So all I can tell you is that this is what `odbc` does. But I no longer remember any of the details of how it is used."