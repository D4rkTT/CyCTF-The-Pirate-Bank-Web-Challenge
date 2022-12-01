# The pirate bank part 2 (Finals - 300 point)

## Analysis
As part 1 we got updated source code of first part with new features and old bug fixed.

The new feature allow us to export user transactions into PDF file, the main goal here to get 1M$ again.

After some code analysis i found that `transactions.php` accept parameter `order_type` and check it if it equal `asc` or `desc` using code:

```php
if (isset($_REQUEST['order_type'])) {
  $order = $_REQUEST['order_type'];
  filter_order($order);
}

function  filter_order($order)
{
  if ($_REQUEST['order_type'] == 'asc' or $_REQUEST['order_type'] == 'desc') {
    return true;
  } else {
    return false;
  }
}
```
But the app doesn't check for `filter_order()` result, so we can easly set  `$order` to any value.

## Exploit
Next stage is loading transaction of the user with function `get_transactions()` with 2 params. `$_SESSION['username']` and `$order` but wait the function pass the `$order` to sql query without any filter :D

```php
$query = "SELECT * FROM `transactions` where from_account=$id or to_account=$id order by trn_date $order";
```
Now we have sql injection but wait a min. we have very limited injection options as our injected code come after `order by`, i tried a lot of options but finally i got an idea to exploit it using `LIMIT` option.

We will guess the users passwords by showing transactions count equal to ascii code of the char. in every password chars. with this query 

```sql
limit (SELECT unicode(substr(password,1,1)) FROM users where id=1)
```
But we need to store a lot of transactions into the database so i made a lot of transactions with 0$ to any user :D

We know that the passwords are md5 hash so the length will be from `substr(password,1,1)` to to `substr(password,33,1)`.

Next challenge is getting transactions count from the PDF so i used `pdfplumber` in python.

After getting our first user hash, i realised that the hash is not crackable then second user (id=2) also same thing so i asked challenge owner he said that not all hashes are crackable.

So i need to get all users passwords and try to crack them all.

## Scripting
In 10 min i wrote a script to automate all the process and store all passwords hashes with users id to a file and another file with hashes only to easily use it on online cracking service, it also support multitasking so we can get all users as fast as possible.

You can find the script in this repo `exploit_part_two.py`

## Flag
After collecting 1M$ i found the flag on home page :D
