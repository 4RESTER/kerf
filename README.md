![REPL](https://github.com/kevinlawler/kerf/raw/master/repl-header.png)

What is Kerf?
-------------

Kerf is a columnar tick database for Linux/OSX/BSD/iOS/Android. It is written in C and speaks JSON and SQL.

See Kerf on TimeStored: http://www.timestored.com/time-series-data/kerf-database

**Contact Kevin (e.g., licensing, feature/documentation requests):**

  k.concerns@gmail.com

Whirlwind Language Guide:
-------------------------
**TYPES**


<pre><code>  CHAR 
         "abc" or 'abc'
  
  INT  
          1
  
  FLOAT
          2.0 or 1e6 or 1.2E+01
  
  STAMP
          2015.03.31 or 01:23:45.877 or 2015.03.31T01:23:45.877
  
  NULL
          null
  
  ARRAY 
          [5, 6, 7, 8]
  
  MAP  
          {b:2, c:3, d:4}
  
  TABLE   
         {{b:2, c:3, d:4}}
</code></pre>
  
  There are a few other types which we'll skip discussing for now.


**OPERATIONS**

  Let's look at some good ways to make arrays. 'Range' comes directly from Python
  and accepts 1, 2, or 3 arguments:

    range(4)
      [0, 1, 2, 3]
  
    range(2, 6)
      [2, 3, 4, 5]
  
    range(0, 20, 3)
      [0, 3, 6, 9, 12, 15, 18]

  'Rand' accepts 0, 1, or 2 arguments:

    rand()       //FLOAT from [0,1)
      0.164771
  
    rand(5)      //INT
      2
  
    rand(9.0)    //FLOAT
      8.86153
  
    rand(4, 3.0) //4x FLOAT
      [2.44598, 2.87178, 1.14531, 0.676305]

    rand(4, [11, 22]) //from a list
      [22, 11, 11, 22]

  Rand may return different values for you.

  Now let's look ahead:

  The exponentiation operator ** comes from Ruby and Python.

    range(10**6) //a big list

    timing true
    sum(range(10**6))  //sum first million numbers
      499999500000
    
  Now back to basic arithmetic:

    + - * / ** plus minus times divide pow
            //wait, which is the operation and which is the name?
    2 + 2
    plus(2, 2)
            //I guess they both work
    2 plus 2
            //and that works too

  This is nice because parenthesized prefix notation disambiguates dyadic/binary infix operations.

            //oh, one of those guys

Then the ambiguous-appearing

    0.5 * x**2 

becomes

    times(1/2, x**2) 
            //well that's not so bad
            //
            //actually, I like that better... I wonder why?

            
Or, emphasizing the center operation:
  
  
    (divide(1, 2) * pow(x,2))


Neither of these are ambiguous. Of course, you can always fall back to parentheses:
 
 
    ((1 / 2) * (x**2))


Format code as if order-of-operations does not exist. 

I find the "functional" notation for arithmetic also helps when the arguments are arrays or maps instead of scalars. This can cue the reader that something heavier-duty is happening. 

  Other common operators are present. The exclamation point '!' is not,
  the percent sign '%' is modulo, and so on.

    !0
     1
    not 0
     1

    -33 % 4
      3  //mathematical definition
    -33 mod 4
      3

    .Math.TAU / 2
      3.14159

**JSON**

  Kerf speaks JSON:

    eval('1+1')
      2
  
    a: [[1, 2, 3], {a:"alpha", b:"bravo", c:"3pO"}, null]
    match(a, eval(json_from_kerf(a)))
      1

**ASSIGNMENT**

  Assignment is ':', the colon character. It's colon and
  not '=' because:

  1. JSON uses : for assignment, as in {a:1} 
  2. SQL  uses = for comparison, as in WHERE user_id=456
  3. Kerf is a superset of both JSON and SQL.

Which looks like

    a: [11, 22, 33, 44]
    a[2]
        33


  We could force assignment to be '=' but I don't think it improves
  the language.

    a[0]:5
    a
      [5, 22, 33, 44]

  Indexing into maps:
  
    a: {b:2, c:3}
    a['c']
      3
    {b:2, c:3}['c']
      3

**VECTOR OPERATIONS**

  Arrays vectorize automatically. This means CHAR, INT, FLOAT, and STAMP types are fast and efficient in lists of the same kind. Arrays also mostly keep track of when they're sorted. This means Kerf will invisibly use binary search or interpolation search if it appears advantageous.

  The following notion of conformability comes from K: 

  Adding a single value to a longer list applies it like so:
  `100  + [0, 10, 20]`
  gives
  ` [100, 110, 120]`

  Kerf extends this notion to work with lists of length 1 as well:
 
 
    [100] + [0, 10, 20]
 
 
  gives


      [100, 110, 120]


  This also works piecewise:
  
  
      [100, 110, 120] + [40, 50, 60]
  
  
  gives
  
  
        [140, 160, 180]


  Conformability extends all the way down. This
  
  
    [[1], [1,1,1]] + [[2], [2, 2, 2]]

  
  gives 


      [[3], [3, 3, 3]]

  Flatten? Sure
  
  
    flatten [[3], [3, 3, 3]]
      [3, 3, 3, 3]


  Works with maps and tables, too.
  
    {a:2, b:20} + {a:3, b:30, c:100}
      {a:5, b:50, c:100}


Some operations yield array-wise results. 


    ![0, 1, 0]
      [1, 0, 1]

    [2, 3, 4, 4, 4] <= [3, 3, 3, 3, 3]
      [1, 1, 0, 0, 0]


  Operations are optimized for vectors
  
  
    timing 1
    a: range(10**6)
    sum(a)
    a+a
    min(a)


**TABLES**

  Kerf extends JSON to include the concept of tables. Tables are created just like maps except you use double curly-braces. The names of the keys in that case are instead the names of the columns. So `{{a:1, b:2}}` is a table and `{a:1, b:2}` is a map. The convenience constructor `{{a,b,c}}` also creates a table. This table will have empty arrays for columns. Table columns are always arrays. If you pass something that isn't an array it will be coerced into an array.

  The following are all equivalent ways to make a table:
  

      {{id:1, time:now(), brightness:48.6}}
    is the same as 
      {{id:[1], time:[now()], brightness:[48.6]}}
    is the same as 
      INSERT INTO {{id, time, brightness}} VALUES (1, now(), 48.6)       //single insert
    is the same as 
      INSERT INTO {{id, time, brightness}} VALUES [[1], [now()], [48.6]] //bulk insert/append
    is the same as
      INSERT INTO {{id, time, brightness}} VALUES {id:1, time:now(), brightness:48.6}   //insert map
    is the same as
      INSERT INTO {{id, time, brightness}} VALUES {{id:1, time:now(), brightness:48.6}} //append table
    is the same as
      INSERT INTO {{}} VALUES {id:1, time:now(), brightness:48.6} //empty tables are special
    is the same as
      a:{{}}
      INSERT INTO a VALUES {id:1, time:now(), brightness:48.6}
    is the same as
      id:[1]
      time:[now()]
      brightness:[48.6]
      {{id:id, time:time, brightness:brightness}}


  They are all pretty printed [ugly in the alpha] as


    |id|time|brightness|
    [1, 2015.04.01T00:45:15.598, 48.6]


  SQL inserts and updates are forms of assignment: they are always "saved". Bulk inserts are much faster than single inserts. The columns id, time, and brightness are vectorized as INT, STAMP, and FLOAT vectors respectively. The preceding tables all exist in-memory only.
  
**READS/WRITES**

   You can read and write arbitrary objects, including in-memory tables, using the following functions. These are not really designed for transactional reads and writes, more like per-session reads and writes.


      read_from_path('path.to.file')
      write_to_path('path.to.file', object);
      

For an on-disk table that handles transactional writes, you'll want a mapped object. Warning: currently the iOS operating system restricts _virtual_ memory allocations to something less than 2G in size, _even on_ devices with 64-bit pointers. So mapping very large tables will not get far around the memory limitations of the mobile device. Apple really should look into raising it: it may be a legacy restriction from some now-outdated concerns. On OS X the virtual memory limit is effectively unrestricted.

You can open tables on disk via the `open_table(filepath)` call. Here it is via the Objective-C API:


    NSString *path = [[kerf suggestedTableDirectoryPath] stringByAppendingPathComponent:@"my.table"];
    [kerf jsonObjectFromCall:@"a: open_table($1)" withArgumentArray:@[path]]
    [kerf jsonObjectFromCall:@"insert into a values {{id: 4}}"]);
    [kerf jsonObjectFromCall:@"a"]


  Modifications to the variable cause the inserts to persist to the disk. They will be there the next time you open the table. Most variables in Kerf use reference counting or copy-on-write to ensure uniqueness. Mapped values like opened tables are different: all reference the same open item. Changes to one affect the other.


