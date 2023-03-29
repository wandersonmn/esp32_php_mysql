# ESP32-MySQL 
Send data to a MySQL database using ESP32

# Hardware
* ESP32
# Software
* ESP32 Micropython Firmware v1.16 (https://docs.micropython.org/en/latest/esp32/tutorial/intro.html)
* BIPES (https://bipes.net.br/beta2/ui/)
* XAMPP (https://www.apachefriends.org/pt_br/download.html)
#Overview

The solution involves the creation of a local server and a MySQL database through XAMPP. To process the HTTP requests received from ESP32 and store the data in a MySQL table, a PHP application will be developed on the local server. The following are the steps for implementing this solution:

![xampp_esp32](https://user-images.githubusercontent.com/50183447/228396942-f11e04e3-af53-486d-bd88-9a65fca36080.JPG)

# DATABASE

1. Download and install XAMPP 
2. Start the apache server and MySQL database on XAMP CONTROL PANEL
![xampp_soft](https://user-images.githubusercontent.com/50183447/228387989-45223eb4-a110-469a-a1b7-5b0014e60441.JPG)
1. Open the command line and navigate to the folder 
> $cd C:\xampp\mysql\bin
2. Set root user password
> $mysqladmin -u root password YOUR_PASSWORD
3. Log in to the MySQL Server (this will request your password that you've just set)
> $mysql.exe -u root -p

you can use the root user but it's recomended to create a new user, to create a new user:
> MARIADB[(none)]>CREATE USER 'new_username'@'localhost' IDENTIFIED BY 'new_user_password';<br />
> MARIADB[(none)]>GRANT ALL PRIVILEGES ON *.* TO 'new_username'@'localhost';<br />
> MARIADB[(none)]>FLUSH PRIVILEGES;<br />

4. Save the following code as .php file to the specified path C:/xampp/htdocs/
```php
<?php
$data = json_decode(file_get_contents('php://input'), true);

if(isset($data['parameters'])) {

   $parameters = $data['parameters'];
   $servername = $data['server'];
   $username = $data['user'];
   $password = $data['pass'];
   $database_name = $data['database'];
   $table_name = $data['table'];

#create string with columns and values to insert in the table
   $columns = "(";
   $values = "(";
   $create_fields = "";

   for($i=0; $i<count($parameters); $i++){
      $columns .= $parameters[$i]["column"].",";

      switch ($parameters[$i]["type"]){
         case "txt":
            $values .="'".$parameters[$i]["data"]."'";
            $create_fields .=",".$parameters[$i]["column"]." VARCHAR(100) DEFAULT \"\"";
            break;
         case "dat":
            $dado = $parameters[$i]["data"];
            array_splice($dado,3,1);
            array_splice($dado,6,1);
            $values.= "STR_TO_DATE('".implode(",",$dado)."','%Y,%m,%d,%H,%i,%s')";
            $create_fields .=",".$parameters[$i]["column"]." DATETIME";
            break;
         case "boo":
            $values .= $parameters[$i]["data"];
            $create_fields .=",".$parameters[$i]["column"]." BOOL";
            break;
         default:
            $values .= $parameters[$i]["data"];
            $create_fields .=",".$parameters[$i]["column"]." DOUBLE";
      }
      $values .=",";
   }

   $columns = rtrim($columns,",");
   $values = rtrim($values,",");
   $columns .=")";
   $values .=")";
    
   // create a connection with the database
   $connection = new mysqli($servername, $username, $password);
   // check a connection with the database
   if ($connection->connect_error) {
      die("MySQL connection failed: " . $connection->connect_error);
   }
   
   // Check if database exists
   $result = $connection->query("SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA WHERE SCHEMA_NAME = '$database_name'");
   
   if (mysqli_num_rows($result) > 0) {
      echo "Database exists <br>";
   } else{
      $connection->query("CREATE DATABASE $database_name");
      echo "Database created <br>";
      
   }

   $connection = new mysqli($servername, $username, $password, $database_name);

   $result = $connection->query("SHOW TABLES LIKE '$table_name'");
   if (mysqli_num_rows($result) > 0) {
      echo "Table exists <br>";
   }else {
   $connection->query("
   CREATE TABLE $table_name (
      id INT UNSIGNED NOT NULL AUTO_INCREMENT
      $create_fields
      ,PRIMARY KEY (id)
   )
   ");
   echo "Table Created <br>";
   }

   $sql = "INSERT INTO $table_name $columns VALUES $values";

   if ($connection->query($sql) === TRUE) {
      echo "Data inserted in the table";
   } else {
      echo "Error: " . $sql . " => " . $connection->error;
   }

   $connection->close();

} else {
   echo "Any data to insert";
}

?>

```
# BIPES


