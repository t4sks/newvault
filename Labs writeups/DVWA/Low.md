# Оглавление:



----
# Command Injection Low 

```php
<?php      if( isset( $_POST[ 'Submit' ]  ) ) {    // Get input    
$target = $_REQUEST[ 'ip' ];    // Determine OS and execute the ping command.    
if( stristr( php_uname( 's' ), 'Windows NT' ) ) {        // Windows        
$cmd = shell_exec( 'ping  ' . $target );       }       
else {        // *nix        
$cmd = shell_exec( 'ping  -c 4 ' . $target );       }    // Feedback for the end user    
echo "<pre>{$cmd}</pre>";   }      ?>`|
```

Здесь команда исполняется напрямую в терминале из за чего мы можем просто использовать не IP а использовать знак &, чтобы запустить вторую команду внезависимости от выполнения первой и получить все что нам нужно, по факту RCE

---
# File Inclusion Low

```php
<?php      // The page we wish to display   
$file = $_GET[ 'page' ];      
?>
```

по факту проверки нет мы можем указать любой файл на локальной машине и он его прочитает если у нас достаточно прав, то есть никакой валидации тут нет 
# SQL injection

```php
<?php  
  
if( isset( $_REQUEST[ 'Submit' ] ) ) {    
// Get input    
$id = $_REQUEST[ 'id' ];    
// Check database
$query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";    
$result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );    
// Get results    
while( $row = mysqli_fetch_assoc( $result ) ) {
         // Get values
         $first = $row["first_name"];        $last  = $row["last_name"];
         // Feedback for end user
         echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";  
    }    mysqli_close($GLOBALS["___mysqli_ston"]);  
}  
  
?>
```

ошибка в том что здесь параметр id попадает в запрос и никак не параметризуется а просто объединяются строки, можем так же реализовать любую атакую SQL
# Blind SQL Injection

```php
<?php  
  
if( isset( $_GET[ 'Submit' ] ) ) {    
// Get input    
$id = $_GET[ 'id' ];    
// Check database    
$getid  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $getid ); 
// Removed 'or die' to suppress mysql errors  
  
    // Get results
        $num = @mysqli_num_rows( $result ); // The '@' character suppresses errors    
        if( $num > 0 ) {        // Feedback for end user        
        echo '<pre>User ID exists in the database.</pre>';  
    }  
    else {        // User wasn't found, so the page wasn't!       
     header( $_SERVER[ 'SERVER_PROTOCOL' ] . ' 404 Not Found' );        
     // Feedback for end user        
     echo '<pre>User ID is MISSING from the database.</pre>';  
    }  
  
    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);  
}  
  
?>
```

здесь уязвимость в том что, мы должны по ответу приложения понять что происходит, моя timebased сработала, тут так же пользовательский ввод без параметризации вставляется в SQL запрос