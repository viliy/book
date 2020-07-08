# 6. foreach

## 执行结果


理解一下foreach的整个执行流程：
* 将当前指针执行到第一个值
* 将第一个值赋值给$v

这样就不难理解发生的事了

```php

$data = ['a', 'b', 'c'];

foreach ($data as $k => $v) {
    $v = &$data[$k];  // $v = &$data[0] = 'a'; $v = &$data[1] = 'b'; $v = &$data[2] = 'c';
}

for($i = 0; $i < count($data); $i++) {
    $info = $data[$i];
    $info = &$data[$i];
}

print_r($data);

//    Array
//    (
//        [0] => c
//        [1] => c
//    [2] => c
//    )

```

## 关于key => value

foreach循环一次，数组中的指针移动一次，数组中的指针并不是$v指向$arr[$k]内存地址的那条线，不是$v的指向直接指向数组的下一个单元，而是数组的指针移动到下一个单元，如果为空，$k和$v的指向则不用移动，如果不为空，$k和$v的指向移动到数组的下一个单元

```php

$arr = ['a', 'b', 'c'];

foreach ($arr as $key => $value) {}

echo "$k - $value";

// 2 - c

```

```php

$a = [1, 2, 3];
$c = $a;

foreach ($a as &$value) {
    $a = [];
    echo $value;  // 1
}


foreach ($c as $key => $item) {
    $c[] = $item * 10;
}

var_dump(current($c));  // 1


$array = [0, 1, 2];
foreach ($array as &$val1)
{
    var_dump(current($array)); // 1
}

```


# PHP7 特性调整

* 一、foreach()循环对数组内部指针不再起作用。
* 二、按照值进行循环的时候，foreach是对该数组的拷贝操作
* 三、按照引用进行循环的时候，对数组的修改会影响循环。
* 四、对简单对象plain (non-Traversable) 的循环。
* 五、对迭代对象(Traversable objects)对象行为和之前一致。

1

```php

$array = [0, 1, 2];
foreach ($array as &$val) 
{
    var_dump(current($array)); // php7 : 1 1 1; php 1,2 false
}

```

2

```php

$array = [0, 1, 2];
$ref =& $array; // Necessary to trigger the old behavior
foreach ($array as $val) {
    var_dump($val);   // php7:0 1 2;  php: 0,2
    unset($array[1]);
}

```
3

```php

$array = [0];
foreach ($array as &$val) {
    var_dump($val);  // php7/:0 1;   php: 0
    $array[1] = 1;
}

```