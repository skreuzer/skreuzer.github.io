---
layout: post
title: Storing IP addresses in MySQL
tags: ["mysql"]
---
Lets say you have an IP address, 192.168.0.10, and want to store it in a
table. The most common method people seem to use it to store it as a CHAR(15).

However, you probably want to search on this column and therefore want
an index on it.

MySQL has two built-in functions, INET_ATON() which converts Internet addresses
from the numbers-and-dots notation into a 32-bit unsigned integer, and INET_NTOA()
which does the opposite.

Putting it to the test:

{% highlight mysql %}
mysql> SELECT INET_ATON('192.168.0.10') AS ipn;
+------------+
| ipn        |
+------------+
| 3232235530 |
+------------+

mysql> SELECT INET_NTOA(3232235530) AS ipa;
+--------------+
| ipa          |
+--------------+
| 192.168.0.10 |
+--------------+
{% endhighlight %}

So you can store an IP address in an INT UNSIGNED (4 bytes) which is
more efficient and faster than a CHAR(15). Naturally, you can
call the function while you're inserting, so something like this is fine
also:

{% highlight mysql %}
INSERT INTO tbl VALUES (..., INET_ATON('192.168.0.10'), ...)
{% endhighlight %}

In MySQL 5.0, you can even do this transformation inside a LOAD DATA INFILE
command, without using temporary columns:

{% highlight mysql %}
LOAD DATA INFILE 'filename'
INTO TABLE tbl
...
(col1, ..., @ipa1, ..., coln)
SET ipn = INET_ATON(@ipa);
{% endhighlight %}

So in the list of columns you assign this column to a server-side
variable, and then assign the transformed value to the proper column in
the SET clause.