**TIME MATH**

  We previously saw absolute time stamps of the form
  
  
    2015.04.01 or 2015.03.31T01:23:45.877
  
  
  We can compare them


    2015.04.02 < 2015.05.01 
      1


  We can modify them using relative times of the form `1y` or `1y3d` or `4h55i06s` and so on. So 
  
  
    2015.04.01 + 1y1m1d
      2016.05.02
  
    
  Or alternatively as
  
  
      2015.04.01 + 1y + 1m + 1d
        2016.05.02
  
  
  And `2015.04.01 + 1h2i3s` gives `2015.04.01T01:02:03.000`.


    now_date() + 1d
      2015.04.02

    now_time()
      21:36:00.762

    minus(now_time(), 25 * 1h)
      20:36:02.005


  The current list of possibilities is: `ymdhis` year month day hour minute second.
  
  Note that while `minus` could be perfectly well defined as an operation on absolute STAMPs (for a given reduced form, it could return a relative stamp), at least for the time being in Kerf it throws a type error. This is easy to get around, do
  
  
      2015.04.07 + 1d <= 2015.04.08
  
  
  instead of 
  
  
      (2015.04.08 - 2015.04.07) <= 1d 
  
  
The reason you might want to avoid producing relative times is that the full range of useful relative dates and times exceeds a 64-bit width and so is not vectorized.
  
