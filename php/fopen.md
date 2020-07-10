# 2. fopen

## 排它锁

```php

      $fp = fopen("/tmp/lock.txt","w+");
      if(flock($fp, LOCK_EX)){// 进行排它型锁定
          fwrite($fp,"Write something here\n");
          flock($fp, LOCK_UN);// 释放锁定
      }else{
          echo "Couldn't lock the file !";
      }
      fclose($fp);

```


## 递归目录文件

```php

     function get_dir_info($path){
           $handle = opendir($path);//打开目录返回句柄
           while(($content = readdir($handle))!== false){
                 $new_dir = $path . DIRECTORY_SEPARATOR . $content;
                 if($content == '..' || $content == '.'){
                        continue;
                 }
                 if(is_dir($new_dir)){
                       echo "<br>目录：".$new_dir . '<br>';
                       get_dir_info($new_dir);
                 }else{
                       echo "文件：".$path.':'.$content .'<br>';
                 }
           }
       }
       
       $dir = '';
       get_dir_info($dir);


```