To extract individual parts from times, use  

  
    a: 2015.03.16T04:05:06.7890123456
    a: [a, a] //optional, to see how it works in vector form
    
    a['date']   //stamp
    a['time']   //stamp
    a['year']   //int
    a['month'] 
    a['day']
    a['hour']
    a['minute']
    a['second']
    a['millisecond']
    a['nanosecond'] 
    
  
**SQL (SELECT)**

  Kerf is a superset of SQL. This means Kerf speaks SQL, and you can write SQL anywhere inside of Kerf code. Let's start by building a suitable table.


    n: 10**4
    ids: range(1, n+1)
    stamps: plus(NOW(), 1s + mapright range(n))
    heartrates: 80 + rand(n, 100.0)  
    labels: range(6)
    lanes: take(n, join(labels, reverse labels))
    running: {{id: ids, stamp: stamps, heartrate: heartrates, lane: lanes}} 


  Someone is running a zigzag across a six-lane track with a random heartbeat. This is not exactly realistic data but let's go with it. We can count the number of rows in the table:


    select count(*) as rows from running
    |rows|
    [10000]


And there's no reason we can't run SQL inside of JSON:


    [{a:1, b: select count(*) from running}, select count(*) from running]


  Let's verify the count of the table:


    equal(count(running), n)


  Peek at the first 3 rows:


    first(3, running)
      |id|stamp|heartrate|lane|
      [1, 2015.04.01T19:13:33.917, 96.4771, 0]
      [2, 2015.04.01T19:13:34.917, 107.397, 1]
      [3, 2015.04.01T19:13:35.917, 108.356, 2]
  
  
  Get the bounds on the time:


    select first(stamp), last(stamp) from running
      |stamp|stamp1|
      [2015.04.01T19:13:33.917, 2015.04.01T22:00:12.917]


  Alternatively


    [first(running.stamp), last(running.stamp)]
    [2015.04.01T19:13:33.917, 2015.04.01T22:00:12.917]


  And perform GROUP BY and WHERE queries:


    select avg(heartrate) from running where heartrate > 100 group by lane
      |lane|heartrate|
      [1, 139.192]
      [2, 140.283]
      [4, 139.772]
      [5, 140.244]
      [3, 140.167]
      [0, 138.541]

  Nested subqueries:
  
  
    select * from (select avg(heartrate) from running where heartrate > 100 group by lane) where heartrate = max(heartrate)
      |lane|heartrate|
      [2, 140.283]


  We can store the results of queries in other variables.


    b: select max(heartrate) from running where lane = 2
      |heartrate|
      [179.952]


  And retrieve the cell value only like so:
  
  
    first(b.heartrate)
      179.952


  The supported SQL WHERE comparison methods currently are:
  
  
      < > = <= >= == != <> 


  The supported SQL GROUP BY aggregation methods currently are: 
  
  
      min max sum count first last avg std var
   
   
  Kerf can use our nicely sorted ID range to perform fast lookups even without an index. Table traits are undocumented at this point.
  
**ADVANCED TYPES**

There are two advanced types which we can use for specialized columns: 


    ENUM (HASH)
                  enum ['red', 'blue', 'red']  or hash ['red', 'blue', 'red']
    INDEX (SORT)
                  index [1, 2, 3]
    

Both are variations on ARRAYs or VECTORs. An `enum` is like a "local" string interning object. It keeps only one reference to each object and stores appearances as fixed-width indices. It is useful for storing repetitions of strings and lists, which cannot otherwise efficiently be stored as vectors. In all other respects an `enum` appears to be an array.

An `index` is like an array except with an attached b-tree. This can make lookups and range queries more efficient. (The storage format of the index will be breaking after the alpha.) Don't use an `index` for data you can guarantee will always be sorted ascending, such as autoincrementing primary keys: Kerf will track sorted arrays and doesn't need a special index.

Lambdas are also a type which can be stored.

There is another hypothetical advanced type called an `ATLAS`, which is the schemaless NoSQL equivalent of a table. Atlases are automatically indexed in such a way that all key-queries are indexed. 

**INTERPROCESS COMMUNICATION**

Kerf instances are designed to be networked. The data structures serialize directly without any intermediate conversion.

To start a Kerf server on port 1234 execute the command:

    ./kerf -p 1234

You can communicate with this instance either via a Kerf client or via the Kerf SDK/API from another program (e.g., Python or Java or Objective-C).

A client can be a plain old Kerf instance:

    ./kerf
  
In the client paste each of the lines individually:
    
    socket: open_socket("localhost","1234")

    send_async(socket, "table: {{sym:hash[], time:[], price:[]}}")

    do(100) {send_async(socket, "insert into table values {sym:$1, time:$2, price:$3}", [rand(["AAPL","MSFT","IBM"]), now(),  20.0 + rand(10.0) ])}
    
    
Then in the server execute:

    root
    
    table
    
    select avg(price) from table group by sym
  
    
Then in the client execute

    close_socket(socket)

Currently IPC requires the user to store the socket handle. Probably what will happen is we will remove this and have all IPC calls use the server and port. It would be simple for Kerf to manage a hashtable of hosts and ports pointing to socket handles, and to keep or refresh them as necessary, and so we should probably do that.


**CONTROL FLOW**

  Control flow is designed to be as generic as possible. You probably don't need it yet, but Kerf uses: 
  
  
    if(b){x} else if(c){y} else{z} 
    do(n){x} 
    while(b){x}
    for(a;b;c){x} 
    def myfunc(arg1, arg2) {x}
    function myfunc(arg1, arg2) {x}
  
  
  All portions must be properly (parenthesized) and {curly-braced}: no skipping. Lambdas are: `{[arg1, arg2] arg1+arg2}`. Lambda function recursion is `self` or `this`. Early return is `return`, otherwise return the final eval. Note: ending the final eval with a semicolon causes null to be returned. Commas `,` and semicolons `;` are usually interchangeable. Comments are `//`.

**ALL FUNCTIONS / RESERVED WORDS**

  The following are all functions or reserved words that work and you can try. You can grab the full list of reserved symbols from inside Kerf by calling the "reserved()" method. Currently, the full list in the alpha is:


    
    inf nan nil null root true false select update insert upsert delete from
    group where order limit values not hash distinct part car transpose negate
    eval reverse ident ascend index descend which enumerate floor len atom join
    mod mins times plus minus alter divide less equals greater rand take drop
    maxes match lesseq greatereq equal noteq exp unique count first last avg
    std var min max sum enlist or and explode add subtract negative string
    flatten hashed enum btree indexed key primary nonnull unique global globals
    range repeat tolower toupper pow abs ceil sqrt ln log lg sin cos tan asin
    acos atan sinh cosh tanh timing now now_date now_time kerf_from_json
    json_from_kerf reserved sleep open_table read_from_path write_to_path fold
    refold mapdown mapright mapleft mapback reduce rereduce converge reconverge
    self this def function if do while for else return 



  These all work, with a few exceptions, but are mostly not yet documented.

**MISC CODE SAMPLES**

Examples for average, standard deviation, and variance (avg, std, var).


    a: [12, 2.4, 8]   //assign an array to 'a' using JSON notation
    a: range(6)       //integers [0, 1, 2, 3, 4, 5]
    a: rand(8, 100.0) //eight random floats from the interval [0.0, 100.0)
  
    REPL or API (Cheating):
      avg(a)
      std(a)
      var(a)
  
    REPL or API (Simplified):
      (sum a)/count a              
      sqrt var a                   
      (sum (a - avg a)**2)/count a 
    
    REPL or API (Traditional):
      sum(a)/count(a)               
      sqrt(var(a))                  
      sum((a - avg(a))**2)/count(a)
    
    API Argument Passing, each with 1 argument (Traditional):
      sum($1)/count($1)               
      sqrt(var($1))                  
      sum(($1 - avg($1))**2)/count($1)
    
    Function Definition (Traditional):
      def my_func_avg(a) {sum(a)/count(a)}
      def my_func_std(a) {sqrt(var(a))}
      def my_func_var(a) {sum((a - avg(a))**2)/count(a)}
    
    Lambdas (Traditional):
      {[a] sum(a)/count(a)}
      {[a] sqrt(var(a))}
      {[a] sum((a - avg(a))**2)/count(a)}
    
    REPL or API (Variant Takes)
      (plus fold a)/len(a)
      sum((a minus avg(a)) pow 2) divide count(a)
      pow(std a, 2) 
  

**MISC LANGUAGE SPECS**

  Kerf is written in C. Kerf is a superset of both JSON and SQL. Kerf compiles
  to Kerf bytecode. Memory management is automatic and invisible to the user.
  Internally, Kerf uses reference counting. Kerf does not garbage collect. Kerf
  uses copy-on-write. Kerf uses a memory pool, so warmed operations are faster.
  Kerf does not expose pointers. It does not use globally interned strings.
  Certain objects will intern strings locally. Kerf data structures use
  optimized hash tables and b-trees. All Kerf objects serialize automatically
  and use the same [decompressed] representation in-memory, on-disk, and over
  the network. By default the PRNG is initialized with a nondeterministic seed.
  By default times are UTC.
